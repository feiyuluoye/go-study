## GO Fsnotify学习与使用

[TOC]

#### 说明

`fsnotify` 是 Go 的一个文件系统通知库，可以监视文件或目录的变化。基本用法如下：

1. 安装库：
   ```bash
   go get github.com/fsnotify/fsnotify
   ```

2. 创建 watcher：
   ```go
   watcher, err := fsnotify.NewWatcher()
   if err != nil {
       log.Fatal(err)
   }
   defer watcher.Close()
   ```

3. 添加要监视的文件或目录：
   ```go
   err = watcher.Add("your/file/or/directory")
   if err != nil {
       log.Fatal(err)
   }
   ```

4. 处理事件：
   ```go
   go func() {
       for {
           select {
           case event, ok := <-watcher.Events:
               if !ok {
                   return
               }
               fmt.Println("event:", event)
           case err, ok := <-watcher.Errors:
               if !ok {
                   return
               }
               fmt.Println("error:", err)
           }
       }
   }()
   ```

#### demo

要在 Gin 中使用 `fsnotify` 监控多个文件或目录，可以按照以下步骤操作：

1. **安装依赖**：
   
   ```bash
   go get github.com/gin-gonic/gin
   ```

2. **创建 Gin 应用和监视器**：
   ```go
   package main
   
   import (
       "fmt"
       "github.com/fsnotify/fsnotify"
       "github.com/gin-gonic/gin"
       "log"
       "os"
   )
   
   func main() {
       router := gin.Default()
       watcher, err := fsnotify.NewWatcher()
       if err != nil {
           log.Fatal(err)
       }
       defer watcher.Close()
   
       // 添加多个监视的文件或目录
       paths := []string{"path/to/dir1", "path/to/file1", "path/to/dir2"}
       for _, path := range paths {
           err = watcher.Add(path)
           if err != nil {
               log.Fatal(err)
           }
       }
   
       go func() {
           for {
               select {
               case event, ok := <-watcher.Events:
                   if !ok {
                       return
                   }
                   fmt.Println("event:", event)
               case err, ok := <-watcher.Errors:
                   if !ok {
                       return
                   }
                   fmt.Println("error:", err)
               }
           }
       }()
   
       // 启动 Gin 服务
       router.GET("/", func(c *gin.Context) {
           c.String(200, "Monitoring files...")
       })
   
       router.Run(":8080")
   }
   ```

这样就可以在 Gin 应用中监控多个文件和目录的变化。需要深入了解某个特定功能吗？