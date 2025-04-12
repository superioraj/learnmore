# Microbit_Joypad Program Code Python
```
import sys
import time
import pyautogui
from PyQt5.QtWidgets import (
    QApplication, QWidget, QPushButton, QLabel, QLineEdit,
    QVBoxLayout, QHBoxLayout, QComboBox, QPlainTextEdit
)
from PyQt5.QtCore import Qt, QTimer, QThread, pyqtSignal
from PyQt5.QtGui import QKeySequence
import serial
import serial.tools.list_ports

###############################################################################
# 1. 깜빡이는 키 매핑 입력 위젯 (BlinkingKeyMappingLineEdit)
###############################################################################
class BlinkingKeyMappingLineEdit(QLineEdit):
    _current_capturer = None

    def __init__(self, parent=None):
        super().__init__(parent)
        self.setReadOnly(True)
        self.setAlignment(Qt.AlignLeft)
        self._capturing = False
        self._prev_value = self.text()
        self._blink_timer = QTimer(self)
        self._blink_timer.setInterval(500)
        self._blink_timer.timeout.connect(self._toggleBlink)
        self._blink_visible = True
        self._origin_placeholder = "키를 입력해주세요"

    def mousePressEvent(self, event):
        super().mousePressEvent(event)
        if BlinkingKeyMappingLineEdit._current_capturer and \
           BlinkingKeyMappingLineEdit._current_capturer is not self:
            BlinkingKeyMappingLineEdit._current_capturer.cancelCapture()
        BlinkingKeyMappingLineEdit._current_capturer = self
        self._startCapture()

    def keyPressEvent(self, event):
        self._blink_timer.stop()
        self._blink_visible = True
        key_seq = QKeySequence(event.modifiers() | event.key())
        new_value = key_seq.toString()
        if new_value == "" and event.key() == Qt.Key_Space:
            new_value = "space"
        self.setText(new_value)
        self._prev_value = new_value
        self._capturing = False
        self.releaseKeyboard()
        self.setPlaceholderText("")

    def _startCapture(self):
        self._prev_value = self.text()
        self.clear()
        self._capturing = True
        self.grabKeyboard()
        self._blink_timer.start()
        self._blink_visible = True
        self.setPlaceholderText(self._origin_placeholder)

    def cancelCapture(self):
        self._blink_timer.stop()
        self._blink_visible = True
        self.releaseKeyboard()
        if self._capturing:
            if self._prev_value:
                self.setText(self._prev_value)
            else:
                self.clear()
        self._capturing = False

    def _toggleBlink(self):
        if self._blink_visible:
            self.setPlaceholderText("")
        else:
            self.setPlaceholderText(self._origin_placeholder)
        self._blink_visible = not self._blink_visible

###############################################################################
# 2. 시리얼 통신 스레드
###############################################################################
class SerialThread(QThread):
    data_received = pyqtSignal(str)
    
    def __init__(self, port, baudrate=9600, parent=None):
        super().__init__(parent)
        self.port = port
        self.baudrate = baudrate
        self.running = True

    def run(self):
        try:
            ser = serial.Serial(self.port, self.baudrate, timeout=1)
            while self.running:
                if ser.in_waiting:
                    line = ser.readline().decode('utf-8').strip()
                    if line:
                        self.data_received.emit(line)
        except Exception as e:
            print("시리얼 통신 오류:", e)
    
    def stop(self):
        self.running = False
        self.quit()
        self.wait()

###############################################################################
# 3. 메인 GUI (JoypadWindow)
###############################################################################
class JoypadWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Micro:bit Joypad Controller")
        self.mapping = {
            'A버튼': '', 'B버튼': '', '로고터치': '',
            '위로 기울이기': '', '아래로 기울이기': '',
            '왼쪽 기울이기': '', '오른쪽 기울이기': '',
            '흔들기': '', '조도 20 미만': '', '조도 21 이상': ''
        }
        self.delay = 100
        self.serial_thread = None
        self.initUI()

    def initUI(self):
        main_layout = QVBoxLayout()

        # COM 포트 선택 영역
        com_layout = QHBoxLayout()
        com_label = QLabel("COM 포트:")
        com_label.setAlignment(Qt.AlignRight)
        self.com_combo = QComboBox()
        self.refresh_com_ports()
        com_layout.addWidget(com_label)
        com_layout.addWidget(self.com_combo)
        main_layout.addLayout(com_layout)

        # 시리얼 연결/해제 버튼 영역
        button_layout = QHBoxLayout()
        self.connect_button = QPushButton("시리얼 연결")
        self.connect_button.clicked.connect(self.start_serial)
        button_layout.addWidget(self.connect_button)
        self.disconnect_button = QPushButton("시리얼 끄기")
        self.disconnect_button.clicked.connect(self.stop_serial)
        button_layout.addWidget(self.disconnect_button)
        main_layout.addLayout(button_layout)

        # 이벤트별 매핑 입력란 영역
        self.inputs = {}
        for key in self.mapping:
            row_layout = QHBoxLayout()
            label = QLabel(f"{key} 매핑:")
            label.setAlignment(Qt.AlignRight)
            key_input = BlinkingKeyMappingLineEdit()
            self.inputs[key] = key_input
            row_layout.addWidget(label)
            row_layout.addWidget(key_input)
            main_layout.addLayout(row_layout)

        # 키 입력 딜레이 설정 영역
        delay_layout = QHBoxLayout()
        delay_label = QLabel("키 딜레이 (ms):")
        delay_label.setAlignment(Qt.AlignRight)
        self.delay_input = QLineEdit(str(self.delay))
        self.delay_input.setAlignment(Qt.AlignRight)
        delay_layout.addWidget(delay_label)
        delay_layout.addWidget(self.delay_input)
        main_layout.addLayout(delay_layout)

        # 하단: 키보드 입력 로그 창 (최대 3줄)
        self.log_output = QPlainTextEdit()
        self.log_output.setReadOnly(True)
        self.log_output.setMaximumBlockCount(3)  # 최대 3줄 유지
        main_layout.addWidget(self.log_output)

        self.setLayout(main_layout)

    def refresh_com_ports(self):
        self.com_combo.clear()
        ports = serial.tools.list_ports.comports()
        for port in ports:
            self.com_combo.addItem(port.device)

    def start_serial(self):
        selected_port = self.com_combo.currentText()
        if not selected_port:
            print("연결 가능한 COM 포트가 없습니다.")
            return
        self.serial_thread = SerialThread(selected_port)
        self.serial_thread.data_received.connect(self.handle_serial_data)
        self.serial_thread.start()
        print(f"시리얼 포트 연결됨: {selected_port}")
        self.log_output.appendPlainText(f"시리얼 포트 연결됨: {selected_port}")

    def stop_serial(self):
        if self.serial_thread is not None:
            self.serial_thread.stop()
            self.serial_thread = None
            print("시리얼 포트 연결이 해제되었습니다.")
            self.log_output.appendPlainText("시리얼 포트 연결이 해제되었습니다.")

    def handle_serial_data(self, data):
        # 갱신: 각 매핑 입력란의 값을 딕셔너리에 저장
        for key, key_input in self.inputs.items():
            self.mapping[key] = key_input.text()
        try:
            self.delay = int(self.delay_input.text())
        except ValueError:
            self.delay = 100
        QTimer.singleShot(self.delay, lambda: self.simulate_key_press(data))

    def simulate_key_press(self, data):
        """
        마이크로비트에서 전송된 이벤트(data)와 매핑된 키값에 따라,
        pyautogui를 통해 실제 키 입력을 시뮬레이션하고, 해당 결과를 로그 창에 출력합니다.
        """
        key_str = self.mapping.get(data, '')
        if not key_str:
            print(f"[{data}] 매핑된 키가 없습니다.")
            self.log_output.appendPlainText(f"[{data}] 매핑된 키가 없습니다.")
            return

        log_msg = f"[{data}] 이벤트 발생 → 매핑된 키: {key_str}"
        print(log_msg)
        self.log_output.appendPlainText(log_msg)

        time.sleep(0.1)  # 짧은 지연을 추가

        tokens = [t.strip() for t in key_str.split('+')]
        if len(tokens) > 1:
            tokens = [self._normalize_key_token(t) for t in tokens]
            pyautogui.hotkey(*tokens)
        else:
            single_key = self._normalize_key_token(tokens[0])
            pyautogui.press(single_key)

    def _normalize_key_token(self, token: str) -> str:
        token = token.lower()
        mapping_dict = {
            'ctrl': 'ctrl',
            'control': 'ctrl',
            'shift': 'shift',
            'alt': 'alt',
            'left': 'left',
            'right': 'right',
            'up': 'up',
            'down': 'down',
            'enter': 'enter',
            'return': 'enter',
            'win': 'win'
        }
        return mapping_dict.get(token, token)

    def closeEvent(self, event):
        self.stop_serial()
        event.accept()

###############################################################################
# 4. 실행부
###############################################################################
if __name__ == '__main__':
    app = QApplication(sys.argv)  
    window = JoypadWindow()
    window.show()
    sys.exit(app.exec_())
```

# Microbit Board Flashing Python 
```
from microbit import *
uart.init(baudrate=9600)

def send_event(event_str):
    """
    이벤트 발생 시, UART를 통해 event_str 문자열과 개행문자("\n")를 전송합니다.
    """
    uart.write(event_str + "\n")

# 이전 상태 변수들을 초기화합니다.
prev_gesture_event = None         # 최근 전송된 기울기 이벤트 ("위로 기울이기", "아래로 기울이기", 등)
prev_light_region = None           # 이전 조도 상태 ("조도 20 미만", "조도 21 이상")
prev_logo_touched = False          # 이전 로고 터치 상태

while True:
    # 1. 버튼 이벤트 (A, B 버튼은 .was_pressed()를 이용하여 한 번만 감지)
    if button_a.was_pressed():
        send_event("A버튼")
    if button_b.was_pressed():
        send_event("B버튼")
    
    # 2. 로고 터치 이벤트 (Micro:bit V2의 경우에 지원; V1에서는 에러 발생 시 무시)
    try:
        current_logo_touched = pin_logo.is_touched()
    except Exception:
        current_logo_touched = False
    if current_logo_touched and not prev_logo_touched:
        send_event("로고터치")
    prev_logo_touched = current_logo_touched

    # 3. 흔들기 이벤트
    if accelerometer.was_gesture("shake"):
        send_event("흔들기")

    # 4. 기울기 이벤트 (상태 변화에 따라 한 번만 전송)
    # accelerometer.current_gesture()는 "up", "down", "left", "right", "face up", "face down" 등을 반환합니다.
    # 여기서는 "up", "down", "left", "right" 만을 처리합니다.
    gesture = accelerometer.current_gesture()
    gesture_event = None
    if gesture == "up":
        gesture_event = "위로 기울이기"
    elif gesture == "down":
        gesture_event = "아래로 기울이기"
    elif gesture == "left":
        gesture_event = "왼쪽 기울이기"
    elif gesture == "right":
        gesture_event = "오른쪽 기울이기"
    # 상태가 변경되었을 때만 이벤트를 전송합니다.
    if gesture_event is not None and gesture_event != prev_gesture_event:
        send_event(gesture_event)
        prev_gesture_event = gesture_event
    elif gesture_event is None:
        # 원하는 기울기 이벤트가 아니면 이전 상태를 초기화합니다.
        prev_gesture_event = None

    # 5. 조도 센서 이벤트 (전환 시점에만 전송)
    # display.read_light_level()의 반환값이 0~255 사이의 정수값일 경우,
    # 20 미만이면 "조도 20 미만", 21 이상이면 "조도 21 이상"으로 구분합니다.
    light_level = display.read_light_level()
    if light_level < 20:
        light_region = "조도 20 미만"
    else:
        light_region = "조도 21 이상"
    if light_region != prev_light_region:
        send_event(light_region)
        prev_light_region = light_region

    sleep(100)

```
