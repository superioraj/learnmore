# instructable.com 프로젝트_
  - 자전거 헬멧 방향 표시기
```
from microbit import *
import random
random.seed(3433)   # enter your lucky number 
de = 100   # sets display delay time in ms
ff1 = 100    # sets firefly delay time 1 in ms
ff2 = 50    # sets firefly delay time 2 in ms
fn = 3     # sets number of firefly seed points
thresh_z = 80   # threshold value for backwards
thresh_x = 350  # threshold value for sidewards
# define images
image_l_1 = Image(   
    "00900:"
    "09000:"
    "97531:"
    "09000:"
    "00900")
              
image_l_2 = Image(   
    "09000:"
    "90000:"
    "75319:"
    "90000:"
    "09000")
                         
image_l_3 = Image(   
    "90000:"
    "00009:"
    "53197:"
    "00009:"
    "90000")
              
image_l_4 = Image(   
    "00009:"
    "00090:"
    "31975:"
    "00090:"
    "00009")
    
image_l_5 = Image(   
    "00090:"
    "00900:"
    "19753:"
    "00900:"
    "00090")
    
image_r_1 = Image(   
    "00900:"
    "00090:"
    "13579:"
    "00090:"
    "00900")
              
image_r_2 = Image(   
    "00090:"
    "00009:"
    "91357:"
    "00009:"
    "00090") 
    
image_r_3 = Image(   
    "00009:"
    "90000:"
    "79135:"
    "90000:"
    "00009")  
    
image_r_4 = Image(   
    "90000:"
    "09000:"
    "57913:"
    "09000:"
    "90000")   
    
image_r_5 = Image(   
    "09000:"
    "00900:"
    "35791:"
    "00900:"
    "09000")
    
image_z_1 = Image(   
    "90009:"
    "00000:"
    "00900:"
    "00000:"
    "90009")
image_z_2 = Image(   
    "09090:"
    "90009:"
    "00000:"
    "90009:"
    "09090")    
# start the program    
while True: 
    
    print((accelerometer.get_x(), accelerometer.get_y(), accelerometer.get_z()))
    # to be used with serial monitor or plotter for threshold value optimization; 
    # mute with '#' if not used
    
    if ((accelerometer.get_z() > thresh_z)      # head bent back, adjust if required
            or (button_a.is_pressed() and button_b.is_pressed())):   # for control purposes
           
        display.show(Image.DIAMOND_SMALL) 
        sleep(de) 
        display.show(Image.DIAMOND)
        sleep(de)
        display.show(image_z_2) 
        sleep(de)  
        display.show(image_z_1) 
        sleep(de)               
        display.clear()
        
    elif ((accelerometer.get_x() < (-1*thresh_x)) 
            # direction indicator left; bend head about 20 degrees left
            # adjust as needed
            or button_a.is_pressed()):    # for testing purposes 
                
        display.show(image_l_1)
        sleep(de)
        display.show(image_l_2)
        sleep(de)
        display.show(image_l_3)
        sleep(de) 
        display.show(image_l_4)
        sleep(de)
        display.show(image_l_5)
        sleep(de)
        display.clear()
        
    elif ((accelerometer.get_x() > thresh_x)  
            # direction indicator right; to activate bend head about 20 degrees right
            or button_b.is_pressed()): 
        
        display.show(image_r_1)
        sleep(de)
        display.show(image_r_2)
        sleep(de)
        display.show(image_r_3)
        sleep(de) 
        display.show(image_r_4)
        sleep(de)
        display.show(image_r_5)
        sleep(de)  
        display.clear()
             
    else:          # 'firefly' pattern generator
        for g in range(0, fn):           # seed a given number (fn) of pixels  
            x = random.randint(0, 4)     # picks a random position
            y = random.randint(0, 4)
            v = 9                        # seed brightness maximum
            # v = random.randint(0, 9)   # optional: randomized seed brightness
            display.set_pixel(x, y, v)
        # set firefly velocity
        sleep(ff1)                        # display for ff ms              
        
        # reduces intensity of all pixels by one step
        for j in range(0, 5):                # for every pixel of the LED array           
            for i in range(0, 5):
                b = display.get_pixel(i, j)  # get current intensity
                if (b > 0):
                    f = b - 1                # reduce brightness by one
                else:
                    f = 0                    # sets 0 as lowest allowed value
                display.set_pixel(i, j, f)
                
        sleep(ff2)
```
