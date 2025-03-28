# 关于创建

[Package JavaFX applications | IntelliJ IDEA Documentation (jetbrains.com)](https://www.jetbrains.com/help/idea/packaging-javafx-applications.html#java_fx_build_artifact)

# 简明教程

## 启动
虽说有两种启动方式，但其根本都是调用重写public void start(Stage stage)

```java
@Override  
public void start(Stage stage) throws IOException {  
FXMLLoader fxmlLoader = new FXMLLoader(HelloApplication.class.getResource("hello-view.fxml"));  
Scene scene = new Scene(fxmlLoader.load(), 320, 240);  
stage.setTitle("Hello!");  
stage.setScene(scene);  
stage.show();  
}  
  
public static void main(String[] args) {  
launch();  
}
```

```java
public class Hello2 {  
public static void main(String[] args) {  
Application.launch(HelloApplication.class);  
//注意，这里可以传入args，Application.launch(HelloApplication.class,args);但由于我们不做演示，所以省略
}  
}

```


## 生命周期
1. 建立`Application`类的实例。
2. 调用`init`方法。默认的`init`方法什么也不做，但我们可以重新该方法，以实现特定的初始化任务。
3. 调用`start`方法。这是一个抽象方法，因此我们创建的`Application`的子类必须实现该方法。`start`方法用于创建并显示UI接口。
4. 消息处理，完成用户任务，等待应用程序结束。
5. 调用`stop`方法。该方法提供了在程序结束前完成一些必要任务的机会，如内存释放等。



## 场景图



场景图中的节点分为两类：

- 分支节点：`Parent`及其实现类。
- 叶子节点：除分支节点的外的其它节点，或者其它不能有子类的叶子类。每个场景图树中只有一个节点没有父节点，这个节点被称为“根”节点。

节点的限制如下：

- 一个节点在场景图中只能出现一次，如果一个程序向分支点添加了一个子节点，并且该子节点已经是其它分支节点的子节点或场景的根节点，则该节点将自动从其前父节点中移除。
- 节点之间不能存在循环结构。





## 渲染界面

![image.png](http://obimage.wenzhuo4657.cn/20240505153703.png)


流程
0:继承Stage创建舞台，后续加载舞台的方法是该类已经实现的show方法
@Override public final void show() {  
super.show();  
}
1，加载fxml文件获取Parent 
2，通过Parent获取场景Scene ，
3，设置Scene其他效果
4,通过Parent获取node节点在java代码设置监听器进行控制等，

5，渲染舞台，程序启动


注意，3、4步骤其实可以省略，如果仅仅是为了启动javafx程序，可以直接获取场景渲染界面



```
public abstract class UIObject extends Stage{
1,
private static final String RESOURCE_NAME = "/fxml/login/login.fxml";
Parent root=FXMLLoader.load(getClass().getResource(RESOURCE_NAME));

2,
Scene scene = new Scene(root);


3,
scene.setFill(Color.TRANSPARENT);  
setScene(scene);  
initStyle(StageStyle.TRANSPARENT);  
setResizable(false);


4,
obtain();  
initView();  
initEventDefine();

5，
show（）

}


```




# 源码学习


## javafx类型系统


链接；[全面详细的JavaFX国语核心教程（持续更新）_javafx教程-CSDN博客](https://blog.csdn.net/qq_45295475/article/details/125736509)



![[img/Pasted image 20231212092609.png]]

由于javaBeanXXXXProety构造器的访问级别为受保护，如果我們想要创建javaBeanXXXXProety实例，需要通过下面的示例进行

```
JavaBeanBooleanPropertyBuilder javaBeanBooleanPropertyBuilder=JavaBeanBooleanPropertyBuilder.create(); 
//注意：这里由于JavaBeanBooleanPropertyBuilder的构造器也不可直接访问，但其有静态方法create()可以创建，
javaBeanBooleanPropertyBuilder.name("王");  //设置属性名称
javaBeanBooleanPropertyBuilder.bean(new User()); //监听实例 
javaBeanBooleanPropertyBuilder.beanClass(User.class);  //如果想要监听该类下所有实例，可加可不加
JavaBeanBooleanProperty javaBeanBooleanProperty= javaBeanBooleanPropertyBuilder.build();
```



![[img/Pasted image 20231212090925.png]]


![[img/Pasted image 20231212091147.png]]
