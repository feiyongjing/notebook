# Jenkins简介
Jenkins是一个开源的、用Java编写的持续集成和持续交付（CI/CD）工具。它提供了一种简单易用的方式来自动化构建、测试和部署软件。Jenkins的主要目标是帮助开发团队加快软件开发过程，提高软件质量，并通过自动化流程减少手动操作和重复性工作

### Jenkins具有以下特点和优势
- 持续集成：Jenkins支持通过持续集成管道（Pipeline）来自动化构建、测试和部署。它能够检测代码的变更，并触发相应的构建和测试过程，确保及时地发现和解决问题。
- 插件生态系统：Jenkins拥有丰富的插件生态系统，可以扩展各种功能和集成其他工具。无论是构建工具、版本控制系统、测试框架还是部署平台，都可以通过插件进行集成，满足不同项目的需求。
- 可扩展性：Jenkins具有良好的可扩展性，可以根据项目的需求进行定制和配置。它支持并行化构建和分布式构建，可以在多个节点上执行任务，提高构建的效率和并发能力。
- 多平台支持：Jenkins可以运行在各种操作系统上，包括Windows、Linux和Mac OS等。它也可以与各种开发工具和平台无缝集成，适用于不同的开发环境。
- 可视化界面：Jenkins提供了直观的用户界面，方便用户进行配置、监控和管理。用户可以通过Web界面轻松地创建和管理任务，查看构建结果和日志等信息


# Jenkins安装
官网安装地址：https://www.jenkins.io/download/ 选择对应的安装方式进行安装

### docker 安装 Jenkins
~~~shell
# 测试使用：会自动更新到新 LTS 版本
docker pull jenkins/jenkins:lts

# 拉取指定版本镜像（生产首选，避免自动更新导致问题）：指定jenkins的TLS版本 和 容器内的JDK 版本
docker pull jenkins/jenkins:2.541.2-jdk17

# 启动容器（挂载数据卷，避免数据丢失）
docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:2.541.2-jdk17    

# 必须挂载 jenkins_home 数据卷：避免容器删除后配置 / 插件 / 构建记录丢失
# 如需访问宿主机 Docker，挂载 /var/run/docker.sock（需注意权限：容器内 jenkins 用户需加入 docker 组）


docker run -d --name jenkins -p 8180:8080 -p 50000:50000 -v 'E:\project\test\jenkins-home':/var/jenkins_home jenkins/jenkins:2.541.2-jdk17   
~~~

### Jenkins使用
参考1：https://www.cnblogs.com/nhdlb/p/18561435
参考2：https://blog.csdn.net/2301_79104307/article/details/148461186
参考3：https://www.bilibili.com/video/BV1Z2qYBSERP/?spm_id_from=333.337.search-card.all.click&vd_source=a0453d1b6998ae404eb4f6652e9b92d6








