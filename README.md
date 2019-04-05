## Step 3: Create model
이 페이지에서는 AWS DeepRacer용 RL 모델을 만들고 모델 교육을 시작하는 것을 배웁니다. 이 페이지는 몇 개의 섹션이 있습니다. LAB을 시작하기 앞서 먼저 스크롤 다운 하여 어떤 내용들이 있는지 확인해 봅니다. 여기서는 AWS DeepRacer 자동차가 스스로 트랙에서 자율 운행 하기 위해 사용 될 모델을 만들 것입니다. 그러기 위해서 제일 먼저 레이스 트랙을 선택하고, 모델이 선택 할 수 있는 행동을 정의하고, 원하는 운전 행동을 장려하기 위해 사용할 보상 함수을 디자인 하고, 훈련 중에 사용되는 하이퍼파라미터를 조정 할 것입니다. 

### <font color=blue>**Info**</font> **Buttons**
콘솔 안에서 <font color=blue>**Info**</font>  해당 버튼을 누르면, information 창이 스크린 오른 쪽에서부터 앞으로 나타 나게 됩니다.  Info 창은 창 안에서 별도 링크를 선택하지 않는 한 그대로 오른편에 남아있게 됩니다. 내용을 다 읽으신 뒤에는 창을 닫을 수 있습니다.

## 3.1 Model details
먼전 맨 위의 Model details부터 시작 합니다. 여기서는 모델 이름을 지정하고 모델에 대한 설명을 입력합니다. 서비스를 처음 사용하는 경우 **Create Resources** 버튼을 선택합니다. 이렇게하면 AWS DeepRacer가 사용자를 대신하여 다른 AWS 서비스를 호출하는 데 필요한 IAM 역할, 훈련 및 검증에 사용되는 VPC 스택, 보상 함수를 검증하기 위해 Python 3로 쓰여진 AWS DeepRacer 람다 함수, 그리고 모델 아티팩트가 저장 될 AWS DeepRacer S3 버킷이 생성됩니다. 이 섹션에서 오류가 나면 저희에게 알려주십시오.

![Model Details](img/model_details.png)
모델의 이름과 설명을 입력하고 다음 섹션으로 스크롤하십시오.

## 3.2 Environment simulation
워크샵에서 자세히 설명했듯이, RL 모델을 훈련하는 것은 시뮬레이션 된 레이스 트랙에서 이루어지게 됩니다. 이 섹션에서는 모델을 교육 할 트랙을 선택하게됩니다. AWS RoboMaker는 시뮬레이션 환경을 구성하는데 사용됩니다. 

모델을 훈련 할 때는, 실제로 레이스하기를 원하는 트랙을 미리 생각해 보고 최총적으로 레이스 할 트랙과 가장 가장 유사한 트랙을 선택 하여 훈련합니다. 이것은 모델의 성능을 보장하지는 않지만 모델이 레이싱장에서 최고의 성능을 발휘할 확률을 최대화합니다. 또한 직선 모양의 트랙을 선택 하여 훈련하는 경우 모델이 좌,우측 턴을 할거라는 기대는 하지 않는게 좋습니다. 

Section 2에서 AWS DeepRacer League에 대한 자세한 내용을 제공 할 예정이지만, 리그에서 경기하기 위해 훈련 할 트랙을 선택할 때는 유의해야 할 사항이 있습니다.

-  [Summit Circuit](https://aws.amazon.com/deepracer/summit-circuit/),실제 레이스 트랙은 Invent 2018 트랙이 될 것이므로 선택한 AWS Summit 중 어느 곳에서 경주를하려고한다면 re:Invent 트랙에서 모델을 훈련 시키십시오.

- 버추얼 서킷의 매번 새로운 경기용 트랙이 생기며, 이 경기용 트랙을 직접 훈련 할 수는 없습니다. 대신 테마와 디자인이 매 경기 트랙과 동일하지는 않지만 유사하게 할 것입니다. 이렇게 하려면 모델이 일반화 되어야하며, 경기 트렉에 Overfitting해서는 않됩니다.

오늘의 Lab에서는 시간이 가능한 한 최대로 여러 분들이 Summit에서 레이스를 참가를 위해 준비 될 수 있도록 도와 드릴 예정입니다. Rre:Invent 2018 트랙을 선택하고 다음 섹션으로 스크롤하십시오.

## 3.3 Action space
이 섹션에서는 훈련 과정 및 실 주행에서 선택 할 수 있는 Action Space를 정의합니다. Action은 자동차가 취할 수 있는 스피드와 조향각의 조합 입니다. 
AWS DeepRacer에서는 Continuous Action Sapcerk 아닌 Discrete Acation space를 사용합니다. 이 Discrete Action Space를 정의하기 위해 최대 속도(Maximum Speed), 속도 레벨(Speed Levels), 최대 조향 각도(Maximum Steering Angle), 그리고 조향 레벨 (Steering Levels) 을 지정하게됩니다.

![action space](img/Action_Space.png)

Inputs

- 최대 조향 각도는 차량의 앞 바퀴가 왼쪽과 오른쪽으로 회전 할 수있는 최대 각도입니다. 바퀴가 얼마나 멀리 회전 할 수 있는지에 대한 한계가 있으므로 최대 회전 각도는 30도입니다.
- 조향 레벨은 양측의 최대 조향 각도 사이의 조향 간격의 수를 나타냅니다. 따라서 최대 조향 각도가 30 도인 경우 + 30 도가 왼쪽이고 -30 도가 오른쪽입니다. 조향 레벨이 5 인 경우 왼쪽에서 오른쪽 방향으로 30도, 15도, 0도, -15도 및 -30 도의 조향 각도가 동작 공간에 표시됩니다.  조향 각도은 언제나 0도를 기준으로 대칭입니다.
- 최대 속도는 자동차가 시뮬레이터에서 운전할 최대 속도를 m/s 로 측정 한 것입니다.
- 속도 레벨은 최대 속도(포함)에서 0까지의 속도 레벨의 개수를 나타냅니다. 따라서 최대 속도가 3m/s이고 속도 레벨이 3 인 경우 Action Space 에 1 m/s, 2m/s, and 3 m/s의 속도 설정이 포함됩니다. 간단히 3m/s 를 3으로 나누면 1m/s가 되고,  0m/s 에서 3m/s로 1m/s씩 증가합니다. 0m/s는 Action Space에 포함되지 않습니다.

Based on the above example the final action space will include 15 discrete actions (3 speeds x 5 steering angles), that should be listed in the AWS DeepRacer service. If you haven't done so please configure your action space. Feel free to use what you want to use. Larger action spaces may take a bit longer to train.

Hints

- Your model will not perform an action that is not in the action space. Similarly, if your model is trained on a track that that never required the use of this action, for example turning won't be incentivized on a straight track, the model won't know how to use this action as it won't be incentivized to turn. Thus as you start thinking about building a robust model make sure you keep the action space and training track in mind.  
- Specifying a fast speed or a wide steering angle is great, but you still need to think about your reward function and whether it makes sense to drive full-speed into a turn, or exhibit zig-zag behavior on a straight section of the track.
- For real world racing you will have to play with the throttle in the webserver user interface of AWS DeepRacer to make sure your car is not driving faster than what it learned in the simulator.


## 3.4 Reward function
In reinforcement learning, the reward function plays a **critical** role in training your models. The reward function is used to incentivize the driving behavior you want the agent to exhibit when using your trained RL model to make driving decisions. 

The reward function evaluates the quality of an action's outcome, and reward the action accordingly. The reward is calculated in the simulation, during training, after each action is taken. You will provide the logic that goes into the reward function, using Python 3 syntax. In order to code this logic you have a number of measurements available from the simulation, exposed as variables.

The following list contains the variables you can use in your reward function. Note these are updated from time to time as our engineers and scientists find better ways of doing things, so adjust your previous reward functions accordingly. At the time of the Santa Clara workshop these variables and descriptions are correct in the AWS DeepRacer service in the AWS console. Thus please ignore the descriptions you may read in the AWS DeepRacer service, and make sure you use these variable names and descriptions. The most prominent difference is throttle and steering.


| Variable Name        | Type                     | Description                                                                                                                                                                                                                                                                                         |
|----------------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| on_track             | boolean                  | If any of the four wheels is on the track, track defined as the road surface and the border lines, then one_track is True. If all four wheels is off the track, then on_track is False. As soon as on_track is False the car will reset.                                                            |
| x                    | Float                    | Returns the x coordinate of the center of the from axle of the car, in unit meters.                                                                                                                                                                                                                 |
| y                    | Float                    | Returns the y coordinate of the center of the from axle of the car, in unit meters.                                                                                                                                                                                                                 |
| distance_from_center | Float [0, track_width/2] | Absolute distance from the center of the track. Center of the track is determined by the line that links all center waypoints.                                                                                                                                                                      |
| car_orientation      | Float (-pi,pi]           | Returns the direction the car is facing in radians. When car faces in direction of x increasing, with y constant, then return 0. When car faces in direction of y increasing, with x constant, then returns pi/2. When car faces in direction of y decreasing, with x constant, then returns -pi/2. |
| progress             | Float [0,100]              | Percentage of the track complete.                                                                                                                                                                                                                                                                   |
| steps                | Integer                  | Number of steps completed. One step is one (state, action, next state, reward tuple).                                                                                                                                                                                                               |
| throttle             | Float                    | The desired speed of the car in meters per second. This should tie back to the selected action space.                                                                                                                                                                                                       |
| steering             | Float                    | The desired steering of the car in radians. This should tie back to the selected action space, but using radians as opposed to degrees. Note that + angles indicate going left, and negative angles indicate going right. This is aligned with 2d geometric processing.                                                                                                                                                                    |
| track_width          | Float                    | The width of the track, in unit meters.                                                                                                                                                                                                                                                             |
| waypoints            | List                     | Ordered list of waypoints along the center of the track, each item containing the (x, y) coordinates of the waypoint. The list starts at zero                                                                                                                                                        |
| closest_waypoint     | Integer                  | Index of the closest waypoint, as measured by straight-line distance. This waypoint can be behind or in-front of the car.                                                                                                                                                                           |

Here is a visual explanation of some of the reward function parameters.

![rewardparams](img/reward_function_parameters_illustration.png)

Here is a visualization of the waypoints used for the re:Invent track. You will only have access to the centerline waypoints in your reward function. Note also that you can recreate this graph by just printing the list of waypoints in your reward function and then plotting them. When you use a print function in your reward function, the output will be placed in the AWS RoboMaker logs. You can do this for any track you can train on. We will discuss logs later.

![waypoints](img/reinventtrack_waypoints.png)

A useful method to come up with a reward function, is to think about the behavior you think a car that drives well will exhibit. A simple example would be to reward the car for staying on the road. This can be done by setting reward = 1, always. This will work in our simulator, because when the car goes off the track we reset it, and the car starts on the track again so we don't have to fear rewarding behavior that leads off the track. However, this is probably not the best reward function, because it completely ignores all other variables that can be used to craft a good reward function.

Below we provide a few reward function examples. 

**Example 1**:Basic reward function that promotes centerline following.
Here we first create three bands around the track, using the three markers, and then proceed to reward the car more for driving in the narrow band as opposed to the medium or the wide band. Also note the differences in the size of the reward. We provide a reward of 1 for staying in the narrow band, 0.5 for staying in the medium band, and 0.1 for staying in the wide band. If we decrease the reward for the narrow band, or increase the reward for the medium band, we are essentially incentivizing the car to be use a larger portion of the track surface. This could come in handy, especially when there are sharp corners.


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
        

Hint: Don't provide rewards equal to zero. The specific optimizer that we are using struggles when the reward given is zero. As such we initialize the reward with a small value. 

**Example 2**:Advanced reward function that penalizes excessive steering and promotes centerline following.


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

**Example 3**:Advanced reward function that penalizes going slow and promotes centerline following.


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

Using the above examples you can now proceed to craft your own reward function. Here are a few other tips:

- You can use the waypoints to calculate the direction from one waypoint to the next.
- You can use the right-hand rule from 2D gaming to determine on which side of the track you are on.
- You can scale rewards exponentially, just cap them at 10,000.
- Keep your action space in mind when using throttle and steering in your reward function
- To go from degrees to radians in python import math and then use math.radians(the degrees you want)
- To keep track of episodes in the logs where your car manages to complete a lap, consider giving a finish bonus (aka reward += 10000) where progress = 100. This is because once the car completes a lap progress will not go beyond 100%, but the simulation will continue. The model will keep on training until it reaches the stopping time, but that does not imply the final model is the best model, especially when it comes to racing in the real world. This is a temporary workaround as we will solve.

Once you are done creating your reward function be sure to use the **Validate** button to verify that your code syntax is good before training begins. When you start training this reward function will be stored in a file in your S3, but also make sure you copy and store it somewhere to ensure it is safe.

Here is my example reward function using the first example above.

![rewardfunction](img/reward_function.png)

Please scroll to the next section.
