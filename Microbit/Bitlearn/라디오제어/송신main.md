# 송신자 코드
```
from microbit import *
import radio

radio.on()
radio.config(group=7)

while True:
    if button_a.was_pressed():
        radio.send('1')  # 전진
    elif button_b.was_pressed():
        radio.send('2')  # 후진
    elif pin0.is_touched():
        radio.send('3')  # 좌회전
    elif pin1.is_touched():
        radio.send('4')  # 우회전
    elif accelerometer.was_gesture('shake'):
        radio.send('5')  # 정지
    sleep(100)
```
