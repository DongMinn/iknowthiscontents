# Spring-Security 알게된 내용 정리



###1. Spring Security  란

- 스프링 시큐리티가 무엇인지는.. 구글링 하면 더 자세하게 나오니까 생략...



### 2. 회사 공용 어드민 인증을 만들면서 더 자세히 알게 된 내용?

```java
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .regexMatcher("^/api/.*|/admin/.*|/|/logout|") // /api/*
                .securityContext()
                .and()
                .csrf().disable()
                .exceptionHandling()
                .authenticationEntryPoint(redirectAuthenticationEntryPoint)
                .accessDeniedHandler(new JsonAccessDeniedHandler())
                .and()
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/")
                .addLogoutHandler(new AuthenticationServerLogoutHandler(authServerAdaptor))
                .deleteCookies(cookieName)
                .invalidateHttpSession(true)
                .clearAuthentication(true)
                .and()
                .addFilterAt(cookieAuthenticationFilter, AbstractPreAuthenticatedProcessingFilter.class)
                .authenticationProvider(new AccessTokenAuthenticationProvider(authServerAdaptor))
                .authorizeRequests()
                .anyRequest()
                .authenticated()
                .accessDecisionManager(accessDecisionManager());
    }
```

- 이 코드는  ```WebSecurityConfigurerAdapter``` 를 상속받은 커스텀 config 파일에 있는 configure 메소드 이다.
- 보면 알겠지만.. regexMatcher에 해당하는 호출만... 아래 addFilterAt에 추가해준 filter에서 작업?을 수행하게 만든다. 
  (해당하지 않는 api  도 filter에 걸리지만.. 시큐리티 인증작업(AuthenticationProvider)은 하지 않는다. )
- 진행의 흐름을 나열하자면... regexMatcher 에 해당하는 api 호출이 들어오면.. filter에 들어가고.. filter에서 전처리 작업/(후처리 작업) 을 한 후... AuthenticationProvider에서 인증 작업을 한 후.. 그 결과를(Authentication) 리턴 해 주게 된다. 
  이 리턴된 Authentication  은 ```SecurityContextHolder.getContext().setAuthentication(authentication);```  을 통해서 SecurityContextHolder 에 저장되게 되고.. 이 걸 이용해서 다른 곳에서 사용 할 수 있게 된다. 
- filter 전처리 작업 후 ```SecurityContextHolder.getContext().setAuthentication(authentication);``` 를 
  해주면    
  -> AuthenticationProvider 에서 인증 
  -> filter 후처리 작업 
  이 전처리 후처리를 나누는 기준은 ```chain.doFilter(request, response);```  이 코드 전이 전처리 후가 후처리가 된다.

```java
@Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest servletRequest = ((HttpServletRequest) request);
        Optional<Cookie> cookieOptional = getAuthCookie(servletRequest);

        if (!cookieOptional.isPresent() || requiresAuthentication(cookieOptional.get())) {
            Authentication authentication = cookieOptional
                    .map(cookie -> new AccessToken(cookie.getValue(), getClientIPAddress(servletRequest)))
                    .orElse(null);

            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        //전처리 
        chain.doFilter(request, response);
        //후처리 
    }
```





 







