# 概念

- gin支持加载html模板, 然后根据模板参数进行配置并返回相应的数据
- 本质上就是[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)替换，可以使用 LoadHTMLGlob() 或者 LoadHTMLFiles() 方法加载模板文件
- 也可以使用自定义模板渲染器
- gin默认只允许使用一个模板文件，如果要使用多个，需要自定义html渲染



# 示例

```go
func main() {
  router := gin.Default()
  //加载templates目录下的所有文件
  router.LoadHTMLGlob("templates/*")
  //加载多个子目录下的文件，如果在不同目录下有同名文件，也可以使用
  //router.LoadHTMLGlob("templates/**/*")
  //加载templates目录下的指定文件
  //router.LoadHTMLFiles("templates/template1.html", "templates/template2.html")
  //使用templates/index.tmpl进行渲染
  router.GET("/index", func(c *gin.Context) {
    c.HTML(http.StatusOK, "index.tmpl", gin.H{
      "title": "Main website",
    })
  })
  router.Run(":8080")
}
```



# 自定义模板渲染

```go
import "html/template"

func main() {
  /*
  file1和file2是两个渲染文件
  ParseFiles函数的参数可以是多个（文件路径+文件名），如果文件名都一样，就只有最后一个会生效
  */
  router := gin.Default()
  html := template.Must(template.ParseFiles("file1", "file2"))
  //把html里的文件设为模板
  router.SetHTMLTemplate(html)
  router.Run(":8080")
}
```



# 自定义分隔符

```go
  r := gin.Default()
  //Delims函数会设置模板的左右界
  r.Delims("{[{", "}]}")
  r.LoadHTMLGlob("/path/to/templates")
```



# 多模板

