# 키보드 라이브러리 활용하기

## 어떤 키를 눌렀는지 확인하기
* keyboard.read_key()로 어떤 키를 눌렀는지 확인합니다.
```python
from time import sleep
import keyboard

while True:
    print(keyboard.read_key())
    sleep(0.01)
```
* 'esc'키를 누르면 while문에서 빠져나오도록 합니다.
```python
from time import sleep
import keyboard

while True:
    print(keyboard.read_key())
    sleep(0.01)
    if keyboard.read_key() == 'esc':
        break
```
* keyboard.is_pressed("키이름")로 해당 키를 눌렀는지 확인합니다.
```python
from time import sleep
import keyboard

while True:
    if keyboard.is_pressed('esc'):
        break
    sleep(0.01)

print("esc키를 눌렀습니다.")
```

## 키보드로 드론 조종하기
* 필요한 모듈을 가져옵니다.
```python
from time import sleep
from e_drone.drone import *
from e_drone.protocol import *
import keyboard
```
* 드론 객체를 만듭니다.
```python
drone = Drone()
drone.open("자신의 com포트")
```
* 드론 조종하는 함수를 사용합니다.
```python
drone.sendTakeOff() #이륙
drone.sendLanding() #착륙
drone.sendControl(R, P, Y, T) #드론 조종하기
drone.sendControlWhile(R, P, Y, T, 시간) #시간을 입력해서 드론 조종하기
```
* 아래와 같이 키를 누르면 드론이 비행하도록 코딩을 합니다.
```python
while True:
    if keyboard.is_pressed("1"):
        print("TakeOff")
        sleep(0.1)
        drone.sendTakeOff()
        sleep(0.1)
        drone.sendControlWhile(0, 0, 0, 0, 4000) 
    elif keyboard.is_pressed("0"):
        print("Landing")
        drone.sendLanding()
        sleep(0.01)
        drone.sendLanding()
        sleep(0.01)
    elif keyboard.is_pressed("w"):
        print("Up")
        drone.sendControl(0, 0, 0, 50)
```
* wasd키로 쓰로틀과 요우값을 바꿉니다.
* 화살표키로 피치와 롤값을 바꿉니다. "up", "down", "left", "right"가 화살표키입니다.
```python
from time import sleep
from e_drone.drone import *
from e_drone.protocol import *
import keyboard

drone = Drone()
drone.open("com3")

while True:
    if keyboard.is_pressed("1"):
        print("TakeOff")
        sleep(0.1)
        drone.sendTakeOff()
        sleep(0.1)
        drone.sendControlWhile(0, 0, 0, 0, 4000) 
    elif keyboard.is_pressed("0"):
        print("Landing")
        drone.sendLanding()
        sleep(0.01)
        drone.sendLanding()
        sleep(0.01)
    elif keyboard.is_pressed("w"):
        print("Up")
        drone.sendControl(0, 0, 0, 50)
    elif keyboard.is_pressed("s"):
        print("Down")
        drone.sendControl(0, 0, 0, -50)
    elif keyboard.is_pressed("a"):
        print("CCW")
        drone.sendControl(0, 0, 50, 0)
    elif keyboard.is_pressed("d"):
        print("CW")
        drone.sendControl(0, 0, -50, 0)
    elif keyboard.is_pressed("up"):
        print("Forward")
        drone.sendControl(0, 50, 0, 0)
    elif keyboard.is_pressed("down"):
        print("Back")
        drone.sendControl(0, -50, 0, 0)
    elif keyboard.is_pressed("left"):
        print("Left")
        drone.sendControl(-50, 0, 0, 0)
    elif keyboard.is_pressed("right"):
        print("Right")
        drone.sendControl(50, 0, 0, 0)
```      
