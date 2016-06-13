layout: post
title: 任意文件下载漏洞的简单防护措施
published: true
comments: true
sharing: true
footer: true
categories: [安全]
tags: [java,security,download,vulnerability]
date: 2016-06-13 13:17:07
keywords: Java，安全漏洞，任意文件下载
---

![security](/images/blog/download-vulnerability/security.jpg)

我们在开发Web应用时，经常会提供文件下载的功能。工程师们一般会考虑遵循“单一原则”，会开发一个将请求中的`file`或`filePath`作为参数，来下载指定的文件。这样开发一个下载的功能，就能支持所有的下载需求了。

比如，输入这样的URL

> http://tonydeng.github.io/download?file=123.txt

就能够下载`123.txt`这个文件了。

这样的确很方便，但是，大家有没有想过，这样的功能可能会出现什么样的安全隐患或者漏洞呢？

<!-- more -->

来，我们先看看例子：

下面是一段提供文件下载的`spring mvc`的java代码，使用`filePath`来指定要下载的文件。

```java
@Controller
@RequestMapping("/file/*")
public class FileDownloadController {

    private final Logger log = LoggerFactory.getLogger(FileDownloadController.class);

    @RequestMapping("download.do")
    public void fileDownload(HttpServletResponse response,
                             HttpServletRequest request,
                             @RequestParam("filePath") String filePath) throws Exception {

        File file = new File(Constants.TMP_PATH + filePath);
        if (null == file || !file.exists()) {
            return;
        }
        OutputStream toClient = null;
        try {
            // 清空response
            response.reset();
            // 设置response的Header
            response.addHeader("Content-Disposition", "attachment;filename=" + new String(filePath.getBytes("utf-8")));
            response.addHeader("Content-Length", "" + file.length());
            response.setContentType("application/octet-stream; charset=utf-8");
            toClient = new BufferedOutputStream(response.getOutputStream());
            toClient.write(FileUtil.getByteForFile(file));
            toClient.flush();

        } catch (IOException ex) {
            log.error("file download error", ex);
        } finally {
            if (null != toClient) {
                toClient.close();
            }
            file.delete();
        }
    }

}
```

我们做个测试，在下载的目录下添加一个`123.txt`的文件。

```bash
echo "123abc一二三" > 123.txt
```

测试`123.txt`是否可以下载。

```
http http://localhost:8080/file/download.do\?filePath\=123.txt
HTTP/1.1 200 OK
Content-Disposition: attachment;filename=123.txt
Content-Length: 16
Content-Type: application/octet-stream; charset=utf-8
Server: Jetty(9.3.8.v20160314)

123abc一二三

```

看起来上面的代码和测试结果，貌似没有什么问题，也能够方便的提供文件下载服务（只需要将要下载的文件保存在`Constants.TMP_PATH`这个常量中指定的目录下就可以了）。

好，貌似下载的工作就完成了，我们可以考虑做别的功能了。

稍等，既然我们这边blog要聊*任意文件下载漏洞*，那这个漏洞到底是什么呢？我们不是已经指定了文件下载的目录了吗？

那我们在继续做做做测试，在指定的下载目录的上一级来创建一个`456.txt`文件。

```bash
echo "456def四五六" > ../456.txt
```

测试`456.txt`是否可以下载。

```
http://localhost:8080/file/download.do\?filePath\=../456.txt
HTTP/1.1 200 OK
Content-Disposition: attachment;filename=../456.txt
Content-Length: 16
Content-Type: application/octet-stream; charset=utf-8
Server: Jetty(9.3.8.v20160314)

456def四五六

```

啊！！！居然能够下载！！！

其实，还有更恐怖的事情。


```
# 获取系统用户信息
http://localhost:8080/file/download.do\?filePath\=../../../../../../../../etc/passwd

# 脱裤
http://localhost:8080/file/download.do\?filePath\=../../../../../../../../data/mysql/data/mysql.dat

......
```

情绪稳定之后，我们肯定要问一问，这个漏洞出现在哪儿呢？

我们来看看这两行代码。

```java
//通过输入的filePath参数+加上预设的下载目录，获取最终下载地址
File file = new File(Constants.TMP_PATH + filePath);
 
//读取文件并写入到Response
toClient.write(FileUtil.getByteForFile(file));
```

专门把这两行提出来，大家应该能够理解这个漏洞出现在哪儿了吧。


那我们可以通过什么样的方式来解决这个漏洞呢？

1. 是否可以通过`HTTP Request`中的`Referrer`来做判断？ **貌似会误杀。**
2. 是否可以指定的文件名来做判断？ **这样太麻烦了，就不灵活了。**
3. 是否可以通过操作系统和Web容器的文件读写权限来限制？ **研究文件权限，发现根本不可行。**
4. ......

其实，我们可以通过一个简单粗暴的方式就能解决这个安全漏洞。

```java
if (null == file || !file.exists() 
                || !file.getCanonicalFile().getParent().equals(new File(Constants.TMP_PATH).getCanonicalPath())) {
	return;
}
```

原理就不解释了，看看[Java的File API文档](http://docs.oracle.com/javase/8/docs/api/java/io/File.html#getCanonicalPath--)吧。