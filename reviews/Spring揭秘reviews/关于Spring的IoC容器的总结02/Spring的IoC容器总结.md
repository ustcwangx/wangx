## 			Spring 的IoC容器总结

### 1 IoC容器背后的秘密

![Snipaste_2018-01-31_20-35-24](C:\Users\wangxin\Desktop\pics\Snipaste_2018-01-31_20-35-24.png)

​	Spring的IoC容器所起的作用，就像上图所展示的那样，它会以某种方式加载Configuration Metadata(通常也就是XML格式的配置信息)，然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。

​	Spring的IoC容器实现以上功能的过程，基本上可以按照类似的流程划分为两个阶段，即容器启动阶段和Bean实例化阶段。

* **容器启动阶段**

  ​	容器启动伊始，首先会通过某种途径加载Configuration MetaData，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载的Configuration MetaData