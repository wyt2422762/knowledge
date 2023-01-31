# logrus

## 1. logrus简介

logrus是目前Github上star数量最多的日志包，功能强大、性能高效、高度灵活，还提供了自定义插件的功能。很多优秀的开源项目，例如：docker、prometheus等都使用了logrus。logrus除了具有日志的基本功能外，还具有如下特性：
1. 支持常用的日志级别，logrus支持如下日志级别：Debug、Info、Warn、Error、Fatal和Panic。
2. logrus的hook机制允许使用者通过hook的方式将日志分发到任意地方，例如：本地文件、标准输出、elasticsearch、logstash、kafka等。
3. 支持自定义日志格式，logrus内置了2种格式：JSONFormatter和TextFormatter。除此之外，logrus允许使用者通过实现Formatter接口，来自定义日志格式。
4. 结构化日志记录，logrus的Field机制可以允许使用者自定义日志字段，而不是通过冗长的消息来记录日志。
预设日志字段，logrus的Default Fields机制可以给一部分或者全部日志统一添加共同的日志字段，例如给某次HTTP请求的所有日志添加X-Request-ID字段。  
5. Fatal handlers：logrus允许注册一个或多个handler，当发生fatal级别的日志时调用。当我们的程序需要优雅关闭时，该特性会非常有用。

## 2. logrus使用

### 2.1 简单使用

```GO
package main

import (
    "io"
    "os"
    "runtime"
    "strings"

    log "github.com/sirupsen/logrus"
)

func main(){
    //设置日志输出为json格式
    log.SetFormatter(&log.JSONFormatter{
        // 设置日期格式
        TimestampFormat: "2006-01-02 15:04:05.000",
    })
    // 设置日志输出为字符串模式
    // log.SetFormatter(&log.TextFormatter{})

    // 设置日志输出级别
    // log.SetLevel(log.ErrorLevel)

    //设置日志输出流(控制台输出)
    log.SetOutput(os.Stdout)

    //获取当前文件的路径
    _, filename, _, _ := runtime.Caller(0)

    // 日志文件路径
    logFilePath := filename[:strings.LastIndex(filename, "/")] + "/logs/test.log"

    log.WithField("logFilePath", logFilePath).Info("logFilePath")

    file, err := os.OpenFile(logFilePath, os.O_APPEND|os.O_CREATE, 0666)

    if err != nil {
        log.WithField("err", err).Error("fail to create or open log file")
        return
    }

    //控制台和文件复合输出流
    fileAndStdoutStream := io.MultiWriter(os.Stdout, file)

    //设置日志输出流(同时输出控制台和文件)
    log.SetOutput(fileAndStdoutStream)

    log.WithFields(log.Fields{
        "wyt": "test",
    }).Error("something is wrong")
}
```

上述代码将输出日志

```json
{"level":"info","logFilePath":"D:/code/go-study/logrusDemo/logs/test.log","msg":"logFilePath","time":"2023-01-31 16:12:44.935"}
{"level":"error","msg":"something is wrong","time":"2023-01-31 16:12:44.939","wyt":"test"}
```

在实际项目中，往往只需要一个全局的Logger

```Go
package main

import (
    "io"
    "os"
    "runtime"
    "strings"

    log "github.com/sirupsen/logrus"
)

// 全局Logger
var Logger = log.StandardLogger()

func main(){
    // 设置全局Logger的日志格式
    Logger.SetFormatter(&log.JSONFormatter{
        // 设置日期格式
        TimestampFormat: "2006-01-02 15:04:05.000",
    })

    // 设置全局Logger的日志级别
    Logger.SetLevel(log.InfoLevel)

    //设置日志输出流(控制台输出)
    Logger.SetOutput(os.Stdout)

    //获取当前文件的路径
    _, filename, _, _ := runtime.Caller(0)

    // 日志文件路径
    logFilePath := filename[:strings.LastIndex(filename, "/")] + "/logs/test.log"

    Logger.WithField("logFilePath", logFilePath).Info("logFilePath")

    file, err := os.OpenFile(logFilePath, os.O_APPEND|os.O_CREATE, 0666)

    if err != nil {
        Logger.WithField("err", err).Error("fail to create or open log file")
        return
    }

    //控制台和文件复合输出流
    fileAndStdoutStream := io.MultiWriter(os.Stdout, file)

    //设置全局Logger日志输出流(同时输出控制台和文件)
    Logger.SetOutput(fileAndStdoutStream)

    Logger.WithFields(log.Fields{
        "wyt": "test",
    }).Error("something is wrong")
}
```

### 2.2 Hook

logrus最令人心动的功能就是其可扩展的HOOK机制了，通过在初始化时为logrus添加hook，logrus可以实现各种扩展功能。

