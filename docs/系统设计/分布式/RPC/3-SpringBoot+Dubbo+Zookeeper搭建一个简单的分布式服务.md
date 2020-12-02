# 💪 SpringBoot + Dubbo + Zookeeper 搭建一个简单的分布式服务

---

## 1. 启动Zookeeper 环境

安装好 Zookeeper 后，运行 `zKServer.cmd`

<img src="https://gitee.com/veal98/images/raw/master/img/20201129122149.png" style="zoom:67%;" />

> 🚨 **不要**运行客户端`zkCli.cmd`

## 2. 服务接口 Interface

新建一个文件夹 springboot-dubbo 并用 IDEA  打开。

**File->New->Module…** , 创建 Maven 类型的项目 dubbo-interface

### ① 创建接口类

```java
public interface HelloService {
    public  String sayHello(String name);
}
```

### ② 将项目打成  jar 包供其他项目使用

点击右边的 Maven 然后选择 **install** ，这样 jar 包就打好了：

<img src="https://gitee.com/veal98/images/raw/master/img/20201129104051.png" style="zoom: 67%;" />

## 3. 服务提供者 Provider

New Module，创建一个 SpringBoot 模块 dubbo-provider，添加 Web 依赖：

<img src="https://gitee.com/veal98/images/raw/master/img/20201129104539.png" style="zoom:67%;" />

### ① 导入依赖

```xml
<dependency>
    <groupId>com.smallbeef</groupId>
    <artifactId>dubbo-interface</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>

<!--引入dubbo的依赖-->
<dependency>
    <groupId>com.alibaba.spring.boot</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>

<!-- 引入zookeeper的依赖 -->
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

### ② 配置 Dubbo 相关信息

配置很简单，这主要得益于 springboot 整合 dubbo 专属的`@EnableDubboConfiguration`注解提供的 Dubbo 自动配置。

```properties
# 配置端口
server.port = 8333

# 当前服务/应用名称
spring.dubbo.application.name = dubbo-provider

# 注册中心的地址
spring.dubbo.application.registry = zookeeper://127.0.0.1:2181
```

### ③ 实现接口

注意： **`@Service` 注解使用的时 Dubbo 提供的而不是 Spring 提供的**。

```java
import com.alibaba.dubbo.config.annotation.Service;
import com.smallbeef.service.HelloService;
import org.springframework.stereotype.Component;

@Component
@Service
public class HelloServiceImpl implements HelloService {
    @Override
    public String sayHello(String name) {
        return "Hello" + name;
    }
}
```

### ④ 服务提供者启动类编写

注意：不要忘记**加上 `@EnableDubboConfiguration` 注解开启 Dubbo 的自动配置**。

```java
import com.alibaba.dubbo.spring.boot.annotation.EnableDubboConfiguration;

@SpringBootApplication
@EnableDubboConfiguration // 开启dubbo的自动配置
public class DubboProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboProviderApplication.class, args);
    }

}
```

## 4. 服务消费者 Consumer

同 dubbo-provider，创建一个添加 Web、Dubbo、Zookeeper 依赖的 SpringBoot 模块 dubbo-consumer

<img src="https://gitee.com/veal98/images/raw/master/img/20201129110432.png" style="zoom:67%;" />

### ① 配置 Dubbo 相关信息

```properties
# 配置端口
server.port = 8330

# 当前服务/应用名称
spring.dubbo.application.name = dubbo-consumer

# 注册中心的地址
spring.dubbo.application.registry = zookeeper://127.0.0.1:2181
```

### ② 编写一个简单 Controller 调用远程服务

```java
import com.alibaba.dubbo.config.annotation.Reference;
import com.smallbeef.service.HelloService;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
	
    // @Reference是dubbo的注解，注入分布式的远程服务对象
    @Reference
    private HelloService helloService;

    @RequestMapping("/hello")
    public String hello(){
        String hello = helloService.sayHello("world");
        System.out.println(helloService.sayHello("CS-Wiki"));
        return hello;
    }
}
```

💡：`@Reference` 和 `@Resource` / `@Autowired` 的区别

- `@Resource` 和 `@Autowired` 注入的是本地 Spring 容器中的对象
- `@Reference` 是 Dubbo 的注解，也是注入，他**注入的是分布式的远程服务的对象**，需要 Dubbo 配置使用

### ③ 服务消费者启动类编写

同样的，**加上 `@EnableDubboConfiguration` 注解开启 Dubbo 的自动配置**。

```java
@SpringBootApplication
@EnableDubboConfiguration
public class DubboConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DubboConsumerApplication.class, args);
    }

}
```

## 5. 运行测试

运行 Zookeeper 后，**先运行服务提供者，再运行服务消费者**，浏览器访问 http://localhost:8330/hello 页面返回 Hello world，控制台输出 Hello CS-Wiki。🎉 一个简单的分布式服务实验成功

## 📚 References

- [超详细，新手都能看懂 ！使用SpringBoot+Dubbo 搭建一个简单的分布式服务](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484809&idx=1&sn=a789eba40404e6501d51b24345b28906&source=41#wechat_redirect)