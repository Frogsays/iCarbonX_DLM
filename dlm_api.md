hatbot需求

## 1. 接受用户信息API

用途：接受用户信息，返回分析结果，可能会需要对应的widget展示或修改页面风格

调用例子：见具体example

endpoint: /nlp/dlm

method: POST

params: msg：string, id: integer，time_zone: string，cache:string

returns: 
```
{
intent: [enum]   // list of enum
data: {}     // 可能包括营养元素/运动消耗等信息，具体见example
options: [{text: string,
           value: string}] // 展示给用户的选项bubble内容
response: []    // 富文本，list中每个element都使用一个独立的对话框展示
element_style: []    // 对应展示response里每一个元素的style， 默认为现在的风格
ui_style: string    // 默认为default, i.e. 现在使用的默认风格 
cache: string      // cache保存上文信息，每次post时需要带上最新的cache
}
```
### 其中data的可能情况：
#### 这里只列出了data的内容，其余部分仍然参考上面例子

  1. 食物录入：

curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=苹果，香蕉&accountId=1&time_zone=utc+8&cache=some_internal_state'

```
 "food":  {
      "GI": 48.5,
      "GI_list": {
        "苹果": 36.0,
        "香蕉": 61.0
      },
      "GL": 9.14,
      "GL_list": {
        "苹果": 4.86,
        "香蕉": 13.42
      },
      "items": [
        {
          "date": "2018-04-02",
          "grams": 150.0,
          "high_gi_list": [],
          "hour": 18,
          "minutes": 6,
          "timestamp": 21400000000,
          "name": "苹果",
          "ori_name": "苹果",
          "quantity": null,
          "quantityUnit": null,
          "type": "FoodRecord"
        },
        {
          "date": "2018-04-02",
          "grams": 150.0,
          "high_gi_list": [],
          "hour": 18,
          "minutes": 6,
          "timestamp": 21400000000,
          "name": "香蕉",
          "ori_name": "香蕉",
          "quantity": null,
          "quantityUnit": null,
          "type": "FoodRecord"
        }
      ],
      "nutrition": {
        "碳水化合物": 53.25,
        "能量": 228.0,
        "脂肪": 0.6000000000000001,
        "膳食纤维": 3.5999999999999996,
        "蛋白质": 2.3999999999999995
      },
      "nutrition_ratio": {
        "碳水化合物": 0.9342105263157895,
        "脂肪": 0.02368421052631579,
        "膳食纤维": 1,
        "蛋白质": 0.04210526315789473
      },
    }
```
  2. 运动录入情况：

 curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=我跑了三小时，游了两小时泳&accountId=1&time_zone=utc+8&cache=some_internal_state'

```
    "sport": [
      {
        "duration": 180.0,
        "calorie": 500,
        "name": "跑步"
      },
      {
        "duration": 120.0,
        "calorie": 500,
        "name": "游泳"
      }
    ]
```
  3. 睡眠录入情况：

curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=我从昨天晚上10点睡到早上8点&accountId=1&time_zone=utc+8&cache=some_internal_state'

```
    "sleep": [
      {
        "date": "2018-04-01",
        "duration": 600,
        "end": "08:00:00",
        "start": "22:00:00"
      }
    ]
```
  4. 情绪录入情况：

curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=我今天心情很不好&accountId=1&time_zone=utc+8&cache=some_internal_state'

```
 "emotion": [
      {
        "category": "sadness",
        "mood": "不高兴"
      }
    ]
```
  5. 健康录入情况：

curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=我今天肚子疼&accountId=1&time_zone=utc+8&cache=some_internal_state'

```
    "health": [
      {
        "keyword": "肚子疼"
      }
    ]
```
  6. 药物录入情况：

curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=我三点吃了药&accountId=1&time_zone=utc+8&cache=some_internal_state'
```
  "medicine": [
    {
      "date": "2018-04-02",
      "grams": null,
      "hour": 3,
      "minutes": 0,
      "name": "药",
    }
  ],
```
可能会有多种场景混合的情况，或者还有其他场景的录入

7. 图片录入示例：
curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=查看图片&accountId=1&time_zone=utc+8&cache=some_internal_state'

返回结果： 
```json
{
    "intent": [],
    "data": [],
    "option": null,
    "response": [
        "这是你要的图片",
        "/9j/4AAQSkZJRgABAQAAAQABAAD...(后面省略了）"
    ],
    "element_style": [
        "text",
        "img_base64"
    ],
    "ui_style": null,
    "cache": {
        "status": "normal",
        "data": []
    }
}
```

查看url图片：curl -X POST http://211.159.138.208:50012/nlp/dlm -d 'msg=查看url图片&accountId=1&time_zone=utc+8&cache=some_internal_state'

```json
{
    "intent": [],
    "data": [],
    "option": null,
    "response": [
        "这是url图片",
        "https://www.icarbonx.com/Templates/tanyun/images/pro_03.png"
    ],
    "element_style": [
        "text",
        "img_url"
    ],
    "ui_style": null,
    "cache": {
        "status": "normal",
        "data": []
    }
}
```
说明：返回的图片为base64编码的图片，返回的response中每个元素对应的格式在element_style中有说明
文字是：text
base64图片是：img_base64
url图片是: img_url
其他的未来再扩展
base64图片解码的python解码过程如下：
```python
import requests
import base64
from io import BytesIO
from PIL import Image

url = 'http://211.159.138.208:50012/nlp/dlm'
data = {'msg': '查看图片', 'accountId': 1}
res = requests.post(url, data=data).json()
img = res.get('response')[1]

byte_data = img.encode('utf-8')
b64_str = base64.b64decode(byte_data)
buffer = BytesIO(b64_str)
im = Image.open(buffer)
im.show()
```

### response 的范例

[
 "2018-04-02 18:21:00 记录用餐一次，饭可能含有米饭，为了血糖平稳请尽量少吃。",
​     
 "2018-04-02 18:21:00 记录用餐一次，饭可能含有米饭，为了血糖平稳，请尽量少吃。识别到本餐总热量超标,美食面前不可贪嘴；并且膳食纤维摄入少,多吃点蔬菜调理肠胃。[:/happy]" // emoji
  ]

以上list中，每一个元素单独是一个对话泡泡，比如上面list有两个element，所以会有两个对话框



## 2. 用户端和专家端用到的 相关搜索API
用途：查找某个词在库中的模糊匹配结果（比如食物名，返回模糊匹配的食物），用type说明需要搜索哪个类型的库

调用例子：例如用户想修改食物名为“苹果派”，输入顺序为“苹果”+“派”，则将按顺序调用
```
/nlp/dlm/search?query=苹果 
/nlp/dlm/search?query=苹果派
```
返回分别为“苹果”及“苹果派”在食物库中的相关搜索结果

endpoint：/nlp/dlm/search

method: GET

params: query: string, type: string

returns:
[string]    // 根据分数排序

## 3. 选项bubble API

用途：用户点击选项bubble后调用的接口

调用例子：例如*接受用户信息*返回 options: 
[{text: '介绍', value: 'miwo-introduce'}, {text: '功能', value: 'miwo-function'}]
用户点击了“介绍”，则会post对应的'miwo-introduce'，并返回对应的内容

endpoint: /nlp/dlm/bubble

method: POST

params: bubble_value: string, id: integer, time_zone: string, cache: string

returns: 同*接受用户信息API*


## 4. 图片识别API

用途：用户上传图片后调用的接口

调用例子：用户拍照或上传食物图片，返回食物的解析结果

endpoint: /nlp/dlm/image

method: POST

params: files: the image file, id: integer,  time_zone: string, cache: string

returns: 同_接受用户信息API_，不过目前唯一可能识别出来的只有食物


## 5. 专家请求用户信息API

用途：专家端调用，用于获得一定时间内的对应用户信息

调用例子：专家1，配对用户10，请求2018年4月3日一天的数据进行血糖管理，POST参数为
```
{
	pid: 1,
	qid: 10,
	type: 'glycemia',
	start: 1522684800,
	end: 1522771199
}
```

endpoint: /nlp/dlm/profile

method: POST

params: pid: integer，专家id, qid: integer，专家请求的用户id，type: string，请求的数据类型,
start/end: 数据开始/结束的timestamp

returns: 专家端template所需参数


## 6. 专家端请求Chatbot回答的API

用途：专家端每次收到用户message时自动调用，并显示chatbot的回答供参考
调用例子：专家1，配对用户10，收到用户信息为'我头晕'
```
{
	pid: 1,
	qid: 10,
	type: ['glycemia'],
	text: '我头晕',
	cache: 'some_internal_state'
}
```

endpoint: /nlp/dlm/suggest

method: POST

params: pid: integer，专家id, qid: integer，配对的用户id，type: [string]，专家在当前对话中请求过的数据类型,
text: string，专家收到的用户信息

returns: {

    response: [string] // 为Chatbot自动生成的suggestion，list中每个element都使用一个独立的对话框展示

    cache: string // cache保存上文信息，每次post时需要带上最新的cache

}

## 7. 专家端与用户端沟通的API
此API应该无需经过chatbot，不过需要定好



## case业务流设计：

1. 用户发送消息(文本、语音、图片)
2. chatbot解析用户消息，返回意图判断，data，时间
  3.根据用户意图，对data以不同的前端插件进行展示
3. 前端插件支持用户修改
4. 将户状态解析数据（pid-时间-意图-data），存储在数据库中
5. 根据用户对表单的修改，更新用户状态数据库
6. 数据库更改，触发msg queue
7. msg queue触发AU
8. 基于用户数据更新，绘制、更新、存储用户画像
9. 基于用户画像和其他数据，生成用户个性化指标正常范围、评价语句和商品推荐
10. 用户报告传参、渲染、配置、部署
11. 推送用户个性化评价、推荐、综合报告
12. 人类专家协作


## 数据中心需求

数据中心表格表头

外包方读取数据中心方式

外包方写入数据中心方式

## AU需求

模型调用触发

用户状态解析数据库更改的消息队列 msg queue

用户状态解析数据库读取

主动推送的需求

- 推纯文本
- 推链接
- 推图片
- 推表情
- 推h5
- 推多媒体（视频，音乐等）
- *改ui风格*

## 专家端需求



## 开发demo数据中心

生物数据

- 血糖
- 心律

对话数据

- raw msg
- parsed msg

指标数据

- miwo 599
- 血糖健康度
