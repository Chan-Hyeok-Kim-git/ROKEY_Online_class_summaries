# 01장: OpenCV 기반 이미지 처리 기초
신호의 종류와 디지털화
아날로그 신호:연속적, 자연; 디지털 신호: 불연속, 아날로그를 디지털화
디지털화 하려면: 샘플링, 양자화



# 02장: 기하학적 변환
필터: 각 픽셀 주변 값 고려해 수학적 연산 통해 새로운 픽셀값 계산하는 과정

필터 종류
스무딩(블러링): 평균(커널 내 픽셀값 평균값으로 대체(노이즈 감소))☆; cv2.blur() 
가우시안(중심에 가까울수록 가중치 두는 가우시안(정규) 분포 활용);    cv2.GaussianBlur()
중간값(픽셀들 정렬해 중앙값 선택(소금, 후추 노이즈 제거 탁월)),      cv2.medianBlur()
양방향(거리가 가깝고 색상 차이가 적은 픽셀만 평균(엣지 보존))        cv2.bilateralFilter()

모서리 탐지: 소벨(1차 미분 기반, 수평/ 수직 방향의 엣지 검출)☆;    cv2.Sobel()
라플라시안(2차 미분 기반, 방향 구분 없이 전체적인 엣지 강조);       cv2.Laplacian()
캐니(노이즈 제거, 그래디언트, 히스테리시스 임계값 단계적 검출, 흑백만 받음)☆☆;  cv2.Canny()
침식(객체 외곽을 깎아냄(작은 노이즈 제거, 영역 분리))               cv2.erode()

모폴로지(형태학): 팽창(객체 외곽을 확장(구멍 메우기, 영역 연결));   cv2.dilate()
열기(침식 후 팽창(밝은 노이즈 제거, 독립된 개체 분리));             morphologyEx(OPEN)
닫기(팽창 후 침식(객체 내부 검은 구멍 매움));                      morphologyEx(CLOSE)

영상의 기하학적 변환: 이미지 픽셀 좌표 행렬연산 통해 출력 이미지 새로운 좌표로 옮기기
회전: center(중심점)를 기준으로 angle만큼 회전시키고 필요하면 scale 조정(cv2.getRotationMatrix2D)
$$[x'] = [cosΘ -sinΘ]  [x]$$
$$[y'] = [sinΘ  cosΘ]  [y]$$

확대와 축소(scaling): 영상의 좌표 일정 비율로 늘리거나 줄임
$$[x'] = [Sx  0]  [x]     Sx>1: 가로 확대$$
$$[y'] = [0  Sy]  [y]     0<Sx<1: 가로 축소$$

스케일링과 보간: 크기 변경시 빈곳 채우거나 픽셀 값 합치는 보간(Interpolation) 과정 필수
Nearest 가까운 픽셀(계단현상)           매우 빠름
Linear  양산형 보간(기본값)             빠름
Cubic   3차원 스플라인(선명함, 확대용)   보통
Area    픽셀 영역 재샘플링(축소용)      빠름

Affine 변환: 평행선 유지 전략 ☆
직선은 직선으로, 평행선은 평행선 유지하는 변환(사다리꼴, 평행사변형)
직선 → 직선, 평행 → 유지
cv2.getAffineTransform(점1, 점2) 3개의 점으로 변환 지정; 아핀 3점
[x'] = [a11  a12  Tx]  [x] a11, a12, a21, a22 선형 변환 계수
[y'] = [a21  a22  Ty]  [y] 6개의 자유도(Degree of Freedom) → 점 3개로 행렬 계산
Tx, Ty: 평행 이동       [1] → 고정된 값

Perspective 변환: 원근감 반영해 직선의 평행성이 깨지는 투시 변환
3차원 공간 물체 2차원 평면에 투영시 발생하는 원근 효과 반영
평행선 깨짐, Perspective 4점
cv2.getPerspectiveTransform(p1, p2)
[x'] = [h11 h12 h13]  [x]   정규화:x' = X/w', y' = Y/w'; w': 분모
[y'] = [h21 h22 h23]  [y]   8개의 자유도(Degree of Freedom) → 점 4개로 행렬 계산 가능
[w'] = [h31 h32 h33]  [1]   → h33이 고정된 값

2차원이니까 (x,y) 좌표값 나올테고, 점이 3개니까 2*3이 DOF: Affine
점 4개에 (x,y)니까 2*4가 DOF: Perspective

정리
필터링: 스무딩(평균, 가우시안, 중간값(이상치 있을때)), 엣지 검출(소벨, 라플라시안, 캐니), 모폴로지(침식/팽창, 열기, 닫기)
기하학적 변환: 좌표 매핑, 행렬 연산
회전 및 확대/축소: cv2.getRotationMatrix2D(), resize, 보간법(확대 CUBIC, Linear, 축소 Area)
아핀 변환: 직선은 직선, 평행선 평행하게 유지; cv2.getAffineTransform()(3개의 점 매핑 필요); 직사각형이 평행사변형으로 변환 가능
투시 변환(Perspective): 원근감 반영, 소실점에서 만나게됨; cv2.getPerspectiveTransform()(4개 점 매핑); 문서 스캔, 사다리꼴 직사각형으로 핌



# 03장: 영상의 특징 검출
차분(변화): 두 픽셀 값의 차이 계산해 변화 감지하는 연산, 변화를 수학적으로 말하면 차분
공간적 차분: 인접한 픽셀간의 차이 계산 → Edge(경계선 검출)
시간적 차분: 연속된 프레임 간 차이 계산 → 움직임(Motion) 검출

영상의 특징 검출
저수준 특징: 색상, 밝기 및 강도, 엣지, 코너, 블롭
중간 수준 특징: 윤곽선, 형태 기술(설명)자, 질감
고수준 특징: 지역적 특징, 객체 인식, 딥러닝 기반 특징

필터를 활용한 엣지 검출
엣지(Edge): 영상에서 밝기가 급격하게 변화하는 지점, 주로 객체 윤곽선이나 경계에 해당.
소벨 필터: 미분 연산 수행하면서 노이즈 영향 줄이기 위해 고안된 필터(근사화된 미분)
평활화 + 미분 결합(Smoothing + Differentiation)
캐니 엣지 검출 알고리즘: 노이즈 제거 → 그래디언트 계산 → 비최대 억제(NMS☆☆) → 이중임계값 → 엣지 트래킹
☆☆비최대 억제(Non-Max Suppression): 엣지 방향에서 로컬 최대값만 남기고 나머지는 제거해 엣지 얇게 만듦(강한놈 딱 하나 남기고 다 죽이기)
휴 변환(Hough Transform): 영성 내 엣지 점들 파라미터 공간으로 매핑해 누적된 값이 최대인 지점 찾아 도형 검출
휴 변환은 직선 끊겨있거나 노이즈 있어도 교차값 누적 값(Accumulator) 가장 큰 곳 찾아 강건 및 검출 성능 우수
파라미터 공간(p, θ): 점들이 곡선으로 변환되어 한 점(교차점, peak)에서 만남

영상의 객체 검출
엣지 기반 기하학적 검출: 윤곽선 정보 활용한 형태 추론
휴 변환 연결: 이미지 공간 → 파라미터 공간 변환; 수학적 누적 통한 존재 판정
검출 흐름: 엣지 추출 → 공간 변환 → 변수 탐색 → 객체 판정

HSV 쓰는 이유: 색상이 지정되어 밝기(조명)에 덜 민감함



# 04장: 추적 알고리즘
템플릿 매칭: 이미지 내 특정 패턴 탐색 기법
cv2.matchTemplate: 원본 이미지와 템플릿 이미지 간 매칭 정도 분석하여 반환 → 유사도 맵(Result Matrix)
cv2.minMaxLoc: 매칭 방법 제공 함수; cv2.TM_CCOEFF : 상관 계수(-1<CCOEFF<1); cv2.TM_CCOEFF_NORMED(웬만하면 이것만 쓰면 된다)
cv2.TM_SQDIFF (min_loc 사용함)

특징자 추출: 이미지 내에서 강하게 구분되는 독특한 부분, 모서리(corner), 점(blob), 특정 패턴 등 주변과 뚜렷하게 구별되는 지점
특징자 검출 하는 목적: 조명 변화, 회전, 크기 변화에도 잃어버리지 않고 안정적으로 찾을수있는 기준점 만들기 위해
sementic: 문맥
영상 입력 → 특징점 검출 → 기술자 생성

특징자 매칭 vs 템플릿 매칭
템플릿 매칭: 픽셀 기반; 환경 변화에 민감, 기하학적 변형에 취약, 빛의 밝기, 회전, 색상등에 민감
특징자 매칭: 특징점+디스크립터; 강인함; 특징점 추출 → 벡터화 → 매칭
특징자 매칭 옵션: SIFT(제일 옛날), BRIEF(ORB전, 특징점 주변 픽셀 비교), ORB(BRIEF 단점 개선, 제일 많이 씀, 오픈 소스), LOFTR(트랜스포머)

MeanShift 알고리즘
데이터 집합 밀도 분포를 이용해 가장 조밀한 부분을 찾아가는 알고리즘, 주로 색상 히스토그램 기반으로 객체 중심 반복적 추적
색상 모델 생성(추적할 객체 영역(ROI) 색상 히스토그램 계산) → 유사 영역 탐색(유사한 색상 분포 가진 영역 역투영) 
→ 중심 이동(픽셀값 무게 중심 계산 윈도우 중심 그곳 이동) → 반복 수행(수렴하거나 지정된 횟수에 도달할때까지 3번 과정 반복)

CamShift 알고리즘
MeanShift의 문제: 윈도우(탐색 영역)크기 방향 고정이라 객체가 카메라에서 멀어지거나 회전시 추적 실패
해결: Continouously Adaptive MeanShift(CamShift) 매 프레임마다 윈도우 크기 및 방향 스스로 갱신해 적응
색상 모델 생성 → 확률 맵 생성 → MeanShift(가변성 윈도우)

루카스-카나데 알고리즘:희소(Sparse) 특징점 추적 위한 핵심 원리
Optical Flow 추적: 벡터 필드가 아닌 주요hift 수행 → 윈도우 업데이트(윈도우 크기(너비, 높이), 회전각) → 반복 수행
MeanShift(고정된 윈도우 크기) vs CamS 특징점(corner 등) 위주로 계산해 추적
☆밝기 보존 가정: 물체 한 점 시간 지나 위치 변해도 그 점이 가진 밝기 값(intensity)는 변치 않는다고 가정
☆작은 이동 가정: 연속된 프레임에 물체 이동량 매우 작다고 가정.(테일러 급수 근사 가능 위해)
☆국소 영역 가정: 어떤 픽셀 이웃 픽셀들은 서로 비슷한 움직임(Motion) 가진다고 가정.
I(x, y, t) = I(x+dx, y+dy, t+dt)
광류 방정식: IxU+IyV+It = 0; Ix = x방향으로 갔을때 밝기 변화, Iy= y방향 갔을때 밝기 변화, It=시간에 따른 밝기 변화, U,V= 속도

Dense Optical Flow(밀집 광류): 모든 픽셀의 움직임 추적
모든 픽셀 추적(특징점 뿐 만이 아닌 모든 픽셀 추적)
흐름 벡터 계산(각 픽셀(x,y)마다 x축 속도와 y축 속도 포함하는 2차원 벡터 필드 생성해 미세한 움직임까지 표현)
장면 전체의 움직임 파악(배경 객체 포함 전체적인 장면 구조 움직임 이해할 수 있어 동작인식, 영상 분할, 자율주행 등 활용)

Sparse vs Dense: 주요 특징점만 vs 모든 픽셀 추적
        Shift           vs          Optical Flow: 
탐지    HSV(색)                     Intensity(빛의 강도)
기준  색이 모여있나?                 특징점
문제 비슷한 색 만나면 추적 실패        문제 해결

RAFT(Recurrent All-Pairs Field Transforms)
딥러닝 기반 최신 기법: 수학적 최적화가 아닌 (CNN+RNN) 사용해 Optical Flow 추정
All-Pairs Correlations(모든 쌍 상관관계): 두 이미지 프레임 모든 픽셀 쌍의 유사도 계산해 4D Correlation Volume 생성
Recurrent Refinement(반복적 정제): GRU(Gated Reccurent Unit)기반 모듈이 Flow 필드 반복적으로 업데이트 해 점진적으로 정확도 높임

요약
템플릿 매칭: 간단하고 직관적인 구현; 크기, 회전, 밝기 변화에 매우 취약
MeanShift: 빠른 속도, 색상 분포 기반 추적; 객체의 크기 변화 및 회전에 적응 불가
CamShift: 객체의 크기와 회전에 유연하게 적용; 배경과 색상이 비슷하면 실패(색상 의존)
Lucas-Kanade: 특징점 기반의 정밀한 움직임 추적; 큰 움직임이나 빠른 이동에 취약
Dense Flow: 배경을 포함한 장면 전체의 움직임 파악; 모든 픽셀 연산으로 인해 계산 비용이 높음
RAFT: 현존 최고 수준의 정확도와 강건함; 높은 하드웨어 성능(GPU) 필수



# 05장: CNN 기반 분류 앱
CNN 특징: 가중치 공유, 자동 특징 추출, 계층적 학습, 시공간 상관관계 탐색
CNN 아키텍쳐: 합성곱 층(피처맵 만듬), 풀링 층(차원 축소하면서 가장 중요한 정보 유지), 완전 연결 층(평탄화 후 단일 벡터로 변환)
Softmax: 모든 출력값 0~1사이, 합이 1.0
LeNet: 근본, 합성곱 최초 도입
AlexNet: LeNet 보완, MaxPool 도입, ImageNet 기반 성능 검증
VGG & Inception 13개 레이어(합성곱 10개, 완전 연결 3개), 파라미터 1억 3천 8백만개로 많음, 깊이에 집중
ResNet: Skip-Connection 도입, 잔차 학습
DensNet: 모든 계층 서로 연결
데이터를 평탄화(flatten)함. 배치 차원(-1)을 제외한 모든 차원을 하나의 벡터로 만듦.
view() 에서 -1 은 batch_size 자동 계산
self.flattened_features(x) h*w (이미지) * channel(1: 흑백, 3: 컬러)
>> batch dimension 유지, 나머지 차원 (h*w*c) >> 1차원 벡터로 변환
Before flatten → batch_size = 32, x.shape = [32, 16, 5, 5] c=16, h=5, w=5
>> flatten → x.view(-1, 16*5*5) = (-1, 400) >> x.shape = [32, 400]☆☆
계산하는 법 기억하기 배치사이즈 유지, 나머지 곱하기

☆Region Proposal(RP): Object가 있을 만한 후보 영역(bounding box)를 추출하는 알고리즘(Selective search,느려 터짐)
→ 색, 무늬, 크기, 형태에 따라 유사한 region을 grouping(병합), 유사도 겹치는 것들끼리 묶는다

☆IOU(Intersection over Union): 모델이 예측한 결과와 Ground Truth가 얼마나 겹치는지 나타내는 지표
→ 일반적으로 PASCAL VOC Challenge에서는 0.5 이상이면 예측 성공으로 판단.
Left_up
[(x - 0.5 w) , (y - 0.5 h)] [(x’ - 0.5 w’) , (y’ - 0.5 h’)] >> 더 큰 값 선택
right_down
[(x + 0.5 w) , (y + 0.5 h)] [(x’ + 0.5 w’) , (y’ + 0.5 h’)] >> 더 작은 값 선택

gIOU(generallized IOU): 두 box간 겹치는 곳이 없을 때, IOU는 0이 되므로 얼마나 떨어졌는지 알기 어려워지기 때문에 이를 보완하기 위한 지표

NMS(Non-Maximum Suppression, 비최대억제): Object Detection 알고리즘은 object가 있을만한 위치에 많은 Detection 수행하는 경걍이 강함
→ NMS는 Detect된 object의 bounding box 중에 비슷한 위치에 있는 box 제거하고 가장 적합한 box 선택하는 기법
1. 특정 confidence threshold 이하 bounding box 먼저 제거(confidence < 0.5)
2. 가장 높은 confidence score를 가진 box 순으로 내림차순 정렬 후, 아래 로직을 모든 box에 순차적으로 적용
→ 높은 confidence score를 가진 box와 겹치는 다른 box를 모두 조사해 IOU가 특정 threshold 이상인 box 모두 제거(IOU threshold > 0.4)
3. 남아 있는 box만 선택
(Confidence가 높을수록, IOU threshold가 낮을수록 많은 box가 제거됨)
4. confidence 0.5 미만인 박스 제거 → 2. confidence 점수 높은 애들 내림차순 정렬 → 3. IOU threshold 0.4 이상인 박스 제거

mAP(mean Average Precision): 평가 지표
실제 object가 detect된 재현율(recall)의 변화에 따른 정밀도(Precision)를 평균낸 성능 지표
Precision: 예측을 Positive로 한 대상 중 예측과 실제 값이 Positive로 일치한 데이터 비율
(실제 암환자 1명을 정말로 암환자로 맞췄냐, 아닌 사람 암환자라 했으면 Precision 떨어짐.
암환자가 여러명이 동시에 있어도 그 중 1명만 맞혀도 틀린 사람을 지목하진 않았으니 Precision은 100%. Precision = TP / TP + FP)
Recall: 실제 값이 Positive인 대상 중에 예측과 실제 값이 positive로 일치한 데이터 비율
(암환자가 2명 있는데 둘 다 맞췄냐, 1명만 맞췄으면 Recall 50. Recall = TP / TP + FN)
Precision-Recall Curve: Confidence 값에 따른 Recall 값에 대한 Precision 값을 나타낸 곡선
Confidence score가 낮을 수록 더 많은 bounding box를 만들게 되어 Precision은 낮아지고 Recall은 높아짐
Confidence score가 높을 수록 신중하게 예측 bounding box를 만들게 되어 Precision은 높아지고 Recall은 낮아짐

Pascal VOC, MS COCO(데이터 셋들)



# 06장: 신경망 스타일 전이(Neural Style Transfer)
스타일 전이: 이미지 C를 만들기 위해 이미지 B의 스타일을 이미지 A 로 옮기는 것
cnn → 학습을 통해 모델의 가중치 값 변화
스타일 전이 → 학습을 통해 x의 픽셀값 변화(모델의 가중치 변화X)

손실 함수 계산 원리: 콘텐츠 손실(Content Loss)과 스타일 손실(Style Loss) 추출
콘텐츠 손실(얼마나 사람 얼굴을 똑같이 가져오지 못했나?) + 스타일 손실(얼마나 지브리스럽지 않았나?)
이미지 A(콘텐츠) + 이미지 B(스타일) = 이미지 C(스타일 전이 결과)
Ltotal(P,a,x) = αLcontent(P,x), βLstyle(a, x)
x: 원본 이미지; a: 스타일 이미지; P: 콘텐츠 이미지; α, β: 가중치
x: 생성할 이미지; N: 채널 개수, M: h*w(위치 정보)

콘텐츠 손실 함수
Lcontent(p̅, x̅, l) = 1/2 Σ_{i,j}(Flij - Plij)^2
x̅=생성될 이미지, p̅=원본(콘텐츠) 이미지
(생성 이미지 레이어 l(i,j좌표)특징값 - 콘텐츠 이미지 레이어 l(i, j )좌표 특징값)의 제곱
1/2 = 미분해서 내리니까 0되어서 미리 세팅해둔 계수

스타일 손실: 질감, 색상, 패턴 등 시각적 표현 특징 유지
Lstyle(a, x) = Σ{l∈L}(w_l E_l)          ; E_l = 1/{4N_l^2 M_l^2} ∑{i,j} (G_{ij}^l - A_{ij}^l)^2
정규화 안하면 해상도 뭉개짐

G_{ij} = ∑ F_{ik}·F_{jk}
F=Features(특성맵)
i, j: 채널(특징)들 간의 관계를 본다.
∑: 모든 위치의 값을 더해 위치 정보를 삭제한다.
결과: 이미지 전체의 질감, 색상, 패턴의 통계적 특징(스타일)만 추출된다.

Gram Matrix: 특징 벡터들 간의 내적(dot product)을 통해 생성된 상관관계 행렬(Correlation matrix)
Gram Matrix의 의미: 값이 큰 위치 = 이미지의 어떤 시각적 특징들이 얼마나 자주 함께 나타나는가?
해상도에 영향을 많이 받음

손실을 통해 학습하는 것: 픽셀 그 자체

Fl: 레이어 l의 특징맵
Nl: 필터(채널)의 개수
Ml: 공간적 크기(h*w)
Fl ∈ R_{Nl*Ml}
Gl_{i, j} = Σ_{k}(Fl_{ik}Fl_{jk})
                i번째 필터, k번째 위치값
                j번째 필터, k번째 위치값

컨텐츠는 conv 층을 저수준에서 고수준으로 가면서 위치 정보, 형태, 구조, 그대로 갖고있다 → 위치 정보 얼마나 유지했냐에 Loss 값 나옴
스타일은 색상 패턴 질감을 유지하는게 목표(위치 정보는 무시) → 질감 패턴 색상 얼마나 유지했냐에 Loss 결과 값 나옴
Feature map 끼리 비교해서 상관 관계 분석 → Gram Matrix
레이어 많아지면 채널이 많이 나옴 (공간 정보 손실되니 액기스 뽑기 위해 채널을 늘림)
정규화 시키는 이유: 해상도에 상관 없이 스타일만 보려고, 손실 값의 해상도 차이에 따른 균형을 맞추기 위해
학습 대상은 입력 데이터 X → 층 지나면서 계속 업데이트 됨(계속 덧칠 함)
생성중 이미지(x_bar)의 특징맵을 뽑아내는 CNN 모델이 있고,
학습할 때에는 이 모델을 타고 내려오면서 gradient를 계산하는데 CNN(모델)의 가중치는 업데이트 안하고, 가장 아래의 픽셀만 업데이트.

역전파를 통해 가중치 업데이트 되는거는 픽셀 그자체임(모델의 파라미터가 아님)
VGG가 하는 일: 특징 추출, 평가 지표(저수준에서 고수준 가중치를 다 가지고 있기 때문에 기준이 됨)
특징만 추출하기 때문에 분류층 필요없음(날려버림 vgg19.features로 Features 층만 가져옴 classifier 안가져옴)



# 07장: GAN & VAE
기존 AI(Discriminative): 분류, 검출(Localization(어디에있나))
이미지를 보고 판단하는것에 집중

생성형 AI(Generation): 없는 고양이 그리기, 합성(Synthesis): 사람 얼굴을 만화처럼
판단하지 않고 창조하는것에 집중

GAN(Generative Adversarial Network): 두 개의 신경망이 서로 경쟁(Adversarial)하며 학습하여 진짜같은 데이터 생성
잠재 공간(Latent Space): 데이터가 만들어졌을 가능성이 있는 공간
Generator-Discriminator, Encoder-Decoder

z: 노이즈(생성될 이미지)
G(z): 생성자(D가 틀리도록 유도(Maximize Error))
D(G(z)): 판별자(안틀리도록 노력(Minimize Error))
이 두 수치가 균형을 이루도록(Nash Equilibrium)

                              진짜(Real) 이미지 → 
입력 노이즈 벡터(z) → 생성기 → 생성된(Fake) 이미지 → 판별기 → 진짜 가짜 판별
                   생성기 손실(역전파)           판별기 손실(역전파)

LD = -[(log D(x) 진짜를 진짜라고 함) + (log(1-D(G(z))) 가짜를 가짜라고 함)]
LG = -logD(G(Z)) : 판별자를 속이지 못한 로스
min{G} max{D} V(D, G) = {E}{x ~ p_{data}(x)}[\log D(x)] + {E}_{z ~ p_z(z)}[\log(1 - D(G(z)))]
V(D, G)값이 커질수록 판별자가 잘 구분하고 있다는 뜻

DCGAN 생성기: 업샘플링
생성기: 64차원 랜덤 노이즈 → 16*16*128 특징맵 변환 → 업샘플링 통해 32*32 → 업샘플링 통해 64*64 → 마지막에 2차원 합성곱 계층을 거쳐 64*64 RGB 이미지 생성
각 단계마다 3*3커널에 128 채널 사용해 해상도 높임
핵심: DCGAN은 노이즈로부터 점진적으로 해상도 높이며 현실적인 이미지를 생성함

DCGAN 판별기: 다운샘플링
판별기: 64*64 RGB 이미지 받음 → 128채널, 특징맵 크기 32*32 -> 16*16 → 256채널, 특징맵 크기 16*16 -> 8*8 → 512 채널, 8*8 -> 4*4 → 전결합층 연결 후 시그모이드 함수 통해 이미지가 가짜인지 진짜인지 판별
핵심: 판별기는 합성곱과 다운샘플링 반복해 입력 이미지 특징 추출, 최종적으로 진짜 가짜 확률 출력하는 이진 분류기 역할 수행

채널을 늘리고 크기를 줄이는 이유: 정보 손실 최소화

모델별 특성
GAN의 다양한 변형은 손실 함수, 최적화 전략, 네트워크 구조에 따라 다름
SRGAN: 저해상도 이미지 고해상도로 변환(초해상도)
CycleGAN: 두 개의 생성기 사용해 이미지간 스타일 변환 수행
LSGAN: 일반적인 크로스엔트로피 손실 대신 평균 제곱 오차(MSE)를 사용해 학습 안정성 향상

Pix2Pix GAN: 단순한 이미지 생성 아닌 이미지→이미지 변환 작업에 특화
스타일 전이, 색 변환, 스케치→사진 변환 등 Layer 간 mapping 학습 수행

Pix2Pix 생성기
기본 구조: Encoder-Decoder 형태 (U-net 기반)
Encoder(다운샘플링): 1. 8단계 2차원 합성곱 계층으로 구성 → 2. 각 계층: 합성곱 → 정규화 → LeakyReLU 활성화 → 입력 이미지 점진적으로 축소하며 특징 추출
Decoder(업샘플링): 1. 업샘플링 + 합성곱 계층으로 구성 → 2. 각 단계에서 skip connection으로 Encoder 출력과 연결(concat) → 3. 활성화 함수는 ReLU인데 마지막에 tanh 사용
출력: 1. 입력 이미지와 동일 크기의 변환 이미지(256*256*3) 생성 → 이미지 간 변환(예: 흑백→컬러, 스케치→사진 등)에 활용
핵심: Encoder로 압축된 특징 Decoder로 복원해 skip connection을 통해 세밀한 구조 유지하는 U-Net형 생성기

Pix2Pix 판별기
구조: PatchGAN 기반 CNN 판별기
입력: 실제 이미지와 생성된 이미지 나란히 결합(concat)하여 입력
합성곱 계층: 여러 층 conv+BatchNorm+LeakyReLU 구성, 입력 이미지 70*70 크기 작은 패치 단위로 구분해 판별
출력: 각 패치별로 진짜/가짜 확률 예측
핵심: PatchGAN은 이미지 전체가 아닌 국소 패치 단위 진위 판단 수행해 세밀한 질감 효과적으로 학습시킴

VAE(Variational AutoEncoder): Encoder-Decoder구조(U-net과 동일)
이미지를 잠재 공간(Latent Space)의 확률 분포로 변환하여 새로운 데이터를 생성하는 모델
이미지 입력 → 인코더(사진 보고 상세한 설명서(특징) 작성) → (z~N(μ(평균), σ(분산))=샘플링)잠재 벡터에서 샘플링 → 디코더(설명서만 보고 이미지 재구성) → 재구성된 이미지
VAE 개념: 입력 데이터를 잠재 공간에 확률적으로 인코딩 후, 다시 복원하는 생성 모델인데, 단순한 자동인코더와 달리 확률 분포(μ(평균), σ(분산))를 학습해 새로운 데이터 샘플을 생성 할 수 있음
                        ε(사람이 조절 가능(라면스프 조절하듯이 0~1까지 넣어서 μ으로 만들수도 있고 μ+σ로 만들수도 있음))
z = μ(평균) + σ(분산) * ε(ε ~ N(0,1)) → 하는 이유: 선형성이 생겨서(미분 가능해져서) 학습이 가능해짐
ε ~ N(0,1)의 의미: 평균 0, 표준 편차 1인 곳에서 랜덤하게 한 값을 뽑았다(그런데 0~1사이 값임)
loss = Reconstruction + KL Divergence
Reconstruction Loss: 원본 이미지와 얼마나 비슷한가(입력과 복원된 출력 픽셀간 차이 최소화(MSE 등))
KL Divergence(확률분포 P와 Q 사이의 거리): 잠재 공간이 정규분포를 따르는가(잠재 변수 분포를 표준정규분포 N(0,1)에 근사시켜 생성 용이성 확보)

구현
1.데이터셋 준비 → 2. 모델 구조 정의(Encoder, Decoder, reparameterize 함수 구현) → 3. Loss 함수 설정 → 4. 학습 루프 → 5. 이미지 생성

베이지안 정리: P(A|B) = P(B|A)P(A) / P(B)
사건 B가 일어났을때, 사건 A가 일어날 확률 = 사건 A가 일어났다고 가정할 때, 사건 B가 일어날 확률 * 사건 A가 일어날 확률 / 사건 B가 일어날 확률



# 08장: 이미지 캡셔닝
이미지 캡셔닝: 시각적 정보 이해와 언어적 표현 능력이 융합된 멀티모달(Multi-modal) AI의 대표적 기술
기술 정의: 입력된 이미지 내용 분석해 상황 가장 잘 설명하는 자연어 문장 자동 생성하는 AI 기술
생성 예시:"검은색 털을 가진 강아지가 나무 마루 위에 얌전히 앉아 있다."와 같이 구체적인 문장 출력
CV+NLP 융합: 컴퓨터 비전(보는 능력)과 NLP(말하는 능력의) 결합, 시각 정보와 언어 표현이 동시에 필요
활용 분야: 시각장애인 보조 기술, 사진 관리 및 스마트 검색, 전자상거래 업무 자동화, 의료 및 산업 현장 분석

CNN, LSTM
이미지 → CNN 인코더 → 임베딩(특징 벡터) → LSTM(토큰<start>) → LSTM(토큰; 단어) → ... → LSTM(토큰<end>)
RNN → Sequential(이전 단어 기반 예측)
LSTM 단점: 길어지면 길어질수록 weight가 죽는다(확률값이라 0~1사이라 은닉층 통과할수록 죽음(시그모이드와 동일)) → 장기의존성 문제
Teacher Forcing(교사 강요): LSTM이 단어 생성 틀렸을때 정답 강제로 입력해서 패턴 학습 유도(학습 때만 사용!)
벡터화(임베딩): CNN 마지막 FC 계층 직전의 출력 사용해 이미지를 '고정된 길이의 숫자 배열'로 변환.
장기 기억: 긴 문맥 정보 유지(context vector) → 문맥 파악
LSTM 디코더: CNN 인코더의 '마지막 계층에서 출력된 특징 벡터'가 LSTM의 초기 입력으로 전달되어 이미지의 전체적 시각적 내용을 모델에 주입
모델의 학습 과정: 입력 → 모델 예측 → 비교 → Loss 계산(Cross Entropy 손실) → 업데이트(가중치 수정(역전파))

추론 전략: Greedy vs Beam search
Greedy search: 현재 시점에서 확률이 가장 높은 단어 1개만을 선택해 다음 단계로 진행(단기 이익 추구)
장점: 속도와 효율성(계산 비용이 매우 적고 추론 속도 빠름, 메모리 사용량 최소)
단점: 국소 최적해(당장은 최선이나 전체 문장 관점에서는 어색할 수 있고, 한 번 틀리면 되돌릴 수 없음.)
Beam search: 매 시점 상위 k개의 경로 유지하며 탐색, 최종적으로 누적 확률 제일 높은 문장 선택
장점: 품질 향상(전체적인 문맥 고려해 더 자연스럽고 정확한 캡션 생성, 국소 최적해 위험 줄어듬)
단점: 계산 비용(k배 만큼의 메모리와 연산시간 더 필요(속도와 품질의 트레이드 오프))
Tip: Beam width(k)는 3~5가 적절

Attention: 어디를 보고 말하나(모델이 단어 생성할 때 마다 이미지 관련 영역에 집중)
인간의 시각 인식 모방: "강아지가 공을 문다"라고 말하면 공이라고 말하는 순간 시선이 공으로 향함
동적 가중치(LSTM의 매 시점(Time Step)마다 이미지 각 픽셀 영역에 대해 중요도 다르게 계산)
성능 및 정확도 향상(모든 정보 하나의 벡터에 압축하던 기본 방식(임베딩)의 정보 손실 문제 해결)
설명 가능성(모델이 캡션 생성시 이미지 어느 부분 보고 판단했는지 시각화(Heatmap) 해 분석 가능)
캡션 생성 과정: 이미지 인코딩 → 상태 초기화 → 단어 생성 루프 → 문장 완성

성능 평가 지표(Metrics)
BLEU: 생성된 문장과 정답 문장 간 n-gram 정밀도 측정, 기계 번역 분야 표준 지표
연속된 단어(n-gram)가 얼마나 일치하는지 확인
짧은 문장에 패널티 부여(Brevity Penalty)
단순 매칭이라 동의어나 문맥 고려 부족
METEOR: BLEU 단점 보완해 정밀도와 재현율의 조화 평균 사용하며, 유의어 고려
동의어 매칭 및 어간 추출(Stemming) 지원
사람의 평가(Human Judgment)와 높은 상관관계
문법적인 유창성 평가에 유리
CIDEr: 이미지 캡셔닝에 특화, TF-IDF 가중치 사용해 중요 단어의 일치 여부 봄
다수의 정답 캡션 집합(consensus)과 유사도
자주 등장하지 않는 '핵심 단어'에 높은 가중치
현재 캡셔닝 대회(COCO 등)의 주요 평가 기준

Anchor based vs Anchor free: Anchor는 표적 중앙에 중앙 좌표를 넣고 3개의 박스를 그림.
Anchor free는 x,y 좌표를 봐서 비율을 재서 박스를 바로 만듬

트랜스포머: 병렬 처리

Multi-modal



# 09장: 3D 렌더링
가상 세계의 구성: 가상 카메라 배치 → 픽셀 그리드 삽입 → 오브젝트 배치 → 광원 추가(레이 캐스팅) → 최종 렌더링
광원이 빛을 방출 → 물체가 반사 → 카메라가 이를 받아들이는 과정 계산
3D 렌더링 활용처: 게임 그래픽, 메타버스, 영화/광고 CG, AR/VR, 건축/설계, 자율주행 시뮬레이션
현실 세계 → 카메라 촬영 → 3D 복원(NeRF) → 가상 세계 재현

2D                                      3D
좌표계: X,Y 축만 존재                   X,Y,Z(깊이) 축 존재
특징: 평면                             입체 공간(Volume)
깊이 없음                              거리에 따른 원근감 발생
원근감 표현 한계                        카메라 위치에 따른 모습 변화

3D 공간 내의 객체 다루기
점:vertices, 점의 집합: 선(edge), 선의 집합:면(faces)
기본 단위: 삼각형(3개의 점은 항상 하나의 평면 위에 존재하기 때문에 왜곡 없이 완벽한 평면을 보장)

메시: 현실의 물체 디지털로 옮기기 위해 Polygon Mesh라는 껍데기 사용
메시 3요소: V(vertex), E(Edge), F(Face)
표면 속성: 빛 반사 방향을 결정하는 법선(Normal) 벡터, 2D 이미지 입히기 위한 UV 좌표

NeRF(Neural Radiance Fields, 신경 방사 필드) 문제 정의: 2D 이미지 이용해 3D 공간 복원
입력: 보정된 다중 시점 이미지(Calibrated Images)와 각 이미지의 카메라 포즈(위치 방향) 정보 입력
출력: '학습에 사용되지 않은' 새로운 시점(Novel View)에서의 이미지를 합성.
마치 그 자리에 가서 찍은것처럼 자연스러운 장면 생성.
핵심 원리: 공간상 모든 점에 대해 밀도(Density) 및 색상(Radiance) 정의하는 연속적 함수 딥러닝 네트워크로 학습

Neural Volumetric 3D Scene Representation: 3D 공간 직접 저장(메시/복셀)하는 대신 MLP 신경망으로 표현. 공간상 위치(x,y,z)와 보는 방향(θ, Φ) 입력하면 색상과 밀도 출력
(x, y, z), (θ, φ): 3차원 공간의 한 점이 통과하는 위치, 관찰자의 시점과 방향(카메라가 보는 각도, (방위각, 고도각))
(r, g, b, a)
F(x, d) → (c, σ) c: 색상, σ: 부피 밀도
MLP+포지셔널 인코딩: MLP 사용해 고주파수 디테일(선명함) 표현하기 위해 좌표에 위치 인코딩 적용


Differentiable Volumetric Rendering(볼륨 렌더링): 신경망이 예측한 공간 정보 모아 2D 이미지로 변환(렌더링), 이 과정이 수학 미분가능, 딥러닝 학습 가능해짐.
C(r) = ∑ Ti · (1 - exp(-σiδi)) · ci
C(r):최종 픽셀; Ti: 누적 투과도(투과율), (1 - exp(-σiδi))=αi:불투명도, σi:부피 밀도, δi:거리, ci:색상
T = (exp(-∑σiδi))
보일랑말랑한 색상=티(T)알(알파)씨(c)

Optimization via Analysis-by-Synthesis: '합성을 통한 분석', 렌더링 된 이미지 실제 사진과 같아지도록 로스 계산해 신경망 스스로 업데이트
The Plepnoptic function: 어디서 보는가? 어떤 방향인가?
플렌옵틱 함수(Plenoptic Function): 모든 가능한 시점, 방향, 시간, 파장에 따른 빛의 세기 변화를 함수로 표현
7차원의 function 축약되어 표현됨 IPF = P(θ, φ, ω, τ, Vx, Vy, Vz)
P(Vx, Vy, Vz) : 3차원 공간 내에서의 위치, P(θ, φ) : 광선의 방향(방위각과 고도), P(ω, τ): 광선의 파장과 주파수(ω)와 시간(τ), (c, σ) = FΘ(x, y, z, θ, φ)

카메라 파라미터: 외부(Extrinsic): 월드 좌표계에서 카메라 위치와 회전(포즈)
내부(Intrinsic): 초점 거리(Focal length), 주점(Principal Point, 사물 중심)등 렌즈 특성

광선 방정식: r(t) = o + t · d
o:원점, d:방향(방위각, 고도각으로 만듬), t:거리(월드 좌표계 값)

카메라 원점 + 방향 → 3차원(좌표), 63차원(해상도 높임) → 색상(rgb), 밀도(시그마) 출력

x = r · cos(φ) · sin(θ)
y = r · sin(φ) (높이)
z = r · cos(φ) · cos(θ)

좌표계 변환(Coordinate Transform): 이미지 픽셀 좌표(u, v)를 카메라 좌표계로 변환한 뒤, 다시 카메라 자세 행렬을 곱해 월드 좌표계 방향 벡터 구함.

Ray Casting: 레이저 발사 → 충돌 감지 → 최초 충돌 선택(가장 가까이 있는 물체만 인식됨)
Ray Tracing: 이미지에서 해당 픽셀 위치에 보이는 객체 뭔지 파악하는 것
Ray Origin: 광선의 출발점
Ray Direction: 픽셀마다 광선 쏘는 방향

NeRF Volume 렌더링
공간의 정의: 모든 점은 밀도와 방사 복사도를 가짐(x,y,z,σ,c) 비어있으면 밀도가 0에 가깝고, 물체 있으면 높음
광선 적분: 카메라에서 픽셀로 쏜 광선을 따라 여러 점 샘플링, 각 점의 불투명도와 색상 누적(적분)해 최종 픽셀 색 결정
사실적 표현 vs 높은 연산량



# 10장:XAI
XAI(Explanable AI): 블랙박스 AI vs 설명 가능한 AI
XAI 실제 활용: 의료 및 헬스케어, 금융, 제조 및 품질 관리, 자율주행 자동차, 리테일 및 추천 시스템, 법률

CAM: AI 모델이 이미지 어디 보고 정답 선택했는지 시각적으로 보여줌.
작동원리: 입력 이미지(CNN에 입력) → 특징맵 추출(마지막 CNN층에서 특징 추출) → 가중치 결합 → 히트맵 생성(붉은색=중요)
CAM
필요조건: CNN끝단에 GAP(Global Average Pooling)레이어가 반드시 있어야함
핵심 아이디어: GAP 레이어 가중치 사용해 특징맵 결합
해석력: 특정 클래스 판단하는데 중요한 위치 히트맵으로 표시
한계점: 모델 구조 바꿔야해 이미 배포된 모델에는 쓰기 어려움
Grad-CAM: 양수만 남기기 위해 ReLU사용
이미 학습된 대부분의 CNN 모델에 즉시 적용 가능
역전파된 기울기 정보 사용해 중요도 계산
CAM 동일하게 위치 표시, 적용범위 훨씬 넓음
픽셀 단위의 아주 세밀한 경계선은 여전히 흐릿(CAM이 더 선명할 수도 있음)

Saliency Map: 민감도(Sensitivity) 검사
핵심 질문: 입력 이미지 픽셀 아주 조금 바꿨을때, AI 예측 결과가 얼마나 크게 변하는가?
과정: 전처리 → 기울기 계산 → 시각화

Integrated Gradients: 등산하듯 누적(출발점부터 도착점까지 모든 기울기 더함)
딥러닝 모델 출력에 대한 입력 특징의 기여도 정량적으로 계산
입력값과 기준값(Baseline, 정보가 없는 상태(검정 이미지)) 사이 잇는 선형 경로 따라 기울기 누적으로 적분
경로: Baseline 에서 실제 이미지까지 조금씩 픽셀값 변화 시키면서 이동(이미지가 서서히 밝아지는 과정)
각 입력 특징이 모델 결과에 얼마나 영향 주었는지 계산해 모델의 결정 근거 시각적으로 설명
Saliency 보다 노이즈 적고, 이론적으로 일관성 보장
산 입구에서 정상까지 경사가 여럿 있을때 각 경사가 정상까지 가는데 기여한 기여도의 합 = 100% →1

Saliency                vs              Integrated Gradients
로컬 기울기                             누적 기울기
계산 매우 빠름 직관적인 민감도 해석   기울기 포화 문제 해결, 이론적 완전성 보장
시각적 노이즈 많음                   계산 비용 높음, 적절한 BaseLine 설정 필요
빠름                                느림(높은 신뢰도와 규제 대응에 필요할 때)



# 11장: 객체검출 1: Two-Stage Detection
초기 접근-Sliding Window: Window 일정한 간격으로 이동시키면서 각 위치에서 객체 검출 수행, Window 스케일 고정하고 Scale을 변경한 여러 이미지 사용
근본적 한계: 이미지 대부분 배경인데 모든 영역 검사해야함 → 계산량 급증, 실시간처리 거의 불가능, 효율성 낮음

Regional Proposal: Selective Search 알고리즘
객체가 있을 가능성이 높은 후보 구역 추출해 검사 효율 높임, 모든 영역 검사 대신 2000개의 유망한 영역 선택
Over-segmentation → 유사 영역 병합 → Regional Proposal 생성
이미지 색상과 질감 기반으로 초기 분할 → 컬러|무늬|크기|형태 유사한 인접 segements 반복적 그룹화 → 병합 영역 Bounding Box로 변환해 최종 후보 영역 목록 작성

IoU: 검출 정확도 평가 핵심
gIoU: 겹치는 곳이 없을때 IoU가 0이므로(통일됨) gIoU 사용함

mAP☆☆: 모든 클래스에 대한 AP의 평균값

R-CNN: 근본, Regional Proposal로 이미지 쪼개고(2000개) 그 쪼갠 모든것이 CNN돌림. 느려서 안씀(84시간)
Fast R-CNN: 발전판, Regional Proposal은 함 ROI는 만들고 이걸 맥스풀링함. 안씀(9.5시간)
Faster R-CNN

PASCAL VOC, MS COCO: MS COCO로 더 엄격하게 평가



# 12장: 객체검출 2: One-Stage Detection
Two-Stage Detection            vs              One-stage Detection
정확도 우선                                         속도 우선
Faster R-CNN, Mask R-CNN                           YOLO, SSD
영역 제안 → 분류 및 회귀             단일 신경망이 이미지 전체 한번 보며 위치와 클래스 동시 예측
높은 정확도                                       매우 빠른 속도
느린 속도                                      정확도 트레이드오프
의료 영상, 정밀검사, 서버분석              자율주행, CCTV, 모바일 앱, 로봇

YOLO v5(실전 엔지니어링 완성)
PyTorch 기반 쉬운 사용성과 배포 편의성 극대화 산업 표준 자리잡음
(CSPDarknet 백본, Mosaic Augmentation, Auto Anchor)
CSP(Cross Stage Partial) 구조 도입: 연산량 줄이고 학습 능력 강화 
구조: Backbone(CSPDarknet) 연산 → Neck(PANet, Path Aggretion Network) 백본에서 추출된 특징 융합 → Head(YOLO Layer) 검출
과정: Feature map 뽑고(CNN) → Split & Merge(입력 특징 둘로 나누어연산(Conv), 우회 경로 하나 만듬)
다중 스케일 전략: 소형 객체(80x80 grid/얕은 특징/작은 물체), 중형 객체(40x40/중간/일반), 대형(20x20/깊은/큰)
필터를 큰거를 쓴다 → 넓게 보겠다(객체 하나가 화면 가득 채우지 않겠다)
FPN, PANet: FPN(↓) 무엇인지에 대한 의미 정보 아래로 전파, PAN(↑) 어디에 있는지에 대한 위치 정보 위로 다시 끌어올림
자동 앵커: K-means(군집 분석 방법, centroid(중심) 계속 옮겨가면서 군집을 나눔 ) 자동 산출
Mosaic Augmentation: 서로 다른 4장 학습 이미지 무작위 크롭하고 배치해 한 개 학습 이미지로 합성 → 소형 객체 검출력 향상, 배치 크기 의존도 감소
실무: v5s(메모리 제약 심한 환경), v5m(균형), v5l, v5x(실시간보다  0.1% 정확도 향상)

YOLO v8(앵커 프리)
복잡한 앵커 설정 제거후 헤드 분리해 유연성 및 정확도 동시 확보
(Anchor-Free Base, Decoupled Head, Task-Aligned Assigner)
☆☆Anchor-Free로 전환: 사전 정의된 앵커 박스 크기에 의존해 최적 앵커 찾기위한 K-means등 추가 연산과 하이퍼파라미터 튜닝이 필수였음.
(마치 지도 격자 vs GPS 좌표: 격자에 구애받지 않고 객체가 있는 곳 직접 찍음 or 옷 사이즈 맞추기: 기성복 vs 맞춤복)
중심점 예측: 객체 중심점과 중심점으로 부터 거리 직접 예측
분리형 헤드: 분류와 위치 회귀 브랜치 분리해 학습 간섭 줄임
C2f(Cross Stage Partial + Feature Flow): 일부 채널만 잔차 경로로 우회시켜 계산 줄이고 표현력 유지
입력 분할(split)후, 각 단계의 결과 모두 합침(Concat)
다중 스케일 전략: 소형 객체(80x80 grid/얕은 특징/작은 물체), 중형 객체(40x40/중간/일반), 대형(20x20/깊은/큰)
v8에서도 다중 스케일 전략 그대로 사용
PAN-FPN은 의미(Semantic)와 위치(Spatial) 단서를 상하로 재순환시켜, 모든 스케일의 객체 검출 성능을 극대화
Decoupled Head(분리 헤드): 분류 브랜치와 회귀 브랜치 물리적 분리해 독립적인 경로로 학습
장점: 학습 안정성 & 수렴 속도 가속; 효과: Classfication / Regression 간섭 제거; 결괴: mAP 상승 및 예측 품질 향상
t = sα × uβ (t: 통합점수; s: 분류 점수; u: 위치 정확도; α,β: 가중치(보통 0.5, 6.0))
CIoU Loss: 단순 겹침 넘어 중심 거리와 종횡비까지 고려해 박스 정밀하게 맞춤
☆☆Loss(CIoU) = (1 - IoU(얼마나 겹쳤나)) + (ρ2(b, bgt) / c2 (중심점 간 유클리드 거리)) + αv(종횡비 일치도)
Focal Loss = FL(pt) = - αt(1 - pt)γ log(pt) (쉬운 문제 배점 낮게, 어려운 문제 배점 높게)
CIoU(위치 정밀도) + Focal(난이도별 가중) + 정렬 할당으로 검출의 정합성과 어려운 케이스 대응력 동시에 높임.

YOLO v10(NMS-Free)
후처리(NMS) 과정 학습 단계 내재화 해 추론 지연 시간 획기적으로 단축
(NMS-Free Training, Dual Label Assignment, Efficiency-Accuracy)
NMS의 한계: 하이퍼파라미터 민감(IoU 임계값 설정에 따라 검출 결과 크게 달라짐), 추론 지연 시간 증가(GPU 병렬 연산 후 별도의 순차적 연산이 필요해 전체 속도 저하), 배포 복잡성(TensorRT등 최적화 시 NMS 플러그인 구현이 까다로움)
End-to-End 최적화: 학습 단계에서 중복 스스로 억제, 추론 시 후처리 필요없는 깔끔한 결과 출력(기존: 예측 → NMS → 결과; v10: 예측 → 결과(즉시))
Dual Label Assignment: One-to-Many(예선) + One-to-One(본선): 두가지 할당 방식 동시에 사용
One-to-Many(예선): 하나의 GT(정답)에 여러 예측박스 할당해 학습 신호 풍부하게 제공(Recall 강화)
One-to-One(본선): 하나의 정답에 오직 하나의 예측 박스만 매칭해 중복 억제(NMS 불필)
Dual Assignment는 중복 억제 과정을 학습 단계에 내재화하여, 추론 시 NMS 후처리 없이도 깔끔한 결과 얻음.
Large-Kernel DWConv(대커널 심층별 컨볼루션): 7*7 커널을 Depthwise로 적용해 연산 효율 높임(넓은 Receptive Field(수용 영역)확보, Depthwise 연산으로 파라미터 수 감소)
CIB(Compack Inverted Block) 구조: 확장(Expand) → 깊이별(Depthwise) → 축소(Project) 흐름 통해 정보 손실 최소화 해 연산량 줄임
Depthwise Convolution: 채널별로 따로 계산(원래 C,H,W 면 R, G, B 종이 한장씩 전체적으로 보는데, H,W,C면 픽셀 하나에 [R, G, B]값이 동시에 들어옴)
Depthwise 파라미터: (k * k)(커널 사이즈) * Cin(입력 채널) * Cout(출력 채널)

        YOLO v5 안정성          YOLO v8 표준/범용               YOLO v10 초고속/효율
아키텍처 CSP + PANet            C2f + PAN-FPN                   CIB + Partial SA
앵커방식 Anchor-Based           Anchor-Free                     Anchor-Free
헤드구조 Coupled Head           Decoupled Head                  Decoupled Head
후처리   NMS 필수               NMS 필수                         NMS-Free
손실함수 GIoU / CIoU            CIoU + Focal + DFL              Dual Assignment
성능지표(v5s 기준)               (v8s 기준)                      (v10s 기준)
mAP     37.4                    44.9                           46.3
Params  7.2M                    11.1M                          8.0M
추천용도 레거시호환, 튜닝자유도   고성능, 쉬운학습/배포            엣지디바이스, 최소지연

안정성(v5), 고성능 표준(v8), 초저지연 엔드투엔드(v10) 중 프로젝트 제약사항(하드웨어, 데이터)에 맞춰 선택
정확도(v5) → 구조적 유연함(v8) → 효율적 종단학습(v10)

산업별YOLO 선택 가이드
자율주행(Autonomous Driving)
권장모델: v8 / v10 (Large/Medium)
복잡한 장면 대응: 앵커프리(v8) 및 NMS-Free(v10) 구조가 밀집된 객체 인식에 유리
Latency 임계값: v10의 후처리 제거로 추론 지연 시간을 극소화 하여 실시간성 확보

보안CCTV (Surveillance)
권장모델: v5 (Medium) 또는 v8 (Small)
안정성/가성비:24시간 가동 서버 부하를 고려한 중소형 모델 선택
검증된 파이프라인: v5는 배포 안정성이 매우 높아 레거시 시스템 통합에 유리

의료영상(Medical Imaging)
권장모델: v8 (Large/X-Large)
정확도 최우선: 속도 보다 False Negative(미검출) 방지가 핵심, 대형 모델 사용 권장
희귀 케이스: v8의 Focal Loss 등은 데이터불균형(정상vs 질병) 학습에 효과적

모바일& 엣지(Mobile Edge)
권장모델: v5n / v8n / v10n (Nano)
리소스 제약: 배터리 소모, 발열, 메모리 한계 극복을 위한 Nano급 모델 필수
경량화 효율: v10n은 CIB블록으로 적은 연산량 대비 높은 mAP 달성

YOLO v5                       YOLO v8                          YOLO v10
실전안정성& 엔지니어링          Anchor-Free & 구조 혁신           NMS-Free & End-to-End
CSP + PAN 구조 정립            Anchor-Free (좌표 직접 예측)      NMS-Free (후처리 제거)
Anchor 기반 (Auto Anchor)      Decoupled Head (분리형 헤드)      Dual Assignment (중복 억제 학습)
Mosaic 증강으로 소데이터극복    C2f 모듈로 기울기흐름개선          효율적 대커널 컨볼루션



# 13장: 의료영상, SSD
의료 영상 분석: DICOM(Digital Imaging and Communications in Medicine) 의료 영상과 관련 정보 함께 저장하는 국제 표준 파일 형식
JPG/PNG: 사진 픽셀 정보만 포함          DICOM: 이미지+환자정보+촬영정보+장비정보
DICOM 파일 구조:헤더 정보(파일 식별자, 버전 정보), 메타 데이터(환자명, 나이, 성별, 촬영날짜, 병원명, 장비 모델), 픽셀 데이터(실제 의료 영상 픽셀 정보)
ROC Curve(Receiver Operating Characteristic Curve): 민감도와 거짓 양성률 사이의 트레이드 오프 보여줌
TPR(민감도)= TP / (TP + FN)     FPR(거짓양성률)= FP / (FP + TN)
의료 AI: Recall 중시 상황: 암, 폐렴 같은 질병 스크리닝, 환자 놓치면 안될때; Precision 중시 상황: 침습적 치료 결정: 잘못된 양성 진단이 큰 비용/ 부작용 초래(수술 필요한 경우), F1-Score 균형: Precision과 Recall 동시 고려(전반적인 품질 지표로 활용)

YOLOv1: 7*7 그리드로 이미지 나눔, Output(7*7(5*B+C))(5: dx,dy,dw,dh,confidence; B: Base Box, C: 클래스)
YOLOv3: 1*1*(B*(5+C)) B=3(바운딩 박스), SSD 영향 받음(앵커박스 도입), FPN(피처 피라미드 네트워크) 도입

SSD: Single Shot Multi-Box Detector
One Forward Pass(Single Shot): Regional Proposal 단계 없는 1-Stage Detector
Multi-Scale Feature Maps: 다양한 크기 객체 검출 위해 여러 해상도 Feature Map 사용 (38*38 → 1*1)
Default Boxes(Anchors) 각 피처맵 샐마다 고정된 크기 및 비율 가진 Default Box 미리 정의
종횡비(Aspect Ratio): 각 피처맵의 셀마다 다양한 비율 박스 생성해 여러 형태의 객체에 대응(기본 4~6개) 
매칭되면 Default Box Positive로 매칭(박스 8732개 보유, 약 66%의 박스는 38*38(작은게 잘 안잡힘))
예측 메커니즘
분류: C+1 클래스 확률(+1=배경)
위치 회귀: 4개 오프셋(Δcx,Δcy,Δw,Δh)예측해 중심점 이동 및 너비/높이 비율 조정해 박스 조절
후처리:NMS통해 중복 제거
손실함수: L(x,c,l,g)=1/N(Lconf(x,c)+αLloc(x,l,g))
N:매칭된 디폴트 박스 개수; x:매칭 여부 나타내는 지표변수; c:클래스 분류 예측값; l:예측된 박스 좌표; g:정답 박스 좌표; α: 두 손실 사이 비중 조절하는 
좌표 파라미터:
중심 좌표 오프셋(Center)→핵심원리:절대 좌표 직접 학습 하지 X, Default Box 크기(dw, dh)로 정규화된 상대적 거리(Offset)를 학습해 위치불변성(Translation Invariance) 확보.
크기 오프셋(Size)→핵심원리:크기 비율에 로그를 취함, 작은 박스와 큰 박스 간 오차 스케일 맞춤. 이는 다양한 크기 객체 동일한 비중으로 학습하는 스케일 불변성(Scale Invariance) 확보. 
Confidence Loss: Softmax + Cross Entropy
Hard Negative Mining: 이미지 내 객체(Positive)보다 배경(Negative)이 압도적으로 많다. 모든 배경 학습시 모델이 배경만 잘 맞춤, 객체는 못 찾는 문제 발생.
Positive: 물체(클래스); Negative: 배경; Easy Negative(하늘, 바다같은 그냥 검출하기 쉬운 배경)
Hard Negative: 복잡한 배경(3:1 비율로 Positive 개수의 3배만큼 상위 Negative 선택, Positive 3개면 Negative 12개 선택)
SSD 학습 파이프라인: 1.순전파(모든 박스 예측) → 2.박스 매칭(IoU 체크) → 3.마이닝(Hard Negative: 3:1) →  4.손실계산(L_conf + L_loc) → 5.역전파(가중치 업데이트)
Default Box Matching 전략:SSD는 학습시 어떤 Default Box가 Ground Truth(실제 정답)에 해당하는지 결정하기 위해 IoU(Intersection over Union)를 기반으로 두 단계의 매칭 전략 사용.
1단계: Best Prior Matching 가장 높은 IoU를 가진Default Box를 하나씩 우선 매칭, 이는 모든 실제 객체가 최소 하나의 Default Box와 반드시 연결되도록 보장.
2단계: Threshold Matching 남은Default Box들 중, Ground Truth와의 IoU가 0.5 이상인 모든 Box들을 추가로 매칭합니다. 이는 하나의 객체에 대해 여러개 Default Box가 학습에 참여할 수 있게 되어 예측의 안정성 높임



# 14장: ViT, Transformer
Attention: Q != k=v; Self-Attention: Q=k=v
Encoder: Bert; Decoder: GPT
Multi-Head Attention & Transformer 구조: 12개의 다른 구도에서 바라보는 카메라
RNN/LSTM의 3가지 한계:
1. 순차 처리 → 병렬화 불가: 이전 단계 계산 완료해야 다음 단계 시작 가능. GPU의 병렬 처리 능력 활용 못해 학습 속도 느림.
2. 장거리 의존성: 문장 길어질수록 앞부분 정보 소실. Gradient Vanishing 문제로 인해 문맥 유지 어려움.
3. 정보 병목 현상: 모든정보를고정된크기의Hidden State 벡터 하나에 압축 해야 하므로 정보 손실 발생.
Transformer의 혁신
1. 완벽한 병렬화(Parallelization): 모든 위치 단어 동시 입력받아 계산. GPU 활용 극대화해 학습 속도 10배 이상 향상.
2. 직접연결(Direct Connection): Self-Attention을 통해 모든 위치가 서로 직접 참조. 거리 상관 없이 정보 전달 가능
3. 무한한메모리접근:정보 압축 않고 Attention 메커니즘으로 필요한 정보만 선택적 가져와 사용.

Self-Attention의 핵심: Query, Key, Value
Query: 질문, Key: 색인, Value: 실제 내용
(예시) "The animal didn't cross the street because it was too tired" → 여기서 it 은 무엇인가?
Query(it): it은 무엇을 가리키는 대명사? Key(모든 단어): 각 단어가 자신의 특성(명사, 동사 등 노출) → 매칭 결과: it의 Query와 Animal의 key가 제일 높은 유사도(Attention Score) 가짐 → Value 가져오기: animal의 value(동물, 주어 등)을 it에 반영
Self-Attention의 계산 5단계: Q, K, V 생성(Linear Projection) → Attention Score 계산 → Scaling(스케일링) → Softmax 적용 → Value 가중합(Weighted Sum)
입력 벡터 X에 학습 가능한 가중치 행렬 곱해 3개의 벡터 생성 → Query와 Key 내적 통해 두 단어간 연관성(유사도) 계산 → 차원 크기(d_k)의 제곱근으로 나누어 값의 크기 조정(gradient 안정화) → 점수를 0~1 사이 확률값 변환(전체 합=1) → Attention 확률 가중치로 해 Value 벡터들 합 계산
Attention(Q, K, V) = softmax(QKT/√dk)*V

Multi-Head Attention: Query, Key, Value 각각 여러 개 head로 분할 후 각 헤드마다 독립적으로 어텐션 계산 후 결과 결합. → 모델이 여러 표현 공간에서 동시에 정보 집중 가능(병렬)
Multi-Head Attention: 구문적 관계 파악 집중, 의미론적 유사성 연결해 문맥 형성 가능, 대명사 및 개채명 참조 가능, 장거리 의존성 해결

BERT: Multi-Head Attention 적극 활용, 12개 head 사용, 각 헤드 차원 64(768/12)

위치 정보 인코딩(Positional Encoding): Attention 메커니즘은 단어의 순서를 고려하지 않기 때문에, 위치 정보를 주입해 순서 정보 제공하고 입력 벡터를 재정의해 모델이 단어의 의미뿐 아닌 문장내 위치 정보까지 함께 학습.

Transformer Encoder 블록 구조: 1. Multi-Head Self-Attention → 2. Add & Layer Normalization → 3. Feed Forward Network(FFN) → 4. Encoder Stack(x N)
Feed-Forward Network(FFN) 상세: FFN(x) = GELU(xW₁ + b₁)W₂ + b₂
x: 입력 벡터; W₁: 확장(768 → 3072); W₂: 축소(3072 → 768)
왜4배로확장하는가? (Expansion):저차원(768)공간 정보 고차원(3072)으로 투영해 더 복잡,풍부한 특징 학습. ReLU/GELU 비선형 함수가 이 고차원 공간에서 작동, 표현력 극대화.
핵심구성요소
1. Multi-Head Self-Attention: 입력 시퀀스 내 모든 위치 간 관계 모델링
2. Residual Connection: 입력을 출력에 더하여 gradient flow 개선
3. Layer Normalization: 각 층의 출력을 정규화하여 학습 안정화
4. Feed-Forward Network: 위치별로 독립적인 2층 MLP 적용
(d_model → 4×d_model → d_model)
설계철학
• Residual connection과 Layer Normalization은 깊은 네트워크 학습을 가능하게 합니다. 
• BERT는 12~24층, GPT-3는 96층의 Encoder/Decoder를 쌓아 고수준 표현을 학습합니다.
• FFN은 attention으로 모은 정보를 비선형 변환하여복잡한패턴을포착합니다. 
• ReLU나 GELU 활성화 함수를사용합니다.

Transformer Decoder 블록

BERT와 파인튜닝: Transfer Learning의 핵심 → 사전학습; 파인튜닝; DistilBERT의 효율성(파라미터 40% 줄이고 속도 60% 향상시킨 경량화 모델, 성능 원본 97% 수준 유지)

ViT(Vision Transformer): 224*244 이미지 16*16 패치 14*14개로 쪼갬
CNN vs ViT: 대규모 데이터에서 ViT가 승리(대규모일경우 CNN의 경우 Gradient 소실 발생하는데 ViT는 병렬이라 학습 가능)
ViT 전개: 이미치 패치 분할(196개 패치 생성 → 16*16*3=768 벡터로 Flatten) → 선형 투영 및 위치 임베딩(768 차원 매핑, 학습가능한 위치 임베딩 더함, CLS 토큰 붙여 분류에 활용) → Transformer Encoder 적용(12층, 12헤드, 768 은닉차원 사용, CLS 토큰 분류 헤드에 입력)
ViT 입력 처리: 이미지 패치 분할 → Flatten → Linear Projection → Position Embedding 추가 → [CLS] 토큰 추가



# 15장: 강화학습 개론
강화학습: 에이전트가 환경과 상호작용하며 보상을 최대화하는 행동 전략을 스스로 학습하는 머신러닝 패러다임
지도학습과 달리 정답 레이블 없이 시행착오 통해 학습, 게임 AI, 로봇 제어, 자율주행, 추천 시스템 등 다양한 분야에서 혁신적인 성과.
핵심 구성 요소: Agent, Environment, Reward, Policy
실제 적용 사례: 게임AI, 로봇제어, 자율주행, 추천 시스템

MDP(Markov Decision Process): 강화학습 문제 수학적 정의하는 프레임워크, 현재 상태만으로 미래 예측할 수 있다는 마르코프 속성 가정.
P(S_t+1|S_t,S_t-1,...,S_0)=P(S_t+1|S_t) → 미래를 알고 싶으면 모든 과거가 아닌 현재만 알면 된다
MDP 구성 요소: State(S): 에이전트가 관찰하는 환경의 현재 상황; Action(A): 에이전트가 선택할 수 있는 행동의 집합; Reward(R):특정 행동의 즉각적인 보상, Policy 학습의 피드백 신호 역할; Transition(P):행동에 따른 다음 상태로의 전이 확률; Discount Factor(γ): 미래 보상 현재 가치 결정하는 0~1사이 값, 장기적 전략 수정

Value Function, Bellman Equation: 특정 상태 또는 상태-행동 쌍이 얼마나 좋은지 나타내는 기대 누적 보상
V(s): 상태 s 에서 정책 π를 따를 때의 기대 수익
Q(s,a): 상태 s에서 행동 a를 취한 후 정책 π를 따를 때의 기대 수익

상태(S), 행동(A)

Q-learning: 각 상태-행동 싸의 가치(Q-value)를 테이블에 저장하고 반복적으로 업데이트해 최적 정책 학습하는 off-policy 알고리즘
1. Q-table 초기화 → 2. 행동 선택 → Q-value 업데이트 → 수렴

DQN(Deep Q-Network): Q-table 신경망으로 대체 → 고차원 상태 공간 문제 해결
Experience Replay:과거 경험 메모리에 저장하고 무작위로 샘플링해 학습, 데이터간 상관관계 제거 후 학습 안정성 높임
Target Network: 일정 주기마다 업데이트되는 별도 네트워크 → 목표값 계산; 학습 과정 발산 방지
Deep Neural Network: 다층 신경망 → Q-function 근사. 복잡한 패턴, 고차원 입력(이미지 등) 효과적 처리
Q-network는 항상 갱신; Target은 s가 변하면 갱신
Q(s, α) Q(s, α) + α(r + γmaxQ(s', α') - Q(s, α))

                Value-based             vs              Policy-based
대표 알고리즘   Q-Learning, DQN                         Policy Gradient, PPO
          Q-value 학습 후 간접적 정책 도출        파라미터화 된 정책 직접 학습
          Q(s,a)→argmax→결정적(Deterministic)     π(a|s)→확률 분포→확률적(Stochastic)
             주로 이산 행동 공간에 적합            연속 행동 곤간 처리에 유리

Policy Gradient Theorem 핵심: 기대 보상 최적화는 정책 찾기; 좋은 결과 낸 행동 확률 높이고, 나쁜 결과 낸 행동 확률 낮추기
Reinforce 알고리즘(동일 행동에 대한 불확실한 보상이 gradient 방향을 계속 바꿈) → Baseline 개념 도입(절대적 보상 크기가 아닌 '평균 대비' 상대 성능이 중요함)
Advantage Function: 행동이 평균보다 얼마나 더 좋았나(상대적 가치 평가)

Actor-Critic Architecture 학습: 매 스텝마다 실시간 학습(Actor: 행동 선택; Critic: 행동 평가)
행동 선택 및 실행 → Critic 업데이트(TD Error) TD target과 현재 가치 차이 최소화 → Actor 업데이트
Temporal Differece Error: 현재의 예측과 잠시 후 알게 된 사실 사이의 차이
Criticd이 가치를 추정하기에 REINFORCE보다 분산이 낮음. 그러나 Critic 추정값이 부정확하면 편향이 발생

A2C (Advantage Actor-Critic): 동기식(Synchronous) 업데이트와 정교한 Advantage 추정으로 학습 안정화
병렬 환경여러개 독립된 환경에서 동시에 데이터 수집해 한번에 Gradient 업데이트(A, B, C가 일하고 각각 보고서 제출)
Actor-Critic의 한계: 학습 불안정성: 급격한 정책 업데이트로 인한 성능 붕괴 현상(Policy Collapse)
Trust Region 개념: 안전한 학습을 위한 정책 업데이트 제한(급커브 급지)
제약조건: 𝐾𝐿 ( 𝜋𝑜𝑙𝑑 | | 𝜋𝑛𝑒𝑤 ) ≤ 𝛿; KL:KL Divergence, 𝛿:최대 변화량

PPO: Clipping 복잡한 KL 대신, 간단한 비율 제한(Clipping)으로 안전성 확보
중요도 비율: 이전 정책 대비 현재 정책 확률 비율 계산해 정책 얼마나 변했는지 측정
$$r_t(θ) = {π_{\theta}(a_t|s_t)}{π_{\theta_{old}}(a_t|s_t)}$$
$$π{\theta}: 현재 업데이트하려는 새로운 정책$$
$$π{\theta_{old}}: 데이터를 수집했을 때 사용했던 이전 정책$$
$$L^{CLIP}(\theta) = \hat{E}_t \left[ \min(r_t(\theta)\hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\hat{A}_t) \right]$$

PPO 강점: Sample Efficiency: 중요도 샘플링으로 데이터 여러 번 재사용해 학습 속도 향상, Stability: Clipping 통해 정책 급격하게 변하는 것 방지해 학습 붕괴 예방, Simplicity: 복잡한 2차 최적화 없이 1차 미분(First-order)만으로 구현 가능

실무적용표준
RLHF (ChatGPT): 인간 피드백 기반 미세 조정
Robotics: 보행, 조작 등 연속 제어 문제 해결
안정성, 효율성, 범용성에서 현재 최고의 선택



→ α ☆