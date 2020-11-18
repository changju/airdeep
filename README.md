# Air Deep API Documentation
##  작성 이력
| 날짜 | 내용 | 작성자 |
|:--------:|:---------:|:-------|
|2020-11-17| 최초 작성 | 이창주 |
|          |           |        |
</br>

## 1. 소개
### 1.1 목적
    본 문서의 내용은 AQS 서비스연동 기능을 제공하기 위한 규격이다. 

### 1.2 적용 범위 
     본 문서의 내용은 AQS 서비스 제공 계획의 변경 및 관련 표준 규격의 갱신등에 다라 추후 보완 및 변경 될 수 있다.
     본 문서에서 기술되지 않은 부분에 대한 기능 구현 시 반드시 협의하여 구현하여야 한다.

### 1.3 프로토콜
      - HTTP/1.1

### 1.4 용어 정의 및 약어
      - API : Application Program Interface
      - AQS : Air Quality Services
      - REST : Representational State Transfer

****
</br>

## 2. 기본 정보
### 2.1 서버 정보
  | 서버 | URL |   설명 |
  |:-----|:----|:-------|
  |테스트 서버|https://stream.mobideep.co.kr/test| 테스트 서버의 URL | 
  |상용 서버|https://stream.mobideep.co.kr/prod| 상용 서버의 URL | 
  ||||
****
</br>

## 3.  서비스 연동
### 3.1 서비스 연동 콜 플로우
<img src="https://github.com/changju/airdeep/raw/master/airdeep-seq-1.jpg" width="60%" title="air deep flow diagram" alt="Airdeepflow"></img>
<br/>

### 3.2 서비스 연동 API
* API Interface format
  -  AQS 연동 API 의 Request URI 는 다음의 형태를 갖는다.
      | HTTP Method | Request URI |
      |:--------:|:---------:|
      |POST, PUT, GET|/ {Server domain} / {Distribution type} / {Distribution version} / {Service type} / {Service Version} / {Root API Type} / {Sub API Type} |
      |          |           | 

#### 3.2.1 인증
* Request URl
  ```
  GET /stream.mobideep.co.kr/prod/v1/auth/{clientName}?userName={userName}&userPassword={userPassword}
  ```
</br>

* HTTP Request Header 
   ```
   Content-Type : application/json
   ```
</br>

* Path 파라미터 설명
  
  | Name | 설명 | Value |
  |:-----|:----|:-------|
  |clientName|고객 이름|(예) airdeep-lge | 
  ||||
</br>

* Query 파라미터 설명
  | Name | 설명 | Value |
  |:-----|:----|:-------|
  |userName|유저 이름|(예) user-airdeep-lge |
  |userPassword|유저 패스워드|(예) password-airdeep-lge |
  ||||
</br>

* HTTP Response Body
  ```
    {
       "token": "[token]",
       "expiresIn": [Second]
    }
  ```
  </br>

 * 각 파라미터 설명
      | 파라미터 |  타입 | 설명 |
      |:--------|:---------|:---------|
      |token| String | API에서 사용할 토큰, 이 값은 이 후 모든 API요청시 Authorization 헤더에 포함하도록 한다.   |
      |expiresIn| Integer | 토큰 만료 기간. 단위는 초이다. 기본적으로 3600초(=1시간) 이다.</br>API Call 할때마다 auth를 하는 것은 좋지 않으며 만료기간 직전에 다시 얻어서 사용하도록 한다. |
      ||| |
</br>

#### 3.2.2 센서 데이터 전달
* Request URl

      POST /stream.mobideep.co.kr/prod/v1/aqs/lge.device.sensor/{DEVICE-ID}

* HTTP Request Header 

      - Content-Type : application/json
      - Authorization : 인증 과정에서 얻은 API Token

* HTTP Request Body
  ```
    {
       "deviceId": "[DEVICE-ID]",
       "msgId": "[MESSAGE-ID]"
       "msgDate": "2020-11-11T14:54:05.349+09:00",
       "msgDateFormat": "yyyy-MM-dd HH:mm:ssZ",
       "type": 255,
       "data": "06221E0079096C0957063E0608000500090005000100",
       "ext": {
       } 
    }
  ```
 * Path 파라미터 설명
  
    | Name | 설명 | Value |
    |:-----|:----|:-------|
    | DEVICE-ID | 장치의 고유 UUID 값| (예) UUID | 
    |||| 
    <br>
 * 각 파라미터 설명

      | 파라미터 | 타입 | 설명 |
      |:--------|:---------|:---------|
      |deviceId| String |장치의 고유 UUID 값 |
      |msgId| String | 메시지 구분을 위한 ID, 응답으로 이 값을 전달한다. |
      |msgDate| Datetime | msgDateFormat 에 따른 메시지 생성 시간 |
      |msgDateFormat| String | msgDate 의 형식을 나타낸다. |
      |type| Integer | data 에 포함된 데이터 타입을 나타낸다. <br> sensor data: 255  |
      |data| String | 위 타입에 따른 데이터 값 |
      |ext| | 확장을 고려하여 사용 예정 |
      ||||
    <br>
* HTTP Response Body
  ```
    {
       "code": 200,
       "msgId": "[MESSAGE-ID]"
    }
  ```
 * 각 파라미터 설명

      | 파라미터 | 타입 | 설명 |
      |:--------|:---------|:---------|
      |code| Integer | 결과 전달, 200 (Success) |
      |msgId| String | 요청시 전달한 msgId 를 응답으로 전달한다. |
      ||||


#### 3.2.3 명령 전달
* AQS  탐지 코드 전달

      AQS 빅데이터 결과 값을 전달하기 위해 사용되는 API 이다.
* Request URl

      POST https://{Server URL}/prod/v1/aqs/lge.device.sensor/alert

* HTTP Request Header 

      - Content-Type : application/json
      - Authorization : 인증 과정에서 얻은 API Token

* HTTP Request Body
  ```
    {
       "clientId": "796b1d446b054d0b9e95191720d87528",
       "deviceId": "45bd4ac505ab5f1948cb9dc840e5dbb0",
       "detectionCode": "C110", 
       "msgId": "[MESSAGE-ID]",
       "msgDate":"2020-11-11T14:54:05.349+09:00",
       "msgDateFormat": "yyyy-MM-dd HH:mm:ssZ"
    }
  ```   


 * 각 파라미터 설명
      | 파라미터 | 타입 | 설명 |
      |:--------|:---------|:---------|
      |clientId| String | 탐지가 된 Client 의 고유 UUID 값 |
      |deviceId| String | 탐지가 된 Device 의 고유 UUID 값 |
      |detectionCode| String | 흡연 감지 코드.<br> “C110” : 연초 흡연이 탐지<br> “C120” : 궐련 흡연이 탐지 <br> “C210” : 공기질 나쁜 상황으로 탐지 <br> “C220” : 이상 공기 상황 탐지 |
      |msgId | String | 메시지 구분을 위한 ID, 응답으로 이 값을 전달한다. |
      |msgDate| Datetime | msgDateFormat 에 따른 메시지 생성 시간 |
      |msgDateFormat| String | msgDate 의 형식을 나타낸다. |
      ||| |
<br>

### 3.2.4 예외 시나리오
#### 3.2.4.1 요청 재시도 시나리오
  - Data Streamer는  실패시 아래와 같은 로직으로 Retry 시도를 한다.
   
    <img src="https://github.com/changju/airdeep/raw/master/airdeep-exception-1.jpg" width="70%" title="air deep flow diagram" alt="AirdeepExceptionFlow1"></img>
<br/>

  - AIR-DEEP은 요청 실패시 아래와 같은 로직으로 Retry 시도를 한다.<br>탐지 결과 수신측(DETECT RECEIVER) 에서 수신된 메시지의 시간이 2분을 초과한 경우 실패처리 한다.
  

    <img src="https://github.com/changju/airdeep/raw/master/airdeep-exception-2.jpg" width="70%" title="air deep flow diagram" alt="AirdeepExceptionFlow2"></img>
<br/>

****














