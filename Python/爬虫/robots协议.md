### 一般网站都会设置robots协议，访问 https://域名/robots.txt 可以看到robots协议，规定了网站的那些资源释放可以爬取

# 例如
https://www.bilibili.com/robots.txt

### Robots.txt文件怎么写？

基本语法

User-agent:指定对哪些爬虫生效！*号代表全部搜索引擎，百度(Baiduspide)、谷歌(Googlebot)、360(360Spider)
Disallow:不允许抓取
Allow:允许抓取
#:注释

例如以下禁止所有站点的爬虫
User-agent: *
Disallow: /