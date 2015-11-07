layout: post
title: Java发送邮件出现"no object dch for mime type multipart/mixed"异常的解决办法及过程
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [mailcap,java,engineered,email]
date: 2015-10-30 18:00:50
keywords: mailcap,java,engineered,email, no object dch for mime type multipart/mixed"异常的解决办法及过程
---

![email error](/images/blog/email-error/error.jpg)

前两天写了一个发送邮件的功能，结果出现了一个比较灵异的状况，现在整理一下解决办法和中间的过程。

<!-- more -->

# 发送邮件配置及代码

通过Java来发送邮件现在已经是非常方便就实现的功能。只需要使用`commons-email`这个第三方的包就可以很轻松的完成功能。

`commons-email`的`maven`依赖配置

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-email</artifactId>
    <version>1.3.3</version>
</dependency>
```

它会传递依赖`mail`和`activation`这两个包

```
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>mail</artifactId>
    <version>1.4.5</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
 </dependency>
```

发送邮件的代码也非常简单：

```
    /**
     * 发送邮件
     *
     * @param recipients
     * @param subject
     * @param content
     * @throws TException
     */
    public void sendMails(List<Recipient> recipients, String subject, String content) {
        try {
            List<InternetAddress> addresses = getReplys(recipients);
            if(log.isInfoEnabled()){
                log.info("send mail to recipients:'{}' subject:'{}'", addresses, subject);
            }
            if(CollectionUtils.isNotEmpty(addresses)){
                Email email = settingEmail(new HtmlEmail());
                email.setSubject(subject);
                email.setMsg(content);
                email.setTo(addresses);
                email.send();
            }
        } catch (EmailException e) {
            e.printStackTrace();
        }

    }
    
     /**
     * 设置邮件配置
     *
     * @param email
     * @return
     * @throws EmailException
     */
    private Email settingEmail(Email email) throws EmailException {
        email.setHostName(mailConfig.getProperty("mail.host"));
        email.setCharset(mailConfig.getProperty("mail.charset"));
        email.setSSLOnConnect(true);
        email.setAuthentication(mailConfig.getProperty("mail.username"), mailConfig.getProperty("mail.password"));
        email.setFrom(mailConfig.getProperty("mail.from"));
        return email;
    }
```

OK，代码写完了，单元测试也没有问题，一般情况下，就认为这个功能完成了，那么就打包部署吧。我现在使用了`Thrift`作为RPC框架，没有想到部署到测试环境中发现使用`Thrift`客户端调用发送邮件服务却无法正常发送邮件。

# 灵异状况

查看日志，却发现出现了下面的异常。

```
Caused by: javax.activation.UnsupportedDataTypeException: no object DCH for MIME type multipart/mixed;
	boundary="----=_Part_0_1434202756.1446193111088"
	at javax.activation.ObjectDataContentHandler.writeTo(DataHandler.java:896)
	at javax.activation.DataHandler.writeTo(DataHandler.java:317)
	at javax.mail.internet.MimeBodyPart.writeTo(MimeBodyPart.java:1485)
	at javax.mail.internet.MimeMessage.writeTo(MimeMessage.java:1773)
	at com.sun.mail.smtp.SMTPTransport.sendMessage(SMTPTransport.java:1121)
	... 17 more
```

当时的第一反应，是否是邮件服务器那边做了调整，出现新的状况？毕竟之前的单元测试也没有问题。

那重新写一个新的单元测试看看是怎么回事。

```
    @Test
    public void testSendMails(){
        List<Recipient> recipients = Lists.newArrayList(
                new Recipient("tonydeng","tonydeng@github.com","","")
        );

        notifyUtils.sendMails(recipients,"test","content");
    }
```

跑了好几次，都可以正常发送邮件，那是怎么回事？难道是通过thrift之后，中间有些信息发生变化？

将代码通过maven打包成功可执行的jar，启动thrift server，在调用Agent项目的单元测试，果然出现**“no object DCH for MIME type multipart/mixed;”**的异常。

果然很灵异，一般来说，在邮件服务的单元测试跑通之后，这个功能就应该是没有问题了。

现在出现了在服务端写的单元测试可以跑通，客户端的单元测试却出现异常。

# 解决办法

翻墙通过Google查了一下，主要的原因都是因为`mailcap`配置的问题。解决方案有两个。

## 一个是在代码中加上一段mailcap配置

```
        MailcapCommandMap mc = (MailcapCommandMap) CommandMap.getDefaultCommandMap();
        mc.addMailcap("text/html;; x-java-content-handler=com.sun.mail.handlers.text_html");
        mc.addMailcap("text/xml;; x-java-content-handler=com.sun.mail.handlers.text_xml");
        mc.addMailcap("text/plain;; x-java-content-handler=com.sun.mail.handlers.text_plain");
        mc.addMailcap("multipart/*;; x-java-content-handler=com.sun.mail.handlers.multipart_mixed");
        mc.addMailcap("message/rfc822;; x-java-content-handler=com.sun.mail.handlers.message_rfc822");
        CommandMap.setDefaultCommandMap(mc);
```

## 一个是将javax.mail下的mailcap相关文件复制到当前项目的`META-INF`目录下。

查看了一下`MailcapCommandMap`源码，在`META-INF`中加入`mailcap`配置文件，也应该能够和在代码中加入`mailcap`配置起到一样的作用。

附上`MailcapCommandMap`初始化方法：

```
public MailcapCommandMap() {
        ArrayList dbv = new ArrayList(5);
        MailcapFile mf = null;
        dbv.add((Object)null);
        LogSupport.log("MailcapCommandMap: load HOME");

        String ex;
        try {
            ex = System.getProperty("user.home");
            if(ex != null) {
                String path = ex + File.separator + ".mailcap";
                mf = this.loadFile(path);
                if(mf != null) {
                    dbv.add(mf);
                }
            }
        } catch (SecurityException var7) {
            ;
        }

        LogSupport.log("MailcapCommandMap: load SYS");

        try {
            ex = System.getProperty("java.home") + File.separator + "lib" + File.separator + "mailcap";
            mf = this.loadFile(ex);
            if(mf != null) {
                dbv.add(mf);
            }
        } catch (SecurityException var6) {
            ;
        }

        LogSupport.log("MailcapCommandMap: load JAR");
        this.loadAllResources(dbv, "META-INF/mailcap");
        LogSupport.log("MailcapCommandMap: load DEF");
        synchronized(MailcapCommandMap.class) {
            if(defDB == null) {
                defDB = this.loadResource("/META-INF/mailcap.default");
            }
        }

        if(defDB != null) {
            dbv.add(defDB);
        }

        this.DB = new MailcapFile[dbv.size()];
        this.DB = (MailcapFile[])((MailcapFile[])dbv.toArray(this.DB));
    }
```

按照我的习惯，能够配置的就不要写在代码里了，看样子正解应该是在`META-INF`文件夹中加入`mailcap`配置文件了。

于是从`activation`包中将mailcap.default文件拷贝到`META-INF`中，并更新了maven打包的配置。

结果，还是不行？

难道还是要在代码中加入`mailcap`的配置？尝试了一下在代码中加入相关配置代码，的确可以正常发送邮件，但是这样的解决办法是在是太不爽了。

# 那继续看看还有哪儿出问题了。

于是我在发送邮件之前将当期的支持的mimetype都打印出来看看。

```
     if(log.isInfoEnabled()){
            MailcapCommandMap mc = (MailcapCommandMap) CommandMap.getDefaultCommandMap();
            log.info("mail mimetypes:'{}' multipart/mixed:'{}' ", mc.getMimeTypes(), mc.getAllCommands("multipart/mixed"));
     }
```

发现在server端单元测试的输出和打包后的输出有很大的差异。

* 单元测试的输出

```
mail mimetypes:'[text/html, message/rfc822, multipart/*, text/xml, text/plain, text/*, image/jpeg, image/gif]' multipart/mixed:'[javax.activation.CommandInfo@349859e0]'
```

* 打包后的输出

```
mail mimetypes:'text/*'
```

看样子应该是打包之后配置加载问题。

再重新check一下`MailcapCommandMap`的代码，看样子应该是这段代码的缘故。

```
        synchronized(MailcapCommandMap.class) {
            if(defDB == null) {
                defDB = this.loadResource("/META-INF/mailcap.default");
            }
        }

```

看样子，应该是defDB中有`[text/*]`,所以，就没有走到读取`/META-INF/mailcap.default`文件

# 最终解决办法

其实也很简单，将`mailcap.default`重命名为`mailcap`就好了。`MailcapCommandMap`不管怎么样都会读取到`mailcap`文件了。

```
        LogSupport.log("MailcapCommandMap: load JAR");
        this.loadAllResources(dbv, "META-INF/mailcap");
        LogSupport.log("MailcapCommandMap: load DEF");
        synchronized(MailcapCommandMap.class) {
            if(defDB == null) {
                defDB = this.loadResource("/META-INF/mailcap.default");
            }
        }

```

# 参考

https://developer.jboss.org/thread/176802?_sscc=t
http://www.jguru.com/faq/view.jsp?EID=237257
http://jimwayne.blogspot.jp/2013/02/no-object-dch-for-mime-type.html
