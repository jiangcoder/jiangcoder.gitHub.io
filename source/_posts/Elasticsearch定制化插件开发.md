title: Elasticsearch定制化插件开发
toc: true
categories: elasticsearch
tags:
  - elasticsearch
thumbnail: /logo/elasticsearch.jpg
---
一，一般的插件开发方式：
    按照官网教程，每次都得打包、替换、重启，这是一个很不方便的过程，固然可以通过testCase来做debug，但是所见即所得的编码习惯，直接上手debug，才是最高效的方式。
   介绍插件开发的博客何其多，个人私以为都没有get到G点，其实深入研究下elasticsearch源码，fix 这个问题并不难，下面希望通过这篇文章帮助到大家。
二，elasticsearch插件的加载机制
①：Node节点启动过程，Elasticsearch.java会调用Bootstrap.java中的init函数。
```java
static void init(...)  {
        ...
        INSTANCE = new Bootstrap();
        INSTANCE.setup(true, environment);
        ...
        INSTANCE.start();
}
```
②：Node节点通过setup方法进行实例化。
```java
private void setup(boolean addShutdownHook, Environment environment) throws BootstrapException {
         
    node = new Node(environment) {
 
     ...
}
```
③：Node.java类中会包含各类的service服务，其中包括PluginsService服务。在实例化PluginsService服务时会传参
```java
environment.pluginsFile(),classpathPlugins等参数。而pluginsFile()即是elasticsearch所指定的plugin目录，elasticsearch会扫描该路径下所有的插件，并加载进来。
   public Node(Environment environment) {
    this(environment, Collections.emptyList());
}
 
protected Node(final Environment environment, Collection<Class<? extends Plugin>> classpathPlugins) {
    ...
    this.pluginsService = new PluginsService(tmpSettings, environment.modulesFile(), environment.pluginsFile(), classpathPlugins);
}
```

④：classpathPlugins参数介绍：
```java
public Node(Environment environment) {
        this(environment, Collections.emptyList());
    }
```
在elasticsearch源码中，这个参数Collection<Class<? extends Plugin>> classpathPlugins一直都是空集合。
没有任何地方注入修改该参数。elasticsearch不但会扫描插件所在路径中的插件，同样也会加载classpathPlugins中所指定的插件，只不过问题是elasticsearch没有给我们提供相应的参数！！！！
三，如何更优雅的开发开发插件
    接上一段小节④，我们只要利用classpathPlugins该参数，就可以在elasticsearch源码环境中进行debug了！！！
    我的实现思路如下，通过继承Node.java，并重写Node类的构造方法，然后在bootstrap中直接实例化该子类，便可以通过elasticsearch直接bug 插件源码了。
   下面贴出我的实现代码，供大家参考：
```java
import org.elasticsearch.Version;
import org.elasticsearch.env.Environment;
import org.elasticsearch.node.Node;
import org.elasticsearch.plugins.Plugin;

import java.util.Collection;

public class EmbeddedNode extends Node {

  private Version version;
  private Collection<Class<? extends Plugin>> plugins;

  public EmbeddedNode(Environment environment, Version version, Collection<Class<? extends Plugin>> classpathPlugins) {
    super(environment,  classpathPlugins);
    this.version = version;
    this.plugins = classpathPlugins;
  }

  public Collection<Class<? extends Plugin>> getPlugins() {
    return plugins;
  }

  public Version getVersion() {
    return version;
  }
}
```

```java    
private void setup(boolean addShutdownHook, Environment environment) throws BootstrapException {
      .....
       //注释Node初始化源码
       /* node = new Node(environment) {
            @Override
            protected void validateNodeBeforeAcceptingRequests(
                final Settings settings,
                final BoundTransportAddress boundTransportAddress, List<BootstrapCheck> checks) throws NodeValidationException {
                BootstrapChecks.check(settings, boundTransportAddress, checks);
            }
        };*/

        Collection plugins = new ArrayList<>();
        Collections.addAll(plugins,   AnalysisIkPlugin.class, HelloPlugin.class, AnalysisMMsegPlugin.class);//, ,AnalysisMMsegPlugin.class
        node = new EmbeddedNode(environment, Version.CURRENT, plugins) {
            @Override
            protected void validateNodeBeforeAcceptingRequests(final Settings settings, final BoundTransportAddress boundTransportAddress, List<BootstrapCheck> checks) throws NodeValidationException {
                BootstrapChecks.check(settings, boundTransportAddress, checks);
            }
        };
    }
```
 
四，部署插件相关的注意事项：
     有关插件开发的详细配置，es插件的种类，在此不再赘述，具体可参考官方文档，更权威，更直接。
下面贴个图，本人在elasticsearch中同时整合了多个插件，以供学习研究时用，直接debug，个人感觉十分不错。
<p><img alt="" src="/logo/plugin.jpg"></p>
 