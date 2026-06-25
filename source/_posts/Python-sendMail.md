---
title: Pyhon发送邮件实践(自动化打包续)
date: 2017-09-01 11:19:53
tags: Python, 邮件, 自动化
categories: Python
---

## 起因
机器运行的越来慢，每天启动模拟器需要十几分钟，打个发布包也需要十分钟左右，实在受不了这个速度，一咬牙一跺脚就把机器格式化了(其实是作死行为，为后来半个月工作带来了不小的麻烦)。重装部分软件后发现我的打包脚本邮件有时候发送邮件到个别邮箱时，邮件无法准时到达。当时是怎么配置发送环境的资料也随电脑格式化烟消云散了。更不想再去重走回头路，按照原来的方法一步步去配置。最近也是在学习Python，有这样的一个机会学以致用使用Python发送邮件貌似不能够错过。
<!-- more -->
## 需求
- 1.发送邮件。能够正常、准时、稳定发送有标题、正文并且带有附件的邮件。
- 2.能够在Mac终端使用命令行操作操作。能与Shell打包脚本之间参数传递
- 3.发送邮件时候能够直观看到发送的进度，上传的网络速率(目前尚未实现)等等

## 实现过程
### 发送带附件的邮件
python使用smtp发送邮件。关于这个我觉得没有什么好说的，详情见[菜鸟教程](http://www.runoob.com/python/python-tutorial.html)和[Python 2.7教程](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)
### 获取邮件发送进度和网速
有时候我希望看到邮件的发送进度，但smtp自身并不能直接显示邮件发送的进度。我们可以通过每秒读取文件数据占总量百分比来实现需求。阅读其源码找到如下代码对其做一定拓展貌似能满足我们的需求。
```
def data(self,msg):
        """SMTP 'DATA' command -- sends message data to server.

        Automatically quotes lines beginning with a period per rfc821.
        Raises SMTPDataError if there is an unexpected reply to the
        DATA command; the return value from this method is the final
        response code received when the all data is sent.
        """
        self.putcmd("data")
        (code,repl)=self.getreply()
        if self.debuglevel > 0 : print "data:", (code,repl)
        if code != 354:
            raise SMTPDataError(code,repl)
        else:
            q = quotedata(msg)
            if q[-2:] != CRLF:
                q = q + CRLF
            q = q + "." + CRLF
            self.send(q)
            (code,msg)=self.getreply()
            if self.debuglevel >0 : print "data:", (code,msg)
            return (code,msg)
```
我们可以在其发送过程中获取每秒钟监测到发送的字节数，并将其输出出来，这样就能获取其发送进度和网速了.
```
          global starttime,sendByteBySec
          endtime = datetime.datetime.now()
          percent = 100. * progress / total
          stdout.write('\r')
          starttime = endtime;
          speedNum = (progress - sendByteBySec)/1024
          if (endtime - starttime).seconds >=1:
             # starttime = endtime;
             # speedNum = (progress - sendByteBySec)/1024
             stdout.write("%s bytes sent of %s [%2.0f%%]|Speed: [%2.0f%%]" % (progress, total, percent,speedNum))
             # sendByteBySec = progress
          else:
              # starttime = endtime;
              # speedNum = (progress - sendByteBySec)/1024
              stdout.write("%s bytes sent of %s [%2.0f%%]|Speed: [%2.0f%%]" % (progress, total, percent,speedNum))

          sendByteBySec = progress
          stdout.flush()
          # print (endtime - starttime).seconds
          if percent >= 100: stdout.write('\n')
```
完整代码如下

```
# !/usr/bin/python
# -*- coding: utf-8 -*-

# import smtplib
from smtplib import SMTP, quotedata, CRLF, SMTPDataError
from email import encoders
from email.header import Header
from email.mime.text import MIMEText
from email.utils import parseaddr, formataddr
from email.MIMEMultipart import MIMEMultipart
from email.mime.base import MIMEBase
from sys import stderr, stdout
import os,sys, time

class ExtendedSMTP(SMTP):
  def data(self, msg):
    self.putcmd("data")
    (code,repl)=self.getreply()
    if self.debuglevel > 0 : print >> stderr, "data:", (code, repl)
    if code != 354:
      raise SMTPDataError(code,repl)
    else:
      q = quotedata(msg)
      if q[-2:] != CRLF:
        q = q + CRLF
      q = q + "." + CRLF

      # begin modified send code
      chunk_size = 2048
      bytes_sent = 0

      while bytes_sent != len(q):
        chunk = q[bytes_sent:bytes_sent+chunk_size]
        self.send(chunk)
        bytes_sent += len(chunk)
        if hasattr(self, "callback"):
          self.callback(bytes_sent, len(q))
      # end modified send code

      (code,msg)=self.getreply()
      if self.debuglevel >0 : print>>stderr, "data:", (code,msg)
      return (code,msg)


def sendMail(filePath,fileName,appName):
    def _format_addr(s):
        name, addr = parseaddr(s)
        return formataddr(( \
            Header(name, 'utf-8').encode(), \
            addr.encode('utf-8') if isinstance(addr, unicode) else addr))

    def callback(progress, total):
          percent = 100. * progress / total
          stdout.write('\r')
          stdout.write("%s bytes sent of %s [%2.0f%%]" % (progress, total, percent))
          stdout.flush()
          if percent >= 100: stdout.write('\n')

    #参数配置
    from_addr = "xyhuangjia@yeah.net"
    password = "***"
    to_addr = ["example@example.com","example2@example.com"]
    smtp_server = "smtp.yeah.net"

    #邮件信息配置
    #正文
    content = u'%s新版本(版本号)的安装包已发送，请将本版本的需要变更之处用文本记录下来发送给本人，谢谢🙏'% sys.argv[3]
    #标题（使用传输过来的数据）
    subject = u'%s的安装包' % sys.argv[3]
    emailFrom = "鶸开发黄小佳"
    msg = MIMEMultipart()
    msg['From'] = _format_addr('%s<%s>' % (emailFrom,from_addr))
    msg['To'] = _format_addr(u'接受者 <%s>' % to_addr)
    msg['Subject'] = Header('%s' % subject, 'utf-8').encode()
    msg.attach(MIMEText('%s'%content, 'plain', 'utf-8'))

    mime = MIMEBase('application', 'octet-stream', filename=fileName)
    with open(filePath, 'rb') as f:
        # 加上必要的头信息:
        mime.add_header('Content-Disposition', 'attachment', filename=fileName)
        mime.add_header('Content-ID', '<0>')
        mime.add_header('X-Attachment-Id', '0')
        mime.set_payload(f.read())
        # 用Base64编码:
        encoders.encode_base64(mime)
        # 添加到MIMEMultipart:
        msg.attach(mime)

    try:
        server = ExtendedSMTP()
        server.callback = callback
        server.connect(smtp_server, 25)
    	# server.set_debuglevel(1)#坑爹的调试模式，打开输出一万行坑了我四天
    	server.login(from_addr, password)
    	server.sendmail(from_addr, to_addr, msg.as_string())
    	server.quit()
    	return True
    except Exception as e:
        raise e
    	return False


if __name__ == '__main__':

    # print sys.argv
    if sendMail(sys.argv[1], sys.argv[2], sys.argv[3]):
        print "\033[32;1m 邮件已发送!\033[0m"
    else:
        print "邮件发送失败!"
```
## 注意：
没事不要开什么调试模式`server.set_debuglevel(1)`打开后会输出代码执行过程，在含有大附件的邮件在发送时会严重的拖慢进度，要注意这个
## 参考文章链接
[Python发送以整个文件夹的内容为附件的邮件的教程](http://m.jb51.net/article/65567.htm)

[smtplib](http://svn.python.org/projects/python/tags/r22a3/Lib/smtplib.py)
