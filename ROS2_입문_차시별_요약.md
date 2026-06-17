# 01차시: ROS2 프로그래밍 기초
로봇SW(ROS2): OS층 → ROS 미들웨어층(RMW):rmw, Fast DDS, DDS 등 → ROS 클라이언트층(RCL): rclcpp, rclpy, API들 → 유저 코드:유저 노드들
토픽: 비동기 단방향 통신☆☆
서비스: 동기식 양방향 메시지 송수신☆☆, 서비스 서버 살아있어야함
액션: 동기식, 비동기식 양방향 메시지 송수신☆☆, 액션 서버, 3개 서비스와 2개 토픽, 토픽은 액션 피드백과 state(상태) 토픽 2개, 서비스가 액션 목표(골)와 액션 결과 요청하는 서비스, 목표 캔슬 전달하는 서비스까지 3개
Goal State Machine: 액션 서버가 상태 정보 전이시켜 클라이언트가 알게해주는 역할

### ROS2에서의 프로그래밍 역할
- ROS2에서 "프로그래밍"의 역할은 로봇 시스템을 설계하고 제어하는 데 중요
- ROS2는 로봇 애플리케이션을 개발하는 데 사용되는 오픈 소스 소프트웨어 프레임워크로, 복잡한 로봇 작업을 분리된 모듈로 나누어 관리할 수 있게 해주며, 이를 통해 다양한 로봇 하드웨어와 소프트웨어 간의 상호작용을 쉽게 할 수 있음
1. 노드 개발
- ROS2는 로봇 시스템을 여러 개의 독립적인 노드로 나누어 작업
- 각 노드는 특정 작업을 수행하며, 프로그래밍을 통해 이러한 노드를 작성하고, 다른 노드와 데이터를 교환하거나 상호작용하도록 만들 수 있음
- 예를 들어, 센서 데이터를 처리하는 노드, 로봇의 모터를 제어하는 노드 등 개발 가능
2. 토픽(Topic)과 메시지(Message) 관리
- ROS2에서는 노드 간의 통신이 토픽을 통해 이루어짐
- 각 노드는 특정 토픽을 구독하거나 발행하여 데이터 송수신
- 프로그래밍을 통해 토픽과 메시지 형식을 정의하고, 이를 통해 로봇 시스템의 다양한 부품들이 정보를 교환하도록 함
3. 서비스(Service)와 액션(Action)
- ROS2는 비동기 작업을 처리하는 서비스를 제공
- 프로그래밍을 통해 특정 요청에 대해 서버와 클라이언트 간의 요청-응답 방식으로 통신 인터페이스 개발 가능
- 또한, 액션을 통해 긴 시간이 걸리는 작업을 비동기적으로 처리 가능
- 예를 들어, 로봇이 경로를 따라가거나 특정 작업을 완료하는 데 시간이 걸릴 때, 이를 처리하기 위한 프로그래밍이 필요
4. 로봇 하드웨어 제어
- ROS2는 로봇 하드웨어를 제어하는 다양한 라이브러리와 드라이버를 제공
- 프로그래밍을 통해 로봇의 센서, 카메라, 모터, 로봇 팔 등을 제어하는 코드를 작성 가능
- 예를 들어, 로봇 팔을 제어하기 위한 역기구학(Inverse Kinematics) 알고리즘을 구현하거나, LiDAR 센서 데이터를 처리하는 프로그램을 작성
5. 로봇 동작의 논리 구현
- ROS2는 복잡한 로봇 동작을 관리할 수 있는 강력한 도구들을 제공
- 이를 통해 로봇의 동작을 제어하는 논리를 작성 가능
- 예를 들어, 로봇이 장애물을 회피하면서 목적지로 가는 경로를 계획하는 알고리즘을 프로그래밍하여 개발
6. 시뮬레이션(Simulation)
- ROS2는 Gazebo와 같은 시뮬레이션 도구와 함께 사용되어 실제 하드웨어 없이 로봇을 테스트하고 
개발
- 프로그래밍을 통해 시뮬레이션 환경에서 로봇의 동작을 제어하고, 실제 환경에서의 동작을 예측
하며, 로봇의 성능을 평가
7. 분산 시스템 구현
- ROS2는 분산 시스템을 기반으로 설계되어 있으며, 이를 통해 여러 컴퓨터가 네트워크를 통해 연결되어 작업을 분담
- 프로그래밍을 통해 여러 장치에서 실행되는 ROS2 노드 간의 통신을 설정하고, 분산 환경에서 로봇 시스템을 조정하는 작업 수행

결론: 효율적인 로봇 개발
- 로봇 하드웨어와 소프트웨어를 연결하는 데 필요한 공통 인터페이스 제공
- 노드라는 모듈화된 구조를 가지며 각각의 노드는 독립적으로 개발, 테스트 및 배포할 수 있어 대규모 시스템에서도 협업이 용이
- ROS2는 전 세계적인 커뮤니티 지원과 방대한 오픈소스 패키지를 제공
- ROS2는 센서, 액추에이터, 카메라 등 다양한 로봇 하드웨어와 호환됨
- Gazebo나 rviz와 같은 시뮬레이터와 통합하여 실제 하드웨어 없이도 복잡한 알고리즘을 테스트하고 디버깅 가능
- Python, C++, MATLAB 등 다양한 언어를 지원하여 개발자가 선호하는 언어를 사용할 수 있음

Underlay
- ROS 환경에서 사용 가능한 기본 ROS 설치를 말함
(일반적으로 ROS Release로 설치되는 ROS 코어 및 표준 라이브러리와 도구를 포함)
- Underlay는 시스템에 설치되며, ‘opt/ros/<ROS_DISTRO>’ 디렉토리에 위치
- ROS Underlay는 기본 라이브러리, 도구, 메시지 및 서비스 정의 +기본 ROS 기능을 제공

Overlay
- User가 설치하거나 개발 중인 패키지를 포함하는 사용자 지정 ROS 작업 공간(사용자는 ROS 자체 패키지 및 노드를 추가, 확장)
- 사용자 홈 디렉토리나 사용자가 지정한 다른 디렉토리에 위치할 수 있음
- ROS패키지 개별적 빌드 및 사용자 작업 공간에 설치 가능(사용자 지정 노드 및 라이브러리를 ROS 환경에 추가 가능)
- 사용자 지정 패키지와 노드를 제공
- Overlay 개발 환경은 설치된 ROS 패키지들에 의존하기에 Underlay 개발 환경에 종속적 
- 설정 스크립트(Setup script)라고 하는 setup.bash의 호출 순서 및 사용방법이 조금씩 달라짐

### Setup.bash vs local_setup.bash
Setup.bash와 local_setup.bash는 설정 스크립트(Setup script)라고 하며, Underlay와 Overlay 구분 없이 모든 워크스페이스에 존재하며 각 설정 스크립트마다 사용 목적은 조금씩 다름
local_setup.bash
- 현재 작업 중인 ROS의 워크스페이스의 환경 설정에 사용
- 직접적으로 설치된 경로가 아니므로 User가 직접 작성하거나 다운로드한 ROS패키지와 노드의 환경 변수를 설정하여 해당 워크스페이스에 사용자 지정 패키지 및 노드를 실행하는데 필요
- ‘install/local_setup.bash’는 사용자의 워크스페이스 디렉토리 내에 위치해야 하며, 해당 디렉토리에는 build, install 디렉토리가 있어야 함
setup.bash
- ‘/opt/ros/humble’에 설치된 ROS의 릴리스 환경을 설정하는데 사용
- 현재 터미널 세션에 ROS의 패키지, 라이브러리의 환경 변수를 추가하여 해당 ROS 릴리스의 도구 및 패키지를 사용할 수 있게 함

colcon_cd
- 터미널 창에서 ‘colcon_cd’ 명령을 사용하면, 셸(shell)의 현재 워크스페이스를 패키지 디렉터리로 빠르게 변경 가능
- ROS1의 ros_cd와 비슷한 기능
- ./.bashrc에 아래와 같이 추가

ROS_LOCALHOST_ONLY
- DDS 기반 RMW 구현의 기본 동작은 멀티캐스트를 통해 도달 가능한 모든 노드를 검색하도록 되어 있음
- 이는 같은 네트워크를 사용할 때 의도치 않게 같은 네트워크내의 다른 개발자와 DDS로 연결됨에 따라 불편함 초래
- ROS_LOCALHOST_ONLY 환경 변수를 True 설정하는 것으로 단일 컴퓨터 안에서만 DDS 사용 가능
- ROS2 Humble의 다음 버전인 ROS2 Iron 부터는 ROS_LOCALHOST_ONLY를 더 이상 사용하지 않음 → 보다 세분화된 옵션을 사용할 수 있는 ROS-AUTOMATIC-DISCOVERY-RANGE로 변경

패키지(Package) 구성요소
- 노드(Node): 특정 작업(로봇 제어, 센서 데이터 처리, 토픽 발행/구독 등)을 수행하는 실행 파일
- 런치 파일(Launch file): 여러 노드를 실행하고, 그들의 매개변수, 토픽 및 서비스를 구성하는 Python 파일
- 설정 파일(Configuration file): YAML 파일로 노드나 노드 그룹의 매개변수, 토픽, 서비스 등을 정의
- 라이브러리(Library): C++ 또는 Python 라이브러리로, 메시지 정의, 알고리즘, 드라이버 등 재사용 가능한 기능을 제공
- 자원(Resource): 이미지, 소리, 모델 등의 데이터 파일로 노드나 시각화 도구에서 사용
- 테스트(Test): 유닛 테스트, 통합 테스트, 시스템 테스트 등으로 패키지의 정확성과 견고성을 검증
- 문서(Documentation): README 파일, 튜토리얼, API 참조 등으로 사용자가 패키지를 이해하고 사용할 수 있도록 도움

package.xml
- 패키지에 대한 메타 정보를 포함하는 파일 (패키지의 신분증 역할)
- 이 파일은 패키지 이름, 버전, 저작자, 라이센스 등의 정보를 정의하며, 패키지의 의존성 패키지와 메시지, 서비스, 액션 등의 정의된 인터페이스 정보도 포함

사용목적
- 소스 코드를 실제 실행 가능한 프로그램이나 라이브러리로 변환하기 위해 colcon build를 수행하면 package.xml을 참조하여, 빌드할 패키지들 사이의 의존성 해석 및 적절한 빌드 순서 결정
- 또한, 패키지 의존성 설치 시 rosdep이 이 파일의 정보를 기반으로 함

### setup.py & setup.cfg
setup.cfg
- 패키지 빌드/설치/배포에 사용(버전/설명/패키지 의존성 관리 등), setuptools를 사용하여 패키지를 배포 준비 시 필요한 정보 제공, 주로 패키지 버전, 설명 등이 여기에 정의됨
- Python 패키지에 대한 선언적인 구성 정보를 제공하며, setuptools의 빌드 및 설치 과정에서 활용

setup.py
- setuptools를 사용하여 패키지를 배포할 준비를 할 때, 필요한 정보를 담고 있음
- Python 패키지에 대한 프로그래매틱한 구성 정보를 제공하며, setuptools를 통한 빌드 및 설치 과정에서 사용됨
- Python 패키지의 설치 스크립트, 주로 패키지 버전, 설명, 의존성 등을 포함한 설치 스크립트 역할
- setuptools 라이브러리를 사용, 패키지를 빌드하고 설치하는데 필요한 설정 포함

ROS2에서의 역할
- Python 기반의 ROS2 패키지에 대해 colcon은 setup.cfg(및 setup.py)를 사용하여 패키지의 설치를 처리
- setup.cfg, setup.py는 setuptools를 통해 패키지를 빌드하고 설치하는 방법에 대한 구
성 정보를 제공
- ament_python 패키지 빌드 타입을 사용하는 경우, 파일의 설정이 빌드 과정에 영향을 줄 수 있음

CMakeLists.txt
- 패키지 내 코드를 빌드하는 방법을 기술하는 파일, 이 파일은 빌드 프로세스를 자동화하기 위한 CMake빌드 시스템에 의해 사용
- 빌드에 필요한 컴파일러, 라이브러리, 소스 파일 등을 명시하고, 실행파일, 라이브러리, 메시지, 서비스 등의 빌드 대상 및 의존성 관리를 설정

주요 역할
- ROS2에서 CMakeLists.txt 파일은 패키지의 빌드 규칙을 정의하는 역할
- ROS2는 CMake빌드 시스템을 사용하여 패키지를 빌드하며, CMakeLists.txt는 CMake가 각 패키지를 어떻게 컴파일하고 링크할지 지시하는 지침을 포함
- 패키지에 필요한 최소 CMake버전 지정, 프로젝트 이름과 버전을 설정, 빌드해야 할 타겟(executables, libraries) 정의, 필요한 종속성 패키지를 찾고 링크, 특정 빌드 옵션을 설정하거나 사용자 정의 빌드 규칙 

### CMakeLists.txt와 setup.py/setup.cfg, package.xml간의 비교
CMakeLists.txt: C/C++ 프로젝트에서 주로 사용됨, 코드 컴파일 및 링크 설정, ROS2 메시지 및 서비스 생성 같은 더 광범위한 작업 지원, 빌드 시 필요한 지침을 담고 있으며, 주로 코드 컴파일과 관련이 깊음
setup.py/setup.cfg: Python 패키지의 빌드 및 설치 과정 설정에 사용됨, 주로 Python 관련 설정에 집중
Pacakge.xml: 패키지의 메타데이터와 의존성을 관리하는데 중점, CMakeLists.txt와 함께 작동하여 ROS2 패키지의 빌드와 배포를 가능하게 함
세 파일의 공통점
모두 패키지의 빌드 및 설치 과정에서 의존성 관리와 설정 정의에 사용

pip install .과 python3 setup.py install의 차이
#### pip install
- 의존성 해결: pip는 setup.py(setup.cfg) 또는 pyproject.toml에 명시된 패키지의 의존성을 자동으로 해결하고 설치.
- 가상 환경 친화적: pip는 현재 활성화된 Python 가상 환경에 패키지를 설치하여 시스템 전체 
Python 설치를 변경하지 않고 패키지를 안전하게 설치할 수 있게 해줌
python3 setup.py install
- 의존성 해결 부족: 패키지 의존성을 자동으로 해결하지 못하므로 수동으로 미리 설치해야 함
- 가상 환경과의 호환성 낮음: 이 명령을 사용할 때도 가상 환경에 설치할 수 있지만, pip만큼 가상 환경과의 통합이 자연스럽지는 않음

setup.py & setup.cfg차이점
||setup.py|setup.cfg|
|:---|:---|:---|
접근 방식|   프로그래밍 방식으로 패키지 설정을 제공   |   선언적 방식으로 설정을 제공
유연성|동적 계산과 사용자 정의 명령어 지원하는 높은 유연성 제공 | 보다 간단하고 명확한 패키지 설정 지향
사용 추세|setup.py의 경우 패키지 의존성을 해결해주지 않으므로<br> 가능한만큼 setup.py 파일은 최소화하거나 제거|현대의 Python 패키징은 setup.cfg를 통한 선언적 패키지 설정을 선호

패키지 파일 사용 방법 ROS 2 프로그래밍 전 알아둬야 할 필수 사전 정보
- 패키지 설정 파일: package.xml(ROS pkg의 필수 구성요소)
- 빌드 설정 파일: CMakeLists.txt(순수 Python pkg는 CMakeLists.txt 파일 없음)
- 파이썬 패키지 설정 파일: setup.py(순수한 ROS2 Python pkg에만 사용하는 배포를 위한 설정 파일)
- 파이썬 패키지 환경 설정 파일: setup.cfg(순수 ROS2 Python pkg에만 사용하는 배포 위한 구성 파일)
- RQt 플러그인 설정 파일: plugin.xml(RQT plugin으로 pkg를 작성할때의 필수 구성요소)
- 패키지 변경로그 파일: CHANGELOG.rst(pkg업데이트 내역 기술, 개발 이력 추적)
- 라이선스 파일: LICENSE(pkg코드에 사용된 라이센스 기술)
- 패키지 설명 파일: README.md(markdown 파일)



# 02장: 인터페이스 패키지
### 인터페이스(Interface)
- ROS에서 노드 사이에 데이터를 전송 시 사용되는 토픽(Topic), 서비스(Service), 액션(Action)에서 사용되는 데이터 타입
- 토픽은 msg파일, 서비스는 srv 파일, 액션은 action 파일에 인터페이스 정의
- 일반적으로 std_msgs나 geometry_msgs와 같은 미리 선언된 인터페이스 바로 사용 가능, 하나, 필요에 따라 커스텀 인터페이스 생성 가능
- 단일 패키지를 가진 프로그램에서 사용 시, 해당 패키지 포함시키기도 하지만, 일반적으로 단일 패키지를 가지는 프로그램을 만드는 경우는 거의 없음
- 여러 개의 패키지를 가지는 경우, 별도의 인터페이스 패키지를 생성하여 사용하는 것을 추천 (이 경우 여러 패키지들이 만들어진 인터페이스 패키지를 공유하며 사용 가능)

### ROS2 인터페이스(Interface) 3종류
|종류|설명|예시|
|:---|:---|:---|
|msg (message)|Topic 통신에서 사용되는 데이터 형식|std_msgs/msg/String, sensor_msgs/msg/LaserScan|
|srv (service)|요청/응답 통신용 데이터 형식|example_interfaces/srv/AddTwoInts|
|action|장시간 작업 요청용 데이터 형식 (goal, feedback, result)|nav2_msgs/action/NavigateToPose|

### ROS2 인터페이스(Interface) 비교 요약
|항목|.msg|.srv|.action|
|:---|:---|:---|:---|
|목적|데이터 전송 (Topic)|요청/응답 (Service)|장기 작업 처리 (Action)|
|구조|필드 목록|요청---응답|목표---결과---피드백|
|사용 예| 센서 데이터|덧셈 요청, 파일 열기|경로 따라가기, 위치 이동 등|
|통신 방식|단방향|동기 (요청 후 응답 대기)|비동기 (피드백 + 취소 지원)|

### .msg (Topic 에서 사용되는 메시지)
- Topic 통신에 사용되는 데이터 구조, 단방향 통신 (요청 없이 데이터 전송), .msg 파일로 정의, 기본값, 상수값 정의 가능
- 메시지 구조
```
<필드타입> <필드이름>   예시: std_msgs/msg/String.msg 
<필드타입> <필드이름>   string data
```
- 아주 간단한 메시지, string 타입의 data 필드 하나만 있음, Publisher가 data="hello" 보내면, Subscriber는 문자열 그대로 받음

### .srv(Service 에서 사용되는 메시지)
- 요청/응답 방식 통신, .srv 파일은 요청 → 응답 메시지를 정의, 짧고 즉시 응답 가능한 작업에 적합
--- 구분자로 섹션을 구분
- 메시지 구조
```
#REQUEST                예시: AddTwoInts.srv
<필드타입> <필드이름>      int64 a
---                      int64 b
#RESPONSE                ---
<필드타입> <필드이름>      int64 sum
```
### .action (Action 에서 사용)
- 장시간 실행되는 작업을 위한 통신
- Goal(목표) → 중간, Feedback → Result(결과)
- 취소 가능 / 상태 추적 가능
- 메시지 구조
```
#Goal                     예시: Fibonacci.action
<필드타입> <목표필드>         int32 order
---                          ---
#Result                     int32[] sequence
<필드타입> <결과필드>          ---
---                          int32[] sequence
#Feedback
<필드타입> <피드백필드>
```
상수값과 기본값의 정의요약표
|항목|상수(Constant)|기본값(Default value)|
|:---|:---|:---|
|선언 형식|타입 NAME=값|타입 필드이름 기본값|
|변경 가능|불가(컴파일타임 고정)|가능 (런타임 수정 가능)|
|사용 위치|msg, srv, action 모두|msg만 완전 지원|
|사용 목적|코드 내 고정값 표현|필드 값 초기화 용도|
|예시|int8 OK=1|string name "Robot"|



# 03장: 토픽: 서비스 패키지
Topic 패키지 생성 -실습
- ​Python으로 publisher와 subscriber node를 생성하고 실행하기
→ Topic상으로 message를 전송/수신하는 역할 (talker/listener)

Package.xml 설정
- test_depend
- <test_depend> 태그는 빌드 및 실행 과정 중이 아닌 테스트 단계에서만 필요한 종속성을 지정
- 이는 패키지 개발 시 테스트 자동화를 위한 환경 구성에 필수적
- ament_copyright
- 이 도구는 소스 코드 파일 내에 적절한 저작권 고지 및 라이센스 헤더가 포함되어 있는지 검사
- ROS2 개발에서는 모든 소스 파일이 올바른 저작권 정보를 포함하도록 권장
- ament_flake8
- ament_flake8는 Python 코드의 스타일을 검사하는 도구
- 이는 PEP 8—Python 스타일 가이드를 준수하는지 확인하여 코드의 일관성과 가독성을 높이는 데 도움을 줌
- ament_pep257
- ament_pep257은 Python 코드 내의 docstrings이 PEP 257—docstring 규칙을 따르는 지 검사
- 좋은 문서화 관행을 유지하고 코드의 유지보수성을 높이는 데 중요한 도구
- python3-pytest
- python3-pytest는 Python 코드를 위한 강력한 테스팅 프레임워크
- 이 종속성은 테스트를 정의하고 실행하는 데 필요하며, 다양한 테스트 케이스를 쉽게 작성하고 실행할 수 있게 해줌
- pytest는 테스트의 설정, 실행, 검증 및 리포팅 기능을 제공

setup.py 설정
- entry_points
- entry_points는 Python의 setuptools에서 사용되는 설정의 일부
- 특히 python 패키지를 설치할 때 커맨드 라인 스크립트를 자동으로 성하도록 지시하는 데 사용
- console_scripts
- console_scripts는 entry_points의 하위 항목으로, 커맨드 라인에서 실행할 수 있는 스크립트를 지정
- 'talker = py_pubsub.publisher_member_function:main'
- 이 항목은 'talker'라는 커맨드 라인 명령어를 생성하라는 지시
- 사용자가 커맨드 라인에서 talker라고 입력하면, py_pubsub.publisher_member_function 모듈의 main 함수가 실행
- 결과적으로, 이 설정을 사용하여 python 패키지를 설치하면, 사용자는 커맨드 라인에서 바로 talker 명령어를 사용하여 해당 기능을 실행할 수 있게 됨
- ROS2에서 python 노드를 쉽게 실행할 수 있도록 하는데 특히 유용함
- 이러한 방식을 통해 ROS2는 Python 스크립트를 바로 실행할 수 있는 실행 가능한 커맨드를 제공

setup.cfg 설정
script_dir은 Python 실행 스크립트 (예: entry point로 정의된 노드들)가 설치될디렉터리를 지정 
$base는 설치 루트 디렉토리 (보통 install/ 디렉토리)입니다.
즉, 실행 스크립트가 install/lib/py_pubsub 안에 설치됨
‘ros2 run’ 실행시, path를 제대로 찾게 해주는 역할
- install_scripts=$base/lib/py_pubsub
- 실제 배포용으로 설치할 때도 실행 스크립트를 같은 위치 ($base/lib/py_pubsub)에 두도록 지정.
- 이 경로는 ROS 2의 실행 구조 (lib/<패키지명>/)와 일치시켜서 ros2 run py_pubsub <노드> 같은 명령이 잘 동작하게 합니다.



# 04장. rclpy 이해
rclpy 정의
- ROS2의 Python 클라이언트 라이브러리로, 노드를 만들고 통신할 수 있도록 돕는 핵심 요소
- C++로 작성된 ROS2의 코어 라이브러리(rcl)를 Python 환경에서 활용할 수 있도록 래핑(wrapping)한 것이 특징
- 사용자들은 Python의 간결한 문법과 다양한 라이브러리를 ROS2 기반 시스템에 쉽게 통합

rclpy 구성요소
- 노드(Node)
ROS2 시스템의 기본 실행 단위. 노드는 각각 고유의 이름을 가지며, pub/sub, service/client 등을 통한 통신을 담당
- 퍼블리셔(Publisher)와 서브스크라이버(Subscriber) 
퍼블리셔는 데이터 토픽을 통해 전송하고, 서브스크라이버는 이를 수신. rclpy에서는 간단한 API로 pub/sub를 설정
- 서비스(Service)와 클라이언트(Client)
서비스는 요청-응답 방식의 통신을 처리하며, 클라이언트는 특정 요청을 보내고 응답 대기
- 액션(Action)
긴 시간 동안 실행되는 작업을 요청하고 그 결과를 받을 수 있는 기능. 주로 Motion Planning과 같은 작업에 사용
- 파라미터 서버(Parameter Server)
ROS2 시스템에서 전역적 혹은 로컬 파라미터를 설정하고 가져올 수 있는 시스템

Executor
- ROS2에서 콜백을 실행하는 구성 요소
- 메시지 수신, 서비스 요청 처리, 타이머 이벤트 등 다양한 이벤트에 대한 응답 실행
역할
- 노드가 구독하는 데이터나 이벤트를 적절한 콜백으로 처리하여 메시지의 QoS를 지원(메시지 전달의 신뢰성, 우선순위, 지속성을 설정 가능)
- 시스템의 비동기 처리와 효율성 보장
유형
- SingleThreadExecutor : 단일 스레드에서 콜백을 순차적으로 처리
- MultiThreadExecutor : 여러 스레드에서 콜백을 병렬적으로 처리

ROS2 계층 구조
1. User code : 사용자가 작성한 코드
2. rcl : 클라이언트 라이브러리로, 사용자 코드와 미들웨어를 연결
3. rmw: 미들웨어와 클라이언트 간의 인터페이스 역할. 미들웨어와의 상호작용을 추상화()
4. rmwadapter : RMW와 미들웨어를 연결하는 중간 계층 역할. 이를 통해 ROS2는 특정 DDS 구현체에 종속되지 않고, 다양한 미들웨어를 지원할 수 있는 유연성을 가짐(DDS와 ROS 이어주는 어댑터)
5. Middleware : 실제 메시지 전달과 QoS 설정을 처리하는 계(DDS)

Executor 동작 순서
1. 이벤트 감지 : 노드에서 수신된 토픽, 서비스 요청, 타이머 이벤트 등을 감지
2. 콜백 큐 생성 : 이벤트가 발생할 때마다 대응하는 콜백을 큐(queue)에 수집
3. 콜백 실행 : 큐에 있는 콜백을 적절한 순서대로 실행. 
- SingleThreadedExecutor는 하나씩 처리하고, 
- MultiThreadedExecutor는 병렬로 처리

동작 과정
1. wait : 미들웨어에 메시지가 도착할 때까지 대기
2. take : 새로운 메시지 도착 시, 해당 메시지를 가져옴. 이 과정에서 미들웨어에 저장된 메시지가 클라이언트로 전달됨
3. execute : 메시지 처리를 위한 콜백함수(onGoal, nextCmd, processOdom) 실행

Publisher→DDS network→DDS reader→rmw_take→rcl→WaitSet wakeup→Executor→Subscriber callback

SingleThreadedExecutor : 단일 스레드에서 콜백 실행
- 하나의 스레드만 사용하여 이벤트를 처리하므로, 콜백이 완료될 때까지 다른 작업을 처리할 수 없음
- 주로 처리 속도가 중요한 것이 아니거나, 콜백이 충돌하지 않도록 하기 위해 단일 스레드 환경에서 사용
StaticSingleThreadedExecutor : 단일 스레드에서 정적으로 콜백 실행
- 정적 단일 스레드는 구독, 타이머, 서비스 서버, 액션 서버 등 노드 구조 스캔하는 런타임 비용 최적화
- 노드가 추가될 때 콜백 스캔을 한 번만 수행되며, 다른 두 Executer는 이러한 변화를 정기적으로 스캔
- 정적 단일 스레드 실행자는 초기화 중에 모든 구독, 타이머 등을 생성하는 노드와 함께 사용해야 함

MultiThreadedExecutor : 여러 스레드에서 콜백을 병렬로 실행
- 여러 스레드가 동시에 실행되기 때문에 복잡한 작업이나 멀티태스킹 환경에서 유리
- 그러나 다중 스레드 간의 자원 경쟁이나 동기화 문제가 발생할 수 있으므로, 적절한 
동기화 처리(locking) 필요

||Executor|spin()|
|:---|:---|:---|
|역할|콜백 관리 및 실행|이벤트 루프 실행|
|설명|  노드에 포함된 콜백(토픽, 서비스, 타이머 등)을 관리하고,이벤트가 발생하면 적절한 콜백을 실행하는 역할 수행<br>spin()이 호출되면 노드는 콜백이 발생하기를 기다리며, 이벤트가 감지될 때 콜백을 실행<br>SingleThreadedExecutor와 MultiThreadedExecutor같은 다양한 종류가 있으며, 콜백을 실행하는 방식에 따라 동작 방식 달라짐|Executor가 이벤트 처리하는 루프 실행<br>Executor가 동작할 수 있는 환경을 유지하는 루프이며, 명시적으로 종료하거나 프로그램이 종료될 때까지 계속 실행

#### Executor와 spin()의 관계
- Executor는 콜백을 처리하는 "관리자"이고, spin()은 해당 Executor가 콜백을 계속해서 처리하도록 하는 "루프"
- Executor는 콜백을 실행하는 규칙과 방식을 정의하고, spin()은 그 규칙에 따라 Executor가 동작하도록 해주는 실행 메커니즘
멀티스레드와의 연관성
- spin()을 사용하면 Executor가 콜백을 처리하는 동안 계속해서 대기하지만, MultiThreadedExecutor를 사용하면 여러 콜백을 동시에 병렬로 처리할 수 있음
- 이때도 spin()을 호출하여 이벤트 루프가 유지되지만, 여러 스레드가 동시에 동작하면서 여러 콜백을 병렬 처리 가능
SingleThreadedExecutor + spin(): 한 번에 하나의 콜백만 처리
MultiThreadedExecutor + spin(): 여러 콜백을 동시에 처리

Conclusion
- rclpy는 ROS2에서 Python을 활용해 로봇 시스템을 신속하게 구축할 수 있는 강력한 도구
- ROS2의 유연한 통신 모델과 결합하면 복잡한 로봇 애플리케이션을 손쉽게 설계하고 확장 가능
- Executor와 spin()은 서로 다는 개념이지만, 함께 사용되어 ROS2 시스템에서 콜백을 효율적으로 관리하고 실행

Best practice
노드 이름 관리
- 노드 이름은 고유해야 함, 노드를 생성시 시스템 내 다른 노드들과 충돌치 않도록 이름을 신중히 설정
- 예를 들어, 서비스와 퍼블리셔가 동일한 노드 내에서  작동하는 경우, MultiThreadedExecutor를 사용 가능
타이머 주기 관리
- 타이머 콜백(Callback) 주기는 너무 짧게 설정하지 않도록 주의
- 지나치게 빠른 주기는 CPU부하의 과도한 증가의 원인
에러 핸들링
- ROS2 환경은 네트워크 및 하드웨어 상태 따라 불안정할 수 있으니, 예외상황에 대한 에러 핸들링 철저 필수

최적화 팁
토픽 QoS설정
- 네트워크 환경이나 메시지의 중요도에 따라 QoS(Quality of Service) 설정을 맞춰야 함
- 예를 들어, 메시지 유실이 치명적이지 않은 센서 데이터의 경우 Best Effort 방식을 사용할 수 있음
파라미터 최적화
- 파라미터 서버를 활용해, 노드의 설정 값을 유연하게 바꾸면서 성능 튜닝 가능



# 05장: 액션 패키지
액션 통신: 3개의 서비스와 2개의 토픽으로 구성
서비스: Action Goal(send goal), Action CancelGoal(cancel goal), Action Result(get result)
토픽: Action GoalStatus(status), Action Feedback(feedback)

Goal State Machine
- 비동기 방식은 원하는 타이밍에 적절한 액션을 수행하기 어려워 이를 원활히 구현하기 위해 목표상태(goal_state)를 ROS 2에서 새롭게 선보임. 
- 목표 상태는 목표값을 전달한 후의 상태 머신 구동해 액션 프로세스를 쫒는 것
- 상태머신은 GoalStateMachine으로 다음 그림과 같이 액션 목표전달 이후의 액션의 상태값을 액션 클라이언트에게 전달할 수 있어서 비동기, 동기방식이 혼재된 액션의 처리를 원활하게 할 수 있음.
ROS 2 메시지에서는 action_msgs/msg/GoalStatus 구조체로 정의

Action동작 다이어그램
활성 상태는 세 가지가 있다.
-수락됨-목표가 수락되었으며 실행을 기다리고 있다.
-실행 중-목표는 현재 액션 서버에서 실행 중.
-취소 중-클라이언트가 목표 취소를 요청했고 액션 서버가 취소 요청을 수락. 
이 상태는 액션 서버가 수행해야 할 수 있는 사용자 정의 "정리" 작업에 유용.
그리고 세 가지 최종 상태:
-성공-액션 서버가 목표를 성공적으로 달성했습니다.
-중단됨-목표가 외부 요청 없이 Action Server에 의해 종료되었습니다.
-취소됨-액션 클라이언트의 외부 요청 이후 목표가 취소되었습니다.
설계된 동작에 따라 액션 서버에서 트리거되는 상태 전환:
-실행-승인된 목표의 실행을 시작합니다.
-succeed-목표가 성공적으로 완료되었음을 알립니다.
-중단-목표 처리 중에 오류가 발생하여 중단해야 함을 알립니다.
-취소됨-목표 취소가 성공적으로 완료되었음을 알립니다.
액션 클라이언트에 의해 트리거되는 상태 전환:
-send_goal-목표가 액션 서버로 전송. 상태 머신은 액션 서버가 목표를 수락하는 경우에만 시작.
-cancel_goal-액션 서버에 목표 처리 중단하도록 요청. 액션 서버가 목표 취소 요청을 수락한 경우에만 전환 발생.

Goal State 전달
- 클라이언트가 서버에 목표를 보낸 후, 서버와 Goal Status Topic을 통해 상태를 확인가능.
- 클라이언트는 get_goal_status() 또는 wait_for_result()를 통해 목표 상태를 주기적으로 확인 가능
- Status 업데이트 주기는 서버가 내부적으로 상태가 바뀔 때마다 자동으로 전달됩니다.
- 목표 상태가 바뀔 때마다 publish됨
- 예: ACCEPTED → EXECUTING → SUCCEEDED

항목              Goal Status                     Feedback
목적         목표의 상태 메타 정보            목표 진행 중 상세 정보
예        SUCCEEDED, CANCELED, ABORTED   진행률, 현재 위치, machine state
전달 시점         상태 변경 시            서버가 publish_feedback() 호출 시
주기             상태 변경마다                  자유롭게 정의 가능

Action Server
액션 서버는 액션을 제공. 토픽이나 서비스와 마찬가지로 액션 서버는 이름과 유형을 가진다. 
이름은 네임스페이스를 지정할 수 있으며, 모든 액션 서버 간에 고유해야 한다. 
즉, 동일한 유형을 가진 여러 액션 서버가 (서로 다른 네임스페이스에서) 동시에 실행될 수 있다.

다음의 역할을 담당.
- 다른 ROS 엔티티에 작업 광고
- 하나 이상의 액션 클라이언트로부터 목표수락 또는 거부
- 목표가 수신되고 수락되면 작업을 실행합니다.
- 선택적으로 모든 실행 작업의 진행 상황에 대한 피드백 제공
- 선택적으로 하나 이상의 작업을 취소하기 위한 요청 처리
- 완료된 작업의 결과(성공, 실패 또는 취소 여부 포함)를 결과를 요청한 클라이언트에게 전송.

Action Client
액션 클라이언트는 하나 이상의 Goal(수행할 액션)를 전송하고 진행 상황을 모니터링.
서버당 여러 클라이언트가 있을 수 있지만, 여러 클라이언트 Goal를 동시 처리하는 방법은 서버가 결정.

다음의 역할을 담당.
- 액션 서버로 Goal 전송
- 선택적으로 Action Server에서 Goal에 대한 사용자 정의 피드백을 모니터링.
- 선택적으로 Action Server에서 승인된 Goal의 현재 상태를 모니터링
- 선택적으로 Action Server에서 활성 Goal를 취소하도록 요청.
- 선택적으로 Action Server에서 수신한 Goal에 대한 결과를 확인

Action interface  정의
action은 ROS 메시지 IDL 형식을 사용하여 지정. 
이 사양은 세 섹션으로 구성되며, 각 섹션은 메시지 사양.
1. 목표(GOAL): 액션이 뭘 달성할지, 또한 어떻게 달성할지 설명. 액션 실행 요청 오면 액션 서버로 전송.
2. 결과(RESULT): 작업 결과 설명. 작업 실행 성공적 종료했는지 여부 관계없이 서버서 클라이언트로 전송.
3. 피드백(FEEDBACK): 작업 완료까지 진행 상황 나타냄. 작업 실행 시작부터 완료 전까지 Action Server에서 해당 작업의 클라이언트로 전송. 클라이언트는 이 데이터를 사용해 작업 실행 진행 상황 파악. 
이 섹션들은 모두 비어 있을 수 있음. 세 섹션 사이에는 세 개의 하이픈(---)이 포함된 구분자가 있음.



# 06장: 인터페이스 프로그래밍(응용1)
패키지 설계
- ROS2의 토픽, 서비스, 액션 프로그래밍을 이용해서 각 노드 들이 서로 연동되어 구동하는 패키지 설계
- 프로세스 목적별로 나누어 노드 단위 프로그램 작성하고 노드와 노드 간의 데이터 통신을 고려해 설계
실습 패키지 설계
- 계산기 개발
- 현재 시간과 변수 a, b를 받아 연산하여 결과값 도출
- 연산 결과값을 누적하여 목표치에 도달했을 때 이 결과값을 표시

callback_group
- MutuallyExclusiveCallbackGroup이 기본 설정으로 사용됨
- MutuallyExclusiveCallbackGroup: 한 번에 하나의 콜백 함
수만 실행하도록 제한
- ReentrantCallbackGroup: 제한 없이 콜백 함수를 병렬로 
실행 가능

실행 인자
- 프로그램 실행 시 추가로 입력되는 인수로, main 함수의 매개변수로 사용됨
- 실행 명령어와 함께 전달되어 프로그램 동작에 영향을 줌
예시: $ ros2 run ex_calculator checker –g 100
- ros2 run: ROS2 명령어
- ex_calculator: 패키지 이름
- checker: 실행할 노드
--g 100: 실행 인자, 여기서는 GOAL_TOTAL_SUM 값을 100으로 설정
참고로 파라미터(parameter)는 매개 변수로 풀이하고 아규먼트(argument)는 실행 인자라 풀이된다. C++ 언어에서는 이들의 분류를 더 확실히 하는 편인데 Parameter는 함수 선언시 사용되고 Argument는 함수 호출 시의 인수라고 생각하면 된다
- Parameter 매개 변수, Argument 실행 인자


# 07장: 인터페이스 프로그래밍(응용2)
행맨 만들기:



# 08장: 주피터 노트북 사용



# 09장: ROS2 응용 핵심 정리
Topic 관련 함수
퍼블리셔
publisher = self.create_publisher(MsgType, 'topic_name', qos_profile)
publisher.publish(msg)
서브스크라이버
self.subscription = self.create_subscription(MsgType, 'topic_name', self.callback, qos_profile)

Service 관련 함수
클라이언트
```py
self.cli = self.create_client(SrvType, "service_name")

# 서비스 요청 준비
req = SrvType.Request()
req.param = value

# 서버가 준비될 때까지 대기
while not self.cli.wait_for_service(timeout_sec=1.0):
      self.get_logger().info("Waiting for service...")

# 요청 보내기
future = self.cli.call_async(req)
future.add_done_callback(response_callback)
```
서버
self.srv = self.create_service(SrvType, "service_name", self.callback)

Action 관련 함수
액션 클라이언트
```py
self._action_client = ActionClient(self, ActionType, "action_name")

# 서버 대기
self._action_client.wait_for_server()

# goal 생성
goal_msg = ActionType.Goal()
goal_mag.param = value

# goal 전송
self._send_goal_future = self._action_client.send_goal_async(
      goal_msg, feedback_callback=self.feedback_callback
)

# 결과 기다리기
self._send_goal_future.add_done_callback(self.goal_response_callback)
```
액션 서버
```py
self._action_server = ActionServer(
      self,
      ActionType,
      "action_name",
      self.execute_callback,        # execute 단계에서 실행
      goal_callback=self.on_goal,   # goal 도착시 실행
)
```
rclpy.spin(node) 호출시
```py
while rclpy.ok():
    # 1. wait: DDS WaitSet으로부터 이벤트 기다림
    # 2. take: 누가 publish한 메시지 등 도착
    # 3. execute: 등록된 콜백 함수 실행
```
from rclpy.action import ActionServer, GoalResponse, CancelResponse

ActionServer: 액션 서버 생성 클래스, 노드 안에서 특정 액션 타입 처리 가능하게 만듬
GoalResponse: 골 수락할지 거부할지 나타내는 상수(ACCEPT, REJECT)
CancelResponse: 클라이언트가 요청한 골 취소 요청 응답 상수(ACCEPT, REJECT)






→ ☆ -