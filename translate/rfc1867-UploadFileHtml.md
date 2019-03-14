# RFC 1867  
# HTML 基于表单的文件上传

- 备忘录当前的状态
	
	当前备忘录的定义了的一个对于网络社区而言的实验性协议 。这个备忘录并非是一个标准的网络协议 ，对于提升的建议和讨论是被请求的 ，当前备忘录的分发是没有限制的 。

## 1. 摘要

	现在 ，HTML 表单允许表单产生者请求读取表单的用户信息 。在那些用户的输入是必要的应用能够证明这些

	表单是有用的 。然而 ，这种能力是有限制的 ，因为 HTML 表单没有提供用户上传文件数据的方法 。当需要提供从用户获取文件的服务时 ，需要实现自定义的用户应用 。（ 例如一些自定义的浏览器通过邮件列表来是实现网络交谈 。）当文件上载成为一种特性后 ，许多应用会因此受益 ，这对 HTML 提出了一个扩展建议 ，允许信息的提供者对于文件上载请求表现出一致性和对于文件上载的相应要和 MIME 保持兼容性 。保持向后兼容的策略来允许新的服务对现在的用户代理有作用 。

## 2. HTML 提交文件的表单

	现在的 HTML 表格对于属性类型定义的八个可能值 ，如下 CHECKBOX, HIDDEN, IMAGE,PASSWORD, RADIORESET, SUBMIT, TEXT .

	额外的 , 对于使用了 POST  METHOD 和有默认值 "application/x-www-form-urlencoded" 的表单单元义了默认的 ENCTYPE 属性 .
		
	这个建议使 HTML 做出了两个改变 .
	
	1. 给 INPUT 属性添加了 FILE 可选项 .
	2. 允许 INPUT 标签 ACCEPT 属性 , ACCPUT 属性是媒体类型或者允许输入的模式的列表 .

	文档中还定义了额外的 MIME 类型 - multipart/form-data 和当用户代理被 
	ENCTYPE="multipart/form-data" 和 <INPUT type="file"> 类型标签中断的特殊行为 .

	这些改变都需要被独立的考虑 , 而且都是上传文件的必要性的证明 . 

	表单的作者从用户请求一个或者多个文件 , 例子如下 :
	
	```code
	<FORM EMCTYPE="multipart/form-data" ACTION="_URL_" TYPE="file" >

	File to process: <INPUT NAME="userfile1" TYPE="file">

	<INPUT TYPE="submit" VALUE="Send File">

	</FORM>

	```
	
	对于 HTML DTD 的改变是给实体添加了 "InputType" . 对于 INPUT 标签 , 建议存在一个 ACCEPT 属性,ACCEPT 属性由一串媒体类型组成的列表 .

	```code
	... (other elements) ...
	<!ENTITY % InputType "(TEXT | PASSWORD | CHECKBOX |
							RADIO | SUBMIT | RESET |
							IMAGE | HIDDEN | FILE )">
	<!ELEMENT INPUT - 0 EMPTY>
	<!ATTLIST INPUT
				TYPE %InputType TEXT
				NAME CDATA #IMPLIED  -- required for all but submit and reset
				VALUE CDATA #IMPLIED
				SRC %URI #IMPLIED  -- for image inputs --
				CHECKED (CHECKED) #IMPLIED
				SIZE CDATA #IMPLIED  --like NUMBERS,
										but delimited with comma, not space
				MAXLENGTH NUMBER #IMPLIED
				ALIGN (top|middle|bottom) #IMPLIED
				ACCEPT CDATA #IMPLIED --list of content types
				>
	... (other elements) ...

	```
## 3. 实现的建议
	
	当用户代理对于 HTML 的解释有很大的余地时, 可以根据内容来选择最合适的状态机 , 这个章节给用户代理一些
	实现文件上载的建议 .
	
### 3.1 文件部件的显示 
 
 	当遇见一个 INPUT 标签是 FILE 类型时 , 浏览器应该显示文件名 (文件选中前) , 浏览按钮以及选择方式 .选择浏览按钮会出现文件选择界面 . 基于窗口的浏览器会弹出来选择文件 . 在当前的文件选择窗口 , 用户可以修改当前选择 , 添加新选择的文件 . 浏览器的实现者应该让文件列表可以手动修改 .

	如果存在 ACCEPT 属性 , 浏览器应该强制文件模式与对应的平台适应 .
	
### 3.2 提交动作

 	当用户完成表单 , 并且选择提交单元 , 浏览器应该发送表单数据和选中文件的内容 . 对于发送大量数据的二进制文件或者文本文件时 , 编码类型 application/x-www-form-urlencoded application/x-www-form-urlencoded 是没有效果的 . 然而 , 一个新的媒体类型 multipart/form-data , 建议用来有效的从客户端到服务端传输和填写相关的值 .

### 3.3 使用 multipart/form-data 

	multipart/form-data 的定义在第七章, 选择的边界不会在数据中出现.() 表单发送顺序安装表单中各个域的顺序作为多帧流的一部分,每个部分用初始的HTML表单的 INPUT name 来校验. 在每个部分的媒体类型已知的请况下,每段都需要有对应的标签. (从文件扩展名或者操作系统的类型推断出) 否则被认为是 application/octet-stream. 
		
	如果有多个文件被选中, 用 multipart/mixed 的格式同时传输 .
	
	当 HTTP 协议能够传输任意的二进制文件, 邮件传输的编码格式是7位编码. 给每段提供的值需要被编码,如果提供的值不是默认编码,需要提供 content-transfer-encoding. [RFC 1521 第5章有更详细的内容]
		
	原始的本地文件名同样需要提供, 可以选择 content-disposition: form-data 作为头或者 在多文件下 用 content-disposition: file 作为头来做 filename 参数. 客户端应该采用最有效的方式来传递文件名. 如果客户端操作系统下的文件名不是 US-ASCII 编码, 文件名应该自适应,或者按照 RFC1522 中的方式来编码. 
	
	在服务端, ACTION 指向 URL 通过 CGI 来实现表单动作. 在此类情况下, CGI 程序需要注意 content-type 的类型是 multipart/form-data, 解析此值域. (校验值的正确性, 在子线程中将数据写入本地文件)

### 3.4 其他属性的解释

	用标签 <INPUT TYPE=file> 的 VALUE 属性来表示默认文件, 这种方法的使用依赖平台. 在有多个事物的序列, 这个方法是很有用的. (避免提示用户多次输入相同的文件名) 
		
	用 SIZE=width, height 来指定 SIZE 属性, width 是文件名默认的宽度, height 是显示出的可选择的
文件个数, 当你想要选择多个文件或者在输入界面显示多个文件时, 这就有很大用处了. (一般会在旁边设置一个 browse 按钮). 当 height 没有指定时, 用来显示一个单行 (当你只想选择一个文件时). 当 height 大于 1 时, 用一个带滑条的多行文本区域来显示. 
		
## 4 向后兼容的问题
	
	对于采用增强型的 www 表单机制不是必要的, 但迁移策略的制定还是很有用的 : 用户仍然使用旧版浏览器在文件上传界面上传文件, 使用一个帮助应用 . 当前大多数的 WEB 浏览器 , 当浏览器被给予 < INPUT TYPE=FILE > 会被当作 < INPUT TYPE=TEXT > 并且给用户一个文本框 , 用户在文本框内输入文件名 . 额外的 , 当前浏览器会忽略 < FROM > 中的 ENCTYPE . 用 application/x-www-form-urlencoded 来传输文件数据 . 

## 5 其他思考

### 5.1 压缩, 加密

	这份方案没有解决可能出现的文件压缩问题 . 在经过一些思考后 , 最需要优化的问题是对于让浏览器来选择文件是否压缩 , 这样太复杂 . 一些链路层传输机制通过链路来执行文件压缩 , 在链路层选择优化压缩文件不是很合适 . 浏览器可行的做法是按照需要来对文件数据用某种压缩方法来产生传输加密内容 . 服务端在处理前进行解压缩 . 然而 , 我们的建议并没有包含这个建议 . 

	同样的没有包含数据加密的机制 ; 加密的机制应该郊游其他任何的加密数据传输机制 , 无论是安全 HTTP 还是 mail .

### 5.2 文件传输的延时
		
	在一些情况下 , 在准备接收数据前 , 服务端最好校验表单数据中的各类单元 (用户名 , 账户及其他 ) . 在经过思考后 , 服务器为此实现一系列表单看起来是最好的办法 . 先前校验的数据单元会作为隐藏区域返回给客户端 , 或者通过表单排序 , 好让需要校验的单元首先出现 . 这使得在运行复杂应用的服务器上维护一个事物状态成为可能 , 同样允许那些有着简单输入的能够简单的构建 . 

	在全部的传输中 , HTTP 协议需要一个 content-legth . HTTP 客户端对于所有的文件输入都应当支持 content-length , 当一个文件数据太大以至于不能够处理时 , 服务器应当进入繁忙的状态 , 并且发送一个错误码后uan 
	段

### 5.3 对于文件传输的返回值的选择

### 5.4 对于 <INPUT> 的过载

### 5.5 content-type 的默认值

### 5.6 允许 mailto 的 ACTION 

### 5.7 远程文件的第三方传输

### 5.8 采用 ENCTYPE=x-www-form-urlencod 来传输

### 5.9 CRLF 用作分离标识

### 5.10 非 ASCII 编码的文件名

## 6 例子

## 7. multipart/form-data 的注册

## 8. 安全考虑

## 9. 总结

