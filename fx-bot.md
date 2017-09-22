# fx bot

## 爬虫

解析url比较简单

## DB

存储历史任务，由于外汇的数据比较少，所以使用`pickledb`来存储，一般就几项。主要是计算触发条件使用。

会员信息使用的是`mysql`，部署使用的`phpmyadmin`，比较粗暴的方案

## 定时任务

`apscheduler`使用起来竟然不稳定，运行2，3天就断了，最后仍然采用操作系统级别的定时`cron`。就是不能记住上下文，感觉不是很舒服。

```bash
//crontab -l / -e
* * * * * /usr/sbin/fx-bot
* * * * * sleep 10;/usr/sbin/fx-bot
* * * * * sleep 20;/usr/sbin/fx-bot
* * * * * sleep 30;/usr/sbin/fx-bot
* * * * * sleep 40;/usr/sbin/fx-bot
* * * * * sleep 50;/usr/sbin/fx-bot

//fx-bot，重定向到当前日期下的log文件
python ~/python/fx.py >> ~/python/`date +%Y-%m-%d.log`
```

## 短信

使用短信平台`api`，注意短信号码的个数限制\(`sendOnce`是`1000`条，使用半角逗号隔开手机号\)

## 邮件

阿里云禁掉了`25`端口，所以要使用`ssl`方式发送，然而`envelopes`竟然不支持`ssl`方式，只能使用原始接口

```
def send_mail_ssl(address, title, content):
    message = MIMEText(content, 'plain', 'utf-8')
    message['From'] = Header('FX-BOT<fxbot@acefx.com.cn>', 'utf-8')
    message['To'] = Header('VIPs', 'utf-8')
    message['Subject'] = Header(title, 'utf-8')

    from_address = 'fxbot@****'
    psw = "****"
    try:
        smtp = smtplib.SMTP_SSL("smtp.exmail.qq.com", 465)
        smtp.set_debuglevel(0)
        smtp.login(from_address, psw)
        smtp.sendmail(from_address, address, message.as_string())
        smtp.quit()
        print '邮件发送成功！'
    except smtplib.SMTPConnectError, e:
        print '邮件发送失败，连接失败:', e.smtp_code, e.smtp_error

    except smtplib.SMTPAuthenticationError, e:
        print '邮件发送失败，认证错误:', e.smtp_code, e.smtp_error
    except smtplib.SMTPSenderRefused, e:
        print '邮件发送失败，发件人被拒绝:', e.smtp_code, e.smtp_error
    except smtplib.SMTPRecipientsRefused, e:
        print '邮件发送失败，收件人被拒绝:', e.smtp_code, e.smtp_error
    except smtplib.SMTPDataError, e:
        print '邮件发送失败，数据接收拒绝:', e.smtp_code, e.smtp_error
    except smtplib.SMTPException, e:
        print '邮件发送失败, ', e.message
    except Exception, e:
        print '邮件发送异常, ', str(e)
```





