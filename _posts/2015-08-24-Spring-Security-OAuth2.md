---
layout: post
comments: true
title: Using Spring Security OAuth 2.0 and MongoDB to secure a Microservice/SOA  System
author: malike
categories: [distributed]
tags: [documentation,sample]
image:
 path: /posts/security_question.png
 width: 800
 height: 500
alt: .
---


Before we go straight to the *how-to* and codes. I'd like to take a minute to explain
my choice in using [Spring Security OAuth2.0](http://projects.spring.io/spring-security-oauth/docs/oauth2.html) and [MongoDB](https://www.mongodb.org) to develop a ***Single Sign On Authentication Server***.

With every microservice architecture,there would obviously different services working together.
Unlike the macro or fat -war or fat -jar architecture, managing authentication and authorization is not the same.
Because assuming you have two microservice working together to delivery a service. It is would be a bad idea to replicate
authentication and authorization for both microservices independently.
Now that we are both on the same page that replicating authentication and authorization for all microservices is *really ..erm.*

We both know we need to have a centralized authentication system  where a user authenticates and
is authorized on one system, her privileges are recognized throughout all other microservices in the architecture.

Let's talk a bit about **RESTful services**, **Stateless Sessions** and **Stateful Sessions**.

To access any content on a server **3** things are required to be done

1. **Authentication** - *Verify user identity*
2. **Authorization** - *Is she supposed to have access to that?*
3. **Session Management** - *I know who you are and you have access for this time*


In any RESTful system, Number 3. is not encouraged. *Why?* It would mean that to get things working we would need to authenticate the user for every RESTful request.
This is not good. Assuming I have a huge user database it would be worse.

The best next fit is to give the user a token once authenticated and this can be used for subsequent requests.
Token would be managed separately from the users. Expired tokens deleted(To keep token storage small). This means that on a less busy day we would have just **1** token stored and **(user size ) == token size** on our busiest day. Compared to having
same  **(user size)** for the busiest and less busy days,this is much better.

But in **THEORY**, RESTful systems should be **STATELESS**.

**What's Stateless and Stateful Sessions?**

**STATELESS SESSIONS**

*“In computing, a stateless protocol is a communications protocol that treats each request as an independent transaction that is unrelated to any previous request so that the communication consists of independent pairs of request and response. A stateless protocol does not require the server to retain session information or status about each communications partner for the duration of multiple requests. In contrast, a protocol which requires keeping of the internal state on the server is known as a stateful protocol.”*


**STATEFUL SESSIONS**

Exact opposite of Stateless.



But practically it would be impossible to use a *"pure"* stateless session management for RESTful services by the RESTful standard without having performance issues.

For a frontend applications(*eg: AngularJS or Polymer*) once it does not keep tokens in cookies,to prevent [Cross Site Request Forgery [CSRF or XSRF]](https://en.wikipedia.org/wiki/Cross-site_request_forgery).
And with the rise of  *"MOBILE FIRST"* systems,this would be a problem because mobile apps don't play well with **COOKIE**s.

![_config.yml](/posts/tumblr_nhvyfbefec1qlwkdio1_500-1431380291.gif)


For backend application storing the token would be quite easy. Since there are lots of options for this.
[MongoDB](https://www.mongodb.org) and [Redis](https://redis.io) are very
good candidates.

If the cookie approach is not **"ok"** for frontend applications how do  we store tokens *then?*

[HTML5](http://www.w3schools.com/html/html5_webstorage.asp) comes with ***localStorage*** and ***sessionStorage***.

*"With local storage, web applications can store data locally within the user's browser.
Before HTML5, application data had to be stored in cookies, included in every server request. Local storage is more secure, and large amounts of data can be stored locally, without affecting website performance.
Unlike cookies, the storage limit is far larger (at least 5MB) and information is never transferred to the server.
Local storage is per origin (per domain and protocol). All pages, from one origin, can store and access the same data."*

The main difference between the two is...

*"The sessionStorage object is equal to the localStorage object, except that it stores the data for only one session"*

Although SessionStorage is not supported in many browsers. You can get ***localStorage*** working as ***sessionStorage*** with some extra js codes or just use polyfill to provide this functionality in unsupported browsers.

You can then pick tokens from the ***sessionStorage*** (or ***localStorage*** ) and send as HTTP headers.


Now that we have clear idea on how our ***Single Sign On Authentication Server*** should work,both backend and frontend(and a little about mobile) considered let me introduce you to [OAuth2.0](https://tools.ietf.org/html/rfc6749).

Whats ***[OAuth2.0](https://tools.ietf.org/html/rfc6749)?***

*"The OAuth 2.0 authorization framework enables a third-party
 application to obtain limited access to an HTTP service, either on
 behalf of a resource owner by orchestrating an approval interaction
 between the resource owner and the HTTP service, or by allowing the
 third-party application to obtain access on its own behalf."*

Fortunately it supports what we want our authentication server should achieve (plus more features)
A quick search online would let you know [Google](https://developers.google.com/identity/protocols/OAuth2),[Facebook](https://developers.facebook.com/docs/facebook-login/access-tokens),[Windows Live](https://msdn.microsoft.com/en-us/library/hh243647.aspx),[GitHub](https://developer.github.com/v3/oauth),[Slack](https://api.slack.com/docs/oauth),[Dropbox](https://www.dropbox.com/developers/reference/oauthguide) and [many more](http://oauth.net/2/) are already using it.<br>
They using it doesn't make it *perfect*,it just means we are using them as guinea pigs. :D.


***Short Summary about [OAuth 2.0](http://oauth.net/2/)***

*"The first step of OAuth 2 is to get authorization from the user. For browser-based or mobile apps, this is usually accomplished by displaying an interface provided by the service to the user*.

*OAuth 2 provides several "grant types" for different use cases. The grant types defined are:*

  1. ***Authorization Code*** for apps running on a web server
  2. ***Implicit*** for browser-based or mobile apps
  3. ***Password*** for logging in with a username and password
  4. ***Client credentials*** for application access

With these grant types we can authenticate and authorize users as well clients(which is microservice in our case).

<br>
*I have to add this disclaimer here. For every framework,new technology whatevs I have/want to use.  [Spring](http://spring.io/) is my first go to client or  integration.I mean it's [Spring](http://spring.io/).*
<br>

**Why [Spring Security OAuth2.0](http://projects.spring.io/spring-security-oauth/docs/oauth2.html) ?**

Spring already has integration and support for LDAP(easy integration if your ***Single Sign On Authentication Server*** is an enterprise solution),Social platforms and Databases.

The real question is

***WHY NOT [Spring Security OAuth2.0](http://projects.spring.io/spring-security-oauth/docs/oauth2.html)?***.

<br>

**Issues with [Spring Security OAuth2.0](http://projects.spring.io/spring-security-oauth/docs/oauth2.html)**

Spring Security OAuth 2.0 supports storing tokens in MySQL out of the box.
With the abundance of NoSQL databases which Spring already supports it would be a [better option to integrate with one of them out of the box](https://spring.io/understanding/NoSQL).

But let me mention here that Spring Security OAuth 2.0 supports InMemoryTokenStore and JWT as well.

*"The default InMemoryTokenStore is perfectly fine for a single server (i.e. low traffic and no hot swap to a backup server in the case of failure). Most projects can start here, and maybe operate this way in development mode, to make it easy to start a server with no dependencies.*

*The JdbcTokenStore is the JDBC version of the same thing, which stores token data in a relational database. Use the JDBC version if you can share a database between servers, either scaled up instances of the same server if there is only one, or the Authorization and Resources Servers if there are multiple components. To use the JdbcTokenStore you need "spring-jdbc" on the classpath.*

*The JSON Web Token (JWT) version of the store encodes all the data about the grant into the token itself (so no back end store at all which is a significant advantage). One disadvantage is that you can't easily revoke an access token, so they normally are granted with short expiry and the revocation is handled at the refresh token. Another disadvantage is that the tokens can get quite large if you are storing a lot of user credential information in them. The JwtTokenStore is not really a "store" in the sense that it doesn't persist any data, but it plays the same role of translating between token values and authentication information in the DefaultTokenServices."*

But the good thing is you can implement the TokenStore class and persist your token in any database of your choice.

I chose MongoDB. I need to mention that were some issues with the MongoDB converter using this approach but it finally worked  :)


**Update**
---------------------------------------------------------------------------

*There are some [issues](http://stackoverflow.com/questions/36809721/spring-boot-resource-server-not-able-to-authorize-roles-with-oauth-2-access-toke) with the project,if you use it as it is you would have problems with the [antMatchers](http://docs.spring.io/autorepo/docs/spring-security/3.2.3.RELEASE/apidocs/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)*
*They do not work when you have custom urls to be permitted. For example :*
*If I wanted this url **[http://localhost:8080/donotauthenticate]()** to be permitted with code below*

```java
 http
  .authorizeRequests()
  .antMatchers("/donotauthenticate").permitAll()
  .antMatchers("/**").authenticated()
```

*You'll see it doesn't work when you using this, which I did in my case*

```xml
  <dependency>
	 <groupId>org.springframework.security.oauth</groupId>
	 <artifactId>spring-security-oauth2</artifactId>
	 <version>2.0.1.RELEASE</version>
  </dependency>
```

*Changing it to this version fixed the issue.*

```xml
  <dependency>
 	<groupId>org.springframework.security.oauth</groupId>
	<artifactId>spring-security-oauth2</artifactId>
	<version>2.0.7.RELEASE</version>
  </dependency>
```

*I've not looked into **exactly** what the cause is to file an issue. Anyone who does before me can comment*


> Now that we done talking about our choices. [Lets move on to coding this.](https://github.com/malike/sso-auth)<br>


<br>
<br>
**References**

[https://speakerdeck.com/dsyer/security-for-microservices-with-spring](https://speakerdeck.com/dsyer/security-for-microservices-with-spring)  - Dave Syer

[https://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified](https://aaronparecki.com/articles/2012/07/29/1/oauth2-simplified) - Aaron Parecki

[http://www.w3schools.com/html/html5_webstorage.asp](http://www.w3schools.com/html/html5_webstorage.asp)




