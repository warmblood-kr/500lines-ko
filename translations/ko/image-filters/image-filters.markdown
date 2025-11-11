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
    <td>Loads the pixel data for the image into its `pixels[]` array</td>
  </tr>
  <tr>
    <td>`updatePixels`</td>
    <td>Updates the image with the data in its `pixels[]` array</td>
  </tr>
  <tr>
    <td>`resize`</td>
    <td>Changes the size of an image to a new width and height</td>
  </tr>
  <tr>
    <td>`get`</td>
    <td>Reads the color of any pixel or grabs a rectangle of pixels</td>
  </tr>
  <tr>
    <td>`set`</td>
    <td>Writes a color to any pixel or writes an image into another</td>
  </tr>
  <tr>
    <td>`save`</td>
    <td>Saves the image to a TIFF, TARGA, PNG, or JPEG file</td>
  </tr>
</table>
: \label{500l.imagefilters.pimagemethods} PImage methods
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
loadPixels & Loads the pixel data for the image into its `pixels[]` array \\
updatePixels & Updates the image with the data in its `pixels[]` array \\
resize & Changes the size of an image to a new width and height \\
get & Reads the color of any pixel or grabs a rectangle of pixels \\
set & Writes a color to any pixel or writes an image into another \\
save & Saves the image to a TIFF, TARGA, PNG, or JPEG file \\
\hline
\end{tabular}
}
\caption{PImage methods}
\label{500l.imagefilters.pimagemethods}
\end{table}
</latex>

### File Chooser
Processing handles most of the file choosing process; we just need to call
[`selectInput()`](http://www.processing.org/reference/selectInput_.html), and
implement a callback (which must be public). 

To people familiar with Java this might seem odd; a listener or a lambda
expression might make more sense. However, as Processing was developed as a tool
for artists, for the most part these things have been
abstracted away by the language to keep it unintimidating. This is a choice the
designers made: to prioritize simplicity and approachability over power
and flexibility. If you use the stripped-down Processing editor, rather than
Processing as a library in Eclipse, you don’t even need to define class names. 

Other language designers with different target audiences make different
choices, as they should. For example, in Haskell, a purely
functional language, purity of functional language paradigms is
prioritised over everything else. This makes it a better tool for mathematical
problems than for anything requiring IO.

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

### Responding to Key Presses

Normally in Java, responding to key presses requires adding listeners and implementing
anonymous functions. However, as with the file chooser, Processing handles a lot of
this for us. We just need to implement
[`keyPressed()`](https://www.processing.org/reference/keyPressed_.html).

```java
public void keyPressed() {
  print(“key pressed: ” + key);
}
```

If you run the app again, every time you press a key it will output it to the
console. Later, you’ll want to do different things depending on what key was
pressed, and to do this you just switch on the key value. (This exists in the
`PApplet` superclass, and contains the last key pressed.) 


## Writing Tests 

This app doesn’t do a lot yet, but we can already see number of places where
things can go wrong; for example, triggering the wrong action with key presses.
As we add complexity, we add more potential problems, such as updating the
image state incorrectly, or miscalculating pixel colors after applying
a filter. I also just enjoy (some think weirdly) writing unit tests. Whilst
some people seem to think of testing as a thing that delays checking code in, I
see tests as my #1 debugging tool, and as an opportunity to deeply understand what
is going on in my code.

I adore Processing, but it’s designed to
create visual applications, and in this area maybe unit testing isn’t a huge
concern. It’s clear it isn’t written for testability; in fact it’s written in
such a way that makes it untestable, as is. Part of this is because it hides
complexity, and some of that hidden complexity is really useful in writing unit
tests. The use of static and final methods make it much harder to use mocks
(objects that record interaction and allow you to fake part of your system to
verify another part is behaving correctly), which rely on the ability to
subclass. 

We might start a greenfield project with great intentions to do Test Driven
Development (TDD) and achieve perfect test coverage, but in reality we are
usually looking at a mass of code written by various and assorted people and
trying to figure out what it is supposed to be doing, and how and why it is
going wrong. Then maybe we don’t write perfect tests, but writing tests at all
will help us navigate the situation, document what is happening and move
forward.

We create "seams" that allow us to break something up from its
amorphous mass of tangled pieces and verify it in parts. To do this, we will sometimes
create wrapper classes that can be mocked. These classes do nothing more than hold a
collection of similar methods, or forward calls on to another object that 
cannot be mocked (due to final or static methods), and as such they are 
very dull
to write, but key to creating seams and making the code testable.

I used JUnit for tests, as I was working in Java with Processing as a library.
For mocking I used Mockito. You can download
[Mockito](https://code.google.com/p/mockito/downloads/list) and add the JAR to
your buildpath in the same way you added `core.jar`. I created two helper
classes that make it possible to mock and test the app (otherwise we can’t test
behavior involving `PImage` or `PApplet` methods).

`IFAImage` is a thin wrapper around PImage. `PixelColorHelper` is a wrapper
around applet pixel color methods. These wrappers call the final, and static
methods, but the caller methods are neither final nor static themselves — this
allows them to be mocked. These are deliberately lightweight, and we could have
gone further, but this was sufficient to address the major problem of
testability when using Processing — static, and final methods. The goal 
was to make an app, after all — not a unit testing framework for Processing!

A class called `ImageState` forms the "model" of this application, removing as
much logic from the class extending `PApplet` as possible, for better
testability. It also makes for a cleaner design and separation of concerns:
the `App` controls the interactions and the UI, not the image
manipulation.

## Do-It-Yourself Filters

### RGB Filters
Before we start writing more complicated pixel processing, we can start with a
short exercise that will get us comfortable doing pixel manipulation. We’ll
create standard (red, green, blue) color filters that will allow us to create
the same effect as placing a colored plate over the lens of a camera, only letting
through light with enough red (or green, or blue).

<markdown>
By applying different filters to this image
\aosafigref{500l.imagefilters.frankfurt} (taken on a spring trip to Frankfurt)
it’s almost like the seasons are different. (Remember the four-seasons
paintings we imagined earlier?)  See how much more green the tree becomes when
the red filter is applied.

\aosafigure[240pt]{image-filters-images/frankfurt.jpg}{Four (Simulated) Seasons in Frankfurt}{500l.imagefilters.frankfurt}
</markdown>
<latex>
By applying different RGB filters to an image we can make it almost seem like
the seasons are different depending which colors are filtered out 
and which are emphasized. (Remember the four-seasons paintings
we imagined earlier?) 
</latex>

How do we do it? 

- Set the filter. (You can combine red, green and blue filters as in the image
  earlier; I haven’t in these examples so that the effect is clearer.)

- For each pixel in the image, check its RGB value.

- If the red is less than the red filter, set the red to zero.
- If the green is less than the green filter, set the green to zero.
- If the blue is less than the blue filter, set the blue to zero.
- Any pixel with insufficient of all of these colors will be black.

Although our image is 2-dimensional, the pixels live in a 1-dimensional array
starting top-left and moving [left to right, top to
bottom](https://processing.org/tutorials/pixels/). The array indices for a 4x4
image are shown here:

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
: \label{500l.imagefilters.pixelindices} Pixel indices for a 4x4 image
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
\caption{Pixel indices for a 4x4 image}
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

### Color
As our first example of an image filter showed, the concept and representation
of colors in a program is very important to understanding how our filters work.
To prepare ourselves for working on our next filter, let's explore the concept
of color a bit more.

We were using a concept
in the previous section called "color space", which is way of representing
color digitally. Kids mixing paints learn that colors can be made from
other colors; things work slightly differently in digital (less risk of
being covered in paint!) but similar. Processing makes it really easy to work
with whatever color space you want, but you need to know which one to pick, so
it’s important to understand how they work.

#### RGB colors
The color space that most programmers are familiar with is RGBA: red, green,
blue and alpha; it's what we were using above. In hexadecimal (base 16), the first two digits are the amount of
red, the second two blue, the third two green, and the final two (if they are
there) are the alpha value. The values range from 00 in base 16 (0 in base
10) through to FF (255 in base 10). The alpha represents 
opacity, where 0 is transparent and 100% is opaque.

#### HSB or HSV colors
This color space is not quite as well known as RGB. The first number represents
the hue, the second number the saturation (how intense the color is), and the third
number the brightness. The HSB color space can be represented by a cone: The hue
is the position around the cone, saturation the distance from the centre, and
brightness the height (0 brightness is black).

### Extracting the Dominant Hue from an Image
Now that we’re comfortable with pixel manipulation, let’s do something that we could
only do digitally. Digitally, we can manipulate the image in a way that isn’t so
uniform.

When I look through my stream of pictures I can see themes
emerging. The nighttime series I took at sunset from a boat on Hong Kong
harbour, the grey of North Korea, the lush greens of Bali, the icy whites and
pale blues of an Icelandic winter. Can we take a picture and pull out that main
color that dominates the scene?

It makes sense to use the HSB color space for this — we are interested in the
hue when figuring out what the main color
is. It’s possible to do this using RGB values, but more difficult (we would
have to compare all three values) and it would be more sensitive to darkness.
We can change to the HSB color space using
[colorMode](http://processing.org/reference/colorMode_.html). 

Having settled on this color space, it’s simpler than it would have been using
RGB. We need to find the hue of each pixel, and figure out which is most
"popular". We probably don’t want to be exact — we want to group very similar
hues together, and we can handle this using two strategies.

Firstly we will round the decimals that come back to whole numbers, as this
makes it simple to determine which "bucket" we put each pixel in. Secondly we
can change the range of the hues. If we think back to the cone representation above, we
might think of hues as having 360 degrees (like a circle). Processing uses 255
by default, which is the same as is typical for RGB (255 is FF in hexadecimal).
The higher the range we use, the more distinct the hues in the picture will be.
Using a smaller range will allow us to group together similar hues. Using a 360
degree range, it’s unlikely that we will be able to tell the difference between
a hue of 224 and a hue of 225, as the difference is very small. If we make the
range one-third of that, 120, both these hues become 75 after rounding.

We can change the range of hues using `colorMode`. If we call `colorMode(HSB, 120)`
we have just made our hue detection a bit less than half as exact as if we used
the 255 range. We also know that our hues will fall into 120 "buckets", so we
can simply go through our image, get the hue for a pixel, and add one to the
corresponding count in an array. This will be $O(n)$, where $n$ is the
number of pixels, as it requires action on each one.

```java
for(int px in pixels) {
  int hue = Math.round(hue(px));
  hues[hue]++;
}
```

<markdown>
At the end we can print this hue to the screen, or display it next to the
picture (\aosafigref{500l.imagefilters.hueranges}). 

\aosafigure[240pt]{image-filters-images/hueranges.jpg}{Dominant hue versus size of range (number of buckets) used}{500l.imagefilters.hueranges}

</markdown>

<latex>
At the end we can print this hue to the screen, or display it next to the
picture. 
</latex>

<markdown>
Once we’ve extracted the "dominant" hue, we can choose to either show or hide
it in the image. We can show the dominant hue with varying tolerance (ranges
around it that we will accept). Pixels that don’t fall into this range can be
changed to grayscale by setting the value based on the brightness.
\aosafigref{500l.imagefilters.showdominant} shows the dominant hue determined
using a range of 240, and with varying tolerance. The tolerance is the
amount either side of the most popular hue that gets grouped together. 

\aosafigure[240pt]{image-filters-images/showdominant.jpg}{Showing dominant hue}{500l.imagefilters.showdominant}
</markdown>

<latex>
Once we’ve extracted the "dominant" hue, we can choose to either show or hide
it in the image. We can show the dominant hue with varying tolerance (ranges
around it that we will accept). Pixels that don’t fall into this range can be
changed to grayscale by setting the value based on the brightness.
Alternatively, we can hide the dominant hue by setting the color for pixels with that hue to greyscale, and leaving other pixels as they are. 
</latex>

<markdown>
Alternatively, we can hide the dominant hue. In
\aosafigref{500l.imagefilters.hidedominant}, the images are transposed side by
side: the original in the middle, on the left the dominant hue (the brownish color of the path) is shown, and on
the right the dominant hue is hidden (range 320, tolerance 20).

\aosafigure[240pt]{image-filters-images/hidedominant.jpg}{Hiding dominant hue}{500l.imagefilters.hidedominant}
</markdown>

Each image requires a double pass (looking at each pixel twice), so on images
with a large number of pixels it can take a noticeable amount of time.

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
```

### Combining Filters

With the UI as it is, the user can combine the red, green, and blue filters
together. If they combine the dominant hue filters with the red, green, and
blue filters the results can sometimes be a little unexpected, because of
changing the color spaces.

Processing has some [built-in
methods](https://www.processing.org/reference/filter_.html) that support the
manipulation of images; for example, `invert` and `blur`.

To achieve effects like sharpening, blurring, or sepia we apply
matrices. For every pixel of the image, take the sum of products where each
product is the color value of the current pixel or a neighbor of it, with the
corresponding value of the [filter
matrix](http://lodev.org/cgtutor/filtering.html). There are some special
matrices of specific values that sharpen images.

## Architecture

There are three main components to the app (\aosafigref{500l.imagefilters.architecture}).

### The App
The app consists of one file: `ImageFilterApp.java`. This 
extends `PApplet` (the
Processing app superclass) and handles layout, user interaction, etc. This class
is the hardest to test, so we want to keep it as small as possible.

### Model
Model consists of three files: `HSBColor.java` is a simple container for HSB
colors (consisting of hue, saturation, and brightness). `IFAImage` is a
wrapper around `PImage` for testability. (`PImage` contains a number of final
methods which cannot be mocked.) Finally, `ImageState.java` is the object
which describes the state of the image — what level of filters should be applied,
and which filters — and handles loading the image. (Note: The image needs to be reloaded
whenever color filters are adjusted down, and whenever the dominant hue is
recalculated. For clarity, we just reload each time the image is processed.)

### Color
Color consists of two files: `ColorHelper.java` is where all the image
processing and filtering takes place, and `PixelColorHelper.java` 
abstracts out final `PApplet` methods for pixel colors for testability.

\aosafigure[240pt]{image-filters-images/architecture.jpg}{Architecture diagram}{500l.imagefilters.architecture}

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
