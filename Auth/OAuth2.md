# OAuth 2.0

## 1. OAuth2 Foundation[1]

* Back channel(highly secure)
    * Back to redirect URL with authorization code
    * Backend exchange authorization code for access token
    *
* Front channel(less secure)
* Others
    * Scope: {$scope1} {$scope2}
    * Consent

* OAuth 2.0 flows
    * Authorization code(front channel + back channel)
    * Implicit flow(front channel only)
        * Instead of getting auth code. get the token directly

    * Resource owner password credentials(back channel only)
    * Client credentials(back channel only)

* OpenId Connect
    1. ID token
    2. UserInfo endpoint for getting more user information
    3. Standard set of scopes
    4. Standardized implementation
    5. OpenId Connect authorization code flow
        * Scope: openId profile
        * Like Authorization code flow, exchange auth code for access token and ID token.

* OAuth and OpenId Connect
    * User OAuth 2.0 for Authorization
        * Granting access to your API
        * Getting access to user data in other systems

    * Use OpenId Connect for Authentication
        * Logging the user in
        * Making your accounts available in other systems

### 2. JWS + JWK in a Spring Security OAuth2[2]

#### 2-1. Understanding the Big Picture of JWS and JWK

1. `Client` request `Authorization Servicer` for Access Token. `Authorization Servicer` return JWT(JWS) to `Client`.
2. `Client` request `Resource Server` for resource.
3. `Resource Server` request public keys from `Authorization Servicer`. `Authorization Servicer` return JWK
   to `Resource Server`.
4. `Resource Server` verify Signature and permissions, then return resource to `Client`.

#### 2-2. Configuring the Resource Server(using Spring Security OAuth's Autoconfig features)

* Add the `spring-security-oauth2-autoconfigure` dependency to Resource Server
* Use `@EnableResourceServer` annotation to enable the Resource Server features.
* Configure the JWK Set endpoint of a local Authorization Server
  ```
    security.oauth2.resource.jwk.key-set-uri=http://localhost:8081/sso-auth-server/.well-known/jwks.json
  ```
    * The property translates in the creation of a couple of Spring beans:
        1. a `JwkTokenStore` with the only ability to decode a JWT and verifying its signature
        2. a `DefaultTokenServices` instance to use the former TokenStore

#### 2-3. The JWK Set Endpoint in the Authorization Server

* Note: Since Spring Security doesn't yet offer features to set up an Authorization Server, creating one using Spring
  Security OAuth capabilities is the only option at this stage. It will be compatible with Spring Security Resource
  Server, though.
* Add the `spring-security-oauth2-autoconfigure` dependency to Authorization Server
* Use the `@EnableAuthorizationServer` annotation to configure the OAuth2 Authorization Server mechanisms
* Register an OAuth 2.0 Client using properties
  ```
    security.oauth2.client.client-id=bael-client
    security.oauth2.client.client-secret=bael-secret
  ```
    * With this, our application will retrieve random tokens when requested with the corresponding credentials:
    ```
      curl bael-client:bael-secret\
        @localhost:8081/sso-auth-server/oauth/token \
        -d grant_type=client_credentials \
        -d scope=any
    ```
    * Spring Security
      OAuth `retrieves a random string value by default, not JWT-encoded`: ```"access_token": "af611028-643f-4477-9319-b5aa8dc9408f"```

* Issuing JWTs
    * We can easily change this by creating a `JwtAccessTokenConverter` bean, and using it in a `JwtTokenStore` instance
    * With these changes, when client request a new access token, Auth Server will return a JWT, encoded as a JWS.
    * We can easily identify JWSs; their structure consists of three fields (header, payload, and signature) separated
      by a dot.
      ```
        "access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
          .
          eyJzY29wZSI6WyJhbnkiXSwiZXhwIjoxNTYxOTcy...
          .
          XKH70VUHeafHLaUPVXZI9E9pbFxrJ35PqBvrymxtvGI"
      ```
        * Note By default, Spring signs the header and payload using a Message Authentication Code (MAC) approach.

* Creating a Keystore File
    * By default, Spring signs the header and payload using a Message Authentication Code (MAC) approach(symmetric).
        * Customize the signing key value when we configure the JwtAccessTokenConverter
          bean: ```converter.setSigningKey("bael");```
        * Note: even if we don't publish the signing key, setting up a weak signing key is a potential threat to
          dictionary attacks.

    * If we're going to share keys, it'll be better if we use asymmetric cryptography (particularly, digital signature
      algorithms) to sign the tokens.
    * open the command line in the /bin directory of any JDK or JRE you have in handy: ```cd $JAVA_HOME/bin```
    * run the keytool command, with the corresponding parameters:
      ```
        ./keytool -genkeypair \
          -alias bael-oauth-jwt \
          -keyalg RSA \
          -keypass bael-pass \
          -keystore bael-jwt.jks \
          -storepass bael-pass
      ```
* Adding the Keystore File to Our Application
  ```xml
    <build>
      <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>src/main/resources/filtered</directory>
            <filtering>true</filtering>
        </resource>
      </resources>
    </build>
  ```
* Configuring the TokenStore
    * Configuring our TokenStore with the pair of keys; the private to sign the tokens, and the public to validate the
      integrity.
      ```
        ClassPathResource ksFile = new ClassPathResource("bael-jwt.jks");
        KeyStoreKeyFactory ksFactory = new KeyStoreKeyFactory(ksFile, "bael-pass".toCharArray());
        KeyPair keyPair = ksFactory.getKeyPair("bael-oauth-jwt");
      
        // configure it in our JwtAccessTokenConverter bean:
        converter.setKeyPair(keyPair);
      ```
* The JWK Set Endpoint
    * Add `nimbus-jose-jwt` dependency, which provides some basic JWK implementations, to Auth Server.
    * Creating the JWK Set Endpoint
        * Creating a JWKSet bean using the KeyPair instance we configured
          ```
            @Bean
            public JWKSet jwkSet() {
                RSAKey.Builder builder = new RSAKey.Builder((RSAPublicKey) keyPair().getPublic())
                   .keyUse(KeyUse.SIGNATURE)
                   .algorithm(JWSAlgorithm.RS256)
                   .keyID("bael-key-id");
                   return new JWKSet(builder.build());
            }
          ```
        * Create the endpoint
          ```
            @RestController
            public class JwkSetRestController {
              @Autowired
              private JWKSet jwkSet;
    
              @GetMapping("/.well-known/jwks.json")
              public Map<String, Object> keys() {
                return this.jwkSet.toJSONObject();
              }
            }
          ```
            * The Key Id field we configured in the JWKSet instance translates into the kid parameter.
            * This kid is an arbitrary alias for the key, and it's usually used by the Resource Server to select the
              correct entry from the collection since the same key should be included in the JWT Header.

* Adding the kid Value to the JWT Header
    * Since Spring Security OAuth doesn't support JWK, the issued JWTs won't include the kid Header.
    * We'll create a new class extending the JwtAccessTokenConverter we've been using, and that allows adding header
      entries to the JWTs:
        * configure the parent class as we've been doing, setting up the KeyPair we configured
        * obtain a Signer object that uses the private key from the keystore
        * of course, a collection of custom headers we want to add to the structure
      ```
          public class JwtCustomHeadersAccessTokenConverter extends JwtAccessTokenConverter {
            private Map<String, String> customHeaders = new HashMap<>();
            final RsaSigner signer;
  
            public JwtCustomHeadersAccessTokenConverter(Map<String, String> customHeaders, KeyPair keyPair) {
              super();
              super.setKeyPair(keyPair);
              this.signer = new RsaSigner((RSAPrivateKey) keyPair.getPrivate());
              this.customHeaders = customHeaders;
            }
          }
      ```
    * Override the encode method.
        * Our implementation will be the same as the parent one, with the only difference that we'll also pass the
          custom headers when creating the String token
      ```
          private JsonParser objectMapper = JsonParserFactory.create();
  
          @Override
          protected String encode(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
            String content;
            try {
              content = this.objectMapper
              .formatMap(getAccessTokenConverter()
              .convertAccessToken(accessToken, authentication));
            } catch (Exception ex) {
              throw new IllegalStateException("Cannot convert access token to JSON", ex);
            }
            String token = JwtHelper.encode(content,this.signer,this.customHeaders).getEncoded();
            return token;
          }
      ```
        * use this class now when creating the JwtAccessTokenConverter bean:
      ```
          @Bean
          public JwtAccessTokenConverter accessTokenConverter() {
                  Map<String, String> customHeaders = Collections.singletonMap("kid", "bael-key-id");
                  return new  JwtCustomHeadersAccessTokenConverter(customHeaders,keyPair());
          }
      ```

## References

* [1][OAuth 2.0 and OpenID Connect (in plain English)](https://youtu.be/996OiexHze0)
* [2][JWS + JWK in a Spring Security OAuth2 Application](https://www.baeldung.com/spring-security-oauth2-jws-jwk)




