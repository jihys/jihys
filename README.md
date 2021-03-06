## Step 3: Create model
이 페이지에서는 AWS DeepRacer용 RL 모델을 만들고 모델 교육을 시작하는 것을 배웁니다. 이 페이지는 몇 개의 섹션이 있습니다. LAB을 시작하기 앞서 먼저 스크롤 다운 하여 어떤 내용들이 있는지 확인해 봅니다. 여기서는 AWS DeepRacer 자동차가 스스로 트랙에서 자율 운행 하기 위해 사용 될 모델을 만들 것입니다. 그러기 위해서 제일 먼저 레이스 트랙을 선택하고, 모델이 선택 할 수 있는 행동을 정의하고, 원하는 운전 행동을 장려하기 위해 사용할 보상 함수을 디자인 하고, 훈련 중에 사용되는 하이퍼파라미터를 조정 할 것입니다. 

### <font color=blue>**Info**</font> **Buttons**
콘솔 안에서 <font color=blue>**Info**</font>  해당 버튼을 누르면, information 창이 스크린 오른 쪽에서부터 앞으로 나타 나게 됩니다.  Info 창은 창 안에서 별도 링크를 선택하지 않는 한 그대로 오른편에 남아있게 됩니다. 내용을 다 읽으신 뒤에는 창을 닫을 수 있습니다.

## 3.1 Model details
먼저 맨 위의 Model details부터 시작 합니다. 여기서는 모델 이름을 지정하고 모델에 대한 설명을 입력합니다. 서비스를 처음 사용하는 경우 **Create Resources** 버튼을 선택합니다. 이렇게하면 AWS DeepRacer가 사용자를 대신하여 다른 AWS 서비스를 호출하는 데 필요한 IAM 역할, 훈련 및 검증에 사용되는 VPC 스택, 보상 함수를 검증하기 위해 Python 3로 쓰여진 AWS DeepRacer 람다 함수, 그리고 모델 아티팩트가 저장 될 AWS DeepRacer S3 버킷이 생성됩니다. 이 섹션에서 오류가 나면 저희에게 알려주십시오.

![Model Details](img/model_details.png)
모델의 이름과 설명을 입력하고 다음 섹션으로 스크롤하십시오.

## 3.2 Environment simulation
워크샵에서 자세히 설명했듯이, RL 모델을 훈련하는 것은 시뮬레이션 된 레이스 트랙에서 이루어지게 됩니다. 이 섹션에서는 모델을 교육 할 트랙을 선택하게됩니다. AWS RoboMaker는 시뮬레이션 환경을 구성하는데 사용됩니다. 

모델을 훈련 할 때는, 실제로 레이스하기를 원하는 트랙을 미리 생각해 보고 최총적으로 레이스 할 트랙과 가장 가장 유사한 트랙을 선택 하여 훈련합니다. 이것은 모델의 성능을 보장하지는 않지만 모델이 레이싱장에서 최고의 성능을 발휘할 확률을 최대화합니다. 또한 직선 모양의 트랙을 선택 하여 훈련하는 경우 모델이 좌,우측 턴을 할거라는 기대는 하지 않는게 좋습니다. 

Section 2에서 AWS DeepRacer League에 대한 자세한 내용을 제공 할 예정이지만, 리그에서 경기하기 위해 훈련 할 트랙을 선택할 때는 유의해야 할 사항이 있습니다.

-  실제 레이스 트랙은 Invent 2018 트랙 [Summit Circuit](https://aws.amazon.com/deepracer/summit-circuit/)이 될 것이므로 선택한 AWS Summit 중 어느 곳에서 경주를하려고한다면 re:Invent 트랙에서 모델을 훈련 시키십시오.

- 버추얼 서킷은 매번 새로운 경기용 트랙이 생기며, 이 경기용 트랙을 직접 훈련 할 수는 없습니다. 대신 테마와 디자인이 매 경기마다 동일하지는 않지만 유사합니다. 즉 자율 주행이 성공적이려면 모델이 일반화 되어야하며, 경기 트렉에 Overfitting해서는 않 됩니다.

오늘의 Lab에서는 시간이 가능한 한 최대로 여러 분들이 Summit에서 레이스를 참가를 위해 준비 될 수 있도록 도와 드릴 예정입니다. Rre:Invent 2018 트랙을 선택하고 다음 섹션으로 스크롤하십시오.

## 3.3 Action space
이 섹션에서는 훈련 과정 및 실 주행에서 선택 할 수 있는 Action Space를 정의합니다. Action은 자동차가 취할 수 있는 스피드와 조향각의 조합 입니다. 
AWS DeepRacer에서는 Continuous Action Sapcerk 아닌 Discrete Acation space를 사용합니다. 이 Discrete Action Space를 정의하기 위해 최대 속도(Maximum Speed), 속도 레벨(Speed Levels), 최대 조향 각도(Maximum Steering Angle), 그리고 조향 레벨 (Steering Levels) 을 지정하게됩니다.

![action space](img/Action_Space.png)

Inputs

- 최대 조향 각도는 차량의 앞 바퀴가 왼쪽과 오른쪽으로 회전 할 수있는 최대 각도입니다. 바퀴가 얼마나 멀리 회전 할 수 있는지에 대한 한계가 있으므로 최대 회전 각도는 30도입니다.
- 조향 레벨은 양측의 최대 조향 각도 사이의 조향 간격의 수를 나타냅니다. 따라서 최대 조향 각도가 30 도인 경우 + 30 도가 왼쪽이고 -30 도가 오른쪽입니다. 조향 레벨이 5 인 경우 왼쪽에서 오른쪽 방향으로 30도, 15도, 0도, -15도 및 -30 도의 조향 각도가 동작 공간에 표시됩니다.  조향 각도은 언제나 0도를 기준으로 대칭입니다.
- 최대 속도는 자동차가 시뮬레이터에서 운전할 최대 속도를 m/s 로 측정 한 것입니다.
- 속도 레벨은 최대 속도(포함)에서 0까지의 속도 레벨의 개수를 나타냅니다. 따라서 최대 속도가 3m/s이고 속도 레벨이 3 인 경우 Action Space 에 1 m/s, 2m/s, and 3 m/s의 속도가 포함됩니다. 간단히 3m/s 를 3으로 나누면 1m/s가 되고,  0m/s 에서 3m/s로 1m/s씩 증가합니다. 0m/s는 Action Space에 포함되지 않습니다.

위의 예시 경우, AWS DeepRacer 서비스는 총 15개(3 단계 속도 x 5 단계 조향 각도)의 Action Space를 나열 하게 됩니다. 아직하지 않았다면 Action Space 구성하십시오. 자유롭게 구성 하되, 많은 종류의 Action Space는 훈련하는 데 약간 더 오래 걸릴 수 있음을 인지 합니다.

Hints

- 모델은 Action Space에 없는 행동을 수행하지 않습니다. 또한 이 행동을 하지 않아도되는 트랙에서 모델을 훈련하는 경우 (예 : 직선 트렉에서는 좌우 턴을 배우지 못함) 해당 행동은 어느 때 사용해야 하는 지 몰라 사용 하지 않게 됩니다. 따라서 견고한 모델을 만들 생각을하기 시작할 때 Action Space와 훈련 트랙을 염두에 두십시오.
- 빠른 속도 또는 넓은 조향각을 지정하는 것은 좋지만, 보상 함수와 전속력을 회전으로 전환하는 것이 합리적인지 또는 트랙의 직선 구간에서 지그재그로 행동 하는 것이 좋은 것인지 생각해 봐야 합니다.
- 실제 경주의 경우 AWS DeepRacer의 웹 서버 사용자 인터페이스에서 Throttle을 사용해야 만 자동차가 시뮬레이터에서 배운 것보다 빨리 주행하지 못하게 할 수 있습니다.


## 3.4 Reward function

강화 학습에서는 보상 함수를 잘 설계하는 것이 이 모델 훈련에 **중요한 역할**을 합니다. 보상 함수는 훈련 된 RL 모델이 자율 주행을 할 때 우리가 원하는 바대로 행동 할 수 있도록 좋은 운전 행동을 장려하는 데 사용됩니다.

보상 함수는 행동 결과의 품질을 평가하고 그에 따라 보상합니다. 보상은 훈련 중, 각 행동이 취해진 후에 시뮬레이션에서 계산됩니다. Python 3 구문을 사용하여 보상 함수에 들어갈 로직을 설계 합니다. 이 논리를 코드화 하기 위해 시뮬레이션에서 제공하는 여러 가지 측정 값을 변수로 사용 할 수 있습니다. 

다음 목록에는 보상 함수에 사용할 수 있는 변수가 들어 있습니다. 아래 변수는 더 좋은 방법이 있을 경우 이를 반영하여 수시로 업데이트 됩니다. 따라서 보상 함수를 이에 따라 적절히 조정하십시오. Seoul 워크샾에서는 아래 변수와 설명이 AWS DeepRacer 콘솔에서 현재 사용 가능한 버전입니다. 따라서 AWS DeepRacer 서비스에서 간혹 다르게 설명 되어 있다면 이를 무시하고 아래 변수 이름과 설명을 사용하십시오. 가장 두드러진 차이점은 Throttle과 Steering 입니다.

| 변수 이름	            | 타입                     | 설명                                                                                                                                                                                                                                                                                         |
|----------------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| on_track             | boolean                  | 4 개의 바퀴 중 하나라도 도로 표면과 경계선으로 정의된 트랙에 있으면 one_track은 True 입니다. 4 개의 바퀴가 모두 트랙에서 벗어난 경우 on_track은 False입니다. on_track이 False이면 차량이 리셋됩니다.                     |
| x                    | Float                    | 자동차의 차축 중심의 x 좌표를 미터로 반환합니다.                                                                                                                                                                                                                 |
| y                    | Float                    | 자동차의 차축 중심의 y 좌표를 미터로 반환합니다.                                                                                                                                                           |
| distance_from_center | Float [0, track_width/2] | 트랙 중심에서 절대 거리. 트랙의 중심은 모든 중심 waypoints 를 연결하는 선에 의해 결정됩니다.                                                                                                                                                         |
| car_orientation      | Float (-pi,pi]           | 자동차가 향하는 방향을 Radian 단위로 반환합니다. 차가 x 가 증가하는 방향으로 향하고 y가 일정하다면 0을 반환합니다. 차가 y가 증가하는 방향으로 향하고 x가 일정하면 pi/2를 반환합니다. 차가 y가 감소하는 방향으로 향하고, x가 일정 할 때, -pi/2를 반환합니다. |
| progress             | Float [0,100]              | 트랙 주행 완료율.                                                                                                                                                                                                                                                                   |
| steps                | Integer                  | 완료된 단계 수. 한 단계는 (state, action, next state, reward)의 튜플 입니다.                                                                                                                                                             |
| throttle             | Float                    | 초당 미터 단위의 자동차 속도. 이것은 선택된 Action Space로 연관 되어 있습니다.                                                                                                                                                                        |
| steering             | Float                    | Radian 단위의 원하는 자동차의 조향각도. 이것은 선택된 Action Sapce와 연관 되어 있지만 degree가 아니라 Radian으로 표현 됩니다. + 각도는 왼쪽으로 가고 음수 각도는 오른쪽으로 진행 함을 나타냅니다. 이것은 2D 기하학적 프로세싱과 관련 있습니다.
| track_width          | Float                    | 트랙의 폭 (단위 Meter)                                                                                                                                                                                                                                                          |
| waypoints            | List                     | 트랙 중앙을 따라 있는 Waypoints (x,y) 좌표의 리스트. Waypoints는 0부터 시작함.                                                                    |
| closest_waypoint     | Integer                  | 직선 거리로 측정 한 가장 가까운 Waypoints의 인덱스. 이 Waypoint는 차의 뒤 또는 앞쪽에있을 수 있습니다.                                                                                                                                                          |

보상 함수의 매개 변수에 대한 그림입니다.

![rewardparams](img/reward_function_parameters_illustration.png)

아래는 re:Invent track 에서 사용 되는 waypoints에 대한 그림입니다. 보상 함수에서는 중심선 중간 지점의 waypoints만 사용 하실 수 있습니다. 보상 함수에 waypoints는 목록을 인쇄 한 다음 Plot하여 그래프를 작성 할 수 있습니다. 보상 함수에서 인쇄 기능을 사용하면 출력이 AWS RoboMaker 로그에 저장됩니다. DeepRace에서 제공 되는 다른 트랙들도 동일하게 인쇄해 볼 수 있습니다. 로그들에 대해서는 다음에 논의 할 것입니다.

![waypoints](img/reinventtrack_waypoints.png)

보상 함수 설계하는 방법은, 자동차가 잘 운전 될거라 생각되는 행동에 대해 생각하는 것입니다. 간단한 예로 도로에 머무르는 차량에 대해 보상하는 것입니다. 이는 reward = 1 로 설정하는 할 수 있을 것입니다. 이 방식은 시뮬레이터에서 잘 동작을 할 것입니다. 왜냐하면 자동차가 트랙에서 벗어 났을 때 리셋하고 자동차가 원점에서 다시 시작하기 때문에,  트랙에서 벗어 낫을 경우 보상하는 것을 걱정 할 필요가 없기 때문입니다. 그러나 이것은 좋은 보상 함수를 만드는 데 사용할 수있는 다른 모든 변수를 완전히 무시하기 때문에 아마도 최상의 보상 기능은 아닐 것입니다.

아래에서는 보상 함수 몇 가지를 예제를 제공합니다.

**Example 1**: 중심선을 따라가도록 장려하는 보상 함수. 

여기에서는 먼저 3 개의 마커를 사용하여 트랙 주위에 3 개의 밴드를 만든 다음 중간 또는 넓은 밴드가 아닌 좁은 밴드에서 주행 했을 경우 더 많은 보상을 합니다. 보상의 크기 차이에 유의하십시오. 우리는 좁은 밴드에 머무를 때 1, 중간 밴드에 머무를 때 0.5, 넓은 밴드에 머무를 때 0.1의 보상을 제공합니다. 좁은 밴드에 대한 보상을 줄이거나 중간 대역에 대한 보상을 높이면 본질적으로 차량이 트랙의 넚은 면을 사용하도록 유도합니다. 이것은 특히 날카로운 모서리가있을 때 편리 할 수 있습니다.

	def reward_function(on_track, x, y, distance_from_center, car_orientation, progress, steps, throttle, steering, track_width, waypoints, closest_waypoint):
		marker_1 = 0.1 * track_width
		marker_2 = 0.25 * track_width
		marker_3 = 0.5 * track_width

		reward = 1e-3
		if distance_from_center <= marker_1:
			reward = 1
		elif distance_from_center <= marker_2:
			reward = 0.5
		elif distance_from_center <= marker_3:
			reward = 0.1
		else:
			reward = 1e-3  
		return float(reward)
        

Hint: 보상을 0으로 제공하지 마십시오. 주어진 보상이 0 일 때 우리가 사용하고있는 특정 최적화 도구가 어려움을 겪습니다. 따라서 우리는 보상을 작은 값으로 초기화합니다

**Example 2**:과도한 조향을 처벌하고 중심선을 따라 가도록 유도하는 고급 보상 함수.


	def reward_function(on_track, x, y, distance_from_center, car_orientation, progress, steps, throttle, steering, track_width, waypoints, closest_waypoint):

		import math
		marker_1 = 0.1 * track_width
		marker_2 = 0.25 * track_width
		marker_3 = 0.5 * track_width

		reward = 1e-3
		if distance_from_center <= marker_1:
			reward = 1
		elif distance_from_center <= marker_2:
			reward = 0.5
		elif distance_from_center <= marker_3:
			reward = 0.1
		else:
			reward = 1e-3 

		# penalize reward if the car is steering way too much
		# steering angle is in radians in the reward function
		# assumes your action space maximum steering angle is 30 and you have a steering granularity of at least 5. We will penalize any steering action that requires more than 15 degrees, absolute.
		ABS_STEERING_THRESHOLD = math.radians(15)
		if abs(steering) > ABS_STEERING_THRESHOLD:
			reward *= 0.5

		return float(reward)        

**Example 3**:천천히가는 것에 불이익을주고 중심선을 따라가도록 유도하는 고급 보상 함수.


	def reward_function(on_track, x, y, distance_from_center, car_orientation, progress, steps, throttle, steering, track_width, waypoints, closest_waypoint):

		import math
		marker_1 = 0.1 * track_width
		marker_2 = 0.25 * track_width
		marker_3 = 0.5 * track_width

		reward = 1e-3
		if distance_from_center <= marker_1:
			reward = 1
		elif distance_from_center <= marker_2:
			reward = 0.5
		elif distance_from_center <= marker_3:
			reward = 0.1
		else:
			reward = 1e-3 

		# penalize reward for the car taking slow actions
		# throttle is in m/s
		# the below assumes your action space has a maximum speed of 5 m/s and speed granularity of 3
		# we penalize any throttle less than 2m/s
		THROTTLE_THRESHOLD = 2
		if throttle < THROTTLE_THRESHOLD:
			reward *= 0.5

		return float(reward)

이제 위 예제를 사용하여, 보상 함수를 개발 해 봅니다. 다음은 몇 가지 추가 Tip 입니다:
- waypoints 를 사용하여 한 waypoint 에서 다음waypoint 방향을 계산할 수 있습니다.
- 2D 게임의 right-hand rule 을 사용하여 현재 트랙의 어느쪽에 있는지 확인할 수 있습니다
- 보상을 기하 급수로 조정할 수 있으나 최대 10,000으로 제한하십시오.
- 보상 함수에서 throttle 과 steering을 사용할 때 Action Space를 염두에 두십시오
- 파이선에서 degree를 radians로 변환하기 위해서 math.radians를 import하고 math.radians(원하는 각도)를 사용합니다
- 자동차가 완주 할 경우의 episode를 로그에서 추적하려면 Finish 보너스 (일명 rewards + = 10000)을 부여하는 것이 고려해 볼 수 있습니다. 자동차가  완주 하더라도 더 100%이상 진전이 없지만, 시뮬레이션은 계속 진행이 될것입니다. 이 모델은 Stop time에 도달 할 때까지 계속 훈련 하게 됩니다. 하지만 최종 모델이 더 한다고 하여 최고의 모델이라는 것을 의미하지는 않습니다. 특히 현실 세계에서의 레이싱과 관련해서는 그렇습니다. 이것은 우리가 앞으로 해결해야할 임시 방법입니다.

보상 함수를 만들고 나면 ** Validate ** 버튼을 사용하여 교육을 시작하기 전에 코드 구문이 올바른지 확인하십시오. 교육을 시작하면이 보상 함수가 S3의 파일에 저장되지만 안전하게 별도로 복사 및 저장하도록 합니다.

위의 첫 번째 예제를 사용한 보상 함수가 있습니다

![rewardfunction](img/reward_function.png)

다음 섹션으로 스크롤 하세요. 
