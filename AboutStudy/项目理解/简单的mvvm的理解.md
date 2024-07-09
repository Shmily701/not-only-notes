- EventViewModel是全局的通知，所需监听都可收到

### Viewmodel与ViewDataBinding的关联
1. 通过xml的 <data> <variable> 下的声明对应的ViewModel与Click
2. 框架设计：xml会动态生成的ViewDataBinding
3. 在activity继承类声明指定的ViewModel与ViewDataBinding
4. 最重要的绑定步骤：
    mDatabind.viewmodel = mViewModel  
    mDatabind.click=ProxyClick()      
    而这里mDatabind的属性viewModel与click都是上文xml里声明的两个变量

View与ViewModel的绑定  其中xml扮演了View的角色


### 以app的appLoginActivity为例
>1. 在activity_login.xml中声明<data> <variable>标签并指定viewmodel为LoginViewModel，需要bind的地方通过bind指定 
eg：点击事件等属性的引用可以通过viewmodel.smsVisible 直接类.属性的方式来为xml的布局赋值。

> 2. 通过xml生成ActivityLoginBindingImpl
   ```
   将layoutId与viewDataBinding初始化与layoutId结合起来
        initDataBind().notNull({
            setContentView(it)
        }, {
            setContentView(layoutId())
        })
    （由androidx.databing库实现此功能）

    详情查看 BaseVmVbActivity$initDataBind
    ```
（由androidx.databing库实现此功能）

>3. 在LoginActivity指定ViewModel与ViewDataBinding：
class LoginActivity : BaseActivity<LoginViewModel, ActivityLoginCopyBindingImpl>()

so 当两个xml文件都指定了同一个viewModel那就要看activity在声明时，指定的ViewDataBinding是哪个。
- 亲测，新增 activity_login_copy.xml 后，生成ActivityLoginCopyBindingImpl 在LoginActivity 指定ActivityLoginCopyBindingImpl 则为LoginViewModel与ActivityLoginCopyBindingImpl关联

在Activity中，可以引用viewModle 即


### 约束布局
- ConstraintLayout 每一个布局文件，需要将约束的左右都配置完成
app:layout_constrainStart_toStartof="parent"
app:layout_constrainEnd_toStartof="@+id/tv_mid"

关于Activity与Fragment的绑定关系，请查看MainActivity 


https://www.jianshu.com/u/d41bd226fd04

