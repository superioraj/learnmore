# 서보모터 1개 제어하기 예제
```
from microbit import *
from bitlearn import bitlearn

# bitlearn 인스턴스 생성 (왼쪽: pin0, 오른쪽: pin2)
robot = bitlearn(left_wheel_pin=pin0, right_wheel_pin=pin2)

robot.set_motors_speed(50, 0)
```
