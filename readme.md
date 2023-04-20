# Servlet

Servlet是Java EE的一个标准, 大部分的web服务器都支持此标准, 包括Tomcat, 就像JDBC一样, 由官方定义了一系列接口, 
而具体实现由我们编写, 最后交给web服务器(eg:Tomcat)来运行

## Servlet的作用

通过实现Servlet来进行动态网页响应, 使用Servlet, 可以实现静态网页和Java代码进行动态拼接, 它能够很好地实现动态网页的返回

Servlet并不是专用于http通讯协议, 也可用于其他, 只是http更常见

## 创建Servlet

只需要实现Servlet类即可, 并添加注解@WebServlet来进行注册

```java
@WebServlet("/test")
public class TestServlet implements Servlet {
        //...实现接口方法
}
```

## Servlet的生命周期

首先实现 Servlet 接口的方法

在其中放入打印语句 

```java
public class HelloServlet implements Servlet {

    public HelloServlet(){
        System.out.println("我是构造方法！");
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.println("我是init");
    }

    @Override
    public ServletConfig getServletConfig() {
        System.out.println("我是getServletConfig");
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.println("我是service");
    }

    @Override
    public String getServletInfo() {
        System.out.println("我是getServletInfo");
        return null;
    }

    @Override
    public void destroy() {
        System.out.println("我是destroy");
    }
}
```

访问定义的页面, 观察结果

```text
我是构造方法！
我是init
我是service

我是destroy
```

我们可以多次尝试去访问此页面, 但是init和构造方法只会执行一次, 而每次访问都会执行的是service方法

因此，一个Servlet的生命周期为：

- 首先执行构造方法完成 Servlet 初始化
- Servlet 初始化后调用 init () 方法
- Servlet 调用 service() 方法来处理客户端的请求
- Servlet 销毁前调用 destroy() 方法
- 最后, Servlet 是由 JVM 的垃圾回收器进行垃圾回收的

在Web应用程序运行时, 每当浏览器向服务器发起一个请求时, 都会创建一个线程执行一次service方法, 
来让我们处理用户的请求, 并将结果响应给用户

### ServletRequest和ServletResponse

在service方法中, 还有两个参数 : ServletRequest和ServletResponse, 
实际上, 用户发起的HTTP请求, 就被Tomcat服务器封装为了一个ServletRequest对象, 我们得到是其实是Tomcat服务器帮助我们创建的一个实现类

HTTP请求报文中的所有内容, 都可以从ServletRequest对象中获取, 同理, ServletResponse就是我们需要返回给浏览器的HTTP响应报文实体类封装

ServletRequest还有哪些内容, 我们可以获取到请求的一些信息:

```java
public class HelloServlet implements Servlet {
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        //首先将其转换为HttpServletRequest（继承自ServletRequest，一般是此接口实现）
        HttpServletRequest request = (HttpServletRequest) servletRequest;

        System.out.println(request.getProtocol());  //获取协议版本
        System.out.println(request.getRemoteAddr());  //获取访问者的IP地址
        System.out.println(request.getMethod());   //获取请求方法
        //获取头部信息
        Enumeration<String> enumeration = request.getHeaderNames();
        while (enumeration.hasMoreElements()) {
            String name = enumeration.nextElement();
            System.out.println(name + ": " + request.getHeader(name));
        }
    }
}
```

整个HTTP请求报文中的所有内容, 都可以通过HttpServletRequest对象来获取

#### ServletResponse

这个是服务端的响应内容, 我们可以在这里填写我们想要发送给浏览器显示的内容：

```text
//转换为HttpServletResponse（同上）
HttpServletResponse response = (HttpServletResponse) servletResponse;
//设定内容类型以及编码格式（普通HTML文本使用text/html，之后会讲解文件传输）
response.setHeader("Content-type", "text/html;charset=UTF-8");
//获取Writer直接写入内容
response.getWriter().write("我是响应内容！");
//所有内容写入完成之后，再发送给浏览器
```

在浏览器中打开此界面, 就能够收到服务器发来的响应内容了, 其中响应头部分是由Tomcat生成的一个默认header

## 使用HttpServlet

HttpServlet继承自GenericServlet, 它根据HTTP协议的规则, 完善了service方法

GenericServlet是Servlet的一个直接实现抽象类

其实只需要继承HttpServlet来编写Servlet就可以了, 并且它已经提前实现了一些操作

```java
@Log
@WebServlet("/test")
public class TestServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        resp.getWriter().write("<h1>new way</h1>");
    }
}
```

现在只需要重写对应的请求方式, 就可以快速完成Servlet的编写

## @WebServlet注解

可以直接使用此注解来快速注册一个Servlet

name属性就是Servlet名称, 而urlPatterns和value实际上是同样功能, 就是代表当前Servlet的访问路径

它不仅仅可以是一个固定值, 还可以进行通配符匹配:

```text
@WebServlet("/test/*")
```

上面路径表示, 所有匹配[/test/任意]的路径名称, 都可以访问此Servlet

也可以进行某个扩展名称的匹配:

```text
@WebServlet("*.js")
```

这样的话, 获取任何以js结尾的文件, 都会由我们自己定义的Servlet处理

还可以为一个Servlet配置多个访问路径:

```text
@WebServlet({"/test1", "/test2"})
```

### 接下来loadOnStartup属性

此属性决定了是否在Tomcat启动时就加载此Servlet, 

默认情况下, Servlet只有在被访问时才会加载, 它的默认值为-1, 表示不在启动时加载

可以将其修改为大于等于0的数, 来开启启动时加载. 并且数字的大小决定了此Servlet的启动优先级高低
