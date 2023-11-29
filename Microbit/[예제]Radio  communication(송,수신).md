# 라디오 통신 활용하기

* 마이크로 비트는 라디오 통신을 할 수 있습니다.
* 그룹 번호를 설정하여 통신 환경을 구축할 수 있습니다.
* 각 그룹번호에 따라 수신되는 값이 달라집니다.
* 예를 들어 송신자가 1번 그룹, 수신자 10명이 1번 그룹으로 설정하고
* 송신자가 메세지를 보내면, 수신자 10명의 마이크로 비트에 메세지가 출력됩니다.
* 응용하면, 수신자 그룹을 1-6으로 구분하고, 송신자가 그룹 번호를 선택하여 때에 따라 원하는 정보도 전달 할 수 있습니다.
* 아래는 라디오 통신을 응용한 코드 기본 예제입니다.

## 코드예제(송신자)
```
from microbit import *
import radio

radio.config(group=23, power=7)
radio.on()
while True:
    if accelerometer.was_gesture('shake'):
        radio.send("1")
```

## 코드예제(수신자)
```
from microbit import *
import radio

radio.config(group=23, power=7)
radio.on()
while True:
    message = radio.receive()
    if message=="1":
        display.show(Image.CONFUSED)
        sleep(3000)
        display.clear()
```


---


*마이크로비트로 교사-학생그룹 라디오 통신 하기 
## 코드예제(송신자)
```
from microbit import *
import radio

group_num=0
radio.config(group=group_num, power=7) #gruop_num으로 변수 번호를 전송합니다.
radio.on()
while True:
    group_num=int(input("전송하려는 그룹 1-6을 선택하세요:"))
    messege=input("전달하려는 메세지를 영어로 입력하세요:")
    if group_num=1:
        radio.send("messege")
    elif group_num=2:
        radio.send("messege")
    elif group_num=3:
        radio.send("messege")
    elif group_num=4:
        radio.send("messege")
    elif group_num=5:
        radio.send("messege")
    else group_num=6:
        radio.send("messege")
```


## 코드예제(수신자)
```
from microbit import *
import radio

radio.config(group=1, power=7) #수신자마다 그룹 번호를 1-6으로 지정
radio.on()
while True:
    messege = radio.receive()
    display.scroll(messege)
    sleep(3000)
    display.clear()
```
