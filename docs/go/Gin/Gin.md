# Gin

## 1. Gin继承swagger

### 1.1 什么是swagger
Swagger 是一个 API 生成工具，可以生成文档。 Swagger 是通过编写 yaml 和 json 来实现文档化。并且可以进行测试等工作。通过swagger可以方便的生成接口文档，方便前端进行查看和测试。

### 1.2 安装swagger
安装swagger

```Go
go get -u github.com/swaggo/swag/cmd/swag
```

Go 1.17之后下载可执行文件推荐使用 go install

```Go
go install github.com/swaggo/swag/cmd/swag@latest
```

等待安装完成，在我们的终端中执行 swag init，目录为根目录，于 main.go 同目录。

```Go
swag init
```

执行完成后，会在根目录下新建一个 docs 文件夹。

> docs  
 |  
 |-docs.go  
 |-swagger.json  
 |-swagger.yaml  

下载gin-swagger

```Go
go get -u github.com/swaggo/gin-swagger
go get -u github.com/swaggo/files
```

swagger路由

```Go
package router

import (
    "github.com/gin-gonic/gin"
    "github.com/swaggo/gin-swagger"
    "github.com/swaggo/files"
    docs "github.com/wyt/GinStudy/docs"
)

// swagger-router
func swagRouter(r *gin.Engine) {
    //基础url
    docs.SwaggerInfo.BasePath = ""
    // url := ginSwagger.URL("http://localhost:8080/swagger/doc.json")
    r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
}
```

在API的方法上添加注释

```Go
package router

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/wyt/GinStudy/base"
	"github.com/wyt/GinStudy/middlewares"
)

// test-router
func testRouter(r *gin.Engine) {
	//最简单的请求路由
	// r.GET("/test", test)

	testGroup := r.Group("/test", middlewares.WriteLog())
	{
		testGroup.GET("/1", test)
		testGroup.GET("/2/:id", test2)
	}


}

// 测试 gin-swagger
// @Summary 测试 gin-swagger
// @Schemes
// @Description 测试 gin-swagger
// @Tags 测试 gin-swagger
// @Accept json
// @Produce json
// @param id query string false "id"
// @Success 200 {object} base.Resp
// @Router /test/1 [get]
func test(c *gin.Context) {
    fmt.Printf("收到请求，请求地址%s\n", c.FullPath())
    id := c.Query("id")
    c.JSON(http.StatusOK, base.Resp{
        Code:    http.StatusOK,
        Msg:     "从服务端返回的数据：" + id,
        Success: true,
    })
}

// 测试 gin-swagger
// @Summary 测试 gin-swagger
// @Schemes
// @Description 测试 gin-swagger
// @Tags 测试 gin-swagger
// @Accept json
// @Produce json
// @param id path string true "id"
// @Success 200 {object} base.Resp
// @Router /test/2/{id} [get]
func test2(c *gin.Context) {
    fmt.Printf("收到请求，请求地址%s\n", c.FullPath())
    id := c.Param("id")
    c.JSON(http.StatusOK, base.Resp{
        Code:    http.StatusOK,
        Msg:     "从服务端返回的数据：" + id,
        Success: true,
    })
}
```

* @Summary 是对该接口的一个描述
* @Id 是一个全局标识符，所有的接口文档中 Id 不能重复
* @Tags 是对接口的标注，同一个 tag 为一组，这样方便我们整理接口
* @Version 表明该接口的版本
* @Accept 表示该该请求的请求类型
* @Param 表示参数 分别有以下参数 参数名词 参数类型 数据类型 是否必须 注释 属性(可选参数),参数之间用空格隔开。
* @Success 表示请求成功后返回，它有以下参数 请求返回状态码，参数类型，数据类型，注释
* @Failure 请求失败后返回，参数同上
* @Router 该函数定义了请求路由并且包含路由的请求方式。

具体参数类型，数据类型等可以查看官方文档[swaggo](https://swaggo.github.io/swaggo.io/)

返回结果为结构体的话，可以在结构体的tags中增加example，这是一个swagger标签，用来给该结构体一个示例。

```Go
type Resp struct {
    Code int `json:"code" example:"200"`
    Msg string `json:"msg" example:"成功"`
    Success bool `json:"success" example:"false"`
    Data interface{} `json:"data" `
}
```

当我们完成了所有的代码注释时，在控制台中重新执行 swag init，它会根据我们的注释生成 docs.go 及其对应的 json 和 yaml 文件。

启动我们的项目，访问 xxx/swagger/index.html 就可以查看我们的文档
