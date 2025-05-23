# 빛을 쫓아오는 로봇

```
from microbit import *
from bitlearn import bitlearn
import random

# bitlearn 인스턴스 생성 (왼쪽: pin0, 오른쪽: pin2)
robot = bitlearn(left_wheel_pin=pin0, right_wheel_pin=pin2)

def forward() :
    robot.set_motors_speed(50,50)

def backward() : 
    robot.set_motors_speed(-50,-50)

def right() : 
    robot.set_motors_speed(-50,50)

def left() :
    robot.set_motors_speed(50,-50)

def stop() :
    robot.set_motors_speed(0,0)

while True:
    if display.read_light_level()>=100: 
        forward()
    if display.read_light_level()<100: 
        stop()
```
