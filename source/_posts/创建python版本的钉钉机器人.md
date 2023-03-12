---
title: 创建python版本的钉钉机器人
date: 2023-03-12 16:19:03
categories: 
- 有趣
---

什么年代了还在用传统划水网站？来试试这款chatGPT钉钉机器人吧。chatGPT已经火了几个月了，github上面各种GPT插件和机器人🤖️也是层出不穷，俺随大流也做了一个python版本的钉钉机器人，最近感觉服务差不多稳定了，所以在这里分享给大家

# ★目录
[★前提](#head-1)
[★效果展示](#head-2)
[★创建钉钉机器人](#head-3)
[★加入chatGPT](#head-4)
[★结束](#head-5)

<a id="head-1"></a>
# ★前提
1. 有钉钉管理员权限，没有的话自己建一个测试公司拉小伙伴进来一起划水
2. 有服务器，python3.9以上的环境或者使用docker
3. 有chatGPT的session
4. 有一些python和服务器的基础知识

俗话说巧妇难为无米之炊，钉钉管理员、服务器、chatGPT的session这三个是必须滴，如果不了解python的话，也可以用下面提到的nodejs或者.NET库

<a id="head-2"></a>
# ★效果展示
如果你已经满足了上面的条件，想立即体验的话可以直接克隆到服务器 [dingtalk-chatgpt-bot](https://github.com/XueMeijing/dingtalk-chatgpt-bot) ，修改config.js配置后就可以使用了
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2eb627ae7f244cb933b153021d1c684~tplv-k3u1fbpfcp-watermark.image?)

<a id="head-3"></a>
# ★创建钉钉机器人

## ◇什么是钉钉机器人
[官方文档](https://open.dingtalk.com/document/orgapp/robot-overview) 说：在钉钉，机器人是独立存在的一个应用类型，可以开箱即用，也可以进行二次开发，无需和微应用或者群等场景进行强制绑定。
官方说的有点绕，在俺的理解中，钉钉机器人就是一个代理服务，可以把你的消息转发给第三方，也可以从第三方再由机器人转发回来。机器人通常用来做消息推送或者资料查询
我当时是跟着 [老表](https://xie.infoq.cn/article/0d31f8da82b3191a993e50054) 的教程来的，改了一部分东西

## ◇创建机器人

1. 创建公司
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8e149c78b164bb6b89f397001918146~tplv-k3u1fbpfcp-watermark.image?) 
2. 登录开发者后台，按照如下图示顺序创建应用，提示选择新版和旧版的话选择旧版，注意应用名不能有chatGPT
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfaf47cafebc400cb015114a100acdf8~tplv-k3u1fbpfcp-watermark.image?)
3. 更改配置，这时候保存不了，ip对应的服务还没有启动，我们等下面服务启动之后再来进行这个
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a85d15a112b848bbb32bf3cd45e5dda8~tplv-k3u1fbpfcp-watermark.image?)
4. 点击调试，会创建测试群，测试通过之后上线
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0e4bd1fb05548bf99cdd0791c022d6d~tplv-k3u1fbpfcp-watermark.image?)
5. 在群聊里找到智能群助手，添加机器人，然后@机器人就可以进行玩耍了

## ◇开启服务

1. 安装quart（类似flask，不过可以进行异步处理）
  ```
  pip3 install quart
  ```
2. 创建index.py，写入如下代码
  ```python
  from quart import Quart

  app = Quart(__name__)

  @app.route('/', methods=['GET', 'POST'])
  async def get_data():
    return 'Hello world'

  if __name__ == '__main__':
      app.run(host='127.0.0.1', port=8083)
  ```
3. 开启服务，打开 http://127.0.0.1:8083/ 就能看到熟悉的hello world了，很简单对吧？
  ```
  python3 index.py
  ```
  ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c78985b4e164afa9612f842a8770b1a~tplv-k3u1fbpfcp-watermark.image?)
4. 需要注意的是，127.0.0.1是本地开发调试用的，如果部署到服务器，需要改成0.0.0.0端口，并开启网络防火墙，这部分我也不多说了，需要的这看 [老表](https://xie.infoq.cn/article/3340770024c49b5b1a54597d5) 的原文吧
5. 全部代码如下, 更改```app_secret```为机器人应用信息里的app_secret
  <details>
  <summary>展开查看完整代码</summary>
  ```python
  import base64
  import hmac
  import hashlib
  import requests
  import datetime
  from quart import Quart, request

  app = Quart(__name__)

  @app.route('/', methods=['GET', 'POST'])
  async def get_data():
      # 第一步验证：是否是post请求
      if request.method == "POST":
          try:
              # 签名验证 获取headers中的Timestamp和Sign
              req_data = await request.get_json()
              timestamp = request.headers.get('Timestamp')
              sign = request.headers.get('Sign')
              print('request.data-----\n', req_data)
              # 第二步验证：签名是否有效
              if check_sig(timestamp) == sign:
                  print('签名验证成功-----')
                  # 调用数据处理函数
                  await handle_info(req_data)
                  return str(req_data)
              else:
                  result = '签名验证失败-----'
                  print(result)
                  return result
          except Exception as e:
              result = '出错啦～～'
              print('error', repr(e))
          return str(result)
      return '钉钉机器人:' + str(datetime.datetime.now())

  # 处理自动回复消息
  async def handle_info(req_data):
      # 解析用户发送消息 通讯webhook_url 
      text_info = req_data['text']['content'].strip()
      webhook_url = req_data['sessionWebhook']
      senderid = req_data['senderId']
      answer = '测试成功：' + text_info
      # 调用函数，发送markdown消息
      send_md_msg(senderid, answer, webhook_url)

  # 发送markdown消息
  def send_md_msg(userid, message, webhook_url):
      '''
      userid: @用户 钉钉id
      title : 消息标题
      message: 消息主体内容
      webhook_url: 通讯url
      '''
      message = '<font color=#008000>@%s </font>  \n\n %s' % (userid, message)
      title = '大聪明说'

      data = {
          "msgtype": "markdown",
          "markdown": {
              "title":title,
              "text": message
          },
          "at": {
              "atDingtalkIds": [
                  userid
              ],
          }
      }
      # 利用requests发送post请求
      req = requests.post(webhook_url, json=data)

  # 消息数字签名计算核对
  def check_sig(timestamp):
      app_secret = 'BIQ7O8AqNMRiHrW....'
      app_secret_enc = app_secret.encode('utf-8')
      string_to_sign = '{}\n{}'.format(timestamp, app_secret)
      string_to_sign_enc = string_to_sign.encode('utf-8')
      hmac_code = hmac.new(app_secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
      sign = base64.b64encode(hmac_code).decode('utf-8')
      return sign

  if __name__ == '__main__':
      app.run(host='0.0.0.0', port=8083)
  ```
  </details>

## ◇测试效果
部署成功后再回到机器人配置页面，这时候配置应该就能保存成功了，回到版本管理与发布中点击调试，会创建调试群，这时候@机器人就能收到消息了，结果如下
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ad302efdd2d4a9ba9140cf19e24139f~tplv-k3u1fbpfcp-watermark.image?)

<a id="head-4"></a>
# ★加入chatGPT

如果你测试机器人能收到消息之后，下一步需要做的就是把handle_info的回复改成chatGPT的回复。

## ◇请求代理库PyGPT
这里使用的是 [PawanOsman](https://github.com/PawanOsman/PyGPT) 开发的一个python库，他似乎突破了openAI的某些限制，可以代理我们的请求到  https://chat.openai.com/chat ，看起来就像是在使用网页请求一样，并且请求的历史也可以在官网上看到。所以不像是openAI的官方库那么笨，包括GPT3.5。如果你不是一个python开发者，你也可以使用他的 [nodeJs库](https://github.com/PawanOsman/chatgpt-io) 或者 [.Net库](https://github.com/PawanOsman/ChatGPT.Net) 自行开发非python的机器人

库的使用很简单，如demo所示，把pyGPT的参数修改成自己的session就可以了
```python
import asyncio
from pygpt import PyGPT

async def main():
    chat_gpt = PyGPT('eyJhbGciOiJkaXIiLCJlbmMiOiJBMR0NN....')
    await chat_gpt.connect()
    await chat_gpt.wait_for_ready()
    answer = await chat_gpt.ask('What is the capital of France?')
    print(answer)
    await chat_gpt.disconnect()

if __name__ == '__main__':
    asyncio.run(main())
```
修改handle_info中的answer为chatGPT的回复
```
# 处理自动回复消息
async def handle_info(req_data):
    # 解析用户发送消息 通讯webhook_url 
    text_info = req_data['text']['content'].strip()
    webhook_url = req_data['sessionWebhook']
    senderid = req_data['senderId']
    
    # 请求GPT回复，失败重新请求三次
    retry_count = 0
    max_retry_count = 3
    while retry_count < max_retry_count:
        try:
            chat_gpt = PyGPT('eyJhbGciOiJkaXIiLCJlbmMiOiJBMR0NN....')
            await chat_gpt.connect()
            await chat_gpt.wait_for_ready()
            answer = await chat_gpt.ask(text_info)
            await chat_gpt.disconnect()
            print('answer:\n', answer)
            print('--------------------------')
            break
        except Exception as e:
            retry_count = retry_count + 1
            print('retry_count', retry_count)
            print('error\n', repr(e))
            continue
    if not answer:
        answer = '请求接口失败，请稍后重试'
    # 调用函数，发送markdown消息
    send_md_msg(senderid, answer, webhook_url)
```

有一点需要注意的是，如果我们在钉钉转发过来的http请求里不断的执行上面的代码，每次调用PyGPT都会产生一个新的连接，作者的代理服务器会hold住连接，超过50个socket连接或者短时间内请求太频繁，会被拉黑1～5分钟。所以像这样修改一下代码，在http循环外部创建chat_gpt对象
```
app = Quart(__name__)
# 定义全局对象
chat_gpt = None
...
async def handle_info(req_data):
    ...
    global chat_gpt
    while retry_count < max_retry_count:
        try:
            if chat_gpt is None:
                connect_task = asyncio.create_task(init_connect())
                await connect_task
            answer = await chat_gpt.ask(text_info, senderid)
            print('answer:\n', answer)
            print('--------------------------')
            break
        except Exception as e:
            retry_count = retry_count + 1
            print('retry_count', retry_count)
            print('error\n', repr(e))
            answer = ''
            if retry_count == 2:
                connect_task = asyncio.create_task(init_connect())
                await connect_task
            continue
    if not answer:
        answer = '请求接口失败，请稍后重试'
```
init_connect函数内容如下
```
async def init_connect():
    # 建立连接
    retry_count = 0
    max_retry_count = 3

    while retry_count < max_retry_count:
        try:
            global chat_gpt
            chat_gpt = PyGPT('eyJhbGciOiJkaXIiLCJlbmMiOiJBMR0NN....')
            await chat_gpt.connect()
            await chat_gpt.wait_for_ready()
            break
        except Exception as e: 
            retry_count = retry_count + 1
            print('retry_count', retry_count)
            print('error\n', repr(e))
            continue
```
为了以后修改配置方便，我们可以把GPT_SESSION和APP_SECRET放到一个config.py文件里并导出 
```
GPT_SESSION = ''

APP_SECRET = ''

__all__ = [
  GPT_SESSION,
  APP_SECRET,
]
```
此时index.py的完整代码如下，功能已经可以正常使用了！

<details>
<summary>展开查看完整代码</summary>
```
import base64
import hmac
import hashlib
import requests
from pygpt import PyGPT
import datetime
from quart import Quart, request
import asyncio

import config

app = Quart(__name__)

chat_gpt = None

@app.route('/', methods=['GET', 'POST'])
async def get_data():
    # 第一步验证：是否是post请求
    if request.method == "POST":
        try:
            # 签名验证 获取headers中的Timestamp和Sign
            req_data = await request.get_json()
            timestamp = request.headers.get('Timestamp')
            sign = request.headers.get('Sign')
            print('request.data-----\n', req_data)
            # 第二步验证：签名是否有效
            if check_sig(timestamp) == sign:
                print('签名验证成功-----')
                # 调用数据处理函数
                await handle_info(req_data)
                return str(req_data)
            else:
                result = '签名验证失败-----'
                print(result)
                return result
        except Exception as e:
            result = '出错啦～～'
            print('error', repr(e))
        return str(result)
    return '钉钉机器人:' + str(datetime.datetime.now())

# 处理自动回复消息
async def handle_info(req_data):
    # 解析用户发送消息 通讯webhook_url 
    text_info = req_data['text']['content'].strip()
    webhook_url = req_data['sessionWebhook']
    senderid = req_data['senderId']
    # 请求GPT回复，失败重新请求三次
    retry_count = 0
    max_retry_count = 3
    global chat_gpt
    while retry_count < max_retry_count:
        try:
            if chat_gpt is None:
                connect_task = asyncio.create_task(init_connect())
                await connect_task
            answer = await chat_gpt.ask(text_info, senderid)
            print('answer:\n', answer)
            print('--------------------------')
            break
        except Exception as e:
            retry_count = retry_count + 1
            print('retry_count', retry_count)
            print('error\n', repr(e))
            answer = ''
            if retry_count == 2:
                connect_task = asyncio.create_task(init_connect())
                await connect_task
            continue
    if not answer:
        answer = '请求接口失败，请稍后重试'
    # 调用函数，发送markdown消息
    send_md_msg(senderid, answer, webhook_url)

# 发送markdown消息
def send_md_msg(userid, message, webhook_url):
    '''
    userid: @用户 钉钉id
    title : 消息标题
    message: 消息主体内容
    webhook_url: 通讯url
    '''
    message = '<font color=#008000>@%s </font>  \n\n %s' % (userid, message)
    title = '大聪明说'

    data = {
        "msgtype": "markdown",
        "markdown": {
            "title":title,
            "text": message
        },
        # "msgtype": "text",
        # "text": {
        #     "content": message
        # },
        "at": {
            "atDingtalkIds": [
                userid
            ],
        }
    }
    # 利用requests发送post请求
    req = requests.post(webhook_url, json=data)

# 消息数字签名计算核对
def check_sig(timestamp):
    app_secret = config.APP_SECRET
    app_secret_enc = app_secret.encode('utf-8')
    string_to_sign = '{}\n{}'.format(timestamp, app_secret)
    string_to_sign_enc = string_to_sign.encode('utf-8')
    hmac_code = hmac.new(app_secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
    sign = base64.b64encode(hmac_code).decode('utf-8')
    return sign

async def init_connect():
    # 建立连接
    retry_count = 0
    max_retry_count = 3

    while retry_count < max_retry_count:
        try:
            global chat_gpt
            chat_gpt = PyGPT(config.GPT_SESSION)
            await chat_gpt.connect()
            await chat_gpt.wait_for_ready()
            break
        except Exception as e: 
            retry_count = retry_count + 1
            print('retry_count', retry_count)
            print('error\n', repr(e))
            continue

if __name__ == '__main__':
    # 指定host和port，0.0.0.0可以运行在服务器上对外访问，记得开服务器的网络防火墙端口
    # GCP在VPC network -> firewalls -> 增加一条 VPC firewall rules 指定端口，target填 http-server或https-server
    app.run(host='0.0.0.0', port=8083)
```
</details>

## ◇增加上下文功能
经过使用俺发现此时每次聊天都相当于在官网上重新打开一个聊天窗口，没有上下文的功能。经过调试发现chatGPT的接口和pygpt的源码有一些联系，pygpt的self.socket.call返回对象包括conversationId，messageId，answer，而conversationId正是 [openai](https://chat.openai.com/chat) 地址后面的某个对话的id，messageId是对话内上一条回复的parentId，把官网的参数替换到socket.call的参数里，可以完美衔接上一条对话，有了这个关系做上下文语境就简单多了

这里俺用的是python自带的轻量级数据库sqlite3，
 1. pygpt请求之前的时候带上senderid参数
 2. pygpt响应之前看数据库有没有这个用户，有的话就socket.call使用用户的conversation_id、parent_id，没有就使用pygpt默认的随机数。
 3. 获取pygpt响应后，新用户的话就以senderid为主键保存一条数据（id、conversation_id、parent_id）。已经存在的话就把响应的messageId更新到parent_id。

新建一个sql.py，代码如下，用来导出sql函数
```
import sqlite3

DATABASE = 'database.db'

# 查询结果元组转字典
def dict_factory(cursor, row):
  d = {}
  for idx, col in enumerate(cursor.description):
      d[col[0]] = row[idx]
  return d

# 初始化数据库
def init_db():
  db = sqlite3.connect(DATABASE, check_same_thread=False)
  cursor = db.cursor()
  create_table_query = '''  CREATE TABLE IF NOT EXISTS user(
                            id                TEXT PRIMARY KEY     NOT NULL,
                            name              TEXT                        ,
                            conversation_id   TEXT                 NOT NULL,
                            parent_id         TEXT                 NOT NULL,
                            create_at          timestamp            NOT NULL); '''
  cursor.execute(create_table_query)
  cursor.close()
  db.close()
  print('数据库初始化成功')

# 获取数据库
def get_db():
  db = sqlite3.connect(DATABASE, check_same_thread=False)
  db.row_factory = dict_factory
  return db

# 执行sql语句
def query_db(query, args=(), one=False):
  db = get_db()
  cur = db.cursor()
  cur.execute(query, args)
  rv = cur.fetchall()
  db.commit()
  cur.close()
  db.close()
  return (rv[0] if rv else None) if one else rv

__all__ = [
  init_db,
  query_db
]
```
初始化数据库
```
app = Quart(__name__)

init_db()

chat_gpt = None
...
```
传递 query_db senderid 参数
```python
if chat_gpt is None:
        connect_task = asyncio.create_task(init_connect())
        await connect_task
    answer = await chat_gpt.ask(text_info, query_db, senderid)
    print('answer:\n', answer)
```
把pygpt的源码复制到本地，使用sqlite3保存、更新数据
```python
async def ask(self, prompt, query_db, id='default'):
        if not self.auth or not self.validate_token(self.auth):
            await self.get_tokens()
        conversation = self.get_conversation_by_id(id)
        
        sqlite_get_data_query = """ SELECT * FROM user WHERE id = ? """
        user_record = query_db(sqlite_get_data_query, (id,), True)
        print('user_record', user_record)

        # Fix for timeout issue by Ulysses0817: https://github.com/Ulysses0817
        data = await self.socket.call(event='askQuestion', data={
            'prompt': prompt,
            'parentId': user_record['parent_id'] if user_record else str(conversation['parent_id']),
            'conversationId': user_record["conversation_id"] if user_record else str(conversation['conversation_id']),
            'auth': self.auth
        }, timeout=self.timeout)
        print('ask data---\n', data)
        if 'error' in data:
            print(f'Error: {data["error"]}')
            return f'Error: {data["error"]}'
        try:
            if user_record is None:
                # 插入数据
                sqlite_insert_data_query = """  INSERT INTO user
                                                ('id', 'name', 'conversation_id', 'parent_id', 'create_at')
                                                VALUES (?,?,?,?,?);  """
                query_db(sqlite_insert_data_query, (id, None, data['conversationId'], data['messageId'], datetime.datetime.now()))
                print('插入数据')
            else:
                # 更新数据
                sqlite_update_data_query = """ UPDATE user SET id = ?, name = ?, conversation_id = ?, parent_id = ?, create_at = ? WHERE id = ? """
                query_db(sqlite_update_data_query, (id, None, data['conversationId'], data['messageId'], datetime.datetime.now(), id))
                print('更新数据')
        except Exception as e:
            print('database error\n', repr(e))
        conversation['parent_id'] = data['messageId']
        conversation['conversation_id'] = data['conversationId']
        return data['answer']
```

要是增加一条新的对话怎么办呢，就增加一个/reset命令，删掉那个用户的数据，下次他请求就会打开新聊天窗口了
```
# 处理自动回复消息
async def handle_info(req_data):
    ...
    # 打开新聊天窗口
    if (text_info == '/reset'):
        sqlite_delete_data_query = """ DELETE FROM 'user' WHERE id = ? """
        query_db(sqlite_delete_data_query, (senderid,))
        send_md_msg(senderid, '聊天上下文已重置', webhook_url)
        return
```

## ◇后台运行
<b>注意：</b>我们服务此时在前台运行，如果我们关闭命令行窗口，服务就停止了，要想服务在后台运行并且方便的查看日志，我们可以使用nohup命令，输出的日志保存在nohup.out文件里
```
nohup python3 -u index.py > nohup.out 2>&1 &
```
查看最新30条日志使用tail命令，ctrl+c退出查看日志
```
tail -30f nohup.out
```

<a id="head-5"></a>
# ★结束
以上就是俺划水踩坑的全部内容了，完整代码在 [dingtalk-chatgpt-bot](https://github.com/XueMeijing/dingtalk-chatgpt-bot)，第一次发文，才疏学浅，要是有不足之处还请多多指正