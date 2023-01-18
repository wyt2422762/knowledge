# logrus

## 1. logrus简介

logrus是目前Github上star数量最多的日志包，功能强大、性能高效、高度灵活，还提供了自定义插件的功能。很多优秀的开源项目，例如：docker、prometheus等都使用了logrus。logrus除了具有日志的基本功能外，还具有如下特性：
1. 支持常用的日志级别，logrus支持如下日志级别：Debug、Info、Warn、Error、Fatal和Panic。
可扩展，logrus的hook机制允许使用者通过hook的方式将日志分发到任意地方，例如：本地文件、标准输出、elasticsearch、logstash、kafka等。
2. 支持自定义日志格式，logrus内置了2种格式：JSONFormatter和TextFormatter。除此之外，logrus允许使用者通过实现Formatter接口，来自定义日志格式。
3. 结构化日志记录，logrus的Field机制可以允许使用者自定义日志字段，而不是通过冗长的消息来记录日志。
预设日志字段，logrus的Default Fields机制可以给一部分或者全部日志统一添加共同的日志字段，例如给某次HTTP请求的所有日志添加X-Request-ID字段。  
4. Fatal handlers：logrus允许注册一个或多个handler，当发生fatal级别的日志时调用。当我们的程序需要优雅关闭时，该特性会非常有用。

## 2. logrus使用