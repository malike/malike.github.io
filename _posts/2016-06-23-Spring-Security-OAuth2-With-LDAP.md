---
layout: post
comments: true
title: Exploring LDAP Integration With Spring's AuthenticationProvider,OAuth2 and MongoDB for a SSO service
categories: [distributed]
author: malike
tags: [documentation,sample]
image:
    path: /posts/exploits_of_a_mom.png
    width: 800
    height: 500
alt: .
redirect_from: "/Spring-Security-OAuth2-With-LDAP/"
---

In [this](http://malike.github.io/Spring-Security-OAuth2/)  post I talked about using Spring Security OAuth2 and MongoDB *(or any database of your choice)*. 
Today we are going explore the [AuthenticationProvider](https://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html) in spring by building LDAP or Active Directory authentication into our SSO microservice which can be used by clients or users.

*What is [LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)?* I'm guessing you already know what it is thats why you got here.

Spring Security supports LDAP authentication [out of the box](http://projects.spring.io/spring-ldap/). As the title suggests we 
are building a custom one using  Authentication Provider interface.

[This](https://docs.spring.io/spring-security/site/docs/3.1.x/reference/core-services.html) explains how Spring Authentication Provider interface works read this. 


For this project I forked the codes from [Spring Security OAuth2 with MongoDB](http://malike.github.io/Spring-Security-OAuth2/).

Adding our LDAP Authentication Provider would require 

**1. LDAP dependencies.**

```xml
	<dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-ldap</artifactId>
    </dependency>
        
    <dependency>
      <groupId>org.springframework.ldap</groupId>
      <artifactId>spring-ldap-core</artifactId>
      <version>2.0.4.RELEASE</version>
    </dependency>
```

**2. Implementing  Auth Provider Interface.**

Spring Security **AuthenticationProvider** can be implemented to create a custom auth provider. 

The most important part of AuthenticationProvider is authenticate. Which 

>"Performs authentication with the same contract as AuthenticationManager.authenticate(Authentication)"


In our custom implementation of authenticate we would use configured [LDAPTemplate](http://docs.spring.io/autorepo/docs/spring-ldap/2.0.x/apidocs/org/springframework/ldap/core/LdapTemplate.html) to validate user's credentials and once authentication is a success we can do anything we want. In my case I check if a user exists before  I persist their details.


**3. Configure.**

To configure our custom AuthenticationProvider we just need to let Spring Integration know our custom provider. By default it would pass all authentication details to all configured AuthenticationProviders for either a successful response or an exception.


With this configuration LDAP users would get OAuth2 tokens just as the users we created in [Spring Security OAuth2 with MongoDB](http://malike.github.io/Spring-Security-OAuth2/). So what we've built here is a single sign on authentication microservice
that works for users in MongoDB and LDAP. In due time we would add social and other custom authentication providers.


**4. Changes To Note**

*LDAP Bean*
```xml
	<bean id="userLDAPAuthenticationProvider" class="st.malike.auth.server.service.security.UserLDAPAuthProviderService" />
```	

*New Authentication Provider*
```xml
	 <sec:authentication-manager  alias="authenticationManager"  >  
	 	 <!--new authentication provider-->  
	        <sec:authentication-provider ref="userLDAPAuthenticationProvider" /> 
	        <sec:authentication-provider ref="userAuthProviderService" />                   
	        <sec:authentication-provider
	            user-service-ref="clientDetailsUserService">
	            <sec:password-encoder ref="passwordEncoder" />
	        </sec:authentication-provider>
	    </sec:authentication-manager>
```
The actual code doing the validation is [here](https://github.com/malike/sso-auth-ldap/blob/master/src/main/java/st/malike/auth/server/service/security/UserLDAPAuthProviderService.java) which can you can update based on how the integration with your single sign on microservice works with the LDAP server


**Finally** check this [LDAP injection](https://www.owasp.org/index.php/LDAP_injection) 



> With [AuthenticationProvider](https://docs.spring.io/spring-security/site/docs/3.2.7.RELEASE/apidocs/org/springframework/security/authentication/AuthenticationProvider.html) we can build custom authentication providers to validate from multiple services once we can 'hook' it in the AuthenticationManager. 
>And keep that in mind not to make the overall authentication expensive because every authentication request would go through all the authentication providers.<br/> 
> FYI source codes on [Github](https://github.com/malike/sso-auth-ldap).




