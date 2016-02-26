## 安装开发环境
	1、下载mongodb，安装到D:\mongodb
	2、D:\mongodb下创建目录data
	3、cmd中执行D:\mongodb\bin\mongod.exe --dbpath D:\mongodb\data
	4、访问http://localhost:27017/ ，显示It looks like you are trying to access MongoDB over HTTP on the native driver port.
	5、如果你是windows，需自行安装ruby；mac自带ruby，不需要安装
	7、需安装一堆：npm install -g gulp-ruby-sass; gem install sass; gem install bootstrap-sass; gem install font-awesone-sass; gem install compass; 
	
## 源代码
	1、git上http://git.firstshare.cn/， 域账号登陆，找开发者将你加进项目里
	2、clone代码
	3、进入项目下执行npm install
	4、进入项目下执行npm install -g gulp
	4、进入项目下执行npm install -g webpack

## 本地开发
	1、启动db D:\mongodb\bin\mongod.exe --dbpath D:\mongodb\data
	2、启动监听 gulp
	3、访问http://localhost:13000/

## 部署
	1、手动访问http://fe.firstshare.cn/api/deploy?key=fedoc 触发部署
	
## 启动服务
	进入/opt/www/fe.firstshare.cn目录，执行pm2 start pm2/pro.json 
