# 수신자 코드
```
from microbit import *
import radio
from bitlearn import bitlearn

robot = bitlearn(left_wheel_pin=pin0, right_wheel_pin=pin2)

radio.config(group=1, power=7) #수신자마다 그룹 번호를 1-6으로 지정
radio.on()

def forward() :
    robot.set_motors_speed(50,50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

def backward() : 
    robot.set_motors_speed(-50,-50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

def right() : 
    robot.set_motors_speed(-50,50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

def left() :
    robot.set_motors_speed(50,-50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)


while True:
    messege = radio.receive()

    if messege=='up':
        forward()
    elif messege=='down':
        backward()
    elif messege=='left':
        left()
    elif messege=='right':
        right()    
```
