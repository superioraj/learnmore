# 180도 서보모터 1개 제어하기 예제
```
from microbit import *
from servo import *

# servo 인스턴스 생성 (왼쪽: pin0, 오른쪽: pin2)
robot = servo(left_servo_pin=pin0, right_servo_pin=pin2)

while True :
    robot.set_motors_angle(0, 0)
    sleep(1000)
    robot.set_motors_angle(90, 0)
    sleep(1000)
    robot.set_motors_angle(180, 0)
    sleep(1000)
```
