title: 웹 스프레드시트
author: Audrey Tang
<markdown>
_독학으로 프로그래밍을 배운 프로그래머이자 번역가인 Audrey는 Apple과 독립 계약자로 클라우드 서비스 현지화 및 자연어 기술 분야에서 일하고 있습니다. 이전에 최초로 작동하는 Perl 6 구현을 설계하고 주도했으며, Haskell, Perl 5, Perl 6의 컴퓨터 언어 설계 위원회에서 활동했습니다. 현재는 g0v의 전임 기여자이며 대만 최초의 전자 규칙 제정 프로젝트를 이끌고 있습니다._
</markdown>
이 챕터에서는 웹 브라우저에서 기본적으로 지원하는 세 가지 언어(HTML, JavaScript, CSS)로 작성된 99줄의 웹 스프레드시트를 소개합니다.

이 프로젝트의 ES5 버전은 [jsFiddle](http://jsfiddle.net/audreyt/LtDyP/)에서 사용할 수 있습니다.

<markdown>
_(이 챕터는 [번체 중국어](https://github.com/aosabook/500lines/blob/master/spreadsheet/spreadsheet.zh-tw.markdown)로도 제공됩니다)_.
</markdown>

## 소개

Tim Berners-Lee가 1990년에 웹을 발명했을 때, _웹 페이지_는 꺾쇠괄호로 둘러싸인 _태그_로 텍스트를 마크업하여 콘텐츠에 논리적 구조를 부여하는 HTML로 작성되었습니다. `<a>…</a>` 안에 마크업된 텍스트는 사용자를 웹의 다른 페이지로 안내하는 _하이퍼링크_가 되었습니다.

1990년대에 브라우저들은 HTML 어휘에 다양한 표현 태그들을 추가했는데, 여기에는 Netscape Navigator의 `<blink>…</blink>`와 Internet Explorer의 `<marquee>…</marquee>` 같은 악명 높은 비표준 태그들이 포함되어 사용성과 브라우저 호환성에 광범위한 문제를 일으켰습니다.

HTML을 원래 목적인 문서의 논리적 구조 기술로 제한하기 위해, 브라우저 제조업체들은 결국 두 가지 추가 언어를 지원하기로 합의했습니다: 페이지의 표현 스타일을 기술하는 CSS와 동적 상호작용을 기술하는 JavaScript(JS)입니다.

그 이후로 20년간의 공진화를 통해 세 언어는 더욱 간결하고 강력해졌습니다. 특히 JS 엔진의 개선으로 [AngularJS](http://angularjs.org/)와 같은 대규모 JS 프레임워크를 실용적으로 배포할 수 있게 되었습니다.

오늘날 (웹 스프레드시트와 같은) 크로스 플랫폼 _웹 애플리케이션_은 이전 세기의 플랫폼별 애플리케이션(VisiCalc, Lotus 1-2-3, Excel 등)만큼 보편적이고 인기가 높습니다.

AngularJS로 99줄에 웹 애플리케이션이 얼마나 많은 기능을 제공할 수 있을까요? 실제로 확인해보겠습니다!

## 개요

[spreadsheet](https://github.com/audreyt/500lines/tree/master/spreadsheet/code) 디렉토리는 2014년 후반 버전의 세 가지 웹 언어를 보여주는 쇼케이스입니다: 구조를 위한 [HTML5](http://www.w3.org/TR/html5/), 표현을 위한 [CSS3](http://www.w3.org/TR/css3-ui/), 그리고 상호작용을 위한 JS [ES6 "Harmony"](http://git.io/es6features) 표준입니다. 또한 데이터 지속성을 위해 [웹 스토리지](http://www.whatwg.org/specs/web-apps/current-work/multipage/webstorage.html)를 사용하고, 백그라운드에서 JS 코드를 실행하기 위해 [웹 워커](http://www.whatwg.org/specs/web-apps/current-work/multipage/workers.html)를 사용합니다. 이 글을 작성하는 시점에서 이러한 웹 표준들은 Firefox, Chrome, Internet Explorer 11+ 및 iOS 5+와 Android 4+의 모바일 브라우저에서 지원됩니다.

이제 브라우저에서 [우리의 스프레드시트](http://audreyt.github.io/500lines/spreadsheet/)를 열어보겠습니다 (\aosafigref{500l.spreadsheet.initial}):

\aosafigure[240pt]{spreadsheet-images/01-initial.png}{초기 화면}{500l.spreadsheet.initial}

### 기본 개념

스프레드시트는 2차원으로 구성되며, _열_은 **A**부터 시작하고 _행_은 **1**부터 시작합니다. 각 _셀_은 고유한 _좌표_(예: **A1**)와 _내용_(예: "1874")을 가지며, 다음 네 가지 _유형_ 중 하나에 속합니다:

* 텍스트: **B1**의 "+"와 **D1**의 "->"처럼 왼쪽 정렬됩니다.
* 숫자: **A1**의 "1874"와 **C1**의 "2046"처럼 오른쪽 정렬됩니다.
* 수식: **E1**의 `=A1+C1`처럼 _계산_되어 _값_ "3920"을 가지며, 연한 파란색 배경으로 표시됩니다.
* 빈 셀: **2**행의 모든 셀들은 현재 비어있습니다.

"3920"을 클릭하여 **E1**에 _포커스_를 설정하면 _입력 상자_에 수식이 나타납니다 (\aosafigref{500l.spreadsheet.inputbox}).

\aosafigure[240pt]{spreadsheet-images/02-input.png}{입력 상자}{500l.spreadsheet.inputbox}

이제 **A1**에 포커스를 설정하고 내용을 "1"로 _변경_하면, **E1**이 값을 "2047"로 _재계산_합니다 (\aosafigref{500l.spreadsheet.changed}).

\aosafigure[240pt]{spreadsheet-images/03-changed.png}{변경된 내용}{500l.spreadsheet.changed}

**ENTER**를 눌러 **A2**로 포커스를 이동하고 내용을 `=Date()`로 변경한 다음, **TAB**을 누르고 **B2**의 내용을 `=alert()`로 변경하고, 다시 **TAB**을 눌러 **C2**로 포커스를 설정합니다 (\aosafigref{500l.spreadsheet.error}).

\aosafigure[240pt]{spreadsheet-images/04-error.png}{수식 오류}{500l.spreadsheet.error}

이는 수식이 숫자(**E1**의 "2047"), 텍스트(**A2**의 현재 시간, 왼쪽 정렬), 또는 _오류_(**B2**의 빨간 글자, 가운데 정렬)로 계산될 수 있음을 보여줍니다.

다음으로 무한히 종료되지 않는 무한 루프를 위한 JS 코드인 `=for(;;){}`를 입력해보겠습니다. 스프레드시트는 변경 시도 후 **C2**의 내용을 자동으로 _복원_하여 이를 방지합니다.

이제 **Ctrl-R** 또는 **Cmd-R**로 브라우저에서 페이지를 새로고침하여 스프레드시트 내용이 브라우저 세션 간에 동일하게 유지되는 _지속성_을 확인합니다. 스프레드시트를 원래 내용으로 _재설정_하려면 왼쪽 상단의 '곡선 화살표' 버튼을 누르세요.

### 점진적 향상

99줄의 코드를 자세히 살펴보기 전에, 브라우저에서 JS를 비활성화하고 페이지를 새로고침한 다음 차이점을 확인해 보겠습니다 (\aosafigref{500l.spreadsheet.nojs}).

* 큰 격자 대신 단일 내용 셀이 있는 2x2 테이블만 화면에 남습니다.
* 행과 열 레이블이 `{{ row }}`와 `{{ col }}`로 바뀝니다.
* 재설정 버튼을 눌러도 효과가 없습니다.
* **TAB**을 누르거나 첫 번째 내용 줄을 클릭하면 여전히 편집 가능한 입력 상자가 나타납니다.

\aosafigure[240pt]{spreadsheet-images/05-nojs.png}{JavaScript 비활성화 상태}{500l.spreadsheet.nojs}

동적 상호작용(JS)을 비활성화하면 콘텐츠 구조(HTML)와 표현 스타일(CSS)은 계속 유지됩니다. 웹사이트가 JS와 CSS가 모두 비활성화된 상태에서도 유용하다면 _점진적 향상_ 원칙을 준수한다고 말하며, 이는 가능한 한 가장 많은 사용자가 콘텐츠에 접근할 수 있도록 합니다.

우리의 스프레드시트는 서버 측 코드가 없는 웹 애플리케이션이므로 필요한 로직을 제공하기 위해 JS에 의존해야 합니다. 하지만 스크린 리더나 텍스트 모드 브라우저와 같이 CSS가 완전히 지원되지 않는 환경에서도 올바르게 작동합니다.

\aosafigure[240pt]{spreadsheet-images/06-nocss.png}{CSS 비활성화 상태}{500l.spreadsheet.nocss}

\aosafigref{500l.spreadsheet.nocss}에 나타난 것처럼, 브라우저에서 JS를 활성화하고 대신 CSS를 비활성화하면 다음과 같은 효과가 나타납니다:

* 모든 배경색과 전경색이 사라집니다.
* 한 번에 하나씩 표시되는 대신 입력 상자와 셀 값이 모두 표시됩니다.
* 그 외에는 애플리케이션이 전체 버전과 동일하게 작동합니다.

## 코드 둘러보기

\aosafigref{500l.spreadsheet.architecture}는 HTML과 JS 구성 요소 간의 연결을 보여줍니다. 다이어그램을 이해하기 위해 브라우저가 로드하는 순서와 동일한 순서로 네 개의 소스 코드 파일을 살펴보겠습니다.

\aosafigure[240pt]{spreadsheet-images/00-architecture.png}{아키텍처 다이어그램}{500l.spreadsheet.architecture}


* **index.html**: 19줄
* **main.js**: 38줄 (주석과 빈 줄 제외)
* **worker.js**: 30줄 (주석과 빈 줄 제외)
* **styles.css**: 12줄

### HTML

`index.html`의 첫 번째 줄은 UTF-8 인코딩으로 HTML5로 작성되었음을 선언합니다:

```html
<!DOCTYPE html><html><head><meta charset="UTF-8">
```

`charset` 선언이 없으면 브라우저는 재설정 버튼의 유니코드 기호를 `â†»`로 표시할 수 있는데, 이는 디코딩 문제로 인한 깨진 텍스트의 예인 _모지바케_입니다.

다음 세 줄은 평소와 같이 `head` 섹션 내에 배치된 JS 선언입니다:

```html
  <script src="lib/angular.js"></script>
  <script src="main.js"></script>
  <script>
      try { angular.module('500lines') }
      catch(e){ location="es5/index.html" }
  </script>
```

`<script src="…">` 태그는 HTML 페이지와 동일한 경로에서 JS 리소스를 로드합니다. 예를 들어, 현재 URL이 `http://abc.com/x/index.html`이면 `lib/angular.js`는 `http://abc.com/x/lib/angular.js`를 참조합니다.

`try{ angular.module('500lines') }` 줄은 `main.js`가 올바르게 로드되었는지 테스트하고, 그렇지 않으면 브라우저에 `es5/index.html`로 이동하도록 지시합니다. 이 _리다이렉트 기반 점진적 성능 저하_ 기술은 ES6를 지원하지 않는 2015년 이전 브라우저에서 ES5로 변환된 JS 프로그램 버전을 대체재로 사용할 수 있도록 보장합니다.

다음 두 줄은 CSS 리소스를 로드하고, `head` 섹션을 닫고, 사용자가 볼 수 있는 부분을 포함하는 `body` 섹션을 시작합니다:

```html
  <link href="styles.css" rel="stylesheet">
</head><body ng-app="500lines" ng-controller="Spreadsheet" ng-cloak>
```

위의 `ng-app`과 `ng-controller` 속성은 [AngularJS](http://angularjs.org/)에게 `500lines` 모듈의 `Spreadsheet` 함수를 호출하도록 지시하며, 이는 문서 _뷰_에 _바인딩_을 제공하는 객체인 _모델_을 반환합니다. (`ng-cloak` 속성은 바인딩이 완료될 때까지 문서를 화면에 표시하지 않습니다.)

구체적인 예로, 사용자가 다음 줄에 정의된 `<button>`을 클릭하면, `ng-click` 속성이 트리거되어 JS 모델에서 제공하는 두 개의 명명된 함수인 `reset()`과 `calc()`를 호출합니다:

```html
  <table><tr>
    <th><button type="button" ng-click="reset(); calc()">↻</button></th>
```

다음 줄은 `ng-repeat`를 사용하여 최상단 행에 열 레이블 목록을 표시합니다:

```html
    <th ng-repeat="col in Cols">{{ col }}</th>
```

예를 들어, JS 모델이 `Cols`를 `["A","B","C"]`로 정의하면, 그에 따라 레이블이 지정된 세 개의 헤딩 셀(`th`)이 생성됩니다. `{{ col }}` 표기법은 AngularJS에게 표현식을 _보간_하도록 지시하여 각 `th`의 내용을 현재 `col` 값으로 채웁니다.

마찬가지로, 다음 두 줄은 `Rows`의 값들(`[1,2,3]` 등)을 순회하여 각각에 대해 행을 만들고 맨 왼쪽 `th` 셀에 해당 번호로 레이블을 지정합니다:

```html
  </tr><tr ng-repeat="row in Rows">
    <th>{{ row }}</th>
```

`<tr ng-repeat>` 태그가 아직 `</tr>`로 닫히지 않았기 때문에, `row` 변수는 여전히 표현식에서 사용할 수 있습니다. 다음 줄은 현재 행에 데이터 셀(`td`)을 생성하고 `ng-class` 속성에서 `col`과 `row` 변수를 모두 사용합니다:

```html
    <td ng-repeat="col in Cols" ng-class="{ formula: ('=' === sheet[col+row][0]) }">
```

여기에는 몇 가지 일이 일어나고 있습니다. HTML에서 `class` 속성은 CSS가 다르게 스타일을 지정할 수 있도록 하는 _클래스 이름 집합_을 설명합니다. 여기의 `ng-class`는 표현식 `('=' === sheet[col+row][0])`을 평가하고, 이것이 참이면 `<td>`는 추가 클래스로 `formula`를 받게 되어, **styles.css**의 8번째 줄에서 `.formula` _클래스 선택자_로 정의된 대로 셀에 연한 파란색 배경을 제공합니다.

위 표현식은 `sheet[col+row]`의 문자열에서 `=`가 첫 번째 문자(`[0]`)인지 테스트하여 현재 셀이 수식인지 확인합니다. 여기서 `sheet`는 좌표(예: `"E1"`)를 속성으로, 셀 내용(예: `"=A1+C1"`)을 값으로 하는 JS 모델 객체입니다. `col`은 숫자가 아닌 문자열이므로 `col+row`의 `+`는 덧셈이 아닌 문자열 연결을 의미합니다.

`<td>` 안에서 우리는 `sheet[col+row]`에 저장된 셀 내용을 편집할 수 있는 입력 상자를 사용자에게 제공합니다:

```html
       <input id="{{ col+row }}" ng-model="sheet[col+row]" ng-change="calc()"
        ng-model-options="{ debounce: 200 }" ng-keydown="keydown( $event, col, row )">
```

여기서 핵심 속성은 `ng-model`로, JS 모델과 입력 상자의 편집 가능한 내용 간에 _양방향 바인딩_을 가능하게 합니다. 실제로 이는 사용자가 입력 상자에서 변경을 할 때마다 JS 모델이 내용과 일치하도록 `sheet[col+row]`를 업데이트하고, 모든 수식 셀의 값을 재계산하기 위해 `calc()` 함수를 트리거한다는 의미입니다.

사용자가 키를 누르고 유지할 때 `calc()`의 반복 호출을 피하기 위해, `ng-model-options`는 업데이트 빈도를 200밀리초마다 한 번으로 제한합니다.

여기의 `id` 속성은 좌표 `col+row`로 보간됩니다. HTML 요소의 `id` 속성은 동일한 문서의 다른 모든 요소의 `id`와 달라야 합니다. 이는 `#A1` _ID 선택자_가 클래스 선택자 `.formula`처럼 요소 집합이 아닌 단일 요소를 참조하도록 보장합니다. 사용자가 **UP**/**DOWN**/**ENTER** 키를 누를 때, `keydown()`의 키보드 탐색 로직은 ID 선택자를 사용하여 포커스할 입력 상자를 결정합니다.

입력 상자 후에, 우리는 JS 모델에서 `errs`와 `vals` 객체로 표현되는 현재 셀의 계산된 값을 표시하기 위해 `<div>`를 배치합니다:

```html
      <div ng-class="{ error: errs[col+row], text: vals[col+row][0] }">
        {{ errs[col+row] || vals[col+row] }}</div>
```

수식을 계산할 때 오류가 발생하면, 텍스트 보간은 `errs[col+row]`에 포함된 오류 메시지를 사용하고, `ng-class`는 요소에 `error` 클래스를 적용하여 CSS가 다르게 스타일을 지정할 수 있게 합니다(빨간 글자, 가운데 정렬 등).

오류가 없을 때는 대신 `||`의 오른쪽에 있는 `vals[col+row]`가 보간됩니다. 비어있지 않은 문자열이면, 첫 번째 문자(`[0]`)가 참으로 평가되어 텍스트를 왼쪽 정렬하는 `text` 클래스가 요소에 적용됩니다.

빈 문자열과 숫자 값은 첫 번째 문자가 없기 때문에, `ng-class`는 클래스를 할당하지 않으므로 CSS가 기본적으로 오른쪽 정렬로 스타일을 지정할 수 있습니다.

마지막으로, 열 수준의 `ng-repeat` 루프를 `</td>`로 닫고, 행 수준 루프를 `</tr>`로 닫고, HTML 문서를 다음으로 끝냅니다:

```html
    </td>
  </tr></table>
</body></html>
```

### JS: 메인 컨트롤러

`main.js` 파일은 `index.html`의 `<body>` 요소에서 요구하는 `500lines` 모듈과 그 `Spreadsheet` 컨트롤러 함수를 정의합니다.

HTML 뷰와 백그라운드 워커 사이의 다리 역할로서, 네 가지 작업을 수행합니다:

* 열과 행의 차원과 레이블을 정의합니다.
* 키보드 탐색과 재설정 버튼을 위한 이벤트 핸들러를 제공합니다.
* 사용자가 스프레드시트를 변경하면, 새로운 내용을 워커에게 전송합니다.
* 워커에서 계산된 결과가 도착하면, 뷰를 업데이트하고 현재 상태를 저장합니다.

\aosafigref{500l.spreadsheet.flowchart}의 순서도는 컨트롤러-워커 상호작용을 더 자세히 보여줍니다:

\aosafigure[240pt]{spreadsheet-images/00-flowchart.png}{컨트롤러-워커 순서도}{500l.spreadsheet.flowchart}

이제 코드를 살펴보겠습니다. 첫 번째 줄에서 AngularJS의 `$scope`를 요청합니다:

```javascript
angular.module('500lines', []).controller('Spreadsheet', function ($scope, $timeout) {
```

`$scope`의 `$`는 변수 이름의 일부입니다. 여기서는 AngularJS의 [`$timeout`](https://docs.angularjs.org/api/ng/service/$timeout) 서비스 함수도 요청합니다. 나중에 무한 루프 수식을 방지하는 데 사용할 것입니다.

To put `Cols` and `Rows` into the model, simply define them as properties of `$scope`:

```javascript
  // Begin of $scope properties; start with the column/row labels
  $scope.Cols = [], $scope.Rows = [];
  for (col of range( 'A', 'H' )) { $scope.Cols.push(col); }
  for (row of range( 1, 20 )) { $scope.Rows.push(row); }
```

The ES6 [for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) syntax makes it easy to loop through ranges with a start and an end point, with the helper function `range` defined as a [generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*):


```javascript
  function* range(cur, end) { while (cur <= end) { yield cur;
```

The `function*` above means that `range` returns an [iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/The_Iterator_protocol), with a `while` loop that would  [`yield`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield) a single value at a time. Whenever the `for` loop demands the next value, it will resume execution right after the `yield` line:

```
    // If it’s a number, increase it by one; otherwise move to next letter
    cur = (isNaN( cur ) ? String.fromCodePoint( cur.codePointAt()+1 ) : cur+1);
  } }
```

To generate the next value, we use `isNaN` to see if `cur` is meant as a letter (`NaN` stands for “not a number.”) If so, we get the letter’s [code point value](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/codePointAt), increment it by one, and [convert the codepoint](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/fromCodePoint) back to get its next letter. Otherwise, we simply increase the number by one.

Next up, we define the `keydown()` function that handles keyboard navigation across rows:

```javascript
  // UP(38) and DOWN(40)/ENTER(13) move focus to the row above (-1) and below (+1).
  $scope.keydown = ({which}, col, row)=>{ switch (which) {
```

The [arrow function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/arrow_functions) receives the arguments `($event, col, row)` from `<input ng-keydown>`, using [destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/1.7#Pulling_fields_from_objects_passed_as_function_parameter) to assign `$event.which` into the `which` parameter, and checks if it’s among the three navigational key codes:

```javascript
    case 38: case 40: case 13: $timeout( ()=>{
```

If it is, we use `$timeout` to schedule a focus change after the current `ng-keydown` and `ng-change` handler. Because `$timeout` requires a function as argument, the `()=>{…}` syntax constructs a function to represent the focus-change logic, which starts by checking the direction of movement:

```javascript
      const direction = (which === 38) ? -1 : +1;
```

The `const` declarator means `direction` will not change during the function’s execution. The direction to move is either upward (`-1`, from **A2** to **A1**) if the key code is 38 (**UP**), or downward (`+1`, from **A2** to **A3**) otherwise.

Next up, we retrieve the target element using the ID selector syntax (e.g. `"#A3"`), constructed with a [template string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings) written in a pair of backticks, concatenating the leading `#`, the current `col` and the target `row + direction`:

```javascript
      const cell = document.querySelector( `#${ col }${ row + direction }` );
      if (cell) { cell.focus(); }
    } );
  } };
```

We put an extra check on the result of `querySelector` because moving upward from **A1** will produce the selector `#A0`, which has no corresponding element, and so will not trigger a focus change — the same goes for pressing **DOWN** at the bottom row.

Next, we define the `reset()` function so the reset button can restore the contents of the `sheet`:

```javascript
  // Default sheet content, with some data cells and one formula cell.
  $scope.reset = ()=>{ 
    $scope.sheet = { A1: 1874, B1: '+', C1: 2046, D1: '->', E1: '=A1+C1' }; }
```

The `init()` function tries restoring the `sheet` content from its previous state from the [localStorage](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Storage#localStorage), and defaults to the initial content if it’s our first time running the application:

```javascript
  // Define the initializer, and immediately call it
  ($scope.init = ()=>{
    // Restore the previous .sheet; reset to default if it’s the first run
    $scope.sheet = angular.fromJson( localStorage.getItem( '' ) );
    if (!$scope.sheet) { $scope.reset(); }
    $scope.worker = new Worker( 'worker.js' );
  }).call();
```

위의 `init()` 함수에서 주목할 만한 몇 가지 사항이 있습니다:

* We use the `($scope.init = ()=>{…}).call()` syntax to define the function and immediately call it.
* Because localStorage only stores strings, we _parse_ the `sheet` structure from its [JSON](https://developer.mozilla.org/en-US/docs/Glossary/JSON) representation using `angular.fromJson()`.
* At the last step of `init()`, we create a new [web worker](https://developer.mozilla.org/en-US/docs/Web/API/Worker) thread and assign it to the `worker` scope property. Although the worker is not directly used in the view, it’s customary to use `$scope` to share objects used across model functions, in this case between `init()` here and `calc()` below.

While `sheet` holds the user-editable cell content, `errs` and `vals` contain the results of calculations — errors and values — that are read-only to the user:

```javascript
  // Formula cells may produce errors in .errs; normal cell contents are in .vals
  [$scope.errs, $scope.vals] = [ {}, {} ];
```

With these properties in place, we can define the `calc()` function that triggers whenever the user makes a change to `sheet`:

```javascript
  // Define the calculation handler; not calling it yet
  $scope.calc = ()=>{
    const json = angular.toJson( $scope.sheet );
```

Here we take a snapshot of the state of `sheet` and store it in the constant `json`, a JSON string. Next up, we construct a `promise` from [$timeout](https://docs.angularjs.org/api/ng/service/$timeout) that cancels the upcoming computation if it takes more than 99 milliseconds:

```javascript
    const promise = $timeout( ()=>{
      // If the worker has not returned in 99 milliseconds, terminate it
      $scope.worker.terminate();
      // Back up to the previous state and make a new worker
      $scope.init();
      // Redo the calculation using the last-known state
      $scope.calc();
    }, 99 );
```

Since we made sure that  `calc()` is called at most once every 200 milliseconds via the `<input ng-model-options>` attribute in HTML, this arrangement leaves 101 milliseconds for `init()` to restore `sheet` to the last known-good state and make a new worker.

The worker’s task is to calculate `errs` and `vals` from the contents of`sheet`. Because **main.js** and **worker.js** communicate by message-passing, we need an `onmessage` handler to receive the results once they are ready:

```javascript
    // When the worker returns, apply its effect on the scope
    $scope.worker.onmessage = ({data})=>{
      $timeout.cancel( promise );
      localStorage.setItem( '', json );
      $timeout( ()=>{ [$scope.errs, $scope.vals] = data; } );
    };
```

If `onmessage` is called,  we know that the `sheet` snapshot in `json` is stable (i.e., containing no infinite-looping formulas), so we cancel the 99-millisecond timeout, write the snapshot to localStorage, and schedule a UI update with a `$timeout` function that updates `errs` and `vals` to the user-visible view.

With the handler in place, we can post the state of `sheet` to the worker, starting its calculation in the background:

```javascript
    // Post the current sheet content for the worker to process
    $scope.worker.postMessage( $scope.sheet );
  };

  // Start calculation when worker is ready
  $scope.worker.onmessage = $scope.calc;
  $scope.worker.postMessage( null );
});
```

### JS: Background Worker

메인 JS 스레드 대신 웹 워커를 사용하여 수식을 계산하는 데는 세 가지 이유가 있습니다:

* While the worker runs in the background, the user is free to continue interacting with the spreadsheet without getting blocked by computation in the main thread. 
* Because we accept any JS expression in a formula, the worker provides a _sandbox_ that prevents formulas from interfering with the page that contains them, such as by popping out an `alert()` dialog box.
* A formula can refer to any coordinates as variables. The other coordinates may contain another formula that might end in a cyclic reference. To solve this problem, we use the worker’s _global scope_ object `self`, and define these variables as _getter functions_ on `self` to implement the cycle-prevention logic.

With these in mind, let’s take a look at the worker’s code.

The worker’s sole purpose is defining its `onmessage` handler. The handler takes `sheet`, calculates `errs` and `vals`, and posts them back to the main JS thread. We begin by re-initializing the three variables when we receive a message:

```javascript
let sheet, errs, vals;
self.onmessage = ({data})=>{
  [sheet, errs, vals] = [ data, {}, {} ];
```

In order to turn coordinates into global variables, we first iterate over each property in `sheet`, using a `for…in` loop:

```javascript
  for (const coord in sheet) {
```

ES6 introduces `const` and `let` declares _block scoped_ constants and variables; `const coord` above means that functions defined in the loop would capture the value of `coord` in each iteration.

In contrast, `var coord` in earlier versions of JS would declare a _function scoped_ variable, and functions defined in each loop iteration would end up pointing to the same `coord` variable.

Customarily, formula variables are case-insensitive and can optionally have a `$` prefix. Because JS variables are case-sensitive, we use `map` to go over the four variable names for the same coordinate:

```javascript
    // Four variable names pointing to the same coordinate: A1, a1, $A1, $a1
    [ '', '$' ].map( p => [ coord, coord.toLowerCase() ].map(c => {
      const name = p+c;
```

Note the shorthand arrow function syntax above: `p => ...` is the same as `(p) => { ... }`.

For each variable name, like `A1` and `$a1`, we define an [accessor property](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) on `self` that calculates `vals["A1"]` whenever they are evaluated in an expression:

```javascript
      // Worker is reused across calculations, so only define each variable once
      if ((Object.getOwnPropertyDescriptor( self, name ) || {}).get) { return; }

      // Define self['A1'], which is the same thing as the global variable A1
      Object.defineProperty( self, name, { get() {
```

The `{ get() { … } }` syntax above is shorthand for `{ get: ()=>{ … } }`. Because we define only `get` and not `set`, the variables become  _read-only_ and cannot be modified from user-supplied formulas.

The `get` accessor starts by checking `vals[coord]`, and simply returns it if it’s already calculated:

```javascript
        if (coord in vals) { return vals[coord]; }
```

If not, we need to calculate `vals[coord]` from `sheet[coord]`.

First we set it to `NaN`, so self-references like setting **A1** to `=A1` will end up with `NaN` instead of an infinite loop:

```javascript
        vals[coord] = NaN;
```

Next we check if `sheet[coord]` is a number by converting it to numeric with prefix `+`, assigning the number to `x`, and comparing its string representation with the original string. If they differ, then we set `x` to the original string:

```javascript
        // Turn numeric strings into numbers, so =A1+C1 works when both are numbers
        let x = +sheet[coord];
        if (sheet[coord] !== x.toString()) { x = sheet[coord]; }
```

If the initial character of `x` is `=`, then it’s a formula cell. We evaluate the part after `=` with `eval.call()`, using the first argument `null` to tell `eval` to run in the _global scope_, hiding the _lexical scope_ variables like `x` and `sheet` from the evaluation:

```javascript
        // Evaluate formula cells that begin with =
        try { vals[coord] = (('=' === x[0]) ? eval.call( null, x.slice( 1 ) ) : x);
```

If the evaluation succeeds, the result is stored into `vals[coord]`. For non-formula cells, the value of `vals[coord]` is simply `x`, which may be a number or a string.

If `eval` results in an error, the `catch` block tests if it’s because the formula refers to an empty cell not yet defined in `self`:

```javascript
        } catch (e) {
          const match = /\$?[A-Za-z]+[1-9][0-9]*\b/.exec( e );
          if (match && !( match[0] in self )) {
```

In that case, we set the missing cell’s default value to "0", clear `vals[coord]`, and re-run the current computation using `self[coord]`:

```javascript
            // The formula refers to a uninitialized cell; set it to 0 and retry
            self[match[0]] = 0;
            delete vals[coord];
            return self[coord];
          }
```

 If the user gives the missing cell a content later on in `sheet[coord]`, then the temporary value would be overridden by `Object.defineProperty`.

Other kinds of errors are stored in `errs[coord]`:

```javascript
          // Otherwise, stringify the caught exception in the errs object
          errs[coord] = e.toString();
        }
```

In case of errors, the value of `vals[coord]` will remain `NaN` because the assignment did not finish executing.

Finally, the `get` accessor returns the calculated value stored in `vals[coord]`, which must be a number, a Boolean value, or a string:

```javascript
        // Turn vals[coord] into a string if it's not a number or Boolean
        switch (typeof vals[coord]) { 
            case 'function': case 'object': vals[coord]+=''; 
        }
        return vals[coord];
      } } );
    }));
  }
```

With accessors defined for all coordinates, the worker goes through the coordinates again, invoking each accessor with `self[coord]`, then posts the resulting `errs` and `vals` back to the main JS thread:

```javascript
  // For each coordinate in the sheet, call the property getter defined above
  for (const coord in sheet) { self[coord]; }
  return [ errs, vals ];
}
```

### CSS

**styles.css** 파일은 몇 개의 선택자와 그 표현 스타일만을 포함합니다. 먼저 인접한 셀 사이에 공간을 두지 않고 모든 셀 테두리를 합치도록 테이블 스타일을 지정합니다:

```css
table { border-collapse: collapse; }
```

헤딩 셀과 데이터 셀은 동일한 테두리 스타일을 공유하지만, 배경색으로 구별할 수 있습니다: 헤딩 셀은 연한 회색, 데이터 셀은 기본적으로 흰색, 수식 셀은 연한 파란색 배경을 가집니다:

```
th, td { border: 1px solid #ccc; }
th { background: #ddd; }
td.formula { background: #eef; }
```

각 셀의 계산된 값에 대해 표시 너비가 고정됩니다. 빈 셀은 최소 높이를 받고, 긴 줄은 끝에 줄임표로 잘립니다:

```css
td div { text-align: right; width: 120px; min-height: 1.2em;
         overflow: hidden; text-overflow: ellipsis; }
```

텍스트 정렬과 장식은 `text`와 `error` 클래스 선택자에 반영된 각 값의 유형에 따라 결정됩니다:

```css
div.text { text-align: left; }
div.error { text-align: center; color: #800; font-size: 90%; border: solid 1px #800 }
```

사용자가 편집할 수 있는 `input` 상자의 경우, _절대 위치 지정_을 사용하여 해당 셀 위에 오버레이하고, 셀 값이 있는 아래쪽 `div`가 비쳐 보이도록 투명하게 만듭니다:

```css
input { position: absolute; border: 0; padding: 0;
        width: 120px; height: 1.3em; font-size: 100%;
        color: transparent; background: transparent; }
```

사용자가 입력 상자에 포커스를 설정하면 전경으로 나타납니다:

```css
input:focus { color: #111; background: #efe; }
```

또한 아래쪽 `div`는 한 줄로 축소되어 입력 상자에 완전히 덮입니다:

```css
input:focus + div { white-space: nowrap; }
```

## 결론

이 책이 _500 Lines or Less_이므로, 99줄로 작성된 웹 스프레드시트는 최소한의 예제입니다&mdash;원하는 방향으로 자유롭게 실험하고 확장해 보세요.

다음은 남은 401줄의 공간에서 쉽게 달성할 수 있는 몇 가지 아이디어입니다:

* [ShareJS](http://sharejs.org/), [AngularFire](http://angularfire.com) 또는 [GoAngular](http://goangular.org/)를 사용한 협업 온라인 에디터
* [angular-marked](http://ngmodules.org/modules/angular-marked)를 사용한 텍스트 셀의 Markdown 구문 지원
* [OpenFormula 표준](https://en.wikipedia.org/wiki/OpenFormula)의 일반적인 수식 함수들(`SUM`, `TRIM` 등)
* [SheetJS](http://sheetjs.com/)를 통한 CSV 및 SpreadsheetML과 같은 인기 스프레드시트 형식과의 상호 운용성
* Google Spreadsheet 및 [EtherCalc](http://ethercalc.net/)와 같은 온라인 스프레드시트 서비스에서 가져오기 및 내보내기

### JS 버전에 대한 참고사항

이 챕터는 ES6의 새로운 개념을 보여주는 것을 목표로 하므로, 2015년 이전 브라우저에서 실행하기 위해 소스 코드를 ES5로 변환하는 [Traceur 컴파일러](https://github.com/google/traceur-compiler)를 사용합니다.

2010년 버전의 JS로 직접 작업하는 것을 선호한다면, [as-javascript-1.8.5](https://audreyt.github.io/500lines/spreadsheet/as-javascript-1.8.5/) 디렉토리에 ES5 스타일로 작성된 **main.js**와 **worker.js**가 있습니다. [소스 코드](https://github.com/audreyt/500lines/tree/master/spreadsheet/as-javascript-1.8.5)는 동일한 줄 수로 ES6 버전과 줄별로 비교할 수 있습니다.

더 깔끔한 구문을 선호하는 사람들을 위해, [as-livescript-1.3.0](https://audreyt.github.io/500lines/spreadsheet/as-livescript-1.3.0/) 디렉토리는 **main.ls**와 **worker.ls**를 작성하기 위해 ES6 대신 [LiveScript](http://livescript.net/)를 사용합니다. 이는 JS 버전보다 [20줄 짧습니다](https://github.com/audreyt/500lines/tree/master/spreadsheet/as-livescript-1.3.0).

LiveScript 언어를 기반으로 하여, [as-react-livescript](https://audreyt.github.io/500lines/spreadsheet/as-react-livescript/) 디렉토리는 [ReactJS](https://facebook.github.io/react/) 프레임워크를 사용합니다. 이는 AngularJS에 상응하는 것보다 [10줄 더 길지만](https://github.com/audreyt/500lines/tree/master/spreadsheet/as-react-livescript), 상당히 빠르게 실행됩니다.

이 예제를 다른 JS 언어로 번역하는 데 관심이 있으시면, [풀 리퀘스트](https://github.com/audreyt/500lines/pulls)를 보내주세요&mdash;듣고 싶습니다!
