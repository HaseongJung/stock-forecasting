# APPL 주가예측
### 📖 내용

- 목표: 오늘까지의 데이터를 통해 내일의 APPL 종가를 예측해보자.
- APPL의 최근 20년의 hitorical data를 기본으로 하고,  어떤 변수들을 추가하였을 때 학습 성능이 향상되는지 확인한다.
- KNN, Randomforest, LSTM 모델들로 각각 학습 및 예측 후 각 모델의 성능을 확인한다.
- 학습과정에서 각 모델들의 최적의 파라미터를 찾는다. (Manual Search 및 GridSearchCV)

### 🙋‍♂️ 역할

- KNN regressor, Randomforest regressor 모델을 사용하여 APPL의 주가를 예측한다.
- APPL의 historical data(시가, 저가, 고가, 종가, 거래량)을 제외한 학습 성능을 향상시키는 변수를 모색한다.
- Manual Search 및 GridSearchCV를 활용하여 최적의 파라미터를 찾는다.

---

## Dataset

yfinance라이브러리를 통하여 apple의 historical data(시가, 저가, 고가, 종가, 거래량) 및 추가적인 변수(S&P500, VIX, Crude Oil 등)들의 최근 20년 동안의 데이터를 사용한다.

### Raw dataset (Open, High, Low, Close, Volume)

![historical_data](https://github.com/HaseongJung/stock-forecasting/assets/107913513/617e0ce2-df56-4104-86c6-7e3bc23a3141)

Raw dataset Table

![addedData_heatmap](https://github.com/HaseongJung/stock-forecasting/assets/107913513/28c87652-8fa8-4b33-ad72-125bc677d5c2)

Raw dataset Correlation Coefficient Heatmap

![chart](https://github.com/HaseongJung/stock-forecasting/assets/107913513/e6413a08-4d8c-410f-bf40-3db7445e949d)

Raw dataset Chart

![RawData_distribution](https://github.com/HaseongJung/stock-forecasting/assets/107913513/e2dd587c-bf8d-4ec0-85e8-c4d60064282a)

Raw dataset Distribution of each columns

### **고려한 변수**

- *Dow futures*
- *S&P 500*
- *S&P 500 futures*
- *Nasdaq 100*
- *Nasdaq futures*
- *VIX*

- *Treasury interest rates(5W)*
- *Treasury interest rates(5Y)*
- *Treasury interest rates(10Y)*
- *US dollar index*
- *Crude Oil*

직접 여러 케이스들을 조합하여 학습 및 테스트 후 성능 향상에 도움이 되는 변수들 선별한 후 새로운 데이터셋을 만든다.

### Added Dataset (Raw dataset + S&P500 futures,Nasdaq futures,VIX,Crude Oil)

![addedData](https://github.com/HaseongJung/stock-forecasting/assets/107913513/cc9b00e6-af11-43ab-b3c6-90427e6c32f9)

Added dataset Table

![addedData_heatmap](https://github.com/HaseongJung/stock-forecasting/assets/107913513/28c87652-8fa8-4b33-ad72-125bc677d5c2)

Added dataset Correlation Coefficient Heatmap

![addedData_distrribution](https://github.com/HaseongJung/stock-forecasting/assets/107913513/a363c436-ef17-4348-916b-6cb3f7f1fa87)

Added dataset Distribution of each columns

- 일반적으로 feature들 간의 공분산이 높으면 redundant한 feature이므로 제거하곤 한다.
- 그러나 OLHC는 주가 예측 모델에서 흔히 쓰는 변수들이므로 제거하지 않기로 한다.
- S&P500 futures와 Nasdaq futures 간의 공분산도 높지만, 이 변수들은 애플을 포함한 전반적인 미국 주식 시장을 가장 잘 대변하는 지수이므로 이들 또한 제거하지 않기로 한다.
- `Volume`의 분산이 다른 변수들에 비해 너무 크기 때문에 학습을 하려면 Scaling이 필요하다.

## 예측

- Train data: `2003~2022년도 사이의 데이터`
- Test data: `2023년도의 데이터`

### KNN

![KNN1](https://github.com/HaseongJung/stock-forecasting/assets/107913513/2d228b6d-c2c9-4f9f-91ad-c8a722d00c5d)

KNN (neighbors=5, Basic features

![KNN2](https://github.com/HaseongJung/stock-forecasting/assets/107913513/a067076d-15c6-4243-b72f-a4b609cef169)

KNN (neighbors=5, Added features)

![KNN3](https://github.com/HaseongJung/stock-forecasting/assets/107913513/ae8ed73b-d6f0-4ad8-83e2-1dc745e0dc5f)

KNN (neighbors=50, Added features, GridSearch)

| Model | MAE | RMSE | R^2 |
| --- | --- | --- | --- |
| neighbors=5, Basic features | 4.9101982518246325 | 54.9869701325532 | 0.8383529486339698 |
| neighbors=5, Added features | 5.60815598443644 | 58.88167160741278 | 0.8280603005724001 |
| neighbors-50, Added features, GridSearchCV | 5.60815598443644 | 58.88167160741278 | 0.8280603005724001 |
- 성능 지표 상으로는 Basic features모델이 가장 좋으나 후행성이 매우 커 신뢰도가 가장 떨어진다. **feature들을 추가하였을 때 후행성이 비교적 사라진 모습**을 보인다.
- Manual Search와 GridSearch결과의 하이퍼 파라미터로 예측한 **결과 똑같은 성능을 보인다. Manual Search의 결과인 `neighbors=5`가 비교적 연산량에서 유리하다.**
- 하지만 공통적으로 test시점이 **학습된 시점과 멀어질수록 후행성이 심해지다가 약 5개월 후 시점부터 성능이 현저히 감소**하는 모습이다.

### Radnomforest

![RF1](https://github.com/HaseongJung/stock-forecasting/assets/107913513/e5549a1b-486d-40d7-92d4-d55274685b04)

Randomforest (Basic features)

![RF2](https://github.com/HaseongJung/stock-forecasting/assets/107913513/0ad122e7-b5b5-4b93-a297-7bffc7d443bf)

Randomforest (Added features)

![RF3](https://github.com/HaseongJung/stock-forecasting/assets/107913513/65032d4d-5891-497c-8bf9-80f2b66c137a)

Randomforest (Added features, GridSearch)

| Model | MAE | RMSE | R^2 |
| --- | --- | --- | --- |
| Basic features |  5.148607306575143 | 61.796604631425865 | 0.8195484378090253 |
| Added features |  4.734446543005129 | 59.558043154886455 | 0.8260852357109709 |
| Added features, GridSearchCV |  4.884572316403283 |  62.66058576822016 | 0.8170255363200529 |
- 약 5개월 이후부터 심한 성능 감소를 고려하여 성능 지표를 배제하고 시각화 자료만 본다면 **KNN모델보다 Randomforest모델의 예측 성능이 더 좋은 것으로 보인다.**
- Radomforest모델 또한 Basic features에서는 후행성이 매우 크고, **feature들을 추가하였을 때 비교적 후행성이 감소**한 모습이다.
- 하지만 Randomforest모델 역시 **test시점이 학습된 시점과 멀어질수록 후행성이 커지다가 약 5개월 후부터 성능이 현저히 감소**하는 모습이다.

## 결론

- KNN rgressor모델보다 **Randomforest regressor모델의 성능이 주가 예측에 비교적 더 적합해 보인다.**
- 기본적인 Historical data(시가, 저가, 고가, 종가, 거래량)의 데이터만을 사용하여 예측하였을 때는 후행성이 문제가 심각하다. 이를 **추가적인 변수(S&P500 futures,Nasdaq futures,VIX,Crude Oil)를 추가함으로써 후행성 문제를 조금이나마 완화할 수 있었다.**
- 학습된 시점과 Test시점이 멀어질수록 후행성이 심해지다가 예측 성능이 완전히 감소하는 현상이 나타난다. 우리의 목표인 ‘오늘까지의 데이터로 내일의 종가를 예측하자!’에는 영향이 없어 보이나, **기간을 나눠 test결과를 확인하는 방법보다 Back-testing을 하여 검증하는 것이 더 적합해보인다.**
- Manual-Search방식과 GridSearch방식으로 찾은 최적의 하이퍼파라미터의 결과들에 큰 차이가 없어,  **상황마다 소요 시간이 다르기 때문에 각 상황에 맞는 방식을 사용하는 것이 적합해 보인다.**
