# How to replace Spring Security with custom user authentication logic?

At work, my project required api call at another server to authenticate user login, which means current server has no information about users. 
However, we still wanted to utilize Spring Security for user authorization and session management. 
The idea is to replace the `authenticate()` in my custom `AuthenticationProvider` class.
The one implemented before was using `DaoAuthenticationProvider`, which queries DB for user information and has intimate relationship with `UserDetailsService`.

### **ApiBasedAuthenticationProvider.java**

```
import com.ktnet.cbec.sample.user.domain.dao.UserDao;
import com.ktnet.cbec.sample.user.entity.UserPrincipal;

import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;

public class ApiBasedAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = authentication.getCredentials().toString();
        
        ... API CALLING LOGIC

        UserPrincipal userPrincipal = new UserPrincipal(userDao);
        
        return new UsernamePasswordAuthenticationToken(userPrincipal, password, userPrincipal.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```
- `UserPrincipal` class extends `org.springframework.security.core.userdetails.User`

### SecurityConfig.java
```
@Configuration
class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        AuthenticationProvider provider = new ApiBasedAuthenticationProvider();
        auth.authenticationProvider(provider);
    }
}
```
- assign `AuthenticationManagerBuilder` to use the custom provider

- reference: https://stackoverflow.com/questions/65708171/how-to-create-a-custom-authentication-filter-in-spring-security/65708311#65708311?newreg=474c7c431a7c469c9419e4638a62771a
