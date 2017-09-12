             git多账号配置

1、生成新的ssh公钥和私钥

	命令：gitbash：  “ssh-keygen -t rsa -C "chencanlingg@gmail.com” -f ~/.ssh/perfectionisshit-rsa”

2、添加私钥

	ssh-add ~/.ssh/perfectionisshit_rsa 
	//可以同时添加多个用$连接
	$ ssh-add ~/.ssh/phoenix-ccl-rsa

	如果执行ssh-add时提示"Could not open a connection to your authentication agent"，可以现执行命令：
	ssh-agent bash 然后再执行添加命令

3、修改配置文件

	touch config// (C:\Users\ccl\.ssh)此目录下如果没有config文件就会生成一个config文件
	配置如下：

	# 该文件用于配置私钥对应的服务器
	# Default github user(chencanlin163@163.com)
	Host github-chencanlin.com
	 HostName github.com
	 User git
	 IdentityFile ~/.ssh/id_rsa
	
	# second user(chencanlingg@gmail.com)
	# 建一个github别名，新建的帐号使用这个别名做克隆和更新
	Host github-pis.com
	 HostName github.com
	 User git
	 IdentityFile ~/.ssh/perfectionisshit_rsa
	 
	 # third user(chencanlin163@163.com)
	# 建一个gitlab别名，新建的帐号使用这个别名做克隆和更新
	Host gitlab-phoenix.com
	 HostName gitlab.com
	 User git
	 IdentityFile ~/.ssh/phoenix-ccl-rsa

4、测试

	参照如上配置 测试命令如下：

	ssh -T git@github-chencanlin.com
	ssh -T git@github-pis.com
    ssh -T git@gitlab-phoenix.com
	
![](https://i.imgur.com/YIi3C4A.png)

项目更新提交命令中别名也需要跟着改变
	
	如：git clone git@github.com:chencanlin/FlowLayout.git
	要变为 git clone git@github-chencanlin.com:chencanlin/Flowlayout.git
	
	git remote add pis git@github.com:PerfectionIsShit/gittest.git 变为 git remote add pis git@github-pis.com:PerfectionIsShit/gittest.git
	