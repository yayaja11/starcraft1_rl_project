# Starcraft1 RL Project "Random Bunker Defense" 

##### 시작에 앞서, 본 프로젝트는 [TorchCraft][torch], [BWAPI][bwapi], [SAIDA][saida]의 코드를 참고하여 만들어졌습니다. 

##
##
## 목차
### [문제 정의](#문제-정의)
### [환경 설정 및 구조](#환경-설정-및-구조)
### [설계 이슈](#설계-이슈)
### [모델 이슈](#모델-이슈)
##
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




   [saida]: <https://github.com/TeamSAIDA/SAIDA_RL>
   [torch]: <https://github.com/TorchCraft/TorchCraft>
   [bwapi]: <https://github.com/bwapi/bwapi>
