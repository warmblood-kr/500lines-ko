title: Blockcode: 시각적 프로그래밍 툴킷
author: Dethe Elza
<markdown>
_[Dethe](https://twitter.com/dethe)는 geek dad이자 미적 프로그래머, 멘토이며, [Waterbear](http://waterbearlang.com/) 시각적 프로그래밍 도구의 창시자입니다. 그는 밴쿠버 메이커 교육 살롱을 공동 주최하며, 세상을 로봇 종이접기 토끼로 가득 채우고 싶어합니다._
</markdown>
블록 기반 프로그래밍 언어에서는 프로그램의 일부를 나타내는 블록들을 드래그하고 연결하여 프로그램을 작성합니다. 블록 기반 언어는 단어와 기호를 입력하는 기존의 프로그래밍 언어와 다릅니다.

프로그래밍 언어를 배우는 것은 어려울 수 있습니다. 왜냐하면 이들은 아주 사소한 오타에도 극도로 민감하기 때문입니다. 대부분의 프로그래밍 언어는 대소문자를 구분하고, 모호한 문법을 가지며, 세미콜론이 잘못된 위치에 있거나&mdash;더 나쁘게는 빠져있기만 해도 실행을 거부합니다. 더욱이, 현재 사용되는 대부분의 프로그래밍 언어는 영어에 기반하고 있어 그들의 문법을 현지화할 수 없습니다.

이와 대조적으로, 잘 만들어진 블록 언어는 구문 오류를 완전히 제거할 수 있습니다. 여전히 잘못된 일을 하는 프로그램을 만들 수는 있지만, 잘못된 구문을 가진 프로그램은 만들 수 없습니다: 블록들이 그런 방식으로는 맞지 않기 때문입니다. 블록 언어는 더 발견하기 쉽습니다: 언어의 모든 구조와 라이브러리를 블록 목록에서 바로 볼 수 있습니다. 더욱이, 블록은 프로그래밍 언어의 의미를 바꾸지 않으면서 어떤 인간 언어로든 현지화할 수 있습니다.

\aosafigure[240pt]{blockcode-images/blockcode_ide.png}{사용 중인 Blockcode IDE}{500l.blockcode.ide}

블록 기반 언어는 긴 역사를 가지고 있으며, 그 중 저명한 것들로는 [Lego Mindstorms](http://www.lego.com/en-us/mindstorms/), [Alice3D](http://www.alice.org/index.php), [StarLogo](http://education.mit.edu/projects/starlogo-tng), 그리고 특히 [Scratch](http://scratch.mit.edu/)가 있습니다. 웹에서도 블록 기반 프로그래밍을 위한 여러 도구들이 있습니다: [Blockly](https://developers.google.com/blockly/), [AppInventor](http://appinventor.mit.edu/explore/), [Tynker](http://www.tynker.com/), 그리고 [기타 많은 것들](http://en.wikipedia.org/wiki/Visual_programming_language).

이 장의 코드는 오픈소스 프로젝트 [Waterbear](http://waterbearlang.com/)를 느슨하게 기반으로 하는데, 이것은 언어가 아니라 기존 언어들을 블록 기반 문법으로 래핑하는 도구입니다. 이러한 래퍼의 장점들은 위에서 언급한 것들을 포함합니다: 구문 오류 제거, 사용 가능한 구성 요소의 시각적 표시, 현지화의 용이함. 추가로, 시각적 코드는 때때로 읽고 디버깅하기 더 쉬울 수 있고, 블록은 타이핑 능력이 부족한 어린이들에 의해 사용될 수 있습니다. (블록에 아이콘을 넣어 텍스트 이름과 함께 또는 대신 사용하여 읽기 능력이 없는 어린이들도 프로그램을 작성할 수 있도록 더 나아갈 수도 있지만, 이 예시에서는 그렇게 하지 않습니다.)

이 언어에서 거북이 그래픽을 선택한 것은 어린이들에게 프로그래밍을 가르치기 위해 특별히 만들어진 Logo 언어로 거슬러 올라갑니다. 위의 블록 기반 언어들 중 몇 개는 거북이 그래픽을 포함하고 있으며, 이는 이처럼 엄격하게 제약된 프로젝트에서 포착할 수 있을 만큼 충분히 작은 도메인입니다.

블록 기반 언어가 어떤 것인지 느껴보고 싶다면, 작성자의 [GitHub 저장소](https://dethe.github.io/500lines/blockcode/)에서 이 장에서 구축된 프로그램을 실험해볼 수 있습니다.

## 목표와 구조

이 코드로 몇 가지를 달성하고 싶습니다. 무엇보다도, 가능한 한 간단한 HTML, CSS, JavaScript 구조를 사용하여 간단한 블록 드래그 앤 드롭으로 이미지를 생성하는 코드를 작성할 수 있는 거북이 그래픽용 블록 언어를 구현하고 싶습니다. 둘째로, 그러나 여전히 중요한 것은, 블록 자체가 우리의 미니 거북이 언어 외에 다른 언어들의 프레임워크로 어떻게 사용될 수 있는지를 보여주고 싶습니다.

이를 위해, 거북이 언어에 특화된 모든 것을 하나의 파일 \newline (`turtle.js`)에 캡슐화하여 다른 파일과 쉽게 교체할 수 있도록 합니다. 다른 것들은 거북이 언어에 특화되어서는 안 됩니다; 나머지는 블록 처리(`blocks.js`와 `menu.js`)이거나 일반적으로 유용한 웹 유틸리티(`util.js`, `drag.js`, `file.js`)에 관한 것이어야 합니다. 그것이 목표이지만, 프로젝트의 작은 크기를 유지하기 위해 일부 유틸리티들은 덜 범용적이고 블록과의 사용에 더 특화되어 있습니다.

블록 언어를 작성할 때 인상 깊었던 것은 언어가 자체 IDE라는 것이었습니다. 좋아하는 텍스트 에디터에서 블록을 코딩할 수는 없습니다; IDE는 블록 언어와 병행하여 설계되고 개발되어야 합니다. 이것은 장단점이 있습니다. 긍정적인 측면에서는, 모든 사람이 일관된 환경을 사용하게 되고 어떤 에디터를 사용할지에 대한 종교적 논쟁의 여지가 없습니다. 부정적인 측면에서는, 블록 언어 자체를 구축하는 것으로부터 큰 주의를 분산시킬 수 있습니다.

### 스크립트의 본질

Blockcode 스크립트는 (블록 기반이든 텍스트 기반이든) 어떤 언어의 스크립트와 마찬가지로 따라야 할 연산 순서입니다. Blockcode의 경우 스크립트는 HTML 요소들로 구성되며, 이들은 반복되고, 각각은 해당 블록의 차례가 왔을 때 실행될 특정 JavaScript 함수와 연결됩니다. 일부 블록은 다른 블록들을 포함할 수 있고 (그리고 실행할 책임이 있고), 일부 블록은 함수에 전달되는 숫자 인수들을 포함할 수 있습니다.

대부분의 (텍스트 기반) 언어에서, 스크립트는 여러 단계를 거칩니다: 렉서가 텍스트를 인식된 토큰으로 변환하고, 파서가 토큰들을 추상 구문 트리로 조직화한 다음, 언어에 따라 프로그램이 기계 코드로 컴파일되거나 인터프리터에 공급됩니다. 이는 단순화된 것입니다; 더 많은 단계가 있을 수 있습니다. Blockcode의 경우, 스크립트 영역에서 블록들의 레이아웃이 이미 우리의 추상 구문 트리를 나타내므로, 렉싱과 파싱 단계를 거칠 필요가 없습니다. 우리는 방문자 패턴을 사용하여 그러한 블록들을 반복하고 각 블록과 연결된 미리 정의된 JavaScript 함수를 호출하여 프로그램을 실행합니다.

전통적인 언어처럼 되기 위해 추가 단계를 더하는 것을 막는 것은 아무것도 없습니다. 연결된 JavaScript 함수를 단순히 호출하는 대신, `turtle.js`를 다른 가상 머신을 위한 바이트 코드를 방출하는 블록 언어나 심지어 컴파일러를 위한 C++ 코드로 교체할 수 있습니다. Java 로봇 코드를 생성하고, Arduino를 프로그래밍하고, Raspberry Pi에서 실행되는 Minecraft를 스크립팅하기 위한 블록 언어들이 (Waterbear 프로젝트의 일부로) 존재합니다.

### 웹 애플리케이션

가능한 한 가장 넓은 대상에게 도구를 사용 가능하게 만들기 위해, 이것은 웹 네이티브입니다. HTML, CSS, JavaScript로 작성되었으므로 대부분의 브라우저와 플랫폼에서 작동해야 합니다.

현대 웹 브라우저는 훌륭한 앱을 구축하기 위한 풍부한 도구 세트를 가진 강력한 플랫폼입니다. 구현에 대해 뭔가 너무 복잡해지면, 나는 그것을 "웹 방식"으로 하고 있지 않다는 신호로 받아들이고, 가능한 곳에서는 브라우저 도구를 더 잘 사용하는 방법을 다시 생각해보려고 노력했습니다.

웹 애플리케이션과 전통적인 데스크톱 또는 서버 애플리케이션의 중요한 차이점은 `main()` 또는 다른 진입점의 부재입니다. 브라우저에 이미 내장되어 모든 웹 페이지에서 암시적이기 때문에 명시적인 실행 루프가 없습니다. 우리의 모든 코드는 로드 시 파싱되고 실행되며, 그 시점에서 사용자와의 상호작용을 위해 관심 있는 이벤트에 등록할 수 있습니다. 첫 번째 실행 후, 우리 코드와의 모든 추가 상호작용은 우리가 설정하고 등록한 콜백을 통해 이루어질 것입니다. 이벤트(마우스 이동과 같은), 타임아웃(우리가 지정한 주기성으로 발사되는), 또는 프레임 핸들러(각 화면 다시 그리기마다 호출되는, 일반적으로 초당 60프레임)에 대해 등록하든 상관없이 말입니다. 브라우저는 완전한 기능의 스레드도 노출하지 않습니다 (공유하지 않는 웹 워커만).

## 코드 살펴보기

이 프로젝트 전반에 걸쳐 몇 가지 관례와 모범 사례를 따르려고 노력했습니다. 각 JavaScript 파일은 변수가 전역 환경으로 누출되는 것을 방지하기 위해 함수로 래핑됩니다. 다른 파일에 변수를 노출해야 하는 경우, 파일 이름에 기반하여 파일당 하나의 전역을 정의하고, 그 안에 노출된 함수들을 넣습니다. 이것은 파일의 끝 근처에 있고, 그 파일에 의해 설정된 이벤트 핸들러가 뒤따르므로, 항상 파일의 끝을 훑어보면 어떤 이벤트를 처리하고 어떤 함수를 노출하는지 알 수 있습니다.

코드 스타일은 객체지향이나 함수형이 아닌 절차적입니다. 이러한 패러다임 중 어느 것으로든 같은 일을 할 수 있지만, 그것은 DOM에 대해 이미 존재하는 것에 부과할 더 많은 설정 코드와 래퍼를 요구할 것입니다. [Custom Elements](http://webcomponents.org/)에 대한 최근 작업은 OO 방식으로 DOM과 작업하는 것을 더 쉽게 만들고, [Functional JavaScript](https://leanpub.com/javascript-allonge/read)에 대한 많은 훌륭한 글이 있었지만, 둘 다 약간의 억지스러움을 요구할 것이므로, 절차적으로 유지하는 것이 더 간단하다고 느꼈습니다.

이 프로젝트에는 8개의 소스 파일이 있지만, `index.html`과 `blocks.css`는 앱의 기본 구조와 스타일이므로 논의하지 않겠습니다. JavaScript 파일 중 두 개도 자세히 논의하지 않겠습니다: `util.js`는 일부 헬퍼를 포함하고 다른 브라우저 구현 간의 브릿지 역할을 합니다&mdash;jQuery와 같은 라이브러리와 유사하지만 50줄 미만의 코드입니다. `file.js`는 파일을 로딩하고 저장하며 스크립트를 직렬화하는 데 사용되는 유사한 유틸리티입니다.

다음은 나머지 파일들입니다:

* `block.js`는 블록 기반 언어의 추상적 표현입니다.
* `drag.js`는 언어의 핵심 상호작용을 구현합니다: 사용자가 사용 가능한 블록 목록("메뉴")에서 블록을 드래그하여 프로그램("스크립트")으로 조립할 수 있게 합니다.
* `menu.js`는 일부 헬퍼 코드를 가지고 있으며 실제로 사용자의 프로그램을 실행하는 책임도 있습니다.
* `turtle.js`는 우리 블록 언어(거북이 그래픽)의 세부사항을 정의하고 특정 블록들을 초기화합니다. 이것은 다른 블록 언어를 만들기 위해 교체될 파일입니다.

### `blocks.js`

각 블록은 몇 개의 HTML 요소들로 구성되고, CSS로 스타일이 적용되며, 드래그 앤 드롭과 입력 인수 수정을 위한 일부 JavaScript 이벤트 핸들러가 있습니다. `blocks.js` 파일은 이러한 요소들의 그룹핑을 단일 객체로 생성하고 관리하는 데 도움을 줍니다. 블록 유형이 블록 메뉴에 추가될 때, 언어를 구현하는 JavaScript 함수와 연결되므로, 스크립트의 각 블록은 연결된 함수를 찾고 스크립트가 실행될 때 그것을 호출할 수 있어야 합니다.

\aosafigure[144pt]{blockcode-images/block.png}{예제 블록}{500l.blockcode.block}

블록은 두 가지 선택적 구조 비트를 가집니다. 단일 숫자 매개변수(기본값 포함)를 가질 수 있고, 다른 블록들을 위한 컨테이너가 될 수 있습니다. 이들은 작업하기에 엄격한 제한이지만, 더 큰 시스템에서는 완화될 것입니다. Waterbear에서는 매개변수로 전달될 수 있는 표현식 블록들도 있으며; 다양한 타입의 다중 매개변수가 지원됩니다. 여기 엄격한 제약의 땅에서는 단 하나의 매개변수 타입으로 무엇을 할 수 있는지 보겠습니다.

```html
<!-- The HTML structure of a block -->
<div class="block" draggable="true" data-name="Right">
    Right
    <input type="number" value="5">
    degrees
</div>
```

메뉴의 블록과 스크립트의 블록 사이에는 실질적인 구별이 없다는 점을 주목하는 것이 중요합니다. 드래그는 어디서 드래그되는지에 따라 약간 다르게 처리하고, 스크립트를 실행할 때는 스크립트 영역의 블록들만 보지만, 근본적으로는 같은 구조입니다. 이는 메뉴에서 스크립트로 드래그할 때 블록을 복제할 수 있다는 의미입니다.

`createBlock(name, value, contents)` 함수는 모든 내부 요소들로 채워진 DOM 요소로서의 블록을 반환하며, 문서에 삽입할 준비가 되어 있습니다. 이것은 메뉴를 위한 블록을 생성하거나, 파일이나 `localStorage`에 저장된 스크립트 블록을 복원하는 데 사용될 수 있습니다. 이런 식으로 유연하지만, Blockcode "언어"를 위해 특별히 구축되었고 그것에 대해 가정을 합니다. 따라서 값이 있으면 그 값이 숫자 인수를 나타낸다고 가정하고 "number" 타입의 입력을 생성합니다. 이것이 Blockcode의 제한이므로 괜찮지만, 블록을 다른 타입의 인수나 하나 이상의 인수를 지원하도록 확장한다면, 코드가 변경되어야 할 것입니다.

```javascript
    function createBlock(name, value, contents){
        var item = elem('div',
            {'class': 'block', draggable: true, 'data-name': name},
            [name]
        );
        if (value !== undefined && value !== null){
            item.appendChild(elem('input', {type: 'number', value: value}));
        }
        if (Array.isArray(contents)){
            item.appendChild(
                elem('div', {'class': 'container'}, contents.map(function(block){
                return createBlock.apply(null, block);
            })));
        }else if (typeof contents === 'string'){
            // Add units (degrees, etc.) specifier
            item.appendChild(document.createTextNode(' ' + contents));
        }
        return item;
    }
```

블록을 DOM 요소로 처리하기 위한 몇 가지 유틸리티가 있습니다:

- `blockContents(block)`는 컨테이너 블록의 자식 블록들을 검색합니다. 컨테이너 블록에서 호출되면 항상 목록을 반환하고, 단순 블록에서는 항상 null을 반환합니다
- `blockValue(block)`는 블록이 number 타입의 입력 필드를 가지고 있으면 블록 입력의 숫자 값을 반환하고, 블록에 대한 입력 요소가 없으면 null을 반환합니다
- `blockScript(block)`는 블록을 쉽게 복원할 수 있는 형태로 저장하기 위해 JSON으로 직렬화하기에 적합한 구조를 반환합니다
- `runBlocks(blocks)`는 블록 배열의 각 블록을 실행하는 핸들러입니다 

```javascript
    function blockContents(block){
        var container = block.querySelector('.container');
        return container ? [].slice.call(container.children) : null;
    }

    function blockValue(block){
        var input = block.querySelector('input');
        return input ? Number(input.value) : null;
    }

    function blockUnits(block){
        if (block.children.length > 1 &&
            block.lastChild.nodeType === Node.TEXT_NODE &&
            block.lastChild.textContent){
            return block.lastChild.textContent.slice(1);
        }
    }

    function blockScript(block){
        var script = [block.dataset.name];
        var value = blockValue(block);
        if (value !== null){
            script.push(blockValue(block));
        }
        var contents = blockContents(block);
        var units = blockUnits(block);
        if (contents){script.push(contents.map(blockScript));}
        if (units){script.push(units);}
        return script.filter(function(notNull){ return notNull !== null; });
    }

    function runBlocks(blocks){
        blocks.forEach(function(block){ trigger('run', block); });
    }
```

### `drag.js`

`drag.js`의 목적은 뷰의 메뉴 섹션과 스크립트 섹션 간의 상호작용을 구현하여 정적인 HTML 블록을 동적 프로그래밍 언어로 변환하는 것입니다. 사용자는 메뉴에서 스크립트로 블록을 드래그하여 프로그램을 구축하고, 시스템은 스크립트 영역의 블록들을 실행합니다.

HTML5 드래그 앤 드롭을 사용하고 있으며; 필요한 특정 JavaScript 이벤트 핸들러들이 여기에 정의되어 있습니다. (HTML5 드래그 앤 드롭 사용에 대한 자세한 정보는 [Eric Bidleman의 기사](http://www.html5rocks.com/en/tutorials/dnd/basics/)를 참조하세요.) 드래그 앤 드롭에 대한 내장 지원이 있는 것은 좋지만, 일부 이상한 점들과 꽤 주요한 제한사항들이 있습니다. 예를 들어 이 글을 쓰는 시점에서 모바일 브라우저에서는 구현되지 않는다는 점입니다.

파일의 상단에서 몇 가지 변수를 정의합니다. 드래그할 때, 드래그 콜백 댄스의 다른 단계에서 이들을 참조해야 할 것입니다.

```javascript
    var dragTarget = null; // Block we're dragging
    var dragType = null; // Are we dragging from the menu or from the script?
    var scriptBlocks = []; // Blocks in the script, sorted by position
```

드래그가 시작하고 끝나는 위치에 따라, `drop`은 다른 효과를 가집니다:

* 스크립트에서 메뉴로 드래그하면, `dragTarget`을 삭제합니다 (스크립트에서 블록 제거).
* 스크립트에서 스크립트로 드래그하면, `dragTarget`을 이동합니다 (기존 스크립트 블록 이동).
* 메뉴에서 스크립트로 드래그하면, `dragTarget`을 복사합니다 (스크립트에 새 블록 삽입).
* 메뉴에서 메뉴로 드래그하면, 아무것도 하지 않습니다.

`dragStart(evt)` 핸들러 동안 블록이 메뉴에서 복사되는지 스크립트에서(또는 스크립트 내에서) 이동되는지 추적을 시작합니다. 또한 나중에 사용하기 위해 드래그되지 않는 스크립트의 모든 블록 목록을 가져옵니다. `evt.dataTransfer.setData` 호출은 브라우저와 다른 애플리케이션(또는 데스크톱) 간의 드래그에 사용되는데, 우리는 사용하지 않지만 버그를 해결하기 위해 어쨌든 호출해야 합니다.

```javascript
    function dragStart(evt){
        if (!matches(evt.target, '.block')) return;
        if (matches(evt.target, '.menu .block')){
            dragType = 'menu';
        }else{
            dragType = 'script';
        }
        evt.target.classList.add('dragging');
        dragTarget = evt.target;
        scriptBlocks = [].slice.call(
            document.querySelectorAll('.script .block:not(.dragging)'));
        // For dragging to take place in Firefox, we have to set this, even if
        // we don't use it
        evt.dataTransfer.setData('text/html', evt.target.outerHTML);
        if (matches(evt.target, '.menu .block')){
            evt.dataTransfer.effectAllowed = 'copy';
        }else{
            evt.dataTransfer.effectAllowed = 'move';
        }
    }
```

드래그하는 동안, `dragenter`, `dragover`, `dragout` 이벤트들은 유효한 드롭 대상을 강조표시하는 등의 시각적 단서를 추가할 기회를 제공합니다. 이들 중 `dragover`만 사용합니다.

```javascript
    function dragOver(evt){
        if (!matches(evt.target, '.menu, .menu *, .script, .script *, .content')) {
            return;
        }
        // Necessary. Allows us to drop.
        if (evt.preventDefault) { evt.preventDefault(); }
        if (dragType === 'menu'){
            // See the section on the DataTransfer object.
            evt.dataTransfer.dropEffect = 'copy';  
        }else{
            evt.dataTransfer.dropEffect = 'move';
        }
        return false;
    }
```

마우스를 놓으면 `drop` 이벤트를 받습니다. 여기서 마법이 일어납니다. 어디서 드래그했는지(`dragStart`에서 설정) 그리고 어디로 드래그했는지 확인해야 합니다. 그런 다음 필요에 따라 블록을 복사하거나, 이동하거나, 삭제합니다. 블록 로직에서 우리 자신의 용도로 `trigger()`(`util.js`에 정의됨)를 사용하여 일부 커스텀 이벤트를 발생시키므로, 스크립트가 변경될 때 새로 고칠 수 있습니다.

```javascript
    function drop(evt){
        if (!matches(evt.target, '.menu, .menu *, .script, .script *')) return;
        var dropTarget = closest(
            evt.target, '.script .container, .script .block, .menu, .script');
        var dropType = 'script';
        if (matches(dropTarget, '.menu')){ dropType = 'menu'; }
        // stops the browser from redirecting.
        if (evt.stopPropagation) { evt.stopPropagation(); }
        if (dragType === 'script' && dropType === 'menu'){
            trigger('blockRemoved', dragTarget.parentElement, dragTarget);
            dragTarget.parentElement.removeChild(dragTarget);
        }else if (dragType ==='script' && dropType === 'script'){
            if (matches(dropTarget, '.block')){
                dropTarget.parentElement.insertBefore(
                    dragTarget, dropTarget.nextSibling);
            }else{
                dropTarget.insertBefore(dragTarget, dropTarget.firstChildElement);
            }
            trigger('blockMoved', dropTarget, dragTarget);
        }else if (dragType === 'menu' && dropType === 'script'){
            var newNode = dragTarget.cloneNode(true);
            newNode.classList.remove('dragging');
            if (matches(dropTarget, '.block')){
                dropTarget.parentElement.insertBefore(
                    newNode, dropTarget.nextSibling);
            }else{
                dropTarget.insertBefore(newNode, dropTarget.firstChildElement);
            }
            trigger('blockAdded', dropTarget, newNode);
        }
    }
```


`dragEnd(evt)`는 마우스를 뗄 때 호출되지만, `drop` 이벤트를 처리한 후에 호출됩니다. 여기서 정리하고, 요소에서 클래스를 제거하고, 다음 드래그를 위해 것들을 재설정할 수 있습니다.

```javascript
    function _findAndRemoveClass(klass){
        var elem = document.querySelector('.' + klass);
        if (elem){ elem.classList.remove(klass); }
    }

    function dragEnd(evt){
        _findAndRemoveClass('dragging');
        _findAndRemoveClass('over');
        _findAndRemoveClass('next');
    }
```

### `menu.js`

`menu.js` 파일은 블록이 실행될 때 호출되는 함수들과 연결되는 곳이며, 사용자가 구축하는 스크립트를 실제로 실행하는 코드를 포함합니다. 스크립트가 수정될 때마다 자동으로 다시 실행됩니다.

여기서 "Menu"는 대부분의 애플리케이션에서처럼 드롭다운(또는 팝업) 메뉴가 아니라, 스크립트를 위해 선택할 수 있는 블록들의 목록입니다. 이 파일이 그것을 설정하고, 일반적으로 유용한(따라서 거북이 언어 자체의 일부가 아닌) 루핑 블록으로 메뉴를 시작합니다. 이것은 다른 곳에 맞지 않을 수 있는 것들을 위한 일종의 잡다한 파일입니다.

무작위 함수들을 모으는 단일 파일을 갖는 것은 특히 아키텍처가 개발 중일 때 유용합니다. 깨끗한 집을 유지하는 나의 이론은 어수선한 것들을 위한 지정된 장소를 갖는 것이며, 이는 프로그램 아키텍처를 구축하는 데도 적용됩니다. 하나의 파일이나 모듈이 아직 명확한 위치가 없는 것들을 위한 만능 수용소가 됩니다. 이 파일이 커질수록 떠오르는 패턴을 관찰하는 것이 중요합니다: 몇 개의 관련된 함수들이 별도의 모듈로 분리되거나 (또는 더 일반적인 함수로 결합될 수) 있습니다. 만능 수용소가 무한정 커지는 것을 원하지 않고, 코드를 조직하는 올바른 방법을 알아낼 때까지의 임시 보관소이기만 하면 됩니다.

`menu`와 `script`에 대한 참조를 계속 유지합니다. 왜냐하면 자주 사용하기 때문입니다; DOM을 계속 반복해서 찾을 필요가 없습니다. 또한 메뉴의 블록 스크립트를 저장하는 `scriptRegistry`도 사용할 것입니다. 같은 이름을 가진 여러 메뉴 블록이나 블록 이름 변경을 지원하지 않는 매우 간단한 이름-스크립트 매핑을 사용합니다. 더 복잡한 스크립팅 환경에서는 더 견고한 것이 필요할 것입니다.

`scriptDirty`를 사용하여 마지막으로 실행된 이후 스크립트가 수정되었는지 추적하므로, 계속해서 실행하려고 시도하지 않습니다.

```javascript
    var menu = document.querySelector('.menu');
    var script = document.querySelector('.script');
    var scriptRegistry = {};
    var scriptDirty = false;
```

다음 프레임 핸들러 동안 스크립트를 실행하도록 시스템에 알리고 싶을 때, `scriptDirty` 플래그를 `true`로 설정하는 `runSoon()`을 호출합니다. 시스템은 모든 프레임에서 `run()`을 호출하지만, `scriptDirty`가 설정되지 않으면 즉시 반환됩니다. `scriptDirty`가 설정되면, 모든 스크립트 블록을 실행하고, 스크립트가 실행되기 전후에 필요한 작업을 특정 언어가 처리할 수 있도록 이벤트를 트리거합니다. 이는 블록을 재사용 가능하게(또는 언어를 플러그 가능하게, 어떻게 보느냐에 따라) 만들기 위해 블록-툴킷을 거북이 언어에서 분리합니다.

스크립트 실행의 일부로, 각 블록을 반복하며 `runEach(evt)`를 호출하는데, 이는 블록에 클래스를 설정한 다음 연결된 함수를 찾아 실행합니다. 속도를 늦추면, 각 블록이 실행될 때를 보여주기 위해 강조표시되면서 코드가 실행되는 것을 볼 수 있어야 합니다.

아래의 `requestAnimationFrame` 메서드는 애니메이션을 위해 브라우저에서 제공됩니다. 호출이 이루어진 후 브라우저에 의해 렌더링될 다음 프레임(초당 60프레임)에 호출될 함수를 받습니다. 실제로 얻는 프레임 수는 해당 호출에서 얼마나 빨리 작업을 완료할 수 있느냐에 달려 있습니다.

```javascript
    function runSoon(){ scriptDirty = true; }

    function run(){
        if (scriptDirty){
            scriptDirty = false;
            Block.trigger('beforeRun', script);
            var blocks = [].slice.call(
                document.querySelectorAll('.script > .block'));
            Block.run(blocks);
            Block.trigger('afterRun', script);
        }else{
            Block.trigger('everyFrame', script);
        }
        requestAnimationFrame(run);
    }
    requestAnimationFrame(run);

    function runEach(evt){
        var elem = evt.target;
        if (!matches(elem, '.script .block')) return;
        if (elem.dataset.name === 'Define block') return;
        elem.classList.add('running');
        scriptRegistry[elem.dataset.name](elem);
        elem.classList.remove('running');
    }
```

`menuItem(name, fn, value, contents)`를 사용하여 메뉴에 블록을 추가하는데, 이는 일반적인 블록을 받아서 함수와 연결하고 메뉴 열에 넣습니다.

```javascript
    function menuItem(name, fn, value, units){
        var item = Block.create(name, value, units);
        scriptRegistry[name] = fn;
        menu.appendChild(item);
        return item;
    }
```

`repeat(block)`을 거북이 언어 외부인 여기에 정의하는 이유는 다른 언어들에서 일반적으로 유용하기 때문입니다. 조건문과 변수 읽기/쓰기를 위한 블록들이 있다면 여기나 별도의 언어 간 모듈에 넣을 수도 있지만, 현재는 이런 범용 블록 중 하나만 정의되어 있습니다.

```javascript
    function repeat(block){
        var count = Block.value(block);
        var children = Block.contents(block);
        for (var i = 0; i < count; i++){
            Block.run(children);
        }
    }
    menuItem('Repeat', repeat, 10, []);
```


### `turtle.js`

`turtle.js`는 거북이 블록 언어의 구현입니다. 코드의 나머지 부분에 함수를 노출하지 않으므로, 다른 것들이 그것에 의존할 수 없습니다. 이런 방식으로 하나의 파일을 교체하여 새로운 블록 언어를 만들 수 있고 코어에서 아무것도 깨지지 않을 것을 알 수 있습니다.

\aosafigure[240pt]{blockcode-images/turtle_example.png}{Turtle 코드 실행 예제}{500l.blockcode.turtle}

거북이 프로그래밍은 Logo에 의해 처음 대중화된 그래픽 프로그래밍 스타일로, 펜을 들고 다니는 상상의 거북이가 화면을 걷는 것입니다. 거북이에게 펜을 들라고(그리기를 멈추지만 여전히 이동), 펜을 내려놓으라고(가는 곳마다 선을 남기며), 몇 걸음 앞으로 이동하라고, 또는 몇 도 회전하라고 말할 수 있습니다. 이러한 명령들만으로도, 루핑과 결합하면, 놀랍도록 복잡한 이미지를 만들 수 있습니다.

이 버전의 거북이 그래픽에서는 몇 가지 추가 블록이 있습니다. 기술적으로 `turn right`와 `turn left` 모두가 필요하지 않습니다. 왜냐하면 하나를 가지고 음수로 다른 하나를 얻을 수 있기 때문입니다. 마찬가지로 `move back`은 `move forward`와 음수로 할 수 있습니다. 이 경우 둘 다 갖는 것이 더 균형잡힌 느낌이었습니다.

위의 이미지는 다른 루프 안에 두 개의 루프를 넣고 각 루프에 `move forward`와 `turn right`를 추가한 다음, 결과 이미지가 마음에 들 때까지 매개변수를 대화식으로 조작하여 형성되었습니다.

```javascript
    var PIXEL_RATIO = window.devicePixelRatio || 1;
    var canvasPlaceholder = document.querySelector('.canvas-placeholder');
    var canvas = document.querySelector('.canvas');
    var script = document.querySelector('.script');
    var ctx = canvas.getContext('2d');
    var cos = Math.cos, sin = Math.sin, sqrt = Math.sqrt, PI = Math.PI;
    var DEGREE = PI / 180;
    var WIDTH, HEIGHT, position, direction, visible, pen, color;
```

`reset()` 함수는 모든 상태 변수를 기본값으로 지웁니다. 여러 거북이를 지원한다면, 이러한 변수들은 객체에 캡슐화될 것입니다. 또한 `deg2rad(deg)` 유틸리티가 있는데, UI에서는 도 단위로 작업하지만 그리기는 라디안으로 하기 때문입니다. 마지막으로, `drawTurtle()`은 거북이 자체를 그립니다. 기본 거북이는 단순히 삼각형이지만, 더 미학적으로 보기 좋은 거북이를 그리도록 오버라이드할 수 있습니다.

`drawTurtle`이 거북이 그리기를 구현하기 위해 정의하는 것과 같은 원시 연산을 사용한다는 점에 주목하세요. 때때로 서로 다른 추상화 계층에서 코드를 재사용하고 싶지 않을 수 있지만, 의미가 명확할 때는 코드 크기와 성능에 큰 이득이 될 수 있습니다.

```javascript
    function reset(){
        recenter();
        direction = deg2rad(90); // facing "up"
        visible = true;
        pen = true; // when pen is true we draw, otherwise we move without drawing
        color = 'black';
    }

    function deg2rad(degrees){ return DEGREE * degrees; }

    function drawTurtle(){
        var userPen = pen; // save pen state
        if (visible){
            penUp(); _moveForward(5); penDown();
            _turn(-150); _moveForward(12);
            _turn(-120); _moveForward(12);
            _turn(-120); _moveForward(12);
            _turn(30);
            penUp(); _moveForward(-5);
            if (userPen){
                penDown(); // restore pen state
            }
        }
    }
```

현재 마우스 위치에 주어진 반지름으로 원을 그리는 특수 블록이 있습니다. `drawCircle`을 특별히 처리하는 이유는 `MOVE 1 RIGHT 1`을 360번 반복하여 원을 그릴 수 있지만, 그런 방식으로는 원의 크기를 제어하기가 매우 어렵기 때문입니다.

```javascript
    function drawCircle(radius){
        // Math for this is from http://www.mathopenref.com/polygonradius.html
        var userPen = pen; // save pen state
        if (visible){
            penUp(); _moveForward(-radius); penDown();
            _turn(-90);
            var steps = Math.min(Math.max(6, Math.floor(radius / 2)), 360);
            var theta = 360 / steps;
            var side = radius * 2 * Math.sin(Math.PI / steps);
            _moveForward(side / 2);
            for (var i = 1; i < steps; i++){
                _turn(theta); _moveForward(side);
            }
            _turn(theta); _moveForward(side / 2);
            _turn(90);
            penUp(); _moveForward(radius); penDown();
            if (userPen){
                penDown(); // restore pen state
            }
        }
    }
```

우리의 주요 원시 연산은 `moveForward`로, 기초적인 삼각법을 처리하고 펜이 올라와 있는지 내려가 있는지 확인해야 합니다.

```javascript
    function _moveForward(distance){
        var start = position;
        position = {
            x: cos(direction) * distance * PIXEL_RATIO + start.x,
            y: -sin(direction) * distance * PIXEL_RATIO + start.y
        };
        if (pen){
            ctx.lineStyle = color;
            ctx.beginPath();
            ctx.moveTo(start.x, start.y);
            ctx.lineTo(position.x, position.y);
            ctx.stroke();
        }
    }
```

나머지 거북이 명령의 대부분은 위에서 구축한 것들로 쉽게 정의할 수 있습니다.

```javascript
    function penUp(){ pen = false; }
    function penDown(){ pen = true; }
    function hideTurtle(){ visible = false; }
    function showTurtle(){ visible = true; }
    function forward(block){ _moveForward(Block.value(block)); }
    function back(block){ _moveForward(-Block.value(block)); }
    function circle(block){ drawCircle(Block.value(block)); }
    function _turn(degrees){ direction += deg2rad(degrees); }
    function left(block){ _turn(Block.value(block)); }
    function right(block){ _turn(-Block.value(block)); }
    function recenter(){ position = {x: WIDTH/2, y: HEIGHT/2}; }
```

새로 시작하고 싶을 때, `clear` 함수는 모든 것을 시작했던 곳으로 되돌립니다.

```javascript
    function clear(){
        ctx.save();
        ctx.fillStyle = 'white';
        ctx.fillRect(0,0,WIDTH,HEIGHT);
        ctx.restore();
        reset();
        ctx.moveTo(position.x, position.y);
    }
```

이 스크립트가 처음 로드되고 실행될 때, `reset`과 `clear`를 사용하여 모든 것을 초기화하고 거북이를 그립니다.

```javascript
    onResize();
    clear();
    drawTurtle();
```

이제 위의 함수들을 `menu.js`의 `Menu.item` 함수와 함께 사용하여 사용자가 스크립트를 구축할 블록들을 만들 수 있습니다. 이들은 사용자의 프로그램을 만들기 위해 제자리에 드래그됩니다.

```javascript
    Menu.item('Left', left, 5, 'degrees');
    Menu.item('Right', right, 5, 'degrees');
    Menu.item('Forward', forward, 10, 'steps');
    Menu.item('Back', back, 10, 'steps');
    Menu.item('Circle', circle, 20, 'radius');
    Menu.item('Pen up', penUp);
    Menu.item('Pen down', penDown);
    Menu.item('Back to center', recenter);
    Menu.item('Hide turtle', hideTurtle);
    Menu.item('Show turtle', showTurtle);
```

## 배운 교훈들

### 왜 MVC를 사용하지 않는가?

모델-뷰-컨트롤러(MVC)는 80년대 Smalltalk 프로그램에서 좋은 설계 선택이었고 웹 앱에서 어떤 변형이든 작동할 수 있지만, 모든 문제에 올바른 도구는 아닙니다. 모든 상태(MVC의 "모델")는 어떻든 블록 언어의 블록 요소들에 의해 포착되므로, 모델에 대한 다른 필요(예를 들어, 공유된 분산 코드를 편집하는 경우)가 없다면 JavaScript로 그것을 복제하는 것은 거의 이익이 없습니다.

Waterbear의 초기 버전은 JavaScript에서 모델을 유지하고 DOM과 동기화하기 위해 많은 노력을 기울였는데, 코드의 절반 이상과 버그의 90%가 DOM과 모델을 동기화 상태로 유지하는 것 때문이라는 것을 깨달을 때까지였습니다. 중복을 제거함으로써 코드가 더 간단하고 견고해질 수 있었고, DOM 요소에 모든 상태를 두면서 많은 버그들을 개발자 도구에서 DOM을 보기만 해도 찾을 수 있었습니다. 따라서 이 경우 HTML/CSS/JavaScript에서 이미 가지고 있는 것보다 MVC를 더 분리하여 구축하는 것은 거의 이익이 없습니다.

### 장난감 변화가 실제 변화로 이어질 수 있다

내가 작업하는 더 큰 시스템의 작고 엄격하게 범위가 제한된 버전을 구축하는 것은 흥미로운 실험이었습니다. 때때로 큰 시스템에서는 너무 많은 다른 것들에 영향을 미치기 때문에 변경을 주저하게 되는 것들이 있습니다. 작은 장난감 버전에서는 자유롭게 실험할 수 있고 그것을 더 큰 시스템으로 다시 가져갈 수 있는 것들을 배울 수 있습니다. 나에게, 더 큰 시스템은 Waterbear이고 이 프로젝트는 Waterbear가 구조화되는 방식에 엄청난 영향을 미쳤습니다.

#### 작은 실험들이 실패를 괜찮게 만든다

이 간소화된 블록 언어로 할 수 있었던 실험들 중 일부는:

- HTML5 드래그 앤 드롭 사용,
- DOM을 반복하며 연결된 함수를 호출하여 블록을 직접 실행,
- HTML DOM에서 깔끔하게 실행되는 코드 분리,
- 드래그하는 동안 단순화된 히트 테스트,
- 우리만의 작은 벡터 및 스프라이트 라이브러리 구축(게임 블록용), 그리고
- 블록 스크립트를 변경할 때마다 결과가 표시되는 "라이브 코딩".

실험에 관한 것은 성공할 필요가 없다는 것입니다. 우리는 실패가 중요한 학습 수단으로 취급되는 대신 처벌받는 곳에서 우리 작업의 실패와 막다른 길을 얼버무리는 경향이 있지만, 앞으로 나아가려면 실패가 필수적입니다. HTML5 드래그 앤 드롭이 작동하게 했지만, 모바일 브라우저에서 전혀 지원되지 않는다는 사실이 Waterbear에게는 시작할 수 없다는 것을 의미합니다. 코드를 분리하고 블록을 반복하여 코드를 실행하는 것이 너무 잘 작동해서 이미 그런 아이디어들을 Waterbear로 가져오기 시작했고, 테스트와 디버깅에서 훌륭한 개선을 이뤘습니다. 일부 수정을 가한 단순화된 히트 테스트도 Waterbear로 돌아오고 있고, 작은 벡터와 스프라이트 라이브러리들도 마찬가지입니다. 라이브 코딩은 아직 Waterbear에 적용되지 않았지만, 현재 변경 라운드가 안정되면 도입할 수도 있습니다.

#### 우리가 정말로 구축하려고 하는 것은 무엇인가?

더 큰 시스템의 작은 버전을 구축하는 것은 정말 중요한 부분이 무엇인지에 날카로운 초점을 맞춥니다. 목적을 제공하지 않는(또는 더 나쁘게는, 목적에서 주의를 분산시키는) 역사적 이유로 남겨진 비트들이 있는가? 아무도 사용하지 않지만 유지해야 하는 기능들이 있는가? 사용자 인터페이스가 간소화될 수 있는가? 이 모든 것들이 작은 버전을 만드는 동안 묻기에 좋은 질문들입니다. 레이아웃을 재조직하는 것과 같은 급진적인 변화들은 더 복잡한 시스템을 통해 파급되는 결과에 대해 걱정하지 않고 만들어질 수 있고, 심지어 복잡한 시스템의 리팩토링을 안내할 수도 있습니다.

#### 프로그램은 것이 아니라 과정이다

이 프로젝트의 범위에서 실험할 수 없었던 것들이 있는데, 미래에 blockcode 코드베이스를 사용하여 테스트해볼 수도 있습니다. 기존 블록들로부터 새로운 블록을 생성하는 "function" 블록을 만드는 것은 흥미로울 것입니다. 실행 취소/다시 실행을 구현하는 것은 제약된 환경에서 더 간단할 것입니다. 복잡성을 급격히 확장하지 않으면서 블록이 다중 인수를 받아들이도록 만드는 것은 유용할 것입니다. 그리고 온라인에서 블록 스크립트를 공유하는 다양한 방법을 찾는 것은 도구의 웹 특성을 완전한 원으로 만들 것입니다.
