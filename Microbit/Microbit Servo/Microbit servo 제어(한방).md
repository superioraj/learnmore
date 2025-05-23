# Microbit servo 제어 python 코드
```
# main.py
from microbit import *

# 1) 원본 pin 객체 보관
_orig_pin0 = pin0
_orig_pin1 = pin1
_orig_pin2 = pin2

# 2) PWM 주기 설정 (20ms, 50Hz)
for p in (_orig_pin0, _orig_pin1, _orig_pin2):
    p.set_analog_period(20)

# 3) 서보 각도 → 아날로그 값 변환 상수 계산
ANALOG_MIN = int(1023 * 0.5  / 20)   # 0° (0.5ms)
ANALOG_MID = int(1023 * 1.5  / 20)   # 90° (1.5ms, 정지)
ANALOG_MAX = int(1023 * 2.5  / 20)   # 180° (2.5ms)

# 4) 각도를 값으로 변환하는 헬퍼 함수
def _angle_to_value(angle: int) -> int:
    """0~180°를 0.5ms~2.5ms 아날로그 값(0~1023)으로 선형 변환"""
    if angle < 0:   angle = 0
    if angle > 180: angle = 180
    return int(ANALOG_MIN + (angle / 180) * (ANALOG_MAX - ANALOG_MIN))

# 5) ServoPin 래퍼 클래스 정의
class ServoPin:
    def __init__(self, pin):
        self._pin = pin
    def servo(self, angle: int):
        """주어진 각도로 서보를 회전"""
        self._pin.write_analog(_angle_to_value(angle))

# 6) pin0, pin1, pin2 변수에 래퍼 인스턴스 할당
pin0 = ServoPin(_orig_pin0)
pin1 = ServoPin(_orig_pin1)
pin2 = ServoPin(_orig_pin2)

############################ 코드 설계 ############################

# 7) 사용 예시
while True:
    # A 버튼: pin0 0°면 시계 방향 
    if button_a.was_pressed():
        pin0.servo(0); 
        sleep(500)

    # A 버튼: pin0 180°면 반시계 방향 
    if button_b.was_pressed():
        pin0.servo(180); 
        sleep(500)

    if button_a.was_pressed() and button_b.was_pressed():
        pin0.servo(90); 
        sleep(500)
```
