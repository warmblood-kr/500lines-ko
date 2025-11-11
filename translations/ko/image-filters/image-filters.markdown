title: 나만의 이미지 필터 만들기
author: Cate Huston
<markdown>
_케이트는 기술 업계를 떠나 1년간 자신만의 길을 찾으며 열정 프로젝트인 Show & Hide를 구축했습니다. 현재 Ride의 모바일 엔지니어링 디렉터로 일하며, 모바일 개발과 엔지니어링 문화에 대해 국제적으로 강연하고, Technically Speaking을 공동 큐레이팅하며 Glowforge의 어드바이저로 활동하고 있습니다. 케이트는 정확히 콜롬비아에 살지는 않지만 그곳에서 많은 시간을 보내며, 영국, 호주, 캐나다, 중국, 미국에서 살고 일한 경험이 있습니다. 이전에는 Google의 엔지니어, IBM의 Extreme Blue 인턴, 스키 강사로 일했습니다. 케이트는 [Accidentally in Code](http://www.catehuston.com/blog/)에서 블로그를 쓰고 트위터에서 [\@catehstn](https://twitter.com/catehstn)로 활동하고 있습니다._
</markdown>
## 훌륭한 아이디어 (그런데 실제로는 그리 훌륭하지 않았던)

중국을 여행할 때 같은 장소를 다른 계절에 그린 네 개의 연작 그림을 자주 보게 되었습니다. 색깔&mdash;겨울의 차가운 하얀색, 봄의 연한 색조, 여름의 풍성한 녹색, 가을의 빨강과 노랑&mdash;이 계절을 시각적으로 구분해주는 요소였습니다. 2011년경, 나는 훌륭하다고 생각한 아이디어를 떠올렸습니다: 사진 시리즈를 색상 시리즈로 시각화할 수 있다면 좋겠다고 생각했습니다. 이를 통해 여행과 계절의 변화를 보여줄 수 있을 거라 생각했습니다.

하지만 이미지에서 주요 색상을 계산하는 방법을 몰랐습니다. 이미지를 1x1 정사각형으로 축소한 후 남은 색상을 확인하는 것도 생각해봤지만, 그것은 속임수 같았습니다. 하지만 이미지를 어떻게 표시할지는 알고 있었습니다: [해바라기 레이아웃](http://www.catehuston.com/applets/Sunflower/index.html)이라는 배치 방식이었습니다. 이것은 원들을 배치하는 가장 효율적인 방법입니다.

나는 이 프로젝트를 몇 년 동안 방치했습니다. 일, 생활, 여행, 강연에 정신이 팔려서였죠. 결국 다시 돌아와서 주요 색상을 계산하는 방법을 알아내고 [시각화를 완성했습니다](http://www.catehuston.com/blog/2013/09/02/visualising-a-photo-series/). 그때 이 아이디어가 사실 그리 훌륭하지 않다는 것을 발견했습니다. 진행 과정이 기대만큼 명확하지 않았고, 추출된 주요 색상이 일반적으로 가장 매력적인 색조가 아니었으며, 생성하는 데 오랜 시간이 걸렸고(이미지당 몇 초씩), 멋진 것을 만들기 위해서는 수백 개의 이미지가 필요했습니다(\aosafigref{500l.imagefilters.sunflower}).

\aosafigure[180pt]{image-filters-images/sunflower.jpg}{Sunflower layout}{500l.imagefilters.sunflower}

이것이 실망스러울 수 있다고 생각할지도 모르겠지만, 이 지점에 이르렀을 때 나는 이전에는 접해보지 못했던 많은 것들을 배웠습니다. 색 공간과 픽셀 조작에 대한 지식이었죠. 그리고 멋진 부분 컬러 이미지들을 만들기 시작했습니다. 런던의 빨간 버스나 전화 부스만 컬러로 하고 나머지는 모두 회색조로 처리한 엽서에서 볼 수 있는 그런 이미지 말입니다.

나는 [Processing](https://processing.org/)이라는 프레임워크를 사용했습니다. 프로그래밍 커리큘럼을 개발할 때 익숙해졌고, 시각적 애플리케이션을 쉽게 만들 수 있다는 것을 알고 있었기 때문입니다. 이것은 원래 예술가들을 위해 설계된 도구로, 많은 상용구 코드를 추상화해줍니다. 덕분에 자유롭게 실험해볼 수 있었습니다.

대학교, 그리고 나중에 직장은 다른 사람들의 아이디어와 우선순위로 내 시간을 채워버렸습니다. 이 프로젝트를 완성하는 과정의 일부는 내 자신의 아이디어를 발전시킬 수 있는 시간을 확보하는 방법을 배우는 것이었습니다. 일주일에 약 4시간의 좋은 정신적 시간이 필요했습니다. 따라서 더 빠르게 작업할 수 있게 해주는 도구는 정말 도움이 되었고, 심지어 필수적이었습니다&mdash;비록 테스트 작성과 관련해서는 고유한 문제점들이 있었지만요. 

철저한 테스트가 프로젝트가 어떻게 작동하는지 검증하고, 몇 주, 심지어 몇 달씩 중단된 프로젝트를 다시 시작하기 쉽게 만드는 데 특히 중요하다고 느꼈습니다. 테스트(그리고 블로그 포스트!)가 이 프로젝트의 문서 역할을 했습니다. 아직 해결하지 못한 부분에 대해 어떤 일이 일어나야 하는지 문서화하기 위해 실패하는 테스트를 남겨둘 수 있었고, 중요했던 부분을 잊고 변경했을 때 테스트가 상기시켜 줄 것이라는 확신을 가지고 변경할 수 있었습니다.

이 장에서는 Processing에 대한 몇 가지 세부사항을 다루고, 색 공간, 이미지를 픽셀로 분해하고 조작하는 방법, 그리고 테스트를 염두에 두고 설계되지 않은 것을 단위 테스트하는 방법에 대해 설명합니다. 하지만 최근에 시간을 내지 못했던 아이디어를 발전시키는 데도 도움이 되기를 바랍니다. 여러분의 아이디어가 내 것만큼 끔찍한 것으로 판명되더라도, 과정에서 멋진 것을 만들고 흥미로운 것을 배울 수 있을 겁니다.

## 애플리케이션

이 장에서는 직접 만든 필터를 사용하여 디지털 이미지를 조작할 수 있는 이미지 필터 애플리케이션을 만드는 방법을 보여줍니다. Java로 구축된 프로그래밍 언어이자 개발 환경인 Processing을 사용할 것입니다. Processing에서 애플리케이션을 설정하는 방법, Processing의 몇 가지 기능, 색상 표현의 측면, 그리고 (옛날 사진술에서 사용되던 것을 모방한) 색상 필터를 만드는 방법을 다룰 것입니다. 또한 디지털로만 가능한 특별한 종류의 필터도 만들어볼 것입니다: 이미지의 주요 색조를 결정하고 이를 보이거나 숨기는 것으로, 으스스한 부분 컬러 이미지를 만드는 방법입니다.

마지막으로, 철저한 테스트 모음을 추가하고 테스트 가능성 측면에서 Processing의 몇 가지 제한사항을 처리하는 방법을 다룰 것입니다.

## 배경

오늘날 우리는 사진을 찍고, 조작하고, 몇 초 안에 모든 친구들과 공유할 수 있습니다. 하지만 (디지털 용어로) 아주 오래전에는 몇 주가 걸리는 과정이었습니다.

옛날에는 사진을 찍은 다음, 필름 한 롤을 다 사용하면 현상을 위해 가져다 줬습니다(보통 약국에서). 며칠 후 현상된 사진들을 가져와서&mdash;그 중 많은 사진에 문제가 있다는 것을 발견했습니다. 손이 충분히 안정적이지 않았나? 당시에는 알아차리지 못한 무작위 사람이나 사물이 들어갔나? 과다 노출? 노출 부족? 물론 그때는 이미 문제를 해결하기에는 너무 늦었죠.

필름을 사진으로 바꾸는 과정은 대부분의 사람들이 이해하지 못하는 것이었습니다. 빛이 문제가 되어서 필름을 조심스럽게 다뤄야 했죠. 어두운 방과 화학물질이 관련된 과정이 있었는데, 영화나 TV에서 가끔 보여주곤 했습니다.

하지만 스마트폰 카메라의 포인트 앤 클릭에서 인스타그램 이미지에 이르는 과정을 이해하는 사람들은 아마 더 적을 것입니다. 실제로는 많은 유사점이 있습니다.

### 사진술, 옛날 방식

사진은 빛에 민감한 표면에 빛이 미치는 효과로 만들어집니다. 사진 필름은 할로겐화은 결정으로 덮여 있습니다. (컬러 사진을 만들기 위해서는 추가 레이어가 사용됩니다 — 단순화를 위해 여기서는 흑백 사진만 다루겠습니다.) 

옛날 방식의 사진 — 필름을 사용한 — 을 찍을 때, 빛은 당신이 가리키는 것에 따라 필름에 맞습니다. 그리고 그 지점의 결정들은 빛의 양에 따라 다양한 정도로 변화됩니다. 그런 다음, [현상 과정](http://photography.tutsplus.com/tutorials/step-by-step-guide-to-developing-black-and-white-t-max-film--photo-2580)에서 은염을 금속 은으로 변환하여 네거티브를 만듭니다. 네거티브는 이미지의 밝고 어두운 부분이 반전되어 있습니다. 네거티브가 현상되면, 이미지를 뒤집어서 인쇄하는 또 다른 일련의 단계들이 있습니다.

### 사진술, 디지털 방식

스마트폰이나 디지털 카메라를 사용해서 사진을 찍을 때는 필름이 없습니다. 비슷한 방식으로 작동하는 *능동 픽셀 센서*라는 것이 있습니다. 예전에 은 결정이 있던 자리에 이제는 픽셀 — 작은 사각형들 — 이 있습니다. (사실, 픽셀은 "picture element"의 줄임말입니다.) 디지털 이미지는 픽셀로 구성되며, 해상도가 높을수록 더 많은 픽셀이 있습니다. 이것이 저해상도 이미지가 "픽셀화되었다"고 설명되는 이유입니다 — 사각형들을 보기 시작할 수 있기 때문입니다. 이러한 픽셀들은 배열에 저장되며, 각 배열 "박스"의 숫자는 색상을 포함합니다.

\aosafigref{500l.imagefilters.animals}에서는 뉴욕의 MoMA에서 찍은 풍선 동물들의 고해상도 사진을 볼 수 있습니다. \aosafigref{500l.imagefilters.pixelanimals}는 같은 이미지를 확대한 것이지만 24 x 32 픽셀만 사용했습니다.

\aosafigure[220pt]{image-filters-images/animals.jpg}{Blow-up animals at MoMA NY}{500l.imagefilters.animals}

\aosafigure[220pt]{image-filters-images/pixelanimals.jpg}{Blow-up animals, blown up}{500l.imagefilters.pixelanimals}

얼마나 흐릿한지 보이시나요? 이것을 _픽셀화_라고 부르는데, 이미지가 포함하고 있는 픽셀 수에 비해 너무 크기 때문에 사각형들이 보이게 되는 것을 의미합니다. 여기서는 이미지가 색상의 사각형들로 구성되어 있다는 것을 더 잘 이해할 수 있도록 사용할 수 있습니다.

이러한 픽셀들은 어떻게 생겼을까요? Java의 편리한 `Integer.toHexString`을 사용하여 중간 부분의 일부 픽셀들(10,10에서 10,14까지)의 색상을 출력하면 16진수 색상을 얻습니다:

```
FFE8B1
FFFAC4
FFFCC3
FFFCC2
FFF5B7
```


16진수 색상은 6자리 길이입니다. 처음 두 자리는 빨강 값, 다음 두 자리는 초록 값, 마지막 두 자리는 파랑 값입니다. 때로는 알파 값인 추가 두 자리가 있기도 합니다. 이 경우 `FFFAC4`는 다음을 의미합니다:

\newpage

- 빨강 = FF (16진수) = 255 (10진수)
- 초록 = FA (16진수) = 250 (10진수)
- 파랑 = C4 (16진수) = 196 (10진수)

## 애플리케이션 실행하기

\aosafigref{500l.imagefilters.app}에서는 실행 중인 애플리케이션의 사진을 볼 수 있습니다. 개발자가 디자인한 것 같다는 걸 알고 있지만, 500줄의 Java 코드만 사용할 수 있었기 때문에 뭔가는 포기해야 했습니다! 오른쪽에서 명령어 목록을 볼 수 있습니다. 할 수 있는 일들은 다음과 같습니다:

- RGB 필터를 조정합니다.
- "색조 허용범위"를 조정합니다.
- 주요 색조 필터를 설정하여 주요 색조를 보이거나 숨깁니다.
- 현재 설정을 적용합니다 (모든 키 입력마다 실행하는 것은 비현실적입니다).
- 이미지를 재설정합니다.
- 만든 이미지를 저장합니다.

\aosafigure[266pt]{image-filters-images/app.jpg}{The App}{500l.imagefilters.app}

Processing을 사용하면 작은 애플리케이션을 만들고 이미지 조작을 수행하는 것이 간단해집니다. 매우 시각적인 초점을 가지고 있습니다. Processing이 이제 다른 언어로 이식되었지만, 우리는 Java 기반 버전으로 작업할 것입니다.

이 튜토리얼에서는 Eclipse에서 빌드 경로에 `core.jar`를 추가하여 Processing을 사용합니다. 원한다면 Processing IDE를 사용할 수 있는데, 이는 많은 상용구 Java 코드의 필요성을 제거해줍니다. 나중에 Processing.js로 이식하여 온라인에 업로드하려면 파일 선택기를 다른 것으로 교체해야 합니다.

프로젝트의 [저장소](https://github.com/aosabook/500lines/blob/master/image-filters/SETUP.MD)에 스크린샷과 함께 상세한 지침이 있습니다. Eclipse와 Java에 이미 익숙하다면 필요하지 않을 수도 있습니다.

## Processing 기초

### 크기와 색상

애플리케이션이 작은 회색 창이 되는 것을 원하지 않으므로, 우선 재정의할 두 가지 필수 메서드는 [`setup()`](http://processing.org/reference/setup_.html)과 [`draw()`](http://processing.org/reference/draw_.html)입니다. `setup()` 메서드는 앱이 시작될 때만 호출되며, 앱 창의 크기를 설정하는 것과 같은 일들을 수행하는 곳입니다. `draw()` 메서드는 모든 애니메이션에 대해 호출되거나 `redraw()`를 호출하여 어떤 동작이 트리거된 후에 호출됩니다. (
Processing 문서에 따르면 `draw()`는 명시적으로 호출되어서는 안 됩니다.)

Processing은 애니메이션 스케치를 만들기 위해 잘 작동하도록 설계되었지만, 이 경우에는 애니메이션[^noanim]을 원하지 않고 키 입력에 응답하기를 원합니다. (성능에 부담이 될) 애니메이션을 방지하기 위해 setup에서 [`noLoop()`](http://www.processing.org/reference/noLoop_.html)를 호출할 것입니다. 이는 `draw()`가 `setup()` 직후와 `redraw()`를 호출할 때마다만 호출된다는 것을 의미합니다.

[^noanim]: 애니메이션 스케치를 만들고 싶다면 `noLoop()`을 호출하지 않을 것입니다 (또는, 나중에 애니메이션을 시작하고 싶다면 `loop()`을 호출할 것입니다). 애니메이션의 빈도는 `frameRate()`에 의해 결정됩니다.

```java
private static final int WIDTH = 360;
private static final int HEIGHT = 240;

public void setup() {
  noLoop();

  // Set up the view.
  size(WIDTH, HEIGHT);
  background(0);
}
    
public void draw() {
  background(0);
}
```

이것들은 아직 많은 일을 하지 않지만, `WIDTH`와 `HEIGHT`의 상수를 조정하여 다른 크기를 보기 위해 앱을 다시 실행해보세요.

`background(0)`은 검은색 배경을 지정합니다. `background()`에 전달되는 숫자를 바꿔보세요 — 이것은 알파 값이며, 하나의 숫자만 전달하면 항상 회색조입니다. 대안으로 `background(int r, int g, int b)`를 호출할 수 있습니다.

### PImage

[PImage 객체](http://processing.org/reference/PImage.html)는 이미지를 나타내는 Processing 객체입니다. 이것을 많이 사용할 예정이므로 문서를 읽어보는 것이 좋습니다. 세 개의 필드(\aosatblref{500l.imagefilters.pimagefields})와 우리가 사용할 몇 가지 메서드(\aosatblref{500l.imagefilters.pimagemethods})가 있습니다.

<markdown>
<table>
  <tr>
    <td>`pixels[]`</td>
    <td>이미지의 모든 픽셀 색상을 포함하는 배열</td>
  </tr>
  <tr>
    <td>`width`</td>
    <td>픽셀 단위 이미지 너비</td>
  </tr>
  <tr>
    <td>`height`</td>
    <td>픽셀 단위 이미지 높이</td>
  </tr>
</table>
: \label{500l.imagefilters.pimagefields} PImage 필드
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
pixels[] & 이미지의 모든 픽셀 색상을 포함하는 배열 \\
width & 픽셀 단위 이미지 너비 \\
height & 픽셀 단위 이미지 높이 \\
\hline
\end{tabular}
}
\caption{PImage 필드}
\label{500l.imagefilters.pimagefields}
\end{table}
</latex>

<markdown>
<table>
  <tr>
    <td>`loadPixels`</td>
    <td>이미지의 픽셀 데이터를 `pixels[]` 배열로 로드</td>
  </tr>
  <tr>
    <td>`updatePixels`</td>
    <td>`pixels[]` 배열의 데이터로 이미지 업데이트</td>
  </tr>
  <tr>
    <td>`resize`</td>
    <td>이미지의 크기를 새로운 너비와 높이로 변경</td>
  </tr>
  <tr>
    <td>`get`</td>
    <td>픽셀의 색상을 읽거나 픽셀 사각형 영역을 추출</td>
  </tr>
  <tr>
    <td>`set`</td>
    <td>픽셀에 색상을 쓰거나 다른 이미지에 이미지 삽입</td>
  </tr>
  <tr>
    <td>`save`</td>
    <td>이미지를 TIFF, TARGA, PNG, 또는 JPEG 파일로 저장</td>
  </tr>
</table>
: \label{500l.imagefilters.pimagemethods} PImage 메서드
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
loadPixels & 이미지의 픽셀 데이터를 `pixels[]` 배열로 로드 \\
updatePixels & `pixels[]` 배열의 데이터로 이미지 업데이트 \\
resize & 이미지의 크기를 새로운 너비와 높이로 변경 \\
get & 픽셀의 색상을 읽거나 픽셀 사각형 영역을 추출 \\
set & 픽셀에 색상을 쓰거나 다른 이미지에 이미지 삽입 \\
save & 이미지를 TIFF, TARGA, PNG, 또는 JPEG 파일로 저장 \\
\hline
\end{tabular}
}
\caption{PImage 메서드}
\label{500l.imagefilters.pimagemethods}
\end{table}
</latex>

### 파일 선택기
Processing이 파일 선택 과정의 대부분을 처리해주므로, 우리는 [`selectInput()`](http://www.processing.org/reference/selectInput_.html)을 호출하고 콜백 함수를 구현하기만 하면 됩니다(반드시 public이어야 함).

Java에 익숙한 사람들에게는 이것이 이상하게 보일 수 있습니다. 리스너나 람다 표현식이 더 합리적일 것 같기 때문입니다. 하지만 Processing은 예술가들을 위한 도구로 개발되었기 때문에, 대부분의 이런 것들이 언어에 의해 추상화되어 부담스럽지 않도록 만들어졌습니다. 이것은 설계자들이 내린 선택입니다: 강력함과 유연성보다는 단순함과 접근성을 우선시한 것입니다. Eclipse에서 Processing을 라이브러리로 사용하는 것이 아니라 간소화된 Processing 에디터를 사용한다면, 클래스명을 정의할 필요조차 없습니다.

다른 대상 사용자를 둔 다른 언어 설계자들은 서로 다른 선택을 하며, 그래야 합니다. 예를 들어, 순수 함수형 언어인 Haskell에서는 함수형 언어 패러다임의 순수성이 다른 모든 것보다 우선시됩니다. 이것은 Haskell을 IO가 필요한 작업보다는 수학적 문제에 더 적합한 도구로 만듭니다.

```java
// Called on key press.
private void chooseFile() {
  // Choose the file.
  selectInput("Select a file to process:", "fileSelected");
}

public void fileSelected(File file) {
  if (file == null) {
    println("User hit cancel.");
  } else {
    // save the image
    redraw(); // update the display
  }
}
```

### 키 입력 처리

일반적으로 Java에서 키 입력에 응답하려면 리스너를 추가하고 익명 함수를 구현해야 합니다. 하지만 파일 선택기와 마찬가지로, Processing이 이런 것들을 많이 처리해줍니다. 우리는 [`keyPressed()`](https://www.processing.org/reference/keyPressed_.html)만 구현하면 됩니다.

```java
public void keyPressed() {
  print("key pressed: " + key);
}
```

앱을 다시 실행하면, 키를 누를 때마다 콘솔에 출력됩니다. 나중에는 어떤 키가 눌렸는지에 따라 다른 작업을 하고 싶을 것이고, 이를 위해서는 키 값에 대한 switch문을 사용하면 됩니다. (이는 `PApplet` 슈퍼클래스에 있으며, 마지막에 눌린 키를 포함합니다.) 


## 테스트 작성

이 앱은 아직 많은 기능을 하지 않지만, 이미 문제가 발생할 수 있는 여러 부분을 볼 수 있습니다. 예를 들어 키 입력으로 잘못된 동작을 트리거하는 경우입니다. 복잡성을 추가할수록 이미지 상태를 잘못 업데이트하거나 필터 적용 후 픽셀 색상을 잘못 계산하는 등 더 많은 잠재적 문제를 추가하게 됩니다. 저는 또한 (어떤 사람들은 이상하다고 생각하지만) 단위 테스트를 작성하는 것을 즐깁니다. 어떤 사람들은 테스팅을 코드 체크인을 지연시키는 것으로 생각하는 것 같지만, 저는 테스트를 제 1의 디버깅 도구로, 그리고 코드에서 무슨 일이 일어나고 있는지 깊이 이해할 기회로 봅니다.

저는 Processing을 좋아하지만, 이는 시각적 애플리케이션을 만들기 위해 설계되었고, 이 영역에서는 단위 테스팅이 큰 관심사가 아닐 수도 있습니다. 테스트 가능성을 위해 작성되지 않았다는 것이 명백합니다. 실제로 그대로는 테스트할 수 없도록 작성되었습니다. 이것의 일부는 복잡성을 숨기기 때문이며, 그 숨겨진 복잡성 중 일부는 단위 테스트 작성에 정말 유용합니다. static 및 final 메서드의 사용은 서브클래싱 능력에 의존하는 목(mock)(시스템의 일부를 가짜로 만들어 다른 부분이 올바르게 작동하는지 확인할 수 있도록 상호작용을 기록하는 객체)을 사용하기 훨씬 어렵게 만듭니다. 

우리는 테스트 주도 개발(TDD)을 하고 완벽한 테스트 커버리지를 달성하겠다는 훌륭한 의도로 새로운 프로젝트를 시작할 수도 있지만, 실제로는 다양한 사람들이 작성한 코드 덩어리를 보면서 그것이 무엇을 하도록 되어 있는지, 어떻게 그리고 왜 잘못되고 있는지 알아내려고 하는 경우가 대부분입니다. 그러면 완벽한 테스트를 작성하지는 못하더라도, 테스트를 전혀 작성하지 않는 것보다는 상황을 탐색하고, 무슨 일이 일어나고 있는지 문서화하고, 앞으로 나아가는 데 도움이 될 것입니다.

우리는 얽힌 조각들의 무정형 덩어리에서 무언가를 분해하고 부분적으로 검증할 수 있게 해주는 "솔기(seam)"를 만듭니다. 이를 위해 때로는 모킹될 수 있는 래퍼 클래스들을 만듭니다. 이런 클래스들은 유사한 메서드들의 집합을 보유하거나, (final 또는 static 메서드로 인해) 모킹될 수 없는 다른 객체로 호출을 전달하는 것 이상은 하지 않으며, 그런 점에서 작성하기 매우 지루하지만 솔기를 만들고 코드를 테스트 가능하게 만드는 데 핵심적입니다.

저는 Processing을 라이브러리로 사용하여 Java로 작업했기 때문에 테스트를 위해 JUnit을 사용했습니다. 모킹을 위해서는 Mockito를 사용했습니다. [Mockito](https://code.google.com/p/mockito/downloads/list)를 다운로드하고 `core.jar`를 추가한 것과 같은 방식으로 JAR을 빌드 경로에 추가할 수 있습니다. 앱을 모킹하고 테스트할 수 있게 해주는 두 개의 헬퍼 클래스를 만들었습니다(그렇지 않으면 `PImage`나 `PApplet` 메서드와 관련된 동작을 테스트할 수 없습니다).

`IFAImage`는 PImage 주변의 얇은 래퍼입니다. `PixelColorHelper`는 애플릿 픽셀 색상 메서드들을 감싸는 래퍼입니다. 이런 래퍼들은 final 및 static 메서드들을 호출하지만, 호출자 메서드들 자체는 final도 static도 아닙니다 — 이것이 그것들을 모킹 가능하게 만듭니다. 이것들은 의도적으로 가볍게 만들어졌고, 더 나아갈 수도 있었지만, Processing을 사용할 때의 테스트 가능성의 주요 문제 — static 및 final 메서드들 — 를 해결하는 데 충분했습니다. 결국 목표는 앱을 만드는 것이었습니다 — Processing을 위한 단위 테스트 프레임워크를 만드는 것이 아니라!

`ImageState`라는 클래스가 이 애플리케이션의 "모델"을 형성하며, 더 나은 테스트 가능성을 위해 `PApplet`을 확장하는 클래스에서 가능한 한 많은 로직을 제거합니다. 또한 이것은 더 깔끔한 설계와 관심사 분리를 만듭니다: `App`은 상호작용과 UI를 제어하지, 이미지 조작을 제어하지 않습니다.

## 직접 만드는 필터

### RGB 필터
더 복잡한 픽셀 처리 작성을 시작하기 전에, 픽셀 조작에 익숙해질 수 있는 짧은 연습부터 시작할 수 있습니다. 카메라 렌즈 위에 유색 판을 올려놓는 것과 같은 효과를 만들어내는 표준 (빨강, 초록, 파랑) 색상 필터를 만들어보겠습니다. 충분한 빨강(또는 초록, 또는 파랑)이 있는 빛만 통과시키는 것입니다.

<markdown>
이 이미지 \aosafigref{500l.imagefilters.frankfurt} (프랑크푸르트 봄 여행에서 촬영)에 다양한 필터를 적용하면 마치 계절이 다른 것 같습니다. (앞서 상상했던 사계절 그림들을 기억하시나요?) 빨강 필터가 적용되었을 때 나무가 얼마나 더 초록색이 되는지 보세요.

\aosafigure[240pt]{image-filters-images/frankfurt.jpg}{프랑크푸르트의 네 계절 (시뮬레이션)}{500l.imagefilters.frankfurt}
</markdown>
<latex>
이미지에 다양한 RGB 필터를 적용하면 어떤 색상이 필터링되고 어떤 색상이 강조되는지에 따라 마치 계절이 다른 것처럼 보이게 할 수 있습니다. (앞서 상상했던 사계절 그림들을 기억하시나요?)
</latex>

어떻게 할까요?

- 필터를 설정합니다. (앞의 이미지처럼 빨강, 초록, 파랑 필터를 결합할 수 있습니다. 이 예제들에서는 효과가 더 명확하도록 그렇게 하지 않았습니다.)

- 이미지의 각 픽셀에 대해 RGB 값을 확인합니다.

- 빨강이 빨강 필터보다 작으면, 빨강을 0으로 설정합니다.
- 초록이 초록 필터보다 작으면, 초록을 0으로 설정합니다.
- 파랑이 파랑 필터보다 작으면, 파랑을 0으로 설정합니다.
- 이 모든 색상이 충분하지 않은 픽셀은 검은색이 됩니다.

이미지는 2차원이지만, 픽셀들은 1차원 배열에 저장되며 왼쪽 위에서 시작하여 [왼쪽에서 오른쪽으로, 위에서 아래로](https://processing.org/tutorials/pixels/) 이동합니다. 4x4 이미지의 배열 인덱스는 여기에 표시되어 있습니다:

<markdown>
<table>
  <tr>
    <td>0</td>
    <td>1</td>
    <td>2</td>
    <td>3</td>
  </tr>
  <tr>
    <td>4</td>
    <td>5</td>
    <td>6</td>
    <td>7</td>
  </tr>
  <tr>
    <td>8</td>
    <td>9</td>
    <td>10</td>
    <td>11</td>
  </tr>
  <tr>
    <td>12</td>
    <td>13</td>
    <td>14</td>
    <td>15</td>
  </tr>
</table>
: \label{500l.imagefilters.pixelindices} 4x4 이미지의 픽셀 인덱스
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableOdd}
\begin{tabular}{cccc}
\hline
0 & 1 & 2 & 3 \\
4 & 5 & 6 & 7 \\
8 & 9 & 10 & 11 \\
12 & 13 & 14 & 15 \\
\hline
\end{tabular}
}
\caption{4x4 이미지의 픽셀 인덱스}
\label{500l.imagefilters.pixelindices}
\end{table}
</latex>

```java
public void applyColorFilter(PApplet applet, IFAImage img, int minRed,
      int minGreen, int minBlue, int colorRange) {  
  img.loadPixels();
  int numberOfPixels = img.getPixels().length;
  for (int i = 0; i < numberOfPixels; i++) {
    int pixel = img.getPixel(i);
    float alpha = pixelColorHelper.alpha(applet, pixel);
    float red = pixelColorHelper.red(applet, pixel);
    float green = pixelColorHelper.green(applet, pixel);
    float blue = pixelColorHelper.blue(applet, pixel);
      
    red = (red >= minRed) ? red : 0;
    green = (green >= minGreen) ? green : 0;
    blue = (blue >= minBlue) ? blue : 0;
    
    image.setPixel(i, pixelColorHelper.color(applet, red, green, blue, alpha));
  }
}
```

### 색상
이미지 필터의 첫 번째 예제에서 보았듯이, 프로그램에서 색상의 개념과 표현은 우리 필터가 어떻게 작동하는지 이해하는 데 매우 중요합니다. 다음 필터 작업을 준비하기 위해 색상 개념을 좀 더 살펴보겠습니다.

이전 섹션에서 "색 공간(color space)"이라는 개념을 사용했는데, 이는 색상을 디지털로 표현하는 방법입니다. 물감을 섞는 아이들은 색상이 다른 색상으로부터 만들어질 수 있다는 것을 배웁니다. 디지털에서는 조금 다르게 작동하지만(물감으로 덮여질 위험이 적습니다!) 비슷합니다. Processing은 원하는 어떤 색 공간이든 사용하기 매우 쉽게 만들어주지만, 어떤 것을 선택할지 알아야 하므로 그것들이 어떻게 작동하는지 이해하는 것이 중요합니다.

#### RGB 색상
대부분의 프로그래머들이 익숙한 색 공간은 RGBA입니다: 빨강, 초록, 파랑 그리고 알파. 이것이 위에서 사용한 것입니다. 16진수(16진법)에서 처음 두 자리는 빨강의 양, 두 번째 두 자리는 파랑, 세 번째 두 자리는 초록, 그리고 마지막 두 자리(있는 경우)는 알파 값입니다. 값의 범위는 16진법 00(10진법 0)부터 FF(10진법 255)까지입니다. 알파는 불투명도를 나타내며, 0은 투명하고 100%는 불투명합니다.

#### HSB 또는 HSV 색상
이 색 공간은 RGB만큼 잘 알려져 있지 않습니다. 첫 번째 숫자는 색조(hue)를, 두 번째 숫자는 채도(saturation, 색상이 얼마나 강렬한지)를, 세 번째 숫자는 밝기(brightness)를 나타냅니다. HSB 색 공간은 원뿔로 표현할 수 있습니다: 색조는 원뿔 주위의 위치이고, 채도는 중심으로부터의 거리이며, 밝기는 높이입니다(밝기 0은 검은색).

### 이미지에서 주요 색조 추출하기
이제 픽셀 조작에 익숙해졌으니, 디지털로만 할 수 있는 작업을 해보겠습니다. 디지털에서는 그렇게 균일하지 않은 방식으로 이미지를 조작할 수 있습니다.

제 사진 스트림을 보면 테마들이 나타나는 것을 볼 수 있습니다. 홍콩 항구에서 배를 타고 석양에 찍은 야간 시리즈, 북한의 회색, 발리의 무성한 녹색, 아이슬란드 겨울의 얼음 같은 흰색과 옅은 파랑색. 사진을 가져와서 장면을 지배하는 그 주요 색상을 끌어낼 수 있을까요?

이를 위해 HSB 색 공간을 사용하는 것이 합리적입니다 — 주요 색상이 무엇인지 알아낼 때 우리는 색조에 관심이 있습니다. RGB 값을 사용해서도 가능하지만 더 어렵고(세 값을 모두 비교해야 함) 어둠에 더 민감합니다. [colorMode](http://processing.org/reference/colorMode_.html)를 사용하여 HSB 색 공간으로 변경할 수 있습니다. 

이 색 공간에 정착한 후에는 RGB를 사용하는 것보다 더 간단합니다. 각 픽셀의 색조를 찾고, 어느 것이 가장 "인기가 있는지" 알아내야 합니다. 정확할 필요는 없을 것입니다 — 매우 유사한 색조들을 함께 그룹화하고 싶고, 이를 두 가지 전략으로 처리할 수 있습니다.

첫째로 돌아오는 소수를 정수로 반올림할 것입니다. 이는 각 픽셀을 어떤 "버킷"에 넣을지 결정하기 간단하게 만들어줍니다. 둘째로 색조의 범위를 변경할 수 있습니다. 위의 원뿔 표현을 다시 생각해보면, 색조를 360도(원처럼)로 생각할 수 있습니다. Processing은 기본적으로 255를 사용하는데, 이는 RGB에서 일반적인 것과 같습니다(255는 16진법으로 FF). 사용하는 범위가 높을수록 그림의 색조들이 더 구별됩니다. 더 작은 범위를 사용하면 유사한 색조들을 함께 그룹화할 수 있습니다. 360도 범위를 사용하면, 224의 색조와 225의 색조를 구별할 수 있을 가능성은 낮습니다. 차이가 매우 작기 때문입니다. 범위를 그 3분의 1인 120으로 만들면, 이 두 색조는 모두 반올림 후 75가 됩니다.

`colorMode`를 사용하여 색조의 범위를 변경할 수 있습니다. `colorMode(HSB, 120)`을 호출하면 255 범위를 사용할 때보다 색조 감지가 절반 정도 덜 정확해집니다. 또한 색조들이 120개의 "버킷"에 들어간다는 것을 알고 있으므로, 이미지를 간단히 훑어가며 픽셀의 색조를 가져와서 배열의 해당 카운트에 1을 추가할 수 있습니다. 이는 $O(n)$이 될 것인데, 여기서 $n$은 픽셀의 수이며 각각에 대해 작업이 필요하기 때문입니다.

```java
for(int px in pixels) {
  int hue = Math.round(hue(px));
  hues[hue]++;
}
```

<markdown>
마지막에 이 색조를 화면에 출력하거나 그림 옆에 표시할 수 있습니다 (\aosafigref{500l.imagefilters.hueranges}).

\aosafigure[240pt]{image-filters-images/hueranges.jpg}{사용된 범위 크기(버킷 수)에 대한 주요 색조}{500l.imagefilters.hueranges}

</markdown>

<latex>
마지막에 이 색조를 화면에 출력하거나 그림 옆에 표시할 수 있습니다.
</latex>

<markdown>
"주요" 색조를 추출한 후에는 이미지에서 그것을 보이거나 숨기도록 선택할 수 있습니다. 다양한 허용범위(수락할 주변 범위)로 주요 색조를 표시할 수 있습니다. 이 범위에 들지 않는 픽셀들은 밝기를 기반으로 값을 설정하여 회색조로 변경할 수 있습니다.
\aosafigref{500l.imagefilters.showdominant}는 240의 범위를 사용하고 다양한 허용범위로 결정된 주요 색조를 보여줍니다. 허용범위는 가장 인기 있는 색조의 양쪽으로 함께 그룹화되는 양입니다.

\aosafigure[240pt]{image-filters-images/showdominant.jpg}{주요 색조 표시}{500l.imagefilters.showdominant}
</markdown>

<latex>
"주요" 색조를 추출한 후에는 이미지에서 그것을 보이거나 숨기도록 선택할 수 있습니다. 다양한 허용범위(수락할 주변 범위)로 주요 색조를 표시할 수 있습니다. 이 범위에 들지 않는 픽셀들은 밝기를 기반으로 값을 설정하여 회색조로 변경할 수 있습니다.
또는 그 색조를 가진 픽셀의 색상을 회색조로 설정하고 다른 픽셀들은 그대로 두어 주요 색조를 숨길 수 있습니다.
</latex>

<markdown>
또는 주요 색조를 숨길 수 있습니다. \aosafigref{500l.imagefilters.hidedominant}에서는 이미지들이 나란히 배치되어 있습니다: 가운데가 원본, 왼쪽에는 주요 색조(길의 갈색)가 표시되고, 오른쪽에는 주요 색조가 숨겨져 있습니다(범위 320, 허용범위 20).

\aosafigure[240pt]{image-filters-images/hidedominant.jpg}{주요 색조 숨기기}{500l.imagefilters.hidedominant}
</markdown>

각 이미지는 이중 패스(각 픽셀을 두 번 보기)가 필요하므로, 많은 픽셀을 가진 이미지에서는 상당한 시간이 걸릴 수 있습니다.

```java
public HSBColor getDominantHue(PApplet applet, IFAImage image, int hueRange) {
  image.loadPixels();
  int numberOfPixels = image.getPixels().length;
  int[] hues = new int[hueRange];
  float[] saturations = new float[hueRange];
  float[] brightnesses = new float[hueRange];

  for (int i = 0; i < numberOfPixels; i++) {
    int pixel = image.getPixel(i);
    int hue = Math.round(pixelColorHelper.hue(applet, pixel));
    float saturation = pixelColorHelper.saturation(applet, pixel);
    float brightness = pixelColorHelper.brightness(applet, pixel);
    hues[hue]++;
    saturations[hue] += saturation;
    brightnesses[hue] += brightness;
  }

  // 가장 일반적인 색조를 찾습니다.
  int hueCount = hues[0];
  int hue = 0;
  for (int i = 1; i < hues.length; i++) {
    if (hues[i] > hueCount) {
      hueCount = hues[i];
      hue = i;
    }
  }

  // 표시할 색상을 반환합니다.
  float s = saturations[hue] / hueCount;
  float b = brightnesses[hue] / hueCount;
  return new HSBColor(hue, s, b);
}


public void processImageForHue(PApplet applet, IFAImage image, int hueRange,
    int hueTolerance, boolean showHue) {
  applet.colorMode(PApplet.HSB, (hueRange - 1));
  image.loadPixels();
  int numberOfPixels = image.getPixels().length;
  HSBColor dominantHue = getDominantHue(applet, image, hueRange);
  // 사진을 조작하여 해당 색조에 가깝지 않은 픽셀은 회색조로 만듭니다.
  float lower = dominantHue.h - hueTolerance;
  float upper = dominantHue.h + hueTolerance;
  for (int i = 0; i < numberOfPixels; i++) {
    int pixel = image.getPixel(i);
    float hue = pixelColorHelper.hue(applet, pixel);
    if (hueInRange(hue, hueRange, lower, upper) == showHue) {
      float brightness = pixelColorHelper.brightness(applet, pixel);
      image.setPixel(i, pixelColorHelper.color(applet, brightness));
    }
  }
  image.updatePixels();
}
```

### 필터 결합

현재의 UI에서는 사용자가 빨강, 초록, 파랑 필터를 함께 결합할 수 있습니다. 주요 색조 필터를 빨강, 초록, 파랑 필터와 결합하면 색 공간이 변경되기 때문에 결과가 때로는 예상과 다를 수 있습니다.

Processing에는 이미지 조작을 지원하는 몇 가지 [내장 메서드](https://www.processing.org/reference/filter_.html)가 있습니다. 예를 들어 `invert`와 `blur`입니다.

선명화, 블러링, 세피아 같은 효과를 달성하기 위해서는 매트릭스를 적용합니다. 이미지의 모든 픽셀에 대해, 각 곱이 현재 픽셀 또는 그 이웃의 색상 값과 [필터 매트릭스](http://lodev.org/cgtutor/filtering.html)의 해당 값인 곱들의 합을 취합니다. 이미지를 선명하게 하는 특정 값들의 특별한 매트릭스들이 있습니다.

## 아키텍처

앱에는 세 가지 주요 구성요소가 있습니다 (\aosafigref{500l.imagefilters.architecture}).

### 앱
앱은 하나의 파일로 구성됩니다: `ImageFilterApp.java`. 이는 `PApplet`(Processing 앱 슈퍼클래스)을 확장하며 레이아웃, 사용자 상호작용 등을 처리합니다. 이 클래스는 테스트하기 가장 어렵기 때문에 가능한 한 작게 유지하려고 합니다.

### 모델
모델은 세 개의 파일로 구성됩니다: `HSBColor.java`는 HSB 색상(색조, 채도, 밝기로 구성)에 대한 간단한 컨테이너입니다. `IFAImage`는 테스트 가능성을 위한 `PImage` 주변의 래퍼입니다. (`PImage`는 모킹할 수 없는 여러 final 메서드를 포함합니다.) 마지막으로 `ImageState.java`는 이미지의 상태 — 어떤 수준의 필터를 적용할지, 어떤 필터를 적용할지 — 를 설명하고 이미지 로딩을 처리하는 객체입니다. (주의: 색상 필터가 조정되거나 주요 색조가 재계산될 때마다 이미지를 다시 로드해야 합니다. 명확성을 위해 이미지가 처리될 때마다 다시 로드합니다.)

### 색상
색상은 두 개의 파일로 구성됩니다: `ColorHelper.java`는 모든 이미지 처리와 필터링이 일어나는 곳이고, `PixelColorHelper.java`는 테스트 가능성을 위해 픽셀 색상에 대한 final `PApplet` 메서드들을 추상화합니다.

\aosafigure[240pt]{image-filters-images/architecture.jpg}{아키텍처 다이어그램}{500l.imagefilters.architecture}

### Wrapper Classes and Tests
Briefly mentioned above, there are two wrapper classes (`IFAImage` and
`PixelColorHelper`) that wrap library methods for testability. This is because,
in Java, the keyword "final" indicates a method that cannot be overridden or hidden by
subclasses, which means they cannot be mocked.

`PixelColorHelper` wraps methods on the applet. This means we need to pass the
applet in to each method call. (Alternatively, we could make it a field and set
it on initialization.)

```java
package com.catehuston.imagefilter.color;

import processing.core.PApplet;

public class PixelColorHelper {

  public float alpha(PApplet applet, int pixel) {
    return applet.alpha(pixel);
  }

  public float blue(PApplet applet, int pixel) {
    return applet.blue(pixel);
  }

  public float brightness(PApplet applet, int pixel) {
    return applet.brightness(pixel);
  }

  public int color(PApplet applet, float greyscale) {
    return applet.color(greyscale);
  }

  public int color(PApplet applet, float red, float green, float blue,
           float alpha) {
    return applet.color(red, green, blue, alpha);
  }

  public float green(PApplet applet, int pixel) {
    return applet.green(pixel);
  }

  public float hue(PApplet applet, int pixel) {
    return applet.hue(pixel);
  }

  public float red(PApplet applet, int pixel) {
    return applet.red(pixel);
  }

  public float saturation(PApplet applet, int pixel) {
    return applet.saturation(pixel);
  }
}
```

`IFAImage` is a wrapper around `PImage`, so in our app we don’t initialize a
`PImage`, but rather an `IFAImage` — although we do have to expose the
`PImage` so that it can be rendered.

```java
package com.catehuston.imagefilter.model;

import processing.core.PApplet;
import processing.core.PImage;

public class IFAImage {

  private PImage image;

  public IFAImage() {
    image = null;
  }

  public PImage image() {
    return image;
  }

  public void update(PApplet applet, String filepath) {
    image = null;
    image = applet.loadImage(filepath);
  }

  // Wrapped methods from PImage.
  public int getHeight() {
    return image.height;
  }

  public int getPixel(int px) {
    return image.pixels[px];
  }

  public int[] getPixels() {
    return image.pixels;
  }

  public int getWidth() {
    return image.width;
  }

  public void loadPixels() {
    image.loadPixels();
  }

  public void resize(int width, int height) {
    image.resize(width, height);
  }

  public void save(String filepath) {
    image.save(filepath);
  }

  public void setPixel(int px, int color) {
    image.pixels[px] = color;
  }

  public void updatePixels() {
    image.updatePixels();
  }
}
```

Finally, we have our simple container class, `HSBColor`. Note that it is
immutable (once created, it cannot be changed). Immutable objects are better
for thread safety (something we have no need of here!) but are also easier to
understand and reason about. In general, I tend to make simple model classes
immutable unless I find a good reason for them not to be.

Some of you may know that there are already classes representing color in
[Processing](https://www.processing.org/reference/color_datatype.html) and in
[Java itself](https://docs.oracle.com/javase/7/docs/api/java/awt/Color.html).
Without going too much into the details of these, both of them are more focused
on RGB color, and the Java class in particular adds way more complexity than we
need. We would probably be okay if we did want to use Java’s `awt.Color`; however
[awt GUI components cannot be used in
Processing](http://processing.org/reference/javadoc/core/processing/core/PApplet.html),
so for our purposes creating this simple container class to hold these
bits of data we need is easiest.

```java
package com.catehuston.imagefilter.model;

public class HSBColor {

  public final float h;
  public final float s;
  public final float b;

  public HSBColor(float h, float s, float b) {
    this.h = h;
    this.s = s;
    this.b = b;
  }
}
```

### ColorHelper and Associated Tests

`ColorHelper` is where all the image manipulation lives. The methods in this
class could be static if not for needing a `PixelColorHelper`. (Although we
won’t get into the debate about the merits of static methods here.)

```java
package com.catehuston.imagefilter.color;

import processing.core.PApplet;

import com.catehuston.imagefilter.model.HSBColor;
import com.catehuston.imagefilter.model.IFAImage;

public class ColorHelper {

  private final PixelColorHelper pixelColorHelper;

  public ColorHelper(PixelColorHelper pixelColorHelper) {
    this.pixelColorHelper = pixelColorHelper;
  }

  public boolean hueInRange(float hue, int hueRange, float lower, float upper) {
    // Need to compensate for it being circular - can go around.
    if (lower < 0) {
      lower += hueRange;
    }
    if (upper > hueRange) {
      upper -= hueRange;
    }
    if (lower < upper) {
      return hue < upper && hue > lower;
    } else {
      return hue < upper || hue > lower;
    }
  }

  public HSBColor getDominantHue(PApplet applet, IFAImage image, int hueRange) {
    image.loadPixels();
    int numberOfPixels = image.getPixels().length;
    int[] hues = new int[hueRange];
    float[] saturations = new float[hueRange];
    float[] brightnesses = new float[hueRange];

    for (int i = 0; i < numberOfPixels; i++) {
      int pixel = image.getPixel(i);
      int hue = Math.round(pixelColorHelper.hue(applet, pixel));
      float saturation = pixelColorHelper.saturation(applet, pixel);
      float brightness = pixelColorHelper.brightness(applet, pixel);
      hues[hue]++;
      saturations[hue] += saturation;
      brightnesses[hue] += brightness;
    }

    // Find the most common hue.
    int hueCount = hues[0];
    int hue = 0;
    for (int i = 1; i < hues.length; i++) {
      if (hues[i] > hueCount) {
        hueCount = hues[i];
        hue = i;
      }
    }

    // Return the color to display.
    float s = saturations[hue] / hueCount;
    float b = brightnesses[hue] / hueCount;
    return new HSBColor(hue, s, b);
  }

  public void processImageForHue(PApplet applet, IFAImage image, int hueRange,
      int hueTolerance, boolean showHue) {
    applet.colorMode(PApplet.HSB, (hueRange - 1));
    image.loadPixels();
    int numberOfPixels = image.getPixels().length;
    HSBColor dominantHue = getDominantHue(applet, image, hueRange);
    // Manipulate photo, grayscale any pixel that isn't close to that hue.
    float lower = dominantHue.h - hueTolerance;
    float upper = dominantHue.h + hueTolerance;
    for (int i = 0; i < numberOfPixels; i++) {
      int pixel = image.getPixel(i);
      float hue = pixelColorHelper.hue(applet, pixel);
      if (hueInRange(hue, hueRange, lower, upper) == showHue) {
        float brightness = pixelColorHelper.brightness(applet, pixel);
        image.setPixel(i, pixelColorHelper.color(applet, brightness));
      }
    }
    image.updatePixels();
  }

  public void applyColorFilter(PApplet applet, IFAImage image, int minRed,
      int minGreen, int minBlue, int colorRange) {
    applet.colorMode(PApplet.RGB, colorRange);
    image.loadPixels();
    int numberOfPixels = image.getPixels().length;
    for (int i = 0; i < numberOfPixels; i++) {
      int pixel = image.getPixel(i);
      float alpha = pixelColorHelper.alpha(applet, pixel);
      float red = pixelColorHelper.red(applet, pixel);
      float green = pixelColorHelper.green(applet, pixel);
      float blue = pixelColorHelper.blue(applet, pixel);

      red = (red >= minRed) ? red : 0;
      green = (green >= minGreen) ? green : 0;
      blue = (blue >= minBlue) ? blue : 0;

      image.setPixel(i, pixelColorHelper.color(applet, red, green, blue, alpha));
    }
  }
}
```

We don't want to test this with whole images, because we want images that we
know the properties of and reason about. We approximate this by mocking the
images and making them return an array of pixels — in this case, 5. This
allows us to
verify that the behavior is as expected. Earlier we covered the concept of mock
objects, and here we see their use. We are using
[Mockito](http://docs.mockito.googlecode.com/hg/org/mockito/Mockito.html) as
our mock object framework.

To create a mock we use the `@Mock` annotation on an instance variable, and it will be mocked at runtime by the
`MockitoJUnitRunner`.

To stub (set the behavior of) a method, we use: 

```java
    when(mock.methodCall()).thenReturn(value)
```

To verify a method was called, we use `verify(mock.methodCall())`.

We'll show a few example test cases here; if you'd like to see the rest, visit
the source folder for this project in the [_500 Lines or Less_ GitHub
repository](https://github.com/aosabook/500lines/tree/master/image-filters).

```java
package com.catehuston.imagefilter.color;

/* ... Imports omitted ... */

@RunWith(MockitoJUnitRunner.class)
public class ColorHelperTest {

  @Mock PApplet applet;
  @Mock IFAImage image;
  @Mock PixelColorHelper pixelColorHelper;

  ColorHelper colorHelper;

  private static final int px1 = 1000;
  private static final int px2 = 1010;
  private static final int px3 = 1030;
  private static final int px4 = 1040;
  private static final int px5 = 1050;
  private static final int[] pixels = { px1, px2, px3, px4, px5 };

  @Before public void setUp() throws Exception {
    colorHelper = new ColorHelper(pixelColorHelper);
    when(image.getPixels()).thenReturn(pixels);
    setHsbValuesForPixel(0, px1, 30F, 5F, 10F);
    setHsbValuesForPixel(1, px2, 20F, 6F, 11F);
    setHsbValuesForPixel(2, px3, 30F, 7F, 12F);
    setHsbValuesForPixel(3, px4, 50F, 8F, 13F);
    setHsbValuesForPixel(4, px5, 30F, 9F, 14F);
  }

  private void setHsbValuesForPixel(int px, int color, float h, float s, float b) {
    when(image.getPixel(px)).thenReturn(color);
    when(pixelColorHelper.hue(applet, color)).thenReturn(h);
    when(pixelColorHelper.saturation(applet, color)).thenReturn(s);
    when(pixelColorHelper.brightness(applet, color)).thenReturn(b);
  }

  private void setRgbValuesForPixel(int px, int color, float r, float g, float b, 
            float alpha) {
    when(image.getPixel(px)).thenReturn(color);
    when(pixelColorHelper.red(applet, color)).thenReturn(r);
    when(pixelColorHelper.green(applet, color)).thenReturn(g);
    when(pixelColorHelper.blue(applet, color)).thenReturn(b);
    when(pixelColorHelper.alpha(applet, color)).thenReturn(alpha);
  }

    @Test public void testHsbColorFromImage() {
    HSBColor color = colorHelper.getDominantHue(applet, image, 100);
    verify(image).loadPixels();

    assertEquals(30F, color.h, 0);
    assertEquals(7F, color.s, 0);
    assertEquals(12F, color.b, 0);
  }

  @Test public void testProcessImageNoHue() {
    when(pixelColorHelper.color(applet, 11F)).thenReturn(11);
    when(pixelColorHelper.color(applet, 13F)).thenReturn(13);
    colorHelper.processImageForHue(applet, image, 60, 2, false);
    verify(applet).colorMode(PApplet.HSB, 59);
    verify(image, times(2)).loadPixels();
    verify(image).setPixel(1, 11);
    verify(image).setPixel(3, 13);
  }

  @Test public void testApplyColorFilter() {
    setRgbValuesForPixel(0, px1, 10F, 12F, 14F, 60F);
    setRgbValuesForPixel(1, px2, 20F, 22F, 24F, 70F);
    setRgbValuesForPixel(2, px3, 30F, 32F, 34F, 80F);
    setRgbValuesForPixel(3, px4, 40F, 42F, 44F, 90F);
    setRgbValuesForPixel(4, px5, 50F, 52F, 54F, 100F);

    when(pixelColorHelper.color(applet, 0F, 0F, 0F, 60F)).thenReturn(5);
    when(pixelColorHelper.color(applet, 20F, 0F, 0F, 70F)).thenReturn(15);
    when(pixelColorHelper.color(applet, 30F, 32F, 0F, 80F)).thenReturn(25);
    when(pixelColorHelper.color(applet, 40F, 42F, 44F, 90F)).thenReturn(35);
    when(pixelColorHelper.color(applet, 50F, 52F, 54F, 100F)).thenReturn(45);

    colorHelper.applyColorFilter(applet, image, 15, 25, 35, 100);
    verify(applet).colorMode(PApplet.RGB, 100);
    verify(image).loadPixels();

    verify(image).setPixel(0, 5);
    verify(image).setPixel(1, 15);
    verify(image).setPixel(2, 25);
    verify(image).setPixel(3, 35);
    verify(image).setPixel(4, 45);
  }
}
```

\newpage

Notice that:

- We use the `MockitoJUnit` runner.
- We mock `PApplet`, `IFAImage` (created for expressly this purpose), and `ImageColorHelper`.
- Test methods are annotated with `@Test`[^habits]. If you want to ignore a test (e.g., whilst debugging) you can add the annotation `@Ignore`.
- In `setup()`, we create the pixel array and have the mock image always return it.
- Helper methods make it easier to set expectations for recurring tasks (e.g., `set*ForPixel()`).

[^habits]: Method names in tests need not start with `test` as of JUnit 4, but habits are hard to break.

### Image State and Associated Tests
`ImageState` holds the current "state" of the image — the image itself, and the
settings and filters that will be applied. We'll omit the full implementation
of `ImageState` here, but we'll show how it can be tested. You can visit the source
repository for this project to see the full details.

```java
package com.catehuston.imagefilter.model;

import processing.core.PApplet;
import com.catehuston.imagefilter.color.ColorHelper;

public class ImageState {

  enum ColorMode {
    COLOR_FILTER,
    SHOW_DOMINANT_HUE,
    HIDE_DOMINANT_HUE
  }

  private final ColorHelper colorHelper;
  private IFAImage image;
  private String filepath;

  public static final int INITIAL_HUE_TOLERANCE = 5;

  ColorMode colorModeState = ColorMode.COLOR_FILTER;
  int blueFilter = 0;
  int greenFilter = 0;
  int hueTolerance = 0;
  int redFilter = 0;

  public ImageState(ColorHelper colorHelper) {
    this.colorHelper = colorHelper;
    image = new IFAImage();
    hueTolerance = INITIAL_HUE_TOLERANCE;
  }
  /* ... getters & setters */
  public void updateImage(PApplet applet, int hueRange, int rgbColorRange, 
          int imageMax) { ... }

  public void processKeyPress(char key, int inc, int rgbColorRange,
          int hueIncrement, int hueRange) { ... }

  public void setUpImage(PApplet applet, int imageMax) { ... }

  public void resetImage(PApplet applet, int imageMax) { ... }

  // For testing purposes only.
  protected void set(IFAImage image, ColorMode colorModeState,
            int redFilter, int greenFilter, int blueFilter, int hueTolerance) { ... }
}
```

Here we can test that the appropriate actions happen for the given state; that
fields are incremented and decremented appropriately.

```java
package com.catehuston.imagefilter.model;

/* ... Imports omitted ... */

@RunWith(MockitoJUnitRunner.class)
public class ImageStateTest {

  @Mock PApplet applet;
  @Mock ColorHelper colorHelper;
  @Mock IFAImage image;

  private ImageState imageState;

  @Before public void setUp() throws Exception {
    imageState = new ImageState(colorHelper);
  }

  private void assertState(ColorMode colorMode, int redFilter,
      int greenFilter, int blueFilter, int hueTolerance) {
    assertEquals(colorMode, imageState.getColorMode());
    assertEquals(redFilter, imageState.redFilter());
    assertEquals(greenFilter, imageState.greenFilter());
    assertEquals(blueFilter, imageState.blueFilter());
    assertEquals(hueTolerance, imageState.hueTolerance());
  }

  @Test public void testUpdateImageDominantHueHidden() {
    imageState.setFilepath("filepath");
    imageState.set(image, ColorMode.HIDE_DOMINANT_HUE, 5, 10, 15, 10);

    imageState.updateImage(applet, 100, 100, 500);

    verify(image).update(applet, "filepath");
    verify(colorHelper).processImageForHue(applet, image, 100, 10, false);
    verify(colorHelper).applyColorFilter(applet, image, 5, 10, 15, 100);
    verify(image).updatePixels();
  }

  @Test public void testUpdateDominantHueShowing() {
    imageState.setFilepath("filepath");
    imageState.set(image, ColorMode.SHOW_DOMINANT_HUE, 5, 10, 15, 10);

    imageState.updateImage(applet, 100, 100, 500);

    verify(image).update(applet, "filepath");
    verify(colorHelper).processImageForHue(applet, image, 100, 10, true);
    verify(colorHelper).applyColorFilter(applet, image, 5, 10, 15, 100);
    verify(image).updatePixels();
  }

  @Test public void testUpdateRGBOnly() {
    imageState.setFilepath("filepath");
    imageState.set(image, ColorMode.COLOR_FILTER, 5, 10, 15, 10);

    imageState.updateImage(applet, 100, 100, 500);

    verify(image).update(applet, "filepath");
    verify(colorHelper, never()).processImageForHue(any(PApplet.class), 
                any(IFAImage.class), anyInt(), anyInt(), anyBoolean());
    verify(colorHelper).applyColorFilter(applet, image, 5, 10, 15, 100);
    verify(image).updatePixels();
  }

  @Test public void testKeyPress() {
    imageState.processKeyPress('r', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 5, 0, 0, 5);

    imageState.processKeyPress('e', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);

    imageState.processKeyPress('g', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 5, 0, 5);

    imageState.processKeyPress('f', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);

    imageState.processKeyPress('b', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 5, 5);

    imageState.processKeyPress('v', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);

    imageState.processKeyPress('h', 5, 100, 2, 200);
    assertState(ColorMode.HIDE_DOMINANT_HUE, 0, 0, 0, 5);

    imageState.processKeyPress('i', 5, 100, 2, 200);
    assertState(ColorMode.HIDE_DOMINANT_HUE, 0, 0, 0, 7);

    imageState.processKeyPress('u', 5, 100, 2, 200);
    assertState(ColorMode.HIDE_DOMINANT_HUE, 0, 0, 0, 5);

    imageState.processKeyPress('h', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);

    imageState.processKeyPress('s', 5, 100, 2, 200);
    assertState(ColorMode.SHOW_DOMINANT_HUE, 0, 0, 0, 5);

    imageState.processKeyPress('s', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);

    // Random key should do nothing.
    imageState.processKeyPress('z', 5, 100, 2, 200);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);
  }

  @Test public void testSave() {
    imageState.set(image, ColorMode.SHOW_DOMINANT_HUE, 5, 10, 15, 10);
    imageState.setFilepath("filepath");
    imageState.processKeyPress('w', 5, 100, 2, 200);

    verify(image).save("filepath-new.png");
  }

  @Test public void testSetupImageLandscape() {
    imageState.set(image, ColorMode.SHOW_DOMINANT_HUE, 5, 10, 15, 10);
    when(image.getWidth()).thenReturn(20);
    when(image.getHeight()).thenReturn(8);
    imageState.setUpImage(applet, 10);
    verify(image).update(applet, null);
    verify(image).resize(10, 4);
  }

  @Test public void testSetupImagePortrait() {
    imageState.set(image, ColorMode.SHOW_DOMINANT_HUE, 5, 10, 15, 10);
    when(image.getWidth()).thenReturn(8);
    when(image.getHeight()).thenReturn(20);
    imageState.setUpImage(applet, 10);
    verify(image).update(applet, null);
    verify(image).resize(4, 10);
  }

  @Test public void testResetImage() {
    imageState.set(image, ColorMode.SHOW_DOMINANT_HUE, 5, 10, 15, 10);
    imageState.resetImage(applet, 10);
    assertState(ColorMode.COLOR_FILTER, 0, 0, 0, 5);
  }
}
```

\newpage Notice that:

- We exposed a protected initialization method `set` for testing that helps us quickly get the system under test into a specific state.
- We mock `PApplet`, `ColorHelper`, and `IFAImage` (created expressly for this purpose).
- This time we use a helper (`assertState()`) to simplify asserting the state of the image.

#### Measuring test coverage
I use [EclEmma](http://www.eclemma.org/installation.html#marketplace) to
measure test coverage within Eclipse. Overall for the app we have 81% test
coverage, with none of `ImageFilterApp` covered, 94.8% for `ImageState`, and
100% for `ColorHelper`.

### ImageFilterApp
This is where everything is tied together, but we want as little as possible
here. The App is hard to unit test (much of it is layout), but because we've pushed so much of the app's functionality into our own tested classes, we're able to assure ourselves that the important parts are working as intended.  

We set the size of the app, and do the layout. (These things are verified by
running the app and making sure it looks okay — no matter how good the test coverage,
this step should not be skipped!)

```java
package com.catehuston.imagefilter.app;

import java.io.File;

import processing.core.PApplet;

import com.catehuston.imagefilter.color.ColorHelper;
import com.catehuston.imagefilter.color.PixelColorHelper;
import com.catehuston.imagefilter.model.ImageState;

@SuppressWarnings("serial")
public class ImageFilterApp extends PApplet {

  static final String INSTRUCTIONS = "...";

  static final int FILTER_HEIGHT = 2;
  static final int FILTER_INCREMENT = 5;
  static final int HUE_INCREMENT = 2;
  static final int HUE_RANGE = 100;
  static final int IMAGE_MAX = 640;
  static final int RGB_COLOR_RANGE = 100;
  static final int SIDE_BAR_PADDING = 10;
  static final int SIDE_BAR_WIDTH = RGB_COLOR_RANGE + 2 * SIDE_BAR_PADDING + 50;

  private ImageState imageState;

  boolean redrawImage = true;

  @Override
  public void setup() {
    noLoop();
    imageState = new ImageState(new ColorHelper(new PixelColorHelper()));

    // Set up the view.
    size(IMAGE_MAX + SIDE_BAR_WIDTH, IMAGE_MAX);
    background(0);

    chooseFile();
  }

  @Override
  public void draw() {
    // Draw image.
    if (imageState.image().image() != null && redrawImage) {
      background(0);
      drawImage();
    }

    colorMode(RGB, RGB_COLOR_RANGE);
    fill(0);
    rect(IMAGE_MAX, 0, SIDE_BAR_WIDTH, IMAGE_MAX);
    stroke(RGB_COLOR_RANGE);
    line(IMAGE_MAX, 0, IMAGE_MAX, IMAGE_MAX);

    // Draw red line
    int x = IMAGE_MAX + SIDE_BAR_PADDING;
    int y = 2 * SIDE_BAR_PADDING;
    stroke(RGB_COLOR_RANGE, 0, 0);
    line(x, y, x + RGB_COLOR_RANGE, y);
    line(x + imageState.redFilter(), y - FILTER_HEIGHT,
        x + imageState.redFilter(), y + FILTER_HEIGHT);

    // Draw green line
    y += 2 * SIDE_BAR_PADDING;
    stroke(0, RGB_COLOR_RANGE, 0);
    line(x, y, x + RGB_COLOR_RANGE, y);
    line(x + imageState.greenFilter(), y - FILTER_HEIGHT,
        x + imageState.greenFilter(), y + FILTER_HEIGHT);

    // Draw blue line
    y += 2 * SIDE_BAR_PADDING;
    stroke(0, 0, RGB_COLOR_RANGE);
    line(x, y, x + RGB_COLOR_RANGE, y);
    line(x + imageState.blueFilter(), y - FILTER_HEIGHT,
        x + imageState.blueFilter(), y + FILTER_HEIGHT);

    // Draw white line.
    y += 2 * SIDE_BAR_PADDING;
    stroke(HUE_RANGE);
    line(x, y, x + 100, y);
    line(x + imageState.hueTolerance(), y - FILTER_HEIGHT,
        x + imageState.hueTolerance(), y + FILTER_HEIGHT);

    y += 4 * SIDE_BAR_PADDING;
    fill(RGB_COLOR_RANGE);
    text(INSTRUCTIONS, x, y);
    updatePixels();
  }

  // Callback for selectInput(), has to be public to be found.
  public void fileSelected(File file) {
    if (file == null) {
      println("User hit cancel.");
    } else {
      imageState.setFilepath(file.getAbsolutePath());
      imageState.setUpImage(this, IMAGE_MAX);
      redrawImage = true;
      redraw();
    }
  }

  private void drawImage() {
    imageMode(CENTER);
    imageState.updateImage(this, HUE_RANGE, RGB_COLOR_RANGE, IMAGE_MAX);
    image(imageState.image().image(), IMAGE_MAX/2, IMAGE_MAX/2, 
                imageState.image().getWidth(), imageState.image().getHeight());
    redrawImage = false;
  }

  @Override
  public void keyPressed() {
    switch(key) {
    case 'c':
      chooseFile();
      break;
    case 'p':
      redrawImage = true;
      break;
    case ' ':
      imageState.resetImage(this, IMAGE_MAX);
      redrawImage = true;
      break;
    }
    imageState.processKeyPress(key, FILTER_INCREMENT, RGB_COLOR_RANGE, 
                HUE_INCREMENT, HUE_RANGE);
    redraw();
  }

  private void chooseFile() {
    // Choose the file.
    selectInput("Select a file to process:", "fileSelected");
  }
}
```

Notice that:

- Our implementation extends `PApplet`.
- Most work is done in `ImageState`.
- `fileSelected()` is the callback for `selectInput()`.
- `static final` constants are defined up at the top.

## The Value of Prototyping
In real world programming, we spend a lot of time on productionisation work.
Making things look just so. Maintaining 99.9%
uptime. We spend more time on corner cases than refining algorithms.

These constraints and requirements are important for our users. However there’s
also space for freeing ourselves from them to play and explore.

Eventually, I decided to port this to a native mobile app. Processing has an
Android library, but as many mobile developers do, I opted to go iOS first. I
had years of iOS experience, although I’d done little with CoreGraphics, but I
don’t think even if I had had this idea initially, I would have been able to
build it straight away on iOS. The platform forced me to operate in the RGB
color space, and made it hard to extract the pixels from the image (hello, C).
Memory and waiting was a major risk. 

There were exhilarating moments,
when it worked for the first time. When it first ran on my device... without
crashing. When I optimized memory usage by 66% and cut seconds off the runtime.
And there were large periods of time locked away in a dark room, cursing
intermittently.

Because I had my prototype, I could explain to my business partner and our
designer what I was thinking and what the app would do. It meant I deeply
understood how it would work, and it was just a question of making it work
nicely on this other platform. I knew what I was aiming for, so at the end of a
long day shut away fighting with it and feeling like I had little to show for
it I kept going… and hit an exhilarating moment and milestone the following
morning.

So, how do you find the dominant color in an image? There’s an app for
that: [Show & Hide](http://showandhide.com).
