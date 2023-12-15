# 웹 카드 코드
* url에 이미지 넣기
* html을 기준으로 css & js 넣고 구형(test_01)
  
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Card Effect</title>
    <style>
        .container {
            width: 220px; height: 310px; transition : all 0.1s;
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
            background-image: https://drive.google.com/file/d/1H87CrjyjMZmxwBH41cMoO740PJXdtNYf/view?usp=sharing; /* 여기에 적절한 이미지 URL을 넣으세요 */
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

  }
  .card {
    width: 220px; height: 310px;
    background-image: https://drive.google.com/file/d/1H87CrjyjMZmxwBH41cMoO740PJXdtNYf/view?usp=sharing;
    background-size: cover;
  }
</style>
```
