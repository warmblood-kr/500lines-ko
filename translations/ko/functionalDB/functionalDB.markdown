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
#### Removing an Entity

Removing an entity from our database means adding a layer in which it does not exist. To do this, we need to:

* Remove the entity itself
* Update any attributes of other entities that reference it 
* Clear the entity from our indexes

This "construct-without" process is executed by the `remove-entity` function, which looks very similar to `add-entity`:
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
Reference removal is done by the `remove-back-refs` function:
```clojure
(defn- remove-back-refs [db e-id layer]
   (let [reffing-datoms (reffing-to e-id layer)
         remove-fn (fn[d [e a]] (update-entity db e a e-id :db/remove))
         clean-db (reduce remove-fn db reffing-datoms)]
     (last (:layers clean-db))))
```
We begin by using `reffing-datoms-to` to find all entities that reference ours in the given layer; it returns a sequence of triplets that contain the ID of the referencing entity, as well as the attribute name and the ID of the removed entity.
```clojure
(defn- reffing-to [e-id layer]
   (let [vaet (:VAET layer)]
         (for [[attr-name reffing-set] (e-id vaet)
               reffing reffing-set]
              [reffing attr-name])))

```
We then apply `update-entity` to each triplet to update the attributes that reference our removed entity. (We'll explore how `update-entity` works in the next section.)

The last step of `remove-back-refs` is to clear the reference itself from our indexes, and more specifically from the VAET index, since it is the only index that stores reference information. 

#### Updating an Entity

At its essence, an update is the modification of an entity’s attribute’s value. The modification process itself depends on the cardinality of the attribute: an attribute with cardinality `:db/multiple` holds a set of values, so we must allow items to be added to or removed from this set, or the set to be replaced entirely. An attribute with cardinality `:db/single` holds a single value, and thus only allows replacement.  

Since we also have indexes that provide lookups directly on attributes and their values, these will also have to be updated. 

As with `add-entity` and `remove-entity`, we won't actually be modifying our entity in place, but will instead add a new layer which contains the updated entity.

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
To update an attribute, we locate it with `attr-at` and then use `update-attr` to perform the actual update. 
```clojure
(defn- update-attr [attr new-val new-ts operation]
    {:pre  [(if (single? attr)
            (contains? #{:db/reset-to :db/remove} operation)
            (contains? #{:db/reset-to :db/add :db/remove} operation))]}
    (-> attr
       (update-attr-modification-time new-ts)
       (update-attr-value new-val operation)))
```
We use two helper functions to perform the update. `update-attr-modification-time` updates timestamps to reflect the creation of the black arrows in Figure 1:
```clojure
(defn- update-attr-modification-time  
  [attr new-ts]
       (assoc attr :ts new-ts :prev-ts (:ts attr)))
```
`update-attr-value` actually updates the value:
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
All that remains is to remove the old value from the indexes and add the new one to them, and then construct the new layer with all of our updated components. Luckily, we can leverage the code we wrote for adding and removing entities to do this.

### Transactions

Each of the operations in our low-level API acts on a single entity. However, nearly all databases have a way for users to do multiple operations as a single _transaction_. This means: 

* The batch of operations is viewed as a single atomic operation, so all of the operations either succeed together or fail together.
* The database is in a valid state before and after the transaction.
* The batch update appears to be _isolated_; other queries should never see a database state in which only some of the operations have been applied.

We can fulfill these requirements through an interface that consumes a database and a set of operations to be performed, and produces a database whose state reflects the given changes. All of the changes submitted in the batch should be applied through the addition of a _single_ layer. However, we have a problem: All of the functions we wrote in our low-level API add a new layer to the database. If we were to perform a batch with _n_ operations, we would thus see _n_ new layers added, when what we would really like is to have exactly one new layer.   

The key here is that the layer we want is the _top_ layer that would be produced by performing those updates in sequence. Therefore, the solution is to execute the user’s operations one after another, each creating a new layer. When the last layer is created, we take only that top layer and place it on the initial database (leaving all the intermediate layers to pine for the fjords). Only after we've done all this will we update the database's timestamp.

All this is done in the `transact-on-db` function, which receives the initial value of the database and the batch of operations to perform, and returns its updated value. 

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
Note here that we used the term _value_, meaning that only the caller to this function is exposed to the updated state; all other users of the database are unaware of this change (as a database is a value, and therefore cannot change). 
In order to have a system where users can be exposed to state changes performed by others, users do not interact directly with the database, but rather refer to it using another level of indirection. This additional level is implemented using Clojure's `Atom`, a reference type. Here we leverage the main three key features of an `Atom`, which are:

1. It references a value.
2. It is possible to update the referencing of the `Atom` to another value by executing a transaction (using Clojure's Software Transaction Memory capabilities). The transaction accepts an `Atom` and a function. That function operates on the value of the `Atom` and returns a new value. After the execution of the transaction, the `Atom` references the value that was returned from the function.
3. Getting to the value that is referenced by the `Atom` is done by dereferencing it, which returns the state of that `Atom` at that time.

In between Clojure's `Atom` and the work done in `transact-on-db`, there's still a gap to be bridged; namely, to invoke the transaction with the right inputs.

To have the simplest and clearest APIs, we  would like users to just provide the `Atom` and the list of operations, and have the database transform the user input into a proper transaction.

That transformation occurs in the following transaction call chain:

```
transact →  _transact → swap! → transact-on-db
```

Users call `transact` with the `Atom` (i.e., the connection) and the operations to perform, which relays its input to `_transact`, adding to it the name of the function that updates the `Atom` (`swap!`).

```clojure
(defmacro transact [db-conn & txs]  `(_transact ~db-conn swap! ~@txs))
```

`_transact` prepares the call to `swap!`. It does so by creating a list that begins with `swap!`, followed by the `Atom`, then the `transact-on-db` symbol and the batch of operations.

```clojure
(defmacro  _transact [db op & txs]
   (when txs
     (loop [[frst-tx# & rst-tx#] txs  res#  [op db `transact-on-db]  accum-txs# []]
       (if frst-tx#
           (recur rst-tx# res#  (conj  accum-txs#  (vec frst-tx#)))
           (list* (conj res#  accum-txs#))))))
```

`swap!` invokes `transact-on-db` within a transaction (with the previously prepared arguments), and `transact-on-db` creates the new state of the database and returns it.

At this point we can see that with few minor tweaks, we can also provide a way to ask "what if" questions. This can be done by replacing `swap!` with a function that would not make any change to the system. This scenario is implemented with the `what-if` call chain:

`what-if` $\to$ `_transact` $\to$ `_what-if` $\to$ `transact-on-db`

The user calls `what-if` with the database value and the operations to perform. It then relays these inputs to `_transact`, adding to them a function that mimics `swap!`'s APIs, without its effect (callled `_what-if`).  

```clojure
(defmacro what-if [db & ops]  `(_transact ~db _what-if  ~@ops))
```

`_transact` prepares the call to `_what-if`. It does so by creating a list that begins with `_what-if`, followed by the database, then the `transact-on-db` symbol and the batch of operations.  `_what-if` invokes `transact-on-db`, just like `swap!` does in the transaction scenario, but does not inflict any change on the system.

```clojure
(defn- _what-if [db f txs]  (f db txs))
```
 
Note that we are not using functions, but macros. The reason for using macros here is that arguments to macros do not get evaluated as the call happens; this allows us to offer a cleaner API design where the user provides the operations structured in the same way that any function call is structured in Clojure. 

The above process can be seen in the following examples. For Transaction, the user call: 
```clojure
(transact db-conn  (add-entity e1) (update-entity e2 atr2 val2 :db/add))  
```
changes into: 
```clojure
(_transact db-conn swap! (add-entity e1) (update-entity e2 atr2 val2 :db/add))
```
which becomes: 
```clojure
(swap! db-conn transact-on-db [[add-entity e1][update-entity e2 atr2 val2 :db/add]])
```

For what-if, the user call:

```clojure
(what-if my-db (add-entity e3) (remove-entity e4))
```
changes into: 
```clojure
(_transact my-db _what-if (add-entity e3) (remove-entity e4))
```
then:
```clojure
(_what-if my-db transact-on-db [[add-entity e3] [remove-entity e4]])
```
and eventually: 
```clojure
(transact-on-db my-db  [[add-entity e3] [remove-entity e4]])
```

## Insight Extraction as Libraries

At this point we have the core functionality of the database in place, and it is time to add its *raison d'être*: insights extraction. The architecture approach we used here is to allow adding these capabilities as libraries, as different usages of the database would need different such mechanisms. 

### Graph Traversal

A reference connection between entities is created when an entity’s attribute’s type is `:db/ref`, which means that the value of that attribute is an ID of another entity. When a referring entity is added to the database, the reference is indexed at the VAET index.  
The information found in the VAET index can be leveraged to extract all the incoming links to an entity. This is done in the `incoming-refs` function, which collects all the leaves that are reachable from the entity at that index:

```clojure
(defn incoming-refs [db ts ent-id & ref-names]
   (let [vaet (indx-at db :VAET ts)
         all-attr-map (vaet ent-id)
         filtered-map (if ref-names 
                          (select-keys ref-names all-attr-map) 
                          all-attr-map)]
      (reduce into #{} (vals filtered-map))))
```
We can also go through all of a given entity’s attributes and collect all the values of attributes of type `:db/ref`, and by that extract all the outgoing references from that entity. This is done by the `outgoing-refs` function.

```clojure
(defn outgoing-refs [db ts ent-id & ref-names]
   (let [val-filter-fn (if ref-names #(vals (select-keys ref-names %)) vals)]
   (if-not ent-id []
     (->> (entity-at db ts ent-id)
          (:attrs) (val-filter-fn) (filter ref?) (mapcat :value)))))
```
These two functions act as the basic building blocks for any graph traversal operation, as they are the ones that raise the level of abstraction from entities and attributes to nodes and links in a graph. Once we have the ability to look at our database as a graph, we can provide various graph traversing and querying APIs. We leave this as a solved exercise to the reader; one solution can be found in the chapter's source code (see `graph.clj`).   


## Querying the Database

The second library we present provides querying capabilities, which is the main concern of this section. 
A database is not very useful to its users without a powerful query mechanism. This feature is usually exposed to users through a _query language_ that is used to declaratively specify the set of data of interest. 

Our data model is based on accumulation of facts (i.e., datoms) over time. For this model, a natural place to look for the right query language is _logic programming_. A commonly used query language influenced by logic programming is _Datalog_ which, in addition to being well-suited for our data model, has a very elegant adaptation to Clojure’s syntax. Our query engine will implement a subset of the Datalog language from the [Datomic database](http://docs.datomic.com/query.html).

### Query Language

Let's look at an example query in our proposed language. This query asks: "What are the names and birthdays of entities who like pizza, speak English, and who have a birthday this month?"
```clojure
{  :find [?nm ?bd ]
   :where [
      [?e  :likes "pizza"]
      [?e  :name  ?nm]
      [?e  :speak "English"]
      [?e  :bday (bday-mo? ?bd)]]}
```
#### Syntax

We use the syntax of Clojure’s data literals directly to provide the basic syntax for our queries. This allows us to avoid having to write a specialized parser, while still providing a form that is familiar and easily readable to programmers familiar with Clojure.

A query is a map with two items:

* An item with `:where` as a key, and with a _rule_ as a value. A rule is a vector of _clauses_, and a clause is a vector composed of three _predicates_, each of which operates on a different component of a datom.  In the example above, `[?e  :likes "pizza"]` is a clause.  This `:where` item defines a rule that acts as a filter on datoms in our database (like a SQL `WHERE` clause.)
* An item with `:find` as a key, and with a vector as a value. The vector defines which components of the selected datom should be projected into the results (like a SQL `SELECT` clause.)

The description above omits a crucial requirement: how to make different clauses sync on a value (i.e., make a join operation between them), and how to structure the found values in the output (specified by the `:find` part). 

We fulfill both of these requirements using _variables_, which are denoted with a leading `?`. The only exception to this definition is the "don't care" variable `_`  (underscore).  

A clause in a query is composed of three predicates; \aosatblref{500l.functionaldb.predicates} defines what can act as a predicate in our query language.

<markdown>
<table>
  <tr>
    <td>Name</td>
    <td>Meaning</td>
    <td>Example</td>
  </tr>
  <tr>
    <td>Constant</td>
    <td>Is the value of the item in the datom equal to the constant?</td>
    <td>:likes</td>
  </tr>
  <tr>
    <td>Variable</td>
    <td>Bind the value of the item in the datom to the variable and return true.</td>
    <td>?e</td>
  </tr>
  <tr>
    <td>Don’t-care</td>
    <td>Always returns true.</td>
    <td>_</td>
  </tr>
  <tr>
    <td>Unary operator</td>
    <td>Unary operation that takes a variable as its operand.<br/>
        Bind the datom's item's value to the variable (unless it's an '_').<br/>
        Replace the variable with the value of the item in the datom.<br/>
        Return the application of the operation.</td>
    <td>(bday-mo? _)</td>
  </tr>
  <tr>
    <td>Binary operator</td>
    <td>A binary operation that must have a variable as one of its operands.<br/>
        Bind the datom's item's value to the variable (unless it's an '_').<br/>        
        Replace the variable with the value of the item in the datom.<br/>
        Return the result of the operation.</td>
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
\textbf{Name} & \textbf{Meaning} & \textbf{Example} \\
\hline
Constant & Is the value of the datom item equal to the constant? & \verb|:likes| \\
Variable & Bind the value of the datom item to the variable and return true. & \verb|?e| \\
Don't-care & Always returns true. & \verb|_| \\
Unary operator & \begin{tabular}{@{}l@{}} Unary operation that takes a variable as its operand. \\ Bind the datom's item's value to the variable (unless it's an \verb|_|). \\  Replace the variable with the value of the item in the datom. \\ Return the application of the operation. \end{tabular} & \verb|(bday-mo? _)| \\
Binary operator & \begin{tabular}{@{}l@{}} A binary operation that requires a variable as an operand. \\ Bind the datom's item's value to the variable (unless it's an \verb|_|). \\ Replace the variable with the value of the item in the datom. \\ Return the result of the operation. \end{tabular} & \verb|(&gt; ?age 20)| \\
\hline
\end{tabular}
}
\caption{Predicates}
\label{500l.functionaldb.predicates}
\end{table}
</latex>

#### Limitations of our Query Language 

Engineering is all about managing tradeoffs, and designing our query engine is no different. In our case, the main tradeoff we must address is feature-richness versus complexity. Resolving this tradeoff requires us to look at common use-cases of the system, and from there deciding what limitations would be acceptable. 

In our database, we decided to build a query engine with the following limitations:

* Users cannot define logical operations between the clauses; they are always ‘ANDed’ together. (This can be worked around by using unary or binary predicates.)
* If there is more than one clause in a query, there must be one variable that is found in all of the clauses of that query. This variable acts as a joining variable. This limitation simplifies the query optimizer.
* A query is only executed on a single database. 

While these design decisions result in a query language that is less rich than Datalog, we are still able to support many types of simple but useful queries.

### Query Engine Design

While our query language allows the user to specify _what_ they want to access, it hides the details of _how_ this will be accomplished. The query engine is the database component responsible for yielding the data for a given query. 

This involves four steps:

1. Transformation to internal representation: Transform the query from its textual form into a data structure that is consumed by the query planner.
2. Building a query plan: Determine an efficient _plan_ for yielding the results of the given query. In our case, a query plan is a function to be invoked.
3. Executing the plan: Execute the plan and send its results to the next phase.
4. Unification and reporting: Extract only the results that need to be reported and format them as specified.

#### Phase 1: Transformation

In this phase, we transform the given query from a representation that is easy for the user to understand into a representation that can be consumed efficiently by the query planner. 

The `:find` part of the query is transformed into a set of the given variable names:

```clojure
(defmacro symbol-col-to-set [coll] (set (map str coll)))
```

The `:where` part of the query retains its nested vector structure. However, each of the terms in each of the clauses is replaced with a predicate according to \aosatblref{500l.functionaldb.predicates}. 

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

For each clause, a vector with the variable names used in that clause is set as its metadata. 

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

Iterating over the clauses themselves happens in `q-clauses-to-pred-clauses`:
          
```clojure
(defmacro  q-clauses-to-pred-clauses [clauses]
     (loop [[frst# & rst#] clauses preds-vecs# []]
       (if-not frst#  preds-vecs#
         (recur rst# `(conj ~preds-vecs# (pred-clause ~frst#))))))
```
We are once again relying on the fact that macros do not eagerly evaluate their arguments. This allows us to define a simpler API where users provide variable names as symbols (e.g., `?name`) instead of asking the user to understand the internals of the engine by providing variable names as strings ( e.g., `"?name"`), or even worse, quoting the variable name (e.g., `'?name`).

At the end of this phase, our example yields the following set for the `:find` part: 

```clojure 
#{"?nm" "?bd"} 
``` 

and the following structure in \aosatblref{500l.functionaldb.clauses} for the `:where` part. (Each cell in the _Predicate Clause_ column holds the metadata found in its neighbor at the _Meta Clause_ column.)

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

This structure acts as the query that is executed in a later phase, once the engine decides on the right plan of execution.

#### Phase 2: Making a Plan

In this phase, we inspect the query in order to construct a good plan to produce the result it describes.

In general, this will involve choosing the appropriate index (\aosatblref{500l.functionaldb.indexselection}) and constructing a plan in the form of a function.  We choose the index based on the _single_ joining variable (that can operate on only a single kind of element).

<markdown>
<table>
	<tr>
		<td>Joining variable operates on</td><td>Index to use</td>
	</tr>
	<tr>
		<td>Entity IDs</td><td>AVET</td>
	</tr>
	<tr>
		<td>Attribute names</td><td>VEAT</td>
	</tr>
	<tr>
		<td>Attribute values</td><td>EAVT</td>
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
\textbf{Joining variable operates on} & \textbf{Index to use} \\
\hline
Entity IDs & AVET \\
Attribute names & VEAT \\
Attribute values & EAVT \\
\hline
\end{tabular}
}
\caption{Index Selection}
\label{500l.functionaldb.indexselection}
\end{table}
</latex>

The reasoning behind this mapping will become clearer in the next section, when we actually execute the plan produced. For now, just note that the key here is to select an index whose leaves hold the elements that the joining variable operates on.

Locating the index of the joining variable is done by `index-of-joining-variable`:

```clojure
(defn index-of-joining-variable [query-clauses]
   (let [metas-seq  (map #(:db/variable (meta %)) query-clauses) 
         collapsing-fn (fn [accV v] (map #(when (= %1 %2) %1)  accV v))
         collapsed (reduce collapsing-fn metas-seq)] 
     (first (keep-indexed #(when (variable? %2 false) %1)  collapsed)))) 
```
We begin by extracting the metadata of each clause in the query. This extracted metadata is a 3-element vector; each element is either a variable name or nil. (Note that there is no more than one variable name in that vector.) Once the vector is extracted, we produce from it (by reducing it) a single value, which is either a variable name or nil. If a variable name is produced, then it appeared in all of the metadata vectors at the same index; i.e., this is the joining variable. We can thus choose to use the index relevant for this joining variable based on the mapping described above.

Once the index is chosen, we construct our plan, which is a function that closes over the query and the index name and executes the operations necessary to return the query results.
 

```clojure
(defn build-query-plan [query]
   (let [term-ind (index-of-joining-variable query)
         ind-to-use (case term-ind 0 :AVET 1 :VEAT 2 :EAVT)]
      (partial single-index-query-plan query ind-to-use)))
```

In our example the chosen index is the `AVET` index, as the joining variable acts on the entity IDs.

#### Phase 3: Execution of the Plan

We saw in the previous phase that our query plan ends by calling `single-index-query-plan`. This function will:

1. Apply each predicate clause on an index (each predicate on its appropriate index level).
2. Perform an AND operation across the results.
3. Merge the results into a simpler data structure.

```clojure
(defn single-index-query-plan [query indx db]
   (let [q-res (query-index (indx-at db indx) query)]
     (bind-variables-to-query q-res (indx-at db indx))))
```
To better explain this process we'll demonstrate it using our exemplary query, assuming that our database holds the entities in \aosatblref{500l.functionaldb.exampleentities}.

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

Now it is time to go deeper into the rabbit hole and take a look at the `query-index` function, where our query finally begins to yield some results:

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
This function starts by applying the predicate clauses on the previously chosen index. Each application of a predicate clause on an index returns a _result clause_. 

The main characteristics of a result are:

1. It is built of three items, each from a different level of the index, and each passed its respective predicate. 
2. The order of items matches the index's levels structure. (Predicate clauses are always in EAV order.) The re-ordering is done when applying the index's `from-eav` on the predicate clause. 
3. The metadata of the predicate clause is attached to it. 

All of this is done in the function `filter-index`.

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
Assuming the query was executed on July 4th, the results of executing it on the above data are seen in \aosatblref{500l.functionaldb.queryresults}.
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

Once we have produced all of the result clauses, we need to perform an `AND` operation between them. This is done by finding all of the elements that passed all the predicate clauses:

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

In our example, the result of this step is a set that holds the value *1* (which is the entity ID of USA). 

We now have to remove the items that didn’t pass all of the conditions:

```clojure
(defn mask-path-leaf-with-items [relevant-items path]
     (update-in path [2] CS/intersection relevant-items))
```

Finally, we remove all of the result clauses that are "empty" (i.e., their last item is empty). We do this in the last line of the `query-index` function. Our example leaves us with the items in \aosatblref{500l.functionaldb.filteredqueryresults}.

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

We are now ready to report the results. The result clause structure is unwieldy for this purpose, so we will convert it into an an index-like structure (map of maps)&mdash;with a significant twist. 

To understand the twist, we must first introduce the idea of a _binding pair_, which is a pair that matches a variable name to its value. The variable name is the one used at the predicate clauses, and the value is the value found in the result clauses.

The twist to the index structure is that now we hold a binding pair of the entity-id / attr-name / value in the location where we held an entity-id / attr-name / value in an index: 

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

At the end of phase 3 of our example execution, we have the following structure at hand:
```clojure
{[1 "?e"]{ 
	{[:likes nil]    ["Pizza" nil]}
	{[:name nil]     ["USA" "?nm"]}
	{[:speaks nil]   ["English" nil]} 
	{[:bday nil] ["July 4, 1776" "?bd"]} 
}}
```

#### Phase 4: Unify and Report

At this point, we’ve produced a superset of the results that the user initially asked for. In this phase, we'll extract the values that the user wants. This process is called _unification_: it is here that we will unify the binding pairs structure with the vector of variable names that the user defined in the `:find` clause of the query. 

```clojure
(defn unify [binded-res-col needed-vars]
   (map (partial locate-vars-in-query-res needed-vars) binded-res-col))
```  

Each unification step is handled by `locate-vars-in-query-result`, which iterates over a query result (structured as an index entry, but with binding pairs) to detect all the variables and values that the user asked for.

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
At the end of this phase, the results for our example are:
```
[("?nm" "USA") ("?bd" "July 4, 1776")]
```

#### Running the Show

We've finally built all of the components we need for our user-facing query mechanism, the `q` macro, which receives as arguments a database and a query.

```clojure
(defmacro q
  [db query]
  `(let [pred-clauses#  (q-clauses-to-pred-clauses ~(:where query)) 
         needed-vars# (symbol-col-to-set  ~(:find query))
         query-plan# (build-query-plan pred-clauses#)
         query-internal-res# (query-plan# ~db)]
     (unify query-internal-res# needed-vars#)))
```  
## Summary

Our journey started with a conception of a different kind of database, and ended with one that:

* Supports ACI transactions (durability was lost when we decided to have the data stored in-memory).
* Supports "what if" interactions.
* Answers time-related questions.
* Handles simple datalog queries that are optimized with indexes.
* Provides APIs for graph queries.
* Introduces and implements the notion of evolutionary queries.

There are still many things that we could improve: We could add caching to several components to improve performance; support richer queries; and add real storage support to provide data durability, to name a few.

However, our final product can do a great many things, and was implemented in 488 lines of Clojure source code, 73 of which are blank lines and 55 of which are docstrings. 

Finally, there's one thing that is still missing: a name. 
The only sensible option for an in-memory, index-optimized, query-supporting, library developer-friendly, time-aware functional database implemented in 360 lines of Clojure code is CircleDB.
