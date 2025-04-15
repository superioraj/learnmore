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

# 버튼 누름 지속시간에 따른 점프 지속시간 (ms) 매핑 (4단계)
BUTTON_DURATION_MAPPING = {
    1: 50,    # Level 1: 매우 짧은 점프
    2: 100,   # Level 2: 짧은 점프
    3: 150,   # Level 3: 중간 점프
    4: 200    # Level 4: 긴 점프
}

# 틸트 레벨에 따른 타이머 간격 (ms) 매핑 (정밀 제어를 위해 수정됨)
TILT_INTERVAL_MAPPING = {
    1: 100,   # Level 1: 100ms – 초반 짧은 입력
    2: 80,    # Level 2: 80ms – 수정된 빠른 반응
    3: 60,    # Level 3: 60ms – 중간 짧은 간격
    4: 40     # Level 4: 40ms – 최대 단축된 간격
}

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
# 2. 시리얼 통신 스레드 (SerialThread)
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
# 3. 메인 GUI (JoypadWindow) – 버튼 및 틸트 정밀 제어 (키 눌림 상태 및 지속시간 제어)
###############################################################################
class JoypadWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Micro:bit Joypad Controller")
        # 매핑 정보 (사용자가 지정)
        self.mapping = {
            'A': '', 'B': '', 'Logo': '',
            'Up Tilt': '', 'Down Tilt': '',
            'Left Tilt': '', 'Right Tilt': '',
            'Shake': '', 'Light below 100': '', 'Light above 100': ''
        }
        self.serial_thread = None
        # 현재 눌린 상태를 추적 (실제 키가 눌린 상태)
        self.currently_pressed = {}
        # 타이머: 버튼 혹은 틸트 입력 지속 후 자동으로 keyUp 전송
        self.release_timers = {}
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
        # 시리얼 연결 및 해제 버튼 영역
        button_layout = QHBoxLayout()
        self.connect_button = QPushButton("시리얼 연결")
        self.connect_button.clicked.connect(self.start_serial)
        button_layout.addWidget(self.connect_button)
        self.disconnect_button = QPushButton("시리얼 끄기")
        self.disconnect_button.clicked.connect(self.stop_serial)
        button_layout.addWidget(self.disconnect_button)
        main_layout.addLayout(button_layout)
        # 매핑 및 타이머 간격 입력 영역 (기본값 300ms)
        self.inputs = {}
        self.repeat_interval_inputs = {}
        for key in self.mapping:
            row_layout = QHBoxLayout()
            label = QLabel(f"{key} 매핑:")
            label.setAlignment(Qt.AlignRight)
            key_input = BlinkingKeyMappingLineEdit()
            self.inputs[key] = key_input
            row_layout.addWidget(label)
            row_layout.addWidget(key_input)
            interval_label = QLabel("타이머 간격 (ms):")
            interval_input = QLineEdit("300")
            interval_input.setFixedWidth(60)
            row_layout.addWidget(interval_label)
            row_layout.addWidget(interval_input)
            self.repeat_interval_inputs[key] = interval_input
            main_layout.addLayout(row_layout)
        # 키보드 입력 로그 창
        self.log_output = QPlainTextEdit()
        self.log_output.setReadOnly(True)
        self.log_output.setMaximumBlockCount(3)
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
        msg = f"시리얼 포트 연결됨: {selected_port}"
        print(msg)
        self.log_output.appendPlainText(msg)
    def stop_serial(self):
        if self.serial_thread is not None:
            self.serial_thread.stop()
            self.serial_thread = None
            msg = "시리얼 포트 연결이 해제되었습니다."
            print(msg)
            self.log_output.appendPlainText(msg)
    def handle_serial_data(self, data):
        # 매핑 정보 업데이트
        for key, key_input in self.inputs.items():
            self.mapping[key] = key_input.text()
        # 수신된 메시지 형식에 따라 이벤트 처리 수행
        if "_release:" in data:
            key, level_str = data.split("_release:")
            try:
                level = int(level_str)
            except ValueError:
                level = 1
            self.handle_button_release(key.strip(), level)
        elif any(data.startswith(direction) for direction in ["Up Tilt", "Down Tilt", "Left Tilt", "Right Tilt"]):
            if ":" in data:
                direction, level_str = data.split(":", 1)
                try:
                    level = int(level_str)
                except ValueError:
                    level = 1
            else:
                direction = data
                level = 1
            self.handle_tilt_event(direction.strip(), level)
        elif data in {"Shake", "Light below 100", "Light above 100"}:
            self.simulate_key_press_once(data)
        else:
            msg = f"[{data}] 매핑된 키가 없습니다."
            print(msg)
            self.log_output.appendPlainText(msg)
    def handle_button_release(self, key, level):
        """
        버튼 릴리즈 이벤트 처리: 버튼 레벨에 따라 지정된 지속시간만큼 keyDown 후 keyUp 수행함.
        """
        duration = BUTTON_DURATION_MAPPING.get(level, 50)
        self.simulate_key_event(key, 'down')
        self.log_output.appendPlainText(f"[{key}] 단발성 점프 (레벨 {level}) 실행 (지속시간 {duration}ms)")
        QTimer.singleShot(duration, lambda key=key: self.simulate_key_event(key, 'up'))
    def handle_tilt_event(self, direction, level):
        """
        틸트 이벤트 처리: 수정된 단축된 타이머 간격을 기반으로 정밀하게 키 입력을 제어하며,
        반복 입력 시에도 이전 타이머를 재설정하여 과도한 입력 유지 현상을 방지함.
        """
        interval = TILT_INTERVAL_MAPPING.get(level, 100)
        self.simulate_key_event(direction, 'down')
        self.schedule_key_release(direction, interval)
    def schedule_key_release(self, key, interval):
        """
        지정된 키에 대해 기존 타이머가 존재할 경우 이를 중단한 후 새 타이머를 설정하여,
        일정 시간 후 자동으로 keyUp 이벤트를 전송함.
        """
        if key in self.release_timers:
            self.release_timers[key].stop()
        release_timer = QTimer(self)
        release_timer.setSingleShot(True)
        release_timer.setInterval(interval)
        release_timer.timeout.connect(lambda key=key: self.simulate_key_event(key, 'up'))
        self.release_timers[key] = release_timer
        release_timer.start()
    def simulate_key_event(self, key, event_type):
        """
        매핑된 키 문자열을 분석하여 pyautogui 모듈을 통해 실제 키 이벤트(누름/해제)를 전송함.
        """
        key_str = self.mapping.get(key, '')
        if not key_str:
            msg = f"[{key}] 매핑된 키가 없습니다."
            print(msg)
            self.log_output.appendPlainText(msg)
            return
        tokens = [t.strip() for t in key_str.split('+')]
        tokens = [self._normalize_key_token(t) for t in tokens]
        msg = f"[{key}] {event_type} 이벤트 → 매핑된 키: {key_str}"
        print(msg)
        self.log_output.appendPlainText(msg)
        if event_type == 'down':
            if not self.currently_pressed.get(key, False):
                for token in tokens:
                    pyautogui.keyDown(token)
                self.currently_pressed[key] = True
        elif event_type == 'up':
            if self.currently_pressed.get(key, False):
                for token in reversed(tokens):
                    pyautogui.keyUp(token)
                self.currently_pressed[key] = False
    def simulate_key_press_once(self, key):
        """
        단발성 이벤트 처리: 한 번의 키 입력(또는 핫키 조합)을 실행함.
        """
        key_str = self.mapping.get(key, '')
        if not key_str:
            msg = f"[{key}] 매핑된 키가 없습니다."
            print(msg)
            self.log_output.appendPlainText(msg)
            return
        msg = f"[{key}] 단발성 이벤트 → 매핑된 키: {key_str}"
        print(msg)
        self.log_output.appendPlainText(msg)
        tokens = [t.strip() for t in key_str.split('+')]
        if len(tokens) > 1:
            tokens = [self._normalize_key_token(t) for t in tokens]
            pyautogui.hotkey(*tokens)
        else:
            pyautogui.press(self._normalize_key_token(tokens[0]))
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
# 4. 프로그램 실행부
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
import utime

# -----------------------------------------------------------
# 사용자 설정값 (필요에 따라 조정)
# -----------------------------------------------------------
THRESHOLD_TILT = 300      # 기울기 판별 임계값 (±300)
BUTTON_LEVEL_THRESHOLDS = [200, 400, 600]  # 단/중/장 누름 구분 (ms)
LOOP_DELAY = 20           # 메인 루프 딜레이 (ms)
# -----------------------------------------------------------

# 함수: 버튼 누름 지속 시간에 따른 레벨 결정
def get_button_level(duration_ms):
    if duration_ms < BUTTON_LEVEL_THRESHOLDS[0]:
        return 1
    elif duration_ms < BUTTON_LEVEL_THRESHOLDS[1]:
        return 2
    elif duration_ms < BUTTON_LEVEL_THRESHOLDS[2]:
        return 3
    else:
        return 4

# 함수: 기울기 초과량에 따른 틸트 레벨 결정 (예: 0~100: level1, 101~200: level2, 201~300: level3, ≥301: level4)
def get_tilt_level(intensity):
    if intensity < 100:
        return 1
    elif intensity < 200:
        return 2
    elif intensity < 300:
        return 3
    else:
        return 4

# 시리얼 초기화 (PC와 9600 baud)
uart.init(baudrate=9600)

# 버튼 상태 기록용 변수
a_press_start = 0
b_press_start = 0
logo_press_start = 0

a_was_pressed = False
b_was_pressed = False
logo_was_pressed = False

while True:
    now = utime.ticks_ms()
    
    # --- 버튼 A 처리 ---
    if button_a.is_pressed():
        if not a_was_pressed:
            a_press_start = now
        a_was_pressed = True
    else:
        if a_was_pressed:
            duration = utime.ticks_diff(now, a_press_start)
            level = get_button_level(duration)
            uart.write("A_release:" + str(level) + "\n")
        a_was_pressed = False
        
    # --- 버튼 B 처리 ---
    if button_b.is_pressed():
        if not b_was_pressed:
            b_press_start = now
        b_was_pressed = True
    else:
        if b_was_pressed:
            duration = utime.ticks_diff(now, b_press_start)
            level = get_button_level(duration)
            uart.write("B_release:" + str(level) + "\n")
        b_was_pressed = False
        
    # --- Logo 처리 (예: pin_logo 또는 pin0 터치 사용; 여기서는 pin_logo 사용) ---
    if pin_logo.is_touched():
        if not logo_was_pressed:
            logo_press_start = now
        logo_was_pressed = True
    else:
        if logo_was_pressed:
            duration = utime.ticks_diff(now, logo_press_start)
            level = get_button_level(duration)
            uart.write("Logo_release:" + str(level) + "\n")
        logo_was_pressed = False
        
    # --- 틸트 처리 ---
    x = accelerometer.get_x()
    y = accelerometer.get_y()
    
    # Up Tilt 처리
    if y > THRESHOLD_TILT:
        intensity = y - THRESHOLD_TILT
        level = get_tilt_level(intensity)
        uart.write("Down Tilt:" + str(level) + "\n")
    # Down Tilt 처리
    if y < -THRESHOLD_TILT:
        intensity = abs(y) - THRESHOLD_TILT
        level = get_tilt_level(intensity)
        uart.write("Up Tilt:" + str(level) + "\n")
    # Right Tilt 처리
    if x > THRESHOLD_TILT:
        intensity = x - THRESHOLD_TILT
        level = get_tilt_level(intensity)
        uart.write("Right Tilt:" + str(level) + "\n")
    # Left Tilt 처리
    if x < -THRESHOLD_TILT:
        intensity = abs(x) - THRESHOLD_TILT
        level = get_tilt_level(intensity)
        uart.write("Left Tilt:" + str(level) + "\n")
        
    # --- Shake 처리 (1회 전송) ---
    if accelerometer.was_gesture("shake"):
        uart.write("Shake\n")
        
    # --- 조도(Light) 처리 (필요시 추가 구현) ---
    # 예를 들어 display.read_light_level()을 이용할 수 있음.
    
    utime.sleep_ms(LOOP_DELAY)

```
