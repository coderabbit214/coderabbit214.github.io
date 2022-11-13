# gRPC


# gRPC

## 环境准备

```bash
brew install grpc
brew install protobuf
brew install protoc-gen-go
brew install protoc-gen-go-grpc
```

## HelloWord

1. 创建文件`./pb/hello_grpc.proto`

   ```protobuf
   syntax = "proto3";

   package hello_grpc;

   option go_package = "./;hello_grpc";

   message Req {
     string message = 1;
   }

   message Res {
     string message = 1;
   }

   service HelloGRPC {
     rpc SayHi(Req) returns (Res);
   }
   ```

2. 生成文件

   ```bash
   protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./pb/hello_grpc.proto
   ```

3. 创建服务端文件`./server/main.go`

   ```go
   package main

   import (
   	"context"
   	"fmt"
   	"google.golang.org/grpc"
   	"net"
   	hello_grpc2 "test/grpc/01hello_word/pb"
   )

   //1.取出server
   //2.挂载方法
   //3.注册服务
   //4.创建监听

   // 1取出server
   type server struct {
   	hello_grpc2.UnimplementedHelloGRPCServer
   }

   // SayHi 2挂载方法
   func (s *server) SayHi(ctx context.Context, req *hello_grpc2.Req) (res *hello_grpc2.Res, err error) {
   	fmt.Println(req.GetMessage())
   	return &hello_grpc2.Res{Message: "我是从服务端返回的信息"}, nil
   }

   func main() {
   	// 3注册服务
   	listen, err := net.Listen("tcp", ":8888")
   	if err != nil {
   		fmt.Println("net.Listen err:", err)
   		return
   	}
   	defer listen.Close()

   	s := grpc.NewServer()

   	hello_grpc2.RegisterHelloGRPCServer(s, &server{})
   	// 4创建监听
   	s.Serve(listen)
   }

   ```

4. 创建客户端文件`./client/main.go`

   ```go
   package main

   import (
   	"context"
   	"fmt"
   	"google.golang.org/grpc"
   	hello_grpc2 "test/grpc/01hello_word/pb"
   )

   //1.创建链接
   //2.new client
   //3.调用client
   //4.获取返回值

   func main() {
   	dial, err := grpc.Dial("127.0.0.1:8888", grpc.WithInsecure())
   	if err != nil {
   		fmt.Println("grpc.Dial err:", err)
   	}
   	defer dial.Close()

   	client := hello_grpc2.NewHelloGRPCClient(dial)

   	hi, err := client.SayHi(context.Background(), &hello_grpc2.Req{Message: "我从客户端来"})
   	if err != nil {
   		fmt.Println("client.SayHi err:", err)
   	}
   	fmt.Println(hi.GetMessage())
   }
   ```

5. 测试

## proto文件详解

### 目录格式

![截屏2022-11-13 18.14.10](https://images-jsh.oss-cn-beijing.aliyuncs.com/work/2022/11/13/20221113-181414.png)

### 基本格式

```protobuf
syntax = "proto3"; //告诉编辑器 用 proto3 解读

package person; //包名

// go_package= "包路径(从mod下开始写);别名"
option go_package = "test/grpc/02proto/pb/person;person";

//引入
import "home/home.proto";


//类似结构体 信息传递的媒介
message Person{
}

//定义服务 请求返回定义
service SearchService {
  //即时响应
  rpc Search(Person) returns (Person);
}
```

### message

```protobuf

//类似结构体
message Person{
  // 类型 名称 = 唯一标识;
  string name = 1;
  int32 age = 2;
  //  bool sex = 3;
  //数组 切片
  repeated string test = 4;
  //map
  map<string, string> test_map = 5;

  //枚举 必须要有0
  enum SEX {
    //如果有的值需要一样则需要设置
    option allow_alias = true;
    MAN = 0;
    WOMAN = 1;
    GIRL = 1;
    OTHER = 2;
  }
  SEX sex = 3;

//不同类型相同名称使用
  oneof TestOneOf{
    string one = 6;
    string two = 7;
    string three = 8;
  }

  home.Home i_home = 9;

  //设置保留关键字 不允许定义
  //  reserved "test_map","test";
  //  reserved 2;
}

//message 嵌套
message Home{
  repeated Person persons = 1;
  //定义
  message V {
    string name = 1;
  }
  //声明
  V v = 2;
}
```

#### oneof 使用说明

1. proto

   ```protobuf
   //省略前边

   message Person {
     oneof TestOneOf{
       string one = 6;
       string two = 7;
       string three = 8;
     }
   }

   service SearchService {
     rpc Search(Person) returns (Person);
   }
   ```

2. server

   ```go
   func (s *spaceServer) Search(context context.Context, req *space.SpaceReq) (res *space.SpaceRes, err error) {
   	name := req.GetName()
   	fmt.Println(name)
   	oneOf := req.TestOneOf
   	switch v := oneOf.(type) {
   	case *space.SpaceReq_One:
   		fmt.Println(v.One)
   	case *space.SpaceReq_Two:
   		fmt.Println(v.Two)
   	case *space.SpaceReq_Three:
   		fmt.Println(v.Three)
   	}
   	res = &space.SpaceRes{
   		Name: "服务端",
   	}
   	return res, nil
   }
   ```

3. client

   ```go
   	dial, err := grpc.Dial("127.0.0.1:8888", grpc.WithInsecure())
   	if err != nil {
   		fmt.Println("grpc.Dial err:", err)
   	}
   	defer dial.Close()
   	client := space.NewSearchServiceClient(dial)
   	//及时相应
   	var p = space.SpaceReq{Name: "asdf", TestOneOf: &space.SpaceReq_One{One: "asdfasdf"}}
   	hi, err := client.Search(context.Background(), &p)
   	if err != nil {
   		fmt.Println("client.SayHi err:", err)
   	}
   	fmt.Println(hi.GetName())
   ```

### import引入其他proto

1. 创建需要引入的proto`./pb/home/home.proto`

   ```protobuf
   syntax = "proto3"; //告诉编辑器 用 proto3 解读

   package home; //包名

   // go_package= "包路径(从mod下开始写);别名"
   option go_package = "test/grpc/02proto/pb/home;home";

   message Home{
     string name = 1;
   }
   ```

2. 在需要引入文件中引入

   ```protobuf
   //引入
   import "home/home.proto";

   message Person {
   //使用
     home.Home i_home = 9;
   }
   ```

3. 编译

   ```
   protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./person/person.proto
   protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./home/home.proto
   ```



### service

> 使用 space包下文件

1. proto`./pb/space/space.proto`

   ```protobuf
   syntax = "proto3";

   package space;

   option go_package = "test/grpc/02proto/pb/space;space";

   message SpaceReq{
     string name = 1;
   }

   message SpaceRes{
     string name = 1;
   }

   //定义服务
   service SearchService {
     //即时响应
     rpc Search(SpaceReq) returns (SpaceRes);
     //入参为流
     rpc SearchIn(stream SpaceReq) returns (SpaceRes);
     //相应为流
     rpc SearchOut(SpaceReq) returns (stream SpaceRes);
     //出入都为流
     rpc SearchIO(stream SpaceReq) returns (stream SpaceRes);
   }
   ```

2. service

   ```go
   package main

   import (
   	"context"
   	"fmt"
   	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
   	"google.golang.org/grpc"
   	"net"
   	"net/http"
   	"test/grpc/02proto/pb/space"
   )

   type spaceServer struct {
   	space.UnimplementedSearchServiceServer
   }

   func (s *spaceServer) Search(context context.Context, req *space.SpaceReq) (res *space.SpaceRes, err error) {
   	name := req.GetName()
   	fmt.Println(name)
   	res = &space.SpaceRes{
   		Name: "服务端",
   	}
   	return res, nil
   }
   func (s *spaceServer) SearchIn(server space.SearchService_SearchInServer) error {
   	for {
   		req, err := server.Recv()
   		//传输结束
   		if err != nil {
   			server.SendAndClose(&space.SpaceRes{Name: "完成了"})
   			break
   		}
   		fmt.Println(req)
   	}
   	return nil
   }
   func (s *spaceServer) SearchOut(req *space.SpaceReq, server space.SearchService_SearchOutServer) error {
   	name := req.Name
   	fmt.Println(name)

   	for i := 0; i < 10; i++ {
   		server.Send(&space.SpaceRes{Name: "服务端"})
   	}
   	return nil
   }
   func (s *spaceServer) SearchIO(server space.SearchService_SearchIOServer) error {
   	strChan := make(chan string, 1)

   	go func() {
   		for {
   			recv, err := server.Recv()
   			if err != nil {
   				fmt.Println(err)
   				strChan <- "结束"
   				break
   			}
   			fmt.Println(recv)
   			strChan <- recv.Name
   		}
   	}()

   	for {
   		name := <-strChan
   		if "结束" == name {
   			break
   		}
   		server.Send(&space.SpaceRes{Name: name})
   	}

   	return nil
   }

   func main() {
   	listen, err := net.Listen("tcp", ":8888")
   	if err != nil {
   		fmt.Println("net.Listen err", err)
   		return
   	}
   	s := spaceServer{}
   	server := grpc.NewServer()
   	space.RegisterSearchServiceServer(server, &s)
   	server.Serve(listen)
   }
   ```

3. client

   ```go
   package main

   import (
   	"context"
   	"fmt"
   	"google.golang.org/grpc"
   	"sync"
   	"test/grpc/02proto/pb/space"
   )

   func main() {
   	dial, err := grpc.Dial("127.0.0.1:8888", grpc.WithInsecure())
   	if err != nil {
   		fmt.Println("grpc.Dial err:", err)
   	}
   	defer dial.Close()
   	client := space.NewSearchServiceClient(dial)
   	//及时相应
   	//var p = space.SpaceReq{Name: "asdf", TestOneOf: &space.SpaceReq_One{One: "asdfasdf"}}
   	//hi, err := client.Search(context.Background(), &p)
   	//if err != nil {
   	//	fmt.Println("client.SayHi err:", err)
   	//}
   	//fmt.Println(hi.GetName())

   	//输入为流
   	//c, _ := client.SearchIn(context.Background())
   	//for i := 0; i < 10; i++ {
   	//	c.Send(&space.SpaceReq{Name: "哈哈哈"})
   	//	time.Sleep(1 * time.Second)
   	//}
   	//recv, _ := c.CloseAndRecv()
   	//fmt.Println(recv)

   	//输出为流
   	//out, _ := client.SearchOut(context.Background(), &space.SpaceReq{Name: "1324"})
   	//for {
   	//	recv, err := out.Recv()
   	//	if err != nil {
   	//		fmt.Println(err)
   	//		break
   	//	}
   	//	fmt.Println(recv)
   	//}

   	//输入输出为流
   	group := sync.WaitGroup{}
   	group.Add(1)
   	io, _ := client.SearchIO(context.Background())
   	go func() {
   		for {
   			recv, err2 := io.Recv()
   			if err2 != nil {
   				fmt.Println(err2)
   				group.Done()
   				break
   			}
   			fmt.Println("服务端返回：", recv)
   		}
   	}()

   	for i := 0; i < 10; i++ {
   		err := io.Send(&space.SpaceReq{Name: "1324"})
   		if err != nil {
   			fmt.Println(err)
   			break
   		}
   	}
   	io.CloseSend()
   	group.Wait()
   }
   ```

## grpc-gateway

### 安装插件

```bash
go install \
    github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway
```

### 代码编写

1. proto

   ```protobuf
   service SearchService {
     //即时响应
     rpc Search(SpaceReq) returns (SpaceRes){
       option(google.api.http) = {
         post:"/api/space",
         body:"*"
       };
     };
   }
   ```

2. 编译

   ```bash
   protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative --grpc-gateway_out . --grpc-gateway_opt paths=source_relative ./space/space.proto
   ```

3. server

   ```go

   func main() {

   	go registerGetaway()

   	listen, err := net.Listen("tcp", ":8888")
   	if err != nil {
   		fmt.Println("net.Listen err", err)
   		return
   	}
   	s := spaceServer{}
   	server := grpc.NewServer()
   	space.RegisterSearchServiceServer(server, &s)
   	server.Serve(listen)
   }

   //具体实现
   func registerGetaway() {
   	mux := runtime.NewServeMux()
   	conn, _ := grpc.DialContext(
   		context.Background(),
   		"127.0.0.1:8888",
   		grpc.WithBlock(),
   		grpc.WithInsecure(),
   	)
   	gwServer := &http.Server{
   		Handler: mux,
   		Addr:    ":8090",
   	}
   	space.RegisterSearchServiceHandler(context.Background(), mux, conn)
   	gwServer.ListenAndServe()
   }
   ```

4. 发送post请求测试

