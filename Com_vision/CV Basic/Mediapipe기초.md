# MediaPipe로 손가락을 인식하기
## MediaPipe 알아보기
* MediaPipe를 사용해서 손가락을 인식할 수 있습니다.
* MediaPipe의 Hands 모델은 손가락과 손의 위치를 알려줍니다.
* https://google.github.io/mediapipe/solutions/hands.html 에서 사용법을 확인합니다.
* 손가락을 점으로 나타냅니다.
![image](https://user-images.githubusercontent.com/76088532/146646503-670e339c-87ca-4741-adb9-9391607c0fe3.png)
* pip로 MediaPipe를 설치합니다.
```
pip install mediapipe
```

## MediaPipe로 손가락 좌표 확인하기
* 손가락 마디마다 지정된 번호가 있습니다. ```landmark[번호]```로 좌표를 확인할 수 있습니다. 
* ```mp.solutions.drawing_utils```와 ```mp.solutions.hands```는 점과 연두색으로 손가락의 마디를 표현해주는 유틸리티입니다.
* ```mp_hands.Hands```로 손가락을 인식하는 모듈을 초기화합니다.
* ```max_num_hands```로 인식하려는 손의 수를 정합니다.
* ```min_detection_confidence```와 ```min_tracking_confidence```는 옵션입니다. 둘 다 0.5로 설정합니다.
* 웹캠이 열려있으면(```capture.isOpened()```) 영상을 읽어서 손을 인식합니다. 
* ```cv.flip(frame, 1)```로 화면을 뒤집어서 거울모드로 만듭니다.
* OpenCV는 BGR 색깔 시스템을 사용합니다. MediaPipe는 RGB 색깔 시스템을 사용합니다. ```cv.COLOR_BGR2RGB```로 BGR을 RGB로 만듭니다.
* hands.process()로 전처리와 모델 추론을 함께 합니다.
* 처리가 되면 ```results.multi_hand_landmarks```가 ```True```가 됩니다.
* ```results.multi_hand_landmarks```에는 ```mp_hands.Hands```로 정한 손의 수만큼 리스트로 값이 저장되어 있습니다.
* ```print(type(results.multi_hand_landmarks))```으로 확인합니다.
* for문을 사용해서 손가락에 관한 정보를 확인합니다.
* ```finger1 = hand_landmarks.landmark[8]```와 같이 변수에 할당(저장)해서 사용합니다.
* ```mp_drawing.draw_landmarks()```로 뼈마디를 그립니다.
* ```cv.imshow()```로 이미지를 보여줍니다.
* ```esc(27)```키를 누르면 while문에서 빠져나오고 창을 닫습니다. ```if cv.waitKey(1) == 27:```
* ```max_num_hands``` 값을 바꿔서 확인해봅니다.
```python
import cv2 as cv
import mediapipe as mp

mp_drawing = mp.solutions.drawing_utils
mp_hands = mp.solutions.hands

capture = cv.VideoCapture(0)

with mp_hands.Hands(
    max_num_hands=1,
    min_detection_confidence=0.5,
    min_tracking_confidence=0.5
    ) as hands:

    while capture.isOpened():
        ret, frame = capture.read()
        if not ret:
            continue
        frame = cv.cvtColor(cv.flip(frame, 1), cv.COLOR_BGR2RGB)
        results = hands.process(frame)

        frame = cv.cvtColor(frame, cv.COLOR_RGB2BGR)
        if results.multi_hand_landmarks:
            for hand_landmarks in results.multi_hand_landmarks:
                finger1 = hand_landmarks.landmark[8]             
                mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)
        cv.imshow("Finger", frame)
        if cv.waitKey(1) == 27:
            break
        
capture.release()
cv.destroyAllWindows()
```
  
* ```hand_landmarks.landmark[번호].x와 hand_landmarks.landmark[번호].y```로 좌푯값을 확인할 수 있습니다.
* 손가락을 움직여서 값이 어떻게 변하는지 확인합니다. 
```python
if results.multi_hand_landmarks:
    for hand_landmarks in results.multi_hand_landmarks:
        finger1 = hand_landmarks.landmark[8]
        print(finger1.x, finger1.y)
        mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)
```

## 좌푯값을 나타내기
* 좌푯값을 ```cv.putText()```로 나타낼 수 있습니다.
* ```font = cv.FONT_HERSHEY_DUPLEX```로 폰트를 정합니다.
* ```float("{0:0.3f}".format(좌푯값))```으로 나타내고 싶은 소수점 자리수를 정할 수 있습니다.
```python
if results.multi_hand_landmarks:
    for hand_landmarks in results.multi_hand_landmarks:
        finger1 = hand_landmarks.landmark[8]
        finger1_x = float("{0:0.3f}".format(finger1.x))
        finger1_y = float("{0:0.3f}".format(finger1.y))
        coordinate = "x:{0}, y:{1}".format(finger1_x, finger1_y)
        cv.putText(frame, coordinate, (30, 30), font, 1, (255,0,0))
        mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)

```
* ```cv.putText(frame, coordinate, org=(30, 30), fontFace=font, fontScale=1, color=(255,0,0), thickness=2)```와 같이 매개변수에 값을 대입해서 코딩할 수도 있습니다.
* 엄지와 검지 손가락의 간격을 계산해서 값을 나타낼 수 있습니다.
* ```abs()```로 절대값으로 나타냅니다.
```python
if results.multi_hand_landmarks:
    for hand_landmarks in results.multi_hand_landmarks:
        finger1 = hand_landmarks.landmark[4]
        finger2 = hand_landmarks.landmark[8]
        diff = abs(finger1.x - finger2.x)
        diff = int(diff*100)                
        diff_text = "difference:{0}".format(diff)
        cv.putText(frame, diff_text, org=(30, 30), fontFace=font, fontScale=1, color=(255,0,0), thickness=2)
        mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)
```   
