# 마이크로비트 데이터 로깅 실습

` 마이크로 비트 파이썬 코드를 활용하여 데이터 로깅을 할 수 있습니다.
` 온도, 소리, 조도, 가속도를 측정하여 데이터를 축적할 수 있습니다.
` 아래 코드는 온도, 소리, 조도를 측정하고 이를 시리얼 창에 print() 합수로 츨력하는 코드입니다.
` 마이크로 비트 데이터 로깅은 실시간으로 되지는 않습니다.
` 코드를 플래싱했을 때 부터 측정이 되고, 마이크로 비트를 재연결하면 USB 폴더에 Mydata파일이 만들어 집니다.
` Mydata파일로 들어가서 측정된 값을 확인할 수 있으며, 그래프로 변환하여 분석할 수 있습니다.

```
from microbit import *
import log

log.set_labels('temperature', 'sound', 'light')

@run_every(ms=3000) #ms는 미리세건이며 1000이 1초 입니다. 여기 시간은 변경하여 측정 속도를 조철할 수 있습니다.
def log_data():
    log.add({
      'temperature': temperature(),
      'sound': microphone.sound_level(),
      'light': display.read_light_level()
    })


while True:

    print('temperature:{}, sound:{}, light:{}'.format(temperature(),microphone.sound_level(),display.read_light_level()  ))
    sleep(2000)
```
