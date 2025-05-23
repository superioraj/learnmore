# 마이크로 비트 파이썬 에디터 시리얼 통신 코드

```
from microbit import *

uart.init(baudrate=115200)  # 기본 블루투스 보드레이트 설정 대체 가능

# 무한 반복 루프
while True:
    if uart.any():
        received_bytes = uart.read()
        
        if received_bytes:
            received_string = received_bytes.decode('utf-8').strip()

            if received_string == "0":  #이미지 분류 클래스 문자 조건 판단
                display.show(Image.HEART) #해당 조건 문자 판별 후 실행되는 코드 입력
            elif received_string == "1":
                display.show(Image.HAPPY)
            elif received_string == "2":
                display.show(Image.SAD)
            else:
                display.clear()
```
