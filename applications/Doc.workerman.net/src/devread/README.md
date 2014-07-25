# 开发前必读

使用WorkerMan开发应用，你需要了解以下内容：

## 一、WorkerMan开发与普通PHP开发的不同之处

基于WorkerMan开发与普通应用程序开发除了HTTP协议相关的变量函数无法直接使用外，其它都是通用的。

### 1、应用层协议
普通PHP开发一般是基于HTTP应用层协议的WEB程序开发，可以借助与很多开源的MVC框架。由于协议是固定的HTTP协议，HTTP请求的协议解析和处理全部由nginx/apache一类的服务器软件代劳了，所以开发者无需特别关注协议的部分。而WorkerMan开发则不同，WorkerMan的目标是解决非HTTP协议的应用程序开发，所以就需要开发者关注协议的解析部分。

#### 由于主要是非HTTP协议的应用，将会影响以下变量函数或者功能有所不同：

普通的WorkerMan应用程序无法使用HTTP协议相关的变量与函数。如```$_POST $_GET $_REQUEST $_SESSION $_FILE header() setcookie()```等。

无法直接使用基于HTTP协议的MVC框架的很多功能，如HTTP路由。但是可以引用并使用这些MVC框架的提供的类库如mysql类等。

### 2、请求周期差异
在Web开发中，有请求周期的概念，即一个HTTP请求到来后，PHP会初始化环境```$_GET/$_POST```等，然后从入口文件开始，运行过程中会载入相应的文件并执行，然后输出执行的结果返回给客户端，然后关闭本次请求并清理环境。

在WorkerMan开发中，也有请求周期的概念，WorkerMan请求周期是从解析应用层协议（```dealInput```）开始，然后进入到请求处理阶段(```dealProcess```)，处理阶段通过接口(```sendToClient```)完成数据的返回。可以说WorkerMan就是不断的通过运行```dealInput/dealProcess```两个函数来处理业务请求的。没有重复的的环境初始化和环境清零步骤。

由于请求周期的差异，WorkerMan只需要从磁盘载入一次PHP文件，相同的文件在后续的请求不需要再次载入，会直接使用内存中已经编译好的缓存。这里需要注意尽量在文件头部使用```require_once```载入文件，避免重复载入相同的文件导致类的重定义等问题。

由于请求周期的差异，WorkerMan运行过程中产生的全局变量、类的静态成员不会在请求后释放。也就是说在WorkerMan中可以永久保存一些变量或者对象。例如mysql类一般是单例的，在第一次请求时遍保存在了类的静态成变量中，所以后续的所有请求都会复用这个链接，实现了真正的数据库长链接，这对于性能的提升是非常有帮助的（极大的减少了数据库链接三次握手、关闭四次握手、及数据库用户认证的网络开销）。需要注意的是在数据库链接长时间空闲时，mysql服务端可能会主动关闭链接，这就需要开发者在数据库异常时判断是否是mysql主动关闭了链接，并重连。

## 二、关于TCP与应用层协议
TCP是基于流的，也就是说客户端向服务端发送多个请求是通过TCP传输层协议分段传送到服务端的，这就导致服务端收到的请求可能是不完整的，也有可能是多个请求连在一起的。要把请求从连续的TCP流中区分开，就需要有一个应用层协议，例如HTTP协议就是一个应用层协议。应用层协议一个主要作用就是用来区分TCP流中的请求边界。这也是WorkerMan开发中一个难点。

## 三、长链接与短链接
根据你的应用是长链接应用还是短链接应用的不同，在WorkerMan开发中可能有所不同。

如果是短链接应用，一般业务流程非长简单，每次链接请求后都会断开，业务进程一般不会保存业务相关的数据，更不用维护客户端的长链接。可参考WorkerMan中的基本开发流程开发会比较简单高效。

如果是长链接应用，这类应用都有一个共同的特点，就是服务端要主动向客户端推送数据，类似聊天、游戏类的应用。这类应用需要服务端维护客户端的链接数据，要为客户端分配唯一标识从而能够区分不同的客户端，还要有向某个客户端或者全部客户端发送数据的接口，这些对于普通开发者来说比较复杂并且容易出错，所以WorkerMan在基本开发流程的基础上开发了一套长链接应用框架，即ChatDemo。如果开发者是开发长链接应用，则看诶参考Gateway\Worker开发流程。
