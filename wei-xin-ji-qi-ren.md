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



