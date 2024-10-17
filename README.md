[合集 \- Go(1\)](https://github.com)1\.web服务器静态资源下载10\-17收起
#### 1\. 使用 Beego 实现静态文件下载


Beego 是一个强大的 Go Web 框架，提供了处理静态文件的功能。通过简单的配置，我们可以将本地文件夹作为静态资源目录，并为用户提供下载链接。


##### 1\.1 配置静态文件路径


首先，在 `main.go` 中，我们使用 `SetStaticPath` 将本地的 `staticfiles` 目录映射为可以通过 URL 访问的静态资源路径。




```
package main

import (
    "github.com/beego/beego/v2/server/web"
)

func main() {
    // 设置静态资源路径，映射 /staticfiles 到本地 ./staticfiles 文件夹
    web.SetStaticPath("/staticfiles", "./staticfiles")
    web.Run()
}
```


通过这段代码，我们将本地的 `./staticfiles` 目录映射到 `http://localhost:8080/staticfiles`，用户可以通过该 URL 直接访问文件。


#### 2\. 文件目录展示与下载功能


接下来，为了让用户能够方便地浏览文件目录并下载文件，我们需要实现一个控制器来展示指定目录下的文件列表，并生成对应的下载链接。


##### 2\.1 实现控制器


在 Beego 中，控制器负责处理路由请求。我们创建一个 `FileController`，其中定义了 `Get` 方法来读取指定目录，并将文件列表传递给模板。




```
package controllers

import (
    "os"
    "github.com/beego/beego/v2/server/web"
)

type FileController struct {
    web.Controller
}

// @router /getfiles [get]
func (c *FileController) Get() {
    // 要展示的目录路径
    dirPath := "./staticfiles"

    // 读取目录内容
    files, err := os.ReadDir(dirPath)
    if err != nil {
        c.Data["error"] = "无法读取目录: " + err.Error()
        c.TplName = "error.tpl"
        return
    }

    // 将文件列表传递给模板
    c.Data["files"] = files
    c.Data["directory"] = dirPath
    c.TplName = "directory.tpl"
}
```


在上面的代码中，`os.ReadDir` 函数用于读取 `staticfiles` 目录下的所有文件和文件夹。若发生错误，则渲染 `error.tpl` 模板并显示错误信息。否则，将文件列表传递给 `directory.tpl` 模板进行展示。




```
ns := beego.NewNamespace("/v1",
    beego.NSNamespace("/file",
        beego.NSInclude(&controllers.FileController{})),
)
beego.AddNamespace(ns)
```


将这个Contorller注册到router中


##### 2\.2 模板文件展示目录


为了展示文件列表并提供下载功能，我们创建一个简单的 HTML 模板 `views/directory.tpl`，将文件名展示给用户，并为每个文件生成对应的下载链接。




```
DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文件目录title>
head>
<body>
    <h1>下载文件h1>
    <ul>
        {{range .files}}
        {{if not .IsDir}}
        <li>
            <a href="/staticfiles/{{.Name}}" download="{{.Name}}">
                {{.Name}}
            a>
        li>
        {{end}}
        {{end}}
    ul>
body>
html>
```


在这个模板中，使用 Go 模板语法遍历从控制器传递来的 `files` 列表。对于每个文件，生成一个  标签，并使用 `download` 属性提供文件下载。


#### 3\. 错误处理页面


如果在读取目录时发生错误，我们会渲染一个错误页面 `views/error.tpl`。该页面展示错误信息，并提示用户返回或重试。




```
DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>错误页面title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 50px;
        }
        .error-container {
            max-width: 600px;
            margin: auto;
            padding: 20px;
            border: 1px solid #f5c6cb;
            background-color: #f8d7da;
            color: #721c24;
        }
        h1 {
            color: #721c24;
        }
    style>
head>
<body>
    <div class="error-container">
        <h1>发生错误h1>
        <p>{{.error}}p>
        <p>请返回并重试。p>
    div>
body>
html>
```


该模板通过 `{{.error}}` 渲染从控制器传递的错误信息，并通过简单的样式使其更加易于理解。


#### 4\. 使用 Docker 映射静态文件夹


为了使文件夹的管理更加灵活，并且在容器化应用中实现静态文件的持久化存储，我们可以通过 Docker 将本地文件夹映射到容器内部。


##### 4\.1 Docker 映射文件夹


在 `docker-compose.yml` 中，我们通过 `volumes` 选项将主机上的 `staticfiles` 文件夹映射到容器中的 `/app/staticfiles` 目录。




```
version: '3'
services:
  web:
    image: your-beego-image
    ports:
      - "8080:8080"
    volumes:
      - /d/commanddemo/staticfiles:/app/staticfiles
```


在这里，`/d/commanddemo/staticfiles` 是主机上的文件夹路径，`/app/staticfiles` 是容器内部的路径。通过这种方式，主机和容器中的文件可以保持同步，任何对文件的更新都会立即反映在容器内。


#### 5\. 运行 Beego 项目


完成上述步骤后，您可以运行 Beego 项目。访问 `http://localhost:8080/getfiles`，您将看到目录中的文件列表，并可以直接下载这些文件。


![](https://images.cnblogs.com/cnblogs_com/chenyishi/1348350/o_240408130234_wx.png) 本博客参考[飞数机场](https://ze16.com)。转载请注明出处！
