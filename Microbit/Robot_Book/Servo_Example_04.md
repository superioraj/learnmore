# for문으로 180도 서보 제어 예제
```
from microbit import *
from servo import *

robot = servo(left_servo_pin=pin0, right_servo_pin=pin2)

def Servo_Moving():
    for angle in range(0, 181, 10):  
        robot.set_motors_angle(angle, 0) 
        sleep(200) 

while True:
    Servo_Moving()
```
