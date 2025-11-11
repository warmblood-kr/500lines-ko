title: 광학 문자 인식 (OCR)
author: Marina Samuel

## 소개

만약 여러분의 컴퓨터가 설거지도 하고, 빨래도 하고, 저녁도 만들어 주고,
집 청소까지 해준다면 어떨까요? 대부분의 사람들이 그런 도움을 받는다면 기뻐할 거라고
확신합니다! 그런데 컴퓨터가 인간과 똑같은 방식으로 이런 작업들을 수행하려면
무엇이 필요할까요?

저명한 컴퓨터 과학자 앨런 튜링은 기계가 인간과 구별할 수 없는 지능을
가질 수 있는지를 판별하는 방법으로 튜링 테스트를 제안했습니다. 이 테스트는
인간이 숨겨진 두 개체(하나는 인간, 다른 하나는 기계)에게 질문을 하고
어느 쪽이 어느 쪽인지를 구별하려고 시도하는 것입니다. 심문자가 기계를
식별할 수 없다면, 그 기계는 인간 수준의 지능을 가진 것으로 간주됩니다.

튜링 테스트가 지능에 대한 유효한 평가인지, 그리고 우리가 그런 지능적인
기계를 만들 수 있는지에 대해서는 많은 논란이 있지만, 어느 정도의 지능을
가진 기계들이 이미 존재한다는 것은 의심할 여지가 없습니다. 현재 로봇이
사무실을 돌아다니며 작은 업무들을 수행하거나 알츠하이머 환자들을 돕는
소프트웨어가 있습니다. 인공지능(A.I.)의 더 일반적인 예로는 구글이
키워드로 검색할 때 여러분이 찾는 것을 추정하는 방식이나, 페이스북이
뉴스 피드에 무엇을 넣을지 결정하는 방식이 있습니다.

A.I.의 잘 알려진 응용 분야 중 하나가 광학 문자 인식(OCR)입니다.
OCR 시스템은 손으로 쓴 문자의 이미지를 입력으로 받아 기계가 읽을 수 있는
텍스트로 해석하는 소프트웨어입니다. 은행 기계에 손으로 쓴 수표를 입금할 때
별다른 생각 없이 하지만, 배경에서는 흥미로운 작업이 진행되고 있습니다.
이 챕터에서는 인공 신경망(ANN)을 사용해 숫자를 인식하는 간단한 OCR 시스템의
실제 작동 예제를 살펴볼 것입니다. 하지만 먼저 좀 더 맥락을 설정해 보겠습니다.


## 인공지능이란 무엇인가?
\label{sec.ocr.ai}
튜링의 지능 정의는 합리적으로 들리지만, 결국 지능을 구성하는 것이
무엇인지는 근본적으로 철학적 논쟁입니다. 하지만 컴퓨터 과학자들은
특정 유형의 시스템과 알고리즘을 AI의 여러 분야로 분류해 왔습니다.
각 분야는 특정한 문제 집합을 해결하는 데 사용됩니다. 이러한 분야에는
다음 예제들과 [많은 다른 분야들](http://www-formal.stanford.edu/jmc/whatisai/node2.html)이
포함됩니다:

- 세계에 대한 미리 정의된 지식을 바탕으로 한 논리적이고 확률적인 추론과 추리.
  예를 들어 [퍼지 추론](http://www.cs.princeton.edu/courses/archive/fall07/cos436/HIDDEN/Knapp/fuzzy004.htm)은
  온도가 높고 습도가 높다는 것을 감지했을 때 온도조절기가 언제 에어컨을 켤지
  결정하는 데 도움을 줄 수 있습니다
- 휴리스틱 탐색. 예를 들어 체스 게임에서 모든 가능한 수를 탐색하고
  자신의 위치를 가장 개선하는 수를 선택하여 최선의 다음 수를 찾는 데
  탐색을 사용할 수 있습니다
- 피드백 모델을 가진 기계학습(ML). 예를 들어 OCR과 같은 패턴 인식 문제들.

일반적으로 ML은 패턴을 식별하도록 시스템을 훈련시키기 위해 대용량 데이터 세트를
사용하는 것을 포함합니다. 훈련 데이터 세트는 레이블이 있을 수도 있고(주어진 입력에 대해
시스템의 예상 출력이 명시됨) 레이블이 없을 수도 있습니다(예상 출력이 명시되지 않음).
레이블이 없는 데이터로 시스템을 훈련하는 알고리즘을 _비지도_ 알고리즘이라 하고,
레이블이 있는 데이터로 훈련하는 것을 _지도_ 알고리즘이라 합니다. OCR 시스템을
만들기 위한 많은 ML 알고리즘과 기법들이 존재하며, 그중 ANN도 하나의 접근법입니다.

## 인공 신경망
### ANN이란 무엇인가?
\label{sec.ocr.ann}
ANN은 서로 통신하는 상호 연결된 노드들로 구성된 구조입니다. 이 구조와
기능은 생물학적 뇌에서 발견되는 신경망에서 영감을 받았습니다. [헤브 이론](http://www.nbb.cornell.edu/neurobio/linster/BioNB420/hebb.pdf)은
이러한 네트워크가 물리적으로 구조와 연결 강도를 바꾸어 패턴을 식별하는 법을
학습할 수 있는 방법을 설명합니다. 마찬가지로, 일반적인 ANN(\aosafigref{500l.ocr.ann}에 표시됨)은
네트워크가 학습하면서 업데이트되는 가중치를 가진 노드 간 연결을 가지고 있습니다.
"+1"로 표시된 노드들을 _편향(bias)_이라고 합니다. 가장 왼쪽의 파란색 노드 열은
_입력 노드_이고, 가운데 열은 _은닉 노드_를 포함하며, 가장 오른쪽 열은 _출력 노드_를
포함합니다. _은닉 계층_으로 알려진 은닉 노드의 열이 여러 개 있을 수 있습니다.

\aosafigure[360pt]{ocr-images/ann.png}{인공 신경망}{500l.ocr.ann}

\aosafigref{500l.ocr.ann}에서 모든 원형 노드 안의 값들은 해당 노드의 출력을
나타냅니다. 계층 $L$에서 위에서부터 $n$번째 노드의 출력을 $a_n(L)$이라 하고,
계층 $L$의 $i$번째 노드와 계층 $L+1$의 $j$번째 노드 사이의 연결을
$w^{(L)}_{ji}$라고 하면, 노드 $a^{(2)}_2$의 출력은 다음과 같습니다:

$$
a^{(2)}_2 = f(w^{(1)}_{21}x_1 + w^{(1)}_{22}x_2 + b^{(1)}_{2})
$$

여기서 $f(.)$는 _활성화 함수_라고 하고 $b$는 _편향_입니다. 활성화 함수는
노드가 어떤 유형의 출력을 가질지를 결정하는 의사결정자입니다.
편향은 ANN의 정확도를 향상시키기 위해 추가될 수 있는 고정된 출력 1을 가진
추가 노드입니다. 이 둘에 대한 더 자세한 내용은 \aosasecref{sec.ocr.feedforward}에서
살펴보겠습니다.

이런 유형의 네트워크 토폴로지를 _순방향 공급_ 신경망이라고 부르는데,
네트워크에 사이클이 없기 때문입니다. 노드의 출력이 자신의 입력으로
되돌아가는 노드들을 가진 ANN을 순환 신경망이라고 합니다. 순방향 ANN을
훈련시키기 위해 적용할 수 있는 많은 알고리즘들이 있으며, 일반적으로
사용되는 알고리즘 중 하나를 _역전파_라고 합니다. 이 챕터에서 구현할
OCR 시스템은 역전파를 사용할 것입니다.

### ANN을 어떻게 사용하는가?
다른 대부분의 ML 접근법과 마찬가지로, 역전파를 사용하는 첫 번째 단계는
우리의 문제를 ANN으로 해결할 수 있는 문제로 변환하거나 축소하는 방법을
결정하는 것입니다. 다시 말해, 입력 데이터를 ANN에 공급할 수 있도록
어떻게 조작할 것인가입니다. OCR 시스템의 경우, 주어진 숫자에 대한
픽셀의 위치를 입력으로 사용할 수 있습니다. 종종 입력 데이터 형식을
선택하는 것이 이렇게 간단하지 않다는 점을 주목할 필요가 있습니다.
예를 들어, 대형 이미지에서 모양을 식별하기 위해 분석하는 경우,
이미지 내의 윤곽선을 식별하기 위해 이미지를 전처리해야 할 수 있습니다.
이러한 윤곽선이 입력이 될 것입니다.

입력 데이터 형식을 결정했다면, 다음은 무엇일까요? 역전파는 지도 알고리즘이므로,
\aosasecref{sec.ocr.ai}에서 언급한 바와 같이 레이블이 있는 데이터로 훈련되어야 합니다.
따라서 픽셀 위치를 훈련 입력으로 전달할 때, 연관된 숫자도 함께 전달해야 합니다.
이는 그려진 숫자와 연관된 값의 대용량 데이터 세트를 찾거나 수집해야 함을 의미합니다.

다음 단계는 데이터 세트를 훈련 세트와 검증 세트로 분할하는 것입니다.
훈련 데이터는 ANN의 가중치를 설정하기 위해 역전파 알고리즘을 실행하는 데 사용됩니다.
검증 데이터는 훈련된 네트워크를 사용해 예측을 만들고 그 정확도를 계산하는 데 사용됩니다.
우리 데이터에서 역전파와 다른 알고리즘의 성능을 비교한다면, [데이터를 분할](http://www-group.slac.stanford.edu/sluo/Lectures/stat_lecture_files/sluo2006lec7.pdf)하여
50%는 훈련용으로, 25%는 두 알고리즘의 성능 비교용으로(검증 세트), 나머지 25%는
선택된 알고리즘의 정확도 테스트용으로(테스트 세트) 사용할 것입니다.
알고리즘을 비교하지 않으므로, 25% 세트 중 하나를 훈련 세트의 일부로 묶어서
데이터의 75%를 네트워크 훈련에 사용하고 25%를 잘 훈련되었는지 검증하는 데 사용할 수 있습니다.

ANN의 정확도를 식별하는 목적은 두 가지입니다. 첫째, _과적합_ 문제를 피하기 위해서입니다.
과적합은 네트워크가 검증 세트보다 훈련 세트에 대한 예측에서 훨씬 높은 정확도를
가질 때 발생합니다. 과적합은 선택한 훈련 데이터가 충분히 일반화되지 않아서
개선이 필요하다는 것을 알려줍니다. 둘째, 여러 다른 수의 은닉 계층과 은닉 노드의
정확도를 테스트하는 것은 가장 최적의 ANN 크기를 설계하는 데 도움이 됩니다.
최적의 ANN 크기는 정확한 예측을 만들기에 충분한 은닉 노드와 계층을 가지면서도
훈련과 예측을 느리게 할 수 있는 계산 오버헤드를 줄이기 위해 가능한 한 적은
노드/연결을 가져야 합니다. 최적 크기가 결정되고 네트워크가 훈련되면,
예측할 준비가 된 것입니다!

## 간단한 OCR 시스템의 설계 결정사항들
\label{sec.ocr.decisions}
지난 몇 단락에서 순방향 ANN의 기본사항과 사용 방법을 살펴보았습니다.
이제 OCR 시스템을 어떻게 구축할 수 있는지에 대해 이야기할 때입니다.

먼저 우리 시스템이 무엇을 할 수 있기를 원하는지 결정해야 합니다.
단순하게 유지하기 위해, 사용자가 한 자리 숫자를 그릴 수 있도록 하고
그려진 숫자로 OCR 시스템을 훈련시키거나 시스템이 그려진 숫자가
무엇인지 예측하도록 요청할 수 있게 하겠습니다. OCR 시스템이
단일 기계에서 로컬로 실행될 수도 있지만, 클라이언트-서버 설정은
훨씬 더 많은 유연성을 제공합니다. 이는 ANN의 크라우드소싱 훈련을
가능하게 하고 강력한 서버가 집약적인 계산을 처리할 수 있게 합니다.

우리의 OCR 시스템은 5개 파일로 나뉜 5개의 주요 구성 요소로 구성됩니다:

- 클라이언트 (`ocr.js`)
- 서버 (`server.py`)
- 간단한 사용자 인터페이스 (`ocr.html`)
- 역전파로 훈련된 ANN (`ocr.py`)
- ANN 설계 스크립트 (`neural_network_design.py`)

사용자 인터페이스는 간단할 것입니다: 숫자를 그릴 수 있는 캔버스와
ANN을 훈련시키거나 예측을 요청하는 버튼들. 클라이언트는 그려진 숫자를
수집하고, 배열로 변환하고, 훈련 샘플이나 예측 요청으로 처리되도록
서버에 전달할 것입니다. 서버는 ANN 모듈에 API 호출을 함으로써
훈련이나 예측 요청을 단순히 라우팅할 것입니다.
ANN 모듈은 첫 번째 초기화 시에 기존 데이터 세트로 네트워크를 훈련할 것입니다.
그런 다음 ANN 가중치를 파일에 저장하고 후속 시작 시에 다시 로드할 것입니다.
이 모듈은 훈련과 예측 로직의 핵심이 일어나는 곳입니다. 마지막으로,
설계 스크립트는 다양한 은닉 노드 수를 실험하고 무엇이 가장 잘 작동하는지
결정하기 위한 것입니다. 이러한 조각들이 함께 매우 단순하지만
기능적인 OCR 시스템을 제공합니다.

이제 시스템이 높은 수준에서 어떻게 작동할지 생각해 보았으니,
개념을 코드로 구현할 때입니다!

### 간단한 인터페이스 (`ocr.html`)
앞서 언급했듯이, 첫 번째 단계는 네트워크 훈련을 위한 데이터를 수집하는 것입니다.
손으로 쓴 숫자들의 시퀀스를 서버에 업로드할 수도 있지만, 그것은 어색할 것입니다.
대신에, 사용자가 HTML 캔버스를 사용해 페이지에서 실제로 숫자를 손으로 쓸 수 있게
할 수 있습니다. 그런 다음 네트워크를 훈련시키거나 테스트하는 몇 가지 옵션을
제공할 수 있는데, 네트워크 훈련에는 어떤 숫자가 그려졌는지 명시하는 것도 포함됩니다.
이런 방식으로 사람들을 웹사이트로 안내해 그들의 입력을 받음으로써 데이터 수집을
쉽게 아웃소싱하는 것이 가능합니다. 시작하기 위한 HTML이 여기 있습니다. 

```html
<html>
<head>
	<script src="ocr.js"></script>
	<link rel="stylesheet" type="text/css" href="ocr.css">
</head>
<body onload="ocrDemo.onLoadFunction()">
	<div id="main-container" style="text-align: center;">
		<h1>OCR Demo</h1>
		<canvas id="canvas" width="200" height="200"></canvas>
		<form name="input">
			<p>Digit: <input id="digit" type="text"> </p>
			<input type="button" value="Train" onclick="ocrDemo.train()">
			<input type="button" value="Test" onclick="ocrDemo.test()">
			<input type="button" value="Reset" onclick="ocrDemo.resetCanvas();"/>
		</form> 
	</div>
</body>
</html>
```

### OCR 클라이언트 (`ocr.js`)
HTML 캔버스에서 하나의 픽셀은 보기 어려울 수 있으므로, ANN 입력을 위한
하나의 픽셀을 10x10 실제 픽셀의 정사각형으로 표현할 수 있습니다.
따라서 실제 캔버스는 200x200 픽셀이고 ANN의 관점에서는 20x20 캔버스로
표현됩니다. 아래의 변수들은 이러한 측정값들을 추적하는 데 도움이 될 것입니다.


```javascript
var ocrDemo = {
    CANVAS_WIDTH: 200,
    TRANSLATED_WIDTH: 20,
    PIXEL_WIDTH: 10, // TRANSLATED_WIDTH = CANVAS_WIDTH / PIXEL_WIDTH
```

그런 다음 새로운 표현에서 픽셀들의 윤곽을 그려서 더 쉽게 볼 수 있게 할 수 있습니다.
여기에 `drawGrid()`에 의해 생성된 파란색 격자가 있습니다.

```javascript
    drawGrid: function(ctx) {
        for (var x = this.PIXEL_WIDTH, y = this.PIXEL_WIDTH; 
                 x < this.CANVAS_WIDTH; x += this.PIXEL_WIDTH, 
                 y += this.PIXEL_WIDTH) {
            ctx.strokeStyle = this.BLUE;
            ctx.beginPath();
            ctx.moveTo(x, 0);
            ctx.lineTo(x, this.CANVAS_WIDTH);
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(this.CANVAS_WIDTH, y);
            ctx.stroke();
        }
    },
```

또한 격자에 그려진 데이터를 서버로 보낼 수 있는 형태로 저장해야 합니다.
간단함을 위해, 색칠되지 않은 검은 픽셀을 `0`으로, 색칠된 흰 픽셀을 `1`로
라벨링하는 `data`라고 불리는 배열을 가질 수 있습니다. 또한 사용자가 숫자를
그리는 동안 픽셀을 흰색으로 칠하기 위해 언제 `fillSquare()`를 호출할지 알 수
있도록 캔버스에 몇 가지 마우스 리스너가 필요합니다. 이러한 리스너들은
그리기 상태에 있는지를 추적하고 간단한 계산을 수행하여 어떤 픽셀을 채워야 하는지
결정하기 위해 `fillSquare()`를 호출해야 합니다. 

```javascript
    onMouseMove: function(e, ctx, canvas) {
        if (!canvas.isDrawing) {
            return;
        }
        this.fillSquare(ctx, 
            e.clientX - canvas.offsetLeft, e.clientY - canvas.offsetTop);
    },

    onMouseDown: function(e, ctx, canvas) {
        canvas.isDrawing = true;
        this.fillSquare(ctx, 
            e.clientX - canvas.offsetLeft, e.clientY - canvas.offsetTop);
    },

    onMouseUp: function(e) {
        canvas.isDrawing = false;
    },

    fillSquare: function(ctx, x, y) {
        var xPixel = Math.floor(x / this.PIXEL_WIDTH);
        var yPixel = Math.floor(y / this.PIXEL_WIDTH);
        this.data[((xPixel - 1)  * this.TRANSLATED_WIDTH + yPixel) - 1] = 1;

        ctx.fillStyle = '#ffffff';
        ctx.fillRect(xPixel * this.PIXEL_WIDTH, yPixel * this.PIXEL_WIDTH, 
            this.PIXEL_WIDTH, this.PIXEL_WIDTH);
    },
```

이제 흥미로운 부분에 가까워지고 있습니다! 서버로 보낼 훈련 데이터를 준비하는
함수가 필요합니다. 여기에 전송할 데이터에 대한 오류 검사를 수행하고,
`trainArray`에 추가한 다음 `sendData()`를 호출하여 전송하는 비교적 간단한
`train()` 함수가 있습니다. 

```javascript
    train: function() {
        var digitVal = document.getElementById("digit").value;
        if (!digitVal || this.data.indexOf(1) < 0) {
            alert("Please type and draw a digit value in order to train the network");
            return;
        }
        this.trainArray.push({"y0": this.data, "label": parseInt(digitVal)});
        this.trainingRequestCount++;

        // Time to send a training batch to the server.
        if (this.trainingRequestCount == this.BATCH_SIZE) {
            alert("Sending training data to server...");
            var json = {
                trainArray: this.trainArray,
                train: true
            };

            this.sendData(json);
            this.trainingRequestCount = 0;
            this.trainArray = [];
        }
    },
```
여기서 주목할 만한 흥미로운 설계는 `trainingRequestCount`, `trainArray`,
그리고 `BATCH_SIZE`의 사용입니다. 여기서 일어나는 일은 `BATCH_SIZE`가
클라이언트가 OCR에 의해 처리되기 위해 서버에 배치 요청을 보내기 전에
추적할 훈련 데이터의 양에 대한 미리 정의된 상수라는 것입니다.
요청을 배치로 처리하는 주된 이유는 한 번에 많은 요청으로 서버를 압도하는 것을
피하기 위해서입니다. 많은 클라이언트가 존재하거나 (예: 많은 사용자가 시스템을
훈련시키는 `ocr.html` 페이지에 있는 경우), 또는 스캔된 그려진 숫자를 가져와서
네트워크를 훈련시키기 위해 픽셀로 변환하는 다른 계층이 클라이언트에 존재한다면,
`BATCH_SIZE`가 1인 경우 많은 불필요한 요청이 발생할 것입니다. 이 접근법은
클라이언트에 더 많은 유연성을 제공하므로 좋지만, 실제로는 필요할 때
서버에서도 배치 처리가 이루어져야 합니다. 악의적인 클라이언트가 의도적으로
서버에 많은 요청을 보내서 서버를 압도하여 시스템을 다운시키는
서비스 거부(DoS) 공격이 발생할 수 있습니다.

또한 `test()` 함수도 필요합니다. `train()`과 유사하게, 데이터의 유효성에 대한
간단한 검사를 수행하고 전송해야 합니다. 하지만 `test()`의 경우에는
사용자가 예측을 요청하고 즉시 결과를 얻을 수 있어야 하므로 배치 처리가 발생하지 않습니다.

```javascript
    test: function() {
        if (this.data.indexOf(1) < 0) {
            alert("Please draw a digit in order to test the network");
            return;
        }
        var json = {
            image: this.data,
            predict: true
        };
        this.sendData(json);
    },
```

마지막으로, HTTP POST 요청을 만들고, 응답을 받고, 그 과정에서 발생할 수 있는
잠재적인 오류를 처리하는 몇 가지 함수가 필요합니다.

```javascript
    receiveResponse: function(xmlHttp) {
        if (xmlHttp.status != 200) {
            alert("Server returned status " + xmlHttp.status);
            return;
        }
        var responseJSON = JSON.parse(xmlHttp.responseText);
        if (xmlHttp.responseText && responseJSON.type == "test") {
            alert("The neural network predicts you wrote a \'" 
                   + responseJSON.result + '\'');
        }
    },

    onError: function(e) {
        alert("Error occurred while connecting to server: " + e.target.statusText);
    },

    sendData: function(json) {
        var xmlHttp = new XMLHttpRequest();
        xmlHttp.open('POST', this.HOST + ":" + this.PORT, false);
        xmlHttp.onload = function() { this.receiveResponse(xmlHttp); }.bind(this);
        xmlHttp.onerror = function() { this.onError(xmlHttp) }.bind(this);
        var msg = JSON.stringify(json);
        xmlHttp.setRequestHeader('Content-length', msg.length);
        xmlHttp.setRequestHeader("Connection", "close");
        xmlHttp.send(msg);
    }
```

### 서버 (`server.py`)

단순히 정보를 중계하는 작은 서버임에도 불구하고, HTTP 요청을 받고 처리하는 방법을
고려해야 합니다. 먼저 어떤 종류의 HTTP 요청을 사용할지 결정해야 합니다.
지난 섹션에서 클라이언트는 POST를 사용하고 있는데, 왜 이렇게 결정했을까요?
데이터가 서버로 전송되고 있으므로, PUT 또는 POST 요청이 가장 합리적입니다.
JSON 본문만 보내면 되고 URL 매개변수는 필요하지 않습니다. 따라서 이론적으로는
GET 요청도 작동할 수 있지만 의미상으로는 맞지 않을 것입니다. 하지만 PUT과 POST
사이의 선택은 프로그래머들 사이에서 길고 지속적인 논쟁입니다; KNPLabs가
이 문제들을 [유머와 함께](https://knpuniversity.com/screencast/rest/put-versus-post) 요약합니다.

또 다른 고려사항은 "train" 대 "predict" 요청을 서로 다른 엔드포인트
(예: `http://localhost/train`과 `http://localhost/predict`)로 보낼지,
아니면 데이터를 별도로 처리하는 동일한 엔드포인트로 보낼지입니다.
이 경우, 각각의 경우에 데이터로 수행되는 작업 간의 차이가 짧은 if 문에
맞을 정도로 작으므로 후자의 접근 방식을 사용할 수 있습니다.
실제로는 서버가 각 요청 유형에 대해 더 세부적인 처리를 수행한다면
이러한 엔드포인트를 분리하는 것이 더 나을 것입니다. 이 결정은 차례로
어떤 서버 오류 코드를 언제 사용할지에 영향을 미쳤습니다. 예를 들어,
페이로드에서 "train"이나 "predict" 모두 지정되지 않으면 400 "Bad Request"
오류가 전송됩니다. 대신 별도의 엔드포인트를 사용했다면 이것은 문제가 되지 않을
것입니다. OCR 시스템에 의해 백그라운드에서 수행되는 처리는 어떤 이유로든
실패할 수 있고, 서버 내에서 올바르게 처리되지 않으면 500 "Internal Server Error"가
전송됩니다. 다시 말해서, 엔드포인트가 분리되었다면 더 적절한 오류를 보내기 위해
세부사항에 들어갈 여지가 더 많았을 것입니다. 예를 들어, 내부 서버 오류가
실제로는 잘못된 요청에 의해 발생했다는 것을 식별하는 것입니다.

마지막으로, OCR 시스템을 언제 어디서 초기화할지 결정해야 합니다.
좋은 접근법은 `server.py` 내에서 초기화하되 서버가 시작되기 전에 하는 것입니다.
이는 첫 번째 실행 시에 OCR 시스템이 처음 시작될 때 기존 데이터에 대해 네트워크를
훈련시켜야 하고 이것이 몇 분 걸릴 수 있기 때문입니다. 이 처리가 완료되기 전에
서버가 시작된다면, 현재 구현에서는 OCR 객체가 아직 초기화되지 않았을 것이므로
훈련이나 예측 요청이 예외를 발생시킬 것입니다. 다른 가능한 구현으로는 새로운 ANN이
백그라운드에서 비동기적으로 훈련되는 동안 처음 몇 개의 쿼리에 사용할 부정확한 초기 ANN을
생성할 수도 있습니다. 이 대안적 접근법은 ANN을 즉시 사용할 수 있게 하지만,
구현이 더 복잡하고 서버가 재설정되는 경우에만 서버 시작 시간을 절약할 것입니다.
이러한 유형의 구현은 고가용성이 필요한 OCR 서비스에 더 유익할 것입니다.

여기에 POST 요청을 처리하는 하나의 짧은 함수에 우리 서버 코드의 대부분이 있습니다. 

```python
    def do_POST(s):
        response_code = 200
        response = ""
        var_len = int(s.headers.get('Content-Length'))
        content = s.rfile.read(var_len);
        payload = json.loads(content);

        if payload.get('train'):
            nn.train(payload['trainArray'])
            nn.save()
        elif payload.get('predict'):
            try:
                response = {
                    "type":"test", 
                    "result":nn.predict(str(payload['image']))
                }
            except:
                response_code = 500
        else:
            response_code = 400

        s.send_response(response_code)
        s.send_header("Content-type", "application/json")
        s.send_header("Access-Control-Allow-Origin", "*")
        s.end_headers()
        if response:
            s.wfile.write(json.dumps(response))
        return
```

### 순방향 ANN 설계하기 (`neural_network_design.py`)
\label{sec.ocr.feedforward}
순방향 ANN을 설계할 때 고려해야 할 몇 가지 요소가 있습니다.
첫 번째는 어떤 활성화 함수를 사용할지입니다. 앞서 활성화 함수를 노드의 출력에 대한
의사결정자로 언급했습니다. 활성화 함수가 내리는 결정의 유형이 어떤 것을 사용할지
결정하는 데 도움이 될 것입니다. 우리의 경우, 각 숫자(0-9)에 대해 0과 1 사이의
값을 출력하는 ANN을 설계할 것입니다. 1에 가까운 값은 ANN이 이것이 그려진 숫자라고
예측한다는 의미이고 0에 가까운 값은 그려진 숫자가 아니라고 예측한다는 의미입니다.
따라서 0에 가깝거나 1에 가까운 출력을 가질 활성화 함수를 원합니다.
또한 역전파 계산을 위해 미분이 필요하므로 미분 가능한 함수가 필요합니다.
이 경우 일반적으로 사용되는 함수는 이 두 제약 조건을 모두 만족하는 시그모이드입니다.
StatSoft는 일반적인 활성화 함수들과 그 속성들의 [좋은 목록](http://www.fmi.uni-sofia.bg/fmi/statist/education/textbook/eng/glosa.html)을
제공합니다.

고려해야 할 두 번째 요소는 편향을 포함할지 여부입니다. 앞서 편향을 몇 번 언급했지만
그것이 무엇인지 또는 왜 사용하는지에 대해서는 실제로 이야기하지 않았습니다.
\aosafigref{500l.ocr.ann}에서 노드의 출력이 어떻게 계산되는지 다시 살펴봄으로써 이를 이해해
보겠습니다. 하나의 입력 노드와 하나의 출력 노드가 있다고 가정하면, 출력 공식은
$y = f(wx)$가 될 것입니다. 여기서 $y$는 출력, $f()$는 활성화 함수, $w$는 노드 간
연결의 가중치, $x$는 노드의 변수 입력입니다. 편향은 본질적으로 출력이 항상 $1$인
노드입니다. 이것은 출력 공식을 $y = f(wx + b)$로 바꿀 것입니다. 여기서 $b$는
편향 노드와 다음 노드 사이의 연결 가중치입니다. $w$와 $b$를 상수로, $x$를 변수로
간주한다면, 편향을 추가하는 것은 $f(.)$에 대한 선형 함수 입력에 상수를 추가하는 것입니다.

따라서 편향을 추가하면 $y$ 절편의 이동을 허용하고 일반적으로 노드의 출력에 더 많은
유연성을 제공합니다. 편향을 포함하는 것이 종종 좋은 관행입니다. 특히 입력과 출력의 수가 적은 ANN의 경우에는 더욱 그렇습니다.
편향은 ANN의 출력에 더 많은 유연성을 허용하므로 ANN에 정확도를 위한 더 많은 여지를 제공합니다.
편향 없이는 우리의 ANN으로 올바른 예측을 할 가능성이 낮아지거나 더 정확한
예측을 위해 더 많은 은닉 노드가 필요할 것입니다.

고려해야 할 다른 요소들은 은닉 계층의 수와 계층당 은닉 노드의 수입니다.
많은 입력과 출력을 가진 더 큰 ANN의 경우, 이러한 수치들은 다양한 값을 시도하고
네트워크의 성능을 테스트함으로써 결정됩니다. 이 경우, 성능은 주어진 크기의 ANN을
훈련시키고 검증 세트의 몇 퍼센트가 올바르게 분류되는지 확인함으로써 측정됩니다.
대부분의 경우, 단일 은닉 계층이 괜찮은 성능을 위해 충분하므로, 여기서는
은닉 노드의 수에 대해서만 실험합니다.

```python
# Try various number of hidden nodes and see what performs best
for i in xrange(5, 50, 5):
    nn = OCRNeuralNetwork(i, data_matrix, data_labels, train_indices, False)
    performance = str(test(data_matrix, data_labels, test_indices, nn))
    print "{i} Hidden Nodes: {val}".format(i=i, val=performance)
```

여기서 5에서 50 사이의 은닉 노드를 5씩 증가시키며 ANN을 초기화합니다.
그런 다음 `test()` 함수를 호출합니다.

```python
def test(data_matrix, data_labels, test_indices, nn):
    avg_sum = 0
    for j in xrange(100):
        correct_guess_count = 0
        for i in test_indices:
            test = data_matrix[i]
            prediction = nn.predict(test)
            if data_labels[i] == prediction:
                correct_guess_count += 1

        avg_sum += (correct_guess_count / float(len(test_indices)))
    return avg_sum / 100
```

내부 루프는 올바른 분류의 수를 계산하여 마지막에 시도된 분류의 수로 나눕니다.
이것은 ANN에 대한 비율 또는 백분율 정확도를 제공합니다. ANN이 훈련될 때마다
가중치가 약간씩 다를 수 있으므로, 이 특정 ANN 구성의 정확도 평균을 구할 수 있도록
외부 루프에서 이 과정을 100번 반복합니다. 우리의 경우, `neural_network_design.py`의
샘플 실행은 다음과 같습니다:

```
PERFORMANCE
-----------
5 Hidden Nodes: 0.7792
10 Hidden Nodes: 0.8704
15 Hidden Nodes: 0.8808
20 Hidden Nodes: 0.8864
25 Hidden Nodes: 0.8808
30 Hidden Nodes: 0.888
35 Hidden Nodes: 0.8904
40 Hidden Nodes: 0.8896
45 Hidden Nodes: 0.8928
```

이 출력에서 15개의 은닉 노드가 가장 최적일 것이라고 결론 내릴 수 있습니다.
10에서 15로 5개 노드를 추가하면 약 1% 더 높은 정확도를 얻을 수 있지만,
정확도를 추가로 1% 향상시키려면 20개 노드를 더 추가해야 합니다.
은닉 노드 수를 늘리면 계산 오버헤드도 증가합니다. 따라서 더 많은 은닉 노드를 가진
네트워크는 훈련과 예측에 더 오랜 시간이 걸릴 것입니다. 그러므로 우리는 극적인
정확도 증가를 가져온 마지막 은닉 노드 수를 사용하기로 선택합니다.
물론, ANN을 설계할 때 계산 오버헤드가 문제가 되지 않고 가능한 한 가장 정확한
ANN을 갖는 것이 최우선 순위인 경우도 있습니다. 그런 경우에는 15개 대신
45개의 은닉 노드를 선택하는 것이 더 나을 것입니다.

### 핵심 OCR 기능

이 섹션에서는 역전파를 통해 실제 훈련이 어떻게 발생하는지, 네트워크를 사용해
예측을 만드는 방법, 그리고 핵심 기능을 위한 기타 주요 설계 결정사항들에 대해
이야기하겠습니다.

#### 역전파를 통한 훈련 (`ocr.py`)

우리는 ANN을 훈련시키기 위해 역전파 알고리즘을 사용합니다. 이것은
훈련 세트의 모든 샘플에 대해 반복되는 4가지 주요 단계로 구성되며,
매번 ANN 가중치를 업데이트합니다.

첫째, 가중치를 작은(-1과 1 사이) 무작위 값으로 초기화합니다.
우리의 경우, -0.06과 0.06 사이의 값으로 초기화하고 이를 `theta1`, `theta2`,
`input_layer_bias`, `hidden_layer_bias` 행렬에 저장합니다.
한 계층의 모든 노드가 다음 계층의 모든 노드에 연결되므로, n이 한 계층의
노드 수이고 m이 인접 계층의 노드 수인 m행 n열의 행렬을 만들 수 있습니다.
이 행렬은 이 두 계층 간의 연결에 대한 모든 가중치를 나타냅니다.
여기서 theta1은 우리의 20x20 픽셀 입력에 대해 400개의 열과 `num_hidden_nodes`개의 행을 가집니다.
마찬가지로, `theta2`는 은닉 계층과 출력 계층 사이의 연결을 나타냅니다.
이것은 `num_hidden_nodes`개의 열과 `NUM_DIGITS`(`10`)개의 행을 가집니다.
다른 두 벡터(1행), `input_layer_bias`와 `hidden_layer_bias`는
편향을 나타냅니다.

```python
    def _rand_initialize_weights(self, size_in, size_out):
        return [((x * 0.12) - 0.06) for x in np.random.rand(size_out, size_in)]
```

```python
            self.theta1 = self._rand_initialize_weights(400, num_hidden_nodes)
            self.theta2 = self._rand_initialize_weights(num_hidden_nodes, 10)
            self.input_layer_bias = self._rand_initialize_weights(1, 
                                                                  num_hidden_nodes)
            self.hidden_layer_bias = self._rand_initialize_weights(1, 10)

```

두 번째 단계는 _순방향 전파_로, 이는 본질적으로 \aosasecref{sec.ocr.ann}에서 설명한 대로
입력 노드부터 시작하여 계층별로 노드 출력을 계산하는 것입니다.
여기서 `y0`은 ANN 훈련에 사용하고자 하는 입력이 담긴 크기 400의 배열입니다.
`theta1`을 전치된 `y0`과 곱해서 크기가 `(num_hidden_nodes x 400) * (400 x 1)`인
두 행렬을 가지고 크기가 num_hidden_nodes인 은닉 계층의 출력 벡터를 얻습니다.
그런 다음 편향 벡터를 더하고 벡터화된 시그모이드 활성화 함수를 이 출력 벡터에 적용하여
`y1`을 얻습니다. `y1`은 은닉 계층의 출력 벡터입니다. 같은 과정이 출력 노드에 대해
`y2`를 계산하기 위해 다시 반복됩니다. `y2`는 이제 인덱스가 그려진 숫자일 가능성을
나타내는 값들을 가진 출력 계층 벡터입니다. 예를 들어, 누군가 8을 그리면,
ANN이 올바른 예측을 했다면 8번째 인덱스에서 `y2`의 값이 가장 클 것입니다.
하지만 6은 8과 더 유사하게 보이고 8을 그릴 때와 같은 픽셀을 사용할 가능성이
더 높으므로 그려진 숫자로서 1보다 더 높은 가능성을 가질 수 있습니다.
`y2`는 ANN이 훈련되는 추가 그려진 숫자마다 더 정확해집니다.

```python
    # The sigmoid activation function. Operates on scalars.
    def _sigmoid_scalar(self, z):
        return 1 / (1 + math.e ** -z)
```

```python
            y1 = np.dot(np.mat(self.theta1), np.mat(data['y0']).T)
            sum1 =  y1 + np.mat(self.input_layer_bias) # Add the bias
            y1 = self.sigmoid(sum1)

            y2 = np.dot(np.array(self.theta2), y1)
            y2 = np.add(y2, self.hidden_layer_bias) # Add the bias
            y2 = self.sigmoid(y2)
```

The third step is _back propagation_, which involves computing the errors at the
output nodes then at every intermediate layer back towards the input. Here we
start by creating an expected output vector, `actual_vals`, with a `1` at the index
of the digit that represents the value of the drawn digit and `0`s otherwise. The
vector of errors at the output nodes, `output_errors`, is computed by subtracting
the actual output vector, `y2`, from `actual_vals`. For every hidden layer
afterwards, we compute two components. First, we have the next layer’s
transposed weight matrix multiplied by its output errors. Then we have the
derivative of the activation function applied to the previous layer. We then
perform an element-wise multiplication on these two components, giving a vector
of errors for a hidden layer. Here we call this `hidden_errors`.

```python
            actual_vals = [0] * 10 
            actual_vals[data['label']] = 1
            output_errors = np.mat(actual_vals).T - np.mat(y2)
            hidden_errors = np.multiply(np.dot(np.mat(self.theta2).T, output_errors), 
                                        self.sigmoid_prime(sum1))
```

Weight updates that adjust the ANN weights based on the errors computed
earlier. Weights are updated at each layer via matrix multiplication. The error
matrix at each layer is multiplied by the output matrix of the previous layer.
This product is then multiplied by a scalar called the learning rate and added
to the weight matrix. The learning rate is a value between 0 and 1 that
influences the speed and accuracy of learning in the ANN. Larger learning rate
values will generate an ANN that learns quickly but is less accurate, while
smaller values will will generate an ANN that learns slower but is more
accurate. In our case, we have a relatively small value for learning rate, 0.1.
This works well since we do not need the ANN to be immediately trained in order
for a user to continue making train or predict requests. Biases are updated by
simply multiplying the learning rate by the layer’s error vector.

```python
            self.theta1 += self.LEARNING_RATE * np.dot(np.mat(hidden_errors), 
                                                       np.mat(data['y0']))
            self.theta2 += self.LEARNING_RATE * np.dot(np.mat(output_errors), 
                                                       np.mat(y1).T)
            self.hidden_layer_bias += self.LEARNING_RATE * output_errors
            self.input_layer_bias += self.LEARNING_RATE * hidden_errors
```

#### Testing a Trained Network (`ocr.py`)

Once an ANN has been trained via backpropagation, it is fairly straightforward
to use it for making predictions. As we can see here, we start by computing the
output of the ANN, `y2`, exactly the way we did in step 2 of backpropagation.
Then we look for the index in the vector with the maximum value. This index is
the digit predicted by the ANN.

```
    def predict(self, test):
        y1 = np.dot(np.mat(self.theta1), np.mat(test).T)
        y1 =  y1 + np.mat(self.input_layer_bias) # Add the bias
        y1 = self.sigmoid(y1)

        y2 = np.dot(np.array(self.theta2), y1)
        y2 = np.add(y2, self.hidden_layer_bias) # Add the bias
        y2 = self.sigmoid(y2)

        results = y2.T.tolist()[0]
        return results.index(max(results))
```

#### Other Design Decisions (`ocr.py`)
Many resources are available online that go into greater detail on the
implementation of backpropagation. One good resource is from a [course by the
University of
Willamette](http://www.willamette.edu/~gorr/classes/cs449/backprop.html). It
goes over the steps of backpropagation and then explains how it can be
translated into matrix form. While the amount of computation using matrices is
the same as using loops, the benefit is that the code is simpler and easier to
read with fewer nested loops. As we can see, the entire training process is
written in under 25 lines of code using matrix algebra.

As mentioned in the introduction of \aosasecref{sec.ocr.decisions}, persisting
the weights of the ANN means we do not lose the progress made in training it
when the server is shut down or abruptly goes down for any reason. We persist
the weights by writing them as JSON to a file. On startup, the OCR loads the
ANN’s saved weights to memory. The save function is not called internally by
the OCR but is up to the server to decide when to perform a save. In our case,
the server saves the weights after each update. This is a quick and simple
solution but it is not optimal since writing to disk is time consuming. This
also prevents us from handling multiple concurrent requests since there is no
mechanism to prevent simultaneous writes to the same file. In a more
sophisticated server, saves could perhaps be done on shutdown or once every few
minutes with some form of locking or a timestamp protocol to ensure no data
loss.

```python
    def save(self):
        if not self._use_file:
            return

        json_neural_network = {
            "theta1":[np_mat.tolist()[0] for np_mat in self.theta1],
            "theta2":[np_mat.tolist()[0] for np_mat in self.theta2],
            "b1":self.input_layer_bias[0].tolist()[0],
            "b2":self.hidden_layer_bias[0].tolist()[0]
        };
        with open(OCRNeuralNetwork.NN_FILE_PATH,'w') as nnFile:
            json.dump(json_neural_network, nnFile)

    def _load(self):
        if not self._use_file:
            return

        with open(OCRNeuralNetwork.NN_FILE_PATH) as nnFile:
            nn = json.load(nnFile)
        self.theta1 = [np.array(li) for li in nn['theta1']]
        self.theta2 = [np.array(li) for li in nn['theta2']]
        self.input_layer_bias = [np.array(nn['b1'][0])]
        self.hidden_layer_bias = [np.array(nn['b2'][0])]
```

## 결론
이제 AI, ANN, 역전파, 그리고 종단간 OCR 시스템 구축에 대해 배웠으므로,
이 챕터의 하이라이트와 큰 그림을 요약해 보겠습니다.

우리는 AI, ANN, 그리고 대략적으로 구현할 내용에 대한 배경지식을 제공하는 것으로
챕터를 시작했습니다. AI가 무엇이고 어떻게 사용되는지에 대한 예를 논의했습니다.
AI는 본질적으로 인간이 하는 것과 유사한 방식으로 질문에 대한 답을 제공할 수 있는
알고리즘 집합 또는 문제 해결 접근법이라는 것을 보았습니다. 그런 다음 순방향 ANN의
구조를 살펴보았습니다. 주어진 노드에서의 출력을 계산하는 것이
이전 노드들의 출력과 그들의 연결 가중치의 곱의 합만큼 간단하다는 것을 배웠습니다.
먼저 입력을 형식화하고 데이터를 훈련 세트와 검증 세트로 분할함으로써 ANN을 사용하는
방법에 대해 이야기했습니다.

배경지식을 갖춘 후에, OCR을 훈련시키거나 테스트하는 사용자 요청을 처리할
웹 기반 클라이언트-서버 시스템 생성에 대해 이야기하기 시작했습니다.
그런 다음 클라이언트가 그려진 픽셀을 배열로 해석하고 훈련이나 테스트를 수행하기 위해
OCR 서버에 HTTP 요청을 수행하는 방법을 논의했습니다. 간단한 서버가 요청을 읽는 방법과
여러 은닉 노드 수의 성능을 테스트해서 ANN을 설계하는 방법을 논의했습니다.
역전파를 위한 핵심 훈련 및 테스트 코드를 살펴보는 것으로 마무리했습니다.

겉으로 보기에는 기능적인 OCR 시스템을 구축했지만, 이 챕터는 실제 OCR 시스템이
어떻게 작동할 수 있는지에 대한 표면만 긁은 것입니다. 더 정교한 OCR 시스템은
전처리된 입력, 하이브리드 ML 알고리즘 사용, 더 광범위한 설계 단계, 또는 기타
추가 최적화를 가질 수 있습니다.
