# D014 vue+django+neo4j贝类生物知识图谱问答系统LTP实现全源码

> 完整项目收费，可联系QQ: 81040295 微信: mmdsj186011 注明从git来的，谢谢！
也可以关注我的B站： 麦麦大数据 https://space.bilibili.com/1583208775
> 

up主B站：  **麦麦大数据**
关注B站，有好处！

编号:  D014 贝类版本
**可以定制其他数据**
## 视频

暂未发
## 1 系统简介
系统简介：本系统是一个基于Vue.js、Django、MySQL和Neo4j构建的贝类知识图谱可视化与问答系统，其核心功能围绕贝类知识的展示、检索和问答展开。主要包括知识图谱功能，允许用户浏览和理解贝类之间的生物学概念关系；模糊检索功能，支持用户通过关键词进行不精确查询；问答系统，提供关于鸟类分类（如科、目、纲）和分布地区的信息。同时，系统还包含用户管理模块，包括登录、注册、修改个人信息、头像和密码等功能，确保用户个性化和安全的使用体验。
## 2 功能设计
该系统采用典型的B/S架构模式，用户通过浏览器访问由Vue.js构建的前端界面。前端利用Vuex进行状态管理，Vue Router处理路由导航，并使用Echarts等图表库实现数据可视化。前端通过RESTful API与Django后端交互，后端处理业务逻辑并与MySQL数据库进行结构化数据存储，同时利用Neo4j存储和查询贝类知识图谱的非结构化数据，为问答系统提供支持。此外，系统包含数据爬虫和预处理模块，负责抓取和整理贝类数据，构建知识图谱并通过数据融合技术确保数据的一致性和准确性，全面支撑系统功能。
### 2.1系统架构图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e03b2fac4a754426890eb978f72dae40.png)
### 2.2 功能模块图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bde1ec742bb74e1ab1e2eaf2fc699ea5.png)
## 3 功能展示
### 3.1 登录 & 注册
登录界面背景是一个视频，展示和本文系统主题相匹配的内容，登录和注册界面在一个界面下，通过按钮来切换，注册界面输入用户名和密码，会检查这个用户是否存在，登录界面则要检查用户名是否存在以及用户名密码是否正确：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/685e023b6cf24674a933d266391790b0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3f915c6b27984a9fa900d6e160d8c89f.png)
### 3.2 主页
如果通过校验，则可以进入主页，在主页是一个左侧菜单，右侧操作面板的布局，右上角是登录用户的头像和退出按钮：
### 3.3 知识图谱
neo4j 界面展示知识图谱：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/173a6bf8d10d42bbab09cf2dc98f1d62.png)
本利用echarts实现的知识图谱可视化：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f5a96e034b444cd8b8a39b04609679d4.png)
支持输入关键词进行知识图谱的检索，从而展示子图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/aea0dc4379f947d2b8176d3236d0642c.png)
### 3.4 问答功能
基于LTP+neo4j 实现贝类的问答：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a5494a4a18c54ac2be402d3951bf417f.png)
### 3.5 个人设置
个人设置方面包含了**用户信息修改**、**密码修改**功能。
用户信息修改中可以上传头像，完成用户的头像个性化设置，也可以修改用户其他信息。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c8d00099b1434a3998c8e359e66ddafc.png)
修改密码需要输入用户旧密码和新密码，验证旧密码成功后，就可以完成密码修改。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d564b32e3ecb40f6ae36eadbc1607aa6.png)

## 4程序代码
### 4.1 代码说明
代码介绍：该代码实现了一个基于Vue.js的海洋贝类知识图谱可视化功能，利用ECharts库生成交互式的知识图谱。首先，通过getOceanData函数获取海洋贝类的数据，并初始化图表容器。然后，定义数据节点（包括贝类名称、分类、描述等）和关系链接，构建知识图谱的数据结构。接着，配置ECharts的关系图表，设置布局、交互和样式，使其支持缩放、拖拽和节点点击事件。最后，通过点击节点触发的事件，弹出贝类的详细信息框，展示其图片和描述信息。
### 4.2 流程图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b3647dfe28ef40a2881d23e104ef0944.png)
### 4.3 代码实例
```python
<template>
  <div id="ocean-graph" :style="{ width, height }"></div>
</template>

<script>
import { ref, onMounted } from 'vue'
import * as echarts from 'echarts'
import 'echarts/extension/dataGenerator/ Constituent'

export default {
  props: {
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '600px'
    }
  },

  setup() {
    const chart = ref(null)

    onMounted(() => {
      initGraph()
    })

    const initGraph = () => {
      // 获取或模拟数据
      const data = getOceanData()
      // 构建节点和关系
      const nodes = data.map(item => ({
        id: item.id,
        name: item.name,
        type: item.type,
        image: item.image
      }))
      const links = data.map(item => ({
        source: item.parentId,
        target: item.id
      }))

      // 初始化图表
      const option = {
        title: {
          text: '海洋贝类知识图谱'
        },
        series: [{
          type: 'graph',
          layout: 'force',
          force: {
            repulsion: 100,
            edgeLength: 100
          },
          data: nodes,
          links: links,
          roam: true,
          label: {
            position: 'right'
          },
          lineStyle: {
            color: '#4f81bd',
            width: 1
          }
        }]
      }

      // 渲染图表
      chart.value = echarts.init(document.getElementById('ocean-graph'))
      chart.value.setOption(option)

      // 点击事件
      chart.value.on('node_click', (params) => {
        // 显示详细信息
        showInfo(nodes.find(node => node.id === params.data.id))
      })
    }

    const showInfo = (node) => {
      // 显示贝类详细信息
      console.log('点击了', node.name)
      // 可以在这里实现弹窗或侧边栏的显示
    }

    const getOceanData = () => {
      // 模拟数据，应替换为真实接口
      return [
        { id: 1, name: '软体动物门', type: '门', parentId: 0 },
        { id: 2, name: '头足纲', type: '纲', parentId: 1 },
        { id: 3, name: '鱿形上目', type: '上目', parentId: 2 },
        { id: 4, name: '烘烘贝', type: '种', parentId: 3 }
        // 此处应扩展为完整的数据结构
      ]
    }

    return {}
  }
}
</script>

```
---
## 联系方式
