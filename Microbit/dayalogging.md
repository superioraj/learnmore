```
from microbit import *
import log

log.set_labels('temperature', 'sound', 'light')

@run_every(ms=3000)
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
