# 미믹 게임
```
from microbit import *
import random

# Images
left  = Image("55000:55000:55000:55000:55000")
right = Image("00055:00055:00055:00055:00055")
wrong = Image("50005:05050:00500:05050:50005")
ok = Image("00000:00005:00050:50500:05000")
numbers = ["05550:00050:05550:00050:05550",
           "05550:00050:05550:05000:05550",
           "00500:05500:00500:00500:05550"]

# Get ready: 3...2...1

for number in numbers:
    display.show(Image(number))
    sleep(300)
    display.clear()
    sleep(500)

# Main loop

level = 1

while True:
    
    # Generate a random sequence
    sequence = []
    
    for i in range(level + 1):
        sequence.append(random.randint(0,1))
        
    for s in sequence:
        if s is 0:
            display.show(left)
        else:
            display.show(right)

        sleep(300)
        display.clear()
        sleep(100)

    # Test user input

    i = 0
    error = False

    while i < len(sequence):

        if button_a.is_pressed():
            if sequence[i] is 0:
                i += 1
            else:
                error = True

            # Wait for button release
            while button_a.is_pressed():
                1

        if button_b.is_pressed():
            if sequence[i] is 1:
                i += 1
            else:
                error = True

            # Wait for button release
            while button_b.is_pressed():
                1
            
        if error:
            for i in range(10):
                display.show(wrong)
                sleep(50)
                display.clear()
                sleep(50)

            display.scroll("Score: " + str(level-1))
            reset()

    display.show(ok)
    sleep(800)
    level += 1
```
