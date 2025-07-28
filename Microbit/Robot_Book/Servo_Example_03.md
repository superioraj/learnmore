# 함수로 180도 서보 제어 예제
```
from microbit import *
from servo import *

robot = servo(left_servo_pin=pin0, right_servo_pin=pin2)

def Servo_Moving():
    robot.set_motors_angle(0, 0)
    sleep(1000)
    robot.set_motors_angle(90, 0)
    sleep(1000)
    robot.set_motors_angle(180, 0) 
    sleep(1000)

while True:
    Servo_Moving()
```
