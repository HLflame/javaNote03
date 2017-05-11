# cookie和Session笔记

标签: javaWeb

----

**Content**

-	[javaWeb中cookie和Session笔记](#cookie和Session笔记)
-	[会话技术](#会话技术)
-	[cookie](#cookie)
-	[Session](#Session)

----
## 会话技术
-	什么是会话:用户打开一个浏览器访问页面,访问网站的很多页面,访问完成后将浏览器关闭的过程称为是一次会话
-	常见的会话技术:
	-	cookie
	-	Session
-	为什么使用会话技术?
	-	私有的数据,购物信息数据保存在会话技术中
## cookie
-	cookie客户端技术
-	将数据保存到客户端浏览器
-	Cookie技术的使用
	-	向浏览器保存数据:
		-	HttpServletResponse有一个方法:
			-	void addCookie(Cookie cookie);
	-	获得浏览器带过来的Cookie:
## Session
-	Session服务器端技术
-	将数据保存到服务器端