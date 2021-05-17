# jasypt 라이브러리를 활용하여 민감정보 암호화하기

jasypt 를 사용하기 위한 방법은 여러개 있따~~~!~!@~!@

`application.properties` 파일에서 비밀번호, IP 등을 가리기 위한 설정이다.

1. build.gradle

```
dependencies {
    implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.3'
}
```

2. JasyptConfig 클래스

```
import org.jasypt.encryption.StringEncryptor;
import org.jasypt.encryption.pbe.PooledPBEStringEncryptor;
import org.jasypt.encryption.pbe.config.SimpleStringPBEConfig;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
// @EncryptablePropertySource("application.properties")
public class JasyptConfig {

    @Value("${jasypt.encryptor.password}")
    private String jasyptPassword;

    @Bean("jasyptStringEncryptor")
    public StringEncryptor stringEncryptor() {
        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword(jasyptPassword);
        config.setAlgorithm("PBEWITHHMACSHA512ANDAES_256");
        config.setKeyObtentionIterations("1000");
        config.setPoolSize("1");
        config.setProviderName("SunJCE");
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
        config.setIvGeneratorClassName("org.jasypt.iv.RandomIvGenerator");
        config.setStringOutputType("base64");
        encryptor.setConfig(config);
        return encryptor;
    }
}
```

- `@SpringBootApplication` 처럼 `@EnableAutoConfiguration` 태그가 포함된 프로젝트는 기본적으로 `lazyjasyptStringEncryptor` 빈이 기본으로 생성된다.
- 기본 빈을 application.properties 의 값들로 커스터마이징할 수도 있지만, 새로운 빈을 생성하여 사용하도록 클래스를 선언한다.
- 여기서 password 설정도 application.properties 에서 읽어오도록 하고 보안을 위해 환경변수로 입력받도록 한다.

3. application.properties

```
jasypt.encryptor.password=${JASYPT_ENCRYPTOR_PASSWORD:NULLPASSWORD}
```

4. 암호화 코드

jasypt 라이브러리로 복호화하기 전에 암호화를 먼저해야하기 떄문에 TestController 를 선언하여 실행 가능한 코드를 작성한다.

```
import org.jasypt.encryption.StringEncryptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import springfox.documentation.annotations.ApiIgnore;

@RestController 
@RequestMapping("/test") 
@ApiIgnore
public class TestController {

    @Autowired
    @Qualifier("jasyptStringEncryptor") // 이거 안쓰면 lazyJasyptStringEncryptor 랑 중복되서 스프링부트 실행이 안됨. found 2 beans
    private StringEncryptor encryptor;

    @Value("${jasypt.string.to.encrypt}")
    private String jasyptStringToEncrypt;

    @GetMapping("/encryptAndPrint")
    public void encryptAndPrint() throws Exception{

        String encryptedText = encryptor.encrypt(jasyptStringToEncrypt);
        System.out.println("encryptedText: " + encryptedText); //암호화된 값 
        
        String plainText = encryptor.decrypt(encryptedText); 
        String printText = plainText.substring(0,4) + plainText.substring(5).replaceAll(".", "*");
        System.out.println("plainText: " + printText); //복호화된 값

    }

}
```

- `@Qualifier`: JasyptConfig.java 에서 커스텀으로 생성한 Encryptor 빈을 선택해주기 위해 빈 이름을 직접 적는다.
- 민감 정보를 http 통신이나 로그에 남지 않게하기 위해 환경변수로 값을 입력 받고, 서버 로그를 통해 암호화한 결과를 확인한다.

5. 암호화된 properties

application.properties 파일에서 암호화된 값을 `ENC(암호화된 값)` 처럼 선언한다.

```
server.ssl.key-store-password=ENC(FCg4lMn2Nb5P8pGEjzBOdnUZvS4EAOdRYiaP5qGTzGMchZyfHKK/pB8H+1U4GpP1)
mariadb.ip=ENC(1wFKXQ88u+FGKLZ+z9YHvhF1h7l0gpRa47wXl2EBeFsYKSm2586ixIEJpTXVpOKO)
```

6. 스프링부트 시작

출처: https://github.com/ulisesbocchio/jasypt-spring-boot
