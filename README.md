# Survey the alert function of google cloud platform and Configuration of AlertManager

## Table of content

### [I. Khảo sát tính năng cảnh báo của Google Cloud Platform](#i-khảo-sát-tính-năng-cảnh-báo-của-google-cloud-platform)

1. [Cảnh báo qua tính năng pub/sub](#1-thông-báo-qua-tính-năng-pubsub-publisher--subscriber)
2. [Cảnh báo qua ứng dụng tin nhắn slack](#2-thông-báo-qua-tính-năng-slack)
3. [Cảnh báo qua tính năng Webhooks](#3-thông-báo-qua-tính-năng-webhooks-ở-đây-em-sử-dụng-webhook-của-một-chat-bot-trên-telegram)
4. [Cảnh báo qua thư email](#4-thông-báo-cảnh-báo-qua-email-người-dùng)
5. [Cảnh báo qua tin nhắn SMS](#5-thông-báo-qua-sms-short-message-service-qua-số-điện-thoại-của-người-dùng)

### [II. Cấu hình Notification của AlertManager](#ii-cấu-hình-notification-của-alertmanager)

1. [Configuration File](#1-configuration-file)
2. [Route](#2-route)
3. [Receiver](#3-receiver)
4. [Inhibit Rule](#4-inhibit-rule)
5. [Thử nghiệm thông báo cảnh báo đến một số nền tẳng tin nhắn như email, slack, telegram](#5-thử-nghiệm-thông-báo-cảnh-báo-đến-một-số-nền-tẳng-tin-nhắn-như-email-slack-telegram-bằng-các-công-cụ-giám-sát-như-alertmanager-prometheus-node-exporter-cadvisor)
  - 5.1 [Mô hình giám sát trên máy tính cá nhân](#51-mô-hình-giám-sát-trên-máy-tính-cá-nhân)
  - 5.2 [Cấu hình gửi cảnh báo thông qua ứng dụng slack](#52-cấu-hình-gửi-cảnh-báo-đến-slack)
  - 5.3 [Cấu hình gửi cảnh báo qua ứng dụng telegram](#53-cấu-hình-cảnh-báo-qua-telegram)
  - 5.4 [Cấu hình gửi cảnh báo qua email](#54-cấu-hình-cảnh-báo-qua-email)

### [III. References](#iii-references)

## I. Khảo sát tính năng cảnh báo của Google cloud platform


### Preview

- google cloud cho phép tạo các "notificatin channel" cho phép sử dụng các kênh đó để thông báo cho người dùng hoặc một nhóm người dùng (team làm việc) khi các chính sách được áp dụng. Khi mà thiết lập các chính sách cảnh bảo, ta có thể chọn ai được phép nhận thông báo từ gcp, ví dụ, ta có thể cài đặt monitoring 1 instances để đẩy alert đến với người dùng thông quá slack channel hoặc là on-call team

#### Notification channel

tạo Notification Channel

<div align="center">
  <img src="assets/pic_3.png">
</div>


GCP hỗ trợ tính năng thông báo/gửi cảnh báo đên đa dạng nền tảng như qua PagerDuty, Slack, Webhook, Email, SMS và tính năng Pub/Sub 

### 1. Thông báo qua tính năng Pub/Sub (publisher / subscriber)
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

Hệ thống Pub/Sub của google khá hữu dụng khí mà chức năng "Notification Channel" không hỗ trợ một số nên tảng, thì ta cũng có thể tự tạo một pipeline riêng để gửi cảnh báo.


<div align="center">
  <img src="assets/pic_2.png">
</div>

1. Thu thập metrics, logs của hệ thổng đang được giám sát
2. Bằng các metric và logs đó chuyển đến hệ thống chính sách cảnh báo giám sát 
3. Các cảnh báo đó được chuyển đến hệ thống notification channel nhằm tạo các message gửi các thông báo đến hệ thống Pub/Sub
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
  <img src="assets/pic_10.png">
</div>


<div align="center">
  <i> Mô hình đơn giản quá trình gửi thông quá qua webhook - Telepush API sẽ nhận các dữ liệu, dich và chuyển hóa thành tin nhắn</i>
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

Dưới đây là nội dung của 2  ```POST REQUEST``` webhooks của alertmanager và gcp


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
  <i>Thêm Notification Channel vào chính sách</i>
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





## II. Cấu hình Notification của AlertManager 

#### 1. Configuration file

Để có thể cài đặt các cài đặt được viết trong config file của alertmaneger ta dùng lệnh :

```sh
  ./alertmanager --config.file=config.yml

```

Trong ```dockerfile``` hoặc ```docker-compose``` 

```yaml
  command:
    - "--config.file=/etc/alertmanager/config.yml"
```

Dockerfile

```docker
   CMD [ "--config.file=/etc/alertmanager/alertmanager.yml"
```

Configuration file được viết dưới dịnh dạng YAML

#### 2. Route

Route là một khối cấu hình dùng để định nghĩa một quy trình định tuyến cảnh báo. Nó sẽ xác định cách cảnh báo sẽ được gom nhóm, xử lý và gửi thông tin cảnh báo đến người dùng 

**matcher :** hỗ trợ cú pháp so sánh với các toán tử  =, !=, =~, !~.
vd  ```service =~ "name_A,name_b"```

Router trong alert manager hoạt động như một cây định tuyến và các ảnh báo sẽ di chuyển giữa các nút. Cơ chế hoạt động sẽ như sau:

- Các cảnh báo khi tới cây route đều đi qua nút cao nhất của cấu hình là ```route```, và nó khớp với tất cả các cảnh báo (khi không có matcher nào được cấu hình)

- các cảnh báo sau đó đi qua các nút con trong cây, Nếu ```continue``` được đặt là ```false``` thì quá trình sẽ dừng lại sau khi tìm thấy nút con đầu tiên khớp với cảnh báo đó (tức là quá trình so khớp sẽ dừng lại và gửi thông báo), Còn ngược lại nếu như ```continue``` bằng ```true``` nó sẽ tiếp tục so khớp với các nút cùng bậc để tìm nút khớp 

Ví dụ sau

```YAML
  route:
    group_by: ['alertname', 'cluster', 'service']
    group_wait: 30s
    group_interval: 5m
    repeat_interval: 3h
    receiver: team-X-mails
  
    routes:

        # route con 1
      - matchers:
          - service=~"foo1|foo2|baz"
        receiver: team-X-mails

        routes:
          - matchers:
              - severity="critical"
            receiver: team-X-pager

        # route con 2
      - matchers:
          - service="files"
        receiver: team-Y-mails

        routes:
          - matchers:
              - severity="critical"
            receiver: team-Y-pager

        # route con 3
      - matchers:
          - service="database"
        receiver: team-DB-pager
        group_by: [alertname, cluster, database]
        routes:
          - matchers:
              - owner="team-X"
            receiver: team-X-pager
            continue: true
          - matchers:
              - owner="team-Y"
            receiver: team-Y-pager
```

Mô hình cây quá trình hoạt động của ```route``` trên vẽ tại https://prometheus.io/webtools/alerting/routing-tree-editor/ 
 
<div align="center">
  <img src = "assets/pic_21.png">
</div>

Quy trình sẽ diễn ra như sau :

- Tại ``route con 1`` khi các cảnh báo có nhãn ```service``` khớp với các giá trị ``foo1``,``foo2`` hoặc ```baz``` thì xẽ được xử lý như sau đối với các cảnh báo có mức độ nghiêm trọng là ```critical``` thì sẽ được gửi đến người/nhóm nhận là ```team-X-pagers```, còn nếu không có mức độ nghiêm trọng như vậy thì cảnh báo sẽ được gửi qua cho ```team-X-mails``` 

- Tại ```route con 2 ``` tương tự như ``route con 1`` nhưng đối với nhẫn ```service``` là ```file``` nếu mức độ nghiêm trong là ```critical``` thì gửi cho ```team-Y-mails``` còn nếu không thì gửi cho ```team-Y-pager```

- Tại ```route con 3``` Xử lý các nhãn ```service``` khớp với giá trị là ```database``` . Nếu như cảnh báo đó có nhãn ```owner``` là **'team-X'** thì gửi cảnh báo đến với ``team-X-pager`` và vì có ```continue: true``` (là sẽ tiếp tục so khớp với nút cũng bậc) nếu sẽ tiếp tục so khớp nếu cảnh báo đó có nhãn ``owner`` là **"team-Y"** thì gửi cho ```team-Y-pager```


### 3. Receiver 

Receriver là tên một hay nhiều thiết lập thông báo trên các nền tảng khác nhau.

AlertManager hỗ trợ thông báo cảnh báo đến đa dạng nền kể đến như là email, pageduty, slack, telegram, wechat v.v ...

```yaml
    # The unique name of the receiver.
    name: <string>
    # Configurations for several notification integrations.
    email_configs:
      [ - <email_config>, ... ]
    opsgenie_configs:
      [ - <opsgenie_config>, ... ]
    pagerduty_configs:
      [ - <pagerduty_config>, ... ]
    pushover_configs:
      [ - <pushover_config>, ... ]
    slack_configs:
      [ - <slack_config>, ... ]
    sns_configs:
      [ - <sns_config>, ... ]
    victorops_configs:
      [ - <victorops_config>, ... ]
    webhook_configs:
      [ - <webhook_config>, ... ]
    wechat_configs:
      [ - <wechat_config>, ... ]
    telegram_configs:
      [ - <telegram_config>, ... ]
    webex_configs:
      [ - <webex_config>, ... ]
```

### 4. Inhibit rule 

**inhibit_rule:** là một thuộc tính trong Prometheus được sử dụng để kiểm soát việc áp dụng các quy tắc cảnh báo. Nó được dùng để tạm ngừng hoặc loại bỏ các cảnh báo dựa trên các điều kiện.

- **target_match:**  Đây là matcher được sử dụng để xác định các cảnh báo mục tiêu mà bạn muốn tạm ngừng hoặc loại bỏ khi một điều kiện xác định xảy ra

- **source_matcher:**  Đây là matcher được sử dụng để xác định các cảnh báo nguồn, tức là các cảnh báo mà khi trùng khớp với các cảnh báo từ target_match, sẽ gây ra tác động đến các cảnh báo từ target_match. 

Khi ```source_matcher``` trùng khớp với ``target_match``, một quy tắc ```inhibit_rule```được áp dụng và cảnh báo từ ```source_matcher``` sẽ không được gửi đi hoặc sẽ bị loại bỏ tạm thời. Điều này giúp tránh các cảnh báo trùng lặp hoặc không cần thiết khi một điều kiện cụ thể xảy ra.

vd:

```YAML
  inhibit_rules:
    - target_match:
        severity: "warning"
      source_match:
        severity: "critical"
      # Apply inhibition if the alertname and instance are the same.
      equal: ["alertname", "instance"]
```

### 5. Thử nghiệm thông báo cảnh báo đến một số nền tẳng tin nhắn như email, slack, telegram bằng các công cụ giám sát như alertmanager, prometheus, node-exporter, cadvisor

##### 5.1 Mô hình giám sát trên máy tính cá nhân


<div align="center">
  <img src="assets/pic_28.png">
</div>

cấu trúc của thư mục 


```sh
  .
  ├── alertmanager
  │   └── config.yml
  ├── docker-compose.yaml
  ├── prometheus1
  │   ├── alert.rules
  │   ├── filesd.json
  │   └── prometheus.yml
  ├── prometheus2
      ├── alert.rules
      ├── filesd.json
      └── prometheus.yml
```

##### 5.2 Cấu hình gửi cảnh báo đến slack

Alertmanager hỗ trợ gửi cảnh báo qua webhook của slack được gọi là Imcomming Webhook

Để tạo được Imcomming webhooks vào trang web https://api.slack.com/apps tạo một app mới 


<div align="center">
  <img src="assets/pic_22.png">
</div>

Sau khi tạo app thành công tại **Features** > **Incoming Webhooks** > tiến hành tạo URL webhooks trang chuyển hướng đến trang ủy quyền chọn channel phù hợp 

<div align="center">
  <img src="assets/pic_24.png">
</div>


Sau khi tạo ```Imcoming Webhook``` thành công sẽ có một url webhook được sinh ra, thêm url đó vào file config của alertmanager

<div align="center">
  <img src="assets/pic_23.png">
</div>


file cấu hình cho chức năng gửi thông báo qua slack:

```yaml
- name: "slack"
  slack_configs:
    - send_resolved: true
      text: "{{ .CommonAnnotations.description }}"
      username: "Prometheus"
      channel: "#test"
      api_url: "https://hooks.slack.com/services/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"
```

Sau khi cấu hình khi có cảnh báo alert manager sẽ gửi một ```POST REQUEST``` đến slack để gửi thông báo cho người dùng .

<div align="center">
  <img src="assets/pic_25.png">
</div>


##### 5.3 Cấu hình cảnh báo qua telegram

Alertmanager cũng hỗ trợ thông báo cảnh báo bằng telegram ```telegram_config```

```yaml
  # Whether to notify about resolved alerts.
  [ send_resolved: <boolean> | default = true ]
  # The Telegram API URL i.e. https://api.telegram.org.
  # If not specified, default API URL will be used.
  [ api_url: <string> | default = global.telegram_api_url ]
  # Telegram bot token
  [ bot_token: <string> ]
  # ID of the chat where to send the messages.
  [ chat_id: <int> ]
  # Message template
  [ message: <tmpl_string> default = '{{ template "telegram.default.message" .}}' ]
  # Disable telegram notifications
  [ disable_notifications: <boolean> | default = false ]
  # Parse mode for telegram message, supported values are MarkdownV2 Markdown, HTML and empty string for plain text.
  [ parse_mode: <string> | default = "MarkdownV2" ]
  # The HTTP client's configuration.
  [ http_config: <http_config> | default = global.http_config ]
```

****Gửi thông báo thông báo đến telegram thông qua chat bot ```Telepush```****

Telepush là một chat bot đơn giản nhằm dịch ```POST``` requests dưới dạng JSON thành các tin nhắn trên telegram, ứng dụng này tương tự như là Gotify hay là ntfy.sh. Chat bot này hô trợ dịch các ```POST REQUEST``` thành tin nhắn văn bản thân thiện với người dùng hơn và đơn giản thì chatbot này còn hỗ trợ webhooks cho phép việc cài đặt thông báo dễ dàng hơn.

config code sử dung webhook của chat bot telepush đẩy thông báo cảnh báo cho người dùng,

```yaml
  - name: "telegram"
    webhook_configs:
      - url: "https://telepush.dev/api/inlets/alertmanager/<chatbot_token>"
        http_config:
```

với ```chatbot_token``` là token của chatbot để có thể tạo token mới cho chatbot telepush gửi lệnh sau ```/start```

<div align="center">
  <img width="300" src="assets/pic_12.jpg">
</div>

Bằng cách này alertmanager có thể gửi được cảnh báo cho người dùng 1 cách nhanh chóng qua telegram với thông điệp thân thiện hơn:

<div align="center">
  <img width="300" src="assets/pic_26.jpg">
</div>

##### 5.4 Cấu hình cảnh báo qua email

Để cấu hình Alertmanager gửi thông báo qua email, bạn cần chỉ định thông tin cần thiết như địa chỉ email người nhận, máy chủ SMTP và thông tin xác thực.

```yaml
  - to: <tmpl_string>
      from: <tmpl_string> | default = global.smtp_from 
      auth_username: <string> | default = global.smtp_auth_username
      smarthost: <string> | default = global.smtp_smarthost
      auth_password: <secret> | default = global.smtp_auth_password 
      auth_password_file: <string> | default = global.smtp_auth_password_file 
      auth_secret: <secret> | default = global.smtp_auth_secret 
      auth_identity: <string> | default = global.smtp_auth_identity 
```

các tham số trên cần các giá trị như sau:

  - ``to`` : là email của người nhận thông báo
  - ``from`` : là email của người gửi thông báo
  - ``smarthost`` : là địa chỉ máy chủ email (địa chỉ máy chủ google là smtp.gmail.com:587)
  - ```auth_username``` : là tên người dùng xác thực SMTP
  - ``auth_password`` : là mật khẩu của tài khoản người dùng xác thực

Trước khi có thể gửi được ```email``` cần phải tạo một ``app password`` để  thay thế cho mật khẩu chính của tài khoản Google. Điều này giúp cung cấp một lớp bảo mật bổ sung và hạn chế quyền truy cập cho các ứng dụng hoặc thiết bị không hỗ trợ xác thực hai yếu tố hoặc yêu cầu mật khẩu đặc biệt. Và giá trị auth_password sẽ nhận giá trị này

```yaml
  - name: "email"
    email_configs:
      - to: "manhduc030402@gmail.com"
        from: 'goldf55f@gmail.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'goldf55f@gmail.com'
        auth_password: 'sercet auth password'
```

sau khi cấu hình cảnh báo sẽ được gửi về email dưới dạng sau

<div align="center">
  <img src="assets/pic_27.png">
</div>

##### 5.5 Thông báo cảnh báo qua SMS

``SMS`` là một hình thức thông báo cực kỳ nhanh chóng. Tin nhắn SMS thường được gửi đi và nhận trọng thời gian ngắn, giúp đảm bảo cảnh báo sẽ được đưa ra một cách nhanh chóng. Điều này quan trong với các trường hợp khẩn cấp và cần phản hồi nhanh.

Tuy nhiên thì Alermanager cũng như Prometheus không hỗ trợ gửi trực tiếp tin nhắn SMS, do đó ta cần tự viết một ```webhook handler``` chịu trách nhiệm thu nhập thông tin và call api thông qua một bên thứ 3 để gửi tin nhắn.

**Dịch vụ gửi tin nhắn Twilio**

Twilio là dịch vụ cho phép bạn xây dựng ứng dụng liên quan đến việc gửi và nhận các tin nhắn và cuộc gọi trên điện thoại. Twilio thuộc top 10 ứng dụng đám mây đình đám của những năm qua. Và Twilio được tin dùng bởi một số tập đoàn lớn như là UBer

<div align="center">
  <img width="300" src="assets/pic_29.png">
</div>

Để thử nghiệm cho chức năng thông báo ta sử dụng tài khoản free trial của twillio, tài khoản trial này sẽ cho ta 15$ để có thể gửi tin nhắn, Sau khi đăng ký tài khoản **```twilio```** chúng sẽ sẽ được cung cấp 1 số điện thoại sẽ đóng vai trò như là người gửi và phải xác nhận số điện thoại đóng vai trò như người nhận thông điệp (ở đây em sử dụng điện thoại các nhân)

<div align="center">
  <img  src="assets/pic_30.png">
</div>

- **Account SID**: Đây là một khóa duy nhất được sử dụng để xác định một Tài khoản chính hoặc Tài khoản phụ Twilio cụ thể và là thông tin xác thực hoạt động như một tên người dùng.
- **Auth Token**: Được sử dụng để xác thực và ủy quyền các yêu cầu API và yêu cầu gửi tin nhắn qua Twilio
- **Twilio phone number** : là số điện thoại mà Twilio cung cấp được sử dụng cho mục đích nhận và gửi các cuộc gọi và tin nhắn SMS thông qua Twilio API.


**Ứng dụng nhận webhook từ alertmanager**

Trong phần này em sẽ tham khảo code của tác giả [Swatto](https://github.com/Swatto) với ứng dụng ```webhook handler``` nhầm thu nhận webhooks được gửi đi bởi alertmanager để gửi chúng như là một tin nhắn sms thông qua ```twilio```.

Ứng dụng webhook handler của tác giả ``Swatto`` được viết bằng ``Go-lang`` với các biến môi trường cụ thể  như là :

```go
  opts := options{
		AccountSid: os.Getenv("SID"), // gia tri SID ma twilio cung cap
		AuthToken:  os.Getenv("TOKEN"),
		Receiver:   os.Getenv("RECEIVER"), // so dien thoai cua nguoi nhan
		Sender:     os.Getenv("SENDER"), // so dien thoai cua nguoi gui, cai ma twilio cung cap
	}
```

funtion được sử dụng để gửi các tin nhắn 

```go
    func sendMessage(o *options, alert []byte) {
    c := twilio.NewClient(o.AccountSid, o.AuthToken) // khoi tao doi tuong khach hang mơi
    body, _ := jsonparser.GetString(alert, "annotations", "summary")

    if body != "" {
      body = findAndReplaceLables(body, alert)
      // cau hinh thong diep can gưi cho nguoi dung, thuong ơ dang rút gọn
      startsAt, _ := jsonparser.GetString(alert, "startsAt")
      parsedStartsAt, err := time.Parse(time.RFC3339, startsAt)
      if err == nil {
        body = "\"" + body + "\"" + " alert starts at " + parsedStartsAt.Format(time.RFC1123)
      }
      // gui tin nhan 
      message, err := twilio.NewMessage(c, o.Sender, o.Receiver, twilio.Body(body))
      if err != nil {
        log.Error(err)
      } else {
        log.Infof("Message %s", message.Status)
      }
    } else {
      log.Error("Bad format")
    }
}
```
Để đơn giản hóa cho tác giả cũng cung cấp ```Dockerfile``` để build image cho ```webhook handler``` này, em đã build lên và lưu trữ trên ```dockerhub``` với images tên ```littlehawk03/promtotwilio:v1.0.0``` với port để send webhook mặc định là 9089. 

Viết service ``sms`` trong [docker-compose.yaml](docker-compose.yaml):

```yaml
  sms:
    image: promtotwilio:latest
    environment:
      - SID=AC573b376f2fff71a0c0a45fb06c1c6f08
      - TOKEN=......
      - RECEIVER=+84349354228
      - SENDER=+12178058972
    restart: unless-stopped
    network_mode: host

```

Đây là cấu hình Alertmanger với service ``sms`` được cung cấp trong Docker compose

```yaml
  - name: "sms"
    webhook_configs:
      - url: 'http://sms:9089/send'
```

**Nhận thông báo cảnh báo qua tin nhắn SMS**

Sau khi khởi chạy docker-compose service ``sms`` sẽ nhận webhook từ url ``http://sms:9089/send`` hoặc ```http://0.0.0.0:9089/send``` bằng các api được cung cấp bởi twilio sẽ gửi tin nhắn cho người dùng 

kết quả : 

<div align="center">
  <img width="300" src="assets/pic_31.jpg">
</div>

 

### References

- https://prometheus.io/docs/alerting/latest/configuration/#string 
- https://github.com/vanduc95/dockprom
- https://github.com/muety/telepush
- https://muetsch.io/sending-prometheus-alerts-to-telegram-with-telepush.html
- https://cloud.google.com/monitoring/support/notification-options
- https://github.com/Swatto/promtotwilio
- https://www.twilio.com/docs/usage/webhooks/sms-webhooks