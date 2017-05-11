#javaWeb总结笔记之request和response

标签:javaWeb

----

**Contents**

----
-	[javaWeb总结笔记-request和response](#javaWeb总结笔记之request和response)
	-	[response]()
	-	[案列一:登录成功后,完成文件的下载]()
		-	[response的概述]()
		-	[response常用的API]()
		-	[文件下载的方式]()
		-	[代码实现]()
		-	[总结]()
			-	[中文文件的下载]()
			-	[response输出响应内容的方法]()
			-	[response输出中文乱码处理]()
	-	[request]()
	-	[案列二:完成用户注册的功能]()
		-	[request的概述]()
		-	[response常用的API]()
		-	[代码实现]()
		-	[总结]()
			-	[处理request接收参数的中文乱码的问题:]()
			-	[Request作为域对象存取数据：]()
			-	[重定向和转发的区别:(redirect和forward的区别)]()


----
## response
## 案列一:登录成功后,完成文件的下载
###	response的概述
	Response:代表响应的对象.从服务器向浏览器输出内容.
### response常用的API
- 响应行	
	- void setStatus(int sc)设置状态码
- 响应头
	- void addDateHeader(String name,long date)
	- void addHeader(String naem,String  value)
	- void addIntHeader(String name, int value)
	- 针对一个key对应多个value的头信息
	- void setDateHeader(String name,long date)
	- void setHeader(String name,String value)
	- void setIntHeader(String name,int value)
	- 针对一个key对应一个value的头信息.
-	响应体
	-	ServletOutputStream getOutputStream()
	-	PrintWriter getWriter()
	-	第一个字节输出流,第二个字符输出流
###	文件下载的方式
-	一种:超链接下载.直接将文件的路径写到超链接的href中.---前提:文件类型,浏览器不支持.
-	二种:手动编写代码的方式完成文件的下载.
	-	设置两个头和一个流
		-	Content-Type		:返回给浏览器文件的MIME的类型.
		-	Content-Disposition	:以下载的形式打开文件.
		-	InputStream			:文件的输入流.
###	代码实现
-	步骤一：将之前的登录功能准备好:
-	步骤二：在文件下载列表页面上添加文件下载的链接:
-	步骤三：完成文件下载的代码的实现:

public class DownloadServlet extends HttpServlet
	 {
	
	private static final long serialVersionUID = 1L;

	
	protected void doGet(HttpServletRequest 	request,HttpServletResponse response) throws 	ServletException, IOException {
		
	// 1.接收参数
		String filename = request.getParameter("filename");
		// 2.完成文件下载:
		// 2.1设置Content-Type头
		String type = this.getServletContext().getMimeType(filename);
		response.setHeader("Content-Type", type);
		// 2.2设置Content-Disposition头
		response.setHeader("Content-Disposition", "attachment;filename="+filename);
		// 2.3设置文件的InputStream.
		String realPath = this.getServletContext().getRealPath("/download/"+filename);
		InputStream is = new FileInputStream(realPath);
		// 获得response的输出流:
		OutputStream os = response.getOutputStream();
		int len = 0;
		byte[] b = new byte[1024];
		while((len = is.read(b))!= -1){
			os.write(b, 0, len);
		}
		is.close();
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}

	}
###	总结
####	中文文件的下载
-	IE浏览器下载中文文件的时候采用的URL的编码
-	Firefox浏览器下载中文文件的时候采用的是Base64的编码.
-	乱码处理后的下载代码

	/**
 * 文件下载的Servlet
 */

public class DownloadServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// 1.接收参数
		String filename = new String(request.getParameter("filename").getBytes("ISO-8859-1"),"UTF-8");
		System.out.println(filename);
		// 2.完成文件下载:
		// 2.1设置Content-Type头
		String type = this.getServletContext().getMimeType(filename);
		response.setHeader("Content-Type", type);
		// 2.3设置文件的InputStream.
		String realPath = this.getServletContext().getRealPath("/download/"+filename);
		
		// 根据浏览器的类型处理中文文件的乱码问题:
		String agent = request.getHeader("User-Agent");
		System.out.println(agent);
		if(agent.contains("Firefox")){
			filename = base64EncodeFileName(filename);
		}else{
			filename = URLEncoder.encode(filename,"UTF-8");
		}
		
		// 2.2设置Content-Disposition头
		response.setHeader("Content-Disposition", "attachment;filename="+filename);
		
		InputStream is = new FileInputStream(realPath);
		// 获得response的输出流:
		OutputStream os = response.getOutputStream();
		int len = 0;
		byte[] b = new byte[1024];
		while((len = is.read(b))!= -1){
			os.write(b, 0, len);
		}
		is.close();
	}
	
	public static String base64EncodeFileName(String fileName) {
		BASE64Encoder base64Encoder = new BASE64Encoder();
		try {
			return "=?UTF-8?B?"
					+ new String(base64Encoder.encode(fileName
							.getBytes("UTF-8"))) + "?=";
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
			throw new RuntimeException(e);
		}
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}

}
####	response输出响应内容的方法
-	向页面响应的方法:
	-	getOutputStream();
	-	getWriter();
-	这两个方法是互斥的.
	-	 * 做出响应的时候只能使用其中的一种流响应.

####	response输出中文乱码处理
-	字节流:
	-	设置浏览器默认打开的编码:
		-	resposne.setHeader(“Content-Type”,”text/html;charset=UTF-8”);
	-	设置中文字节取出的时候编码.
		-	“中文”.getBytes(“UTF-8”);
-	字符流:
	-	设置浏览器打开的时候的编码
		-	resposne.setHeader(“Content-Type”,”text/html;charset=UTF-8”);
	-	设置response的缓冲区的编码
		-	response.setCharacterEncoding(“UTF-8”);
	-	简化的写法:response.setContentType(“text/html;charset=UTF-8”);
## request
## 案列二:完成用户注册的功能
###	request的概述
-	Request代表用户的请求

### request常用的API
-	功能一：获得客户机相关的信息
	-	获得请求的方式:
		-	String getMethod()
	-	获得请求的路径:
		-	String getRequestURI()
		-	StringBuffer getRequestURL()
	-	获得客户机相关的信息:
		-	String getRomoteAddr() ip地址信息
	-	获得工程名:
		-	String getContextPath()
-	功能二:获得从页面中提交的参数:
	-	String getParameter(String name)
	-	Map getParameterMap()
	-	Enumeration getParameterNames()
	-	String[] getParameterValues(String name)
-	功能三:作为域对象存取数据:
	-	void removeAttribute(String name)
	-	void setAttribute(String name,Object o)
	-	Object getAttribute(String name)
-	演示request获得客户机的信息


public class RequestServlet extends HttpServlet {

	private static final long serialVersionUID = 1L;
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// 获得请求方式：
		String method = request.getMethod();
		System.out.println("请求方式:"+method);
		// 获得客户机的IP地址：
		String ip = request.getRemoteAddr();
		System.out.println("IP地址:"+ip);
		// 获得用户的请求的路径:
		String url = request.getRequestURL().toString();
		String uri = request.getRequestURI();
		System.out.println("获得请求的URL:"+url);
		System.out.println("获得请求的URI:"+uri);
		// 获得发布的工程名：
		String contextPath = request.getContextPath();
		System.out.println("工程名:"+contextPath);
		
	}

	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}

}
###	代码实现
###	总结
####	处理request接收参数的中文乱码的问题:
-	现在无论是GET还是POST提交中文的时候,都会出现乱码的问题.
-	解决:
	-	post提交的解决方案:
		-	POST的参数在请求体中,直接到达后台的Servlet.数据封装到Servlet中的request中.request也有一个缓冲区.request的缓冲区也是ISO-8859-1编码.
		-	设置request的缓冲区的编码:
			-	request.setCharacterEncoding(“UTF-8”);  --- 一定要在接收参数之前设置编码就OK.
	-	Get提交的解决方案:
		-	1.修改Tomcat的字符集的编码.(不推荐)
		-	2.使用URLEncoder和URLDecoder进行编码和解码的操作
		-	3.使用String构造的方法:(用它)
			-	String(byte[] bytes,String charsetName)
####	Request作为域对象存取数据：
-	使用request对象存取数据:
	-	request.setAttribute(String name,String value);
	-	request.getAttribute(String name);返回object对象
-	request的作用范围:
	-	作用范围就是一次请求的范围.
	-	创建和销毁:
		-	创建:客户端向服务器发送了一次请求以后,服务器就会创建一个request的对象.
		-	当服务器对这次请求作出了响应之后.
####	重定向和转发的区别:(redirect和forward的区别)
-	1.重定向的地址栏会发生变化,转发的地址栏不变.
-	2.重定向两次请求两次响应,转发一次请求一次响应.
-	3.重定向路径需要加工程名,转发的路径不需要加工程名.
-	4.重定向可以跳转到任意网站,转发只能在服务器内部进行转发.