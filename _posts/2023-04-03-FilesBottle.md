
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