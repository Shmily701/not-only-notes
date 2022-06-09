### MVC
V 视图层 用户界面
M 数据层 数据信息
C 控制层 业务逻辑，响应从"视图层"输入的指令，操作"数据层"中的数据，产生最终结果。

View传送指令到Controller  Contorller完成业务逻辑后，要求Model改变状态，
Model将新的数据发送到View，用户反馈。
V驱动C  C驱动M  M更新V 通信是单向的
Controller 非常薄，只起到路由的作用，而View 非常厚，业务逻辑都部署在View

            View
Controller         Model 

View->Controller->Model->View

### MVP
MVP将Controller改为Presenter，并改变了通信方向。
1. 各部分间为双向通信
2. View与Model[不发生联系]，都通过Presenter传递。
3. View非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署其中。
         View
Presenter     Model
View<->Presenter<->Model 


### MVVM
MVVM模式将Presenter改为ViewModel 
区别在于采用双向绑定:View的变化会自动反应在ViewModel
    
