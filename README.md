## JMeter Test

```shell
alias jmeter="sh /Users/cyper/bin/apache-jmeter-5.6.3/bin/jmeter"
cd spring-security-static-resource
jmeter -n -t jmeter/*.jmx -l result.jtl
```

## Ignoring version(Best performance, not recommended)

```java
@EnableWebSecurity
@Configuration(proxyBeanMethods = false)
public class DefaultSecurityConfig {
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring().requestMatchers("/", "/index.*", "/js/**");
    }


    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests((authorize) ->
                authorize.anyRequest().authenticated()
        );

        http.csrf(AbstractHttpConfigurer::disable);
        return http.build();

    }
}
```

## High performance version

```java
@EnableWebSecurity
@Configuration(proxyBeanMethods = false)
public class DefaultSecurityConfig {
    @Bean
    @Order(0)
    SecurityFilterChain resources(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authorize) -> authorize
                        // 不可使用空字符串,否则直接报错
                        // 静态文件放到 src/main/resources/static 目录即可
                        // 需要配置 WebMvcConfigurer 才能 转发 / 到  /index.html
                        // 这里测试通配符, /index.* 也可以写成 /index.html
                        .requestMatchers("/", "/index.*", "/js/**").permitAll())
                .requestCache(RequestCacheConfigurer::disable)
                .securityContext(AbstractHttpConfigurer::disable)
                .sessionManagement(AbstractHttpConfigurer::disable);

        return http.build();
    }

    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests((authorize) ->
                authorize.anyRequest().authenticated()
        );

        http.csrf(AbstractHttpConfigurer::disable);
        return http.build();

    }
}
```

## Recommended version

```java
@EnableWebSecurity
@Configuration(proxyBeanMethods = false)
public class DefaultSecurityConfig {
    @Bean
    SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests((authorize) -> authorize
                        // 不可使用空字符串,否则直接报错
                        // 静态文件放到 src/main/resources/static 目录即可
                        // 需要配置 WebMvcConfigurer 才能 转发 / 到  /index.html
                        // 这里测试通配符, /index.* 也可以写成 /index.html
                        .requestMatchers("/","/index.*", "/js/**").permitAll()
                        .anyRequest().authenticated()
                );

        http.csrf(AbstractHttpConfigurer::disable);


        return http.build();

    }
}
```

## References
1. https://stackoverflow.com/a/31235990/2497876
2. https://github.com/spring-projects/spring-security/issues/10938