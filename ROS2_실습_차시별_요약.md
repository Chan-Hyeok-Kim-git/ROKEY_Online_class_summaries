# 01장: ROS2 CLI 심화
### ROS2 CLI 명령어
  ros2 <command> <verb> [sub-verb] [options] [arguments]
- command: 동작 지정, 수행할 작업의 유형 나타냄. topic, service, node 등이 올 수 있음
- verbs ,sub-verbs: 특정 동작에 대한 세부 동작(sub-verb) 지정. command가 topic인 경우 pub, echo, list, hz 등 올 수 있음
- options: 명령어 실행 방식 설정하는 추가 파라미터. –h,-r, --node-name,--qos, -ros-args 등이 올 수 있음
- arguments: 실행시 필요한 인수 지정. 특정 노드 이름이나 토픽 이름,서비스 이름 등 올 수 있음

  ROS 2 CLI는 내부적으로 이렇게 동작

1.ros2 실행 → 2.entry point에서 command 선택 → 3.해당 command에 등록된 verb 중 하나 선택

#### ROS2 CLI + arguments
|ros2cli + [verbs]|[arguments]|기능|
|:---|:---|:---|
|ros2 run|package executable|특정 패키지의 특정 노드 실행,(1개의 노드) *executable에 따라 복수 노드도 실행 가능|
|ros2 launch|package launch-file|특정 패키지의 특정 런치 파일 실행 (0개 ~ 복수개의 노드)|

#### ros2 security
- Security는 SROS의 유틸리티로, DDS-Security를 ROS2에서 사용하기 위해 필요한 도구를 모아둔 것
- ros2 security는 ROS 2에서 보안 기능을 설정하고 관리하는 명령어
- ROS 2는 DDS(Data Distribution Service)를 기반으로 하지만 기본적으로 보안이 비활성화
- 이를 활성화하려면 SROS2 (SecureROS 2) 및 DDS-Security 표준을 사용해야 함.
- 보안 기능 활성화하면 인증(Authentication), 암호화(Encryption), 액세스 제어(Access Control) 등의 기능을 사용 가능

[ROS 2 보안기능활용예시]
 1) 자율주행로봇데이터보호
  - 카메라, LiDAR 센서 데이터를 암호화 하여 보호
  - 외부에서 허가되지 않은 노드가 데이터를 읽거나 수정 하지 못하도록 설정
 2) 산업용 로봇 시스템 보안
  - 로봇 제어 명령을 허가된 노드에서만 보낼 수 있도록 제한
  - 공장 네트워크에서 보안이 유지되도록 암호화된 통신 사용
 3) 클라우드 연동 IoT 시스템 보안 강화
  - 클라우드에서 ROS2 기반 로봇과 안전하게 통신
  - TLS 및 DDS 보안 프로토콜을 적용하여 외부 공격으로부터 보호



# 02장: IPC(Intra Process Communication)
### Intra-process communication
- ROS는 복수개의 node를 사용하여 개발이 이루어짐
- 단일 컴퓨팅 시스템에서 복수개의 node 사용시
- 데이터 통신을 위한 작업으로 인한 전체적인 성능 저하 및 메모리 사용량 증가하는 단점
- ROS2에서는 이를 해결하기 위해 IPC(Intra-Process Communication) 제공
- 예, 모바일로봇: 라이다 데이터 노드, 모터 제어 노드, 로봇 위치 추종 노드, 경로 생성 노드 등...

- 서로 다른 프로세스는 송수신 되는 데이터가 여러번 메모리에 복사되어 성능 저하 발생

[기본적인 데이터 복사]
일반적으로 ROS2에서 노드간 데이터 전달할 때, 메시지는 다음과 같은 단계를 거침
1. 퍼블리셔가 메시지 생성
- 사용자가 메시지 생성하면, 해당 데이터가 메모리에서 특정 주소에 저장됨
2. DDS 미들웨어를 통해 데이터 전달
   1. ROS2는 DDS(Data Distribution Service)를 사용하여 메시지를 전달
   1. 일반적인 DDS구현에서는 데이터를 네트워크 버퍼 또는 공유메모리로 복사하여 송신
3. 구독자가 데이터 수신
   1. 수신된 데이터는 다시 사용자 프로그램의 메모리로 복사
   2. 여러개의 구독자가 존재할 경우, 각 구독자에게 별도로 복사

이 과정에서 메모리 복사가 여러 번 발생, 특히 대용량 데이터(이미지, LiDAR 포인트클라우드 등)전송시 메모리 사용량 증가와 성능 저하 발생

- IPC를 이용하면 복수 개의 노드를 단일 프로세스에서 처리해 해당 문제 해결<br>
[Zero-Copy 란?]<br>
데이터 복사를 최소화해 성능을 최적화하는 기술<br>
ROS2에서는 Fast DDS SHM(Shared Memory) 및 Cyclone DDS Iceoryx 등의 DDS 미들웨어에서 지원

[Zero-Copy 방식의 데이터 흐름]
1. 퍼블리셔가 메시지 생성하면, 데이터가 공유메모리(SHM, Shared Memory)에 저장
2. DDS는 데이터를 네트워크로 전송하지 않고, 같은 프로세스 또는 동일한 머신 내 구독자들에게 참조 방식으로 전달
3. 구독자는 데이터를 복사 없이 직접 참조해 사용
4. 즉, 데이터가 한 번만 생성되고 여러 구독자가 이를 직접 읽을 수 있어 메모리 복사가 필요없음

### DDS의 서비스품질(QoS, Quality of Service)
- QoS(Quality of Service)란 쉽게 말해 ‘데이터 통신 옵션’
- ROS2는 TCP 방식과 UDP 방식을 선택적으로 사용 가능
- TCP: 신뢰성(Reliability) 중심
- UDP: 속도 중심
- 이를 위해 ROS2에서는 DDS의 QoS 도입
- 퍼블리셔 또는 서브스크라이버 선언시 QoS를 매개변수로 지정하여 원하는 통신 방식 설정 가능
- QoS로 바꿀 수 있는 것은 데이터 전송시 실시간성(real time) 설정 관련 부분, 대역폭 옵션, 데이터 지속성, 중복성 등

현재 DDS에서 설정 가능한 QoS 항목으로는 22가지가 있음

대표적인 QoS 항목
#### Reliability: ROS2에서는 신뢰도를 우선(reliable)으로 설정하거나 통신 속도 최우선(best effort)으로 설정
|Reliability|신뢰성 또는 속도 우선 설정|
|:---|:---|
|BEST_EFFORT|데이터 송신 집중. 전송 속도 중시하며 네트워크에 따라 유실 발생 가능성|
|RELIABLE|데이터 수신 집중. 신뢰성 중시하며 유실 발생시 재전송을 통해 수신 보장|

#### History: 정해진 크기만큼 데이터를 보관하는기능(=depth)
|History|데이터를 몇 개나 보관할 지 결정하는 QoS 옵션|
|:---|:---|
|KEEP_LAST|정해진 메시지 큐 사이즈(depth)만큼 데이터 보관<br>depth: 메시지 큐 사이즈(KEEP_LAST 설정일 경우에만 유효)
|KEEL_ALL|모든데이터보관(최대 사이즈는 DDS 벤더마다 다름)

#### Durability: 데이터 수신하는 서브스크라이버가 생성되기 전, 데이터의 사용 유무 설정
|Durability|데이터 수신하는 서브스크라이버가 생성되기 전, 데이터의 사용 유무 설정|
|:---|:---|
|TRANSIENT_LOCAL|Subscription이 생성되기 전 데이터도 보관(Publisher에만 적용 가능)|
|VOLATILE|Subscription이 생성되기 전 데이터는 무효|

#### Deadline: 정해진 주기 내 데이터의 발신 및 수신이 없는 경우 이벤트 함수 실행
|Deadline|정해진 주기 내 데이터의 발신 및 수신이 없는 경우 이벤트 함수 실행|
|:---|:---|
|deadline_duration|Deadline을 확인하는 주기|

#### Lifespan: 정해진 주기 내 수신되는 데이터에만 유효판정, 이외 데이터는 삭제
|Lifespan|정해진 주기 내 수신되는 데이터에만 유효 판정, 이외 데이터는 삭제|
|:---|:---|
|lifespan_duration|Lifespan을 확인하는 주기|

#### Liveliness: 정해진 주기 내 노드 또는 토픽의 생사 확인
|Liveliness|정해진 주기 내 노드 또는 토픽의 생사를 확인|
|:---|:---|
|liveliness|자동 또는 매뉴얼로 확인할 지 지정하는 옵션(3가지 중 선택)(AUTOMATIC, MANUAL_BY_NODE, MANUAL_BY_TOPIC)|
|lease_duration|Liveliness를 확인하는 주기|

#### RMW QoS Profile: ROS2의 RMW에서 가장 많이 사용하는 QoS설정을 하나의 세트로 표현한 것

- 목적에 따라 Default, Sensor Data, Service, Action Status, Parameters, Parameters Events의 6가지로 구분

||Default|Sensor Data|Service|Action Status|Parameters|Parameter Events|
|:---|:---|:---|:---|:---|:---|:---|
|Reliability|RELIABLE|BEST_EFFORT|RELIABLE|RELIABLE|RELIABLE|RELIABLE|
|History|KEEP_LAST|KEEP_LAST|KEEP_LAST|KEEP_LAST|KEEP_LAST|KEEP_LAST|
|Depth(History)|10|5|10|1|1000|1000|
|Durability|VOLATILE|VOLATILE|VOLATILE|TRANSIENT|LOCAL|VOLATILE|VOLATILE|

#### History: 메시지를 몇 개까지 버퍼에 저장할지를 설정하는 옵션
- 데이터 전송 시점 이후 보관할 데이터의 정책을 설정하는 옵션
- KEEP_LAST: Depth로 설정한 만큼 사이즈의 데이터보관(최근 몇개까지만저장)
- KEEP_ALL: 모든 데이터 보관(시스템 메모리한도까지)
- 제공된 'py_pubsub' 코드를 활용하여 History QoS 설정 실습
- Publisher와 Subscriber의 QoS 프로파일 값을 변경

#### Reliability: 메시지를 얼마나 신뢰성 있게 전달할지를 설정하는 옵션
- TCP처럼 손실을 방지하면서 신뢰도를 우선시(전달보장): RELIABLE
- UDP처럼 손실을 감안하고 통신속도를 우선시(데이터유실허용, 실시간성): BEST_EFFORT
- 이번 예제에서는 인위적으로 네트워크 손실을 발생한 뒤, BEST_EFFORT를진행
- ROS2에서 기본적으로 제공하는'demo_node_cpp' 패키지를 활용
[활용 사례]
- BEST_EFFORT(약간의손실을 감수하더라도 지연없는빠른처리가더중요한경우)
1. 카메라영상스트리밍(frame 몇개누락되어도괜찮음)
2. LiDAR센서데이터
3. 주행중실시간거리센서
4. 드론영상중계
[활용 사례]
- RELIABLE(모든메시지는 반드시전달되어야하는경우)
1. 로봇제어명령(STOP, TURN, MOVE)
2. 지도데이터전송(SLAM map)
3. 긴급정지신호(Emergency Stop)
4. 산업용로봇공정제어신호

#### Durability: Publisher가 보낸 메시지를 얼마나 오래 저장해서 새로운 Subscriber에게 줄 것인가
- Subscriber가 생성되기 전, 데이터를 사용할지폐기할지에대한QoS 옵션
- TRANSIENT_LOCAL: Publisher가 마지막으로 보낸 메시지를 메모리에 저장. 새로운Subscriber가 연결되면 전달(Publisher에만
적용가능)
- VOLATILE: Subscriber가 연결되기 전 데이터는 사용하지 않고버림. 새로 연결되면새로운메시지부터받음
- 'py_pubsub' 패키지에서 QoS 프로파일을 변경하여 적용
- VOLATILE 로설정하는예제실행
[활용 사례]
- TRANSIENT_LOCAL
1. 새로운Subscriber가 붙었을때 지금현재로봇의상태를알고싶을때
2. SLAM 후완성된맵을Publishing 하는노드
3. 나중에들어오는네비게이션노드가맵을받아야하는경우
[활용 사례]
- VOLATILE
1. 과거영상프레임은의미가없고실시간프레임만받고싶을때
2. Publisher: /camera/image_raw
3. Subscriber: 영상 Viewer

#### History와 Durability의 관계
||History|Durability|
|:---|:---|:---|
|개념|얼마나 많은 메시지(몇개)를 보관할까?|새로 등장한 Subscriber에게 이전 메시지를 줄지 안줄지를 결정
|종류|KEEP_LAST: 마지막n개저장(Depth)<br>KEEP_ALL: 가능한모든메시지저장|VOLATILE: 새로운메시지만받음<br>TRANSIENT_LOCAL: Publisher가최근발행한메시지저장하고나중에등장한Subscriber에게메시지보내줌(개수는History에서결정)|
|예시|VOLOTILE+KEEP_LAST(10)→Publisher는최근10개메시지저장. New Subscriber가나중에붙으면? 과거데이터못받음<br>TRANSIENT_LOCAL + KEEP_LAST(10) →Publisher는최근10개메시지저장. New Subscriber가나중에붙으면? 최근10개중가능한 메시지다시보내줌||
|사례|센서스트리밍(LiDAR등)→VOLATILE+KEEP_LAST(depth 적당히)<br>중요한공지사항(경고/에러메시지)→TRANSIENT_LOCAL+KEEP_LAST(1 ~ 몇개)<br>초기설정정보(맵데이터, Config) →TRANSIENT_LOCAL + KEEP_ALL(메모리여유있는경우)||

※ Durability가TRANSIENT_LOCAL이어야New Subscriber가과거메시지를받을수있다! 얼마나받을수있는지는History(Depth)에따라!

#### Deadline: 정해진 시간안에 메시지가 도착해야 함. 설정한 시간 내 도착하지 않으면 이벤트 발생.
- 정해진 주기 내 데이터의 발신 및 수신이 없는 경우, EventCallback 함수를 실행하는QoS 옵션
- ROS2의 기본패키지 quality_of_service_demo의 deadline.py 예제 살펴보기
- /opt/ros/humble/lib/python3.10/site-packages/quality_of_service_demo_py/deadline.py
[활용 사례]
- 100ms마다 센서 값을 Publishing하는 노드가 센서 고장으로 1초동안 데이터를 못 보낸다면
- Subscriber는 센서가 문제 있다는것을알수있음→센서데이터감시
- 로봇의 Heartbeat 또는 제어 명령: Robot이 일정 주기로 살아있다는 메시지를 보내야 하거나 500ms이상 끊어지면 로봇은 안정상 멈춰야 할 수도 있음. 
- 500ms동안 새로운 명령이 없으면 로봇이 Emergency Stop 모드로 전환해야 할 수도
- 환자의Vital Sign 데이터를 주기적으로 받아야 하는 모니터링 시스템 → 데이터가 일정 주기 이상 끊어지면 즉시 경고 발생

#### Lifespan: 메시지 유효기간(수명) 지정 옵션. Publisher의 데이터 발행 순간 부터 lifespan 지나면 그 데이터 무효
- 정해진 주기 내 수신되는 데이터만 유효 판정, 이외 데이터는 삭제하는 QoS 옵션
- ROS2의 기본 패키지 quality_of_service_demo의 lifespan.py 예제 살펴보기
- /opt/ros/humble/lib/python3.10/site-packages/quality_of_service_demo_py/lifespan.py
[활용 사례]
- 카메라프레임, LiDAR 스캔을 오래된 데이터를 받으면 의미 없으므로 200ms 이상 된 데이터는 버리고 최신 데이터만 받고 싶을 때
- 로봇의 위치정보(/odom) 같은 경우 1초 이상 된 데이터는 현재 위치와 차이가 있으므로 버려야 할 수도 있음
- 장비 상태 알림이 5초 이상 지연 되면 의미 없을수도 있으므로 lifespan을 5초로 설정
- 로봇 팔을 움직이는 명령은 1초가 지난 명령인 경우 무시

#### Liveliness: Publisher가 아직 살아있음을 Subscriber에게 어떻게 보장할지 정하는 방법
- 정해진 주기 내 노드 또는 토픽의 생사를 확인하는 QoS 옵션
- Publisher가 여전히 활성 상태인지를 Subscriber가 확인할 수 있도록 하는 QoS 정책
- 특정 시간 동안 응답하지 않으면 “비활성화됨“ 상태로 판단함(liveliness_lease_duration안에 최소 1번은 신호를 보내야 함)
- Liveliness 설정 값(자동 또는 매뉴얼로 확인할지결정)
- AUTOMATIC: 기본 옵션. Publisher가 메시지 보낼 때 자동으로 활성상태로 간주됨(RMW가 알아서 Publisher를 감시)
- MANUAL_BY_TOPIC: Publisher가 특정 주기마다 DDS에게 “나는 살아있다＂는 신호를 보내는방식(rclpy.assert_liveliness 메서드를 호출)
- ROS2의 기본패키지 quality_of_service_demo의 liveliness.py 예제 살펴보기
- /opt/ros/humble/lib/python3.10/site-packages/quality_of_service_demo_py/liveliness.py
[활용 사례]
- 로봇 제어 명령 감시: Publisher가 죽었을 경우 즉시 로봇 모터를 멈춤
- 센서 데이터 생존 확인: 센서로부터 데이터 전송이 없으면 즉시 다른 예비 센서로 스위치 또는 경고 메시지
- 멀티 로봇 통신 안정성: 각 로봇/드론이 서로 위치/상태를 공유하는 경우 특정 로봇이 통신에서 사라지면 즉시 알아채서 경로 재설정 또는 팀 전략 수정
- 시스템 운영 중 어떤 노드가 죽으면 빠르게 디버깅 해야 함

|QoS|설명|옵션|
|:---|:---|:---|
|Reliability|UDP 처럼 통신속도를 최우선 할지<br>TCP처럼 데이터 손실 방지하며 신뢰도를 우선할지|- BEST_EFFORT(속도우선)<br>- RELIABLE(신뢰도우선)|
|History|통신 상태에 따라 정해진 사이즈 만큼의 데이터를 보관|- KEEP_LAST<br>- KEEP_ALL|
|Durability|데이터를 수신하는 Subscriber가 생성되기 전의 데이터를 사용할지 폐기할지에 대한 설정|- TRANSIENT_LOCAL<br>- VOLATILE(휘발성)|
|Deadline|정해진 주기 안에 데이터가 발신 및 수신되지 않을 경우 이벤트 함수를 실행 시킴|deadline_duration(단위:ms) (700, 1000 등)|
|Lifespan|정해진 주기 안에 수신되는 데이터만 유효 판정하고 그렇지 않은 데이터는 삭제|lifespan_duration(단위:ms) (700, 1000 등)|
|Liveliness|정해진 주기 안에서 노드 혹은 토픽의 생사를 확인|Liveliness(AUTOMATIC, MANUAL_BY_TOPIC)|

# 03장: ROS2 심화 기능2
### Component
1. Composable Node<br>
-여러 노드를 하나의 프로세스에 실행해, 노드간 IPC 비용을 줄이는 기술
-각 노드는 Component로 구현되어야 하며, 실행 시 Container에 로드됨
1. Component<br>
-"Node"를 하나의 클래스 객체처럼 만들고, 다른 Node들과 함께 하나의 프로세스 내에 로드 될 수 있도록 만든 구조
1. Container<br>
-Component들을 실행할 수 있는 빈 실행 환경. 노드를 동적으로 로드 하거나 런타임에 조합 할 수 있게 해줌

|C++ (rclcpp)|Python (rclpy)|
|:---|:---|
|정적타입, 컴파일된.so 생성가능|동적타입, .so 파일없음|
|pluginlib, class_loader 사용 가능|Python에는 해당 플러그인 시스템 없음|
|런타임에 클래스 인스턴스 로딩 가능|Python은 런타임 클래스 로딩 불안정|
|고성능 실시간 통신에 적합|프로세스 격리 필요, IPC 필수|

Python에서 유사하게 하려면 다중 노드를 하나의 프로세스에서 실행

### lifecycle
- ROS 2 Humble에서는 노드 생명주기 관리하는 Lifecycle Node라는 개념 제공
- 노드(Node)의 상태를 명확하게 관리하기 위한 개념으로 공식적으로는 “Managed Nodes”라고
도 불림
- 이것은 노드가 명확한 상태(state)를 갖고 있고, 그 상태간의 전이를 통해 노드를 더 잘 제어할 수 있게해줌. 
- 로봇 시스템에서 안정성, 예측 가능성, 명시적인 상태 관리가 필요할때 유용

- 노드는 주요상태(Unconfigured, Inactive, Active, Finalized)와 전환 상태(Configuring, Activating, Deactivating, Cleaning Up 등)를 가짐
- 노드를 체계적으로 관리 및 상태 전환을 통해 노드를 구성, 활성화, 비활성화, 정리가능

[사용 목적]
- 노드가준비중인지
- 아직데이터수집안한상태인지
- 안전하게작동가능한상태인지
- 위와같이노드의상태변화를명확하게관리

[아래상황에서유용]
- 하드웨어 의존성이 있는 노드
- 여러 노드의 순차적 실행 제어가 필요한 경우
- 안정성이 중요한 로봇(자율주행, 산업용로봇등)

#### Lifecycle Node 사용하는 이유
- 명시적인 상태 전이: configure→activate→deactivate등
- 리소스 관리: configure시센서열기, cleanup시 정리
- 테스트 및 안정성 향상: 상태 기반 테스트 가능

#### 주의사항
- Lifecycle Node는 일반Node보다 조금 더 복잡하며, 주로 센서/액추에이터 같은 리소스를 명확히 제어해야 하는 노드에서 사용
- ROS2의 일부 패키지에서는 Lifecycle을 지원하지 않을 수 있음.

|항목|일반Node (Node)|Lifecycle Node (LifecycleNode)|
|:---|:---|:---|
|상태 전이|없음|명시적 상태 전이(configure, activate, ...)|
|사용 목적|일반작업용노드|정교한 상태 관리 필요 시|
|ROS 기본포함여부|O|O (기본 제공 모듈에 포함)|
|구현 필요 여부|간단|사용자 정의 클래스 필요|

### ROS2 LifecycleNode에서 오버라이드 가능한 콜백 메서드들
|콜백 메서드 이름|호출 시점|설명|
|:---|:---|:---|
|on_configure(self, state)|unconfigured → configuring|노드를 설정하는 단계. <br> 센서/파라미터로딩등|
|on_activate(self, state)|inactive → activating|퍼블리셔나 타이머 활성화 등 동작 시작|
|on_deactivate(self, state)|active → deactivating|동작 중단. 퍼블리셔 비활성화 등|
|on_cleanup(self, state)|inactive → cleaning up|리소스 해제, 초기화 해제|
|on_shutdown(self, state)|어떤 상태에서든 종료 시|노드가 종료될 때 호출|
|on_error(self, state)|예외 상황 발생 시|오류 처리 로직(optional)|

ROS2에서는 Lifecycle 인터페이스를 통해 노드의 상태 확인이나 재실행, 교체가가능

예시: 카메라 센서를 통해 받은 이미지 정보를 발간하는 노드
- 먼저 노드를 동작시키기 전에 카메라와의 통신을 위한 포트가 제대로 잡혔는지 확인
- 만약 노드가 동작되는 도중에 에러가 발생하였다면 잠시 그 동작을 멈추고 에러를 해결한 다음 재시작
- 주변 환경 변화로 인해 에러를 해결할 수 없다면 해당 노드는 종료시키고 준비된 다른 노드를 동작

주요상태
- Unconfigured: 노드가 생성된 직후의 상태, 에러 발생 이후 다시 조정 될 수 있는 상태
- Inactivate: 노드가 동작을 수행하지 않는 상태. 파라미터 등록, 토픽 발간과 구독 추가 삭제 등을(재)구성할 수 있는 상태
- Activate: 노드가 동작을 수행하는 상태.
- Finalized: 노드가 메모리에서 해제되기 직전 상태노드가 파괴되기 전 디버깅이나 내부 검사를 진행할 수 있는 상태

전환상태
- Configuring: 노드를 구성하기 위해 필요한 설정 수행
- CleaningUp: 노드가 처음 생성된 시기의 상태와 동일하게 만드는 과정 수행
- Activating: 노드가 동작을 수행하기 전 마지막 준비 과정 수행
- Deactivating: 노드가 동작을 수행하기 전으로 돌아가는 과정 수행
- ShuttingDown: 노드가 파괴되기 전 필요한 과정 수행
- ErrorProcessing: 사용자 코드가 동작되는 상태에서 발생하는 에러를 해결하기 위한 과정 수행

전환
- Create: 노드를 생성하고 초기 상태로 설정
- Configure: 노드를 구성하여 준비 상태로 만듦
- Cleanup: 노드를 초기화하여 이전 상태로 되돌림
- Activate: 노드를 활성화하여 기능을 수행할 수 있게함
- Deactivate: 노드를 비활성화하여 동작을 멈춤
- Shutdown: 노드를 안전하게 종료하는 과정
- Destroy: 노드를 메모리에서 완전히 제거


Rqt Plugin 사용법 및 실습

#### PyQt의Signal-Slot 구조란?
- PyQt에서 Signal-Slot 구조는 GUI 이벤트(클릭, 입력 등)와 이에 대응하는 함수(동작)를 연결하는 이벤트 기반 프로그래밍 메커니즘
Qt에서 GUI 구성 요소가 어떤 일이 일어났음을 signal로 알려주고, 이에 대한 동작을 slot으로 연결

|용어|설명|
|:---|:---|
|Signal|특정이벤트발생시방출되는신호(예: 버튼클릭)|
|Slot|Signal을 받아처리하는함수(예: 함수실행)|
|connect()|Signal과 Slot을 연결하는함수|

### RQt
#### RQt
RQt는 ROS2 시스템을 GUI로 “보고”, “조작하고”, “디버깅＂하는데 필수적인 도구 모음
- 플러그인 형태로 다양한 도구 및 인터페이스를 구현할 수 있는 ROS의 GUI(Graphical User Interface) 프레임워크
- 토픽, 서비스, 액션같은 ROS2 통신을 시각적으로 보고 조작(디버깅, 모니터링, 개발)
- ROS + Qt의 합성어
- 여러 Plugin을 통해 다양한 기능 제공
- 크로스 플랫폼 지원

크로스 플랫폼의 장점
- 운영 체제에 구애 받지 않고 개발 가능
- 한번 개발하면 여러 OS에서 실행 가능(코드 수정 최소화)
- ROS2가 지원하는 다양한 환경에서 사용 가능

python_qt_binding
- Qt Python API 사용
- PyQt와 PySide는 C++ 기반 Qt 라이브러리를 Python에서 사용할 수 있도록 바인딩한 것이다.
- 바인딩(binding)이란 Python과 C++ 사이에서 데이터를 변환하고 호출할 수있도록 해주는기술
- Python으로 Qt API 사용 시 Qt C++ API 대신, Python으로 바인딩된 API 사용
- 대표적Qt Python API: PyQt, PySide
- Python_qt_binding 패키지의 장점
- PyQt와 PySide를 구분 없이사용가능
- 필요시두바인딩API 간전환가능
- RQt 플러그인패키지사용순서
1. rqt_gui_py.plugin 모듈의 Plugin 클래스 상속
2. qt_gui.plugin 모듈의 Plugin 클래스 상속
3. python_qt_binding.QtCore 모듈의 Qobject 클래스 상속



# 04장: ROS2 심화 기능2
### ros2 security
- Security는 SROS의 유틸리티로, DDS-Security를 ROS2에서 사용하기 위해 필요한 도구를 모아둔 것

- 환경변수 3가지
1. ROS_SECURITY_KEYSTORE: 보안설정 파일을 보관하는 폴더를 지정. demo_keystore
2. ROS_SECURITY_ENABLE: 보안 설정의 On/Off 기능으로 true/false 형태로 설정. Default는 false
3. ROS_SECURITY_STRATEGY: 보안 설정 방법. Enforce로 설정하면 보안 설정 파일이 없는 메시지 통신은 금지, Permissive의 경우 비보안 참여자로 참석시킴.

※ 위 환경변수 3가지는 노드를 실행할 때마다 매번 각 터미널에서 선언해야 함.
ROS2 보안 기능을 지속적으로 사용할 예정이라면 ~/.bashrc에 추가해야 함

### Transformations(좌표 변환)
### tf2
#### tf2: ROS2에서 여러 좌표 프레임 간의 변환과 관계를 추적하고 관리하는 라이브러리
- 로봇 시스템은 일반적으로 세계 프레임, 기본 프레임, 그리퍼 프레임, 헤드 프레임 등과 같이 시간에 따라 변경되는 많은 3D 좌표 프레임을 가짐
- tf2는 시간에 따른 이러한 모든 프레임을 추적하고 다음과 같은 질문을 할 수 있도록 함
  - Q1. 5초 전  world frame 대비 head frame은 어디에 있는가?
  - Q2. Gripper에 있는 물체의 포즈는 내 base에 비해 어떤가?
  - Q3. Map frame에서 base frame의 현재 포즈는 어떤가?

활용 분야
- 로봇 위치 추적 (Localization): map → odom → base_link 변환을 통해 로봇 위치 확인
- 센서 데이터 정렬 (Sensor Fusion): LiDAR, 카메라, IMU 데이터를 로봇 본체 좌표계로 변환
- 로봇 암(Arm) 조작: 다관절 로봇의 각 관절 간 변환을 사용하여 위치 계산
- 드론 및 이동 로봇: GPS 데이터 변환, IMU 데이터 보정 

### World Frame
- World Frame은 좌표계에서 가장 기본이 되는 기준 좌표계(Reference Frame)
- 절대좌표계(Fixed Coordinate System)
- 다른 모든 프레임(로봇의 위치, 센서 데이터 등)이 world frame을 기준으로 상대적 표현됨
- 로봇의 동작에 따라 여러 종류의 world frame이 존재(SLAM →map, 바퀴 기반 →odom, GPS →earth)
### World Frame의 종류
- Map: SLAM, Localization에서 사용되는 대표적인 world frame. 자율주행 로봇의 경로계획(Path Planning)
- Odom: 바퀴 기반 이동(Odometry)에서 사용되는 world frame. 시간이지나면서 누적 오차 발생(Drift). 단기적 위치 변화 추적에 유용
- World: Gazebo  같은 시뮬레이션에서 사용됨. Map과 비슷하지만 물리적 환경에서 존재하는것은 아님.
- Earth: GPS데이터를 사용할 때 적용되는 글로벌 좌표계. 위도/경도 데이터를 변환하여 로봇 위치를 추적. 자율주행 드론, 자동차 등
- Base_link: 로봇 본체 프리임
### 다른 프레임 간 관계
- Earth → Map: GPS 기반 위치 추적. 고정된 월드 프레임
- Map → Odom: Localization 을 통해 Odometry 보정
- Odom(주행거리계) → base_link: 로봇의 현재 위치

#### tf 기본 개념
  - Frame: 좌표계(예, map, odom, base_link, camera_link)
  - Transform :한  프레임에서 다른 프레임으로의 변환 관계(회전 + 이동)
  - TF Tree: 여러 프레임이 계층적으로 연결된 구조
#### tf 핵심 기능
  - TF 브로드캐스터: 프레임간의 변환을 주기적으로 브로드캐스트
  - TF 리스너:  다른 노드에서 TF정보를 구독하여 변환을 확인
  - TF  변환 적용: 특정 프레임 간 좌표 변환 수행
#### Rviz2
- RViz2에서 TF를 시각화하여 디버깅 가능
- 센서 데이터 정렬, 로봇 네이게이션, 로봇 암 조작등에 필수적
- Broadcaster와 Listener를 사용하여 변환 데이터를 송수신
#### [tf tree 예]
- map → odom 변환: SLAM이나 로컬라이제이션에서 로봇의 위치 제공
- odom→base_link: 변환: 로봇의 본체 중심 좌표
- base_link → camera_link, lidar_link 변환: 센서 위치

### URDF
- URDF(Universal Robot Description Format)는 ROS에서 로봇의 형상과 구성을 지정하기 위한 파일 형식
- 로봇의 링크(link), 조인트(joint), 센서, 액추에이터 등의 물리적 요소들을 정의하여 로봇의 3D 모델을 시뮬레이션하거나 실제 로봇 제어 시스템에서 사용할 수 있게 도와줌

#### 주요 구성 요소는 다음과 같음
- 링크(link): 로봇의 물리적인 부분을 나타내며, 고체 물체로 이해할 수 있음. 각 링크는 모양, 관성, 마찰 특성 등과 같은 속성들을 가짐.
- 조인트(joint): 두 링크를 연결하여 로봇이 움직일 수 있게 하는 연결점. 회전 운동이나 직선 운동을 정의할 수 있음
- 재질 및 텍스처: 로봇의 각 링크에 적용할 재질이나 텍스처를 정의하여 외형을 시뮬레이션에서 표현할 수 있음
- 센서 및 액추에이터: URDF 파일을 확장하여 센서나 액추에이터와 같은 로봇의 기능적 요소들을 정의할 수 있음


joint type
fixed:고정
continous: 계속 움직일 수 있음 -> 바퀴,  머리
- 머리의 경우 연속 조인트(continuous joint)로 모델링 됨
- 연속조인트의 경우 모든 각도를 취할 수 있으므로 상세한 제한은 기록하지 않음
prismatic: 프리즘형 -> 손목 같은 그리퍼 암
- 그리퍼 암의 경우 축을 따라 직선 운동을 하는 프리스메틱 조인트(prismatic joint)로 모델링 됨
- limit에 사용되는 단위로 revolute joint와 달리 미터 사용
revolute: 그리퍼
- 그리퍼의 경우 회전 조인트(revolute joint)로 모델링 됨(limit가 있음)
- 조인트의 토크(effort), 하한(lower), 상한(upper) 그리고 최대 속도(velocity) 등을 정의함

#### 움직일 수 있는 로봇 모델 만들기
- GUI 슬라이더: GUI 슬라이더는 다음과 같은 원리로 동작됨
- GUI가 URDF를 구문 분석하고 고정되지 않은 모든 조인트와 제한값(limit)을 찾음
- 슬라이더의 값을 사용하여 ‘sensor_msgs/msg/JointState’ 메시지를 발행
- 발행된 메시지는 robot_state_publisher 패키지에 의해 처리됨(각 관절값에 따라 로봇의 트랜스폼 계산)
- 변환 트리(transform tree)를 사용하여 Rviz에서 로봇의 모든 파트를 시각적으로 표시

|구분|Joint State Publisher|Robot state Publisher|
|:---|:---|:---|
|역할|로봇의 모든 관절(Joint) 상태(위치)를 Publishing|/joint_states를 받아서 링크들 사이의 좌표 변환(TF)를 계산해서 Publishing
|발행하는 토픽|각 Joint의  Position값을 담은 메시지를 생성하고 발행<br>/joint_states<br>Sensor_msgs/msg/JointState|/tf → 움직이는 변환 정보<br> /tf_static → 고정된 링크간 변환<br>Tf2_msgs/msg/TFMessage|
|구독하는 토픽||/joint_states|
|기타|Sensor_msgs/msg/JointState<br>Header: 시간, frame_id<br>Name: 조인트들의 이름<br>Position: 각 조인트의 현재 위치<br>Velocity:  각 조인트의 속도<br>Effort: 각 조인트에 걸리고 있는 힘(토크)|robot_state_publisher는 robot_description parameter(URDF)를 읽음link-joint tree 저장<br>/joint_states 토픽 구독(각 조인트의 현재 위치 값)<br>조인트 값을 기준으로 3D위치와 방향(TF)계산<br>각 링크에 대해 /tf토픽 발행|

joint_state_publisher →   topic   → robot_state_publisher → topic → RViz

              /joint_state 링크간 계산(tf) /tf+/tf_static 로봇모델 + 움직임 시각화

※만약 mobile robot처럼 base_link가 움직인다면?
→Robot_state_publisher는 base_link만 관리. Localization(map) 또는 Odometry(odom)가 TF를 publishing 함


#### 충돌 및 관성 속성 추가 -코드
- geometry: 링크의 충돌 영역의 기하학적 형태를 정의
  - cylender: 원기둥 형태 사용
- inertia: 관성 텐서를 정의
  - mass: 질량(kg)
  - ixx, iyy, izz: 각각 x, y, z축을 중심으로 하는 회전 저항
  - ixy, ixz, iyz: 관성 곱(product of inertia)으로, 링크가 주축 이외의 축에 대해 어떻게 회전하는지를 나타냄(대칭물체는 모두 0)
  - 관성 텐서는 링크의 회전 운동 방정식을 결정하는 데 사용됨.
  - 정확한 관성 값을 제공함으로써 시뮬레이션에서 링크의 회전 움직임이 현실적으로 표현 할 수 있음

### Xacro
- Xacro를 사용하여 URDF 코드를 단순화하기
- XACRO(XML Macros)는 URDF파일을 더 효율적으로 작성하도록 도와주는 XML 기반 매크로 언어
- ROS2에서 로봇 모델을 생성할 때 URDF를 직접 작성하는 대신,
- XACRO를 사용하면 재사용 가능한 코드 블록을 정의하고
- 이를 통해 코드의 중복을 줄이며 유지보수성을 높일 수 있음
- Xacro, URDF, SRDF 알아보기

|URDF|SRDF|XACRO|
|:---|:---|:---|
|Universal Robot Description Format|Semantic Robot Description Format|XML Macro|
|로봇의 물리적 구조(모델) 정의|로봇의 의미론적(운동학적) 정보 정의|URDF의 확장버전(코드 재사용성 증가)|
|링크, 조인트, 관성, 충돌 모델 등|운동학 그룹, 충돌 회피 설정, 엔드 이펙터 설정|URDF|
|Gazebo, Rviz, Moveit!|Moveit!|Gazebo, Rviz, Moveit!|
|로봇의 구조를 정의하는 기본 파일|URDF를 기반으로 Moveit!을 위한 의미론적 설정을 추가하는 파일|URDF를 생성하는 도구|
|URDF|URDF를 SRDF로 변환 불가능. 수작업|Xacro파일을 URDF로 변환해야 Gazebo, Rviz, Movtit!등에서 사용가능|
|URDF 시뮬레이션은 가능하지만 경로계획 수행불가|Moveit!같은 경로 계획 도구 사용하려면 SRDF필요|Xacro는 URDF를 생성하는 도구 실제 사용되는 로봇모델파일은 아님|

#### Using Xacro- Constants
- 이전 R2D2 예제에서는 다음과 같이 중복이 일어남(ex: cylinder의 길이와 반지름을 두 번 지정함). 이러한 경우 하나의 값을 변경하려면 다른 하나도 같이 변경해 주어야 함
- 따라서 왼쪽의 코드를 변경하여 상수 역할을 하는 속성을 지정한 오른쪽 코드와 같이 변경하는 것이 좋음
- 상수의 경우 처음 두 줄에 지정되며, 이 값은 거의 모든 곳에서 어떤 수준에서든 사용 전이나 후에 정의 가능
- 상수는 “${}”의중괄호 안에 상수를 집어넣는 방식으로 사용 가능
- Xacro에서 수식은 기본 사칙연산, 부호 반전(unary minus), 괄호(parenthesis)등을 사용하여 복잡한 표현식 작성 가능



#### Using Xacro- Macros
- 본 섹션에서는 Xacro매크로를다루는 방법에 대하여 알아볼 예정
- Xacro매크로는 ‘<xacro:macro>’ 태그를 사용하여 정의되며, 특정 기능을 수행하는 코드 블록으로, 필요할 때마다 매크로를 호출하여 해당 코드를 재사용이 가능하도록 하는 것
- 즉 매크로는 python의 함수와 비슷한 역할을 한다고 볼 수 있음
- 단순한 매크로의 예시: 
  1. 다음 제공된 것은 단순한 Xacro매크로로, default_origin이라는 이름을 가진 매크로를 정의한 것.
  2. ‘<xacro:default_origin />’를 호출 시 ‘<origin xyz="0 0 0" rpy="0 0 0"/>’가 해당 위치에 삽입 됨
- 더 나아가 매크로에 파라미터를 넣고자 할 경우 다음과 같은 방법 사용 가능
  1. 파라미터를 사용하고자 할 때 매크로 이름 뒤에 ‘params’를 추가 가능
  2. 이 경우<xacro:default_inertial mass="10"/>를 호출 시 해당 위치에 오른쪽 코드가 삽입됨 



# 05장: 시뮬레이션 개발
### 로봇시뮬레이션
컴퓨터 상 가상의 환경 이용해 로봇 동작, 센서 데이터, 환경을 개발하고 테스트하는 기술
### Gazebo
ROS2를 활용할 수 있는 로봇 시뮬레이션, 물리엔진과 3D 그래픽 지원
#### 장점
- 비용 절감:로봇 프로토타입 제작하거나 테스트하는 비용 절감
- 안전성: 사람이나 장비에 대한 사고 위험 경감
- 개발 효율: 여러 대의 로봇을 테스트하거나 개발하여 효율 증가
- 유연성: 날씨, 지형, 장애물 등 다양한 시나리오 실험
- ROS2와 통합하면 실제 로봇 하드웨어 없이도 다양한 로봇 시스템을 개발, 테스트, 디버깅 할 수 있는 강력할 시뮬레이션 환경 제공

- Gazebo의 주요 기능
  1. 물리 엔진 제공(ODE, Bullet, DART)
  2. 센서 시뮬레이션(카메라, 라이다 등)
  3. 3D환경 렌더링
  4. 로봇 모델 시뮬레이션
- ROS2와 Gazebo 통합 패키지
  - Gazebo_ros_pkgs
  
- SDF(Simulation Description Format)
  - 시뮬레이션  world와 로봇의 물리적 특성, 외형, 동작 등을 정의하는데 사용되는 표준 포맷
  - URDF는 로봇 자체를 중심으로 정의하는 반면, SDF는 world 전체, 센서, 플러그인, 조명등 더 많은 요소 포함
  - World, model, link, joint, sensor, plugin등의 주요 요소가 있음
  - Odom → base_link : 변환 : 로봇의 본체 중심 좌표
  - URDF는 Gazebo에서 사용할때 변환이 필요하지만 SDF는 직접 사용 가능

### SLAM
- Simultaneous Localization and Mapping의 약자. 동시적 위치 추적 및 지도 생성이라는 뜻
- 사용 기술
  - Localization: 로봇이 현재 위치를 추정하는 과정
  - Mapping: 주변 환경에 대해 지도를 생성하는 과정
  - Sensor Fusion: 라이다, 카메라, IMU 센서, GPS, 적외선, 초음파 센서 등 다양한 센서 데이터를 통합하여 정확도를 높이는 기술

- Simultaneous Localization: 로봇의 자세(position + orientation)를 획득하는 것
  - Mapping: 환경정보(지도)를 획득하는 것
  - 로봇의 자세(위치+ 방향)와 환경지도를 동시에 획득하는 기술
  - SLAM에서는 아무것도 주어지지 않음
  - 로봇의 자세· 지도 Landmark의 위치 모두 오차를 갖게 됨
  - 오차를 어떻게든 보정해가면서 로봇의 자세와 환경정보를 모두 정확하게 알아내는 것
- Localization
  - Localization에서는 환경정보(지도)가 주어져야 함
  - 주어진 지도 안에서 로봇이 ‘어디에 위치’하고 ‘어떤 방향’을 바라보고 있는지 알아내는 것
  - 지도로부터 획득된 센서 관측데이터를 이용하여 오차를 보정해가면서 로봇의 위치와 방향을 정확하게 알아내는 것
- Mapping
  - Mapping에서는 로봇의 자세(위치와 방향)이 주어져야 함
  - 주어진 로봇의 위치와 방향을 기반으로 ‘주변환경에 대한 정보’를 알아내는 것
  - 센서 관측 데이터와 주어진 로봇 자세를 이용하여 오차를 보정해가면서 환경 정보를 정확하게 알아내는 것

- Loop Closure
  - 로봇은 계속 움직이면서 오차가 누적됨
  - 어느 순간 예전에 왔던 장소로 다시 되돌아오게 되는데, 이때 그 장소를 알아보지 못하면 지도는 점점 더 오류가 커짐
  - 돌아온 장소를 인식하고 과거와 현재의 지도를 연결(정합)하는 작업
  - 이전의 이미지와 특징점을 매칭해 유사도 높으면 같은 지점이라고 판단하고 누적된 오차 보정해 정확도를 개선해야 함
- Kidnapped Robot Problem
  - 로봇이갑작스럽게다른위치로납치(이동)되었을때자신의위치를파악하지못하는상황
  - Why?
    - 물리적 이동(부딪힘이나 인위적인 이동)
    - 센서 오류
    - 껐다 켰을 때
    - 여러 공간이 비슷한 경우
    지도를 작성할 때는 kidnapped problem이 발생하지 않도록 주의를 기울여야 함
  - 좋은 초기 위치 추정 제공→Cartographer는 초기 위치 추정에 크게 의존
  - 센서 구성 최적화(센서 품질 및 다양한 센서 조합) →Sensor Fusion(LiDAR + IMU + Odom)
  - LIDAR scan match 품질 높이기
  - Loop closure 및 submap 파라미터 조정, Global localization Trigger
  - 위와 같은 방법으로 위치 유실 감지 및 회복 전략 구현

### Chicken and Egg Problem - SLAM은 어떻게 해결?
- Localization은 먼저 주변 환경에 대한 정보를 알아야 함, Mapping은 먼저 현재 로봇의 위치와 방향에 대한 정보를 알아야 함 → 추정과 반복을 사용해서 해결. 위치와 지도를 동시에 ＂조금씩” 추정하면서 점점 더 정확하게 만들어 감
- SLAM의 일반적인 해결 방식
  1. 초기화(Initialization): 로봇이 임의의 위치에서 시작. 센서 데이터(LiDAR, 카메라) 수집
  2. 동시 추정(Simultaneous Estimation):  현재 센서 데이터를 기반으로 자기 위치 추정 동시에 주변 지도 업데이트
  3. 루프 클로징(Loop Closing): 전에 방문한 위치를 다시 방문하면 과거의 지도와 현재의 지도를 정렬해 오류를 줄임
  4. 루프 클로징은 오류 누적을 정정하는 핵심 단계
  5. 최적화(Graph Optimization): 시간에 따라 쌓인 위치와 맵 데이터를 그래프 형대로 모델링하고 그래프 최적화를 통해 위치와 맵을 동시에 정제

### NAV
- Navigation은 SLAM을 활용하여 로봇을 원하는 위치까지 자동으로 도달하게 제어하는 것
- 사용 기술
  - Path Planning : 목표 지점까지 최적의 경로를 찾는 과정
  - Behavior Tree : 로봇의 동작을 트리 형태로 구성하여 유동적으로 관리하고 제어하는 알고리즘
  - Trajectory Tracking : 계획된 경로를 따라 로봇을 제어하는 기술

### SLAM & NAV
||SLAM(Simultaneous Localization & Mapping)|Nav2(Navigation)|
|:---|:---|:---|
|역할|맵을 생성하며 로봇의 위치를 추정|이미 존재하는 맵에서 경로를 계획하고 이동|
|입력|데이터센서 정보(LiDAR, IMU등)|맵, 로봇위치, 목표 위치|
|출력|결과2D맵, 로봇 위치(/map, /tf)|이동 경로, 속도 명령(/cmd_vel)|
|실행|시점맵이 없는 환경에서 최초 탐색에 사용|맵이 준비된 이후 목표 지점으로 이동할 때 사용|
|대표 패키지|slam_toolbox, cartographer, gmapping|nav2_bringup, bt_navigator, planner_server|

### SLAM : Visual
- 라이다에서거리데이터를통해3D 점구름, point cloud를 생성하는SLAM 이전프레
임과현재프레임에서얻은포인트클라우드데이터를비교하여, 로봇의상대적위치와
환경지도를동시에계산
SLAM for efficient Lidar Labeling|Sama
- 장점: 어두운환경에서정확한거리를측정
- 한계: 비용이높고외부노이즈에민감함

|항목|2D Lidar|3D Lidar|
|:---|:---|:---|
|스캔방식|단일 평면(수평 또는 수직)|수직 및 수평 방향 모두에서 거리 측정|
|동작방식|한 개의 레이저가 일정한 각도로 2D 평면을 스캔|여러 개의 레이저가 다양한 각도에서 공간을 스캔하여 3D데이터 생성|
|출력 데이터|거리 및 각도 정보를 포함한 2D포인트(2D 맵)|거리, 각도 고도 정도를 포함한 3D 포인트 클라우드(3D맵)|
|메시지 타입, 토픽|sensor_msgs/LaserScan, /scan|sensor_msgs/PointCloud2, /points or /velodyne_points|
|스캔범위|수m ~ 수십m(보통 360° 수평 스캔)|수m ~ 수백m(30°~120°수직 + 360°수평)|
|적용분야|실내 자율주행, SLAM, 충돌방지|실외 자율주행 차량 드론 등에서 정밀 공간 인식|
|적용 사례|실내, 저비용 로봇, 간단한 SLAM, 단순한 장애물 회피|복잡한 공간 인식 필요, 정밀한 3D맵, 구조물 인식|

Light Detection and Ranging : 레이저 빛을 쏴서 물체에 반사되어 돌아오는 시간(Time of Flight)을 측정해서 거리(depth)를 계산하는 센서

#### LiDAR SLAM의  종류
|특징|GMapping(OpenSLAM)|Cartographer(Google)|
|:---|:---|:---|
|호환성|주로 ROS1|ROS1 & ROS2 모두 호환|
|맵 환경|2D, static한 상황에 유리|2D & 3D|
|정확도|센서나 Odometry 오류에 민감함|오류에 대해 비교적 강건하고 정확함|
|로컬라이제이션|대략적인 위치 추정더 정확한 위치 추정|
|적용 분야/환경|저사양 하드웨어/단순한 실내 환경|고정밀 맵이 필요한 복잡한 환경 / 정밀 지도
|개발자||Google이 개발한 실시간 SLAM|
|성능|기본적인 환경에서 충분|복잡하고 넓은 환경에서도 우수|

### 오도메트리(Odometry)
- 로봇이 센서 데이터를 기반으로 자신의 이동 거리와 방향을 추정하는 기술
- 주로 바퀴의 회전 정도(Wheel Encoder)를 기준으로 측정(바퀴가 회전한 거리)
- 왼쪽 바퀴와 오른쪽 바퀴의 이동거리를 비교 →직선, 회전, 곡선인지 계산
- 바퀴의 미끄러짐이나 로봇의 회전 오차(Drift) 등으로 인해 누적 오차가 발생할 수 있음
- GPS는 실내에서 불가. IMU의 경우 자세는 잘 측정하지만 오차가 빠르게 누적
- 단독 사용 어려움. 복잡한 환경에서는 IMU, SLAM과 보완 필요. Kalman Filter는 비선형 상태추정 알고리즘(확률적으로 정확한 상태 추정)
- Odometry는 로봇 바퀴 회전량, IMU등을 이용해 자기 위치 추정 → Kalman Filter로 더 정밀하고 신뢰도 높은 상태(속도, 위치 등)를 추정

※ Kalman Filter
1. Prediction(예측 단계) : 이전상태로 부터 현재 상태를 예측 (속도가 이 정도니까 지금쯤 이 위치에 있을 것 같다)
2. Correction(업데이트 단계) : 실제 센서로 측정된 값을 바탕으로 예측값 보정 (GPS값과는 약간 다르니 중간쯤으로 값을 조정)

Step0. 센서 초기화: 수행하고자 하는 과제와 환경에 따라 라이다 센서의 파라미터 초기화
[라이다 센서의 주요 파라미터]
- Angle parameters(각도)
  - Angular Resolution : 한 스캔 회전에서 연속된 두 포인트 간의 각도 차이(작을 수록 더 정밀)
  - Field of View(FOV) : 시야각. LiDAR가 커버할 수 있는 전체 회전 각도(270°FOV는 뒤쪽 90°는 못 봄)
  - Vertical Angle : 수직각도. 3D LiDAR의 경우 여러 개의 수직 레이저 빔이 있고 수직각도 범위도 중요함
- Time parameters(시간)
  - Scam Rate : 1초에 몇 번 전체 스캔을 수행하는지? 10Hz →10회/초당
  - Timestamp : 각 포인트나 스캔이 언제 수집되었는지 기록. 움직이는 로봇의 위치보정을 위해 동기화 필요
  - Time of Flight : 빛이 물체에 반사되어 돌아오는데 걸린 시간. 이 값으로 거리를 계산
- Range parameters(거리)
  - Minimum Range, Maximum Range : 감지 가능한 최소 및 최대거리(ex, 0.3m ~ 100m, ±2cm)
  - Accuracy / Precision : 거리 측정의 오차. 정밀도가 높을 수록 안정적인 맵 생성
  - Noise, Signal Strength : 먼 거리나 어두운 물체에서 측정 신호가 약할 수 있어 노이즈가 커질 수 있음

Step1. 데이터 수집
- 라이다센서로distance 정보수집
- 2D, 3D 포인트 클라우드 형태로 들어옴.
- Topic으로 퍼블리시되며 SLAM Node가 구독함

Step2. 특징점 추출
SLAM : LiDAR
- 수집한 정보(Point Cloud)에 대해 의미 있는 특징점추출(코너, 평면, 경계선 등)
  - 전체점을 다 쓰면 계산량이 너무 많고, 모든 점이 의미 있는 정보를 담고 있지는 않음
  - 그러므로 의미 있는 지형적 정보가 담긴 점들만 선택하는 전략

- 특징점을찾는방법: 곡률분석(Curvature)
  - Edge Features : 모서리, 경계선, 급격한 고도 변화가 있는 부분. 즉, 곡률이 높은 점. 평면에 해당하는 점은 특징점에서 제외
  - Planar Features : 평평한 바닥, 벽 등 주변보다 곡률이 낮은 점
$$C = \frac{\|p_i - Mean(p)\|}{n}$$
곡률 𝐶는 다음과 같이 𝑝𝑖(현재위치)에 대한 주변점들의 평균값의 차이로 계산
즉, 특정점이 주변점들에 비해 구분되는 값을 가진다면 곡률 𝐶는 커짐

Step3. 특징점 매칭
- Point Cloud Registration과 ICP를 이용하여 특징점 매칭
  - 현재 프레임의 특징점과 이전 프레임에 있는 특징점 계산(2개 스캔간 위치 차이를 보정)
  - 즉, 서로 다른 시점에서 얻은 포인트 클라우드 데이터 간의 공통적인 구조(특징)을 찾아서 상대위치를 계산하는 과정
  - ICP, NDT등 매칭 알고리즘 사용
  - 이 과정을 통해 로봇의 현재 위치(Pose)를 추정하고 전체 지도를 
  - 점점 정밀하게 업데이트 수행

ICP란?(Iterative Closest Points)
- 두 개의 포인트 클라우드 간의 위치 및 자세를 맞추기 위해 반복적으로 가장 가까운 점끼리 매칭한 뒤 변환 행렬을 계산해 정렬하는 알고리즘 
- 라이다가 이동하며 수집한 어떤 Point Cloud를 다른 Point Cloud의 공간으로 변환하는 과정
$$P1= Q×P2+ r → P2를 P1으로 변환(Q는 회전 행렬, r은 이동 행렬)$$
- 이때 Point to Point ICP의 경우 점과 점 사이의 유클리드 거리를 최소화하는 방향으로 환 행렬을 찾는것으로 가장 기본적인 알고리즘
- 한계: 두포인트클라우드가충분히가깝지않거나점의수가많으면계산비용이커질수있음

Step4. 지도 업데이트
- 포인트 클라우드 데이터를 시간순으로 누적하여 점진적으로 지도를 작성<br>
추정된 위치를 기준으로 새로운 센서 데이터를 기존 맵에 통합, 이때 지도의 신뢰도를 높이기 위해 Occupancy Grid Mapping 알고리즘을 사용
- Occupancy Grid Mapping
  - 로봇이 주변 환경을 2D 격자 지도(Grid Map)형태로 표현할 때 사용하는 방법
  - 지도를 다수의 cell 단위(2차원 배열)로 쪼개서 각 셀의 점유 확률을 계산
  - 점유 확률은 해당 cell을 장애물이 점유하고 있을 확률을 뜻하며
  - 센서 데이터와 로봇의 위치에 따라 결정됨
  - 맵이 다 완성되면 cell에 지정된 확률에 따라 Occupied/Free로 결정됨

Occupancy Grid Mapping의 3가지 기본 가정
1. Cell을 구성하는 영역은Occupied이거나Free 둘 중하나의상태만을 가짐(binary variable)
2. Cell은 정적이며 시간이지나도변하지않음
3. 각셀은독립적으로처리됨

### 센서융합 SLAM
- 카메라와 라이다 그리고 다른 센서들의 데이터를 융합하여 상호보완적으로 SLAM을 수행
- 특히 딥러닝 모델로 여러 데이터를 직접 결합하여 지도를 작성하는 End-to-end 모델에 사용
- 필요한 이유
  1. 정확도 향상: 센서 단독 정보보다 융합정보가 신뢰도 높음
  2. 강건성 : 하나의 센서가 오류발생해도 다른 센서로 보완 가능
  3. 다양한 정보 수집 : 위치, 방향, 거리, 속도, 형태 등 복합 정보 처리
- 장점: 조명 변화, 외부 환경에 더 강건한 시스템
- 한계: 데이터 처리량 증가로 실시간 구현 문제(각 센서들을 어떻게 동기화 시킬 것인가?)
- 대표적인 알고리즘 : Kalman Filter, Particle Filter, Graph-based Optimization, Deep Learning기반 Fusion
[전통적인 Sensor Fusion 방식]
1. 각각의 센서 전처리
2. 각각의 센서 데이터 특징 추출
3. Feature Fusion(특징 결합)
4. 위치 추정 / 지도 작성(SLAM)

|센서|역할|
|:---|:---|
|LiDAR|고정밀 거리 측정, 장애물 인식|
|Camera|차선, 신호등, 객체 분류|
|IMU|차량 자세, 회전 정보|
|GPS|전역 위치|

### Groot 
  - Behavior Tree를 설계 편집 시각화하는 도구 
  - Groot 자체는 행동 트리를 실행하는 엔진이 아니고, 트리를 만들고 모니터링하는 편집/디버깅 도구

  Groot를 쓰는 이유
  - Behavior Tree 구조를 직관적으로 시각화
  - 코드 작성 없이 행동 흐름을 설계
  - 실행 중인 트리 상태를 실시간으로 이해·분석
  - 복잡한 로봇 행동을 모듈화·재사용 가능하게 설계

  Behavior Tree
  - 자율 로봇이나 게임 AI에서 의사결정과 행동 흐름을 계층적 트리 구조로 표현하는 기법.<br>
  트리의 각 노드는 조건 검사, 액션 실행, 제어 흐름(시퀀스, 선택 등)을 담당하고, 루트에서부터 자식 노드로 내려가며 실행 결과(success/failure/running)를 판단

### Navigation –Nav2의 주요서버
#### Nav2는 ROS2기반의 자율네비게이션모듈
   - 모듈형 소프트웨어 프레임워크
   - 자율주행(Autonomous navigation)을 가능하게 해줌
- 주요 기능 3가지
  - 로봇의 현재 위치를 파악(Localization)
  - 목표 위치까지 갈 수 있는 경로를 계획(Path Planning)
  - 그 경로를 따라 로봇을 실제로 움직임(Path Execution / Control)
- 주요 구성 요소
  - LifeCycle Manager(수명 주기 관리)
    - Nav2의 모든노드의상태를제어
    - Behavior Tree의 노드를 초기화하고 관리
  - Behavior Tree
    - 전체 navigation 행동을 조율하는 노드
    - Nav2의 핵심제어로직이구현된부분(목표 수신 → 경로 계획 → 이동)
    - 논리 흐름을 트리 형태로 구성하고 Navigation server와 통신하며 여러 작업을 제어함
  - Navigation Servers
    - Recovery server : 경로 재설정이나 비상 상황에서 로봇을 복구함
    - Planner server : 지도 정보를 기반으로 경로를 생성
    - Controller server : 로봇이 경로를 따라 주행하도록 실시간 제어
    - Smoother server : 경로의 급격한 변화에 대해 로봇의움직임을최적화
  - 기타 노드
    - amcl : Adaptive Monte Carlo Localization(로봇 위치 추정)
    - waypoint_follower : 여러 지점을 따라가는 방식의 경로 지원

AMCL은 확률 기반 입자 필터를 사용하여, 로봇이 지도 위에서 자신의 위치를 
실시간으로 추정하는 ROS 표준 위치 추정 알고리즘

Nav2의주요작업을수행하는4가지서버
- Planner Server : Dijkstra나 A*(경로 탐색 알고리즘)등을 사용하여 로봇 위치에서 목표 지점까지의최적경로계산
- Controller Server : DWA 알고리즘을 사용하여 경로를 따라가기 위한 local planning을 구함(cmd_vel 생성)
- Smoother Server : Planner가 생성한 경로를 입력으로 받아, 로봇 주변 환경 정보를 나타내는 costmap을 기반으로 경로를 
더 부드럽게 변경
- Recovery Server : 네비게이션 실패 시 clear costmap이나 회전, 후진 등의 복구 동작을 실행

경로 생성 → 경로 보정 → 경로 추종 → 장애 회복
Plan → Controller → Smoother → Recovery

#### Dijkstra(다익스트라) → 전역 경로계획, planner server
- 최단 경로 탐색 알고리즘(Planner Server에서 실행)
- 인공위성 GPS 소프트웨어 등에서 가장 많이 사용됨
- 특정한 하나의 정점에서 다른 모든 정점으로 가는 최단 경로를 알려주는 탐색 알고리즘

#### Planner Server : A*
A*(A-star)는 출발 꼭지점에서 목표 꼭지점까지의경로를찾기위해고안된알고리즘으로, 각 꼭짓점에 대한 평가 함수 𝑓(𝑥) 는 다음과 같음:
$$𝑓(𝑥) = 𝑔(𝑥) + ℎ(𝑥)$$
- 𝑔(𝑥) : 출발점에서 현재점(x)까지의 거리
- ℎ(𝑥): 현재점(x)에서 목적지까지의 예상 거리<br>
A*는경로를생성시𝑓(𝑥) 가최소화되는방향으로경로를생성한다.<br>
즉 목적지 방향으로의 탐색을 우선시하며, 그로 인해모든경로를탐색하는Dijkstra에비해빠른성능을보인다.

#### Costmap(Smooter server)
- Map에비용을표시하여robot이이동할때참고할수있게구성한것
- 고비용: 장애물과가까움
- 저비용: 장애물없음, 자유롭게움직일수있는영역

#### DWA(Dynamic Window Approach) → 로컬 경로 추종/장애물 회피, controller server
- 로봇의 동역학 제약(속도, 가속도)을 고려하여 로컬에서
최적의 속도 명령을 선택하는 알고리즘(Controller Server에서 실행)
- 로봇이 실시간으로 충돌없이 주변 장애물을 피하면서 목적지를 향해
안정적으로 이동할 수 있도록 선속도, 각속도조합을 매 순간 계산하는 방식

- DWA는 2D 평면상의 로봇을 제어하여 실시간으로 장애물을 회피하고 목적지에 도달하는 알고리즘.
로봇이 가질수 있는 최대의 선속도(v)와 최대의 각속도(w)를 이용하여 다음 time step까지 이동할 수 있는 영역을 만들고 이 영역을 dynamic window라고 함<br>
이 영역 내에서
  1. 로봇의 방향을 목표 방향과 최대로 일치시키고
  2. 장애물과 부딪히지 않고
  3. 최대의 속도로 운행할 수 있게 로봇을 제어

[핵심 개념]
1. 속도 샘플링: 로봇의 선, 각속도에서 가능한 후보 샘플링(Dynamic Window)
2. 모션 시뮬레이션 : 각 후보 속도 조합에 대해 시뮬레이션(어디로 움직이는지 예측)
3. 평가 함수 : 각 시뮬레이션 궤적을 평가해서 점수화
4. 최적 궤적 선택 : 가장 높은 점수를 받은 궤적의 선속도(v), 각속도(w)를 로봇에 적용

### Visual SLAM의  종류
|특징|ORB<br>(Oriented FAST and Rotated BRIEF)|RTAB<br>(RealTime Appearance-Based Mapping)|
|:---|:---|:---|
|사용 카메라|주로 단안(모노) 카메라 또는 Stereo Camera|RGB-D카메라 혹은 양안(스테레오) 카메라|
|깊이 감지|깊이 감지 범위가 짧음|깊이 정보를 더 잘 감지함|
|병렬 처리|작업을 병렬적으로 처리|단계별 수행으로 상대적으로 처리시간이 오래 걸림|
|맵 생성과 최적화|중요한 위치(키프레임)를 선택, 맵을 만들고 수정|더 밀도 있는 3D 매핑을 수행하여 정확도를 높임|
|정확도|야외 환경에서 정확도 감소|야외 환경에서도 강건함|
|기타|정밀한 위치 추정과 맵핑이 필요한 경우|실시간 처리와 대규모 환경에서의 확장성이 중요한 경우|
|활용사례|연구 개발, 정밀 네비게이션|실시간 로봇 내비게이션, 3D맵핑|
|ROS통합|가능하지만 복잡할 수 있음|매우 용이(ROS2 package제공)하고 주로 많이 사용함|

RGB-D Camera: 깊이(Depth)와 컬러(RGB) 데이터를 모두 실시간으로 받아들이는 카메라


# 06장: ROS2와 OpenCV



# 07장: ROS2와 3차원 영상 및 시각화
- Open3D
  - 컴퓨터 그래픽과 3D 데이터 처리 작업을 위한 Python 라이브러리
  - 오른손 좌표계 사용
  - 이미지 numpy 배열 혹은 PointCloud 형식으로 표현
  - 3D 점들의 집합(포인트 클라우드)이나 3D 모델링 계산하고 모양과 위치 분석하는 작업 수행
- 3D 데이터 처리
  - 인간이 물체를 '보는' 방식과 이를 '이해하고 분석하는' 과정을 모방함
  - 센서나 스캐너를 통해 3D 포인트 클라우드 데이터를 받아서 전처리/분석/시각화를 수행함
  - 자율 주행, 로봇 공학, AR/VR 등 다양한 분야에서 활용됨
- PointCloud
  - LiDAR센서, RGB-D센서 등으로 부터 수집되는 데이터
  - 물체에 빛/신호를 보내서 돌아오는 시간을 기록하여 각 빛/신호당 거리 정보 계산하고 하나의 점(Point)를 생성 
  - 3차원 공간상에 퍼져 있는 여러 포인트(Point)의 집합(Cloud)을 의미. (x, y, z)의 3차원 정보
  - 2D 이미지와는 다르게 깊이(Z축)정보를 가지고 있으며 Nx3의  numpy 배열로 표현.각 n줄은 하나의 점과 Mapping
- Voxel
  - 'Volume'과 'Pixel'의 합성어로, 3차원 공간에서의 '픽셀'을 의미
  - 2D에서의 픽셀이 점으로 이미지를 표현한다면, 3D에서는 voxel이 작은 정육면체로 3D 물체를 표현
  - Voxel의 크기는 필요에 따라 조절할 수 있음

### PointCloud 데이터를 다루는 3D 인공지능의 발전(다양한 딥러닝 모델) 
1. PointNet
   - 데이터 변환 없이 Pointcloud 데이터를 그대로 입력해서 학습하는 모델. Standford 대학에서 발표
   - Classification, Semantic Segmentation 수행 가능
2. Voxelnet
   - Pointcloud 데이터로부터 Voxel Feature를 추출 후 이를 해석해서 물체를 검출하는  3D Object Detection 모델
   - 3차원 공간을 Voxel 단위로 나눈 후 Voxel 안의 점들을 Voxel Feature Encoding Layer라는 딥러닝 네트워크를 거쳐 Feature Map 얻어냄
1. PointPillars
   - Pillars라는 Point Cloud Encoder를 사용해 Pointcloud로부터 격자 형태의 Feature Map을 얻고 해석해 물체를 검출하는 3D Object Detection모델
   - Pointcloud 데이터를 특정 시점에서 투영시킨 다음 2D  격자 단위의 Feature Map을 얻어낸 것이 특징
1. Dynamic Graph CNN(DGCNN)
   - 가장 가까운 이웃을 기반으로 동적으로 그래프를 구성하여 Pointcloud데이터에서 특징을 추출
   - 이 동적 그래프 구조를 통해 CNN과 유사한 연산을 수행

### 로봇 공학 및 자율주행에서의 좌표계(Coordinate System)
1. World 좌표계
   - 작업 환경 전체의 기준이 되는 전역(절대) 좌표계
   - 로봇, 물체, 센서 등 모든 요소들의 절대적인 위치를 표현하는 기준. 작업 공간의 특정 위치에 고정
2. Base 좌표계
   - 로봇 자체의 기준 좌표계. 로봇의 바닥이나 첫번째 관절에 위치
3. Joint 좌표계
   - 각 관절(Joint)마다 정의된 로컬 좌표계. 관절에서의 움직임에 따라 변함
   - Forward Kinematics, Inverse Kinematics 계산이 중요
4. Tool 좌표계
   - 로봇팔 끝단(end effector, tool)에 정의된 좌표계
   - 일반적으로 Gripper, Drill, 용접기 등의 말단 장치 기준
5. Map 좌표계
   - 로봇이 만들어낸 또는 주어진 2D/3D 맴 기준 좌표계. 주로  SLAM, Navigation에서 사용
6. Odometry 좌표계
   - 로봇의 출발 시점을 원점으로 하는 좌표계. 시간이 지남에 따라 오차가 누적됨. 바퀴 회전 등을 통해 추정된 위치를 기준으로 한 로컬 좌표계
7. 기타 좌표계 : Sensor좌표계(LiDAR, IMU등의 센서 고유 좌표계), Camera 좌표계(카메라의 렌즈 중심을 원점으로 하여 Z축이 보는 방향) 등

### 로봇 모션
1. MoveJ
   - 로봇의 각 관절이 현재 각도에서 목표 각도로 동시에 이동 후 동시에 멈춤
   - 목표 관절 각도를 입력[Joint1, Joint2, … Joint6]
2. MoveL
   - 로봇 TCP를 직선을 유지하며 목표점까지 이동
   - 목표 위치 및 회전 값을 입력: [X, Y, Z, Rx, Ry, Rz]
3. MovePeriodic
   - 일정한 진폭과 주기로 왕복 이동

TCP: Tool Center Point의 약자로 로봇에 장착된 도구의 위치와 방향을 카르테시안 좌표계로 표현함

|타입|MoveJ|MoveL|
|:---|:---|:---|
|제어 방식|로봇의 모든 관절이 현재 각도에서 목표 각도로 동시에 이동 후 동시에 멈춤|로봇 끝단의 TCP가 선택한 좌표계에 대해 선형 모션(Linear motion)으로 이동|
|장점|이동 속도가 빠름<br>로봇 특이점(Singularity)의 영향을 받지 않음|TCP 이동 경로 직선 유지하므로 경로 미리 인지 가능<br>목표 위치(X, Y, Z, A, B, C) 통해 끝단 위치 예측 용이|
|단점|모든 축이 동시 회전하니 이동 경로 예측불가<br>각도로 표기하니 로봇 끝단 위치 및 자세 예측 어려움|MoveJ에 비해 상대적으로 모션 속도가 느림<br>로봇 특이점(Singularity)의 영향을 받음|
|용도/동작|특이점 회피 시 사용<br>장애물이 없는 환경에서 원거리 이동시 적합|물체 회피 및 정밀한 미세 이동에 적합<br>직선 궤적이 필요한 작업(용접, 실링 등)에 사용|

### 특이점(Singularity)
  작업공간의 제한이나 구조적 한계로로봇을 제어할 수 없는 상태를 의미함
   - 다관절 로봇에서 특이점(Singularity)란 간단하게 설명하면 로봇이 이동 중 자신의 다음 자세를 계산하기 어려워하는 위치(또는 점)
   - 다관절 로봇의 경우 로봇의 끝단을 기준으로 이동하는 동안의 각 관절의 각도를 계산
   - 예를 들면 아래 그림 1의 상태에서 로봇이 빨간 점으로 이동하고자 할 때, 로봇은 그림 2처럼 다음 자세를 A 자세가 되도록 각 관절을 움직여야 하는 건지 B 자세로 움직여야 하는 건지 판단을 할 수가 없는 상태가 되며 이 위치(또는 점)이 특이점

### 로봇 제어 기초 개념
1. ACC(가속도)
로봇 관절의 속도 변화량을 제어하는 파라미터<br>
가속도가 너무 크면 로봇이 정지 상태에서 급격하게 자세를 바꾸기 때문에 위험할 수 있음
2. VEL(속도)
로봇 관절의 속도를 제어하는 파라미터<br>
속도가 너무 크면 동작 중 더 위험한 안전사고를 발생시킬 수 있음
3. Sync Operation(동기 구동)
실행 중인 모션 명령어가 완전히 끝난 후 다음 명령어로 이동
4. Async Operation(비동기 구동)
명령을 실행하자마자 바로 다음 명령 실행(예, 로봇 arm 이동 중 그리퍼 동작)
5. Radius(반경)
반경(mm)을 설정하여 도착점에 도달하기 전 반경 구간에서는 비동기 구동을 활성화
6. Blending Mode(블랜딩 모드)
반경(Radius)을 적용했을 때 선행 모션을 유지하여 중첩 실행할 것인지(Duplicate) 선행 모션을 무시하고 덮어씌울 것인지(Override)**를 선택하는 옵션 연속된 움직임을 보다 부드럽고 자연스럽게 수행. 각 지점에서 정지하지 않고 꺾이지 않게 부드럽게 이어지는 경로 생성

### Sync vs Async
|Sync|Async|
|:---|:---|
|한 번에 하나의 모션 명령어만 수행하는 것으로 기본값으로 설정되어 있음 제어의 예측 가능성이 높음|한 번에 여러 개의 모션 명령어를 수행하는 것으로 모션이 부드럽게 연결됨 동작을 빠르게 수행하여 작업 효율의 증대 But 제어 로직이 복잡해질 수 있음|

### Blending Option : Duplicate vs Override
|Duplicate|Override|
|:---|:---|
|- TCP가 반경에 진입했을 때<br>- 선행 모션을 유지하면서 후행 모션을 실행<br>- 작업 연속성을 고려한 복잡한 동작 구현에 적합<br>- 정확도가 중요한 연속경로(용접, 그리기)<br>- 각 점을 거의 찍으면서 부드럽게 곡선으로 이동<br>|- TCP가 반경에 진입했을 때 <br>- 선행 모션을 덮어쓰는 식으로 후행 모션을 실행<br>- 즉각적인 작업전환으로 비상상황 대처에 적합<br>- 속도가 중요한 연속작업(Palletizing, Pick & Place)<br>- 미리 방향을 틀어서 이동|

### 로봇 외력 (External Force)
로봇이 환경과 상호작용하거나 외부에서 힘이 가해질 때 로봇에 작용하는 힘이나 토크

1. Compliance Control (순응 제어)
외력에 순응하며 로봇의 움직임을 조절(용수철원리) 외력이 사라지면 로봇이 원래의 목표 위치로 복귀
강성(stiffness)에 따라 부드럽게 반응하는 정도를 조절할 수 있음
주로 충돌 방지, 부드러운 이동, 그리고 안전성이 필요한 환경에서 활용
1. Force Control (힘 제어)
로봇이 외력에 대해 평형을 이루도록 힘을 제어 로봇의 자세가 특이점 영역에 들어가면 힘 제어가 불가능
할 수 있으므로 자세 변경 필요

#### 강성이란?
외력에 대해 얼마나 부드럽게 반응할 것인가를 결정하는 파라미터. 강성이 작을수록 로봇이 더 부드럽게 작동하고 원상태로 복귀하는 시간이 길어짐.<br>
$K = F/X$ (K는 강성, F는 외력, X는 이동거리, K는 스프링 상수의 역할)



# 08장: ROS2 프로그래밍 실습 핵심정리
|ros2cli + [verbs]|[sub-verbs]|기능|
|:---|:---|:---|
|ros2 pkg|create<br>executables<br>list<br>prefix<br>xml|새로운 ROS2 패키지 생성<br>지정 패키지의 실행 파일 목록 출력<br>사용 가능한 패키지 목록 출력<br>지정 패키지의 저장 위치 출력<br>지정 패키지의 패키지 정보 파일(xml) 출력|
|ros2 node|info<br>list|실행 중인 노드 중 지정한 노드의 정보 출력<br>실행 중인 모든 노드의 목록 출력|
|ros2 topic|bw<br>delay<br>echo<br>find<br>hz<br>info<br>list<br>pub<br>type|지정 토픽의 대역폭 측정<br>지정 토픽의 지연시간 측정<br>지정 토픽의 데이터 출력<br>지정 타입을 사용하는 토픽 이름 출력<br>지정 토픽의 주기 측정<br>지정 토픽의 정보 출력<br>사용 가능한 토픽 목록 출력<br>지정 토픽의 토픽 발행<br>지정 토픽의 토픽 타입|

|ros2cli + [verbs]|[sub-verbs]|기능|
|:---|:---|:---|
|**ros2 service**|call|지정한 서비스에 서비스 요청(Request)을 전달하고 응답을 받음|
||find|특정 서비스 타입을 사용하는 서비스들의 이름을 출력|
||list|현재 활성화되어 있는 사용 가능한 서비스 목록 출력|
||type|지정한 서비스의 데이터 타입(Interface)을 출력|
|**ros2 action**|info|지정한 액션의 클라이언트 및 서버 정보 출력|
||list|현재 활성화되어 있는 사용 가능한 액션 목록 출력|
||send_goal|지정한 액션 서버에 액션 목표(Goal)를 전송하고 결과 피드백을 받음|
|**ros2 interface**|list|시스템에서 사용 가능한 모든 인터페이스(msg, srv, action) 목록 출력|
||package|특정 패키지 내에서 정의되어 사용 가능한 인터페이스 목록 출력|
||packages|인터페이스를 포함하고 있는 모든 패키지들의 목록 출력|
||proto|지정한 인터페이스 타입의 기본 프로토타입(구조 예시)을 출력|
||show|지정한 인터페이스의 구체적인 데이터 필드 형태와 구조를 출력|

|ros2cli+[verbs]|[sub-verbs]|기능|
|:---|:---|:---|
|**ros2 param**|`delete`|실행 중인 노드의 특정 파라미터를 삭제|
||`describe`|특정 파라미터의 타입 및 제약 조건 등의 상세 정보 출력|
||`dump`|현재 노드의 모든 파라미터 설정을 YAML 파일로 저장|
||`get`|특정 파라미터의 현재 값을 읽어옴|
||`list`|현재 활성화된 노드들의 사용 가능한 파라미터 목록 출력|
||`set`|특정 파라미터의 값을 새로운 값으로 변경(쓰기)|
|**ros2 bag**|`info`|녹화된 rosbag 파일의 메타데이터(시간, 토픽 수, 메시지 양 등) 정보 출력|
||`play`|녹화된 rosbag 데이터를 다시 재생하여 토픽을 발행|
||`record`|현재 발행되고 있는 토픽 데이터를 rosbag 파일로 기록(저장)|

|대분류 (ros2cli)+[verbs]|[sub-verbs] (options)|기능|
|:---|:---|:---|
|**ros2 extensions**|(-a)<br>(-v)|ros2cli의 extension 목록 출력<br>(ros2cli개발용으로 사용, 일반적 사용 x)|
|**ros2 extension_points**|(-a)<br>(-v)|ros2cli의 extension point 목록 출력<br>(ros2cli개발용으로 사용, 일반적 사용 x)|
|**ros2 daemon**|`start`|daemon 시작|
||`status`|daemon 상태 보기|
||`stop`|daemon 정지|
|**ros2 multicast**|`receive`|multicast 수신|
||`send`|multicast 전송|

|대분류 (ros2cli)+[verbs]|[sub-verbs](options)|기능|
|:---|:---|:---|
|**ros2 doctor**<br>**ros2 wtf**|`hello`|ROS 설정 및 네트워크, 패키지 버전, rmw 미들웨어 등과 같은 잠재적 문제를 확인하는 도구<br><br>doctor와 동일함 (ros2 doctor의 alias / WTF: Where's The Fire)|
||(-r)||
||(-rf)||
||(-iw)||
|**ros2 lifecycle**<br>(별도 Chapter에 설명)|`get`||라이프사이클 정보 출력|
||`list`|지정 노드의 사용 가능한 상태 전이 목록 출력|
||`nodes`|라이프사이클을 사용하는 노드 목록 출력|
||`set`|라이프사이클 상태 전환 트리거|

### ROS arguments
- ROS arguments는 주로 run 또는 launch 명령어와 같은 ROS2 실행 명령어와 함께 사용되며 “--ros-args” 옵션을 통해 지정
- 많이 사용되는 ROS arguments
- -r __ns:=사용할 네임스페이스
- -r __node:=변경할 노드 이름
- -r 본래의 토픽/서비스/액션명:=변경할이름
- -p 파라미터 이름:=변경할 파라미터 값
- --params-file 파라미터 파일

||DDS (Data Distribution Service)|QoS (Quality of Service)|
|:---|:---|:---|
|**개념**|- ROS2 기본 통신 미들웨어로 중앙 서버 없이 분산 네트워크에서 데이터를 교환<br>- DDS는 ROS2의 기본 통신 프로토콜|- DDS통신의 품질을 조정하는 설정으로, 메시지 신뢰성, 내구성, 저장 개수 등을 관리<br>- DDS를 통해 통신 품질을 조정하는 방식<br>- QoS설정에 따라 DDS의 동작 방식이 달라지며 각 애플리케이션에 맞는 성능을 최적화<br>- 네트워크 환경과 응용 프로그램의 요구사항에 따라 적절한 QoS를 설정해야 함|
|**특징**|실시간 데이터 교환을 위한 미들웨어 표준, ROS2의 통신기반이 되는 핵심 기술|- QoS 주요 설정 6종류<br>Reliability(신뢰성), Durability(내구성), History(데이터 저장 개수), Lifespan(수명), Deadline(데드라인), Liveliness(활성 상태)|
|**기타**|- Publisher-subscriber 모델<br>- Brokerless(중앙 서버 없음)<br>- 자동 발견(Discovery) : 네트워크상의 노드들이 서로를 자동으로 인식하고 연결<br>- 신뢰성 & 확장성 : 다양한 QoS 설정을 통해 신뢰성과 성능을 조절|- BEST_EFFORT + VOLITILE : 카메라 영상 스트리밍(실시간성 우선, 일부 프레임 손실 가능)<br>- RELIABLE + TRANSIENT_LOCAL : 로봇 센서 데이터(정확한 수신 보장, 최신 데이터 유지)<br>- KEEP_LAST(10) + LIFESPAN(5s) : 5초 동안 최신 10개의 데이터를 유지하는 센서 데이터|

|QoS 설정 옵션|상세 설명|세부 항목 / 설정값|
|:---|:---|:---|
|**Reliability**|UDP처럼 통신 속도를 최우선 할지, TCP처럼 데이터 손실을 방지하며 신뢰도를 우선할지 설정|- `BEST_EFFORT` (속도 우선)<br>- `RELIABLE` (신뢰도 우선)|
|**History**|통신 상태에 따라 정해진 사이즈 만큼의 데이터를 보관|- `KEEP_LAST`<br>- `KEEP_ALL`|
|**Durability**|데이터를 수신하는 Subscriber가 생성되기 전의 데이터를 사용할지 폐기할지에 대한 설정|- `TRANSIENT_LOCAL`<br>- `VOLATILE` (휘발성)|
|**Deadline**|정해진 주기 안에 데이터가 발신 및 수신되지 않을 경우 이벤트 함수를 실행시킴|`deadline_duration` (단위: ms)<br>(예: 700, 1000 등)|
|**Lifespan**|정해진 주기 안에서 수신되는 데이터만 유효판정하고 그렇지 않은 데이터는 삭제|`lifespan_duration` (단위: ms)<br>(예: 700, 1000 등)|
|**Liveliness**|정해진 주기 안에서 노드 혹은 토픽의 생사를 확인|`Liveliness`<br>(`AUTOMATIC`, `MANUAL_BY_TOPIC`)|


PyQt의 Signal-Slot 구조란?
PyQt에서 Signal-Slot 구조는 GUI 이벤트(클릭, 입력 등)와 이에 대응하는 함수(동작)를 
연결하는 이벤트 기반 프로그래밍 메커니즘입니다.
Qt에서 GUI 구성 요소가 어떤 일이 일어났음을 signal로 알려주고, 이에 대한 동작을 
slot으로 연결합니다

|용어|설명|
|:---|:---|
|**Signal**|특정 이벤트 발생 시 방출되는 신호 (예: 버튼 클릭)|
|**Slot**|Signal을 받아 처리하는 함수 (예: 함수 실행)|
|**connect()**|Signal과 Slot을 연결하는 함수|

| 위젯 | 설명 / 용도 | Signal 예시 |
| :--- | :--- | :--- |
| **QPushButton** | 클릭할 수 있는 일반적인 버튼 위젯 | `clicked`, `pressed`, `released` |
| **QLineEdit** | 한 줄의 텍스트를 입력받는 창 | `textChanged`, `returnPressed` |
| **QSlider** | 슬라이더 바를 움직여 숫자를 조절하는 위젯 | `valueChanged` |
| **QComboBox** | 드롭다운 목록에서 항목을 선택하는 위젯 | `currentIndexChanged`, `activated` |

| 패키지 이름 | 설명 |
| :--- | :--- |
| **rqt_gui** | RQT의 기본 GUI 프레임워크 |
| **rqt_console** | ROS 로그 메시지 출력 및 디버깅 도구 |
| **rqt_graph** | ROS 노드, 토픽, 서비스 연결 구조 시각화 |
| **rqt_plot** | ROS 토픽 데이터를 그래프로 표시 |
| **rqt_reconfigure** | ROS 파라메터를 실시간으로 조정 |
| **rqt_bag** | ROS bag 파일 기록 및 재생 GUI |

### URDF
- URDF(Universal Robot Description Format)는 ROS에서 로봇의 형상과 구성을 지정하기 위한 파일 형식
- 로봇의 링크(link), 조인트(joint), 센서, 액추에이터 등의 물리적 요소들을 정의하여 로봇의 3D 모델을 시뮬레이션하거나 
실제 로봇 제어 시스템에서 사용할 수 있게 도와줌
- 주요 구성 요소는 다음과 같음
- 링크(link) : 로봇의 물리적인 부분을 나타내며, 고체 물체로 이해할 수 있음. 각 링크는 모양, 관성, 
마찰 특성 등과 같은 속성들을 가짐.
- 조인트(joint) : 두 링크를 연결하여 로봇이 움직일 수 있게 하는 연결점. 회전 운동이나 직선 운동을 
정의할 수 있음
- 재질 및 텍스처 : 로봇의 각 링크에 적용할 재질이나 텍스처를 정의하여 외형을 시뮬레이션에서 
표현할 수 있음
- 센서 및 액추에이터 : URDF 파일을 확장하여 센서나 액추에이터와 같은 로봇의 기능적 요소들을 
정의할 수 있음
