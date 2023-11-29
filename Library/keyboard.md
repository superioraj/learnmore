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

## 키보드로 장애물 피하기 게임 자동 조정하기
* 필요한 모듈을 가져옵니다.
```python
from time import sleep
import keyboard
```
* 코드 샘플

```      
```
