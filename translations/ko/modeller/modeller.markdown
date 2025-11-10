title: 3D 모델러
author: Erick Dransch
<markdown>
_[Erick Dransch](http://erickdransch.com)는 소프트웨어 개발자이자 2D 및 3D 컴퓨터 그래픽스 애호가입니다. 그는 비디오 게임, 3D 특수 효과 소프트웨어, 컴퓨터 지원 설계 도구 개발에 참여했습니다. 현실을 시뮬레이션하는 것과 관련된 일이라면 무엇이든 더 배우고 싶어합니다._
</markdown>
## 개요
인간은 본능적으로 창조적입니다. 우리는 계속해서 새롭고 유용하며 흥미로운 것들을 설계하고 만듭니다. 현대에 들어서는 설계와 창조 과정을 지원하는 소프트웨어를 작성하고 있습니다.
컴퓨터 지원 설계(CAD) 소프트웨어는 창작자들이 건물, 교량, 비디오 게임 아트,
영화 몬스터, 3D 프린팅 가능한 객체 등 많은 것들을 물리적인 버전을 만들기 전에 설계할 수 있게 해줍니다.

CAD 도구의 핵심은 3차원 설계를 2차원 화면에서 보고 편집할 수 있는 것으로 추상화하는 방법입니다.
이러한 정의를 충족하기 위해 CAD 도구는 세 가지 기본 기능을 제공해야 합니다.
첫째, 설계되고 있는 객체를 나타내는 데이터 구조가 있어야 합니다. 이는 사용자가 구축하고 있는 3차원 세계에 대한 컴퓨터의 이해입니다.
둘째, CAD 도구는 사용자의 화면에 설계를 표시할 방법을 제공해야 합니다. 사용자는 3차원을 가진 물리적 객체를 설계하고 있지만, 컴퓨터 화면은 2차원만을 가집니다.
CAD 도구는 우리가 객체를 어떻게 인식하는지 모델링하고, 사용자가 객체의 모든 3차원을 이해할 수 있는 방식으로 화면에 그려야 합니다.
셋째, CAD 도구는 설계되는 객체와 상호작용하는 방법을 제공해야 합니다. 사용자는 원하는 결과를 얻기 위해 설계에 추가하고 수정할 수 있어야 합니다.
또한, 모든 도구는 사용자들이 협업하고, 공유하며, 작업을 저장할 수 있도록 디스크에서 설계를 저장하고 불러오는 방법이 필요합니다.

도메인별 CAD 도구는 해당 도메인의 특정 요구사항에 맞는 많은 추가 기능을 제공합니다. 예를 들어, 건축 CAD 도구는 건물에 가해지는 기후 스트레스를 테스트하는 물리 시뮬레이션을 제공하고,
3D 프린팅 도구는 객체가 실제로 프린팅 가능한지 확인하는 기능을 가지며, 전기 CAD 도구는 구리를 통해 흐르는 전기의 물리학을 시뮬레이션하고, 영화 특수 효과 제품군은
정확하게 불꽃 역학을 시뮬레이션하는 기능을 포함할 것입니다.

하지만 모든 CAD 도구는 최소한 위에서 논의한 세 가지 기능은 포함해야 합니다: 설계를 나타내는 데이터 구조, 이를 화면에 표시하는 능력, 그리고 설계와 상호작용하는 방법.

이를 염두에 두고, Python 500줄로 3D 설계를 어떻게 나타내고, 화면에 표시하며, 상호작용할 수 있는지 살펴보겠습니다.

## 렌더링을 가이드로 활용하기
3D 모델러의 많은 설계 결정들의 추진력은 렌더링 과정입니다.
우리는 설계에서 복잡한 객체들을 저장하고 렌더링할 수 있기를 원하지만, 렌더링 코드의 복잡성은 낮게 유지하고 싶습니다.
렌더링 과정을 살펴보고, 간단한 렌더링 로직으로 임의로 복잡한 객체들을 저장하고 그릴 수 있게 해주는 설계용 데이터 구조를 탐구해봅시다.

### 인터페이스와 메인 루프 관리
렌더링을 시작하기 전에 설정해야 할 몇 가지가 있습니다. 먼저, 설계를 표시할 윈도우를 만들어야 합니다.
둘째, 화면에 렌더링하기 위해 그래픽 드라이버와 통신하고 싶습니다.
그래픽 드라이버와 직접 통신하기보다는, OpenGL이라는 크로스 플랫폼 추상화 레이어와
윈도우를 관리하기 위한 GLUT(OpenGL Utility Toolkit)라는 라이브러리를 사용합니다.

#### OpenGL에 대한 참고사항
<!-- @mikedebo: Are we going to have actual sidebars in the book? If so we can make this into a sidebar (together with the paragraph on GLUT). Keep in mind sidebars can be hard (but not impossible) to do in ebooks. I wouldn't do sidebars unless there are at least three chapters which use them. -->
OpenGL은 크로스 플랫폼 개발을 위한 그래픽 응용 프로그래밍 인터페이스입니다. 플랫폼 간 그래픽 애플리케이션 개발을 위한 표준 API입니다.
OpenGL에는 두 가지 주요 변형이 있습니다: 레거시 OpenGL과 모던 OpenGL.

OpenGL에서의 렌더링은 정점과 법선으로 정의된 폴리곤을 기반으로 합니다. 예를 들어, 정육면체의 한 면을 렌더링하기 위해서는 4개의 정점과 그 면의 법선을 명시합니다.

레거시 OpenGL은 "고정 함수 파이프라인"을 제공합니다. 전역 변수를 설정함으로써 프로그래머는 조명, 색칠, 면 제거 등과 같은 기능의 자동화된 구현을 활성화하고 비활성화할 수 있습니다. 그러면 OpenGL이 활성화된 기능으로 자동으로 장면을 렌더링합니다. 이 기능은 더 이상 사용되지 않습니다.

반면, 모던 OpenGL은 프로그래머가 전용 그래픽 하드웨어(GPU)에서 실행되는 "셰이더"라고 불리는 작은 프로그램들을 작성하는 프로그래밍 가능한 렌더링 파이프라인을 특징으로 합니다. 모던 OpenGL의 프로그래밍 가능한 파이프라인이 레거시 OpenGL을 대체했습니다.

이 프로젝트에서는 더 이상 사용되지 않음에도 불구하고 레거시 OpenGL을 사용합니다. 레거시 OpenGL이 제공하는 고정 기능은 코드 크기를 작게 유지하는데 매우 유용합니다. 필요한 선형 대수학 지식의 양을 줄이고, 우리가 작성할 코드를 단순화합니다.

#### GLUT에 대해
OpenGL과 함께 번들로 제공되는 GLUT은 운영체제 윈도우를 생성하고 사용자 인터페이스 콜백을 등록할 수 있게 해줍니다. 이러한 기본 기능은 우리 목적에 충분합니다. 윈도우 관리와 사용자 상호작용을 위한 더 완전한 기능을 가진 라이브러리를 원한다면, GTK나 Qt 같은 완전한 윈도잉 툴킷 사용을 고려할 것입니다.

#### 뷰어
GLUT과 OpenGL의 설정을 관리하고, 모델러의 나머지 부분을 구동하기 위해 `Viewer`라는 클래스를 생성합니다.
우리는 윈도우 생성과 렌더링을 관리하고, 프로그램의 메인 루프를 포함하는 단일 `Viewer` 인스턴스를 사용합니다.
`Viewer`의 초기화 과정에서 GUI 윈도우를 생성하고 OpenGL을 초기화합니다.

`init_interface` 함수는 모델러가 렌더링될 윈도우를 생성하고 설계가 렌더링되어야 할 때 호출될 함수를 지정합니다.
`init_opengl` 함수는 프로젝트에 필요한 OpenGL 상태를 설정합니다. 행렬을 설정하고, 뒷면 제거를 활성화하며,
장면을 비추기 위한 조명을 등록하고, 객체들이 색칠되기를 원한다고 OpenGL에 알려줍니다.
`init_scene` 함수는 `Scene` 객체들을 생성하고 사용자가 시작할 수 있도록 초기 노드들을 배치합니다. `Scene` 데이터 구조에 대해서는 곧 더 자세히 살펴볼 것입니다.
마지막으로, `init_interaction`은 나중에 논의할 사용자 상호작용을 위한 콜백들을 등록합니다.

`Viewer`를 초기화한 후, `glutMainLoop`을 호출하여 프로그램 실행을 GLUT에 넘깁니다. 이 함수는 절대 리턴되지 않습니다. GLUT 이벤트에 등록한 콜백들은 해당 이벤트들이 발생할 때 호출될 것입니다.

```python
class Viewer(object):
    def __init__(self):
        """ Initialize the viewer. """
        self.init_interface()
        self.init_opengl()
        self.init_scene()
        self.init_interaction()
        init_primitives()

    def init_interface(self):
        """ initialize the window and register the render function """
        glutInit()
        glutInitWindowSize(640, 480)
        glutCreateWindow("3D Modeller")
        glutInitDisplayMode(GLUT_SINGLE | GLUT_RGB)
        glutDisplayFunc(self.render)

    def init_opengl(self):
        """ initialize the opengl settings to render the scene """
        self.inverseModelView = numpy.identity(4)
        self.modelView = numpy.identity(4)

        glEnable(GL_CULL_FACE)
        glCullFace(GL_BACK)
        glEnable(GL_DEPTH_TEST)
        glDepthFunc(GL_LESS)

        glEnable(GL_LIGHT0)
        glLightfv(GL_LIGHT0, GL_POSITION, GLfloat_4(0, 0, 1, 0))
        glLightfv(GL_LIGHT0, GL_SPOT_DIRECTION, GLfloat_3(0, 0, -1))

        glColorMaterial(GL_FRONT_AND_BACK, GL_AMBIENT_AND_DIFFUSE)
        glEnable(GL_COLOR_MATERIAL)
        glClearColor(0.4, 0.4, 0.4, 0.0)

    def init_scene(self):
        """ initialize the scene object and initial scene """
        self.scene = Scene()
        self.create_sample_scene()

    def create_sample_scene(self):
        cube_node = Cube()
        cube_node.translate(2, 0, 2)
        cube_node.color_index = 2
        self.scene.add_node(cube_node)

        sphere_node = Sphere()
        sphere_node.translate(-2, 0, 2)
        sphere_node.color_index = 3
        self.scene.add_node(sphere_node)

        hierarchical_node = SnowFigure()
        hierarchical_node.translate(-2, 0, -2)
        self.scene.add_node(hierarchical_node)

    def init_interaction(self):
        """ init user interaction and callbacks """
        self.interaction = Interaction()
        self.interaction.register_callback('pick', self.pick)
        self.interaction.register_callback('move', self.move)
        self.interaction.register_callback('place', self.place)
        self.interaction.register_callback('rotate_color', self.rotate_color)
        self.interaction.register_callback('scale', self.scale)

    def main_loop(self):
        glutMainLoop()

if __name__ == "__main__":
    viewer = Viewer()
    viewer.main_loop()
```
`render` 함수에 대해 자세히 살펴보기 전에, 선형대수학에 대해 약간 논의해야 합니다.

### 좌표 공간
우리 목적상, 좌표 공간은 원점과 보통 $x$, $y$, $z$ 축인 3개의 기저 벡터 집합입니다.

### 점
3차원의 어떤 점이든 원점으로부터 $x$, $y$, $z$ 방향의 오프셋으로 나타낼 수 있습니다. 점의 표현은 그 점이 속한 좌표 공간에 상대적입니다. 같은 점이
다른 좌표 공간에서는 다른 표현을 가집니다. 3차원의 어떤 점이든 어떤 3차원 좌표 공간에서든 나타낼 수 있습니다.

### 벡터
벡터는 두 점 간의 차이를 각각 $x$, $y$, $z$ 축에서 나타내는 $x$, $y$, $z$ 값입니다.

### 변환 행렬
컴퓨터 그래픽스에서는 다른 유형의 점들에 대해 여러 개의 다른 좌표 공간을 사용하는 것이 편리합니다. 변환 행렬은 한 좌표 공간에서 다른 좌표 공간으로 점들을 변환합니다.
벡터 $v$를 한 좌표 공간에서 다른 공간으로 변환하기 위해, 변환 행렬 $M$을 곱합니다: $v' = M v$.
일반적인 변환 행렬로는 이동, 크기 조정, 회전이 있습니다.

### 모델, 월드, 뷰, 프로젝션 좌표 공간
\aosafigure[250pt]{modeller-images/newtranspipe.png}{변환 파이프라인}{500l.modeller.newtranspipe}

화면에 항목을 그리기 위해서는 몇 가지 다른 좌표 공간 사이를 변환해야 합니다.

Eye Space에서 Viewport Space까지의 모든 변환을 포함하는 \aosafigref{500l.modeller.newtranspipe}[^transimage]의 오른쪽 부분은 모두 OpenGL이 우리를 위해 처리해줄 것입니다.

[^transimage]: 이미지를 제공해주신 Dr. Anton Gerdelan에게 감사드립니다. 그의 OpenGL 튜토리얼 책은 [http://antongerdelan.net/opengl/](http://antongerdelan.net/opengl/)에서 이용할 수 있습니다.

눈 공간에서 동질 클립 공간으로의 변환은 `gluPerspective`에 의해 처리되고, 정규화된 장치 공간과 뷰포트 공간으로의 변환은 `glViewport`에 의해 처리됩니다.
이 두 행렬은 함께 곱해져서 GL_PROJECTION 행렬로 저장됩니다.
이 프로젝트에서는 이러한 행렬들이 어떻게 작동하는지에 대한 용어나 세부사항을 알 필요가 없습니다.

하지만, 다이어그램의 왼쪽 부분은 우리가 직접 관리해야 합니다. 모델(메쉬라고도 함)의 점들을 모델 공간에서 월드 공간으로 변환하는 행렬을 정의하는데, 이를 모델 행렬이라고 합니다. 또한 월드 공간에서 눈 공간으로 변환하는 뷰 행렬도 정의합니다.
이 프로젝트에서는 이 두 행렬을 결합하여 ModelView 행렬을 얻습니다.

전체 그래픽스 렌더링 파이프라인과 관련된 좌표 공간에 대해 더 알아보려면, [*Real Time Rendering*](http://www.realtimerendering.com/)의 2장이나 다른 컴퓨터 그래픽스 입문서를 참조하세요.

### 뷰어를 통한 렌더링
`render` 함수는 렌더 시간에 수행되어야 하는 OpenGL 상태를 설정하는 것으로 시작됩니다. `init_view`를 통해 프로젝션 행렬을 초기화하고, 장면 공간에서 월드 공간으로 변환하는 변환 행렬로 ModelView 행렬을 초기화하기 위해 상호작용 멤버의 데이터를 사용합니다. Interaction 클래스에 대해서는 아래에서 더 자세히 살펴볼 것입니다. `glClear`로 화면을 지우고 장면에 자기 자신을 렌더링하도록 지시한 다음, 단위 격자를 렌더링합니다.

격자를 렌더링하기 전에 OpenGL의 조명을 비활성화합니다. 조명이 비활성화된 상태에서 OpenGL은 광원을 시뮬레이션하기보다 단색으로 항목들을 렌더링합니다. 이렇게 하면 격자가 장면과 시각적으로 구별됩니다.
마지막으로, `glFlush`는 버퍼를 플러시하고 화면에 표시할 준비가 되었다고 그래픽 드라이버에 신호를 보냅니다.

```python
    # class Viewer
    def render(self):
        """ The render pass for the scene """
        self.init_view()

        glEnable(GL_LIGHTING)
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

        # Load the modelview matrix from the current state of the trackball
        glMatrixMode(GL_MODELVIEW)
        glPushMatrix()
        glLoadIdentity()
        loc = self.interaction.translation
        glTranslated(loc[0], loc[1], loc[2])
        glMultMatrixf(self.interaction.trackball.matrix)

        # store the inverse of the current modelview.
        currentModelView = numpy.array(glGetFloatv(GL_MODELVIEW_MATRIX))
        self.modelView = numpy.transpose(currentModelView)
        self.inverseModelView = inv(numpy.transpose(currentModelView))

        # render the scene. This will call the render function for each object
        # in the scene
        self.scene.render()

        # draw the grid
        glDisable(GL_LIGHTING)
        glCallList(G_OBJ_PLANE)
        glPopMatrix()

        # flush the buffers so that the scene can be drawn
        glFlush()

    def init_view(self):
        """ initialize the projection matrix """
        xSize, ySize = glutGet(GLUT_WINDOW_WIDTH), glutGet(GLUT_WINDOW_HEIGHT)
        aspect_ratio = float(xSize) / float(ySize)

        # load the projection matrix. Always the same
        glMatrixMode(GL_PROJECTION)
        glLoadIdentity()

        glViewport(0, 0, xSize, ySize)
        gluPerspective(70, aspect_ratio, 0.1, 1000.0)
        glTranslated(0, 0, -15)

```
### 무엇을 렌더링할 것인가: 장면
이제 월드 좌표 공간에서 그리기를 처리하는 렌더링 파이프라인을 초기화했으니, 무엇을 렌더링할 것인가요? 우리의 목표는
3D 모델들로 구성된 설계를 갖는 것임을 기억하세요. 설계를 담을 데이터 구조가 필요하고, 설계를 렌더링하기 위해 이 데이터 구조를 사용해야 합니다.
위에서 뷰어의 렌더 루프에서 `self.scene.render()`를 호출한다는 점을 주목하세요. 장면이란 무엇일까요?

`Scene` 클래스는 설계를 표현하는 데 사용하는 데이터 구조의 인터페이스입니다. 데이터 구조의 세부사항을 추상화하고
렌더링, 아이템 추가, 아이템 조작 등 설계와 상호작용하는 데 필요한 인터페이스 함수들을 제공합니다. 뷰어가 소유하는 하나의 `Scene` 객체가 있습니다.
`Scene` 인스턴스는 `node_list`라고 불리는 장면의 모든 아이템들의 목록을 보관합니다. 또한 선택된 아이템도 추적합니다.
장면의 `render` 함수는 단순히 `node_list`의 각 멤버에 대해 `render`를 호출합니다.

```python
class Scene(object):

    # the default depth from the camera to place an object at
    PLACE_DEPTH = 15.0

    def __init__(self):
        # The scene keeps a list of nodes that are displayed
        self.node_list = list()
        # Keep track of the currently selected node.
        # Actions may depend on whether or not something is selected
        self.selected_node = None

    def add_node(self, node):
        """ Add a new node to the scene """
        self.node_list.append(node)

    def render(self):
        """ Render the scene. """
        for node in self.node_list:
            node.render()
```

### 노드
Scene의 `render` 함수에서 Scene의 `node_list`에 있는 각 아이템에 대해 `render`를 호출합니다. 그런데 그 목록의 요소들은 무엇일까요? 우리는 그것들을 *노드*라고 부릅니다.
개념적으로, 노드는 장면에 배치될 수 있는 모든 것입니다.
객체 지향 소프트웨어에서 우리는 `Node`를 추상 기본 클래스로 작성합니다. `Scene`에 배치될 객체들을 나타내는 모든 클래스들은 `Node`를 상속받을 것입니다.
이 기본 클래스는 장면에 대해 추상적으로 추론할 수 있게 해줍니다.
코드베이스의 나머지 부분은 표시하는 객체들의 세부사항을 알 필요가 없습니다; 그것들이 `Node` 클래스라는 것만 알면 됩니다.

각 `Node` 유형은 자신을 렌더링하고 다른 상호작용을 위한 자체 동작을 정의합니다.
`Node`는 자신에 대한 중요한 데이터를 추적합니다: 이동 행렬, 크기 행렬, 색상 등. 노드의 이동 행렬에
크기 조정 행렬을 곱하면 노드의 모델 좌표 공간에서 월드 좌표 공간으로의 변환 행렬이 나옵니다.
노드는 또한 축 정렬 경계 상자(AABB)를 저장합니다. 아래에서 선택에 대해 논의할 때 AABB에 대해 더 자세히 살펴보겠습니다.

`Node`의 가장 간단한 구체적 구현은 *프리미티브*입니다. 프리미티브는 장면에 추가될 수 있는 단일 고체 모양입니다. 이 프로젝트에서 프리미티브는 `Cube`와 `Sphere`입니다. 

```python
class Node(object):
    """ Base class for scene elements """
    def __init__(self):
        self.color_index = random.randint(color.MIN_COLOR, color.MAX_COLOR)
        self.aabb = AABB([0.0, 0.0, 0.0], [0.5, 0.5, 0.5])
        self.translation_matrix = numpy.identity(4)
        self.scaling_matrix = numpy.identity(4)
        self.selected = False

    def render(self):
        """ renders the item to the screen """
        glPushMatrix()
        glMultMatrixf(numpy.transpose(self.translation_matrix))
        glMultMatrixf(self.scaling_matrix)
        cur_color = color.COLORS[self.color_index]
        glColor3f(cur_color[0], cur_color[1], cur_color[2])
        if self.selected:  # emit light if the node is selected
            glMaterialfv(GL_FRONT, GL_EMISSION, [0.3, 0.3, 0.3])
        
        self.render_self()

        if self.selected:
            glMaterialfv(GL_FRONT, GL_EMISSION, [0.0, 0.0, 0.0])
        glPopMatrix()

    def render_self(self):
        raise NotImplementedError(
            "The Abstract Node Class doesn't define 'render_self'")

class Primitive(Node):
    def __init__(self):
        super(Primitive, self).__init__()
        self.call_list = None

    def render_self(self):
        glCallList(self.call_list)


class Sphere(Primitive):
    """ Sphere primitive """
    def __init__(self):
        super(Sphere, self).__init__()
        self.call_list = G_OBJ_SPHERE


class Cube(Primitive):
    """ Cube primitive """
    def __init__(self):
        super(Cube, self).__init__()
        self.call_list = G_OBJ_CUBE
```

노드 렌더링은 각 노드가 저장하는 변환 행렬에 기반합니다. 노드의 변환 행렬은 크기 조정 행렬과 이동 행렬의 조합입니다. 노드의 유형에 관계없이, 렌더링의 첫 번째 단계는
모델 좌표 공간에서 뷰 좌표 공간으로 변환하는 변환 행렬로 OpenGL ModelView 행렬을 설정하는 것입니다.
OpenGL 행렬들이 최신 상태가 되면, 노드에게 자신을 그리는 데 필요한 OpenGL 호출들을 만들도록 지시하기 위해 `render_self`를 호출합니다. 마지막으로,
이 특정 노드에 대해 만든 OpenGL 상태의 모든 변경사항을 되돌립니다. OpenGL의 `glPushMatrix`와 `glPopMatrix` 함수를 사용하여
노드를 렌더링하기 전후에 ModelView 행렬의 상태를 저장하고 복원합니다.
노드가 색상, 위치, 크기를 저장하고 렌더링 전에 이것들을 OpenGL 상태에 적용한다는 점을 주목하세요.

노드가 현재 선택되어 있으면, 빛을 방출하도록 만듭니다. 이런 식으로 사용자는 어떤 노드를 선택했는지 시각적으로 알 수 있습니다.

프리미티브를 렌더링하기 위해 OpenGL의 호출 목록 기능을 사용합니다.
OpenGL 호출 목록은 한 번 정의되어 단일 이름 아래에 묶인 일련의 OpenGL 호출들입니다.
호출들은 `glCallList(LIST_NAME)`으로 실행될 수 있습니다. 각 프리미티브(`Sphere`와 `Cube`)는 렌더링에 필요한 호출 목록을 정의합니다(표시되지 않음).

예를 들어, 정육면체의 호출 목록은 원점에 중심이 있고 모서리가 정확히 1 단위 길이인 정육면체의 6개 면을 그립니다. \newpage

```python
# Pseudocode Cube definition
# Left face
((-0.5, -0.5, -0.5), (-0.5, -0.5, 0.5), (-0.5, 0.5, 0.5), (-0.5, 0.5, -0.5)),
# Back face
((-0.5, -0.5, -0.5), (-0.5, 0.5, -0.5), (0.5, 0.5, -0.5), (0.5, -0.5, -0.5)),
# Right face
((0.5, -0.5, -0.5), (0.5, 0.5, -0.5), (0.5, 0.5, 0.5), (0.5, -0.5, 0.5)),
# Front face
((-0.5, -0.5, 0.5), (0.5, -0.5, 0.5), (0.5, 0.5, 0.5), (-0.5, 0.5, 0.5)),
# Bottom face
((-0.5, -0.5, 0.5), (-0.5, -0.5, -0.5), (0.5, -0.5, -0.5), (0.5, -0.5, 0.5)),
# Top face
((-0.5, 0.5, -0.5), (-0.5, 0.5, 0.5), (0.5, 0.5, 0.5), (0.5, 0.5, -0.5))
```

프리미티브만 사용하는 것은 모델링 애플리케이션에게 상당히 제한적일 것입니다. 3D 모델은 일반적으로 여러 프리미티브로
구성됩니다(또는 삼각형 메쉬로, 이는 이 프로젝트의 범위를 벗어납니다).
다행히, `Node` 클래스의 설계는 여러 프리미티브로 구성된 `Scene` 노드들을 용이하게 합니다. 사실, 복잡성을 추가하지 않고도
노드들의 임의 그룹화를 지원할 수 있습니다.

동기부여로, 매우 기본적인 도형을 고려해봅시다: 세 개의 구체로 만들어진 전형적인 눈사람, 즉 눈 도형입니다. 이 도형이 세 개의 별개 프리미티브로 구성되어 있어도, 단일 객체로 취급할 수 있기를 원합니다.

다른 노드들을 포함하는 `Node`인 `HierarchicalNode`라는 클래스를 생성합니다. 이것은 "자식들"의 목록을 관리합니다.
계층적 노드들의 `render_self` 함수는 단순히 각 자식 노드에 대해 `render_self`를 호출합니다.
`HierarchicalNode` 클래스로 장면에 도형들을 추가하는 것이 매우 쉬워집니다.
이제 눈 도형을 정의하는 것은 이를 구성하는 모양들과 그들의 상대 위치 및 크기를 지정하는 것만큼 간단합니다.

\aosafigure[240pt]{modeller-images/nodes.jpg}{`Node` 서브클래스의 계층구조}{500l.modeller.hierarchy}


```python
class HierarchicalNode(Node):
    def __init__(self):
        super(HierarchicalNode, self).__init__()
        self.child_nodes = []

    def render_self(self):
        for child in self.child_nodes:
            child.render()
```

\newpage

```python
class SnowFigure(HierarchicalNode):
    def __init__(self):
        super(SnowFigure, self).__init__()
        self.child_nodes = [Sphere(), Sphere(), Sphere()]
        self.child_nodes[0].translate(0, -0.6, 0) # scale 1.0
        self.child_nodes[1].translate(0, 0.1, 0)
        self.child_nodes[1].scaling_matrix = numpy.dot(
            self.scaling_matrix, scaling([0.8, 0.8, 0.8]))
        self.child_nodes[2].translate(0, 0.75, 0)
        self.child_nodes[2].scaling_matrix = numpy.dot(
            self.scaling_matrix, scaling([0.7, 0.7, 0.7]))
        for child_node in self.child_nodes:
            child_node.color_index = color.MIN_COLOR
        self.aabb = AABB([0.0, 0.0, 0.0], [0.5, 1.1, 0.5])
```
`Node` 객체들이 트리 데이터 구조를 형성한다는 점을 관찰할 수 있을 것입니다. `render` 함수는 계층적 노드들을 통해 트리를 깊이 우선 순회합니다.
순회하면서 월드 공간으로의 변환에 사용되는 `ModelView` 행렬들의 스택을 유지합니다.
각 단계에서 현재 `ModelView` 행렬을 스택에 푸시하고, 모든 자식 노드의 렌더링을 완료하면,
행렬을 스택에서 팝하여 부모 노드의 `ModelView` 행렬이 스택의 맨 위에 남도록 합니다.


`Node` 클래스를 이런 식으로 확장 가능하게 만들면, 장면 조작과 렌더링을 위한 다른 코드는 변경하지 않고도
장면에 새로운 유형의 모양들을 추가할 수 있습니다. 하나의 `Scene` 객체가 많은 자식을 가질 수 있다는 사실을 추상화하기 위해 노드 개념을 사용하는 것은 컴포지트 디자인 패턴으로 알려져 있습니다.


### 사용자 상호작용
이제 우리의 모델러가 장면을 저장하고 표시할 수 있으니, 이와 상호작용하는 방법이 필요합니다.
우리가 촉진해야 하는 두 가지 유형의 상호작용이 있습니다.
첫째, 장면의 시각적 관점을 변경하는 능력이 필요합니다. 눈, 즉 카메라를 장면 주위로 움직일 수 있어야 합니다.
둘째, 새로운 노드를 추가하고 장면의 노드들을 수정할 수 있어야 합니다.

사용자 상호작용을 활성화하기 위해서는 사용자가 키를 누르거나 마우스를 움직일 때를 알아야 합니다. 다행히 운영체제는 이러한 이벤트가 언제 발생하는지 이미 알고 있습니다. GLUT은 특정 이벤트가 발생할 때마다 호출될 함수를 등록할 수 있게 해줍니다.
키 누름과 마우스 움직임을 해석하는 함수들을 작성하고, 해당 키들이 눌렸을 때 GLUT이 그러한 함수들을 호출하도록 지시합니다.
사용자가 어떤 키를 누르는지 알게 되면, 입력을 해석하고 의도된 액션을 장면에 적용해야 합니다.

운영체제 이벤트를 듣고 그 의미를 해석하는 로직은 `Interaction` 클래스에서 찾을 수 있습니다.
앞서 작성한 `Viewer` 클래스가 단일 `Interaction` 인스턴스를 소유합니다.
마우스 버튼이 눌렸을 때(`glutMouseFunc`), 마우스가 움직였을 때(`glutMotionFunc`), 키보드 버튼이 눌렸을 때(`glutKeyboardFunc`), 그리고 화살표 키가 눌렸을 때(`glutSpecialFunc`) 호출될 함수들을 등록하기 위해 GLUT 콜백 메커니즘을 사용할 것입니다.
입력 이벤트를 처리하는 함수들을 곧 살펴볼 것입니다.

```python
class Interaction(object):
    def __init__(self):
        """ Handles user interaction """
        # currently pressed mouse button
        self.pressed = None
        # the current location of the camera
        self.translation = [0, 0, 0, 0]
        # the trackball to calculate rotation
        self.trackball = trackball.Trackball(theta = -25, distance=15)
        # the current mouse location
        self.mouse_loc = None
        # Unsophisticated callback mechanism
        self.callbacks = defaultdict(list)
        
        self.register()

    def register(self):
        """ register callbacks with glut """
        glutMouseFunc(self.handle_mouse_button)
        glutMotionFunc(self.handle_mouse_move)
        glutKeyboardFunc(self.handle_keystroke)
        glutSpecialFunc(self.handle_keystroke)

```

#### 운영체제 콜백
사용자 입력을 의미있게 해석하기 위해서는
마우스 위치, 마우스 버튼, 키보드에 대한 지식을 결합해야 합니다. 사용자 입력을 의미있는 액션으로 해석하는 것은 많은 코드 줄을 필요로 하기 때문에, 메인 코드 경로에서 벗어나 별도의 클래스에 캡슐화합니다.
`Interaction` 클래스는 코드베이스의 나머지 부분으로부터 관련 없는 복잡성을 숨기고 운영체제 이벤트를 애플리케이션 수준 이벤트로 변환합니다.

```python
    # class Interaction 
    def translate(self, x, y, z):
        """ translate the camera """
        self.translation[0] += x
        self.translation[1] += y
        self.translation[2] += z

    def handle_mouse_button(self, button, mode, x, y):
        """ Called when the mouse button is pressed or released """
        xSize, ySize = glutGet(GLUT_WINDOW_WIDTH), glutGet(GLUT_WINDOW_HEIGHT)
        y = ySize - y  # invert the y coordinate because OpenGL is inverted
        self.mouse_loc = (x, y)

        if mode == GLUT_DOWN:
            self.pressed = button
            if button == GLUT_RIGHT_BUTTON:
                pass
            elif button == GLUT_LEFT_BUTTON:  # pick
                self.trigger('pick', x, y)
            elif button == 3:  # scroll up
                self.translate(0, 0, 1.0)
            elif button == 4:  # scroll up
                self.translate(0, 0, -1.0)
        else:  # mouse button release
            self.pressed = None
        glutPostRedisplay()

    def handle_mouse_move(self, x, screen_y):
        """ Called when the mouse is moved """
        xSize, ySize = glutGet(GLUT_WINDOW_WIDTH), glutGet(GLUT_WINDOW_HEIGHT)
        y = ySize - screen_y  # invert the y coordinate because OpenGL is inverted
        if self.pressed is not None:
            dx = x - self.mouse_loc[0]
            dy = y - self.mouse_loc[1]
            if self.pressed == GLUT_RIGHT_BUTTON and self.trackball is not None:
                # ignore the updated camera loc because we want to always
                # rotate around the origin
                self.trackball.drag_to(self.mouse_loc[0], self.mouse_loc[1], dx, dy)
            elif self.pressed == GLUT_LEFT_BUTTON:
                self.trigger('move', x, y)
            elif self.pressed == GLUT_MIDDLE_BUTTON:
                self.translate(dx/60.0, dy/60.0, 0)
            else:
                pass
            glutPostRedisplay()
        self.mouse_loc = (x, y)

    def handle_keystroke(self, key, x, screen_y):
        """ Called on keyboard input from the user """
        xSize, ySize = glutGet(GLUT_WINDOW_WIDTH), glutGet(GLUT_WINDOW_HEIGHT)
        y = ySize - screen_y
        if key == 's':
            self.trigger('place', 'sphere', x, y)
        elif key == 'c':
            self.trigger('place', 'cube', x, y)
        elif key == GLUT_KEY_UP:
            self.trigger('scale', up=True)
        elif key == GLUT_KEY_DOWN:
            self.trigger('scale', up=False)
        elif key == GLUT_KEY_LEFT:
            self.trigger('rotate_color', forward=True)
        elif key == GLUT_KEY_RIGHT:
            self.trigger('rotate_color', forward=False)
        glutPostRedisplay()
```

#### 내부 콜백
위의 코드 스니펫에서, `Interaction` 인스턴스가 사용자 액션을 해석할 때 액션 유형을 설명하는 문자열과 함께 `self.trigger`를 호출한다는 점을 알 수 있을 것입니다. `Interaction` 클래스의 `trigger` 함수는 애플리케이션 수준 이벤트 처리를 위해 사용할 간단한 콜백 시스템의 일부입니다.
`Viewer` 클래스의 `init_interaction` 함수가 `register_callback`을 호출하여 `Interaction` 인스턴스에 콜백들을 등록한다는 점을 기억하세요.

```python
    # class Interaction
    def register_callback(self, name, func):
        self.callbacks[name].append(func)
```
사용자 인터페이스 코드가 장면에서 이벤트를 트리거해야 할 때, `Interaction` 클래스는 해당 특정 이벤트에 대해 저장된 모든 콜백들을 호출합니다:

```python
    # class Interaction
    def trigger(self, name, *args, **kwargs):
        for func in self.callbacks[name]:
            func(*args, **kwargs)
```

이 애플리케이션 수준 콜백 시스템은 시스템의 나머지 부분이 운영체제 입력에 대해 알아야 할 필요성을 추상화합니다. 각 애플리케이션 수준 콜백은 애플리케이션 내에서 의미있는 요청을 나타냅니다.
`Interaction` 클래스는 운영체제 이벤트와 애플리케이션 수준 이벤트 사이의 변환기 역할을 합니다.
이는 GLUT 외에 다른 툴킷으로 모델러를 포팅하기로 결정한다면, 새로운 툴킷의 입력을 동일한 의미있는 애플리케이션 수준 콜백 집합으로 변환하는 클래스로 `Interaction` 클래스만 교체하면 된다는 의미입니다. \aosatblref{500l.tbl.callbacks}에서 콜백과 인수들을 사용합니다.

<markdown>
|콜백       | 인수          | 목적  |
|:--------------|:-------------------|:---------|
|`pick`         | x:number, y:number | 마우스 포인터 위치의 노드를 선택합니다. |
|`move`         | x:number, y:number | 현재 선택된 노드를 마우스 포인터 위치로 이동합니다. |
|`place`        | shape:string, x:number, y:number | 지정된 유형의 모양을 마우스 포인터 위치에 배치합니다. |
|`rotate_color` | forward:boolean | 현재 선택된 노드의 색상을 색상 목록을 통해 앞으로 또는 뒤로 회전시킵니다. |
|`scale`        | up:boolean | 매개변수에 따라 현재 선택된 노드를 위 또는 아래로 크기 조정합니다. |

: \label{500l.tbl.callbacks} 상호작용 콜백과 인수들
</markdown>
<latex>
\begin{table}
\centering
{\footnotesize
\rowcolors{2}{TableOdd}{TableEven}
\begin{tabular}{lll}
\hline
\textbf{Callback}
& \textbf{Arguments}
& \textbf{Purpose}
\\
\hline
pick
& x:number, y:number
& 마우스 포인터 위치의 노드를 선택합니다
\\
place &
shape:string, x:number, y:number &
지정된 유형의 모양을 마우스 포인터 위치에 배치합니다.
\\
rotate\_color &
forward:boolean &
현재 선택된 노드의 색상을 회전시킵니다.
\\
scale &
up:boolean &
현재 선택된 노드를 위 또는 아래로 크기 조정합니다.
\\
\hline
\end{tabular}
}
\caption{Interaction callbacks and arguments}
\label{500l.tbl.callbacks}
\end{table}
</latex>

이 간단한 콜백 시스템은 이 프로젝트에 필요한 모든 기능을 제공합니다. 하지만 실제 3D 모델러에서는 사용자 인터페이스 객체들이 동적으로 생성되고 삭제되는 경우가 많습니다.
그런 경우에는 객체들이 이벤트에 대한 콜백을 등록하고 해제할 수 있는 더 정교한 이벤트 리스닝 시스템이 필요할 것입니다.

### 장면과의 인터페이싱
콜백 메커니즘을 통해 `Interaction` 클래스로부터 사용자 입력 이벤트에 대한 의미있는 정보를 받을 수 있습니다. 이제 이러한 액션들을 `Scene`에 적용할 준비가 되었습니다.

#### 장면 이동
이 프로젝트에서는 장면을 변환함으로써 카메라 모션을 달성합니다. 즉,
카메라는 고정된 위치에 있고 사용자 입력이 카메라를 움직이는 대신 장면을 이동시킵니다. 카메라는 `[0, 0, -15]`에 배치되고
월드 공간 원점을 향합니다. (대안적으로, 장면 대신 카메라를 이동시키기 위해 원근 행렬을 변경할 수도 있습니다.
이 설계 결정은 프로젝트의 나머지 부분에 거의 영향을 주지 않습니다.)
`Viewer`의 `render` 함수를 다시 살펴보면, `Scene`을 렌더링하기 전에 OpenGL 행렬 상태를 변환하는 데 `Interaction` 상태가 사용된다는 것을 볼 수 있습니다.
장면과의 상호작용에는 두 가지 유형이 있습니다: 회전과 이동.

#### 트랙볼로 장면 회전하기
*트랙볼* 알고리즘을 사용하여 장면의 회전을 달성합니다. 트랙볼은 3차원에서 장면을 조작하기 위한 직관적인 인터페이스입니다.
개념적으로, 트랙볼 인터페이스는 마치 장면이 투명한 구 안에 있는 것처럼 기능합니다. 구의 표면에 손을 대고 밀면 구가 회전합니다. 마찬가지로, 오른쪽 마우스 버튼을 클릭하고 화면에서 움직이면 장면이 회전합니다.
트랙볼의 이론에 대해 더 알아보려면 [OpenGL Wiki](http://www.opengl.org/wiki/Object_Mouse_Trackball)에서 확인할 수 있습니다.
이 프로젝트에서는 [Glumpy](https://code.google.com/p/glumpy/source/browse/glumpy/trackball.py)의 일부로 제공되는 트랙볼 구현을 사용합니다.

마우스의 현재 위치를 시작 위치로, 마우스 위치의 변화를 매개변수로 하여 `drag_to` 함수를 사용해 트랙볼과 상호작용합니다.

```python
self.trackball.drag_to(self.mouse_loc[0], self.mouse_loc[1], dx, dy)
```

결과 회전 행렬은 장면이 렌더링될 때 뷰어의 `trackball.matrix`입니다.

#### 여담: 쿼터니언
회전은 전통적으로 두 가지 방법 중 하나로 표현됩니다. 첫 번째는 각 축 주변의 회전 값입니다; 이를 부동소수점 숫자의 3-튜플로 저장할 수 있습니다.
회전에 대한 다른 일반적인 표현은 쿼터니언입니다. 이는 $x$, $y$, $z$ 좌표를 가진 벡터와 $w$ 회전으로 구성된 요소입니다. 쿼터니언 사용은 축별 회전에 비해 많은 이점을 가집니다; 특히, 수치적으로 더 안정적입니다. 쿼터니언을 사용하면 김벌 록과 같은 문제를 피할 수 있습니다.
쿼터니언의 단점은 작업하기에 덜 직관적이고 이해하기 어렵다는 것입니다. 용감하게 쿼터니언에 대해 더 알고 싶다면, [이 설명](http://3dgep.com/?p=1815)을 참조할 수 있습니다.

트랙볼 구현은 장면의 회전을 저장하기 위해 내부적으로 쿼터니언을 사용하여 김벌 록을 피합니다. 다행히, 트랙볼의 행렬 멤버가
회전을 행렬로 변환해주기 때문에 쿼터니언을 직접 다룰 필요는 없습니다.

#### 장면 이동
장면을 이동하는 것(즉, 슬라이딩하는 것)은 회전보다 훨씬 간단합니다. 장면 이동은 마우스 휠과 왼쪽 마우스 버튼으로 제공됩니다. 왼쪽 마우스
버튼은 $x$와 $y$ 좌표에서 장면을 이동시킵니다. 마우스 휠을 스크롤하면 z 좌표에서 장면을 이동시킵니다
(카메라 쪽으로 또는 카메라에서 멀어지게). `Interaction` 클래스는 현재 장면 이동을 저장하고 `translate` 함수로 이를 수정합니다.
뷰어는 렌더링 중에 `glTranslated` 호출에서 사용하기 위해 `Interaction` 카메라 위치를 검색합니다.

#### 장면 객체 선택
이제 사용자가 원하는 관점을 얻기 위해 전체 장면을 이동하고 회전할 수 있으니, 다음 단계는 사용자가 장면을 구성하는 객체들을 수정하고 조작할 수 있게 하는 것입니다.

사용자가 장면의 객체들을 조작하기 위해서는 아이템들을 선택할 수 있어야 합니다.

아이템을 선택하기 위해, 마치 마우스 포인터가 장면으로 광선을 쏘는 것처럼 마우스 클릭을 나타내는 광선을 생성하기 위해 현재 프로젝션 행렬을 사용합니다. 선택된 노드는 광선과 교차하는 카메라에 가장 가까운 노드입니다.
따라서 피킹 문제는 광선과 장면의 노드들 사이의 교차점을 찾는 문제로 축소됩니다. 그래서 질문은: 광선이 노드에 맞는지 어떻게 알 수 있을까요?

광선이 노드와 교차하는지 정확히 계산하는 것은 코드의 복잡성과 성능 모두 측면에서 도전적인 문제입니다. 각 프리미티브 유형에 대해 광선-객체 교차 검사를 작성해야 할 것입니다.
많은 면을 가진 복잡한 메쉬 지오메트리를 가진 장면 노드들의 경우, 정확한 광선-객체 교차를 계산하는 것은 각 면에 대해 광선을 테스트해야 하며
계산상 비용이 많이 들 것입니다.

코드를 간결하게 유지하고 성능을 합리적으로 유지하는 목적으로, 광선-객체 교차 테스트를 위해 간단하고 빠른 근사치를 사용합니다.
우리의 구현에서 각 노드는 그것이 차지하는 공간의 근사치인 축 정렬 경계 상자(AABB)를 저장합니다.
광선이 노드와 교차하는지 테스트하기 위해, 광선이 노드의 AABB와 교차하는지 테스트합니다. 이 구현은 모든 노드가
교차 테스트를 위한 동일한 코드를 공유한다는 의미이고, 모든 노드 유형에 대해 성능 비용이 일정하고 작다는 의미입니다.

```python
    # class Viewer
    def get_ray(self, x, y):
        """ 
        Generate a ray beginning at the near plane, in the direction that
        the x, y coordinates are facing 

        Consumes: x, y coordinates of mouse on screen 
        Return: start, direction of the ray 
        """
        self.init_view()
    
        glMatrixMode(GL_MODELVIEW)
        glLoadIdentity()
    
        # get two points on the line.
        start = numpy.array(gluUnProject(x, y, 0.001))
        end = numpy.array(gluUnProject(x, y, 0.999))
    
        # convert those points into a ray
        direction = end - start
        direction = direction / norm(direction)
    
        return (start, direction)
    
    def pick(self, x, y):
        """ Execute pick of an object. Selects an object in the scene. """
        start, direction = self.get_ray(x, y)
        self.scene.pick(start, direction, self.modelView)
```

어떤 노드가 클릭되었는지 결정하기 위해, 광선이 어떤 노드들에 맞는지 테스트하기 위해 장면을 순회합니다. 현재 선택된 노드의 선택을 해제한 다음 광선 원점에 가장 가까운 교차점을 가진 노드를 선택합니다.

```python
    # class Scene
    def pick(self, start, direction, mat):
        """ 
        Execute selection.
            
        start, direction describe a Ray. 
        mat is the inverse of the current modelview matrix for the scene.
        """
        if self.selected_node is not None:
            self.selected_node.select(False)
            self.selected_node = None
    
        # Keep track of the closest hit.
        mindist = sys.maxint
        closest_node = None
        for node in self.node_list:
            hit, distance = node.pick(start, direction, mat)
            if hit and distance < mindist:
                mindist, closest_node = distance, node
    
        # If we hit something, keep track of it.
        if closest_node is not None:
            closest_node.select()
            closest_node.depth = mindist
            closest_node.selected_loc = start + direction * mindist
            self.selected_node = closest_node
```
`Node` 클래스 내에서 `pick` 함수는 광선이 `Node`의 축 정렬 경계 상자와 교차하는지 테스트합니다.
노드가 선택되면, `select` 함수는 노드의 선택된 상태를 토글합니다.
AABB의 `ray_hit` 함수가 상자의 좌표 공간과
광선의 좌표 공간 사이의 변환 행렬을 세 번째 매개변수로 받는다는 점을 주목하세요. 각 노드는 `ray_hit` 함수 호출을 만들기 전에 행렬에 자신의 변환을 적용합니다.

```python
    # class Node
    def pick(self, start, direction, mat):
        """ 
        Return whether or not the ray hits the object

        Consume:  
        start, direction form the ray to check
        mat is the modelview matrix to transform the ray by 
        """

        # transform the modelview matrix by the current translation
        newmat = numpy.dot(
            numpy.dot(mat, self.translation_matrix), 
            numpy.linalg.inv(self.scaling_matrix)
        )
        results = self.aabb.ray_hit(start, direction, newmat)
        return results

    def select(self, select=None):
       """ Toggles or sets selected state """
       if select is not None:
           self.selected = select
       else:
           self.selected = not self.selected
    
```

광선-AABB 선택 접근법은 이해하고 구현하기 매우 간단합니다. 하지만 특정 상황에서는 결과가 틀릴 수 있습니다.

\aosafigure[240pt]{modeller-images/AABBError.png}{AABB 오류}{500l.modeller.aabberror}

예를 들어, `Sphere` 프리미티브의 경우, 구체 자체는 AABB의 각 면의 중앙에서만 AABB에 닿습니다.
하지만 사용자가 Sphere의 AABB 모서리를 클릭하면, 사용자가 Sphere를 지나서 뒤에 있는 무언가를 클릭하려고 했더라도
Sphere와의 충돌이 감지될 것입니다 (\aosafigref{500l.modeller.aabberror}).

복잡성, 성능, 정확성 사이의 이러한 트레이드오프는 컴퓨터 그래픽스와 소프트웨어 엔지니어링의 많은 영역에서 흔합니다.

#### 장면 객체 수정
다음으로, 사용자가 선택된 노드들을 조작할 수 있게 하고 싶습니다. 선택된 노드를 이동하거나, 크기를 조정하거나, 색상을 변경하고 싶을 수 있습니다.
사용자가 노드를 조작하는 명령을 입력하면, `Interaction` 클래스는 입력을 사용자가 의도한 액션으로 변환하고 해당 콜백을 호출합니다.

`Viewer`가 이러한 이벤트 중 하나에 대한 콜백을 받으면, `Scene`의 적절한 함수를 호출하고, 이는 다시 현재 선택된 `Node`에 변환을 적용합니다.

```python
    # class Viewer
    def move(self, x, y):
        """ Execute a move command on the scene. """
        start, direction = self.get_ray(x, y)
        self.scene.move_selected(start, direction, self.inverseModelView)

    def rotate_color(self, forward):
        """
        Rotate the color of the selected Node.
        Boolean 'forward' indicates direction of rotation.
        """
        self.scene.rotate_selected_color(forward)

    def scale(self, up):
        """ Scale the selected Node. Boolean up indicates scaling larger."""
        self.scene.scale_selected(up)
```

#### 색상 변경
색상 조작은 가능한 색상들의 목록으로 수행됩니다. 사용자는 화살표 키로 목록을 순환할 수 있습니다. 장면은 색상 변경 명령을
현재 선택된 노드에 전달합니다.

```python
    # class Scene
    def rotate_selected_color(self, forwards):
        """ Rotate the color of the currently selected node """
        if self.selected_node is None: return
        self.selected_node.rotate_color(forwards)
```

각 노드는 현재 색상을 저장합니다. `rotate_color` 함수는 단순히 노드의 현재 색상을 수정합니다. 노드가 렌더링될 때 색상은 `glColor`로 OpenGL에 전달됩니다.

```python
    # class Node
    def rotate_color(self, forwards):
        self.color_index += 1 if forwards else -1
        if self.color_index > color.MAX_COLOR:
            self.color_index = color.MIN_COLOR
        if self.color_index < color.MIN_COLOR:
            self.color_index = color.MAX_COLOR
```

#### 노드 크기 조정
색상과 마찬가지로, 장면은 선택된 노드가 있다면 모든 크기 조정 수정사항을 그 노드에 전달합니다.

```python
    # class Scene
    def scale_selected(self, up):
        """ Scale the current selection """
        if self.selected_node is None: return
        self.selected_node.scale(up)

```

각 노드는 크기를 저장하는 현재 행렬을 저장합니다. 각각의 방향에서 매개변수 $x$, $y$, $z$로 크기를 조정하는 행렬은 다음과 같습니다:

<latex>
$$
   \begin{bmatrix}
   x & 0 & 0 & 0 \\
   0 & y & 0 & 0 \\
   0 & 0 & z & 0 \\
   0 & 0 & 0 & 1 \\
   \end{bmatrix}
$$
</latex>
<markdown>
$$
    \begin{bmatrix}
    x & 0 & 0 & 0 \\
    0 & y & 0 & 0 \\
    0 & 0 & z & 0 \\
    0 & 0 & 0 & 1 \\ 
    \end{bmatrix}
$$
</markdown>

사용자가 노드의 크기를 수정하면, 결과 크기 조정 행렬이 노드의 현재 크기 조정 행렬로 곱해집니다.

```python
    # class Node
    def scale(self, up):
        s =  1.1 if up else 0.9
        self.scaling_matrix = numpy.dot(self.scaling_matrix, scaling([s, s, s]))
        self.aabb.scale(s)
```

`scaling` 함수는 $x$, $y$, $z$ 크기 조정 인수들의 목록이 주어지면 그러한 행렬을 반환합니다.

```python
def scaling(scale):
    s = numpy.identity(4)
    s[0, 0] = scale[0]
    s[1, 1] = scale[1]
    s[2, 2] = scale[2]
    s[3, 3] = 1
    return s
```

#### 노드 이동
노드를 이동하기 위해, 피킹에 사용했던 것과 동일한 광선 계산을 사용합니다. 현재 마우스 위치를 나타내는 광선을 장면의
`move` 함수에 전달합니다. 노드의 새 위치는 광선 위에 있어야 합니다.
광선에서 노드를 배치할 위치를 결정하기 위해서는 카메라로부터의 노드 거리를 알아야 합니다. 노드가 선택되었을 때(`pick` 함수에서) 카메라로부터의 노드 위치와 거리를 저장했으므로, 여기서 그 데이터를 사용할 수 있습니다.
대상 광선을 따라 카메라로부터 같은 거리에 있는 점을 찾고 새 위치와 이전 위치 사이의 벡터 차이를 계산합니다.
그런 다음 결과 벡터로 노드를 이동합니다. 

```python
    # class Scene
    def move_selected(self, start, direction, inv_modelview):
        """ 
        Move the selected node, if there is one.
            
        Consume: 
        start, direction describes the Ray to move to
        mat is the modelview matrix for the scene 
        """
        if self.selected_node is None: return
    
        # Find the current depth and location of the selected node
        node = self.selected_node
        depth = node.depth
        oldloc = node.selected_loc
    
        # The new location of the node is the same depth along the new ray
        newloc = (start + direction * depth)
    
        # transform the translation with the modelview matrix
        translation = newloc - oldloc
        pre_tran = numpy.array([translation[0], translation[1], translation[2], 0])
        translation = inv_modelview.dot(pre_tran)
    
        # translate the node and track its location
        node.translate(translation[0], translation[1], translation[2])
        node.selected_loc = newloc
```

새 위치와 이전 위치는 카메라 좌표 공간에서 정의된다는 점을 주목하세요. 우리의 이동은 월드 좌표 공간에서 정의되어야 합니다.
따라서 모델뷰 행렬의 역행렬을 곱하여 카메라 공간 이동을 월드 공간 이동으로 변환합니다.

크기 조정과 마찬가지로, 각 노드는 이동을 나타내는 행렬을 저장합니다. 이동 행렬은 다음과 같습니다:

$$
   \begin{bmatrix}
   1 & 0 & 0 & x \\
   0 & 1 & 0 & y \\
   0 & 0 & 1 & z \\
   0 & 0 & 0 & 1 \\
   \end{bmatrix}
$$

노드가 이동될 때, 현재 이동을 위한 새로운 이동 행렬을 구성하고, 렌더링 중에 사용하기 위해 노드의
이동 행렬에 곱합니다.

```python
    # class Node
    def translate(self, x, y, z):
        self.translation_matrix = numpy.dot(
            self.translation_matrix,
            translation([x, y, z]))
```

`translation` 함수는 $x$, $y$, $z$ 이동 거리를 나타내는 목록이 주어지면 이동 행렬을 반환합니다.

```python
def translation(displacement):
    t = numpy.identity(4)
    t[0, 3] = displacement[0]
    t[1, 3] = displacement[1]
    t[2, 3] = displacement[2]
    return t
```

#### 노드 배치
노드 배치는 피킹과 이동 모두의 기법을 사용합니다. 노드를 배치할 위치를 결정하기 위해 현재 마우스 위치에 대한 동일한 광선 계산을 사용합니다.

```python
    # class Viewer
    def place(self, shape, x, y):
        """ Execute a placement of a new primitive into the scene. """
        start, direction = self.get_ray(x, y)
        self.scene.place(shape, start, direction, self.inverseModelView)
```

새 노드를 배치하기 위해, 먼저 해당 노드 유형의 새 인스턴스를 생성하고 장면에 추가합니다.
사용자의 커서 아래에 노드를 배치하고 싶으므로, 카메라로부터 고정된 거리에 있는 광선 위의 점을 찾습니다.
다시, 광선은 카메라 공간에서 표현되므로, 결과 이동 벡터를 역 모델뷰 행렬로 곱하여 월드 좌표 공간으로 변환합니다.
마지막으로, 계산된 벡터로 새 노드를 이동합니다. \newpage

```python
    # class Scene
    def place(self, shape, start, direction, inv_modelview):
        """ 
        Place a new node.
            
        Consume:  
        shape the shape to add
        start, direction describes the Ray to move to
        inv_modelview is the inverse modelview matrix for the scene 
        """
        new_node = None
        if shape == 'sphere': new_node = Sphere()
        elif shape == 'cube': new_node = Cube()
        elif shape == 'figure': new_node = SnowFigure()
    
        self.add_node(new_node)
    
        # place the node at the cursor in camera-space
        translation = (start + direction * self.PLACE_DEPTH)
    
        # convert the translation to world-space
        pre_tran = numpy.array([translation[0], translation[1], translation[2], 1])
        translation = inv_modelview.dot(pre_tran)
    
        new_node.translate(translation[0], translation[1], translation[2])
```

## 요약
축하합니다! 작은 3D 모델러를 성공적으로 구현했습니다!

\aosafigure[240pt]{modeller-images/StartScene.png}{샘플 장면}{500l.modeller.samplescene}

장면의 객체들을 나타내는 확장 가능한 데이터 구조를 개발하는 방법을 살펴봤습니다. 컴포지트 디자인 패턴과 트리 기반
데이터 구조를 사용하면 렌더링을 위해 장면을 순회하기 쉽게 만들고 복잡성을 추가하지 않고도
새로운 유형의 노드를 추가할 수 있게 해준다는 것을 알았습니다. 이 데이터
구조를 활용하여 설계를 화면에 렌더링하고, 장면 그래프의 순회에서 OpenGL 행렬을 조작했습니다. 애플리케이션 수준 이벤트를 위한 매우 간단한 콜백 시스템을 구축하고, 운영
체제 이벤트 처리를 캡슐화하는 데 사용했습니다. 광선-객체 충돌
감지의 가능한 구현들과 정확성, 복잡성, 성능 사이의 트레이드오프에 대해 논의했습니다.
마지막으로, 장면의 내용을 조작하는 방법들을 구현했습니다.

실제 3D 소프트웨어에서 이와 동일한 기본 구성 요소들을 찾을 수 있을 것으로 기대할 수 있습니다. 장면 그래프 구조와 상대적 좌표 공간은
CAD 도구부터 게임 엔진까지 많은 유형의 3D 그래픽 애플리케이션에서 발견됩니다.
이 프로젝트의 한 가지 주요 단순화는 사용자 인터페이스에 있습니다. 실제 3D 모델러는
완전한 사용자 인터페이스를 가져야 하며, 이는 우리의 간단한 콜백 시스템 대신 훨씬 더 정교한 이벤트 시스템을 필요로 할 것입니다.

이 프로젝트에 새로운 기능을 추가하기 위한 추가 실험을 할 수 있습니다. 다음 중 하나를 시도해보세요:

* 임의 모양을 위한 삼각형 메쉬를 지원하는 `Node` 유형을 추가하기.
* 모델러 액션의 실행 취소/다시 실행을 허용하는 실행 취소 스택을 추가하기.
* DXF와 같은 3D 파일 형식을 사용하여 설계를 저장/불러오기.
* 렌더링 엔진 통합: 사실적 렌더러에서 사용하기 위해 설계를 내보내기.
* 정확한 광선-객체 교차로 충돌 감지를 개선하기.

## 추가 탐구
실제 3D 모델링 소프트웨어에 대한 추가 통찰을 위해, 몇 가지 오픈 소스 프로젝트들이 흥미롭습니다.

[Blender](http://www.blender.org/)는 오픈 소스 완전 기능 3D 애니메이션 제품군입니다. 비디오용 특수 효과 구축이나 게임 제작을 위한 완전한 3D 파이프라인을 제공합니다. 모델러는 이
프로젝트의 작은 부분이며, 모델러를 대형 소프트웨어 제품군에 통합하는 좋은 예입니다.

[OpenSCAD](http://www.openscad.org/)는 오픈 소스 3D 모델링 도구입니다. 상호작용적이지 않습니다; 대신, 장면을 생성하는 방법을 지정하는 스크립트 파일을 읽습니다. 이는 설계자에게 "모델링 과정에 대한 완전한 제어"를 제공합니다.

컴퓨터 그래픽스의 알고리즘과 기법에 대한 더 많은 정보를 위해서는 [Graphics Gems](http://tog.acm.org/resources/GraphicsGems/)가 훌륭한 자료입니다.
