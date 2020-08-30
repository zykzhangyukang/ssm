## ssm与shiro 整合

#### 1. 添加依赖

```xml
     <!--   shiro的依赖     -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>${shiro-spring-version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>${shiro-ehcache-version}</version>
        </dependency>
```


#### 2. 添加shiro的拦截器 (web.xml)

```xml
  <filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    <init-param>
      <param-name>targetFilterLifecycle</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <servlet-name>web-dispatcher</servlet-name>
  </filter-mapping>
```

#### 3. 配置shiro和spring整合

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 缓存管理器 使用Ehcache实现 -->
    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
    </bean>

    <!-- 凭证匹配器 -->
    <bean id="credentialsMatcher" class="com.coderman.common.shiro.credentials.RetryLimitHashedCredentialsMatcher">
        <constructor-arg ref="cacheManager"/>
        <property name="hashAlgorithmName" value="md5"/>
        <property name="hashIterations" value="2"/>
        <property name="storedCredentialsHexEncoded" value="true"/>
    </bean>

    <!--自定义realm-->
    <bean id="userRealm" class="com.coderman.common.shiro.realm.UserRealm">
        <property name="credentialsMatcher" ref="credentialsMatcher"/>
        <property name="cachingEnabled" value="true"/>
        <property name="authenticationCachingEnabled" value="true"/>
        <property name="authenticationCacheName" value="authenticationCache"/>
        <property name="authorizationCachingEnabled" value="true"/>
        <property name="authorizationCacheName" value="authorizationCache"/>
    </bean>

    <!-- 配置 shiro 的核心组件：securityManager -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="userRealm"/>
    </bean>

    <!-- 配置shiro的一些拦截规则，id必须和web.xml中的 shiro 拦截器名一致 -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- Shiro的核心安全接口,这个属性是必须的 -->
        <property name="securityManager" ref="securityManager"/>
        <!-- 身份认证失败，则跳转到登录页面的配置 -->
        <property name="loginUrl" value="/loginPage.do"/>
        <!-- 登录成功后的页面 -->
        <property name="successUrl" value="/mainPage.do"/>
        <!-- 权限认证失败，则跳转到指定页面 -->
        <property name="unauthorizedUrl" value="/unauthorized"/>  <!-- 登录后访问没有权限的页面后跳转的页面 -->
        <!-- Shiro连接约束配置,即过滤链的定义 -->
        <property name="filterChainDefinitions">
            <value>
                <!-- 注意：规则是有顺序的，从上到下，拦截范围必须是从小到大的 -->
                <!--  url = 拦截规则（anon为匿名，authc为要登录后，才能访问，logout登出过滤） -->
                /loginPage.do = anon
                /user/login.do = anon
                /user/logout.do = logout
                /** = authc
            </value>
        </property>
    </bean>
</beans>

```

#### 4. 添加ehcache配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache name="es">

    <diskStore path="java.io.tmpdir"/>

    <!-- 登录记录缓存 锁定1分钟 -->
    <cache name="passwordRetryCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true">
    </cache>

</ehcache>

```

#### 5. RBAC系统搭建
