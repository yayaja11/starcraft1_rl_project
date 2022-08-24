# Starcraft1 RL Project "Random Bunker Defense" 

##### 시작에 앞서, 본 프로젝트는 [TorchCraft][torch], [BWAPI][bwapi], [SAIDA][saida], [세인트워터멜론님][web1]의 코드를 참고하여 만들어졌습니다. 

##
2022-04-03 Update 1: Dueling, Double Algorithm 추가
##
2022-08-06 Update 2: Parallel Learning 추가
##

#### 스타크래프트의 유즈맵 "랜덤 벙커 디펜스"를 플레이하는 RL 기반 봇

##
## 목차
### [1. 문제 정의](#문제-정의)
### [2. 환경 설정 및 구조](#환경-설정-및-구조)
### [3. 설계 이슈](#설계-이슈)
### [4. 모델 이슈](#모델-이슈)
### [5. 결과](#결과)
####
####
####
####
####
####
####
####
####
####
####
####
####
####
####
##

### 문제 정의 
- 무엇을 풀 것인가? 
  - 랜덤 벙커 디펜스6 
    - 35 스테이지로 이루어진 디펜스 게임
    - 플레이어는 벙커를 적재적소에 건설하고 업그레이드를 통해 마지막까지 살아남는 것이 목적
    - 벙커는 영웅벙커와 일반벙커가 있음
    - 가끔씩 벙커를 건설하면 추가 금액(행운)을 줌
    - 행운이라는 요소가 게임을 클리어하는데 중요한 요소로 작용
 
- 평범한 스타 유저들은 게임을 잘 할까?

![Brood-War-2022-03-21-22-43-56](https://user-images.githubusercontent.com/19571027/159280610-f2e81cc5-50de-44ec-93e3-c890538f4ef3.gif)
##### -> 26스테이지 사망
##
##
![2](https://user-images.githubusercontent.com/19571027/159283649-370b0cab-39a2-41cf-9af1-7fa236118888.gif)
##### -> 21스테이지 사망
##
- 여러판을 플레이해본 결과 클리어율은 약 10% 
- 숙련된 플레이어의 경우 행운에 따라 약 30%의 클리어율을 보임


###  환경 설정 및 구조

##### 환경 설정
- 윈도우 10
- 파이썬 3.6
- BWAPI 4.2
- Torchcraft 3.6
- Gym Environment
- GTX970 SUPERJET

##### 환경 구조
Starcraft ~ BWAPI ~ Torchcraft ~ Agent
<img src="https://user-images.githubusercontent.com/19571027/159266080-844e7d50-e479-4fa2-adbe-f26aa9cd9aa9.png" width="700" height="300"/>


### 설계 이슈
- state
  - 건물 위치, 벙커 종류, 돈, 가스, 행운, 시간, 업그레이드
- action
  - 특정 위치 (일반 or 영웅)벙커 건설,  업그레이드 
- discrete or continuous
  - 스타크래프트는 일반적으로 continuous임. 하지만 랜덤 벙커 디펜스의 경우 스테이지를 기준으로 행동이 구분되고 많은 상호작용이 필요없을 것이라 판단, discrete하게 환경 개발
- 맵 자체 버그
  - 미완성된 벙커의 오른편에 새로운 벙커를 지을 경우 돈이 무한이 됨
    - Agent에서 버그 악용 -> 맵 수정을 통해 버그 사용시 패배처리
  - 특정 위치에 건물을 건설할 경우 적 유닛 생성에 장애 발생
    - Agent에서 버그 악용 -> 해당 위치 건설 제한
- BWAPI 이슈
  - 유즈맵은 맵개발자 임의로 건물의 가격을 조정가능. 하지만 BWAPI는 건물 건설이 가능하더라도 사용자의 자원이 default 가격에 미치지 못하면 아무런 행동도 하지 않음
    - 맵 수정을 통해 전체 금액 상승

### 모델 이슈
- 1.ver
  - DQN으로 무작정 학습시켜보기
    - -> 잘안됨
  - 보상 조정
    - 처음에 죽을 경우 큰 패널티, 나중에 죽을 경우 작은 패널티
    - 행운이 나왔을 때 보상 (플레이어는 약간의 테크닉으로 행운 확률을 향상할 수 있음. 이걸 학습하는 것이 목적)
    - -> 잘안됨

- 2.ver
  - 벙커디펜스는 같은 자리에 벙커를 다시 지을 수 없음. 어떻게 이것을 학습시키지?
    - Agent에 이전 행동들을 기억하고 동일한 장소에 벙커를 지을 경우 패널티
    - -> 좋은 장소에 다시 지으려고 하기 때문에 학습 도중 과도한 패널티를 받음
    - -> 그 결과 학습시킨 가중치가 엉망이되어 아무곳에나 짓는 모습을 볼 수 있음
  - 무조건 패널티를 주는 것이 아닌, 중복이라는 state를 같이 전달하고 네트워크를 더 깊게 생성
    - -> 약 2000 에피소드까지 학습시켰으나 모델의 성능 향상이 없음

- 3.ver
  - Masking 기법 적용
    - 중복된 행동을 했을 때, 패널티를 주지 않음. 대신 action과 동일한 size의 mask를 생성하여 건설된 곳은 애초에 선택하지 않도록 함
    - -> 가중치의 저하 없이 올바른 action 수행
  - 보상 조정
    - Simple is Best
    - 복잡한 보상 체계 대신 오래버티면 더 높은 점수 부여
    - -> 성공적으로 학습

### 결과
  
- 그래프
![image](https://user-images.githubusercontent.com/19571027/160402835-de091e9b-1c79-4d05-83dc-14a3d3e22383.png)
  - 붉은색 선: 클리어 점수(목적치)
  - 푸른색 선: 점수
  - 점점 게임 실력이 향상되는 그래프
  - 최종적으로 35스테이지중 평균 30스테이지 클리어

- 클리어 확률
  - 최대 10게임중 7게임 클리어 (70%)
![image](https://user-images.githubusercontent.com/19571027/160403304-da319f67-497c-4e83-85c3-152fe33819ea.png)


- 최소 행운으로 클리어한 게임(운이 중요한 요소임에도 불구하고 실력으로 깼다는 의미)
  - 클리어한 게임의 평균 행운은 약 34회
  - 최소 행운으로 클리어한 게임의 평균 행운은 16회
 ![3](https://user-images.githubusercontent.com/19571027/160409713-3b33af97-5b33-48b7-bdb7-b375ab72de29.gif)
(이 배치가 최적에 가장 가까울지도?)

- 어떤 위치에 영웅벙커(대미지가 쌔지만 조금 더 비싼)를 더 많이 지었을까
    - 자리별 영웅 벙커를 지은 횟수: [0, 158, 333, 149, 258, 229, 381, 457, 195, 232, 188, 123, 67, 226, 143, 286, 152, 167, 110, 233, 176, 46, 257, 118, 822, 160, 261, 223, 672, 217, 138, 141, 278, 160, 159, 203, 187, 212, 559, 168, 177, 132, 268, 195, 246, 310, 136, 202, 509, 249, 156, 76, 400, 178, 225, 112, 733]
![영웅벙커순위](https://user-images.githubusercontent.com/19571027/160416163-5fd832ab-6a95-4cb0-8a3b-3f333b17b462.jpg)
      - 기대와는 살짝 다른 배치
      - 클리어한 경우도 살펴보자
##

   - (클리어한 게임의)자리별 영웅 벙커를 지은 횟수: [0, 24, 48, 28, 63, 49, 73, 98, 30, 33, 16, 32, 9, 26, 33, 62, 41, 29, 28, 41, 23, 3, 50, 17, 50, 32, 22, 61, 46, 49, 40, 30, 53, 9, 21, 16, 39, 41, 37, 38, 22, 16, 49, 28, 34, 46, 26, 47, 92, 49, 32, 12, 29, 26, 30, 9, 40]
![클리어영웅벙커](https://user-images.githubusercontent.com/19571027/160420415-ae037774-0c4c-4dae-bf53-5ab559824c6b.jpg)
      - (컴퓨터는 이 위치들이 주요 지점이라 생각한 것 같다)
      - 사람의 직관으론 사거리배치가 가장 중요할 것 같음
        - -> 위치는 크게 중요한 요소가 아니었을 수 있음
        - -> 학습이 덜 되었을 가능성도 존재
        - -> 나보다 컴퓨터가 똑똑한 걸 수도..

- 업그레이드의 중요성
  - 클리어하기위해서는 업그레이드는 평균적으로 7업~9업이 가장 많음
  - 6업도 깰 수는 있다. 하지만 평균 행운 34회보다 6회높은 40회일때 클리어가 가능함
  
- 컴퓨터는 사람과 다르게 시작하자마자 (딜레이없이) 행동을 취함으로써 동일한 행동이 가능함 = 랜덤 일부 극복
  - ex) 매 게임 시작하자마자 특정 위치에 벙커를 건설할 경우, 동일한 건물이 건설됨(매번 같은 속도로 명령할 수 없기 때문에 사람에게는 불가능)


   [saida]: <https://github.com/TeamSAIDA/SAIDA_RL>
   [torch]: <https://github.com/TorchCraft/TorchCraft>
   [bwapi]: <https://github.com/bwapi/bwapi>
   [web1]: <https://pasus.tistory.com/133>

