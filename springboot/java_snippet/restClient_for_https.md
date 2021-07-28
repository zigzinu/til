# How to call a **https** url when using `RestTemplate` Object?

1. Create `RestTemplate` bean with specific configurations.

```java
import java.security.KeyManagementException;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.security.cert.X509Certificate;

import javax.net.ssl.SSLContext;

import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.TrustStrategy;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
		    SpringApplication.run(MyApplication.class, args);
	  }

    @Bean
    RestTemplate getRestTemplate() throws KeyStoreException, NoSuchAlgorithmException, KeyManagementException {
		TrustStrategy acceptingTrustStrategy = (X509Certificate[] chain, String authType) -> true;

		SSLContext sslContext = org.apache.http.ssl.SSLContexts.custom()
						.loadTrustMaterial(null, acceptingTrustStrategy)
						.build();
	
		SSLConnectionSocketFactory csf = new SSLConnectionSocketFactory(sslContext
				,NoopHostnameVerifier.INSTANCE);
	
		CloseableHttpClient httpClient = HttpClients.custom()
						.setSSLSocketFactory(csf)
						.build();
	
		HttpComponentsClientHttpRequestFactory requestFactory =
						new HttpComponentsClientHttpRequestFactory();
	
		requestFactory.setHttpClient(httpClient);
		RestTemplate restTemplate = new RestTemplate(requestFactory);
		return restTemplate;
    }
}

```

2. Write your code with injected dependency.

```java
import java.net.URI;
import java.nio.charset.Charset;

import com.fasterxml.jackson.databind.JsonNode;
import com.example.request.MyDto;

import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriComponentsBuilder;

import lombok.RequiredArgsConstructor;

@RestController
@RequestMapping("/test") 
@RequiredArgsConstructor
public class TestController {

    private final RestTemplate restTemplate;

    @PostMapping("/api/orderProd")
    public JsonNode OrderProdInsertApi(@RequestBody MyDto dto) throws Exception{
        
        // URL
        URI targetUrl= UriComponentsBuilder
                .fromUriString("https://localhost:8443/test/db/orderProd")                             
                .build().encode().toUri();

        // header
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(new MediaType("application","json", Charset.forName("UTF-8"))); 
    
        // body
        HttpEntity<MyDto> entity = new HttpEntity<>(dto, headers);

        // http call
        ResponseEntity<JsonNode> response = this.restTemplate.exchange(targetUrl
                , HttpMethod.POST
                , entity
                , JsonNode.class);
        
        // response
        JsonNode responseBody = response.getBody();

        return responseBody;
    }
}
```

## Possible Error 1:

```
Resolved [org.springframework.web.client.ResourceAccessException: I/O error on POST request for "https://localhost:8443/test/db/orderProd": PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target; nested exception is javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target]
```

### CAUSE:
1. Https url is hosted by server with private (not trusted) cacert.
2. You created `new RestTemplate()` with no specific configuration at all. 

## Possible Error 2:

```
Resolved [org.springframework.web.client.ResourceAccessException: I/O error on POST request for "https://localhost:8443/test/db/orderProd": Certificate for <localhost> doesn't match any of the subject alternative names: []; nested exception is javax.net.ssl.SSLPeerUnverifiedException: Certificate for <localhost> doesn't match any of the subject alternative names: []]
```

### FIX:

Create `SSLConnectionSocketFactory` instance with `NoopHostnameVerifier.INSTANCE` argument.

```java
SSLConnectionSocketFactory csf = new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE)
```

References
- https://stackoverflow.com/questions/39762760/javax-net-ssl-sslexception-certificate-doesnt-match-any-of-the-subject-alterna
- https://pythonq.com/so/java/405472
