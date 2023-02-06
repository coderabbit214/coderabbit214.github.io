# Go WebAssembly 实践


# Go-WebAssembly实践

> 在网页端实现一个json转struct

## Hello Word

1. 创建项目目录 

   ```
   Documents/  
   └── webassembly
       ├── assets
       └── cmd
           ├── server
           └── wasm
   ```

2. 在 webassembly初始化go项目

   ```
   go mod init teswebassembly  
   ```

3. 在`webassembly/cmd/wasm`下创建`main.go`

   ```go
   package main
   
   import (  
       "fmt"
   )
   
   func main() {  
       fmt.Println("Go Web Assembly")
   }
   ```

4. 编译

   ```bash
   cd webassembly/cmd/wasm/  
   GOOS=js GOARCH=wasm go build -o  ../../assets/json.wasm  
   ```

   生成了在浏览器中运行的二进制文件

5. 获取`wasm_exec.js`用来调用`json.wasm `

   ```bash
   cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" webassembly/assets/  
   ```

6. 在`webassembly/cmd/wasm`下创建`index.html`

   ```html
   <html>  
       <head>
           <meta charset="utf-8"/>
           <script src="wasm_exec.js"></script>
           <script>
               const go = new Go();
               WebAssembly.instantiateStreaming(fetch("json.wasm"), go.importObject).then((result) => {
                   go.run(result.instance);
               });
           </script>
       </head>
       <body></body>
   </html>  
   ```

7. 目前目录

   ```
   Documents/  
   └── webassembly
       ├── assets
       │   ├── index.html
       │   ├── json.wasm
       │   └── wasm_exec.js
       └── cmd
           ├── server
           └── wasm
               └── main.go
           └── go.mod
   ```

8. 做一个简单服务器用来运行`index.html`,编译器自带也可以

   在`webassembly/cmd/server`下创建`main.go`

   ```go
   package main
   
   import (  
       "fmt"
       "net/http"
   )
   
   func main() {  
       err := http.ListenAndServe(":8080", http.FileServer(http.Dir("../../assets")))
       if err != nil {
           fmt.Println("Failed to start server", err)
           return
       }
   }
   ```

9. 运行

   ```bash
   cd webassembly/cmd/server/  
   go run main.go  
   ```

   ![截屏2023-02-06 10.37.17](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2023/02/06/20230206-103722.png)

   在控制台中可以看到打印

## 功能实现

> Json2Go 使用的是 github.com/m-zajac/json2go

在`webassembly/cmd/wasm/main.go`中添加

1. 添加功能代码

   ```go
   import (
   	"github.com/m-zajac/json2go"
   )
   
   func Json2Go(input string) string {
   	parser := json2go.NewJSONParser("Document")
   	parser.FeedBytes([]byte(input))
   	res := parser.String()
   	return res
   }
   ```

2. 将函数暴露到`Javascript`

   ```go
   import (
   	"syscall/js"
   )
   
   func main() {
   	fmt.Println("Go Web Assembly")
   	js.Global().Set("json2Go", Json2GoWrapper())
     //避免 Error: Go program has already exited
     <-make(chan bool)
   }
   
   func Json2GoWrapper() js.Func {
   	jsonFunc := js.FuncOf(func(this js.Value, args []js.Value) any {
   		if len(args) != 1 {
   			return "Invalid no of arguments passed"
   		}
   		inputJSON := args[0].String()
   		res := Json2Go(inputJSON)
   		return res
   	})
   	return jsonFunc
   }
   ```

   `syscall/js` 编译器报错设置

    ![截屏2023-02-06 10.44.08](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2023/02/06/20230206-104427.png)

3. 编写ui

   ```html
   <html>
   <head>
       <meta charset="utf-8"/>
       <script src="wasm_exec.js"></script>
       <script>
           const go = new Go();
           WebAssembly.instantiateStreaming(fetch("json.wasm"), go.importObject).then((result) => {
               go.run(result.instance);
           });
       </script>
   </head>
   <body>
   <textarea id="jsoninput" name="jsoninput" cols="80" rows="20"></textarea>
   <input id="button" type="submit" name="button" value="pretty json" onclick="json(jsoninput.value)"/>
   <textarea id="jsonoutput" name="jsonoutput" cols="80" rows="20"></textarea>
   </body>
   <script>
       var json = function (input) {
           jsonoutput.value = json2Go(input)
       }
   </script>
   </html>
   ```

4. 测试

   ![截屏2023-02-06 10.49.09](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2023/02/06/20230206-104913.png)

## DOM操作

1. 修改`webassembly/cmd/wasm/main.go`中`Json2GoWrapper`方法

   ```go
   func Json2GoWrapper() js.Func {
   	jsonfunc := js.FuncOf(func(this js.Value, args []js.Value) any {
   		if len(args) != 1 {
   			result := map[string]any{
   				"error": "Invalid no of arguments passed",
   			}
   			return result
   		}
   		jsDoc := js.Global().Get("document")
   		if !jsDoc.Truthy() {
   			result := map[string]any{
   				"error": "Invalid no of arguments passed",
   			}
   			return result
   		}
   		jsonOuputTextArea := jsDoc.Call("getElementById", "jsonoutput")
   		if !jsonOuputTextArea.Truthy() {
   			result := map[string]any{
   				"error": "Invalid no of arguments passed",
   			}
   			return result
   		}
   		inputJSON := args[0].String()
   		res := Json2Go(inputJSON)
   		jsonOuputTextArea.Set("value", res)
   		return nil
   	})
   	return jsonfunc
   }
   ```

2. 修改`webassembly/assets/index.html`

   ```html
   <script>
       var json = function (input) {
           var result = json2Go(input)
           if (( result != null) && ('error' in result)) {
               console.log("Go return value", result)
               jsonoutput.value = ""
               alert(result.error)
           }
           // jsonoutput.value = json2Go(input)
       }
   </script>
   ```

## 错误处理

由于js无法处理`error`类型，对应表：https://pkg.go.dev/syscall/js#ValueOf

![截屏2023-02-06 10.59.14](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2023/02/06/20230206-105918.png)

所以错误处理返回`map[string]interface{}`

```go
result := map[string]any{
	"error": "error message",
}
```

## Json参数传递

1. `index.html`

   ```html
   <script>
       var json = function (input) {
           jsonoutput.value = json2Go({"json": input})
       }
   </script>
   ```

2. Go接收

   ```go
   func Json2GoWrapper() js.Func {
   	jsonFunc := js.FuncOf(func(this js.Value, args []js.Value) any {
   		if len(args) != 1 {
   			return "Invalid no of arguments passed"
   		}
   		inputJSON := args[0].Get("json").String()
   		res := Json2Go(inputJSON)
   		return res
   	})
   	return jsonFunc
   }
   ```

   
