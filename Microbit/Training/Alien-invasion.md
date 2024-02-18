# 에얼리언 인베이젼
```
from microbit import *

# Images
numbers = ["05550:00050:05550:00050:05550",
           "05550:00050:05550:05000:05550",
           "00500:05500:00500:00500:05550"]

# Initialization
alive = True
player = [4.0,2.0]
enemyPath =  [[-2,i] for i in range(5)]
enemyPath += [[-1,i] for i in reversed(range(5))]
enemyPath += [[0,i] for i in range(5)]
enemyPath += [[1,i] for i in reversed(range(5))]
enemyPath += [[2,i] for i in range(5)]
enemyPath += [[3,i] for i in reversed(range(5))]
enemyPath += [[4,i] for i in range(5)]
moveThreshold = 0
enemies = [i for i in range(1,10)]
enemySpeed = 0.1
missiles = []
reloadWait = 0
score = 0
stage = 0

# Buttons initialization
releasedA = True
releasedB = True

# Get ready: 3...2...1
for number in numbers:
    display.show(Image(number))
    sleep(300)
    display.clear()
    sleep(500)

while alive:
    
    # Update player position
    jtilt = accelerometer.get_x()
    player[1] += + jtilt * 0.0005

    # Keep player inside the screen

    if player[1] < 0:
        player[1] = 0
    if player[1] > 4:
        player[1] = 4

    # Get button presses
    if button_a.is_pressed():
        if releasedA:
            1
            releasedA = False
    else:
        releasedA = True

    if button_b.is_pressed():
        if releasedB and reloadWait > 1.0: # Shoot
            missiles.append([int(player[0]), int(player[1])])
            releasedB = False
            reloadWait = 0
    else:
        releasedB = True

    reloadWait += 0.1
    
    # Update missile position
    deletions = []
    for i in range(len(missiles)):
         missiles[i][0] -= 0.25
         if missiles[i][0] < 0:
             deletions.append(i)

    # Delete missiles out of screen
    if len(deletions) > 0:
        deletions.sort(reverse=True)
    for i in deletions:
        missiles.pop(i)

    # Update enemies position
    moveThreshold += enemySpeed + stage * 0.03
    if int(moveThreshold) > 1.0:
        moveThreshold = 0
        for i in range(len(enemies)):
            enemies[i] += 1

    # Check if enemy has reached the ground
    for e in enemies:
        if e >= 30:
            alive = False
            break

    # Check missile collisions
    mdeletions = []
    edeletions = []
    for m in range(len(missiles)):
        for e in range(len(enemies)):
            if missiles[m] == enemyPath[enemies[e]]:
                mdeletions.append(m)
                edeletions.append(e)
                score += 1

    # Delete collided missiles and downed enemies
    if len(mdeletions) > 0:
        mdeletions.sort(reverse=True)
    for i in mdeletions:
        missiles.pop(i)

    if len(edeletions) > 0:
        edeletions.sort(reverse=True)
    for i in edeletions:
        enemies.pop(i)

    # Check if all enemies have been defeated
    if len(enemies) == 0:
        stage += 1
        enemies = [i for i in range(1,10)]
        
    # Render
    frame = [[0 for i in range(5)] for j in range(5)]

    frame[int(player[0])][int(player[1])] = 5

    for m in missiles:
        frame[int(m[0])][int(m[1])] = 5

    for e in enemies:
        if e > 9:
            frame[enemyPath[e][0]][enemyPath[e][1]] = 5

    for i in range(5):
        for j in range(5):
            display.set_pixel(j, i, frame[i][j])

# Game over
frame = [[0 for i in range(5)] for j in range(5)]
for e in enemies:
    if e > 9:
        frame[enemyPath[e][0]][enemyPath[e][1]] = 5

for i in range(10):
    for i in range(5):
        for j in range(5):
            display.set_pixel(j, i, frame[i][j])
    sleep(100)
    display.clear()
    sleep(100)

display.scroll("Score: " + str(score))
reset()
```
