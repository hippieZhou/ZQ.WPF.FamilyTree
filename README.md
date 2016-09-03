# Family-Tree(Prism Tips)


## 1. 框架搭建
### 1.1 安装 prism 包（通过NuGet Packages 管理器来安装prism6 程序包）
### 1.2 App->Bootstrapper->Shell
App修改如下：
```xaml
<Application x:Class="ZQ.PrismUnityApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:ZQ.PrismUnityApp">
</Application>
```
```C#
public partial class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        var bootstrapper = new Bootstrapper();
        bootstrapper.Run();
    }
}
```
创建Bootstrapper类
```C#
class Bootstrapper : UnityBootstrapper
{
    protected override DependencyObject CreateShell()
    {
        return Container.TryResolve<Shell>();
    }

    protected override void InitializeShell()
    {
        Application.Current.MainWindow.Show();
    }

    protected override void ConfigureModuleCatalog()
    {
        #region 基于代码方式的模块加载方法
        var typeGuidance = typeof(Module.Guidance.GuidanceModule);
        var guidanceModule = new ModuleInfo()
        {
            ModuleName = typeGuidance.Name,
            ModuleType = typeGuidance.AssemblyQualifiedName
        };

        var typeSyutsou = typeof(Module.Syutsou.SyutsouModule);
        var syutsouModule = new ModuleInfo()
        {
            ModuleName = typeSyutsou.Name,
            ModuleType = typeSyutsou.AssemblyQualifiedName
        };

        var typeSettings = typeof(Module.Settings.SettingsModule);
        var settingsModule = new ModuleInfo()
        {
            ModuleName = typeSettings.Name,
            ModuleType = typeSettings.AssemblyQualifiedName
        };

        var typeAbout = typeof(Module.About.AboutModule);
        var aboutModule = new ModuleInfo()
        {
            ModuleName = typeAbout.Name,
            ModuleType = typeAbout.AssemblyQualifiedName
        };

        this.ModuleCatalog.AddModule(guidanceModule);
        this.ModuleCatalog.AddModule(syutsouModule);
        this.ModuleCatalog.AddModule(settingsModule);
        this.ModuleCatalog.AddModule(aboutModule);

        #endregion
    }
}
```
创建主Shell
```xaml
 <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="120"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Grid Grid.Column="0" Background="AliceBlue">
            <StackPanel VerticalAlignment="Bottom">
                <RadioButton IsChecked="True" GroupName="rb_Menu" Content="祖训" Command="{Binding ChooseMenuCmd}" CommandParameter="{Binding Content, RelativeSource={RelativeSource Self}}"/>
                <RadioButton GroupName="rb_Menu" Content="世祖" Command="{Binding ChooseMenuCmd}" CommandParameter="{Binding Content, RelativeSource={RelativeSource Self}}"/>
                <RadioButton GroupName="rb_Menu" Content="设置" Command="{Binding ChooseMenuCmd}" CommandParameter="{Binding Content, RelativeSource={RelativeSource Self}}"/>
                <RadioButton GroupName="rb_Menu" Content="关于" Command="{Binding ChooseMenuCmd}" CommandParameter="{Binding Content, RelativeSource={RelativeSource Self}}"/>
            </StackPanel>
        </Grid>
        <TabControl SelectedIndex="{Binding ModuleIndex}" Grid.Column="1" prism:RegionManager.RegionName="{StaticResource MainRegion}" />
    </Grid>
```
创建主Shell对应的ViewModel
```C#
[Flags]
public enum Menu
{
    祖训 = 0x01,
    世祖 = 0x02,
    设置 = 0x03,
    关于 = 0x04
};

public class MainViewModel : BindableBase
{
    private string _title = "Prism APP";
    public string Title
    {
        get { return _title; }
        set { SetProperty(ref _title, value); }
    }

    private int _moduleIndex;
    public int ModuleIndex
    {
        get { return _moduleIndex; }
        set { SetProperty(ref _moduleIndex, value); }
    }

    private DelegateCommand<string> _chooseMenuCmd;
    public DelegateCommand<string> ChooseMenuCmd
    {
        get
        {
            return _chooseMenuCmd ?? (_chooseMenuCmd = new DelegateCommand<string>((str) =>
                {
                    Menu menu = Menu.关于;
                    if (Enum.TryParse(str, out menu))
                    {
                        switch (menu)
                        {
                            case Menu.祖训:
                                this.ModuleIndex = 0;
                                break;
                            case Menu.世祖:
                                this.ModuleIndex = 1;
                                break;
                            case Menu.设置:
                                this.ModuleIndex = 2;
                                break;
                            case Menu.关于:
                                this.ModuleIndex = 3;
                                break;
                            default:
                                return;
                        }
                        Debug.WriteLine(menu);
                    }
                }));
        }
    }
}
```
各个子模块进行注册
由于每个模块的注册方法都类似，代码也基本相同，这里简单列出设置模块的示例代码：
```C#
public class SettingsModule : IModule
{
    IRegionManager _regionManager;
    public SettingsModule(RegionManager regionManager)
    {
        _regionManager = regionManager;
    }
    public void Initialize()
    {
        _regionManager.RegisterViewWithRegion("MainRegion", typeof(Views.MainView));
    }
}
```


## 2. 模块注册
模块注册有多种方式，这里我简要汇总一下（从目前个人接触的来看，：））
### 2.1 在代码中注册模块

### 2.2 通过XAML文件注册模块

### 2.3 通过配置文件注册模块

### 2.4 在文件目录中查找模块来进行注册

参考链接：
[prism 4 模块配置 管理](http://www.mamicode.com/info-detail-1116983.html)


## 3. 模块切换
模块切换有多种方式，依据个人能力，这里我做一个简单汇总，后期会陆续更新
### 3.1 请求页面跳转
### 3.2 动态加载模块
### 3.3 通过事件聚合器来进行模块更新


## 4. 事件通知
#### 4.1 定义相应类型的类，使该类继承自 PubSubEvent<T> 即可（参数T为我们需要传递的实际数据类型）。
```C#
public class Events:PubSubEvent<bool>{}
```
### 4.2 获取全局事件聚合器，进行事件的发布与订阅（发布-订阅模式，可以类比Mvvmlight中的Messenger机制）
```C#
//事件订阅
ServiceLocator.Current.GetInstance<IEventAggregator>().GetEvent<Events>().Subscribe((b)=>
{
	//处理相关逻辑
});

//事件发布
ServiceLocator.Current.GetInstance<IEventAggregator>().GetEvent<Events>().Publish(true);
```
需要说明的是，关于通过事件聚合器来进行事件的收发与订阅是可以全局使用。对于整个项目工程来说，任何一个模块中只要订阅了对应的事件，都可以收到全局任何一个模块中发布的事件。


## 5. 系统日志

