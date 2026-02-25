## Grafana是什么？
Grafana是一款用Go语言开发的开源数据可视化工具

## Grafana Docker安装
Grafana Docker镜像有两个版本:
1. Grafana Enterprise: grafana/grafana-enterprise
2. Grafana开源版: grafana/grafana-oss
推荐直接使用Grafana Enterprise默认版本。因为它是免费的，包含了所有OSS（开源）版本的功能。此外，还可以选择升级到完整的企业功能集，其中包括对企业插件的支持
~~~shell
# 临时启动
docker run -d -p 3000:3000 --name=grafana grafana/grafana-enterprise

# 持久化数据和安装插件启动
docker run -d -p 3000:3000 --name=grafana \ 
  -v "$PWD/data:/var/lib/grafana" \
  -e "GF_INSTALL_PLUGINS=grafana-clock-panel@1.0.1, grafana-simple-json-datasource" \ 
  grafana/grafana-enterprise  

# GF_INSTALL_PLUGINS环境变量：在Grafana启动时将每个插件名称发送到grafana-cli plugins install ${plugin}并安装它们
~~~




