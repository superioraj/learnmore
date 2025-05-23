# 송신자 코드
```
from microbit import *
import radio

radio.config(group=1, power=7) #gruop_num으로 변수 번호를 전송합니다.
radio.on()
while True:
    print("up |down | left | right 를 입력해주세요.")
    messege=input("전달하려는 메세지를 영어로 입력하세요:")
    if messege=="up":
        radio.send(messege)
        sleep(500)
    elif messege=="down":
        radio.send(messege)
        sleep(500)
    elif messege=="left":
        radio.send(messege)
        sleep(500)
    elif messege=="right":
        radio.send(messege)
        sleep(500)
    else:
        print("저장된 명령이 아닙니다.")
        sleep(500)
```
