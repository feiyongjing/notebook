# react常见UI组件库使用
- 国外常用material-ui
  - 官网 https://v4.mui.com/zh/
  - github https://github.com/mui-org/material-ui
- 国内常用蚂蚁金服的ant-design
  - 官网 https://ant-design.antgroup.com/components/overview-cn/
  - github https://github.com/ant-design/ant-design

# ant-design UI使用
### 安装ant-design UI
~~~shell
npm install antd
~~~

### ant-design UI 弹窗组件使用
- 注意ant-design UI v5版本默认就完成了按需引入样式，参考 https://blog.csdn.net/tuzi007a/article/details/129868891
- antd/dist/reset.css 是用来重置一些基础样式
- 如果环境不支持默认的 Tree Shaking 按需加载就参考 https://mobile.ant.design/zh/guide/import-on-demand/
~~~js
import React, { Component } from 'react';
import { Button, Modal, ConfigProvider } from 'antd';

export default class App extends Component {

  state = {
    open: false,
  }

  showModal = () => {
    this.setState({ open: true })
  };

  handleOk = () => {
    this.setState({ open: false })
  };

  handleCancel = () => {
    this.setState({ open: false })
  };

  render() {

    const { open } = this.state

    return (
      <>
        {/* 使用 ConfigProvider 组件标签包裹其他的 ant-design UI组件，并且设置这些组件的样式 */}
        <ConfigProvider
          theme={{
            token: {
              colorPrimary: 'red', // 主题颜色
            }
          }}>
          <Button className='custom-button' type="primary" onClick={this.showModal}>
            添加记录
          </Button>
          <Modal title="记录弹窗标题" open={open} onOk={this.handleOk} onCancel={this.handleCancel}>
            <p>你确认是否添加记录？</p>
          </Modal>
        </ConfigProvider >
      </>
    );
  }
}
~~~





