

* 용어에 대한 설명

어떠한 요청을 서버에게 보내는 과정을 Request라고 합니다.

해당 요청에 대한 결과를 Client에게 보내는 과정을 Response라고 합니다.

요청을 보내고 결과를 받는 쪽 = Cilent

요청을 받고 결과를 보내주는 쪽이 = Server

텔레그램 연동은 구조가 달라지게 됩니다.

Client 요청을 텔레그램 서버가 먼저 받습니다.

우리의 서버가 Telegram에게서 요청을 받아내려고 사용하는 방식이 Webhook입니다.

* pip3 install flask

* [Server.py]

```python
from flask import Flask

app = Flask(__name__)

@app.route('/', methods = ['GET']) ## GET, POST

def server():
  return 'Hello world'

if __name__ == '__main__': 
  app.run(debug = True, host = 'localhost', port = 5000)
```



localhost의 가장 큰 문제는?? 외부와 연결을 할 수 없습니다. 

루프백 주소 = 로컬 컴퓨터의 주소를 설명하는 기본 이름

해당 IP주소는 네트워크에서 다른 컴퓨터와 통신하는데 사용하는 IP가 아니라 단순히 로컬 컴퓨터의 주소 IP일 뿐입니다.

즉 해당 네트워크 안에서 내 컴퓨터를 지칭하는 IP라고 생각하시면 됩니다. 공개되지 않은 주소?

--------------------

### ngrok

기존 방식에서 dns주소를 매핑하려면 공유기에서 접속을 설정하고, 포트 포워딩을 하고 여러가지 설정을 해야하는데 머리가 아픈 경우가 많습니다,

이러한 포트 포워딩, 공유기 접속 설정 등 없이 해당 컴퓨터에서 열어둔 포트 [ex):5000 ]로 직접 접속할 수 있게 하는 방법이 있는데 그 중 많이 사용되는 툴이 바로 ngrok이라는 툴입니다.[ 원리는 신경쓰지 않고 그냥 바로 사용하시면 되겠습니다.]

다운로드 링크 : https://ngrok.com/download

1. ngrok을 다운로드 및 설치

2. ngrok을 실행

3. 수행 전 Flask 서버를 구동

   

   ```
   > ngrok http 5000 --region ap
   ```

4. 그러면 포트 포워딩된 주소가 나타납니다.

   ![스크린샷 2020-10-28 오전 11.25.06](/Users/JMacPro/Desktop/스크린샷 2020-10-28 오전 11.25.06.png)

   

   ### TIP : ngrok은 포트 포워딩된 주소는 생성된 후 8시간 동안만 유효합니다. 24시간 돌리는데는 매우 어렵습니다.. 개발용으로만 참고해주세요. 



-------------

## 텔레그램 API 사용하기

* API 참고 공식 문서 링크 : https://core.telegram.org/bots/api



1. 요청을 보내는 방법 [request]

![스크린샷 2020-10-28 오후 12.14.19](/Users/JMacPro/Library/Application Support/typora-user-images/스크린샷 2020-10-28 오후 12.14.19.png)

Thelegram Bot API에 보내는 모든 요쳥은 HTTPS방식으로 보내야 한다. 

GET과 POST methods를 제공한다. 파라미터를 전달하는 방법을 4가지를 아래 나열했습니다.



* Webhook을 걸어주는 Method를 찾아봅시다.![스크린샷 2020-10-28 오후 12.18.25](/Users/JMacPro/Library/Application Support/typora-user-images/스크린샷 2020-10-28 오후 12.18.25.png)
  1. Method Name : setWebhook
  2. 파라미터를 다 보내는 것이 아닙니다. 즉 Required에서 필수적으로 보내는 url만 필수적으로 넣어야 한다는 말

------------------

* 요청을 한곳에서 관리하기 위해서 config.py 파일을 작성
* Parameter에 있는 url은 Server의 url입니다. 즉 ngrok으로 포트 포워딩 시킨 서버의 주소를 말합니다. 
  ex ) https://abb3c3a5d2ea.ap.ngrok.io [https 주소로 요청을 받기 때문에 https 주소를 사용해야 합니다.]



* [config.py]

  ```python
  class config():
  	def __init__(self):
      ## api_key : 1330902587:AAF0iLjeXQFIRfCCAldcuTJvEQ6boIAH0jo
      
  		self.API_KEY = '1330902587:AAF0iLjeXQFIRfCCAldcuTJvEQ6boIAH0jo' ## Telegram API 키
      self.WEBHOOK_URL = 'https://bfe624810126.ap.ngrok.io' ## URL 주소
      self.SET_WEBHOOK = "https://api.telegram.org/bot{API_KEY}/setWebhook?url={WEBHOOK_URL}".format(API_KEY = self.API_KEY, WEBHOOK_URL = self.WEBHOOK_URL)
      ## Webhook을 세팅하기 위해서 Parameter은 URL query string으로 사용합니다.
      ## 파라미터 이름은 문서에 있는 그대로 보내야합니다. 다르게 보낼 경우 오류가 발생합니다.
  	
  ```

  

----------------

### Webhook을 걸기 위해 Webhook.py 파일을 생성

* 요청을 보내기 위해서 request를 사용합니다.
* 커맨드창(cmd)에서 pip3 install requests



* [Webhook.py] 

```python
import requests
from config import config
 
config = config()

response = requests.get(config.SET_WEBHOOK)
print(response)
```

* 응답이 200이 아닌 다른 코드로 왔다면 아래 테이블 표를 참고해주세요!

![스크린샷 2020-10-28 오후 1.17.38](/Users/JMacPro/Library/Application Support/typora-user-images/스크린샷 2020-10-28 오후 1.17.38.png)



---------------------

###  POST로 요청을 받기 위해서 Server.py를 수정

* [ 메시지를 자신의 서버에서 받기 ]

```python
from flask import Flask, request
app = Flask(__name__)
@app.route('/', methods = ['POST'])

def server():
  data = request.get_json()
	print(data)
  
  return "Hello World!"

if __name__ == '__main__': 
  app.run(debug = True, host = 'localhost', port = 5000)
```

* Server.py를 구동

* telegram Bot에서 메시지를 보내서 로그가 찍히는지 확인

---------

* 정상적으로 구동 되었다면 메시지 다음처럼 출력되게 됩니다.
* ![스크린샷 2020-10-28 오후 8.15.51](/Users/JMacPro/Desktop/스크린샷 2020-10-28 오후 8.15.51.png)



* 하지만 여기서 불편한 점이 있습니다!.. 어떤 점이 가장 불편할까요??







*  https://jsonformatter.curiousconcept.com/  --> JSON 파일 방식의 정리해주는 사이트입니다. 자주 애용 합시다. :)

  ![스크린샷 2020-10-28 오후 8.12.59](/Users/JMacPro/Desktop/스크린샷 2020-10-28 오후 8.12.59.png)

* 해당 사이트에서 정리를 하게 되면 다음과 같이 보기 편하게 출력됩니다.

  

------------------------

### 메시지 보내기 

- 메시지를 보내기 위한 메소드를 API 공식문서에서 확인해봅시다.

![스크린샷 2020-10-28 오후 1.32.40](/Users/JMacPro/Desktop/스크린샷 2020-10-28 오후 1.32.40.png)

* Required = [chat_id, text] 즉 해당하는 2개는 무조건 parameter로 넣어줘야 합니다.
* [config.py]를 다음과 같이 수정합니다.

```python
class config():
	def __init__(self):
		self.API_KEY = 'Telegram Bot API Key' ## Telegram API 키
    self.WEBHOOK_URL = 'https://16d7a6fed154.ap.ngrok.io' ## URL 주소
    self.SET_WEBHOOK = "https://api.telegram.org/bot{API_KEY}/setWebhook?url={WEBHOOK_URL}".format(API_KEY = self.API_KEY, WEBHOOK_URL = self.WEBHOOK_URL)
    ## Webhook을 세팅하기 위해서 Parameter은 URL query string으로 사용합니다.
    ## 파라미터 이름은 문서에 있는 그대로 보내야합니다. 다르게 보낼 경우 오류가 발생합니다.
    
    
    ### 메소드 기본 사용 방법 == https://api.telegram.org/bot<token>/METHOD_NAME
    ### 추가되는 내용
   
    self.SEND_MESSAGE = "https://api.telegram.org/bot{API_KEY}/sendMessage".format(API_KEY = self.API_KEY)
```



1. JSON DATA에게서 사용자 정보를 파싱 -> chat_id , (text는 다른 메시지를 보낼 수 있으므로 파싱하지 않습니다.)

   [bot.py]를 생성하여 코드를 수정 # bot.py 파일 하나 생성하시고

```python
class telegrambot:
    
    def __init__(self):
        
        self.chat_id = None
        self.name = None
        
    def __call__(self, data): ## 파싱 함수로 사용
        
        chat_id = data['message']['chat']['id']
        name = data['message']['chat']['last_name'] + data['message']['chat']['first_name']

        self.chat_id = chat_id
        self.name = name 
    
    def sendMessage(self, text):
        
        params = {'chat_id' : self.chat_id, 'text' : text}
        
```



* json을 request에 실어서 보낼때 어떻게 보낼 것인가?? 사용법을 우리는 모르고 있습니다.

* 구글링을 통해 StackOverFlow에서찾은 예시입니다.

* ![스크린샷 2020-10-28 오후 2.13.22](/Users/JMacPro/Library/Application Support/typora-user-images/스크린샷 2020-10-28 오후 2.13.22.png)

  

* [bot.py]를 수정합니다.

```python
import requests ## 추가
from config import config ## 추가
config = config() ## 추가

class telegrambot: 
  '''
  '''
  def sendMessage(self, text):
        
        params = {'chat_id' : self.chat_id, 'text' : text}
        
        
        requests.post(config.SEND_MESSAGE, json = params) ## 추가


```

### 마지막으로 자신의 서버에서 받은 데이터들을 파싱하여 다시 사용자에게 데이터를 출력하는 부분을 수행하겠습니다.

* [Server.py]

```python
from flask import Flask, request
from bot import telegrambot ## 추가

app = Flask(__name__)

@app.route('/', methods = ['POST'])

def server():
  
  data = request.get_json()
  print(data)
  
  bot = telegrambot() ##인스턴스화 ##추가
  bot(data) ## 추가
  
  print(bot.chat_id) ## 추가
  print(bot.name) ## 추가
  
  bot.sendMessage('안녕 난 MIMA 챗봇이야') ## 추가
  
  return "Hello World!"
  
 if __name__ == "__main__":
    app.run(debug = True, host = 'localhost', port = 5000)
```

-------------

### 최종적으로 Telegram 챗봇에 감성 분석 모델을 실어 올린 모델을 실어내는 과정을 소개하겠습니다.



* Server.py 와 bot.py 두가지 파일을 수정합니다.



- bot.py

```python
import requests
from config import config
config = config()

import pandas as pd
import csv
from konlpy.tag import Okt
from tensorflow.keras.preprocessing.text import Tokenizer
from tensorflow.keras.preprocessing.sequence import pad_sequences
from tensorflow.keras.models import load_model ## import 선언 

class telegrambot:

    def __init__(self): ## 클래스의 인스턴스를 만들고 인스턴스 변수를 초기화 할 때 호출
        
        self.chat_id = None
        self.chat_text = None
        self.predict_text = None ## 나중에 감성 분석이 완료된 텍스트를 저장하기 위해 미리 선언

    def __call__(self, data): ## 파싱 함수로 사용
         ## 다른 함수처럼 객체를 호출 할 때 호출
        
        if "edited_message" in data : ## JSON 데이터 타입으로 메시지가 오는 방식이 2가지이므로 if else문으로 처리
            chat_id = data['edited_message']['chat']['id']
            text_data = data['edited_message']['text'] 
        else :
            chat_id = data['message']['chat']['id']
            text_data = data['message']['text']
        
        self.chat_id = chat_id
        self.chat_text = text_data
        

    def sentiment_predict(self, text) : ## 소형화된 감성 분석 모델을 탑재하기 위한 함수

        tokenizer = Tokenizer(19417, oov_token = 'OOV') # vocab size = 19417
        file = open(r'./xtrain.csv', 'r' , encoding = 'ms949') # 경로 지정
        data = csv.reader(file) 
        data = list(data) ## 정수 인코딩 전 데이터가 필요하기에 현정님이 csv로 뽑아내신 데이터파일을 로드 
        tokenizer.fit_on_texts(data) ## 정수 인코딩을 위한 단어 사전을 만드는 과정
        stopwords = ['의','가','이','은','들','는','좀','잘','걍','과','도','를','으로','자','에','와','한','하다']
        okt = Okt() ## 형태소 분석기를 사용하기 위해서 선언
        loaded_model = load_model('best_model.h5') ## 가중치 모델인 best_model.h5 로드
        max_len = 35 ## 패딩(길이를 똑같이 맞춰주는 작업)을 위해서 선언

## --------------------------- 예측 모델 구동 --------------------------- 구분을 위해서 주석 표시----------------
        pre_sentence = None ## 어떤 값이 들어갈지 모르므로 None으로 처리
        new_sentence = okt.morphs(self.chat_text, stem = True) ## 문장 토큰화
        ## self.chat_text의 값 즉 위의 JSON 데이터에서 파싱해서 받은 text 값을 입력함
    
        new_sentence = [word for word in new_sentence if not word in stopwords] ## 불용어 제거
        # tokenizer.fit_on_texts(new_sentence) # 단어 사전을 43번째 줄에서 만들었으므로 해당 소스를 제외합니다.
        encoded = tokenizer.texts_to_sequences([new_sentence]) ## 정수 인코딩 수행
        pad_new = pad_sequences(encoded, maxlen = max_len) ## 패딩 수행
        score = float(loaded_model.predict(pad_new)) ## best_model.h5 파일을 이용해서 예측 수행

        if(score > 0.5):
            pre_sentence = "{:.2f}% 확률로 긍정 리뷰에 해당합니다. ".format(score * 100)
        else:
            pre_sentence = "{:.2f}% 확률로 부정 리뷰에 해당합니다. ".format((1-score) * 100)

        self.predict_text = pre_sentence

        # self.predict_text = pre_sentence

    def sendMessage(self, text):
        # text_params = {'chat_id' : self.chat_id, 'text' : self.chat_text}
        params = {'chat_id' : self.chat_id, 'text' : self.predict_text}
        # requests.post(config.SEND_MESSAGE, json = text_params)
        requests.post(config.SEND_MESSAGE, json = params)
        
```



- Server.py

```python
from flask import Flask,request
from bot import telegrambot

app = Flask(__name__)

@app.route('/', methods = ['POST'])

def server():
    
    data = request.get_json() ## 기본적으로 지원되는 방식
    print(data)
    bot = telegrambot()
    bot(data)
    # print(bot.chat_id)
    # print(bot.chat_text)
    # print(bot.sentiment_predict('text'))
    bot.sentiment_predict('text')  ## 감성 분석 모델에서 결과를 받기 위해서 추가한 부분입니다.
    bot.sendMessage('text')

    return "Hello World!"

if __name__ == "__main__":
    app.run(debug = True, host = 'localhost', port = 5000)
```

