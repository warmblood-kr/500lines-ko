title: 템플릿 엔진
author: Ned Batchelder
<markdown>
_Ned Batchelder는 오랜 경력을 가진 소프트웨어 엔지니어로, 현재 edX에서 세상을 교육하는
오픈 소스 소프트웨어를 구축하는 일을 하고 있습니다. coverage.py의 메인테이너이며,
Boston Python의 오거나이저이자 많은 PyCon에서 강연했습니다.
[http://nedbatchelder.com](http://nedbatchelder.com)에서 블로그를 운영하며,
한때 백악관에서 저녁 식사를 한 적이 있습니다._
</markdown>
## 소개

대부분의 프로그램은 많은 로직과 약간의 텍스트 데이터를 포함합니다.
프로그래밍 언어는 이런 종류의 프로그래밍에 적합하게 설계되었습니다.
하지만 일부 프로그래밍 작업은 약간의 로직만 포함하고, 대량의
텍스트 데이터를 다룹니다. 이런 작업을 위해서는 텍스트 중심 문제에 더 적합한
도구가 필요합니다. 템플릿 엔진이 바로 그런 도구입니다. 이 장에서는
간단한 템플릿 엔진을 구축해보겠습니다.

이러한 텍스트 중심 작업의 가장 일반적인 예는 웹 애플리케이션입니다.
모든 웹 애플리케이션에서 중요한 단계는 브라우저에 제공할 HTML을 생성하는 것입니다.
완전히 정적인 HTML 페이지는 거의 없습니다. 대부분은 사용자의 이름과 같은
최소한의 동적 데이터를 포함합니다. 일반적으로는 제품 목록, 친구들의
뉴스 업데이트 등과 같은 대량의 동적 데이터를 포함합니다.

동시에 모든 HTML 페이지는 대량의 정적 텍스트를 포함합니다.
이러한 페이지들은 크기가 크며, 수만 바이트의 텍스트를 포함합니다.
웹 애플리케이션 개발자는 해결해야 할 문제가 있습니다: 정적 데이터와 동적 데이터가
혼합된 대용량 문자열을 어떻게 가장 효율적으로 생성할 것인가? 문제를 더 복잡하게 만드는 것은
정적 텍스트가 실제로는 팀의 다른 구성원인 프론트엔드 디자이너가 작성한 HTML 마크업이라는 점이며,
디자이너는 익숙한 방식으로 작업할 수 있어야 합니다.

설명을 위해 다음과 같은 예제 HTML을 생성하고 싶다고 상상해봅시다:

```html
<p>Welcome, Charlie!</p>
<p>Products:</p>
<ul>
    <li>Apple: $1.00</li>
    <li>Fig: $1.50</li>
    <li>Pomegranate: $3.25</li>
</ul>
```

여기서 사용자의 이름은 동적이며, 제품의 이름과 가격도 마찬가지입니다.
제품의 개수도 고정되어 있지 않습니다: 다른 순간에는
표시할 제품이 더 많거나 적을 수 있습니다.

이 HTML을 만드는 한 가지 방법은 코드에 문자열 상수를 두고,
이들을 조합하여 페이지를 생성하는 것입니다. 동적 데이터는 어떤 형태의
문자열 치환을 통해 삽입될 것입니다. 우리의 동적 데이터 중 일부는 반복적입니다.
제품 목록처럼요. 이는 반복되는 HTML 청크가 있다는 것을 의미하며,
따라서 이들은 별도로 처리되어 페이지의 나머지 부분과 결합되어야 합니다.

이 방식으로 예제 페이지를 생성하면 다음과 같을 것입니다:

```python
# The main HTML for the whole page.
PAGE_HTML = """
<p>Welcome, {name}!</p>
<p>Products:</p>
<ul>
{products}
</ul>
"""

# The HTML for each product displayed.
PRODUCT_HTML = "<li>{prodname}: {price}</li>\n"

def make_page(username, products):
    product_html = ""
    for prodname, price in products:
        product_html += PRODUCT_HTML.format(
            prodname=prodname, price=format_price(price))
    html = PAGE_HTML.format(name=username, products=product_html)
    return html
```

이 방법은 작동하지만, 우리에게는 골치 아픈 문제가 있습니다. HTML이 애플리케이션 코드에
포함된 여러 문자열 상수로 분산되어 있습니다. 정적 텍스트가 별도의 조각으로 분리되어 있어서
페이지의 로직을 파악하기 어렵습니다. 데이터가 어떻게 형식화되는지에 대한 세부사항이
Python 코드에 묻혀 있습니다. HTML 페이지를 수정하기 위해서는 프론트엔드 디자이너가
HTML 변경을 위해 Python 코드를 편집할 수 있어야 합니다. 페이지가 열 배(또는 백 배)
더 복잡해진다면 어떤 모습일지 상상해보세요. 곧 작업할 수 없게 될 것입니다.


## 템플릿

HTML 페이지를 생성하는 더 나은 방법은 *템플릿*을 사용하는 것입니다. HTML 페이지는
템플릿으로 작성되며, 이는 파일이 대부분 정적 HTML이고 특별한 표기법을 사용하여
동적 부분이 포함된 것을 의미합니다. 위의 예제 페이지는 템플릿으로 다음과 같이
작성될 수 있습니다:

```html
<p>Welcome, {{user_name}}!</p>
<p>Products:</p>
<ul>
{% for product in product_list %}
    <li>{{ product.name }}:
        {{ product.price|format_price }}</li>
{% endfor %}
</ul>
```

여기서 포커스는 HTML 텍스트에 있으며, 로직이 HTML에 포함되어 있습니다.
이러한 문서 중심 접근 방식을 위의 로직 중심 코드와 비교해보세요.
이전의 프로그램은 대부분이 Python 코드였고, HTML이 Python 로직에
포함되어 있었습니다. 여기서는 프로그램이 대부분 정적 HTML 마크업입니다.

템플릿에서 사용되는 대부분 정적인 스타일은 대부분의 프로그래밍 언어가
작동하는 방식과 반대입니다. 예를 들어, Python의 경우 소스 파일의 대부분은
실행 가능한 코드이며, 리터럴 정적 텍스트가 필요하면 문자열 리터럴에
포함시킵니다:

```python
def hello():
    print("Hello, world!")

hello()
```

Python이 이 소스 파일을 읽을 때, `def hello():`와 같은 텍스트를
실행할 명령어로 해석합니다. `print("Hello, world!")`의 이중 따옴표 문자는
닫는 이중 따옴표까지 다음 텍스트가 리터럴로 의도되었다는 것을 나타냅니다.
이것이 대부분의 프로그래밍 언어가 작동하는 방식입니다:
대부분 동적이고, 일부 정적 조각들이 명령어에 포함되어 있습니다.
정적 조각들은 이중 따옴표 표기법으로 나타납니다.

템플릿 언어는 이를 뒤집습니다: 템플릿 파일은 대부분 정적 리터럴 텍스트이고,
실행 가능한 동적 부분을 나타내는 특별한 표기법이 있습니다.

```html
<p>Welcome, {{user_name}}!</p>
```

여기서 텍스트는 결과 HTML 페이지에 리터럴로 나타나도록 의도되어 있으며,
'`{{`'는 동적 모드로의 전환을 나타내고, 여기서 `user_name` 변수가
출력으로 치환될 것입니다.

Python의 `"foo = {foo}!".format(foo=17)`과 같은 문자열 포맷팅 함수는
문자열 리터럴과 삽입될 데이터로부터 텍스트를 생성하는 데 사용되는
미니 언어의 예입니다. 템플릿은 이 아이디어를 확장하여 조건문이나
루프와 같은 구조를 포함하지만, 차이는 정도의 문제일 뿐입니다.

이러한 파일들은 비슷한 구조지만 세부사항이 다른 많은 페이지를
생성하는 데 사용되기 때문에 템플릿이라고 불립니다.

프로그램에서 HTML 템플릿을 사용하려면 *템플릿 엔진*이 필요합니다:
페이지의 구조와 정적 콘텐츠를 설명하는 정적 템플릿과 템플릿에 삽입할
동적 데이터를 제공하는 동적 *컨텍스트*를 받는 함수입니다.
템플릿 엔진은 템플릿과 컨텍스트를 결합하여 완전한 HTML 문자열을 생성합니다.
템플릿 엔진의 역할은 템플릿을 해석하여 동적 부분을 실제 데이터로 교체하는 것입니다.

참고로, 템플릿 엔진에는 종종 HTML에 특별한 것이 없으며,
어떤 텍스트 결과든 생성하는 데 사용될 수 있습니다. 예를 들어,
일반 텍스트 이메일 메시지를 생성하는 데도 사용됩니다. 하지만 보통은
HTML에 사용되며, 때때로 HTML에서 어떤 문자가 특별한지 걱정하지 않고
값을 HTML에 삽입할 수 있게 해주는 이스케이핑과 같은 HTML 전용 기능을 갖습니다.


## 지원되는 문법

템플릿 엔진은 지원하는 문법이 다양합니다. 우리의 템플릿 문법은
인기 있는 웹 프레임워크인 Django를 기반으로 합니다. 우리는 Python에서
엔진을 구현하고 있으므로, 일부 Python 개념이 우리 문법에 나타날 것입니다.
이미 이 장의 상단에 있는 예제에서 이 문법의 일부를 보았지만,
여기서는 우리가 구현할 모든 문법에 대한 간단한 요약입니다.

컨텍스트의 데이터는 이중 중괄호를 사용하여 삽입됩니다:

```html
<p>Welcome, {{user_name}}!</p>
```

템플릿에 사용할 수 있는 데이터는 템플릿이 렌더링될 때 컨텍스트에서 제공됩니다.
이에 대해서는 나중에 자세히 설명하겠습니다.

템플릿 엔진은 일반적으로 단순화되고 유연한 문법을 사용하여 데이터 내의 요소에
액세스를 제공합니다. Python에서는 이 표현식들이 모두 다른 효과를 가집니다:

```python
dict["key"]
obj.attr
obj.method()
```

우리의 템플릿 문법에서는 이 모든 연산이 점으로 표현됩니다:

```
dict.key
obj.attr
obj.method
```

점은 객체 속성이나 딕셔너리 값에 액세스하며,
결과 값이 호출 가능한 경우 자동으로 호출됩니다. 이는
이러한 연산에 대해 다른 문법을 사용해야 하는 Python 코드와 다릅니다.
이렇게 하면 더 간단한 템플릿 문법이 됩니다:

```html
<p>The price is: {{product.price}}, with a {{product.discount}}% discount.</p>
```

값을 수정하기 위해 _필터_라고 불리는 함수를 사용할 수 있습니다.
필터는 파이프 문자로 호출됩니다:

```html
<p>Short name: {{story.subject|slugify|lower}}</p>
```

흥미로운 페이지를 구축하려면 일반적으로 최소한의 의사결정이 필요하므로
조건문을 사용할 수 있습니다:

```html
{% if user.is_logged_in %}
    <p>Welcome, {{ user.name }}!</p>
{% endif %}
```

루프를 사용하면 페이지에 데이터 컬렉션을 포함할 수 있습니다:

```html
<p>Products:</p>
<ul>
{% for product in product_list %}
    <li>{{ product.name }}: {{ product.price|format_price }}</li>
{% endfor %}
</ul>
```

다른 프로그래밍 언어와 마찬가지로 조건문과 루프는 중첩되어
복잡한 논리적 구조를 구축할 수 있습니다.

마지막으로, 템플릿을 문서화할 수 있도록 주석은
중괄호-해시 사이에 나타납니다:

```html
{# This is the best template ever! #}
```


## 구현 방법

대략적으로 말하면, 템플릿 엔진은 두 가지 주요 단계를 갖습니다: 템플릿을 _파싱_하는 것과
그 다음 템플릿을 _렌더링_하는 것입니다.

구체적으로 템플릿 렌더링에는 다음이 포함됩니다:

* 동적 컨텍스트(데이터의 소스) 관리
* 로직 요소 실행
* 점 접근과 필터 실행 구현

파싱 단계에서 렌더링 단계로 무엇을 전달할지가 핵심 질문입니다.
파싱은 렌더링할 수 있는 무엇을 생성할까요?
두 가지 주요 옵션이 있습니다. 다른 언어 구현에서 사용하는 용어를 느슨하게 빌려와서
*해석*과 *컴파일*이라고 부르겠습니다.

해석 모델에서는 파싱이 템플릿의 구조를 나타내는 데이터 구조를 생성합니다.
렌더링 단계는 그 데이터 구조를 순회하며, 발견한 명령어를 기반으로
결과 텍스트를 조립합니다. 실제 사례로는 Django 템플릿 엔진이 이 방법을 사용합니다.

컴파일 모델에서는 파싱이 직접 실행 가능한 코드 형태를 생성합니다.
렌더링 단계는 그 코드를 실행하여 결과를 생성합니다. Jinja2와 Mako는
컴파일 접근 방식을 사용하는 템플릿 엔진의 두 가지 예입니다.

우리의 엔진 구현은 컴파일을 사용합니다: 템플릿을 Python 코드로 컴파일합니다.
실행될 때, Python 코드가 결과를 조립합니다.

여기서 설명하는 템플릿 엔진은 원래 HTML 보고서를 생성하기 위해
coverage.py의 일부로 작성되었습니다. coverage.py에서는 템플릿이 몇 개뿐이지만,
같은 템플릿에서 많은 파일을 생성하기 위해 반복적으로 사용됩니다.
전반적으로 템플릿을 Python 코드로 컴파일하면 프로그램이 더 빠르게 실행됩니다.
컴파일 과정이 좀 더 복잡하긴 하지만 한 번만 실행하면 되고,
컴파일된 코드의 실행은 여러 번 실행되면서 데이터 구조를 여러 번 해석하는 것보다 빠르기 때문입니다.

템플릿을 Python으로 컴파일하는 것은 좀 더 복잡하지만,
생각만큼 나쁘지는 않습니다. 게다가 모든 개발자가 말할 수 있듯이,
프로그램을 작성하는 프로그램을 작성하는 것이 프로그램을 작성하는 것보다 더 재미있습니다!

우리의 템플릿 컴파일러는 코드 생성이라고 불리는 일반적인 기법의 작은 예입니다.
코드 생성은 프로그래밍 언어 컴파일러를 포함하여 많은 강력하고 유연한 도구의 기반이 됩니다.
코드 생성은 복잡해질 수 있지만, 도구 상자에 갖고 있으면 유용한 기법입니다.

템플릿의 다른 응용에서는 각 템플릿이 몇 번만 사용될 경우
해석 접근 방식을 선호할 수 있습니다. 그러면 Python으로 컴파일하는 노력이
장기적으로 보상받지 못할 것이고, 더 간단한 해석 과정이
전반적으로 더 나은 성능을 발휘할 수 있습니다.


## Python으로 컴파일하기

템플릿 엔진의 코드를 살펴보기 전에, 그것이 생성하는 코드를 먼저 살펴봅시다.
파싱 단계는 템플릿을 Python 함수로 변환할 것입니다.
다시 우리의 작은 템플릿입니다:

```html
<p>Welcome, {{user_name}}!</p>
<p>Products:</p>
<ul>
{% for product in product_list %}
    <li>{{ product.name }}:
        {{ product.price|format_price }}</li>
{% endfor %}
</ul>
```

우리 엔진은 이 템플릿을 Python 코드로 컴파일할 것입니다. 결과로 나오는 Python
코드는 약간 더 빠른 코드를 생성하는 몇 가지 단축키를 선택했기 때문에
특이하게 보입니다. 다음은 Python 코드입니다(가독성을 위해 약간 재구성됨):

```python
def render_function(context, do_dots):
    c_user_name = context['user_name']
    c_product_list = context['product_list']
    c_format_price = context['format_price']

    result = []
    append_result = result.append
    extend_result = result.extend
    to_str = str

    extend_result([
        '<p>Welcome, ',
        to_str(c_user_name),
        '!</p>\n<p>Products:</p>\n<ul>\n'
    ])
    for c_product in c_product_list:
        extend_result([
            '\n    <li>',
            to_str(do_dots(c_product, 'name')),
            ':\n        ',
            to_str(c_format_price(do_dots(c_product, 'price'))),
            '</li>\n'
        ])
    append_result('\n</ul>\n')
    return ''.join(result)
```

각 템플릿은 컨텍스트라고 불리는 데이터 딕셔너리를 받는 `render_function` 함수로 변환됩니다.
함수의 본문은 컨텍스트에서 데이터를 지역 이름으로 언패킹하는 것으로 시작하는데,
반복 사용에서 더 빠르기 때문입니다. 모든 컨텍스트 데이터는 `c_` 접두사를 가진 지역 변수로 들어가므로
충돌 걱정 없이 다른 지역 이름을 사용할 수 있습니다.

템플릿의 결과는 문자열이 될 것입니다. 부분들로부터 문자열을 구축하는 가장 빠른 방법은
문자열 리스트를 생성하고 마지막에 그것들을 함께 조인하는 것입니다.
`result`는 문자열 리스트가 될 것입니다. 이 리스트에 문자열을 추가할 것이므로,
`result_append`와 `result_extend`라는 지역 이름으로 `append`와 `extend` 메서드를 캡처합니다.
마지막으로 생성하는 지역 변수는 `str` 내장 함수를 위한 `to_str` 단축키입니다.

이런 종류의 단축키는 특이합니다. 더 자세히 살펴봅시다. Python에서
`result.append("hello")`와 같은 객체의 메서드 호출은 두 단계로 실행됩니다.
먼저, result 객체에서 append 속성을 가져옵니다: `result.append`.
그런 다음 가져온 값이 함수로 호출되며 `"hello"` 인수를 전달합니다.
이 단계들이 함께 수행되는 것에 익숙하지만, 실제로는 별개입니다.
첫 번째 단계의 결과를 저장하면, 저장된 값에 두 번째 단계를 수행할 수 있습니다.
따라서 이 두 Python 코드 조각은 같은 일을 합니다:

```python
# The way we're used to seeing it:
result.append("hello")

# But this works the same:
append_result = result.append
append_result("hello")
```

템플릿 엔진 코드에서는 두 번째 단계를 몇 번 수행하든 첫 번째 단계를 한 번만 수행하도록
이렇게 분리했습니다. append 속성을 찾는 데 드는 시간을 피할 수 있어서
작은 시간을 절약할 수 있습니다.

이것은 마이크로 최적화의 예입니다: 속도의 미세한 향상을 얻는 특이한 코딩 기법입니다.
마이크로 최적화는 가독성이 떨어지거나 더 혼란스러울 수 있으므로,
입증된 성능 병목 지점인 코드에서만 정당화됩니다. 개발자들은 마이크로 최적화가 얼마나
정당한지에 대해 의견이 분분하며, 일부 초보자들은 과도하게 사용합니다.
여기서의 최적화는 성능을 향상시킨다는 것을 보여주는 타이밍 실험 후에만 추가되었습니다.
비록 조금일지라도 말입니다. 마이크로 최적화는 Python의 일부 특이한 측면을 사용하므로
교육적일 수 있지만, 자신의 코드에서 과도하게 사용하지는 마세요.

`str`에 대한 단축키도 마이크로 최적화입니다. Python의 이름은 함수에 대해 지역,
모듈에 대해 전역, 또는 Python에 대해 내장일 수 있습니다. 지역 이름을 찾는 것이
전역이나 내장을 찾는 것보다 빠릅니다. `str`은 항상 사용할 수 있는 내장 함수라는 사실에
익숙하지만, Python은 여전히 사용될 때마다 `str`이라는 이름을 찾아야 합니다.
지역 변수에 넣으면 지역 변수가 내장 함수보다 빠르기 때문에 또 다른 작은 시간을 절약할 수 있습니다.

이러한 단축키들이 정의되면, 우리의 특정 템플릿에서 생성된 Python 라인들을 위한 준비가 됩니다.
문자열은 추가할 문자열이 하나인지 여러 개인지에 따라 `append_result` 또는 `extend_result`
단축키를 사용하여 결과 리스트에 추가될 것입니다. 템플릿의 리터럴 텍스트는
간단한 문자열 리터럴이 됩니다.

append와 extend를 모두 갖는 것은 복잡성을 추가하지만, 우리는 템플릿의 가장 빠른 실행을
목표로 하고 있으며, 하나의 항목에 extend를 사용한다는 것은 extend에 전달할 수 있도록
하나의 항목으로 새 리스트를 만든다는 뜻입니다.

`{{ ... }}`의 표현식은 계산되고, 문자열로 변환되어 결과에 추가됩니다.
표현식의 점은 우리 함수로 전달된 `do_dots` 함수에 의해 처리되는데,
점 표현식의 의미가 컨텍스트의 데이터에 따라 달라지기 때문입니다:
속성 접근이거나 항목 접근일 수 있고, 호출 가능할 수도 있습니다.

논리 구조 `{% if ... %}`와 `{% for ... %}`는 Python 조건문과 루프로 변환됩니다.
`{% if/for ... %}` 태그의 표현식은 `if` 또는 `for` 문의 표현식이 되고,
`{% end... %}` 태그까지의 내용은 문의 본문이 됩니다.


<!-- [[[cog from cogutil import include ]]] -->
<!-- [[[end]]] -->


## 엔진 작성하기

이제 엔진이 무엇을 할지 이해했으므로, 구현을 살펴보겠습니다.


### Templite 클래스

템플릿 엔진의 핵심은 Templite 클래스입니다. (이해하시겠죠? 템플릿이지만 라이트입니다!)

Templite 클래스는 작은 인터페이스를 가집니다. 템플릿의 텍스트로 Templite 객체를 생성하고,
나중에 그것의 `render` 메서드를 사용하여 특정 컨텍스트(데이터 딕셔너리)를
템플릿을 통해 렌더링할 수 있습니다:

```python
# Make a Templite object.
templite = Templite('''
    <h1>Hello {{name|upper}}!</h1>
    {% for topic in topics %}
        <p>You are interested in {{topic}}.</p>
    {% endfor %}
    ''',
    {'upper': str.upper},
)

# Later, use it to render some data.
text = templite.render({
    'name': "Ned",
    'topics': ['Python', 'Geometry', 'Juggling'],
})
```

객체가 생성될 때 템플릿의 텍스트를 전달하여 컴파일 단계를 한 번만 수행하고,
나중에 `render`를 여러 번 호출하여 컴파일된 결과를 재사용할 수 있습니다.

생성자는 또한 값들의 딕셔너리인 초기 컨텍스트를 받습니다. 이들은 Templite 객체에
저장되고, 템플릿이 나중에 렌더링될 때 사용할 수 있습니다. 이는 이전 예제의 `upper`와 같이
어디서나 사용할 수 있기를 원하는 함수나 상수를 정의하는 데 좋습니다.

Templite의 구현을 논의하기 전에, 먼저 정의해야 할 도우미가 있습니다: CodeBuilder입니다.


### CodeBuilder

우리 엔진의 작업 대부분은 템플릿을 파싱하고 필요한 Python 코드를 생성하는 것입니다.
Python 생성을 돕기 위해 우리는 Python 코드를 구성할 때 부기를 처리해주는
CodeBuilder 클래스를 가지고 있습니다. 이 클래스는 코드 라인을 추가하고, 들여쓰기를 관리하며,
마지막으로 컴파일된 Python에서 값을 제공합니다.

하나의 CodeBuilder 객체는 완전한 Python 코드 청크에 대해 책임집니다.
우리 템플릿 엔진에서 사용될 때, Python 청크는 항상 하나의 완전한 함수 정의입니다.
하지만 CodeBuilder 클래스는 하나의 함수만 있을 것이라고 가정하지 않습니다.
이렇게 하면 CodeBuilder 코드가 더 일반적이고 나머지 템플릿 엔진 코드와
덜 결합됩니다.

보게 되겠지만, 우리는 또한 중첩된 CodeBuilder를 사용하여 거의 완료될 때까지
무엇이 될지 모르더라도 함수의 시작 부분에 코드를 넣을 수 있게 합니다.

CodeBuilder 객체는 함께 최종 Python 코드가 될 문자열 리스트를 보관합니다.
필요한 유일한 다른 상태는 현재 들여쓰기 레벨입니다:

<!-- [[[cog include("templite.py", first="class CodeBuilder", numblanks=2) ]]] -->
```python
class CodeBuilder(object):
    """Build source code conveniently."""

    def __init__(self, indent=0):
        self.code = []
        self.indent_level = indent
```
<!-- [[[end]]] -->

CodeBuilder doesn't do much. `add_line` adds a new line of code, which
automatically indents the text to the current indentation level, and supplies a
newline:

<!-- [[[cog include("templite.py", first="def add_line", numblanks=3, dedent=False) ]]] -->
```python
    def add_line(self, line):
        """Add a line of source to the code.

        Indentation and newline will be added for you, don't provide them.

        """
        self.code.extend([" " * self.indent_level, line, "\n"])
```
<!-- [[[end]]] -->

`indent`와 `dedent`는 들여쓰기 레벨을 증가시키거나 감소시킵니다:

<!-- [[[cog include("templite.py", first="INDENT_STEP = 4", numblanks=3, dedent=False) ]]] -->
```python
    INDENT_STEP = 4      # PEP8 says so!

    def indent(self):
        """Increase the current indent for following lines."""
        self.indent_level += self.INDENT_STEP

    def dedent(self):
        """Decrease the current indent for following lines."""
        self.indent_level -= self.INDENT_STEP
```
<!-- [[[end]]] -->

`add_section`은 다른 CodeBuilder 객체에 의해 관리됩니다. 이를 통해
코드의 한 지점에 대한 참조를 유지하고, 나중에 텍스트를 추가할 수 있습니다.
`self.code` 리스트는 대부분 문자열 리스트이지만, 이러한 섹션들에 대한 참조도 보유합니다:

<!-- [[[cog include("templite.py", first="def add_section", numblanks=1, dedent=False) ]]] -->
```python
    def add_section(self):
        """Add a section, a sub-CodeBuilder."""
        section = CodeBuilder(self.indent_level)
        self.code.append(section)
        return section
```
<!-- [[[end]]] -->

`__str__`은 모든 코드가 포함된 단일 문자열을 생성합니다. 이는
단순히 `self.code`의 모든 문자열을 함께 조인합니다. `self.code`가 섹션을 포함할 수 있으므로,
이는 다른 `CodeBuilder` 객체를 재귀적으로 호출할 수 있음에 주목하세요:

<!-- [[[cog include("templite.py", first="def __str__", numblanks=1, dedent=False) ]]] -->
```python
    def __str__(self):
        return "".join(str(c) for c in self.code)
```
<!-- [[[end]]] -->

`get_globals`는 코드를 실행하여 최종 값을 생성합니다. 이는 객체를 문자열화하고,
그것을 실행하여 정의를 얻고, 결과 값을 반환합니다:

<!-- [[[cog include("templite.py", first="def get_globals", numblanks=1, dedent=False) ]]] -->
```python
    def get_globals(self):
        """Execute the code, and return a dict of globals it defines."""
        # A check that the caller really finished all the blocks they started.
        assert self.indent_level == 0
        # Get the Python source as a single string.
        python_source = str(self)
        # Execute the source, defining globals, and return them.
        global_namespace = {}
        exec(python_source, global_namespace)
        return global_namespace
```
<!-- [[[end]]] -->

이 마지막 메서드는 Python의 일부 특이한 기능을 사용합니다. `exec` 함수는
Python 코드를 포함하는 문자열을 실행합니다. `exec`의 두 번째 인수는
코드에 의해 정의된 전역 변수들을 수집할 딕셔너리입니다. 따라서
예를 들어, 이렇게 한다면:

```python
python_source = """\
SEVENTEEN = 17

def three():
    return 3
"""
global_namespace = {}
exec(python_source, global_namespace)
```

그러면 `global_namespace['SEVENTEEN']`은 17이고, `global_namespace['three']`은
`three`라는 이름의 실제 함수입니다.

우리는 CodeBuilder를 하나의 함수만 생성하는 데 사용하지만, 여기에는
그 사용을 제한하는 것이 없습니다. 이렇게 하면 클래스가 구현하기 더 간단하고
이해하기 더 쉬워집니다.

CodeBuilder를 사용하면 Python 소스 코드 청크를 생성할 수 있으며, 우리 템플릿 엔진에 대한
특정 지식이 전혀 없습니다. Python에서 세 개의 다른 함수가 정의되도록 사용할 수 있고,
그러면 `get_globals`가 세 개의 값, 즉 세 개의 함수의 딕셔너리를 반환할 것입니다.
우연히도 우리 템플릿 엔진은 하나의 함수만 정의하면 됩니다. 하지만 그 구현 세부사항을
템플릿 엔진 코드에 유지하고 CodeBuilder 클래스에서 제외하는 것이 더 나은 소프트웨어 설계입니다.

실제로 사용하는 방식&mdash;단일 함수를 정의하는&mdash;에서도 `get_globals`가
딕셔너리를 반환하도록 하면 정의한 함수의 이름을 알 필요가 없기 때문에
코드가 더 모듈화됩니다. Python 소스에서 어떤 함수 이름을 정의하든,
`get_globals`에서 반환된 딕셔너리에서 그 이름을 검색할 수 있습니다.

이제 Templite 클래스 자체의 구현에 들어가서 CodeBuilder가 어떻게 그리고 어디서 사용되는지
살펴볼 수 있습니다.


### Templite 클래스 구현

우리 코드의 대부분은 Templite 클래스에 있습니다. 논의했듯이, 컴파일 단계와 렌더링 단계를 모두 가집니다.


#### 컴파일하기

템플릿을 Python 함수로 컴파일하는 모든 작업은 Templite 생성자에서 발생합니다.
먼저 컨텍스트들이 저장됩니다:

<!-- [[[cog include("templite.py", first="def __init__(self, text, ", numblanks=3, dedent=False) ]]] -->
```python
    def __init__(self, text, *contexts):
        """Construct a Templite with the given `text`.

        `contexts` are dictionaries of values to use for future renderings.
        These are good for filters and global values.

        """
        self.context = {}
        for context in contexts:
            self.context.update(context)
```
<!-- [[[end]]] -->

매개변수로 `*contexts`를 사용한 것에 주목하세요. 별표는 임의의 수의
위치 인수가 튜플로 패킹되어 `contexts`로 전달됨을 나타냅니다. 이것을
인수 언패킹이라고 하며, 호출자가 여러 개의 다른 컨텍스트 딕셔너리를 제공할 수 있음을 의미합니다.
이제 다음 호출들이 모두 유효합니다:

```python
t = Templite(template_text)
t = Templite(template_text, context1)
t = Templite(template_text, context1, context2)
```

컨텍스트 인수들(있는 경우)은 컨텍스트의 튜플로 생성자에 제공됩니다.
그런 다음 `contexts` 튜플을 반복하면서 각각을 차례로 처리할 수 있습니다.
우리는 단순히 제공된 모든 컨텍스트의 내용을 가진 `self.context`라는
하나의 결합된 딕셔너리를 생성합니다. 컨텍스트에서 중복 이름이 제공되면
마지막 것이 우선됩니다.

컴파일된 함수를 가능한 한 빠르게 만들기 위해, 컨텍스트 변수를 Python 지역 변수로 추출합니다.
우리가 만나는 변수 이름 집합을 유지하여 그 이름들을 얻을 것이지만, 또한
템플릿에 정의된 변수들의 이름, 즉 루프 변수들도 추적해야 합니다:

<!-- [[[cog include("templite.py", first="self.all_vars", numblanks=1, dedent=False) ]]] -->
```python
        self.all_vars = set()
        self.loop_vars = set()
```
<!-- [[[end]]] -->

나중에 이들이 우리 함수의 프롤로그 구성을 돕는 데 어떻게 사용되는지 볼 것입니다.
먼저, 앞서 작성한 CodeBuilder 클래스를 사용하여 컴파일된 함수 구축을 시작하겠습니다:

<!-- [[[cog include("templite.py", first="code = CodeBuilder", numblanks=2, dedent=False) ]]] -->
```python
        code = CodeBuilder()

        code.add_line("def render_function(context, do_dots):")
        code.indent()
        vars_code = code.add_section()
        code.add_line("result = []")
        code.add_line("append_result = result.append")
        code.add_line("extend_result = result.extend")
        code.add_line("to_str = str")
```
<!-- [[[end]]] -->

여기서 CodeBuilder 객체를 구성하고, 라인을 작성하기 시작합니다. 우리 Python 함수는
`render_function`이라고 불릴 것이며, 두 개의 인수를 받습니다:
`context`는 사용해야 하는 데이터 딕셔너리이고, `do_dots`는 점 속성 접근을 구현하는 함수입니다.

여기서 컨텍스트는 Templite 생성자로 전달된 데이터 컨텍스트와 렌더 함수로 전달된
데이터 컨텍스트의 결합입니다. 이는 Templite 생성자에서 만든 템플릿에 사용할 수 있는
완전한 데이터 집합입니다.

CodeBuilder가 매우 간단하다는 점에 주목하세요: 함수 정의에 대해 "알지" 못하고,
단지 코드 라인에 대해서만 압니다. 이렇게 하면 구현과 사용 모두에서 CodeBuilder가 간단해집니다.
너무 많은 특화된 CodeBuilder를 정신적으로 보간할 필요 없이 여기서 생성된 코드를 읽을 수 있습니다.

`vars_code`라는 섹션을 만듭니다. 나중에 변수 추출 라인을 그 섹션에 작성할 것입니다.
`vars_code` 객체를 사용하면 필요한 정보가 있을 때 나중에 채울 수 있는 함수의
위치를 저장할 수 있습니다.

그런 다음 네 개의 고정 라인이 작성되어, 결과 리스트, 그 리스트에 추가하거나 확장하는
메서드의 단축키, 그리고 `str()` 내장 함수의 단축키를 정의합니다.
앞서 논의했듯이, 이 특이한 단계는 렌더링 함수에서 조금 더 많은 성능을 짜냅니다.

`append`와 `extend` 단축키를 모두 가지는 이유는 결과에 추가할 라인이 하나인지
여러 개인지에 따라 가장 효과적인 방법을 사용할 수 있기 때문입니다.

다음으로 출력 문자열 버퍼링을 돕는 내부 함수를 정의합니다:

<!-- [[[cog include("templite.py", first="buffered =", numblanks=1, dedent=False) ]]] -->
```python
        buffered = []
        def flush_output():
            """Force `buffered` to the code builder."""
            if len(buffered) == 1:
                code.add_line("append_result(%s)" % buffered[0])
            elif len(buffered) > 1:
                code.add_line("extend_result([%s])" % ", ".join(buffered))
            del buffered[:]
```
<!-- [[[end]]] -->

컴파일된 함수에 들어가야 하는 출력 청크를 생성할 때, 우리 결과에 추가하는
함수 호출로 변환해야 합니다. 반복되는 추가 호출을 하나의 확장 호출로 결합하고 싶습니다.
이것도 또 다른 마이크로 최적화입니다. 이를 가능하게 하기 위해 청크를 버퍼링합니다.

`buffered` 리스트는 아직 우리 함수 소스 코드에 작성되지 않은 문자열들을 보관합니다.
템플릿 컴파일이 진행되면서 `buffered`에 문자열을 추가하고, if문이나
루프의 시작 또는 끝과 같은 제어 흐름 지점에 도달할 때 함수 소스로 플러시할 것입니다.

`flush_output` 함수는 *클로저*인데, 이는 자신 밖의 변수를 참조하는 함수에 대한 멋진 용어입니다.
여기서 `flush_output`은 `buffered`와 `code`를 참조합니다. 이렇게 하면 함수 호출이 간단해집니다:
`flush_output`에게 어떤 버퍼를 플러시할지, 어디로 플러시할지 알려줄 필요가 없습니다;
그것은 그 모든 것을 암묵적으로 압니다.

하나의 문자열만 버퍼링된 경우, `append_result` 단축키가 사용되어 결과에 추가됩니다.
둘 이상이 버퍼링된 경우, `extend_result` 단축키가 모든 것과 함께 사용되어
결과에 추가됩니다. 그런 다음 버퍼된 리스트가 지워져서 더 많은 문자열이 버퍼링될 수 있습니다.

나머지 컴파일 코드는 `buffered`에 추가하여 함수에 라인을 추가하고,
결국 `flush_output`을 호출하여 CodeBuilder에 작성할 것입니다.

이 함수가 제자리에 있으면, 컴파일러에서 다음과 같은 코드 라인을 가질 수 있습니다:

```python
buffered.append("'hello'")
```

이는 우리의 컴파일된 Python 함수가 다음 라인을 갖는다는 의미입니다:

```python
append_result('hello')
```

이는 템플릿의 렌더링된 출력에 `hello` 문자열을 추가할 것입니다. 여기서는 여러 수준의
추상화가 있어서 명확하게 유지하기 어려울 수 있습니다. 컴파일러는
`buffered.append("'hello'")`를 사용하며, 이는 컴파일된 Python 함수에서 `append_result('hello')`를 생성하고,
실행될 때 템플릿 결과에 `hello`를 추가합니다.

Templite 클래스로 돌아가봅시다. 제어 구조를 파싱할 때, 제대로 중첩되어 있는지 확인하고 싶습니다.
`ops_stack` 리스트는 문자열의 스택입니다:

<!-- [[[cog include("templite.py", first="ops_stack", numblanks=1, dedent=False) ]]] -->
```python
        ops_stack = []
```
<!-- [[[end]]] -->

`{% if .. %}` 태그를 만나면(예를 들어), 스택에 `'if'`를 푸시할 것입니다.
`{% endif %}` 태그를 찾으면, 스택을 팝하고 스택의 맨 위에 `'if'`가 없으면 오류를 보고할 수 있습니다.

이제 실제 파싱이 시작됩니다. 정규 표현식 또는 *regex*를 사용하여 템플릿 텍스트를
여러 토큰으로 분할합니다. 정규식은 복잡한 패턴 매칭을 위한 매우 간결한 표기법이라서
벅찰 수 있습니다. 패턴을 매치하는 복잡성이 자신의 Python 코드가 아닌 정규 표현식 엔진에서
C로 구현되므로 매우 효율적이기도 합니다. 다음이 우리의 정규식입니다:

<!-- [[[cog include("templite.py", first="tokens =", numblanks=1, dedent=False) ]]] -->
```python
        tokens = re.split(r"(?s)({{.*?}}|{%.*?%}|{#.*?#})", text)
```
<!-- [[[end]]] -->

이것은 복잡해 보입니다; 분석해봅시다.

`re.split` 함수는 정규식을 사용하여 문자열을 분할합니다. 우리 패턴이 괄호로 묶여 있으므로,
매치가 문자열을 분할하는 데 사용되고, 분할 리스트의 조각으로도 반환됩니다.
우리 패턴은 태그 문법과 일치하지만, 문자열이 태그에서 분할되고 태그도 반환되도록
괄호로 묶었습니다.

정규식의 `(?s)` 플래그는 점이 개행 문자와도 일치해야 함을 의미합니다.
다음으로 세 가지 대안의 괄호로 묶인 그룹이 있습니다: `{{.*?}}`는 표현식과 일치하고,
`{%.*?%}`는 태그와 일치하며, `{#.*?#}`는 주석과 일치합니다.
이 모든 것에서 우리는 `.*?`를 사용하여 임의의 수의 문자와 일치시키지만,
일치하는 가장 짧은 시퀀스를 사용합니다.

`re.split`의 결과는 문자열 리스트입니다. 예를 들어, 이 템플릿 텍스트는:

```html
<p>Topics for {{name}}: {% for t in topics %}{{t}}, {% endfor %}</p>
```

다음 조각들로 분할될 것입니다:

```python
[
    '<p>Topics for ',               # literal
    '{{name}}',                     # expression
    ': ',                           # literal
    '{% for t in topics %}',        # tag
    '',                             # literal (empty)
    '{{t}}',                        # expression
    ', ',                           # literal
    '{% endfor %}',                 # tag
    '</p>'                          # literal
]
```

텍스트가 이렇게 토큰으로 분할되면, 토큰을 반복하여 각각을 차례로 처리할 수 있습니다.
유형에 따라 분할하여 각 유형을 별도로 처리할 수 있습니다.

컴파일 코드는 이 토큰들에 대한 루프입니다:

<!-- [[[cog include("templite.py", first="for token", numlines=1, dedent=False) ]]] -->
```python
        for token in tokens:
```
<!-- [[[end]]] -->

각 토큰을 검사하여 네 가지 경우 중 어느 것인지 확인합니다. 처음 두 문자만 보면 충분합니다.
첫 번째 경우는 주석인데, 처리하기 쉽습니다: 그냥 무시하고 다음 토큰으로 넘어가면 됩니다:

<!-- [[[cog include("templite.py", first="if token.", numlines=3, dedent=False) ]]] -->
```python
            if token.startswith('{#'):
                # Comment: ignore it and move on.
                continue
```
<!-- [[[end]]] -->

`{{...}}` 표현식의 경우, 앞뒤의 두 중괄호를 잘라내고, 공백을 제거한 다음,
전체 표현식을 `_expr_code`로 전달합니다:

<!-- [[[cog include("templite.py", first="elif token.startswith('{{')", numlines=4, dedent=False) ]]] -->
```python
            elif token.startswith('{{'):
                # An expression to evaluate.
                expr = self._expr_code(token[2:-2].strip())
                buffered.append("to_str(%s)" % expr)
```
<!-- [[[end]]] -->

`_expr_code` 메서드는 템플릿 표현식을 Python 표현식으로 컴파일할 것입니다.
그 함수는 나중에 볼 것입니다. 표현식의 값을 문자열로 강제하기 위해 `to_str` 함수를 사용하고,
그것을 우리 결과에 추가합니다.

세 번째 경우는 중요한 것입니다: `{% ... %}` 태그들입니다. 이들은 Python 제어 구조가 될
제어 구조입니다. 먼저 버퍼된 출력 라인을 플러시해야 하고, 그런 다음 태그에서 단어 리스트를 추출합니다:

<!-- [[[cog include("templite.py", first="elif token.startswith('{%')", numlines=4, dedent=False) ]]] -->
```python
            elif token.startswith('{%'):
                # Action tag: split into words and parse further.
                flush_output()
                words = token[2:-2].strip().split()
```
<!-- [[[end]]] -->

이제 태그의 첫 번째 단어를 기반으로 세 가지 하위 경우가 있습니다: `if`, `for`, 또는 `end`.
`if` 경우는 우리의 간단한 오류 처리와 코드 생성을 보여줍니다:

<!-- [[[cog include("templite.py", first="if words[0] == 'if'", numlines=7, dedent=False) ]]] -->
```python
                if words[0] == 'if':
                    # An if statement: evaluate the expression to determine if.
                    if len(words) != 2:
                        self._syntax_error("Don't understand if", token)
                    ops_stack.append('if')
                    code.add_line("if %s:" % self._expr_code(words[1]))
                    code.indent()
```
<!-- [[[end]]] -->

`if` 태그는 단일 표현식을 가져야 하므로, `words` 리스트에는 두 개의 요소만 있어야 합니다.
그렇지 않으면 `_syntax_error` 도우미 메서드를 사용하여 구문 오류 예외를 발생시킵니다.
`endif` 태그를 확인할 수 있도록 `ops_stack`에 `'if'`를 푸시합니다.
`if` 태그의 표현식 부분은 `_expr_code`로 Python 표현식으로 컴파일되고,
Python `if` 문의 조건부 표현식으로 사용됩니다.

두 번째 태그 유형은 `for`인데, 이는 Python `for` 문으로 컴파일될 것입니다:

<!-- [[[cog include("templite.py", first="elif words[0] == 'for'", numlines=13, dedent=False) ]]] -->
```python
                elif words[0] == 'for':
                    # A loop: iterate over expression result.
                    if len(words) != 4 or words[2] != 'in':
                        self._syntax_error("Don't understand for", token)
                    ops_stack.append('for')
                    self._variable(words[1], self.loop_vars)
                    code.add_line(
                        "for c_%s in %s:" % (
                            words[1],
                            self._expr_code(words[3])
                        )
                    )
                    code.indent()
```
<!-- [[[end]]] -->

구문을 확인하고 스택에 `'for'`를 푸시합니다. `_variable` 메서드는 변수의 구문을 확인하고,
우리가 제공하는 집합에 추가합니다. 이것이 컴파일 중에 모든 변수의 이름을 수집하는 방법입니다.
나중에 컨텍스트에서 얻는 모든 변수 이름을 언패킹하는 함수의 프롤로그를 작성해야 할 것입니다.
이를 올바르게 수행하려면 우리가 만난 모든 변수의 이름인 `self.all_vars`와
루프에 의해 정의된 모든 변수의 이름인 `self.loop_vars`를 알아야 합니다.

함수 소스에 한 라인, `for` 문을 추가합니다. 우리의 모든 템플릿 변수는 `c_`를 앞에 붙여
Python 변수로 바뀝니다. 그래서 Python 함수에서 사용하는 다른 이름과 충돌하지 않을 것임을 압니다.
`_expr_code`를 사용하여 템플릿에서 반복 표현식을 Python의 반복 표현식으로 컴파일합니다.

우리가 처리하는 마지막 종류의 태그는 `end` 태그입니다; `{% endif %}` 또는 `{% endfor %}`입니다.
컴파일된 함수 소스에 미치는 효과는 같습니다: 단순히 앞서 시작된 `if` 또는 `for` 문을 끝내기 위해
들여쓰기를 해제하는 것입니다:

<!-- [[[cog include("templite.py", first="elif words[0].startswith('end')", numlines=11, dedent=False) ]]] -->
```python
                elif words[0].startswith('end'):
                    # Endsomething.  Pop the ops stack.
                    if len(words) != 1:
                        self._syntax_error("Don't understand end", token)
                    end_what = words[0][3:]
                    if not ops_stack:
                        self._syntax_error("Too many ends", token)
                    start_what = ops_stack.pop()
                    if start_what != end_what:
                        self._syntax_error("Mismatched end tag", end_what)
                    code.dedent()
```
<!-- [[[end]]] -->

여기서 end 태그에 필요한 실제 작업은 한 라인입니다: 함수 소스의 들여쓰기를 해제하는 것입니다.
이 절의 나머지는 모두 템플릿이 올바르게 형성되었는지 확인하는 오류 검사입니다.
이것은 프로그램 번역 코드에서 흔하지 않은 일이 아닙니다.

오류 처리에 대해 말하자면, 태그가 `if`, `for`, 또는 `end`가 아니라면,
그것이 무엇인지 모르므로 구문 오류를 발생시킵니다:

<!-- [[[cog include("templite.py", first="else:", numlines=2, dedent=False) ]]] -->
```python
                else:
                    self._syntax_error("Don't understand tag", words[0])
```
<!-- [[[end]]] -->

세 가지 다른 특수 문법(`{{...}}`, `{#...#}`, `{%...%}`)을 완료했습니다.
남은 것은 리터럴 콘텐츠입니다. `repr` 내장 함수를 사용하여 토큰에 대한 Python 문자열 리터럴을 생성하여
리터럴 문자열을 버퍼된 출력에 추가할 것입니다:

<!-- [[[cog include("templite.py", first="else:", after="Don't understand tag", numblanks=1, dedent=False) ]]] -->
```python
            else:
                # Literal content.  If it isn't empty, output it.
                if token:
                    buffered.append(repr(token))
```
<!-- [[[end]]] -->

`repr`을 사용하지 않았다면, 컴파일된 함수에서 다음과 같은 라인을 갖게 될 것입니다:

```python
append_result(abc)      # Error! abc isn't defined
```

값이 다음과 같이 따옴표로 묶여야 합니다:

```python
append_result('abc')
```

`repr` 함수는 우리를 위해 문자열 주위에 따옴표를 제공하며,
필요한 곳에 백슬래시도 제공합니다:

```python
append_result('"Don\'t you like my hat?" he asked.')
```

출력에 빈 문자열을 추가하는 의미가 없으므로 `if token:`으로 토큰이 빈 문자열인지 먼저 확인함에 주목하세요.
우리의 정규식이 태그 문법으로 분할하고 있기 때문에, 인접한 태그들 사이에는 빈 토큰이 있을 것입니다.
여기서의 확인은 컴파일된 함수에 쓸모없는 `append_result("")` 문을 넣는 것을 피하는 쉬운 방법입니다.

그것으로 템플릿의 모든 토큰에 대한 루프가 완료됩니다. 루프가 끝나면, 모든 템플릿이 처리된 것입니다.
마지막으로 확인해야 할 것이 하나 있습니다: `ops_stack`이 비어있지 않다면, end 태그가 누락된 것입니다.
그런 다음 버퍼된 출력을 함수 소스로 플러시합니다:

<!-- [[[cog include("templite.py", first="if ops_stack:", numblanks=2, dedent=False) ]]] -->
```python
        if ops_stack:
            self._syntax_error("Unmatched action tag", ops_stack[-1])

        flush_output()
```
<!-- [[[end]]] -->

함수의 시작 부분에 섹션을 생성했었습니다. 그 역할은 컨텍스트에서 템플릿 변수를
Python 지역 변수로 언패킹하는 것이었습니다. 이제 전체 템플릿을 처리했으므로,
모든 변수의 이름을 알고 있어서 이 프롤로그에 라인을 작성할 수 있습니다.

정의해야 할 이름이 무엇인지 알기 위해 약간의 작업을 해야 합니다. 샘플 템플릿을 보면:

```html
<p>Welcome, {{user_name}}!</p>
<p>Products:</p>
<ul>
{% for product in product_list %}
    <li>{{ product.name }}:
        {{ product.price|format_price }}</li>
{% endfor %}
</ul>
```

여기서 사용되는 변수는 `user_name`과 `product` 두 개입니다. `all_vars` 집합은
둘 다 `{{...}}` 표현식에 사용되므로 이 두 이름을 모두 가질 것입니다.
하지만 `product`는 루프에 의해 정의되므로 프롤로그에서 컨텍스트에서 추출해야 하는 것은
`user_name`뿐입니다.

템플릿에서 사용되는 모든 변수는 `all_vars` 집합에 있고, 템플릿에서 정의되는 모든 변수는
`loop_vars`에 있습니다. `loop_vars`의 모든 이름은 루프에서 사용되므로
이미 코드에서 정의되었습니다. 따라서 `loop_vars`에 없는 `all_vars`의 모든 이름을 언패킹해야 합니다:

<!-- [[[cog include("templite.py", first="for var_name", numblanks=1, dedent=False) ]]] -->
```python
        for var_name in self.all_vars - self.loop_vars:
            vars_code.add_line("c_%s = context[%r]" % (var_name, var_name))
```
<!-- [[[end]]] -->

각 이름은 함수의 프롤로그에서 한 라인이 되어, 컨텍스트 변수를 적절히 이름 지어진 지역 변수로 언패킹합니다.

템플릿을 Python 함수로 컴파일하는 작업이 거의 완료되었습니다. 우리 함수는 `result`에 문자열을 추가해왔으므로,
함수의 마지막 라인은 단순히 그것들을 모두 함께 조인하여 반환하는 것입니다:

<!-- [[[cog include("templite.py", first='code.add_line("return', numlines=2, dedent=False) ]]] -->
```python
        code.add_line("return ''.join(result)")
        code.dedent()
```
<!-- [[[end]]] -->

이제 컴파일된 Python 함수의 소스 작성을 완료했으므로,
CodeBuilder 객체에서 함수 자체를 얻어야 합니다. `get_globals` 메서드는
우리가 조립해온 Python 코드를 실행합니다. 우리 코드는 함수 정의(`def render_function(..):`로 시작)라는 점을
기억하세요. 따라서 코드를 실행하면 `render_function`을 정의하지만 `render_function`의 본문은 실행하지 않습니다.

`get_globals`의 결과는 코드에서 정의된 값들의 딕셔너리입니다.
그것에서 `render_function` 값을 가져와서 Templite 객체의 속성으로 저장합니다:

<!-- [[[cog include("templite.py", first="self._render_function =", numlines=1, dedent=False) ]]] -->
```python
        self._render_function = code.get_globals()['render_function']
```
<!-- [[[end]]] -->

이제 `self._render_function`은 호출 가능한 Python 함수입니다. 나중에 렌더링 단계에서 사용할 것입니다.


#### 표현식 컴파일하기

컴파일 과정의 중요한 부분을 아직 보지 못했습니다: 템플릿 표현식을 Python 표현식으로 컴파일하는
`_expr_code` 메서드입니다. 우리 템플릿 표현식은 단일 이름만큼 간단할 수 있습니다:

```
{{user_name}}
```

또는 속성 접근과 필터의 복잡한 시퀀스일 수 있습니다:

```
{{user.name.localized|upper|escape}}
```

우리의 `_expr_code` 메서드는 이 모든 가능성을 처리할 것입니다. 모든 언어의 표현식과 마찬가지로,
우리 것도 재귀적으로 구축됩니다: 큰 표현식은 더 작은 표현식들로 구성됩니다.
전체 표현식은 파이프로 분리되어 있고, 첫 번째 조각은 점으로 분리되어 있는 식입니다.
따라서 우리 함수는 자연스럽게 재귀 형태를 취합니다:

<!-- [[[cog include("templite.py", first="def _expr_code", numlines=2, dedent=False) ]]] -->
```python
    def _expr_code(self, expr):
        """Generate a Python expression for `expr`."""
```
<!-- [[[end]]] -->

고려할 첫 번째 경우는 표현식에 파이프가 있는 것입니다. 있다면,
파이프 조각들의 리스트로 분할합니다. 첫 번째 파이프 조각은 Python 표현식으로 변환하기 위해
`_expr_code`에 재귀적으로 전달됩니다.

<!-- [[[cog include("templite.py", first="if ", after="def _expr_code", numlines=6, dedent=False) ]]] -->
```python
        if "|" in expr:
            pipes = expr.split("|")
            code = self._expr_code(pipes[0])
            for func in pipes[1:]:
                self._variable(func, self.all_vars)
                code = "c_%s(%s)" % (func, code)
```
<!-- [[[end]]] -->

나머지 파이프 조각들 각각은 함수의 이름입니다. 값은 최종 값을 생성하기 위해 함수를 통과합니다.
각 함수 이름은 프롤로그에서 적절히 추출할 수 있도록 `all_vars`에 추가되는 변수입니다.

파이프가 없었다면, 점이 있을 수 있습니다. 그렇다면, 점으로 분할합니다.
첫 번째 부분은 Python 표현식으로 변환하기 위해 `_expr_code`에 재귀적으로 전달되고,
그런 다음 각 점 이름이 차례로 처리됩니다:

<!-- [[[cog include("templite.py", first="elif ", after="def _expr_code", numlines=5, dedent=False) ]]] -->
```python
        elif "." in expr:
            dots = expr.split(".")
            code = self._expr_code(dots[0])
            args = ", ".join(repr(d) for d in dots[1:])
            code = "do_dots(%s, %s)" % (code, args)
```
<!-- [[[end]]] -->

점이 어떻게 컴파일되는지 이해하려면, 템플릿의 `x.y`가 작동하는 것에 따라 Python에서
`x['y']` 또는 `x.y` 중 하나를 의미할 수 있음을 기억하세요; 결과가 호출 가능하면 호출됩니다.
이 불확실성은 컴파일 시가 아닌 런타임에 그 가능성들을 시도해야 한다는 것을 의미합니다.
따라서 `x.y.z`를 함수 호출 `do_dots(x, 'y', 'z')`로 컴파일합니다. 점 함수는
다양한 접근 방법을 시도하고 성공한 값을 반환할 것입니다.

`do_dots` 함수는 런타임에 컴파일된 Python 함수로 전달됩니다.
조금 후에 그 구현을 볼 것입니다.

`_expr_code` 함수의 마지막 절은 입력 표현식에 파이프나 점이 없었던 경우를 처리합니다.
그 경우에는 단지 이름입니다. 그것을 `all_vars`에 기록하고, 접두사가 붙은 Python 이름을 사용하여
변수에 접근합니다:

<!-- [[[cog include("templite.py", first="else:", after="def _expr_code", numlines=4, dedent=False) ]]] -->
```python
        else:
            self._variable(expr, self.all_vars)
            code = "c_%s" % expr
        return code
```
<!-- [[[end]]] -->


#### 도우미 함수들

컴파일 중에 몇 가지 도우미 함수를 사용했습니다. `_syntax_error` 메서드는
단순히 좋은 오류 메시지를 조합하고 예외를 발생시킵니다:

<!-- [[[cog include("templite.py", first="def _syntax_error", numblanks=1, dedent=False) ]]] -->
```python
    def _syntax_error(self, msg, thing):
        """Raise a syntax error using `msg`, and showing `thing`."""
        raise TempliteSyntaxError("%s: %r" % (msg, thing))
```
<!-- [[[end]]] -->

`_variable` 메서드는 변수 이름을 검증하고 컴파일 중에 수집한 이름 집합에 추가하는 데 도움을 줍니다.
정규식을 사용하여 이름이 유효한 Python 식별자인지 확인한 다음, 이름을 집합에 추가합니다:

<!-- [[[cog include("templite.py", first="def _variable", numblanks=4, dedent=False) ]]] -->
```python
    def _variable(self, name, vars_set):
        """Track that `name` is used as a variable.

        Adds the name to `vars_set`, a set of variable names.

        Raises an syntax error if `name` is not a valid name.

        """
        if not re.match(r"[_a-zA-Z][_a-zA-Z0-9]*$", name):
            self._syntax_error("Not a valid name", name)
        vars_set.add(name)
```
<!-- [[[end]]] -->

그것으로 컴파일 코드가 완료되었습니다!


#### 렌더링

남은 것은 렌더링 코드를 작성하는 것뿐입니다. 템플릿을 Python 함수로 컴파일했으므로,
렌더링 코드는 할 일이 많지 않습니다. 데이터 컨텍스트를 준비하고,
컴파일된 Python 코드를 호출해야 합니다:

<!-- [[[cog include("templite.py", first="def render(", numblanks=3, dedent=False) ]]] -->
```python
    def render(self, context=None):
        """Render this template by applying it to `context`.

        `context` is a dictionary of values to use in this rendering.

        """
        # Make the complete context we'll use.
        render_context = dict(self.context)
        if context:
            render_context.update(context)
        return self._render_function(render_context, self._do_dots)
```
<!-- [[[end]]] -->

`Templite` 객체를 구성할 때 데이터 컨텍스트로 시작했다는 점을 기억하세요.
여기서 그것을 복사하고, 이 렌더링을 위해 전달된 모든 데이터를 병합합니다.
복사는 연속적인 렌더링 호출이 서로의 데이터를 보지 않도록 하기 위함이고,
병합은 데이터 조회에 사용할 단일 딕셔너리를 갖기 위함입니다. 이것이 템플릿이 구성될 때
제공된 컨텍스트와 렌더 시간에 지금 제공되는 데이터로부터 하나의 통합된 데이터 컨텍스트를
구축하는 방법입니다.

`render`에 전달된 데이터가 Templite 생성자에 전달된 데이터를 덮어쓸 수 있음에 주목하세요.
생성자에 전달된 컨텍스트는 필터 정의와 상수 같은 전역적인 것들을 가지고 있고,
`render`에 전달된 컨텍스트는 그 한 번의 렌더링을 위한 특정 데이터를 가지고 있기 때문에
그런 일은 일반적으로 발생하지 않습니다.

그런 다음 단순히 컴파일된 `render_function`을 호출합니다. 첫 번째 인수는
완전한 데이터 컨텍스트이고, 두 번째 인수는 점 의미론을 구현할 함수입니다.
매번 같은 구현을 사용합니다: 우리 자체의 `_do_dots` 메서드입니다.

<!-- [[[cog include("templite.py", first="def _do_dots", numblanks=1, dedent=False) ]]] -->
```python
    def _do_dots(self, value, *dots):
        """Evaluate dotted expressions at runtime."""
        for dot in dots:
            try:
                value = getattr(value, dot)
            except AttributeError:
                value = value[dot]
            if callable(value):
                value = value()
        return value
```
<!-- [[[end]]] -->

컴파일 중에 `x.y.z`와 같은 템플릿 표현식은 `do_dots(x, 'y', 'z')`로 변환됩니다.
이 함수는 점-이름들을 반복하며, 각각에 대해
속성으로 시도해보고, 실패하면 키로 시도해봅니다. 이것이
우리의 단일 템플릿 문법이 `x.y` 또는 `x['y']` 중 하나로 작동할 수 있는 유연성을 제공합니다.
각 단계에서 새 값이 호출 가능한지도 확인하고, 호출 가능하면 호출합니다.
모든 점-이름을 다 처리하면, 손에 있는 값이 우리가 원하는 값입니다.

여기서 다시 Python 인수 언패킹(`*dots`)을 사용하여 `_do_dots`가
임의의 수의 점 이름을 받을 수 있게 했습니다. 이는 템플릿에서 마주치는
모든 점 표현식에 대해 작동할 유연한 함수를 제공합니다.

`self._render_function`을 호출할 때 점 표현식을 평가하는 데 사용할 함수를 전달하지만,
항상 같은 것을 전달한다는 점에 주목하세요. 그 코드를 컴파일된 템플릿의 일부로 만들 수도 있었지만,
모든 템플릿에 대해 같은 8줄이고, 그 8줄은 템플릿이 작동하는 방법의 정의의 일부이지
특정 템플릿의 세부사항의 일부가 아닙니다. 그 코드가 컴파일된 템플릿의 일부가 되는 것보다는
이렇게 구현하는 것이 더 깔끔하게 느껴집니다.


## 테스트

템플릿 엔진과 함께 모든 동작과 엣지 케이스를 다루는 테스트 스위트가 제공됩니다.
실제로 저는 500줄 제한을 약간 초과했습니다: 템플릿 엔진이 252줄이고 테스트가 275줄입니다.
이것은 잘 테스트된 코드의 일반적인 특징입니다: 제품보다 테스트에 더 많은 코드가 있습니다.


## 빠진 것들

완전한 기능을 갖춘 템플릿 엔진은 우리가 여기서 구현한 것보다 훨씬 더 많은 것을 제공합니다.
이 코드를 작게 유지하기 위해 다음과 같은 흥미로운 아이디어들을 제외했습니다:

* 템플릿 상속과 포함
* 사용자 정의 태그
* 자동 이스케이핑
* 필터에 대한 인수
* else와 elif와 같은 복잡한 조건부 로직
* 둘 이상의 루프 변수를 가진 루프
* 공백 제어

그럼에도 불구하고 우리의 간단한 템플릿 엔진은 유용합니다. 실제로 이것은
coverage.py에서 HTML 보고서를 생성하는 데 사용되는 템플릿 엔진입니다.


## 마무리

252줄로 간단하면서도 유능한 템플릿 엔진을 갖게 되었습니다. 실제 템플릿 엔진들은
훨씬 더 많은 기능을 가지고 있지만, 이 코드는 프로세스의 기본 아이디어를 제시합니다:
템플릿을 Python 함수로 컴파일한 다음, 함수를 실행하여 텍스트 결과를 생성하는 것입니다.
