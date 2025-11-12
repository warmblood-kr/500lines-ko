title: 고고학에서 영감을 받은 데이터베이스
author: Yoav Rubin
<markdown>
_Yoav Rubin은 Microsoft의 선임 소프트웨어 엔지니어이며, 이전에는 IBM Research에서 연구원 및 마스터 인벤터로 활동했습니다. 현재 클라우드 데이터 보안 분야에서 일하고 있으며, 과거에는 클라우드 또는 웹 기반 개발 환경 구축에 집중했습니다. Yoav는 신경과학 분야의 의학 연구 석사 학위와 정보 시스템 공학 학사 학위를 보유하고 있습니다. Twitter에서는 [\@yoavrubin](https://twitter.com/yoavrubin)으로 활동하며, 때때로 [http://yoavrubin.blogspot.com](http://yoavrubin.blogspot.com)에서 블로그를 작성합니다._
</markdown>
## 소개

소프트웨어 개발은 종종 엄격한 프로세스로 여겨지며, 입력은 요구사항이고 출력은 작동하는 제품입니다. 하지만 소프트웨어 개발자들도 사람이며, 그들만의 관점과 편견을 가지고 있어서 이것이 작업 결과에 영향을 미칩니다.

이 장에서는 일반적인 관점의 변화가 잘 연구된 소프트웨어 유형인 데이터베이스의 설계와 구현에 어떤 영향을 미치는지 살펴보겠습니다.

데이터베이스 시스템은 데이터를 저장하고 쿼리하도록 설계됩니다. 이는 모든 정보 작업자가 하는 일이지만, 시스템 자체는 컴퓨터 과학자들에 의해 설계되었습니다. 그 결과, 현대의 데이터베이스 시스템은 데이터가 무엇이고 데이터로 무엇을 할 수 있는지에 대한 컴퓨터 과학자들의 정의에 크게 영향을 받습니다.

예를 들어, 대부분의 현대 데이터베이스는 새 데이터를 추가하고 기존 데이터를 유지하는 대신 기존 데이터를 제자리에서 덮어쓰는 방식으로 업데이트를 구현합니다. [Rich Hickey](http://www.infoq.com/presentations/Value-Values)가 "장소 지향 프로그래밍"이라고 명명한 이 메커니즘은 저장 공간을 절약하지만 특정 레코드의 전체 히스토리를 검색하는 것을 불가능하게 만듭니다. 이러한 설계 결정은 "히스토리"가 저장 비용보다 덜 중요하다는 컴퓨터 과학자의 관점을 반영합니다.

반면에 고고학자에게 기존 데이터가 어디에서 찾을 수 있는지 물어본다면, 답은 "바라건대, 그냥 아래에 묻혀 있을 것입니다"일 것입니다.

(면책조항: 일반적인 고고학자의 견해에 대한 제 이해는 몇 개의 박물관 방문, 여러 위키피디아 글 읽기, 그리고 인디아나 존스 시리즈 전체 시청에 기반합니다.)

### 고고학자처럼 데이터베이스 설계하기

우리의 친근한 고고학자에게 데이터베이스를 설계해 달라고 요청한다면, 요구사항이 발굴 현장에서 발견되는 것들을 반영할 것으로 예상할 수 있습니다:

* 모든 데이터는 현장에서 발견되고 분류됩니다.
* 더 깊이 파면 과거의 상태가 드러날 것입니다.
* 같은 층에서 발견된 유물들은 같은 시대의 것입니다.
* 각 유물은 서로 다른 시대에 축적된 상태로 구성될 것입니다.

예를 들어, 벽 한 층에는 로마 기호가 있을 수 있고, 아래층에는 그리스 기호가 있을 수 있습니다. 이 두 관찰 결과 모두 벽의 상태의 일부로 기록됩니다.

이 비유는 \aosafigref{500l.functionaldb.exc}에서 시각화됩니다:

* 전체 원은 발굴 현장입니다.
* 각 고리는 _층_입니다 (여기서는 0부터 4까지 번호가 매겨집니다).
* 각 조각은 라벨이 붙은 유물입니다 ('A'부터 'E'까지).
* 각 유물은 '기호' 속성을 가집니다 (빈 칸은 업데이트가 이루어지지 않았음을 의미합니다).
* 실선 화살표는 층 간의 기호 변화를 나타냅니다
* 점선 화살표는 유물 간의 관심 있는 임의의 관계입니다 (예: 'E'에서 'A'로).

\aosafigure[240pt]{functionalDB-images/image_0.png}{발굴 현장}{500l.functionaldb.exc}

고고학자의 언어를 데이터베이스 설계자가 사용하는 용어로 번역하면:

* 발굴 현장은 _데이터베이스_입니다.
* 각 유물은 해당 _ID_를 가진 _엔티티_입니다.
* 각 엔티티는 시간이 지남에 따라 변할 수 있는 _속성_들의 집합을 가집니다.
* 각 속성은 특정 시간에 특정 _값_을 가집니다.

이것은 여러분이 작업하는 데 익숙한 데이터베이스와는 매우 다르게 보일 수 있습니다. 이 설계는 함수형 프로그래밍 영역의 아이디어를 사용하기 때문에 때때로 "함수형 데이터베이스"라고 불립니다. 이 장의 나머지 부분에서는 그러한 데이터베이스를 구현하는 방법을 설명합니다.

함수형 데이터베이스를 구축하고 있으므로, Clojure라는 함수형 프로그래밍 언어를 사용할 것입니다.

Clojure는 기본 제공되는 불변성, 고차 함수, 메타프로그래밍 기능과 같이 함수형 데이터베이스의 좋은 구현 언어가 되는 여러 특성을 가지고 있습니다. 하지만 궁극적으로 Clojure가 선택된 이유는 깨끗하고 엄밀한 설계에 대한 강조 때문이며, 이는 몇 안 되는 프로그래밍 언어가 가진 특징입니다. 

## 기초 다지기

데이터베이스를 구성하는 핵심 구조체들을 선언하는 것부터 시작해보겠습니다.

```clojure
(defrecord Database [layers top-id curr-time])
```

데이터베이스는 다음으로 구성됩니다:

1. 각각 고유한 타임스탬프를 가진 엔티티 층들 (그림 1의 고리들).
2. 다음으로 사용 가능한 고유 ID인 top-id 값.
3. 데이터베이스가 마지막으로 업데이트된 시간.


```clojure
(defrecord Layer [storage VAET AVET VEAT EAVT])
```

각 층은 다음으로 구성됩니다:

1. 엔티티를 위한 데이터 저장소.
2. 데이터베이스 쿼리 속도를 높이는 데 사용되는 인덱스들. (이러한 인덱스들과 그 이름의 의미는 나중에 설명됩니다.)

우리의 설계에서, 하나의 개념적 '데이터베이스'는 여러 `Database` 인스턴스로 구성될 수 있으며, 각각은 `curr-time`에서의 데이터베이스 스냅샷을 나타냅니다. `Layer`는 엔티티의 상태가 그들이 나타내는 시간 사이에 변경되지 않았다면 다른 `Layer`와 정확히 같은 엔티티를 공유할 수 있습니다.

### 엔티티

저장할 엔티티가 없다면 데이터베이스는 쓸모가 없으므로, 다음으로 엔티티를 정의합니다. 앞서 논의한 대로, 엔티티는 ID와 속성 목록을 가지며, `make-entity` 함수를 사용하여 생성합니다.

```clojure
(defrecord Entity [id attrs])

(defn make-entity
   ([] (make-entity :db/no-id-yet))
   ([id] (Entity.  id {})))
```

ID가 주어지지 않으면 엔티티의 ID는 `:db/no-id-yet`으로 설정되며, 이는 다른 무언가가 ID를 부여하는 책임을 진다는 의미입니다. 이것이 어떻게 작동하는지는 나중에 보겠습니다.

#### 속성

각 속성은 이름, 값, 그리고 가장 최근 업데이트의 타임스탬프와 그 이전 타임스탬프로 구성됩니다. 각 속성은 또한 `type`과 `cardinality`를 설명하는 두 필드를 가집니다.

속성이 다른 엔티티와의 관계를 나타내는 데 사용되는 경우, 그 `type`은 `:db/ref`이고 값은 관련된 엔티티의 ID가 됩니다. 이 간단한 타입 시스템은 확장 지점 역할도 합니다. 사용자는 자유롭게 자신만의 타입을 정의하고 이를 활용하여 데이터에 추가적인 의미를 제공할 수 있습니다.

속성의 `cardinality`는 속성이 단일 값을 나타내는지 값들의 집합을 나타내는지를 지정합니다. 이 필드를 사용하여 이 속성에 허용되는 연산 집합을 결정합니다.

속성 생성은 `make-attr` 함수를 사용하여 수행됩니다. 

```clojure
(defrecord Attr [name value ts prev-ts])

(defn make-attr
   ([name value type ; these ones are required
       & {:keys [cardinality] :or {cardinality :db/single}} ]
     {:pre [(contains? #{:db/single :db/multiple} cardinality)]}
    (with-meta (Attr. name value -1 -1) {:type type :cardinality cardinality})))
```

이 생성자 함수에서 사용된 몇 가지 흥미로운 패턴들이 있습니다:

* cardinality 매개변수가 허용 가능한 값인지 검증하기 위해 Clojure의 _계약에 의한 설계(Design by Contract)_ 패턴을 사용합니다.
* 값이 주어지지 않은 경우 `:db/single`의 기본값을 제공하기 위해 Clojure의 구조분해(destructuring) 메커니즘을 사용합니다.
* 속성의 데이터(이름, 값, 타임스탬프)와 메타데이터(타입, 카디널리티)를 구분하기 위해 Clojure의 메타데이터 기능을 사용합니다. Clojure에서 메타데이터 처리는 `with-meta`(설정) 및 `meta`(읽기) 함수를 사용하여 수행됩니다.

속성은 엔티티의 일부인 경우에만 의미를 가집니다. `add-attr` 함수로 이 연결을 만들며, 이 함수는 주어진 속성을 엔티티의 속성 맵(`:attrs`라고 함)에 추가합니다.

속성의 이름을 직접 사용하는 대신, Clojure의 관용적인 맵 사용법을 준수하기 위해 먼저 키워드로 변환한다는 점에 주목하세요.

```clojure
(defn add-attr [ent attr]
   (let [attr-id (keyword (:name attr))]
      (assoc-in ent [:attrs attr-id] attr)))
```

### 저장소

지금까지 우리는 _무엇을_ 저장할지에 대해서는 많이 이야기했지만, _어디에_ 저장할지에 대해서는 생각하지 않았습니다. 이 장에서는 가장 간단한 저장 메커니즘인 메모리 내 데이터 저장을 사용합니다. 이는 확실히 신뢰할 수 없지만, 개발과 디버깅을 단순화하고 프로그램의 더 흥미로운 부분에 집중할 수 있게 해줍니다.

간단한 _프로토콜_을 통해 저장소에 접근할 것이며, 이를 통해 데이터베이스 소유자가 선택할 수 있는 추가적인 저장소 제공자를 정의할 수 있게 됩니다.

```clojure
(defprotocol Storage
   (get-entity [storage e-id] )
   (write-entity [storage entity])
   (drop-entity [storage entity]))
```

\noindent 그리고 여기에 맵을 저장소로 사용하는 프로토콜의 인메모리 구현이 있습니다:

```clojure
(defrecord InMemory [] Storage
   (get-entity [storage e-id] (e-id storage))
   (write-entity [storage entity] (assoc storage (:id entity) entity))
   (drop-entity [storage entity] (dissoc storage (:id entity))))
```

### 데이터 인덱싱

데이터베이스의 기본 요소들을 정의했으므로, 이제 쿼리하는 방법에 대해 생각해볼 수 있습니다. 데이터를 구조화한 방식의 특성상, 모든 쿼리는 필연적으로 엔티티의 ID와 그 속성들 중 일부의 이름과 값 중 적어도 하나에 관심을 가질 것입니다. `(entity-id, attribute-name, attribute-value)`의 이 삼중항(triplet)은 쿼리 과정에 충분히 중요해서 명시적인 이름을 부여합니다: _datom_.

Datom은 사실(fact)을 나타내기 때문에 중요하며, 우리의 데이터베이스는 사실을 축적합니다.

이전에 데이터베이스 시스템을 사용해본 적이 있다면, 평균 쿼리 시간을 줄이기 위해 추가 공간을 소비하는 보조 데이터 구조인 _인덱스_의 개념에 이미 익숙할 것입니다. 우리 데이터베이스에서 인덱스는 datom의 구성 요소를 특정 순서로 저장하는 3단계 구조입니다. 각 인덱스는 datom의 구성 요소를 저장하는 순서에서 그 이름을 파생합니다.

예를 들어, \aosafigref{500l.functionaldb.eavt}에 스케치된 인덱스를 살펴보겠습니다:

* 첫 번째 단계는 엔티티 ID를 저장합니다
* 두 번째 단계는 관련된 속성 이름을 저장합니다
* 세 번째 단계는 관련된 값을 저장합니다

이 인덱스는 최상위 맵이 엔티티 ID를 보유하고, 두 번째 단계가 속성 이름을 보유하며, 리프가 값을 보유하므로 EAVT라고 명명됩니다. "T"는 데이터베이스의 각 층이 자체 인덱스를 가진다는 사실에서 비롯되며, 따라서 인덱스 자체가 특정 시간과 관련이 있습니다. 

\aosafigure[240pt]{functionalDB-images/image_1.png}{EAVT}{500l.functionaldb.eavt}

\aosafigref{500l.functionaldb.avet}는 다음과 같은 이유로 AVET라고 불리는 인덱스를 보여줍니다:

* 첫 번째 단계 맵은 속성 이름을 보유합니다.
* 두 번째 단계 맵은 (속성의) 값을 보유합니다.
* 세 번째 단계 집합은 (첫 번째 단계에 속성이 있는 엔티티의) 엔티티 ID를 보유합니다.

\aosafigure[240pt]{functionalDB-images/image_2.png}{AVET}{500l.functionaldb.avet}

우리의 인덱스는 맵의 맵으로 구현되며, 루트 맵의 키들이 첫 번째 단계 역할을 하고, 각 키는 인덱스의 두 번째 단계 역할을 하는 키들을 가진 맵을 가리키며, 값들은 인덱스의 세 번째 단계입니다. 세 번째 단계의 각 요소는 인덱스의 리프를 보유하는 집합입니다.

각 인덱스는 datom의 구성 요소를 정규 'EAV' 순서(entity_id, attribute-name, attribute-value)의 어떤 순열로 저장합니다. 그러나 인덱스 _외부에서_ datom과 작업할 때는 정규 형식이어야 합니다. 따라서 각 인덱스에 이러한 순서로 변환하고 되돌리는 `from-eav` 및 `to-eav` 함수를 제공합니다.

대부분의 데이터베이스 시스템에서 인덱스는 선택적 구성 요소입니다. 예를 들어, PostgreSQL이나 MySQL과 같은 RDBMS(관계형 데이터베이스 관리 시스템)에서는 테이블의 특정 컬럼에만 인덱스를 추가하도록 선택할 것입니다. 우리는 각 인덱스에 속성이 이 인덱스에 포함되어야 하는지 여부를 결정하는 `usage-pred` 함수를 제공합니다. 

```clojure
(defn make-index [from-eav to-eav usage-pred]
    (with-meta {} {:from-eav from-eav :to-eav to-eav :usage-pred usage-pred}))

 (defn from-eav [index] (:from-eav (meta index)))
 (defn to-eav [index] (:to-eav (meta index)))
 (defn usage-pred [index] (:usage-pred (meta index)))
```

우리 데이터베이스에는 네 개의 인덱스가 있습니다: EAVT(\aosafigref{500l.functionaldb.eavt} 참조), AVET(\aosafigref{500l.functionaldb.avet} 참조), VEAT, VAET. 이들은 `indexes` 함수에서 반환되는 값들의 벡터로 접근할 수 있습니다.

```clojure
(defn indexes[] [:VAET :AVET :VEAT :EAVT])
```

모든 것이 어떻게 함께 작동하는지 보여주기 위해, 다음 다섯 엔티티를 인덱싱한 결과가 \aosatblref{500l.functionaldb.indextable}에서 시각화됩니다.

1. 율리우스 카이사르(줄여서 JC)는 로마에 거주합니다
2. 브루투스(줄여서 B)는 로마에 거주합니다
3. 클레오파트라(줄여서 Cleo)는 이집트에 거주합니다
4. 로마의 강은 티베르강입니다
5. 이집트의 강은 나일강입니다
 
<markdown>
<table>
  <tr>
    <td>EAVT 인덱스</td>
    <td>AVET 인덱스</td>
  </tr>
  <tr>
    <td><ul>
<li>
<span style="background-color:lightblue">JC</span> ⇒ {<span style="background-color:lightgreen">lives-in</span> ⇒ {<span style="background-color:pink">Rome</span>}}
</li>
<li>
<span style="background-color:lightblue">B</span>  ⇒ {<span style="background-color:lightgreen">lives-in</span> ⇒ {<span style="background-color:pink">Rome</span>}}
</li>
<li>
<span style="background-color:lightblue">Cleo</span> ⇒ {<span style="background-color:lightgreen">lives-in</span> ⇒ {<span style="background-color:pink">Egypt</span>}}
</li>
<li>
<span style="background-color:lightblue">Rome</span> ⇒ {<span style="background-color:lightgreen">river</span> ⇒ {<span style="background-color:pink">Tiber</span>}}
</li>
<li>
<span style="background-color:lightblue">Egypt</span> ⇒ {<span style="background-color:lightgreen">river</span> ⇒ {<span style="background-color:pink">Nile</span>}}
</li>
</ul></td>
<td><ul>
<li>
<span style="background-color:lightgreen">lives-in</span> ⇒ {<span style="background-color:pink">Rome</span> ⇒ {<span style="background-color:lightblue">JC, B</span>}}</br>
                         {<span style="background-color:pink">Egypt</span> ⇒ {<span style="background-color:lightblue">Cleo</span>}}
</li>
<li>
<span style="background-color:lightgreen">river</span> ⇒ {<span style="background-color:pink">Rome</span> ⇒ {<span style="background-color:lightblue">Tiber</span>}}</br>
{<span style="background-color:pink">Egypt</span> ⇒ {<span style="background-color:lightblue">Nile</span>}}
</li>
</ul></td>
  </tr>
  <tr>
    <td>VEAT 인덱스</td>
    <td>VAET 인덱스</td>
  </tr>
  <tr>
    <td><ul>
<li>
<span style="background-color:pink">Rome</span> ⇒ {<span style="background-color:lightblue">JC</span> ⇒ {<span style="background-color:lightgreen">lives-in</span>}}<br/>
{<span style="background-color:lightblue">B</span> ⇒ {<span style="background-color:lightgreen">lives-in</span>}}
</li>
<li>
<span style="background-color:pink">Egypt</span> ⇒ {<span style="background-color:lightblue">Cleo</span> ⇒ {<span style="background-color:lightgreen">lives-in</span>}}
</li>
<li>
<span style="background-color:pink">Tiber</span> ⇒ {<span style="background-color:lightblue">Rome</span> ⇒ {<span style="background-color:lightgreen">river</span>}}
</li>
<li>
<span style="background-color:pink">Nile</span> ⇒ {<span style="background-color:lightblue">Egypt</span> ⇒ {<span style="background-color:lightgreen">river</span>}}
</li></ul></td>
<td><ul>
<li>
<span style="background-color:pink">Rome</span> ⇒ {<span style="background-color:lightgreen">lives-in</span> ⇒ {<span style="background-color:lightblue">JC, B</span>}}
</li>
<li>
<span style="background-color:pink">Egypt</span> ⇒ {<span style="background-color:lightgreen">lives-in</span> ⇒ {<span style="background-color:lightblue">Cleo</span>}}</li>
<li>
<span style="background-color:pink">Tiber</span> ⇒ {<span style="background-color:lightgreen">river</span> ⇒ {<span style="background-color:lightblue">Rome</span>}}
</li>
<li>
<span style="background-color:pink">Nile</span> ⇒ {<span style="background-color:lightgreen">river</span> ⇒ {<span style="background-color:lightblue">Egypt</span>}}
</li></ul></td>
  </tr>
</table>
: \label{500l.functionaldb.indextable} Indexes
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
\textbf{EAVT index}
& \textbf{AVET index}
\\
\hline
JC $\Rightarrow$ \{lives-in $\Rightarrow$ \{Rome\}\} & lives-in $\Rightarrow$ \{Rome $\Rightarrow$ \{JC, B\}\}, \{Egypt $\Rightarrow$ \{Cleo\}\} \\
B $\Rightarrow$ \{lives-in $\Rightarrow$ \{Rome\}\}  & river $\Rightarrow$ \{Rome $\Rightarrow$ \{Tiber\}\}, \{Egypt $\Rightarrow$ \{Nile\}\} \\
Cleo $\Rightarrow$ \{lives-in $\Rightarrow$ \{Egypt\}\} & \\ 
Rome $\Rightarrow$ \{river $\Rightarrow$ \{Tiber\}\}  & \\ 
Egypt $\Rightarrow$ \{river $\Rightarrow$ \{Nile\}\}  & \\
\hline
\textbf{VEAT index}
& \textbf{VAET index}
\\
\hline
Rome $\Rightarrow$ \{JC $\Rightarrow$ \{lives-in\}\}, \{B $\Rightarrow$ \{lives-in\}\} & Rome $\Rightarrow$ \{lives-in $\Rightarrow$ \{JC, B\}\} \\
Egypt $\Rightarrow$ \{Cleo $\Rightarrow$ \{lives-in\}\}                                & Egypt $\Rightarrow$ \{lives-in $\Rightarrow$ \{Cleo\}\} \\ 
Tiber $\Rightarrow$ \{Rome $\Rightarrow$ \{river\}\}                                   & Tiber $\Rightarrow$ \{river $\Rightarrow$ \{Rome\}\} \\
Nile $\Rightarrow$ \{Egypt $\Rightarrow$ \{river\}\}                                   & Nile $\Rightarrow$ \{river $\Rightarrow$ \{Egypt\}\} \\
\hline
\end{tabular}
}
\caption{Indexes}
\label{500l.functionaldb.indextable}
\end{table}
</latex>

\newpage

### 데이터베이스

이제 데이터베이스를 구성하는 데 필요한 모든 구성 요소를 갖추었습니다. 데이터베이스 초기화는 다음을 의미합니다:

* 데이터가 없는 초기 빈 층 생성
* 빈 인덱스 집합 생성
* `top-id`와 `curr-time`을 0으로 설정

```clojure
(defn ref? [attr] (= :db/ref (:type (meta attr))))

(defn always[& more] true)

(defn make-db []
   (atom
       (Database. [(Layer.
                   (fdb.storage.InMemory.) ; storage
                   (make-index #(vector %3 %2 %1) #(vector %3 %2 %1) #(ref? %));VAET
                   (make-index #(vector %2 %3 %1) #(vector %3 %1 %2) always);AVET
                   (make-index #(vector %3 %1 %2) #(vector %2 %3 %1) always);VEAT
                   (make-index #(vector %1 %2 %3) #(vector %1 %2 %3) always);EAVT
                  )] 0 0)))
```
하지만 한 가지 문제가 있습니다: Clojure의 모든 컬렉션은 불변입니다. 쓰기 연산은 데이터베이스에서 매우 중요하므로, 우리는 구조를 *Atom*으로 정의합니다. 이는 원자적 쓰기 기능을 제공하는 Clojure 참조 타입입니다.

AVET, VEAT, EAVT 인덱스에는 `always` 함수를 사용하고 VAET 인덱스에는 `ref?` 술어를 사용하는 이유가 궁금할 것입니다. 이는 이러한 인덱스들이 서로 다른 시나리오에서 사용되기 때문이며, 이는 나중에 쿼리를 자세히 살펴볼 때 보게 될 것입니다.

### 기본 접근자

데이터베이스를 위한 복잡한 쿼리 기능을 구축하기 전에, 시스템의 다른 부분들이 어느 시점에서든 연관된 식별자로 우리가 구축한 구성 요소들을 검색할 수 있도록 하는 하위 수준 API를 제공해야 합니다. 데이터베이스의 소비자들도 이 API를 사용할 수 있지만, 그들은 그 위에 구축된 더 완전한 기능을 가진 구성 요소들을 사용할 가능성이 높습니다.

이 하위 수준 API는 다음 네 개의 접근자 함수로 구성됩니다:

```clojure
(defn entity-at
   ([db ent-id] (entity-at db (:curr-time db) ent-id))
   ([db ts ent-id] (get-entity (get-in db [:layers ts :storage]) ent-id)))

(defn attr-at
   ([db ent-id attr-name] (attr-at db ent-id attr-name (:curr-time db)))
   ([db ent-id attr-name ts] (get-in (entity-at db ts ent-id) [:attrs attr-name])))

(defn value-of-at
   ([db ent-id attr-name]  (:value (attr-at db ent-id attr-name)))
   ([db ent-id attr-name ts] (:value (attr-at db ent-id attr-name ts))))

(defn indx-at
   ([db kind] (indx-at db kind (:curr-time db)))
   ([db kind ts] (kind ((:layers db) ts))))
```

우리는 데이터베이스를 다른 값과 마찬가지로 취급하므로, 이러한 함수들 각각은 데이터베이스를 인수로 받습니다. 각 요소는 연관된 식별자로 검색되며, 선택적으로 관심 있는 타임스탬프를 사용합니다. 이 타임스탬프는 조회를 적용할 해당 층을 찾는 데 사용됩니다.

#### 진화

기본 접근자의 첫 번째 사용법은 "과거로의 읽기" API를 제공하는 것입니다. 이는 우리 데이터베이스에서 업데이트 연산이 (덮어쓰기와 반대로) 새 층을 추가하여 수행되기 때문에 가능합니다. 따라서 `prev-ts` 속성을 사용하여 해당 층에서 속성을 살펴보고, 속성의 값이 시간에 따라 어떻게 진화했는지 관찰하기 위해 히스토리 깊숙이 계속 들여다볼 수 있습니다.

`evolution-of` 함수는 정확히 그런 일을 합니다. 이는 각각 속성의 업데이트 타임스탬프와 값으로 구성된 쌍들의 시퀀스를 반환합니다.
```clojure
(defn evolution-of [db ent-id attr-name]
   (loop [res [] ts (:curr-time db)]
     (if (= -1 ts) (reverse res)
         (let [attr (attr-at db ent-id attr-name ts)]
           (recur (conj res {(:ts attr) (:value attr)})  (:prev-ts attr))))))
```
## 데이터 동작 및 생명 주기

지금까지 우리의 논의는 데이터의 구조에 초점을 맞췄습니다: 핵심 구성 요소가 무엇이며 어떻게 함께 집계되는지. 이제 시스템의 역학을 탐구할 때입니다: 추가-업데이트-제거 _데이터 생명주기_를 통해 시간이 지남에 따라 데이터가 어떻게 변경되는지.

이미 논의했듯이, 고고학자의 세계에서 데이터는 실제로 변경되지 않습니다. 한번 생성되면 영원히 존재하며, 새로운 층의 데이터에 의해 세상으로부터 숨겨질 뿐입니다. 여기서 "숨겨진다"는 용어가 중요합니다. 오래된 데이터는 "사라지지" 않습니다&mdash;그것은 묻혀 있으며, 오래된 층을 노출시킴으로써 다시 드러낼 수 있습니다. 반대로, 데이터를 업데이트한다는 것은 그 위에 다른 것을 가진 새 층을 추가하여 기존 것을 가리는 것을 의미합니다. 따라서 그 위에 "아무것도 없는" 층을 추가함으로써 데이터를 "삭제"할 수 있습니다.

이것은 데이터 생명주기에 대해 이야기할 때, 실제로는 시간이 지남에 따라 데이터에 층을 추가하는 것에 대해 이야기하고 있다는 뜻입니다. 

### 기본 필수 요소

데이터 생명주기는 세 가지 기본 연산으로 구성됩니다:

* `add-entity` 함수를 사용한 엔티티 추가
* `remove-entity` 함수를 사용한 엔티티 제거
* `update-entity` 함수를 사용한 엔티티 업데이트

이러한 함수들이 가변성의 착각을 제공하더라도, 각 경우에 실제로 하고 있는 일은 데이터에 다른 층을 추가하는 것뿐이라는 점을 기억하세요. 또한 Clojure의 영속 데이터 구조를 사용하고 있으므로, 호출자의 관점에서는 "제자리" 변경과 동일한 비용을 지불하지만 (즉, 무시할 수 있는 성능 오버헤드), 데이터 구조의 다른 모든 사용자에게는 불변성을 유지합니다.

#### 엔티티 추가

엔티티를 추가하려면 세 가지 작업을 해야 합니다:

* 추가를 위한 엔티티 준비 (ID와 타임스탬프 부여)
* 엔티티를 저장소에 배치
* 필요에 따라 인덱스 업데이트

이러한 단계들은 `add-entity` 함수에서 수행됩니다.

```clojure
(defn add-entity [db ent]
   (let [[fixed-ent next-top-id] (fix-new-entity db ent)
         layer-with-updated-storage (update-in
                            (last (:layers db)) [:storage] write-entity fixed-ent)
         add-fn (partial add-entity-to-index fixed-ent)
         new-layer (reduce add-fn layer-with-updated-storage (indexes))]
    (assoc db :layers (conj (:layers db) new-layer) :top-id next-top-id)))
```
엔티티 준비는 `fix-new-entity` 함수와 그 보조 함수들인 `next-id`, `next-ts`, `update-creation-ts`를 호출하여 수행됩니다.
후자의 두 헬퍼 함수는 데이터베이스의 다음 타임스탬프 찾기(`next-ts`가 수행)와 주어진 엔티티의 생성 타임스탬프 업데이트(`update-creation-ts`가 수행)를 담당합니다. 엔티티의 생성 타임스탬프 업데이트는 엔티티의 속성들을 살펴보고 그들의 `:ts` 필드를 업데이트하는 것을 의미합니다.

```clojure
(defn- next-ts [db] (inc (:curr-time db)))

(defn- update-creation-ts [ent ts-val]
   (reduce #(assoc-in %1 [:attrs %2 :ts ] ts-val) ent (keys (:attrs ent))))

(defn- next-id [db ent]
   (let [top-id (:top-id db)
         ent-id (:id ent)
         increased-id (inc top-id)]
         (if (= ent-id :db/no-id-yet)
             [(keyword (str increased-id)) increased-id]
             [ent-id top-id])))

(defn- fix-new-entity [db ent]
   (let [[ent-id next-top-id] (next-id db ent)
         new-ts               (next-ts db)]
       [(update-creation-ts (assoc ent :id ent-id) new-ts) next-top-id]))
```
엔티티를 저장소에 추가하기 위해, 데이터베이스의 가장 최근 층을 찾아 해당 층의 저장소를 새 층으로 업데이트하며, 그 결과를 `layer-with-updated-storage`에 저장합니다.

마지막으로, 인덱스를 업데이트해야 합니다. 이는 각 인덱스에 대해 (`add-entity` 함수에서 `reduce`와 `partial`된 `add-entity-to-index`의 조합으로 수행됩니다):

* 인덱싱되어야 하는 속성들 찾기 (`add-entity-to-index`에서 속성들에 작동하는 인덱스의 `usage-pred`와 `filter`의 조합 참조)
* 엔티티의 ID로부터 인덱스 경로 구축 (`update-attr-in-index` 함수에서 `from-eav`와 `partial`된 `update-entry-in-index`의 조합 참조)
* 해당 경로를 인덱스에 추가 (`update-entry-in-index` 함수 참조)

```clojure
(defn- add-entity-to-index [ent layer ind-name]
   (let [ent-id (:id ent)
         index (ind-name layer)
         all-attrs  (vals (:attrs ent))
         relevant-attrs (filter #((usage-pred index) %) all-attrs)
         add-in-index-fn (fn [ind attr]
                                 (update-attr-in-index ind ent-id (:name attr)
                                                                  (:value attr)
                                                                  :db/add))]
        (assoc layer ind-name  (reduce add-in-index-fn index relevant-attrs))))

(defn- update-attr-in-index [index ent-id attr-name target-val operation]
   (let [colled-target-val (collify target-val)
         update-entry-fn (fn [ind vl]
                             (update-entry-in-index
                                ind
                                ((from-eav index) ent-id attr-name vl)
                                operation))]
     (reduce update-entry-fn index colled-target-val)))

(defn- update-entry-in-index [index path operation]
   (let [update-path (butlast path)
         update-value (last path)
         to-be-updated-set (get-in index update-path #{})]
     (assoc-in index update-path (conj to-be-updated-set update-value))))
```
이러한 모든 구성 요소들은 주어진 데이터베이스에 새 층으로 추가됩니다. 남은 것은 데이터베이스의 타임스탬프와 `top-id` 필드를 업데이트하는 것뿐입니다. 마지막 단계는 `add-entity`의 마지막 줄에서 발생하며, 이는 업데이트된 데이터베이스도 반환합니다.

또한 `add-entity`를 반복적으로 적용하여 한 번의 호출로 여러 엔티티를 데이터베이스에 추가하는 `add-entities` 편의 함수도 제공합니다.

```clojure
(defn add-entities [db ents-seq] (reduce add-entity db ents-seq))
```
#### 엔티티 제거

데이터베이스에서 엔티티를 제거한다는 것은 그 엔티티가 존재하지 않는 층을 추가하는 것을 의미합니다. 이를 위해서는 다음과 같은 작업이 필요합니다:

* 엔티티 자체를 제거
* 해당 엔티티를 참조하는 다른 엔티티들의 속성 업데이트
* 인덱스에서 해당 엔티티 삭제

이러한 "없이 구성하기" 과정은 `remove-entity` 함수에 의해 실행되며, 이는 `add-entity`와 매우 유사해 보입니다:
```clojure
(defn remove-entity [db ent-id]
   (let [ent (entity-at db ent-id)
         layer (remove-back-refs db ent-id (last (:layers db)))
         no-ref-layer (update-in layer [:VAET] dissoc ent-id)
         no-ent-layer (assoc no-ref-layer :storage 
                                   (drop-entity  
                                          (:storage no-ref-layer) ent))
         new-layer (reduce (partial remove-entity-from-index ent) 
                                 no-ent-layer (indexes))]
     (assoc db :layers (conj  (:layers db) new-layer))))
```
참조 제거는 `remove-back-refs` 함수에 의해 수행됩니다:
```clojure
(defn- remove-back-refs [db e-id layer]
   (let [reffing-datoms (reffing-to e-id layer)
         remove-fn (fn[d [e a]] (update-entity db e a e-id :db/remove))
         clean-db (reduce remove-fn db reffing-datoms)]
     (last (:layers clean-db))))
```
우리는 먼저 `reffing-datoms-to`를 사용하여 주어진 층에서 우리 엔티티를 참조하는 모든 엔티티를 찾습니다. 이는 참조하는 엔티티의 ID, 속성 이름, 그리고 제거되는 엔티티의 ID를 포함하는 삼중항들의 시퀀스를 반환합니다.
```clojure
(defn- reffing-to [e-id layer]
   (let [vaet (:VAET layer)]
         (for [[attr-name reffing-set] (e-id vaet)
               reffing reffing-set]
              [reffing attr-name])))

```
그런 다음 각 삼중항에 `update-entity`를 적용하여 제거된 엔티티를 참조하는 속성들을 업데이트합니다. (`update-entity`가 어떻게 작동하는지는 다음 섹션에서 살펴보겠습니다.)

`remove-back-refs`의 마지막 단계는 인덱스에서, 더 구체적으로는 참조 정보를 저장하는 유일한 인덱스인 VAET 인덱스에서 참조 자체를 삭제하는 것입니다. 

#### 엔티티 업데이트

본질적으로 업데이트는 엔티티 속성의 값을 수정하는 것입니다. 수정 과정 자체는 속성의 카디널리티에 따라 달라집니다: `:db/multiple` 카디널리티를 가진 속성은 값들의 집합을 보유하므로, 이 집합에 항목을 추가하거나 제거하거나 집합 전체를 완전히 교체할 수 있어야 합니다. `:db/single` 카디널리티를 가진 속성은 단일 값을 보유하므로 교체만 허용됩니다.

우리는 또한 속성과 그 값에 대해 직접 조회를 제공하는 인덱스들을 가지고 있으므로, 이들도 업데이트되어야 합니다.

`add-entity` 및 `remove-entity`와 마찬가지로, 실제로는 엔티티를 제자리에서 수정하지 않고 대신 업데이트된 엔티티를 포함하는 새 층을 추가할 것입니다.

```clojure
(defn update-entity
   ([db ent-id attr-name new-val]
    (update-entity db ent-id attr-name new-val :db/reset-to))
   ([db ent-id attr-name new-val operation]
      (let [update-ts (next-ts db)
            layer (last (:layers db))
            attr (attr-at db ent-id attr-name)
            updated-attr (update-attr attr new-val update-ts operation)
            fully-updated-layer (update-layer layer ent-id 
                                              attr updated-attr 
                                              new-val operation)]
        (update-in db [:layers] conj fully-updated-layer))))
```
속성을 업데이트하기 위해, `attr-at`으로 속성을 찾은 다음 `update-attr`을 사용하여 실제 업데이트를 수행합니다. 
```clojure
(defn- update-attr [attr new-val new-ts operation]
    {:pre  [(if (single? attr)
            (contains? #{:db/reset-to :db/remove} operation)
            (contains? #{:db/reset-to :db/add :db/remove} operation))]}
    (-> attr
       (update-attr-modification-time new-ts)
       (update-attr-value new-val operation)))
```
업데이트를 수행하기 위해 두 개의 헬퍼 함수를 사용합니다. `update-attr-modification-time`은 그림 1의 검은 화살표 생성을 반영하도록 타임스탬프를 업데이트합니다:
```clojure
(defn- update-attr-modification-time  
  [attr new-ts]
       (assoc attr :ts new-ts :prev-ts (:ts attr)))
```
`update-attr-value`는 실제로 값을 업데이트합니다:
```clojure
(defn- update-attr-value [attr value operation]
   (cond
      (single? attr)    (assoc attr :value #{value})
      ; now we're talking about an attribute of multiple values
      (= :db/reset-to operation) 
        (assoc attr :value value)
      (= :db/add operation) 
        (assoc attr :value (CS/union (:value attr) value))
      (= :db/remove operation)
        (assoc attr :value (CS/difference (:value attr) value))))
```
남은 것은 인덱스에서 이전 값을 제거하고 새 값을 추가한 다음, 업데이트된 모든 구성 요소로 새 층을 구성하는 것입니다. 다행히도 엔티티 추가 및 제거를 위해 작성한 코드를 활용하여 이를 수행할 수 있습니다.

### 트랜잭션

우리의 하위 수준 API에서 각 연산은 단일 엔티티에 대해 작동합니다. 하지만 거의 모든 데이터베이스는 사용자가 여러 연산을 하나의 _트랜잭션_으로 수행할 수 있는 방법을 제공합니다. 이는 다음을 의미합니다:

* 연산들의 배치는 하나의 원자적 연산으로 간주되므로, 모든 연산이 함께 성공하거나 함께 실패합니다.
* 데이터베이스는 트랜잭션 전후에 유효한 상태를 유지합니다.
* 배치 업데이트는 _격리된_ 것으로 보입니다. 즉, 다른 쿼리는 연산 중 일부만 적용된 데이터베이스 상태를 절대 볼 수 없어야 합니다.

데이터베이스와 수행될 연산 집합을 소비하고 주어진 변경사항이 반영된 상태의 데이터베이스를 생성하는 인터페이스를 통해 이러한 요구사항을 충족할 수 있습니다. 배치에서 제출된 모든 변경사항은 _단일_ 층의 추가를 통해 적용되어야 합니다. 하지만 문제가 있습니다: 하위 수준 API에서 작성한 모든 함수가 데이터베이스에 새 층을 추가합니다. _n_개의 연산으로 배치를 수행한다면 _n_개의 새 층이 추가되는 것을 보게 될 텐데, 우리가 정말 원하는 것은 정확히 하나의 새 층을 갖는 것입니다.

여기서 핵심은 우리가 원하는 층이 해당 업데이트를 순서대로 수행하여 생성될 _최상위_ 층이라는 것입니다. 따라서 해결책은 사용자의 연산을 하나씩 실행하여 각각 새 층을 생성하는 것입니다. 마지막 층이 생성되면, 해당 최상위 층만 취해서 초기 데이터베이스에 배치합니다(모든 중간 층들은 피오르드로 떠나보내며). 이 모든 작업을 완료한 후에만 데이터베이스의 타임스탬프를 업데이트합니다.

이 모든 것은 데이터베이스의 초기 값과 수행할 연산 배치를 받아서 업데이트된 값을 반환하는 `transact-on-db` 함수에서 수행됩니다. 

```clojure
(defn transact-on-db [initial-db ops]
    (loop [[op & rst-ops] ops transacted initial-db]
      (if op
          (recur rst-ops (apply (first op) transacted (rest op)))
          (let [initial-layer  (:layers initial-db)
                new-layer (last (:layers transacted))]
            (assoc initial-db :layers (conj initial-layer new-layer) 
                              :curr-time (next-ts initial-db) 
                              :top-id (:top-id transacted))))))
``` 
여기서 우리가 _값_이라는 용어를 사용했다는 점에 주목하세요. 이는 이 함수의 호출자만이 업데이트된 상태에 노출되며, 데이터베이스의 다른 모든 사용자는 이 변경사항을 알 수 없다는 것을 의미합니다(데이터베이스는 값이고, 따라서 변경될 수 없습니다).
다른 사용자가 수행한 상태 변경에 사용자들이 노출될 수 있는 시스템을 만들기 위해, 사용자들은 데이터베이스와 직접 상호작용하지 않고 대신 또 다른 수준의 간접 참조를 사용하여 참조합니다. 이 추가 수준은 Clojure의 참조 타입인 `Atom`을 사용하여 구현됩니다. 여기서 우리는 `Atom`의 주요한 세 가지 핵심 기능을 활용합니다:

1. 값을 참조합니다.
2. 트랜잭션을 실행하여 `Atom`의 참조를 다른 값으로 업데이트할 수 있습니다(Clojure의 소프트웨어 트랜잭셔널 메모리 기능 사용). 트랜잭션은 `Atom`과 함수를 받습니다. 해당 함수는 `Atom`의 값에 대해 작동하고 새 값을 반환합니다. 트랜잭션 실행 후, `Atom`은 함수에서 반환된 값을 참조합니다.
3. `Atom`이 참조하는 값에 접근하는 것은 역참조를 통해 수행되며, 이는 해당 시점에서 그 `Atom`의 상태를 반환합니다.

Clojure의 `Atom`과 `transact-on-db`에서 수행되는 작업 사이에는 여전히 연결해야 할 간격이 있습니다. 즉, 올바른 입력으로 트랜잭션을 호출하는 것입니다.

가장 간단하고 명확한 API를 제공하기 위해, 사용자가 단지 `Atom`과 연산 목록만 제공하고, 데이터베이스가 사용자 입력을 적절한 트랜잭션으로 변환하도록 하고 싶습니다.

그 변환은 다음 트랜잭션 호출 체인에서 발생합니다:

```
transact →  _transact → swap! → transact-on-db
```

사용자는 `Atom`(즉, 연결)과 수행할 연산들로 `transact`를 호출하며, 이는 입력을 `_transact`에 전달하면서 `Atom`을 업데이트하는 함수의 이름(`swap!`)을 추가합니다.

```clojure
(defmacro transact [db-conn & txs]  `(_transact ~db-conn swap! ~@txs))
```

`_transact`는 `swap!` 호출을 준비합니다. 이는 `swap!`로 시작하고, 그 다음에 `Atom`, 그리고 `transact-on-db` 심볼과 연산 배치가 이어지는 목록을 생성함으로써 수행됩니다.

```clojure
(defmacro  _transact [db op & txs]
   (when txs
     (loop [[frst-tx# & rst-tx#] txs  res#  [op db `transact-on-db]  accum-txs# []]
       (if frst-tx#
           (recur rst-tx# res#  (conj  accum-txs#  (vec frst-tx#)))
           (list* (conj res#  accum-txs#))))))
```

`swap!`는 트랜잭션 내에서 (이전에 준비된 인수들로) `transact-on-db`를 호출하고, `transact-on-db`는 데이터베이스의 새 상태를 생성하여 반환합니다.

이 시점에서 약간의 미세한 수정으로 "만약에"라는 질문을 할 수 있는 방법도 제공할 수 있음을 알 수 있습니다. 이는 `swap!`를 시스템에 어떤 변경도 가하지 않는 함수로 교체함으로써 수행할 수 있습니다. 이 시나리오는 `what-if` 호출 체인으로 구현됩니다:

`what-if` $\to$ `_transact` $\to$ `_what-if` $\to$ `transact-on-db`

사용자는 데이터베이스 값과 수행할 연산들로 `what-if`를 호출합니다. 그러면 이러한 입력을 `_transact`에 전달하면서, `swap!`의 API를 흉내내지만 그 효과는 없는 함수(`_what-if`라고 함)를 추가합니다.

```clojure
(defmacro what-if [db & ops]  `(_transact ~db _what-if  ~@ops))
```

`_transact`는 `_what-if` 호출을 준비합니다. 이는 `_what-if`로 시작하고, 그 다음에 데이터베이스, 그리고 `transact-on-db` 심볼과 연산 배치가 이어지는 목록을 생성함으로써 수행됩니다. `_what-if`는 트랜잭션 시나리오에서 `swap!`가 하는 것처럼 `transact-on-db`를 호출하지만, 시스템에 어떤 변경도 가하지 않습니다.

```clojure
(defn- _what-if [db f txs]  (f db txs))
```
 
여기서 함수가 아닌 매크로를 사용하고 있다는 점에 주목하세요. 여기서 매크로를 사용하는 이유는 매크로의 인수들이 호출이 일어날 때 평가되지 않기 때문입니다. 이를 통해 사용자가 Clojure에서 모든 함수 호출이 구조화되는 것과 같은 방식으로 연산을 구조화하여 제공하는 더 깨끗한 API 설계를 제공할 수 있습니다.

위의 과정은 다음 예시들에서 볼 수 있습니다. 트랜잭션의 경우, 사용자 호출: 
```clojure
(transact db-conn  (add-entity e1) (update-entity e2 atr2 val2 :db/add))  
```
다음으로 변환됩니다:
```clojure
(_transact db-conn swap! (add-entity e1) (update-entity e2 atr2 val2 :db/add))
```
그리고 다음이 됩니다:
```clojure
(swap! db-conn transact-on-db [[add-entity e1][update-entity e2 atr2 val2 :db/add]])
```

what-if의 경우, 사용자 호출:

```clojure
(what-if my-db (add-entity e3) (remove-entity e4))
```
다음으로 변환됩니다:
```clojure
(_transact my-db _what-if (add-entity e3) (remove-entity e4))
```
그 다음:
```clojure
(_what-if my-db transact-on-db [[add-entity e3] [remove-entity e4]])
```
그리고 최종적으로:
```clojure
(transact-on-db my-db  [[add-entity e3] [remove-entity e4]])
```

## 라이브러리로서의 인사이트 추출

이 시점에서 데이터베이스의 핵심 기능이 구현되었으며, 이제 그 *존재 이유*인 인사이트 추출을 추가할 때입니다. 여기서 사용한 아키텍처 접근법은 이러한 기능을 라이브러리로 추가할 수 있도록 하는 것입니다. 데이터베이스의 다양한 용도에서 서로 다른 메커니즘이 필요하기 때문입니다. 

### 그래프 탐색

엔티티 간의 참조 연결은 엔티티 속성의 타입이 `:db/ref`일 때 생성되며, 이는 해당 속성의 값이 다른 엔티티의 ID라는 것을 의미합니다. 참조하는 엔티티가 데이터베이스에 추가되면, 참조는 VAET 인덱스에서 인덱싱됩니다.
VAET 인덱스에서 발견되는 정보는 엔티티에 대한 모든 들어오는 링크를 추출하는 데 활용될 수 있습니다. 이는 해당 인덱스에서 엔티티로부터 도달 가능한 모든 리프를 수집하는 `incoming-refs` 함수에서 수행됩니다:

```clojure
(defn incoming-refs [db ts ent-id & ref-names]
   (let [vaet (indx-at db :VAET ts)
         all-attr-map (vaet ent-id)
         filtered-map (if ref-names 
                          (select-keys ref-names all-attr-map) 
                          all-attr-map)]
      (reduce into #{} (vals filtered-map))))
```
주어진 엔티티의 모든 속성을 살펴보고 `:db/ref` 타입인 속성의 모든 값을 수집함으로써 해당 엔티티에서 나가는 모든 참조를 추출할 수도 있습니다. 이는 `outgoing-refs` 함수에 의해 수행됩니다.

```clojure
(defn outgoing-refs [db ts ent-id & ref-names]
   (let [val-filter-fn (if ref-names #(vals (select-keys ref-names %)) vals)]
   (if-not ent-id []
     (->> (entity-at db ts ent-id)
          (:attrs) (val-filter-fn) (filter ref?) (mapcat :value)))))
```
이 두 함수는 엔티티와 속성에서 그래프의 노드와 링크로 추상화 수준을 높이는 역할을 하므로, 모든 그래프 탐색 연산의 기본 구성 요소 역할을 합니다. 데이터베이스를 그래프로 바라볼 수 있는 능력을 갖추게 되면, 다양한 그래프 탐색 및 쿼리 API를 제공할 수 있습니다. 이는 독자에게 해결된 연습 문제로 남겨둡니다. 한 가지 해결책은 이 장의 소스 코드에서 찾을 수 있습니다(`graph.clj` 참조).   


## 데이터베이스 쿼리

우리가 제시하는 두 번째 라이브러리는 쿼리 기능을 제공하며, 이것이 이 섹션의 주요 관심사입니다.
데이터베이스는 강력한 쿼리 메커니즘 없이는 사용자에게 그다지 유용하지 않습니다. 이 기능은 일반적으로 관심 있는 데이터 집합을 선언적으로 지정하는 데 사용되는 _쿼리 언어_를 통해 사용자에게 노출됩니다. 

우리의 데이터 모델은 시간이 지남에 따라 사실(즉, datom)을 축적하는 것을 기반으로 합니다. 이 모델에서 올바른 쿼리 언어를 찾기 위한 자연스러운 곳은 _논리 프로그래밍_입니다. 논리 프로그래밍의 영향을 받은 일반적으로 사용되는 쿼리 언어는 _Datalog_인데, 이는 우리의 데이터 모델에 매우 적합할 뿐만 아니라 Clojure의 구문에 매우 우아하게 적응됩니다. 우리의 쿼리 엔진은 [Datomic 데이터베이스](http://docs.datomic.com/query.html)의 Datalog 언어 부분집합을 구현할 것입니다.

### 쿼리 언어

제안된 언어의 예시 쿼리를 살펴보겠습니다. 이 쿼리는 "피자를 좋아하고, 영어를 말하며, 이번 달에 생일이 있는 엔티티의 이름과 생일은 무엇인가?"라고 묻습니다:
```clojure
{  :find [?nm ?bd ]
   :where [
      [?e  :likes "pizza"]
      [?e  :name  ?nm]
      [?e  :speak "English"]
      [?e  :bday (bday-mo? ?bd)]]}
```
#### 구문

우리는 쿼리의 기본 구문을 제공하기 위해 Clojure 데이터 리터럴의 구문을 직접 사용합니다. 이를 통해 특수한 파서를 작성할 필요를 피하면서도 Clojure에 익숙한 프로그래머에게 친숙하고 쉽게 읽을 수 있는 형태를 제공할 수 있습니다.

쿼리는 두 개의 항목을 가진 맵입니다:

* `:where`를 키로 하고 _규칙_을 값으로 하는 항목. 규칙은 _절_들의 벡터이고, 절은 세 개의 _술어_로 구성된 벡터이며, 각 술어는 datom의 서로 다른 구성 요소에 대해 작동합니다. 위의 예시에서 `[?e  :likes "pizza"]`는 하나의 절입니다. 이 `:where` 항목은 우리 데이터베이스의 datom에 대한 필터 역할을 하는 규칙을 정의합니다(SQL `WHERE` 절과 같이).
* `:find`를 키로 하고 벡터를 값으로 하는 항목. 벡터는 선택된 datom의 어떤 구성 요소가 결과에 투영되어야 하는지를 정의합니다(SQL `SELECT` 절과 같이).

위의 설명은 중요한 요구사항을 생략합니다: 서로 다른 절들이 값에 대해 어떻게 동기화되는지(즉, 그들 사이에 조인 연산을 만드는 방법), 그리고 발견된 값들이 출력에서 어떻게 구조화되는지(`:find` 부분에서 지정됨). 

우리는 앞에 `?`가 붙어 표시되는 _변수_를 사용하여 이 두 요구사항을 모두 충족합니다. 이 정의의 유일한 예외는 "신경 쓰지 않는" 변수인 `_`(밑줄)입니다.

쿼리의 절은 세 개의 술어로 구성됩니다. \aosatblref{500l.functionaldb.predicates}는 우리 쿼리 언어에서 술어 역할을 할 수 있는 것을 정의합니다.

<markdown>
<table>
  <tr>
    <td>이름</td>
    <td>의미</td>
    <td>예시</td>
  </tr>
  <tr>
    <td>상수</td>
    <td>datom 항목의 값이 상수와 같은가?</td>
    <td>:likes</td>
  </tr>
  <tr>
    <td>변수</td>
    <td>datom 항목의 값을 변수에 바인드하고 참을 반환한다.</td>
    <td>?e</td>
  </tr>
  <tr>
    <td>무관심</td>
    <td>항상 참을 반환한다.</td>
    <td>_</td>
  </tr>
  <tr>
    <td>단항 연산자</td>
    <td>변수를 피연산자로 받는 단항 연산.<br/>
        datom 항목의 값을 변수에 바인드한다('_'가 아닌 경우).<br/>
        변수를 datom 항목의 값으로 치환한다.<br/>
        연산의 적용 결과를 반환한다.</td>
    <td>(bday-mo? _)</td>
  </tr>
  <tr>
    <td>이항 연산자</td>
    <td>피연산자 중 하나가 변수여야 하는 이항 연산.<br/>
        datom 항목의 값을 변수에 바인드한다('_'가 아닌 경우).<br/>
        변수를 datom 항목의 값으로 치환한다.<br/>
        연산의 결과를 반환한다.</td>
    <td>(&gt; ?age 20)</td>
  </tr>
</table>
: \label{500l.functionaldb.predicates} Predicates
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{lll}
\hline
\textbf{이름} & \textbf{의미} & \textbf{예시} \\
\hline
상수 & datom 항목의 값이 상수와 같은가? & \verb|:likes| \\
변수 & datom 항목의 값을 변수에 바인드하고 참을 반환한다. & \verb|?e| \\
무관심 & 항상 참을 반환한다. & \verb|_| \\
단항 연산자 & \begin{tabular}{@{}l@{}} 변수를 피연산자로 받는 단항 연산. \\ datom 항목의 값을 변수에 바인드한다(\verb|_|가 아닌 경우). \\ 변수를 datom 항목의 값으로 치환한다. \\ 연산의 적용 결과를 반환한다. \end{tabular} & \verb|(bday-mo? _)| \\
이항 연산자 & \begin{tabular}{@{}l@{}} 피연산자 중 하나가 변수여야 하는 이항 연산. \\ datom 항목의 값을 변수에 바인드한다(\verb|_|가 아닌 경우). \\ 변수를 datom 항목의 값으로 치환한다. \\ 연산의 결과를 반환한다. \end{tabular} & \verb|(&gt; ?age 20)| \\
\hline
\end{tabular}
}
\caption{Predicates}
\label{500l.functionaldb.predicates}
\end{table}
</latex>

#### 쿼리 언어의 제한사항

엔지니어링은 모두 트레이드오프를 관리하는 것이고, 쿼리 엔진을 설계하는 것도 다르지 않습니다. 우리의 경우, 해결해야 하는 주요 트레이드오프는 기능 풍부함 대 복잡성입니다. 이 트레이드오프를 해결하려면 시스템의 일반적인 사용 사례를 살펴보고, 거기서 어떤 제한사항이 받아들일 만한지 결정해야 합니다.

우리 데이터베이스에서는 다음과 같은 제한사항을 가진 쿼리 엔진을 구축하기로 결정했습니다:

* 사용자는 절 사이에 논리 연산을 정의할 수 없습니다. 항상 'AND'로 함께 연결됩니다. (이는 단항 또는 이항 술어를 사용하여 해결할 수 있습니다.)
* 쿼리에 둘 이상의 절이 있는 경우, 해당 쿼리의 모든 절에서 발견되는 변수가 하나 있어야 합니다. 이 변수는 조인 변수 역할을 합니다. 이 제한사항은 쿼리 최적화기를 단순화합니다.
* 쿼리는 단일 데이터베이스에서만 실행됩니다.

이러한 설계 결정은 Datalog보다 덜 풍부한 쿼리 언어를 만들지만, 여전히 많은 유형의 간단하면서도 유용한 쿼리를 지원할 수 있습니다.

### 쿼리 엔진 설계

우리의 쿼리 언어는 사용자가 _무엇을_ 액세스하고 싶은지 지정할 수 있게 해주지만, _어떻게_ 이것이 달성될지의 세부사항은 숨깁니다. 쿼리 엔진은 주어진 쿼리에 대한 데이터를 산출하는 책임을 지는 데이터베이스 구성 요소입니다.

이는 네 단계를 포함합니다:

1. 내부 표현으로의 변환: 쿼리를 텍스트 형태에서 쿼리 플래너가 소비하는 데이터 구조로 변환합니다.
2. 쿼리 계획 구축: 주어진 쿼리의 결과를 산출하는 효율적인 _계획_을 결정합니다. 우리의 경우, 쿼리 계획은 호출될 함수입니다.
3. 계획 실행: 계획을 실행하고 그 결과를 다음 단계로 보냅니다.
4. 통합 및 보고: 보고되어야 하는 결과만 추출하고 지정된 대로 형식을 맞춥니다.

#### 1단계: 변환

이 단계에서는 주어진 쿼리를 사용자가 이해하기 쉬운 표현에서 쿼리 플래너가 효율적으로 소비할 수 있는 표현으로 변환합니다. 

The `:find` part of the query is transformed into a set of the given variable names:

```clojure
(defmacro symbol-col-to-set [coll] (set (map str coll)))
```

쿼리의 `:where` 부분은 중첩된 벡터 구조를 유지합니다. 하지만 각 절의 각 항들은 \aosatblref{500l.functionaldb.predicates}에 따라 술어로 치환됩니다. 

```clojure
(defmacro clause-term-expr [clause-term]
   (cond
    (variable? (str clause-term)) ;variable
      #(= % %) 
    (not (coll? clause-term)) ;constant 
      `#(= % ~clause-term) 
    (= 2 (count clause-term)) ;unary operator
      `#(~(first clause-term) %) 
    (variable? (str (second clause-term)));binary operator, 1st operand is variable
      `#(~(first clause-term) % ~(last clause-term))
    (variable? (str (last clause-term)));binary operator, 2nd operand is variable
      `#(~(first clause-term) ~(second clause-term) %)))
```

각 절에 대해, 해당 절에서 사용된 변수 이름들을 포함하는 벡터가 메타데이터로 설정됩니다. 

```clojure
(defmacro clause-term-meta [clause-term]
   (cond
   (coll? clause-term)  (first (filter #(variable? % false) (map str clause-term))) 
   (variable? (str clause-term) false) (str clause-term) 
   :no-variable-in-clause nil))
```

We use `pred-clause` to iterate over the terms in each clause: 

```clojure
(defmacro pred-clause [clause]
   (loop [[trm# & rst-trm#] clause exprs# [] metas# []]
     (if  trm#
          (recur rst-trm# (conj exprs# `(clause-term-expr ~ trm#)) 
                       (conj metas#`(clause-term-meta ~ trm#)))
          (with-meta exprs# {:db/variable metas#}))))
```

절 자체에 대한 반복은 `q-clauses-to-pred-clauses`에서 일어납니다:
          
```clojure
(defmacro  q-clauses-to-pred-clauses [clauses]
     (loop [[frst# & rst#] clauses preds-vecs# []]
       (if-not frst#  preds-vecs#
         (recur rst# `(conj ~preds-vecs# (pred-clause ~frst#))))))
```
매크로가 인수를 즉시 평가하지 않는다는 사실을 다시 한번 활용합니다. 이를 통해 사용자가 변수 이름을 문자열(예: `"?name"`)로 제공하거나 더 나쁘게는 변수 이름을 인용(예: `'?name`)하여 엔진의 내부를 이해하도록 요구하는 대신, 변수 이름을 심볼(예: `?name`)로 제공하는 더 간단한 API를 정의할 수 있습니다.

이 단계가 끝날 때, 우리 예시는 `:find` 부분에 대해 다음 집합을 산출합니다: 

```clojure 
#{"?nm" "?bd"} 
``` 

그리고 `:where` 부분에 대해서는 \aosatblref{500l.functionaldb.clauses}에서의 다음 구조를 얻습니다. (_Predicate Clause_ 열의 각 셀은 _Meta Clause_ 열의 이웃에서 발견된 메타데이터를 보유합니다.)

<markdown>
<table>
<tr>
	<td>Query Clause</td>
	<td>Predicate Clause</td>
	<td>Meta Clause</td>
</tr>
<tr>
	<td>[?e  :likes "pizza"]</td>
	<td>[#(= % %)  #(= % :likes)  #(= % "pizza")]</td>
	<td>["?e" nil nil]</td>
</tr>
<tr>
	<td>[?e  :name  ?nm]</td>
	<td>[#(= % %)  #(= % :name) #(= % %)]</td>
	<td>["?e" nil "?nm"]</td>
</tr>
<tr>
	<td>[?e  :speak "English"]</td>
	<td>[#(= % %) #(= % :speak) #(= % "English")]</td>
	<td>["?e" nil nil]</td>
</tr>
<tr>
	<td>[?e  :bday (bday-mo? ?bd)]</td>
	<td>[#(= % %) #(= % :bday) #(bday-mo? %)]</td>
	<td>["?e" nil "?bd"]
</td>
</tr>
</table>
: \label{500l.functionaldb.clauses} Clauses
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{lll}
\hline
\textbf{Query Clause} & \textbf{Predicate Clause} & \textbf{Meta Clause} \\
\hline
\verb|[?e  :likes "pizza"]| & \verb|[#(= % %)  #(= % :likes)  #(= % "pizza")]| & \verb|["?e" nil nil]| \\
\verb|[?e  :name  ?nm]| & \verb|[#(= % %)  #(= % :name) #(= % %)]| & \verb|["?e" nil "?nm"]| \\
\verb|[?e  :speak "English"]| & \verb|[#(= % %) #(= % :speak) #(= % "English")]| & \verb|["?e" nil nil]| \\
\verb|[?e  :bday (bday-mo? ?bd)]| & \verb|[#(= % %) #(= % :bday) #(bday-mo? %)]| & \verb|["?e" nil "?bd"]| \\
\hline
\end{tabular}
}
\caption{Clauses}
\label{500l.functionaldb.clauses}
\end{table}
</latex>

이 구조는 엔진이 올바른 실행 계획을 결정한 후 나중 단계에서 실행되는 쿼리 역할을 합니다.

#### 2단계: 계획 수립

이 단계에서는 쿼리가 설명하는 결과를 생성하기 위한 좋은 계획을 구성하기 위해 쿼리를 검사합니다.

일반적으로 이는 적절한 인덱스(\aosatblref{500l.functionaldb.indexselection})를 선택하고 함수 형태의 계획을 구성하는 것을 포함합니다. 우리는 _단일_ 조인 변수(하나의 종류의 요소에만 작동할 수 있는)를 기반으로 인덱스를 선택합니다.

<markdown>
<table>
	<tr>
		<td>조인 변수가 작동하는 대상</td><td>사용할 인덱스</td>
	</tr>
	<tr>
		<td>엔티티 ID</td><td>AVET</td>
	</tr>
	<tr>
		<td>속성 이름</td><td>VEAT</td>
	</tr>
	<tr>
		<td>속성 값</td><td>EAVT</td>
	</tr>
</table>
: \label{500l.functionaldb.indexselection} Index Selection
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
\textbf{조인 변수가 작동하는 대상} & \textbf{사용할 인덱스} \\
\hline
엔티티 ID & AVET \\
속성 이름 & VEAT \\
속성 값 & EAVT \\
\hline
\end{tabular}
}
\caption{Index Selection}
\label{500l.functionaldb.indexselection}
\end{table}
</latex>

이 매핑 뒤의 논리는 실제로 생성된 계획을 실행할 때인 다음 섹션에서 더 명확해질 것입니다. 지금은 여기서 핵심이 조인 변수가 작동하는 요소들을 리프에 보유하는 인덱스를 선택하는 것이라는 점만 주목하세요.

조인 변수의 인덱스 찾기는 `index-of-joining-variable`에 의해 수행됩니다:

```clojure
(defn index-of-joining-variable [query-clauses]
   (let [metas-seq  (map #(:db/variable (meta %)) query-clauses) 
         collapsing-fn (fn [accV v] (map #(when (= %1 %2) %1)  accV v))
         collapsed (reduce collapsing-fn metas-seq)] 
     (first (keep-indexed #(when (variable? %2 false) %1)  collapsed)))) 
```
쿼리에서 각 절의 메타데이터를 추출하는 것부터 시작합니다. 이 추출된 메타데이터는 3개 요소 벡터이며, 각 요소는 변수 이름이거나 nil입니다. (해당 벡터에는 하나를 넘지 않는 변수 이름이 있다는 점에 주목하세요.) 벡터가 추출되면, 그것으로부터 (리듀싱을 통해) 변수 이름이거나 nil인 단일 값을 생성합니다. 변수 이름이 생성되면, 그것은 모든 메타데이터 벡터에서 같은 인덱스에 나타났다는 의미입니다. 즉, 이것이 조인 변수입니다. 따라서 위에서 설명한 매핑을 기반으로 이 조인 변수에 관련된 인덱스를 사용하도록 선택할 수 있습니다.

인덱스가 선택되면, 쿼리와 인덱스 이름을 닫고 쿼리 결과를 반환하는 데 필요한 연산을 실행하는 함수인 우리의 계획을 구성합니다.
 

```clojure
(defn build-query-plan [query]
   (let [term-ind (index-of-joining-variable query)
         ind-to-use (case term-ind 0 :AVET 1 :VEAT 2 :EAVT)]
      (partial single-index-query-plan query ind-to-use)))
```

우리 예시에서 선택된 인덱스는 조인 변수가 엔티티 ID에서 작동하므로 `AVET` 인덱스입니다.

#### 3단계: 계획 실행

이전 단계에서 우리의 쿼리 계획이 `single-index-query-plan`을 호출하는 것으로 끝난다는 것을 보았습니다. 이 함수는 다음을 수행합니다:

1. 인덱스에서 각 술어 절을 적용합니다(각 술어는 적절한 인덱스 레벨에서).
2. 결과들 간에 AND 연산을 수행합니다.
3. 결과들을 더 간단한 데이터 구조로 병합합니다.

```clojure
(defn single-index-query-plan [query indx db]
   (let [q-res (query-index (indx-at db indx) query)]
     (bind-variables-to-query q-res (indx-at db indx))))
```
이 프로세스를 더 잘 설명하기 위해 우리 데이터베이스가 \aosatblref{500l.functionaldb.exampleentities}의 엔티티들을 보유하고 있다고 가정하고 우리의 예시 쿼리를 사용하여 시연하겠습니다.

<markdown>
<table>
<tr>
	<td>Entity ID</td>
	<td>Attribute Name</td>
	<td>Attribute Value</td>
</tr>
<tr>
	<td>1</td>
	<td>:name </br>
		:likes</br>
		:speak</br>
		:bday 
	</td>
	<td>USA</br>
		Pizza</br>
		English</br>
		July 4, 1776
	</td>
</tr>
<tr>
	<td>2</td>
	<td>:name </br>
		:likes</br>
		:speak</br>
		:bday 
	</td>
	<td>France</br>
		Red wine</br>
		French</br>
		July 14, 1789
	</td>
</tr>
<tr>
	<td>3</td>
	<td>:name </br>
		:likes</br>
		:speak</br>
		:bday 
	</td>
	<td>Canada</br>
		Snow</br>
		English</br>
		July 1, 1867
	</td>
</tr>
</table> 
: \label{500l.functionaldb.exampleentities} Example entities
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{lll}
\hline
\textbf{Entity ID} & \textbf{Attribute Name} & \textbf{Attribute Value} \\
\hline
1 & \begin{tabular}{@{}l@{}} \verb|:name| \\ \verb|:likes| \\ \verb|:speak| \\ \verb|:bday| \end{tabular} & \begin{tabular}{@{}l@{}} USA \\ Pizza \\ English \\ July 4, 1776 \end{tabular} \\
2 & \begin{tabular}{@{}l@{}} \verb|:name| \\ \verb|:likes| \\ \verb|:speak| \\ \verb|:bday| \end{tabular} & \begin{tabular}{@{}l@{}} France \\ Red wine \\ French \\ July 14, 1789 \end{tabular} \\
3 & \begin{tabular}{@{}l@{}} \verb|:name| \\ \verb|:likes| \\ \verb|:speak| \\ \verb|:bday| \end{tabular} & \begin{tabular}{@{}l@{}} Canada \\ Snow \\ English \\ July 1, 1867 \end{tabular} \\
\hline
\end{tabular}
}
\caption{Example entities}
\label{500l.functionaldb.exampleentities}
\end{table}
</latex>

이제 더 깊이 들어가서 우리의 쿼리가 드디어 결과를 산출하기 시작하는 `query-index` 함수를 살펴볼 때입니다:

```clojure
(defn query-index [index pred-clauses]
   (let [result-clauses (filter-index index pred-clauses)
         relevant-items (items-that-answer-all-conditions (map last result-clauses) 
                                                          (count pred-clauses))
         cleaned-result-clauses (map (partial mask-path-leaf-with-items 
                                              relevant-items)
                                     result-clauses)] 
     (filter #(not-empty (last %)) cleaned-result-clauses)))
```
이 함수는 이전에 선택된 인덱스에 술어 절들을 적용하는 것부터 시작합니다. 인덱스에서 술어 절의 각 적용은 _결과 절_을 반환합니다. 

결과의 주요 특징은 다음과 같습니다:

1. 인덱스의 서로 다른 레벨에서 각각 나오고 각각이 해당 술어를 통과한 세 개의 항목으로 구성됩니다.
2. 항목들의 순서는 인덱스의 레벨 구조와 일치합니다. (술어 절들은 항상 EAV 순서입니다.) 재정렬은 술어 절에 인덱스의 `from-eav`를 적용할 때 수행됩니다.
3. 술어 절의 메타데이터가 첨부됩니다.

이 모든 것은 `filter-index` 함수에서 수행됩니다.

```clojure
(defn filter-index [index predicate-clauses]
   (for [pred-clause predicate-clauses
         :let [[lvl1-prd lvl2-prd lvl3-prd] (apply (from-eav index) pred-clause)] 
         [k1 l2map] index  ; keys and values of the first level
         :when (try (lvl1-prd k1) (catch Exception e false))  
         [k2  l3-set] l2map  ; keys and values of the second level
         :when (try (lvl2-prd k2) (catch Exception e false))
         :let [res (set (filter lvl3-prd l3-set))] ]
     (with-meta [k1 k2 res] (meta pred-clause))))
```
쿼리가 7월 4일에 실행되었다고 가정하면, 위 데이터에서 실행한 결과는 \aosatblref{500l.functionaldb.queryresults}에서 볼 수 있습니다.
<markdown>
<table>
<tr>
<td>Result Clause</td><td>Result Meta</td>
</tr>
<tr>
<td>[:likes Pizza #{1}]</td><td>["?e" nil nil]</td>
</tr>
<tr>
<td>[:name USA #{1}]</td><td>["?e" nil "?nm"]</td>
</tr>
<tr>
<td>[:speak "English" #{1, 3}]</td><td>["?e" nil nil]</td>
</tr>
<tr>
<td>[:bday "July 4, 1776" #{1}]</td><td>["?e" nil "?bd"]</td>
</tr>
<tr>
<td>[:name France #{2}]</td><td>["?e" nil "?nm"]</td>
</tr>
<tr>
<td>[:bday "July 14, 1789" #{2}]</td><td>["?e" nil "?bd"]</td>
</tr>
<tr>
<td>[:name Canada #{3}]</td><td>["?e" nil "?nm"]</td>
</tr>
<tr>
<td>[:bday "July 1, 1867" {3}]</td><td>["?e" nil "?bd"]</td>
</tr>
</table>
: \label{500l.functionaldb.queryresults} Query results
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
\textbf{Result Clause} & \textbf{Result Meta} \\
\hline
\verb|[:likes Pizza #{1}]| & \verb|["?e" nil nil]| \\
\verb|[:name USA #{1}]| & \verb|["?e" nil "?nm"]| \\
\verb|[:speak "English" #{1, 3}]| & \verb|["?e" nil nil]| \\
\verb|[:bday "July 4, 1776" #{1}]| & \verb|["?e" nil "?bd"]| \\
\verb|[:name France #{2}]| & \verb|["?e" nil "?nm"]| \\
\verb|[:bday "July 14, 1789" #{2}]| & \verb|["?e" nil "?bd"]| \\
\verb|[:name Canada #{3}]| & \verb|["?e" nil "?nm"]| \\
\verb|[:bday "July 1, 1867" {3}]| & \verb|["?e" nil "?bd"]| \\
\hline
\end{tabular}
}
\caption{Query results}
\label{500l.functionaldb.queryresults}
\end{table}
</latex>

모든 결과 절들을 생성했으면, 그들 사이에 `AND` 연산을 수행해야 합니다. 이는 모든 술어 절들을 통과한 모든 요소들을 찾음으로써 수행됩니다:

```clojure
(defn items-that-answer-all-conditions [items-seq num-of-conditions]
   (->> items-seq ; take the items-seq
         (map vec) ; make each collection (actually a set) into a vector
         (reduce into []) ;reduce all the vectors into one vector
         (frequencies) ;count for each item in how many collections (sets) it was in
         (filter #(<= num-of-conditions (last %))) ;items that answered all conditions
         (map first) ; take from the duos the items themselves
         (set))) ; return it as set
```

우리 예시에서 이 단계의 결과는 값 *1*(USA의 엔티티 ID)을 보유하는 집합입니다. 

이제 모든 조건을 통과하지 못한 항목들을 제거해야 합니다:

```clojure
(defn mask-path-leaf-with-items [relevant-items path]
     (update-in path [2] CS/intersection relevant-items))
```

마지막으로, "비어 있는" 모든 결과 절들(즉, 마지막 항목이 비어 있는)을 제거합니다. 우리는 이를 `query-index` 함수의 마지막 줄에서 수행합니다. 우리 예시는 \aosatblref{500l.functionaldb.filteredqueryresults}의 항목들을 남깁니다.

<markdown>
<table>
<tr>
<td>Result Clause</td><td>Result Meta</td>
</tr>
<tr>
<td>[:likes Pizza #{1}]</td><td>["?e" nil nil]</td>
</tr>
<tr>
<td>[:name USA #{1}]</td><td>["?e" nil "?nm"]</td>
</tr>
<tr>
<td>[:bday "July 4, 1776" #{1}]</td><td>["?e" nil "?bd"]</td>
</tr>
<tr>
<td>[:speak "English" #{1}]</td><td>["?e" nil nil]</td>
</tr>
</table>
: \label{500l.functionaldb.filteredqueryresults} Filtered query results
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{ll}
\hline
\textbf{Result Clause} & \textbf{Result Meta} \\
\hline
\verb|[:likes Pizza #{1}]| & \verb|["?e" nil nil]| \\
\verb|[:name USA #{1}]| & \verb|["?e" nil "?nm"]| \\ 
\verb|[:bday "July 4, 1776" #{1}]| & \verb|["?e" nil "?bd"]| \\
\verb|[:speak "English" #{1}]| & \verb|["?e" nil nil]| \\
\hline
\end{tabular}
}
\caption{Filtered query results}
\label{500l.functionaldb.filteredqueryresults}
\end{table}
</latex>

이제 결과를 보고할 준비가 되었습니다. 결과 절 구조는 이 목적에 다루기 어려우므로, 인덱스와 같은 구조(맵의 맵)로 변환할 것입니다&mdash;중요한 변형과 함께. 

이 변형을 이해하려면, 먼저 _바인딩 쌍_의 개념을 도입해야 합니다. 이는 변수 이름을 그 값과 일치시키는 쌍입니다. 변수 이름은 술어 절에서 사용된 것이고, 값은 결과 절에서 발견된 값입니다.

인덱스 구조에 대한 변형은 이제 인덱스에서 엔티티 ID / 속성 이름 / 값을 보유했던 위치에 엔티티 ID / 속성 이름 / 값의 바인딩 쌍을 보유한다는 것입니다: 

```clojure
(defn bind-variables-to-query [q-res index]
   (let [seq-res-path (mapcat (partial combine-path-and-meta (from-eav index)) 
                               q-res)         
         res-path (map #(->> %1 (partition 2)(apply (to-eav index))) seq-res-path)] 
     (reduce #(assoc-in %1  (butlast %2) (last %2)) {} res-path)))
     
(defn combine-path-and-meta [from-eav-fn path]
    (let [expanded-path [(repeat (first path)) (repeat (second path)) (last path)] 
          meta-of-path (apply from-eav-fn (map repeat (:db/variable (meta path))))
          combined-data-and-meta-path (interleave meta-of-path expanded-path)]
       (apply (partial map vector) combined-data-and-meta-path)))
```

우리 예시 실행의 3단계 끝에서, 우리는 다음 구조를 가지고 있습니다:
```clojure
{[1 "?e"]{ 
	{[:likes nil]    ["Pizza" nil]}
	{[:name nil]     ["USA" "?nm"]}
	{[:speaks nil]   ["English" nil]} 
	{[:bday nil] ["July 4, 1776" "?bd"]} 
}}
```

#### 4단계: 통합 및 보고

이 시점에서, 우리는 사용자가 처음에 요청한 결과의 상위 집합을 생성했습니다. 이 단계에서는 사용자가 원하는 값들을 추출할 것입니다. 이 과정을 _통합_이라고 합니다: 여기서 바인딩 쌍 구조를 사용자가 쿼리의 `:find` 절에서 정의한 변수 이름들의 벡터와 통합할 것입니다. 

```clojure
(defn unify [binded-res-col needed-vars]
   (map (partial locate-vars-in-query-res needed-vars) binded-res-col))
```  

각 통합 단계는 `locate-vars-in-query-result`에 의해 처리되며, 이는 쿼리 결과(인덱스 항목으로 구조화되지만 바인딩 쌍과 함께)를 반복하여 사용자가 요청한 모든 변수와 값을 감지합니다.

```clojure
(defn locate-vars-in-query-res [vars-set q-res]
   (let [[e-pair av-map]  q-res
         e-res (resultify-bind-pair vars-set [] e-pair)]
     (map (partial resultify-av-pair vars-set e-res)  av-map)))

(defn resultify-bind-pair [vars-set accum pair]
   (let [[ var-name _] pair]
      (if (contains? vars-set var-name) (conj accum pair) accum)))

(defn resultify-av-pair [vars-set accum-res av-pair]
   (reduce (partial resultify-bind-pair vars-set) accum-res av-pair))
```
이 단계의 끝에서, 우리 예시의 결과는 다음과 같습니다:
```
[("?nm" "USA") ("?bd" "July 4, 1776")]
```

#### 쇼 실행하기

마침내 사용자 대면 쿼리 메커니즘인 `q` 매크로에 필요한 모든 구성 요소를 구축했습니다. 이 매크로는 데이터베이스와 쿼리를 인수로 받습니다.

```clojure
(defmacro q
  [db query]
  `(let [pred-clauses#  (q-clauses-to-pred-clauses ~(:where query)) 
         needed-vars# (symbol-col-to-set  ~(:find query))
         query-plan# (build-query-plan pred-clauses#)
         query-internal-res# (query-plan# ~db)]
     (unify query-internal-res# needed-vars#)))
```  
## 요약

우리의 여정은 다른 종류의 데이터베이스에 대한 개념에서 시작하여 다음과 같은 기능을 가진 데이터베이스로 끝났습니다:

* ACI 트랜잭션 지원 (데이터를 인메모리에 저장하기로 결정했을 때 내구성은 잃었습니다).
* "만약에" 상호작용 지원.
* 시간 관련 질문에 답변.
* 인덱스로 최적화된 간단한 데이터로그 쿼리 처리.
* 그래프 쿼리를 위한 API 제공.
* 진화적 쿼리 개념 도입 및 구현.

우리가 개선할 수 있는 것들이 여전히 많이 있습니다: 성능 향상을 위해 여러 구성 요소에 캐싱을 추가하고, 더 풍부한 쿼리를 지원하고, 데이터 내구성을 제공하기 위해 실제 스토리지 지원을 추가하는 것 등을 들 수 있습니다.

하지만 우리의 최종 제품은 매우 많은 일을 할 수 있으며, 488줄의 Clojure 소스 코드로 구현되었습니다. 이 중 73줄은 빈 줄이고 55줄은 문서 문자열입니다.

마지막으로, 아직 빠진 것이 하나 있습니다: 이름입니다.
360줄의 Clojure 코드로 구현된 인메모리, 인덱스 최적화, 쿼리 지원, 라이브러리 개발자 친화적, 시간 인식 함수형 데이터베이스의 유일하게 합리적인 선택은 CircleDB입니다.
