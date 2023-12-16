# html, css 영어 단어 워드 테이블 만들기
* Google site 도구에서 활용 가능한 영어단어 표 입니다. 한글 사전 등 다양하게 응용 가능합니다.
* 샘플코드
```
  <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>감정 관련 60가지 단어</title>
    <style>
        body {
            overflow: hidden; /* 스크롤 창을 숨김 */
        }
        
        table {
            border-collapse: collapse;
            width: 100%;
            margin-bottom: 20px;
            color: white; /* 표 안의 글자 색을 흰색으로 설정 */
        }
        
        th, td {
            border: 1px solid #dddddd;
            text-align: left;
            padding: 8px;
            position: relative; /* 상대적 위치 설정 */
        }
        
        th {
            background-color: #f2f2f2;
            color: lightgray; /* 표의 맨 위 상단 첫 번째 행의 글씨 색을 검정으로 설정 */
            font-size: 10pt; /* 글꼴 크기를 15pt로 설정 */
        }
        
        /* 각 단어에 대한 설명을 감춤 */
        .meaning {
            display: none;
            position: absolute;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 5px;
            border-radius: 5px;
            z-index: 1;
        }
        
        /* 마우스를 단어 위에 올렸을 때 설명을 보이도록 함 */
        td:hover .meaning {
            display: block;
        }

        /* 표를 넘칠 경우 표를 잘라내도록 설정 */
        table {
            table-layout: fixed;
        }
        
        tbody {
            max-height: 400px; /* 표의 최대 높이를 설정 */
            overflow-y: auto; /* 수직 스크롤바를 표시 */
        }
    </style>
</head>
<body>
    <h1 style="color: #97999B;">감정 관련 60가지 단어</h1>
    <table>
        <tr>
            <th>영어 단어</th>
            <th>단어 해석</th>
        </tr>
        <tr>
            <td>Joy<span class="meaning">기쁨</span></td>
            <td>Sadness<span class="meaning">슬픔</span></td>
        </tr>
        <tr>
            <td>Anger<span class="meaning">분노</span></td>
            <td>Anxiety<span class="meaning">불안</span></td>
        </tr>
        <tr>
            <td>Love<span class="meaning">사랑</span></td>
            <td>Fear<span class="meaning">두려움</span></td>
        </tr>
        <tr>
            <td>Excitement<span class="meaning">흥분</span></td>
            <td>Trust<span class="meaning">신뢰</span></td>
        </tr>
        <tr>
            <td>Disgust<span class="meaning">혐오</span></td>
            <td>Gratitude<span class="meaning">감사</span></td>
        </tr>
        <tr>
            <td>Hope<span class="meaning">희망</span></td>
            <td>Confusion<span class="meaning">혼란</span></td>
        </tr>
        <tr>
            <td>Satisfaction<span class="meaning">만족</span></td>
            <td>Confidence<span class="meaning">자신감</span></td>
        </tr>
        <tr>
            <td>Regret<span class="meaning">후회</span></td>
            <td>Embarrassment<span class="meaning">부끄러움</span></td>
        </tr>
        <tr>
            <td>Frustration<span class="meaning">짜증</span></td>
            <td>Comfort<span class="meaning">편안함</span></td>
        </tr>
        <tr>
            <td>Bewilderment<span class="meaning">의아함</span></td>
            <td>Lust<span class="meaning">권태</span></td>
        </tr>
        <tr>
            <td>Happiness<span class="meaning">행복</span></td>
            <td>Surprise<span class="meaning">놀라움</span></td>
        </tr>
        <tr>
            <td>Disappointment<span class="meaning">실망</span></td>
            <td>Shame<span class="meaning">부끄러움</span></td>
        </tr>
        <tr>
            <td>Envy<span class="meaning">질투</span></td>
            <td>Hopelessness<span class="meaning">절망</span></td>
        </tr>
        <tr>
            <td>Calmness<span class="meaning">평온</span></td>
            <td>Loneliness<span class="meaning">외로움</span></td>
        </tr>
        <tr>
            <td>Sympathy<span class="meaning">동정</span></td>
            <td>Discontent<span class="meaning">불만</span></td>
        </tr>
        <!-- 추가 20개 감정 단어 -->
        <tr>
            <td>Hope<span class="meaning">희망</span></td>
            <td>Disappointment<span class="meaning">실망</span></td>
        </tr>
        <tr>
            <td>Amusement<span class="meaning">유쾌함</span></td>
            <td>Jealousy<span class="meaning">질투</span></td>
        </tr>
        <tr>
            <td>Loneliness<span class="meaning">외로움</span></td>
            <td>Amazement<span class="meaning">놀람</span></td>
        </tr>
        <tr>
            <td>Shame<span class="meaning">부끄러움</span></td>
            <td>Contentment<span class="meaning">만족</span></td>
        </tr>
        <tr>
            <td>Guilt<span class="meaning">죄책감</span></td>
            <td>Resentment<span class="meaning">원망</span></td>
        </tr>
        <tr>
            <td>Relief<span class="meaning">안도</span></td>
            <td>Surprise<span class="meaning">놀라움</span></td>
        </tr>
        <tr>
            <td>Jealousy<span class="meaning">질투</span></td>
            <td>Enthusiasm<span class="meaning">열정</span></td>
        </tr>
        <tr>
            <td>Loneliness<span class="meaning">외로움</span></td>
            <td>Amusement<span class="meaning">유쾌함</span></td>
        </tr>
        <tr>
            <td>Shame<span class="meaning">부끄러움</span></td>
            <td>Contentment<span class="meaning">만족</span></td>
        </tr>
        <tr>
            <td>Guilt<span class="meaning">죄책감</span></td>
            <td>Resentment<span class="meaning">원망</span></td>
        </tr>
        <tr>
            <td>Relief<span class="meaning">안도</span></td>
            <td>Surprise<span class="meaning">놀라움</span></td>
        </tr>
    </table>
</body>
</html>
```
