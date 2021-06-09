# WebSecurityConfig
 
## TLS 설정시 8080 포트를 사용하고 싶을 때

`application.properties`를 통해 TLS 를 설정하고 나면, 인증에 실패한 요청에 대한 redirect를 8080포트가 아닌 8443포트로 옮겨진다. 이 것을 방지하기 위한 설정을 다음과 같이 한다.

```
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        PortMapperImpl portMapper = new PortMapperImpl();
        portMapper.setPortMappings(Collections.singletonMap("8080","8080"));
        PortResolverImpl portResolver = new PortResolverImpl();
        portResolver.setPortMapper(portMapper);
        LoginUrlAuthenticationEntryPoint entryPoint = new LoginUrlAuthenticationEntryPoint(
                "/user/login");
        entryPoint.setPortMapper(portMapper);
        entryPoint.setPortResolver(portResolver);

        http
            .exceptionHandling()
                .authenticationEntryPoint(entryPoint);
    }
}
```

출처: 
- https://stackoverflow.com/questions/60726583/spring-boot-oauth-endpoint-redirects-to-8443
- https://github.com/spring-projects/spring-security/issues/8140
