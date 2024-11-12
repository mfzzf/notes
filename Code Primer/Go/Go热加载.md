1. 

2. **安装`fresh`插件**：

   go install github.com/gravityblast/fresh@latest

3. **将`GOPATH/bin`添加到`PATH`环境变量**： 如果你使用的是默认的`GOPATH`，可以运行以下命令：

   export PATH=$PATH:$(go env GOPATH)/bin

4. **验证`fresh`是否安装成功**： 运行以下命令来验证`fresh`是否安装成功：

   fresh -v

5. **运行`fresh`**： 在项目根目录下运行以下命令：

   fresh