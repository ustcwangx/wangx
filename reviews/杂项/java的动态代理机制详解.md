### 				java的动态代理机制详解

​	在java的动态代理机制中，有两个重要的类或接口，一个是InvocationHandler(Interface)、另一个则是Proxy(Class)，这一个类和接口是实现我们动态代理所必须用到的。

​	每一个动态代理类都必须要实现InvocationHandler这个接口，