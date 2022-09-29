# Go使用cobra构建命令行工具


# Go使用cobra构建命令行工具

## 安装CLI

```bash
go install github.com/spf13/cobra-cli@latest
```

## 创建项目并引入依赖

```bash
# 创建go项目
mkdir my_command
cd my_command
go mod init my_command

# 使用cobra-cli初始化项目
cobra-cli init
```

## 测试

```bash
go build main.go
main root
```

## 开发

### 命令

```bash
# 添加命令
cobra-cli add version
```

添加后在`cmd`下生成一个新的文件 version.go

文件内容略微修改后如下

```go
/*
Copyright © 2022 NAME HERE <EMAIL ADDRESS>

*/
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
)

// versionCmd represents the version command
var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "一个简短介绍",
	Long: `一个长介绍`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("version命令操作")
	},
}

func init() {
	rootCmd.AddCommand(versionCmd)

	// Here you will define your flags and configuration settings.

	// Cobra supports Persistent Flags which will work for this command
	// and all subcommands, e.g.:
	// versionCmd.PersistentFlags().String("foo", "", "A help for foo")

	// Cobra supports local flags which will only run when this command
	// is called directly, e.g.:
	// versionCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}

```

执行命令

![截屏2022-09-29 21.52.28](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2022-09-29%2021.52.28.png)

#### 接受参数

> 存储在 args中

```go
...
var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "一个简短介绍",
	Long: `一个长介绍`,
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("version命令操作")
		fmt.Println(args)
	},
}
...
```

![截屏2022-09-29 21.57.33](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2022-09-29%2021.57.33.png)

### flag

#### 局部flag

1. 定义变量

   ```go
   /*
   	定义局部变量
   */
   var all string
   ```

2. 在对应文件的init方法中添加flag

   ```go
   versionCmd.Flags().StringVarP(&all, "all", "a", "这是默认值", "这是一条介绍")
   ```

3. 使用 在run方法中

   ```
   fmt.Println("a参数："+all)
   ```

4. 完整代码

   ```go
   /*
   Copyright © 2022 NAME HERE <EMAIL ADDRESS>
   
   */
   package cmd
   
   import (
   	"fmt"
   
   	"github.com/spf13/cobra"
   )
   
   /*
   	定义局部变量
   */
   var all string
   
   // versionCmd represents the version command
   var versionCmd = &cobra.Command{
   	Use:   "version",
   	Short: "一个简短介绍",
   	Long: `一个长介绍`,
   	Run: func(cmd *cobra.Command, args []string) {
   		fmt.Println("version命令操作")
   		fmt.Println(args)
   		fmt.Println("a参数："+all)
   	},
   }
   
   func init() {
   	rootCmd.AddCommand(versionCmd)
   	versionCmd.Flags().StringVarP(&all, "all", "a", "这是默认值", "这是一条介绍")
     //是否必填 默认可以不填
   	versionCmd.MarkFlagRequired("all")
   }
   
   ```

5. 演示

   ![截屏2022-09-29 22.09.40](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2022-09-29%2022.09.40.png)

#### 全局flag

在root中编写，代码与局部flag相同

### 嵌套命令

```bash
cobra-cli add -p "父命令Cmd"
# 举例
cobra-cli add show -p "versionCmd"
```

#### 使用

![截屏2022-09-29 22.17.21](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2022-09-29%2022.17.21.png)

#### 代码解释

![截屏2022-09-29 22.17.55](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2022-09-29%2022.17.55.png)

### 获取命令行参数

常用

```
MinimumNArgs(int) 当参数数目低于配置的最小参数个数时报错  
MaximumNArgs(int) 当参数数目大于配置的最大参数个数时报错  
ExactArgs(int)    如果参数数目不是配置的参数个数时报错  
NoArgs            没有参数则报错  
```

使用

```go
...
var versionCmd = &cobra.Command{
	Use:   "version",
	Short: "一个简短介绍",
	Long: `一个长介绍`,
	Args: cobra.ExactArgs(1),
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("version命令操作")
		fmt.Println(args)
		fmt.Println("a参数："+all)
	},
}
...
```

测试

![截屏2022-09-29 22.20.19](https://images-jsh.oss-cn-beijing.aliyuncs.com/img/%E6%88%AA%E5%B1%8F2022-09-29%2022.20.19.png)
