## 微信机器人

基本原理是模拟web微信登录，通过研究接口制作出的SDK，注意，该SDK的功能有限但是够用

#### itchat

封装好的python库，项目主页 [http://itchat.readthedocs.io/zh/latest/](http://itchat.readthedocs.io/zh/latest/)

本次学习试图建立一种可交互推送机制，目前现有平台稳定的方式有邮件，短信和微信。介于微信的后台比较硬。。。

itchat的demo本身比较完整，接收发送消息等，要想达到可交互，必须解决一个问题，就是如果向已运行的python程序发送指令，本次使用socket来做

#### code

```py
import itchat
import socket
from threading import Thread
from itchat.content import *

MAX_LENGTH = 4096
serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

PORT = 10000
HOST = '127.0.0.1'


def login():
    itchat.auto_login(enableCmdQR=2, loginCallback=callback_login())
    itchat.run(blockThread=False)

@itchat.msg_register([TEXT, MAP, CARD, NOTE, SHARING])
def text_reply(msg):
    if msg['FromUserName'] == "filehelper":
        itchat.send('%s: %s' % (msg['Type'], msg['Text']), msg['FromUserName'])

def callback_login():
    print "login ok"

def handle(clientsocket):
    while 1:
        buf = clientsocket.recv(MAX_LENGTH)
        if buf == '': return  # client terminated connection
        send_msg(buf)

def channel():
    serversocket.bind((HOST, PORT))
    serversocket.listen(10)
    while 1:
        #accept connections from outside
        (clientsocket, address) = serversocket.accept()

        ct = Thread(target=handle, args=(clientsocket,))
        ct.run()


def send_msg(msg):
    if msg == "login":
        login()
        return
    itchat.send(msg, toUserName='filehelper')

if __name__ == '__main__':
    channel()
```

```py
import socket
import sys

HOST = '127.0.0.1'
PORT = 10000
s = socket.socket()
s.connect(HOST, PORT)

while 1:
    msg = raw_input("Command To Send: ")
    if msg == "close":
       s.close()
       sys.exit(0)
    s.send(msg)
```

#### 其他人的工作

无意间看到，做的意思差不多 ： [https://github.com/bluedazzle/wechat\_sender](https://github.com/bluedazzle/wechat_sender)



```js
{//根据UserName查询用户信息
    "UserName": "@63a028af148fb002247be83ce73ae30d", 
    "City": "\u5f90\u6c47", 
    "DisplayName": "", 
    "UniFriend": 0, 
    "OwnerUin": 0, 
    "MemberList": [], 
    "PYQuanPin": "jinxiaogangyaozaoqikanrenjian", 
    "RemarkPYInitial": "", 
    "Uin": 1298147140, 
    "AppAccountFlag": 0, 
    "VerifyFlag": 0, 
    "Province": "\u4e0a\u6d77", 
    "KeyWord": "sjt", 
    "RemarkName": "", 
    "PYInitial": "JXGYZQKRJ", 
    "ChatRoomId": 0, 
    "IsOwner": 0, 
    "HideInputBarFlag": 0, 
    "HeadImgFlag": 1, 
    "EncryChatRoomId": "", 
    "AttrStatus": 33751487, 
    "SnsFlag": 49, 
    "MemberCount": 0, 
    "WebWxPluginSwitch": 0, 
    "Alias": "", 
    "Signature": "\u6211\u8be5\u5982\u4f55\u53bb\u4e86\u89e3", 
    "ContactFlag": 7, 
    "NickName": "\u91d1\u5c0f\u521a\u8981\u65e9\u8d77\u770b\u4eba\u95f4", //金小刚要早起看人间
    "RemarkPYQuanPin": "", 
    "HeadImgUrl": "/cgi-bin/mmwebwx-bin/webwxgeticon?seq=1263700&username=@63a028af148fb002247be83ce73ae30d&skey=@crypt_2b538ced_b9d6556484ab9487e1ed6de2a8d5f181", 
    "Sex": 1, 
    "StarFriend": 0, 
    "Statues": 0
}
```

```js
{//自己发送消息到群组
    "AppInfo": {
        "Type": 0, 
        "AppID": ""
    }, 
    "ImgWidth": 0, 
    "FromUserName": "@63a028af148fb002247be83ce73ae30d", 
    "PlayLength": 0, 
    "OriContent": "", 
    "ImgStatus": 1, 
    "RecommendInfo": {
        "UserName": "", 
        "Province": "", 
        "City": "", 
        "Scene": 0, 
        "QQNum": 0, 
        "Content": "", 
        "Alias": "", 
        "OpCode": 0, 
        "Signature": "", 
        "Ticket": "", 
        "Sex": 0, 
        "NickName": "", 
        "AttrStatus": 0, 
        "VerifyFlag": 0
    }, 
    "Content": "test", 
    "MsgType": 1, 
    "ImgHeight": 0, 
    "StatusNotifyUserName": "", 
    "StatusNotifyCode": 0, 
    "Type": "Text", 
    "NewMsgId": 6904308120000518543, 
    "Status": 3, 
    "VoiceLength": 0, 
    "MediaId": "", 
    "MsgId": "6904308120000518543", 
    "ToUserName": "@@cfd5b07f33fed5cb46c375b3abb9c861e4db31f723aa5ce00cfa0adad203cbc9", 
    "ForwardFlag": 0, 
    "FileName": "", 
    "Url": "", 
    "HasProductId": 0, 
    "FileSize": "", 
    "AppMsgType": 0, 
    "Text": "test", 
    "Ticket": "", 
    "CreateTime": 1499235985, 
    "SubMsgType": 0
}
```

```

```



