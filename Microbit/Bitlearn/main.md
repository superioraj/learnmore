# 전후좌우 함수 기본 코드
```
from microbit import *
from bitlearn import bitlearn

# bitlearn 인스턴스 생성 (왼쪽: pin0, 오른쪽: pin2)
robot = bitlearn(left_wheel_pin=pin0, right_wheel_pin=pin2)

def forward() :
    print('foward')
    display.show(Image.HEART)
    audio.play(Sound.HAPPY)
    robot.set_motors_speed(50,50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

def backward() : 
    print('backward')
    display.show(Image.SAD)
    audio.play(Sound.SAD)
    robot.set_motors_speed(-50,-50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

def right() : 
    print('right')
    display.show(Image.CONFUSED)
    audio.play(Sound.TWINKLE)
    robot.set_motors_speed(-50,50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

def left() :
    print('left')
    display.show(Image.ANGRY)
    audio.play(Sound.GIGGLE)
    robot.set_motors_speed(50,-50)
    sleep(5000)
    robot.set_motors_speed(0,0)
    sleep(500)

forward()
right()
left()
backward()
```
