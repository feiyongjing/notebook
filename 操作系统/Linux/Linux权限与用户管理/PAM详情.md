# PAM 简介
PAM 的全称为“可插拔认证模块（Pluggable Authentication Modules）”。设计的初衷是将不同的底层认证机制集中到一个高层次的API中，从而省去开发人员自己去设计和实现各种繁杂的认证机制的麻烦
请注意：PAM 不仅仅在用户登录时才发挥作用，sudo命令，su命令，passwd命令都会用到 PAM。前文中所有提及“登录”的地方都仅仅是举例，您完全可以用其他需要用户认证的服务（或者命令）去举例，从而更全面地理解 PAM。

## PAM 认证过程
- 使用者执行/usr/bin/passwd程序，并输入密码。
- passwd开始呼叫PAM模块，PAM模块会搜寻passwd程序的PAM 相关设定文件，这个设定文件一般是在/etc/pam.d/里边的与程序同名的文件 ，即PAM 会搜寻/etc/pam.d/passwd这个设置文件。
- 经由/etc/pam.d/passwd 设定文件的数据，取用PAM 所提供的相关模块来进行验证。
- 将验证结果回传给passwd 这个程序，而passwd 这个程序会根据PAM回传的结果决定下一个动作（重新输入密码或者通过验证）

### PAM 的具体工作主要有以下四种类别（type）：account，auth，password 以及 session。这里，我们用非定义化的语言来简单解释一下这四种类别。
1. account：表示账号类接口，主要负责账户合法性检查，确认账号是否过期，是否有权限登录系统等
2. auth：表示鉴别类接口模块类型用于检查用户密码，并分配权限
3. password：口令类接口，控制用户更改密码的全规程
4. session：会话类接口，实现从用户登录成功到退出的会话控制，并且记录用户的登录和注销信息，例如/ver/log/secure 下记录了session open 和session close

# Linux-PAM 的配置文件综述，
### 参考：https://www.cnblogs.com/it-log/p/17516475.html
### 参考：https://www.cnblogs.com/ilinuxer/p/5087447.html
1. pam的配置文件当中为每种应用定义其需要用到的模块，包括驱动、授权机制等。
	配置文件  
	  - 模块文件目录（pam的动态库文件路径）：/lib/security/ 或 /lib64/security/ 或 /lib/x86_64-linux-gnu/security/  
	  - 块配置文件：/etc/security/*.conf  
      - 环境相关的设置：/etc/security/  
	  - 通用配置文件：/etc/pam.conf ，默认不存在。  
	  - 专用配置文件目录：/etc/pam.d/，配置文件通常以每一个使用 PAM 的程序的名称来命令。比如 /etc/pam.d/su，/etc/pam.d/login 等等。还有些配置文件比较通用，经常被别的配置文件引用，也放在这个文件夹下，比如 /etc/pam.d/system-auth
	  - 注意：如/etc/pam.d 存在，/etc/pam.conf 将失效  
2. pam模块通过读取模块配置文件完成用户对系统资源的使用控制  
   - 注意：修改PAM 配置文件将马上生效  
   - 建议：编辑pam规则时 ，保持至少打开一个root会话，以防止root 身份验证错误  
   - 通用配置文件/etc/pam.conf 格式：application 、type 、control 、module-path 、module-arguments，即服务名称 工作类别 控制模式 模块路径 模块参数  
   - 专用配置文件/etc/pam.d/* 格式：type、control、module-path、module-arguments，即工作类别 控制模式 模块路径 模块参数    
   - 说明：
     1. tyoe（工作类别）：包括auth，account，password，session。
	    - auth：认证和授权；
		- account：与账号管理相关的非认证功能；
		- password：用户修改密码时使用；
		- session：用户获取到服务前后使用服务完成后要进行的一些附属性操作
		- -type：表示因为缺失而不能加载的模块将不记录到系统日志, 对于那些不总是安装在系统上的模块有用  
	 2. control（控制模式）：pam如何处理与该服务相关的pam的成功或失败情况？同一种功能的多个检查之间如何进行组合？有两种实现机制。  
	    - 使用一个关键词来定义：
		  - required（必填）：一票否决，表示本模块必须返回成功才能通过认证，但是如果该模块返回失败，失败结果也不会立即通知用户，而是要等到同一type中的所有模块全部执行完毕再将失败结果返回给应用程序。 必要条件
		  - requisite（必要）：一票否决，该模块必须返回成功才能通过认证，但是一旦该模块返回失败，将不再执行同一type（工作类别）内的任何模块，而是直接将控制权返回给应用程序。必要条件
		  - sufficient（足够）:	一票通过， 表明本模块返回成功则通过身份认证的要求，不必再执行同一type（工作类别）内的其它模块，但如果本模块返回失败可忽略。充分条件
		  - optional（可选项）:	表明本模块是可选的，它的成功与否不会对身份认证起关键作用，其返回值一般被忽略
		  - include（包括）：使用其他配置文件中同样功能的相关定义来进行检查
		- 使用一到多组“return status=action”来定义：[status1=action1  status2=action2....]
		  - status：指定此项检查的返回值，其可能的取值有多种。如success。
		  - action：采取的操作，其可能的取值常用的有6种，如ok，done，die，ignore，bad，reset。
		  - ok：模块通过，继续检查
		  - done：模块通过，返回最后结果给应用
		  - bad：结果失败，继续检查
		  - die：结果失败，返回失败结果给应用
		  - ignore：结果忽略，不影响最后结果
		  - reset：忽略已经得到的结果
	 3. module-path（pam模块文件路径，即使用的pam的动态库）：用来指明本模块对应的程序文件路径（默认是只需写pam的动态库名称，会在pam的动态库文件路径的默认路径下查找，也可以写绝对路径）
	 4. module-argument（模块参数）：用来传递给该模块的参数，可以使用man [pam的动态库名称，不需要加文件后缀] 查看模块的参数详细使用，例如 man pam_limits
		- pam_unix.so：	在不同的type（工作类别）下有不同的用途
		- nullok：允许使用空密码；
		- try_first_pass：提示用户输入密码之前，首先检查此前栈中已经得到的密码；
		- pam_env.so：通过配置文件来为用户设定或撤销环境变量，/etc/security/pam_env.conf
		- pam_shells.so：检查用户使用的是否为合法shell，/etc/shells
		- pam_limits.so：资源限制，/etc/sercurity/limits.conf ；/etc/sercurity/limits.d/*