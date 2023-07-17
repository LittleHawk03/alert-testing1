## survey gcp notifacation channel

### 1. preview

- google cloud cho phép tạo các "notificatin channel" cho phép sử dụng các kênh đó để thông báo cho người dùng hoặc một nhóm người dùng (team làm việc) khi các chính sách được áp dụng. Khi mà thiết lập các chính sách cảnh bảo, ta có thể chọn ai được phép nhận thông báo từ gcp, ví dụ, ta có thể cài đặt monitoring 1 instances để đẩy alert đến với người dùng thông quá slack channel hoặc là on-call team

#### Notification channel

tạo Notification Channel

<div align="center">
  <img src="assets/pic_3.png">
</div>


GCP hỗ trợ tính năng thông báo/gửi cảnh báo đên đa dạng nền tảng như qua PagerDuty, Slack, Webhook, Email, SMS và tính năng Pub/Sub 

### 1.2 . Thông báo qua tính năng Pub/Sub (publisher / subscriber)
- Pub/Sub (viết tắt của Publisher/Subscriber) là một mô hình truyền thông trong các hệ thống phân tán, nơi người gửi (publisher) gửi các thông điệp không đồng bộ đến một hoặc nhiều người nhận (subscriber). Trong mô hình này, người gửi và người nhận không trực tiếp liên kết với nhau. Thay vào đó, thông điệp được gửi thông qua một kênh trung gian gọi là "topic" mà người nhận quan tâm và đã đăng ký.

- Một số khái niệm cơ bản :
    1. Pub - Publish: producer sẽ đẩy data vào một chanel.
    2. Sub - Subscribe: consumer đăng ký nhận dữ liệu từ một chanel.
    3. Topic : là một tài nguyên mà các publicsẻr gửi thư tới 
    4. subcription : Là một luông message được gửi đi bởi các topic và các message được gửi đến luông sẽ được gửi đến các subcriber (1 hoặc nhiều subcriber ) đã đằn ký luồng đó

<!-- ![](assets/pic_1.png) -->

<div align="center">
  <img src="assets/pic_1.png">
</div>


#### usecase với chức năng gửi alert tới ứng dụng google chat bằng tính năng Pub/Sub

Hệ thống Pub/Sub của google khá hữu dụng khí mà chức năng "Notification Channel" không hỗ trợ một số nên tảng, thì ta cũng có thể tự tạo một pipeline riêng để gửi cảnh báo q


<div align="center">
  <img src="assets/pic_2.png">
</div>

1. thu thập metrics, logs của hệ thổng đang được giám sát
2. bằng các metric và logs đó chuyển đến hệ thống chính sách cảnh báo giám sát 
3. các cảnh báo đó được chuyển đến hệ thống notification channel nhằm tạo các message gửi các thông báo đến hệ thống Pub/Sub
4. Bằng hệ thống Pub/Sub để thực hiện push subcription nhằm gửi subcription đến Cloud Run service 
5. Cloud Run service có nhiệm vụ đơn giản nhận các các message từ pub/sub và chuyển đổi chúng ra các tin nhắn của google chat

### 2. Thông báo qua tính năng slack

GCP cung cấp khả năng gửi alert đến Slack thông qua các công cụ như Google Cloud Monitoring và Google Cloud Functions. Bằng cách kết hợp các dịch vụ này, người dùng có thể nhận được thông báo trực tiếp trong Slack khi xảy ra các sự kiện quan trọng trong hệ thống của GCP.

- phân tích tính năng thông báo qua slack, mỗi khi chọn chức năng gửi thông báo Slack trang sẽ được truyển hướng đến trang web uy quyền để cho phép GCP có thể gửi được tin nhắn :

<div align="center">
  <img src="assets/pic_5.png">
</div>


- Slack Channel Name: tên cuả channel mà mình mong muốn có thể nhận tin nhắn alert 

- Cloud Alerting Display name: tên của đối tượng gửi thông báo, Khi áp dụng tính năng gửi thông báo cho một chính sách bất kỳ người dùng sẽ dựa vào tên này để có thể thêm vào chức năng thông báo/ cảnh báo của chính sách đó nhằm gửi đến nền tảng mà người dùng mong muốn nhận thông báo đó. 

- để kiểm tra kết nối đến với slack chọn "SEND TEST NOTIFICATION" khi đó đối tượng mà ta đã tạo sẽ gửi đến kênh đã chọn một thông bảo để kiểm tra kết nối.


<div align="center">
  <img src="assets/pic_6.png">
</div>

Khi thiết lập chính sách cảnh báo người dùng (nếu người dùng đó muốn sử dụng dịch vụ Notification Channel) sẽ chọn kênh thông báo mà người dùng mong muốn nhận tín nhắn cảnh báo


<div align="center">
  <img src="assets/pic_7.png">
</div>

<div align="center">
  <i>slack-alert: là tên của Notification channel mà người dùng đã cài đặt tại phần Cloud Alerting Display name</i>
</div>


Kết quả:

<div align="center">
  <img src="assets/pic_8.png">
</div>

<div align="center">
  <i>cảnh báo được gửi về cho người dùng khi mà mức ram vượt qua ngương cảnh báo là 30% (với mục đích test cảnh báo)</i>
</div>

### 3. Thông báo qua tính năng Webhooks (ở đây em sử dụng webhook của một chat-bot trên telegram)

Webhook (cũng có thể gọi là web callback hay HTTP push API) cho phép ứng dụng cung cấp data cho một ứng dụng khác trong thời gian thực. Không như các API điển hình khác ta cần phải thăm dò server thường xuyên để biết xem có events mới hay không, với webhook bất cứ khi nào có event mới server-side sẽ tự động thông báo cho client-side được biết.


#### Gửi thông báo thông báo đến telegram thông qua chat bot ```Telepush```

Telepush là một chat bot đơn giản nhằm dịch ```POST``` requests dưới dạng JSON thành các tin nhắn trên telegram, ứng dụng này tương tự như là Gotify hay là ntfy.sh. CHat bot này hữu ích cho việc monitoring, cảnh báo (alert) và nhiều ứng dụng khác.

<div align="center">
  <img width="500" src="assets/pic_9.png">
</div>

Đây là một dự án mã nguồn được viết bởi ngôn ngữ GO và mã nguồn mở tại link github : https://github.com/muety/telepush ngoài việc có thể  gửi thông báo qua method ```POST``` qua URL ```https://telepush.dev/api/[[inlets/<inlet_name>]|messages]/<recipient>``` thì ta cũng có thể host một chatbot của riêng mình.

#### Thử nghiệm chức năng thông báo cảnh báo qua chatbot telegram telepush sử dụng webhooks

Cấu trúc quá trình cảnh báo người dùng qua telegram chức năng đẩy thông báo qua webhooks bằng chat bot Telepush


<div align="center">
  <img width="1500" height="400" src="assets/pic_10.png">
</div>


<div align="center">
  <i> mô hình đơn giản quá trình gửi thông quá qua webhook - Telepush API sẽ nhận các dữ liệu, dich và chuyển hóa thành tin nhắn</i>
</div>

**Bước 1**: Thêm chat bot vào chat list hoặc thêm vào một group chat


<div align="center">
  <img width="300" src="assets/pic_11.jpg">
</div>


**Bước 2**: Để có thể thông báo được qua chat Bot ta cần phải có được token mới của chat bot này gõ lệnh ```/start``` 

<div align="center">
  <img width="300" src="assets/pic_12.jpg">
</div>

sau khi thành công tạo mới một token cho chatBot hiện tại ta thêm giá trị của token đó vào một URL có cấu trúc như sau ```https://telepush.dev/api/inlets/alertmanager/<YOUR_TOKEN>``` trong ví dụ này YOUR_TOKEN sẽ bằng ```8e6657``` thì URL webhook của chúng ta sẽ là ```https://telepush.dev/api/inlets/alertmanager/8e6657```

**Bước 3:** tạo một Notification channel mới với lựa chọn gửi thông báo cảnh báo đi bằng webhooks.

<div align="center">
  <img width="400" src="assets/pic_13.png">
</div>

Chọn ```TEST CONNECTION``` để kiểm tra kết nối:

<div align="center">
  <img width="500" src="assets/pic_14.png">
</div>


**Bước 4:** Tạo chính sách cảnh báo trên Google Cloud.

<div align="center">
  <img width="500" src="assets/pic_15.png">
</div>

Tại phần Notification Channel chọn channel mà bạn vừa thêm.

**Bước 5:** Kết quả thông báo cảnh báo qua telegram

<div align="center">
  <img width="500" src="assets/pic_16.jpg">
</div>


Theo như trong hình gcp có thể gửi được cảnh báo về cho người dùng tuy nhiên chat bot này chỉ hỗ trợ để đọc và dịch những cảnh báo của ```Alertmanager``` nên như trong hình rất nhiều cảnh báo đã được trả về tuy nhiên lại không có nội dung.


```POST REQUEST``` dạng ```JSON``` của **alertmanager**.

```yaml
  "version": "4",
  "groupKey": <string>,              
  "truncatedAlerts": <int>,          
  "status": "<resolved|firing>",
  "receiver": <string>,
  "groupLabels": <object>,
  "commonLabels": <object>,
  "commonAnnotations": <object>,
  "externalURL": <string>,          
  "alerts": [
    {
      "status": "<resolved|firing>",
      "labels": <object>,
      "annotations": <object>,
      "startsAt": "<rfc3339>",
      "endsAt": "<rfc3339>",
      "generatorURL": <string>,      
      "fingerprint": <string>
    },
    ...
  ]
```

```POST REQUEST``` dạng ```JSON``` của **google cloud platform**

```yaml
  {
  "responseId": "response-id",
  "session": "projects/project-id/agent/sessions/session-id",
  "queryResult": {
    "queryText": "End-user expression",
    "parameters": {
      "param-name": "param-value"
    },
    "allRequiredParamsPresent": true,
    "fulfillmentText": "Response configured for matched intent",
    "fulfillmentMessages": [
      {
        "text": {
          "text": [
            "Response configured for matched intent"
          ]
        }
      }
    ],
    "outputContexts": [
      {
        "name": "projects/project-id/agent/sessions/session-id/contexts/context-name",
        "lifespanCount": 5,
        "parameters": {
          "param-name": "param-value"
        }
      }
    ],
    "intent": {
      "name": "projects/project-id/agent/intents/intent-id",
      "displayName": "matched-intent-name"
    },
    "intentDetectionConfidence": 1,
    "diagnosticInfo": {},
    "languageCode": "en"
  },
  "originalDetectIntentRequest": {}
}


```



### 4. Thông báo cảnh báo qua email người dùng

Google cloud hỗ trợ gửi thông báo cảnh báo cho người dùng thông qua email Khi chính sách cảnh báo được kích hoạt, GCP sẽ gửi các thông báo tương ứng đến địa chỉ email đã chỉ định. Đây là tính năng gửi cảnh báo cơ bản nhất mà google cloud cũng cấp, việc gửi thông báo quá email là tốt tuy nhiên cách này khá nặng nề.

#### Thử nghiệm chứ năng gửi cảnh báo qua email 

**Bước 1:** Tại Notification Channel tạo mới một email Notification Channel.

<div align="center">
  <img src="assets/pic_17.png">
</div>


<div align="center">
  <i>Nhập email muốn nhận cảnh báo</i>
</div>


**Bước 2:** Tạo chính sách cho có chức năng cảnh báo qua email

<div align="center">
  <img src="assets/pic_18.png">
</div>

<div align="center">
  <i>thêm Notification Channel vào chính sách</i>
</div>

**Bước 3:** Kiểm tra kết quả

<div align="center">
  <img  src="assets/pic_19.png">
</div>

<div align="center">
  <i>thêm Notification Channel vào chính sách</i>
</div>


Việc thông báo qua email là một phương pháp gửi cánh báo tốt và cơ bản tuy nhiên nó lại khá cồng kềnh, nếu có nhiều cảnh báo hoặc gửi liên tục các email cảnh bảo đến người dùng gây bất tiện.


### 5. Thông báo qua SMS (short message service) qua số điện thoại của người dùng.

SMS Channel trong Google Cloud Monitoring là một tính năng cho phép gửi cảnh báo thông qua tin nhắn SMS đến người dùng khi xảy ra các sự cố hoặc sự thay đổi quan trọng trong hệ thống. Điều này giúp người dùng nhận được thông báo ngay lập tức ngay cả khi không có kết nối wifi hoặc google, tuy nhiên cần đảm bảo trong vùng phủ sóng.

#### Các sản phẩm SMS của google

- **Blended SMS**: Yêu cầu một cuộc gọi IVR xuyên suốt trong quá trình chat SMS, trong blebder SMS lại có 2 loại:
  
    - **In-call SMS**

    - **Wait time SMS**

- **SMS Channel**: chat SMS có thể được gửi đến và gửi đi độc lập với cuộc gọi IVR

  - **SMS inbound/outbound**: 

  - **Pre-session SMS Deflection**: 

- **SMS to launch App**: 

#### Thử nghiệm chức năng gửi cảnh báo cho người dùng

SMS channel mang lại lới ích cao tuy nhiên có vẻ như google cloud platform chưa hỗ trợ tính năng gửi thông báo qua SMS tại Việt Nam.

<div align="center">
  <img  src="assets/pic_20.png">
</div>

<div align="center">
  <i>Tính năng thông báo chưa được hỗ trợ tại quốc gia sở tại</i>
</div>






