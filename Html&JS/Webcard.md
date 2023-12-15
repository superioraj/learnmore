# 웹 카드 코드

* 결과링크: <https://www.learnmore.co.kr/education/codepython/html-css/html-cards>
* html을 기준으로 css & js 넣고 구현 sample
* background-image: url('xxx') 에서 xxx에 google drive 이미지 파일 url 넣기
* https://drive.google.com/uc?export=view&id=1H87CrjyjMZmxwBH41cMoO740PJXdtNYf 에서 **1H87CrjyjMZmxwBH41cMoO740PJXdtNYf** 부분이 고유 File_ID 입니다.
* 구글 드라이브의 이미지 파일 '공유' 부분에서 File_ID를 확인하고 https://drive.google.com/uc?export=view&id= 뒤에 넣고 코드 38번 줄 back-groundimage에 넣어 주세야 합니다.
  
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Card Effect</title>
    <style>
        .container {
            width: 220px; height: 310px; transition: all 0.1s;
            position: relative;
        }
        .overlay {
            position: absolute;
            width: 220px;
            height: 310px;
            background: linear-gradient(105deg,
            transparent 40%,
            rgba(255, 219, 112, 0.8) 45%,
            rgba(132, 50, 255 ,0.6) 50%,
            transparent 54%);
            filter: brightness(1.1) opacity(0.8);
            mix-blend-mode: color-dodge;
            background-size: 150% 150%;
            background-position: 100%;
            transition: all 0.1s;
        }
        .card {
            width: 220px; height: 310px;
            background-image: url('https://drive.google.com/uc?export=view&id=1H87CrjyjMZmxwBH41cMoO740PJXdtNYf');
            background-size: cover;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="overlay"></div>
        <div class="card"></div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            var container = document.querySelector('.container');
            var overlay = document.querySelector('.overlay');

            container.addEventListener('mousemove', function(e){
                var x = e.offsetX;
                var y = e.offsetY;
                var rotateY = -1/5 * x + 20;
                var rotateX = 4/30 * y - 20;

                overlay.style = `background-position: ${x/5 + y/5}%; filter: opacity(${x/200}) brightness(1.2)`;
                container.style = `transform: perspective(350px) rotateX(${rotateX}deg) rotateY(${rotateY}deg)`;
            });

            container.addEventListener('mouseout', function(){
                overlay.style = 'filter: opacity(0)';
                container.style = 'transform: perspective(350px) rotateY(0deg) rotateX(0deg)';
            });
        });
    </script>
</body>
</html>
```
