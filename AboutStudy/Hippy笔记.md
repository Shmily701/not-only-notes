Hippy是跨平台开发框架。
1. 构建引擎初始化所需参数EngineInitParams
  （注意设置EngineInitParams的必要、非必要参数）
   从构建方法中，调用的EngineInitParams$check方法，很容易看出，必要参数有哪些

   ```
   1. context
   2. imageLoader
   3. if (!EngineInitParams.debugMode) {
         coreJSAssetsPath || coreJSFilePath
      }
      // debugMode = false 时必须设置coreJSAssetsPath或coreJSFilePath（
      // debugMode = true时，所有jsbundle都是从debug server上下载） TODO 这句不懂

    ```    

2. 初始化引擎
   

3. 初始化完成的回调中，加载前端模块
   加载前端模块必要 HippyEngineManagerImpl$loadModule

   1. context
   2. 必须指定加载的组件，且loadParams.componentName 的值必须与js文件中的“appName”一致
   3. if (!EngineInitParams.debugMode) {
         coreJSAssetsPath || coreJSFilePath
         //二者必须设置一个
      }
      //debugMode = false 时必须设置jsAssetsPath或jsFilePath
      //（debugMode = true时，所有jsbundle都是从debug server上下载） TODO 这句不懂


4. 

### 终端事件 
1. 移动端向前端发送消息（用于两端的交互）使用HippyMap
// 带上参数，传递给前端的rootview：比如：Hippy.entryPage: class App extends Component


HippyDrawable 图片包装接口
HippyImageLoader 图片加载 提供了本地  网络图片的加载 TODO TestCode


