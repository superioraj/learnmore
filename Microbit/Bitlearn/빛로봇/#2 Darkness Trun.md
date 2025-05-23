# 어두우면 회전하여 돌아가는 로봇

```
from microbit import *
from bitlearn import bitlearn
import random

robot = bitlearn(left_wheel_pin=pin0, right_wheel_pin=pin2)

def forward():
    robot.set_motors_speed(50, 50)

def backward():
    robot.set_motors_speed(-50, -50)

def right():
    robot.set_motors_speed(-50, 50)

def left():
    robot.set_motors_speed(50, -50)

def stop():
    robot.set_motors_speed(0, 0)

while True:
    light = display.read_light_level()
    print("조도 값:", light)  # 실제 값 확인용

    if light >= 5:  # 기존 10에서 낮춤
        forward()
    else:
        backward()
        sleep(2000)

        if random.randint(0, 1) == 1:
            right()
        else:
            left()
        sleep(5000)
```
