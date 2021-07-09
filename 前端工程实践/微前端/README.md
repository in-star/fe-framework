## 前言
随着项目演变与发展，业务场景越来越繁重，会出现诸多的问题。例如项目中有很多的依赖导致项目启动慢、多个项目之间有共同的依赖，无法实现抽离，在更新依赖的时候，需要手动更新所有的项目、项目之间的相互隔离，无法通过一个统一的系统完成整个链路的操作。

![avatar](https://p0.meituan.net/travelcube/aba723dd5766a90fca78309da0dc980893221.png)

## 解决方案对比


| 模式 | 技术栈统一 | 打包速度 | 工程间通信难度 | 现有工程侵入性 |
| :----: | :----: | :----: | :----: |  :----: |
| npm模式 | 否 | 慢（相对） | 正常 | 高 |
| iframe  | 否 | 正常      | 困难 | 高 |
| 中心路由基座模式  | 否     | 快 | 正常 | 低 |

### 微前端
官网对于微前端的定义
> Techniques, strategies and recipes for building a modern web app with multiple teams that can ship features independently.  
一种用于开发具有多个可以独立发布功能的web应用技术。
#### 特性
+ 高隔离性  
  子应用之间相互不影响，能独立运行，部署。子应用与基座之间亦互不干扰。
+ 技术栈无关  
  对子应用之间使用的技术没有特别的限制。
+ 独立开发、独立部署  
  子应用仓库独立，前后端可独立开发。

微服务的价值在于将一个庞大的系统分割成一个个独立的子系统，各个子系统可以实现独立开发、运行、部署等操作。加快开发效率、提供效能。

#### 架构解析
![avatar](https://cdn.jsdelivr.net/gh/csgajcr/imageroom/2021-4-28/1619619097458-image.png)

- 「Shared Stitching Layer」前端基座工程，根据对应的路由的变化切换对应的子应用，类似与控制台。
- 中间应用「Ads Team」等是一个个独立的子业务系统应用，可以采用不同的开发语言，最终集成到基座上。
- 「API Service」后端提供的基础服务，每个子系统可以调用不同的后端服务。  
  基于以上的架构，每个独立的子系统能实现独立开发、运行、部署，对于一些公共的依赖还能继承到基座上，避免每个子系统都引用一些相同的依赖，同时减慢子系统的运行速度。

### 微前端实践  
采用qiankun进行微前端的服务实践，主要基于qiankun对于原有项目侵入性晓，改造成本比较低。本实践采用vue + qiankun进行微前端demo演示。

#### 构建主应用
+ 创建主工程项目，与正常的vue的应用项目创建流程一致。
+ 安装qiankun依赖
+ 改造项目的入口文件（main.js）  
```
    // 导入qiankun内置函数
    import {
        registerMicroApps, // 注册子应用
        runAfterFirstMounted, // 第一个子应用装载完毕
        setDefaultMountApp, // 设置默认装载子应用
        start // 启动
    } from "qiankun";
    
    let app = null;
    /**
     * 渲染函数
     * appContent 子应用html
     * loading 子应用是否设置loading
     */
    function render({ appContent, loading } = {}) {
        if (!app) {
            app = new Vue({
                el: "#container",
                router,
                store,
                data() {
                    return {
                        content: appContent,
                        loading
                    };
                },
                render(h) {
                    return h(App, {
                        props: {
                        content: this.content,
                        loading: this.loading
                        }
                    });
                }
            });
        } else {
            app.content = appContent;
            app.loading = loading;
        }
    }; 
    
    /**
    * 路由监听
    * @param {*} routerPrefix 前缀
    */
    function genActiveRule(routerPrefix) {
        return location => location.pathname.startsWith(routerPrefix);
    }
    
    // 调用渲染主应用
    render();
    
    // 注册子应用
    registerMicroApps(
        [
            {
                name: "vue-aaa"
                entry: "//localhost:7771",
                render,
                activeRule: genActiveRule("/aaa")
            },
            {
                name: "vue-bbb"
                entry: "//localhost:7772",
                render,
                activeRule: genActiveRule("/bbb")
            },
        ],
        {
            beforeLoad: [ 
                app => {
                    console.log("before load", app);
                }
            ], // 挂载前回调
            beforeMount: [
                app => {
                    console.log("before mount", app);
                }
            ], // 挂载后回调
            afterUnmount: [
                app => {
                    console.log("after unload", app);
                }
            ] // 卸载后回调
        }
     )
     
     // 设置默认子应用,参数与注册子应用时genActiveRule("/aaa")函数内的参数一致
    setDefaultMountApp("/aaa");
    
    // 第一个子应用加载完毕回调
    runAfterFirstMounted(()=>{});
    
    // 启动微服务
    start();

```

#### 构建子应用
子应用入口文件，添加挂载的时候触发的生命周期函数bootstrap、mount、unmount。bootstrap可以注册基建工程传递进来的组件，状态等。mount主要定义子应用相关的router、已经对于vue对象的创建以及挂载操作。
```
import Vue from "vue";
import VueRouter from "vue-router";
import App from "./App.vue";
import "./public-path";
import routes from "./router";

Vue.config.productionTip = false;

// 声明变量管理vue及路由实例
let router = null;
let instance = null;

// 导出子应用生命周期 挂载前
export async function bootstrap(props) {
    console.log(props)
}

// 导出子应用生命周期 挂载前 挂载后
**注意，实例化路由时，判断当运行在qiankun环境时，路由要添加前缀，前缀与主应用注册子应用函数genActiveRule("/aaa")内的参数一致**
export async function mount(props) {
  router = new VueRouter({
    base: window.__POWERED_BY_QIANKUN__ ? "/aaa" : "/",
    mode: "history",
    routes
  });
  instance = new Vue({
    router,
    store,
    render: h => h(App)
  }).$mount("#app");
}

// 导出子应用生命周期 挂载前 卸载后
export async function unmount() {
  instance.$destroy();
  instance = null;
  router = null;
}

// 单独开发环境
window.__POWERED_BY_QIANKUN__ || mount();

```

#### 子应用静态资源路径的配置
当前子应用运行在qiankun环境下时，需要指定子应用的静态资源的路径，以便主应用（基建工程）能识别相应的路径。
```
if (window.__POWERED_BY_QIANKUN__) {
  // eslint-disable-next-line no-undef
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}

```

### 技术难点
#### 主工程与子工程之间的通信
+ 静态数据的传递  
  在子工程挂载到主工程的时候，主工程可以通过prop传递数据到子应用中，这种适合传递静态的数据，例如一些通用组件、主应用的数据状态等。
+ 动态传递  
  通过使用发布订阅模式的第三方类库解决通信问题，类似rxjs、eventbus等。

#### 菜单权限控制
对于登录态，登录逻辑可以由主工程进行实现，登录态下发到相应的子应用中去。菜单权限控制则可以通过在菜单路由中的mete携带可以访问该菜单的角色的标志，菜单组件使用鉴权指令即可实现控制菜单的显示隐藏，从而达到不同的角色之间菜单访问权限的控制。
