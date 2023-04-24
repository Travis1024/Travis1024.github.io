
- nacos服务实例需要处于同一命名空间namespace和同一个分组group，否则gateway routers的lb://调用不到

  TODO：解决分组的问题，后续可能要处理group为同一个模块的问题

- 需要注意knife4j-micro的版本问题（不要使用3.X的版本），各个微服务的子模块需要依赖此模块，与此同时，需要配置：
  
  ```yaml
  # Springfox 使用的路径匹配是基于AntPathMatcher的，而Spring Boot 2.6.X使用的是PathPatternMatcher, 所以需要在配置中修改路径匹配
  
  spring:
    mvc:
      pathmatch:
        matching-strategy: ant_path_matcher
  ```


- dubbo和redis的依赖冲突问题：很烦

  解决方法：使用jedis


- 在静态方法中直接使用注入的bean对象
- mycat的使用

- seata在nacos中的配置文件

- 开启多线程任务之后的bean注入问题：spring为了安全性，禁止线程之间的bean共享



- dubbo 远程访问不到服务器中部署的服务

  - ```shell
    nohup java -jar -DDUBBO_IP_TO_REGISTRY=140.246.171.8 ./filesbottle-ffmpeg-1.0.0-SNAPSHOT.jar --spring.profiles.active=pro > ffmpeg.log &
    ```

  - dubbo.protocol.port = 20880，需要开放服务器的 20880端口

