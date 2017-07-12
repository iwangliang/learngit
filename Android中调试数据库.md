# Android 中调试数据库
先上 gayhub 地址
> https://github.com/amitshekhariitbhu/Android-Debug-Database

## 使用
1. 添加依赖

		debugCompile 'com.amitshekhar.android:debug-db:1.0.1'

2. 查看

	1. 在日志中找到下面的代码，将地址复制到浏览器中打开

			D/DebugDB: Open http://XXX.XXX.X.XXX:8080 in your browser

	2. 在代码中直接打印出地址

			DebugDB.getAddressLog()

	3. 找到手机ip 

			http://{ip}:8080

3. 更多使用查看 gayhub


## 注意
1. 电脑和手机一定要在同一个局域网，保证电脑 ping 手机ip能 ping通
2. 笔记本直接连接wifi即可，如果台式，要保证在同一网段,能ping 通，才能打开上面的网址 