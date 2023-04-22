## Spring Security

### Filters

* AuthorizationFilter
    * An authorization filter that restricts access to the URL using AuthorizationManager.
* ExceptionTranslationFilter
    * Handles any AccessDeniedException and AuthenticationException thrown within the filter chain.
* UsernamePasswordAuthenticationFilter
    * Processes an authentication form submission. Called AuthenticationProcessingFilter prior to Spring Security 3.0.

###

* Create a class extends UsernamePasswordAuthenticationFilter
    * public abstract Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
      throws AuthenticationException, IOException, ServletException;
    * protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain
      chain, Authentication authResult) throws IOException, ServletException;
    * protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
      AuthenticationException failed) throws IOException, ServletException;
* UserDetailsService
    * Spring Security provides some configuration helpers to quickly get common authentication manager features set up
      in your application.
    * The most commonly used helper is the AuthenticationManagerBuilder, which is great for setting up in-memory, JDBC,
      or LDAP user details or for adding a custom UserDetailsService.
  ```
    UserDetails user = User.withUsername(customer.getEmail()).password(customer.getPassword()).authorities("USER").build();
  ```
*

###

* DelegatingFilterProxy
    * public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws
      ServletException, IOException
        * protected Filter initDelegate(WebApplicationContext wac) throws ServletException
* FilterChainProxy
    * private List<Filter> getFilters(HttpServletRequest request)

###  

* Create a config class extends WebSecurityConfigurerAdapter

## Self-defined login page

https://www.youtube.com/watch?v=UWkOQZzAumw&list=PLmOn9nNkQxJEnBjOMEXPLRjoTRNvhHIl2&index=11

###  

* hasAuthority, hasAnyAuthority, hasRole, hasAnyRole
  https://www.youtube.com/watch?v=IL9AU_mKm84&list=PLmOn9nNkQxJEnBjOMEXPLRjoTRNvhHIl2&index=12

###

* @EnableGlobalMethodSecurity
* @Secured, @PreAuthorize, @PostAuthorize, @PreFilter, @PostFilter

### Logout

###

## References

* [Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture/)
* 