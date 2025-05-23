# 수신자 코드
```
from microbit import *
import radio
import audio
from bitlearn import bitlearn

robot = bitlearn(left_wheel_pin=pin0, right_wheel_pin=pin2)

radio.on()
radio.config(group=7)

command = None

while True:
    incoming = radio.receive()
    if incoming:
        command = incoming
        display.scroll(command)

    if command == '1' and accelerometer.was_gesture('shake'):
        display.show(Image.HEART)
        audio.play(Sound.HAPPY)
        robot.set_motors_speed(50, 50)
        sleep(1000)
        robot.set_motors_speed(0, 0)

    elif command == '2' and pin0.is_touched():
        display.show(Image.SAD)
        audio.play(Sound.SAD)
        robot.set_motors_speed(-50, -50)
        sleep(1000)
        robot.set_motors_speed(0, 0)

    elif command == '3' and button_a.was_pressed():
        display.show(Image.ANGRY)
        audio.play(Sound.GIGGLE)
        robot.set_motors_speed(50, -50)
        sleep(1000)
        robot.set_motors_speed(0, 0)

    elif command == '4' and button_b.was_pressed():
        display.show(Image.CONFUSED)
        audio.play(Sound.TWINKLE)
        robot.set_motors_speed(-50, 50)
        sleep(1000)
        robot.set_motors_speed(0, 0)

    elif command == '5' and button_a.is_pressed() and button_b.is_pressed():
        display.show(Image.NO)
        robot.set_motors_speed(0, 0)
        sleep(1000)

    sleep(100)
```
