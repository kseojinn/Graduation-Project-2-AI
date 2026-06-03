# 전공종합설계(2) [004] - 종합설계보고서

**지도교수: 강영명 교수님**

**학생: 강서진** (성결브레인, 컴퓨터공학과 20230812)

안녕하세요. 저는 성결브레인 팀에서 팀장과 인공지능 엔지니어를 맡고 있는 강서진입니다. 이 팀이 만들어지기 초반부터 올바른 방향으로 갈 수 있도록 이끌며, 팀원들이 의견을 제시하며 활발하게 이야기 주고 받을 수 있는 환경을 조성하였습니다. 최대한 팀원들의 의견을 수용하였고 그 결과 탄탄한 시스템으로 이어질 수 있게 되어 기쁩니다.

---
## 저가, 저전력 모듈을 통한 AI 기반 도로결빙 감지 시스템

**팀원 및 역할**

- **강서진: 팀장, 인공지능 엔지니어**
- 이재호: 프로젝트 매니저, 임베디드 엔지니어
- 박경준: 벡엔드 서버 엔지니어
- 박동현: 프론트엔드 서버 엔지니어

**기여**

저희 시스템은 이재호 팀원의 아이디어로 시작하여, 그 만큼 많은 기여를 하였습니다. 저를 포함하여 나머지 팀원들은 이 아이디어가 꽃을 피울 수 있게 지원하였습니다.

**시스템 소개**

저희 시스템은 저가, 저전력 모듈을 통한 AI 기반 도로결빙 감지 시스템입니다. 저가, 저전력이란 중심적인 목적을 두었으며 저렴한 센서들이 탑재된 자체 제작 모듈, 랜덤포레스트 분류기를 활용한 경량 AI 모델로써 이를 설명합니다. 서비스는 웹을 통해 도로 결빙 여부를 사용자에게 제공합니다.

---
## AI 시스템

AI 시스템은 다음과 같이 진행되었습니다.

**AI 시스템 FLOW**

1. **노면 전극 데이터 생성**

    본래의 저희 계획은 2025년도 2학기 겨울이 오기 전에 자체 제작 모듈을 완성한 후 모델에 필요한 학습 데이터를 수집할 계획이었으나, 모듈 제작이 미루어져 수집을 못하게 되었습니다. 최대한 실세계의 데이터를 수집하려고 많은 노력을 기울였으나, 세 개의 라벨 dry, wet, frozen 전극 데이터 값이 확실히 구분되어 데이터 수집없이 만들었습니다. 이렇게 생성한 데이터를 모델 학습에 활용하였습니다.

    **매개변수**
    - time_hour: 시간
    - temperature: 온도
    - humidity: 습도
    - conductivity: 노면 전극

    **결과**: 3가지로 구분
    - 안전
    - 눈길(위험)
    - 빗길(조심)

2. **AI 모델 학습**

    저희 모델은 데이터 특성상 전극 값의 구분이 확실하기 때문에 랜덤포레스트 분류기가 훌륭한 성능을 발휘할 것이라 판단하여 이를 기반으로 하여 학습을 진행하였습니다. 예상한 결과, 정확도 또한 1에 가까운 성능을 보였습니다.

3. **모델 적용**

    모델을 저희 시스템에 적용시키기 위해 파이썬을 기반하여 코드를 작성하였습니다. 

    **모델 사용에 관해**

    ```
    # 필요한 라이브러리 설치
    pip install -r requirements.txt
    ```

    ```
    # 모델 로드
    loaded_model = None
    model_path = 'conductivity-prediction-model.pkl'
    try:
        loaded_model = joblib.load(model_path)
        print("200 OK: File imported successfully")
    ```

    ```
    # 새로운 데이터 예측 함수 정의
    # 시간, 온도, 습도, 전도도 값을 입력받아 도로 상태를 예측
    def predict_road_status(hour, temp, humi, cond):
        # 모델 학습 시 사용했던 특성 순서대로 정렬 (time_hour, temperature, humidity, conductivity)
        new_data = pd.DataFrame([[hour, temp, humi, cond]],
        columns = ['time_hour', 'temperature', 'humidity', 'conductivity'])

        # 예측 수행
        prediction = loaded_model.predict(new_data)

        return prediction[0]
    ```


    ```
    # 측정값 대입
    status = predict_road_status(hour=21, temp=10, humi=100, cond=130)
    ```

    Conductivity의 대략적 기준

    - Dry: 850 ~ 1000 -> "안전"
    - Frozen: 150 ~ 850 -> "눈길(위험)"
    - Wet: 0 ~ 100정도 -> "빗길(조심)"

**코드 파일**

```
Graduation-Project-2-AI/
├── conductivity-prediction-model.pkl       # 생성한 데이터로 학습된 모델
├── dataset-generation.ipynb                # 학습에 필요한 데이터 생성 & 모델 학습
├── road-icing-detection.ipynb              # 모델 적용 코드
└── README.md
```