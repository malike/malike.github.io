---
layout: post
comments: true
title: Why TDD and How To With Fluentlenium, RestAssured, HamCrest and Mockito
author: malike
categories: [tech]
tags: [tdd,test,test-driven-development]
image:
    path: /posts/dilbert_tdd.gif
    width: 800
    height: 500
alt: .
redirect_from: "/Why-TDD-and-How-To/"
---

### 1. What's TDD?

It stands for [Test Driven Development](http://www.jamesshore.com/Agile-Book/test_driven_development.html). Simply meaning the Test Drives the Development(Coding). This is done in small parts and starts with writing a test for a feature. 

This involves a cycle of this 3 steps in small parts.

***a. <span style="color:#EF5350">Red Bar (Fail)</span>: Code for specification that fails.***

***b. <span style="color:#4caf50">Green Bar(Pass)</span> : Code and test enough to make sure specification passes.***

***c. Refactor, Retest (Cleanup)***

Note that the order matters. If step 2 comes before step 1. It's not TDD.



 ### 2. Why you should


 	***a. The next dev : “if small is good, then smaller must be better” - [Bob Martin](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)***

Sometimes we tend to get locked in and code the specs away with just the end user in mind. One very important person we forget is the next dev. The next developer or engineer who is going to modify code.

Unit Testing in TDD *forces* devs to break spaghetti code into smaller units. The fact is every good developer should know how to
break any service into units and not write codes over a certain number of lines. 
Its not something that comes just with TDD. Sticking to TDD's guidelines enforces you to break spaghetti into smaller units.


 	***b. Faster test cycles***
I wasn't always on the TDD train. It felt rather long and unit testing (a central part of TDD ) doesn't really reflect how the end user interacts with the product. So I placed high priorities on end-to-end tests and integration tests using this cycle:

   ***a. Code for production***

   ***b. Test your end points or APIs (Integration test).***

   ***c.  Perform an end to end after putting the whole application together.***

The problem with this flow is I relied a lot on integration and end to end tests. Which are generally slow. 
For example an end to end test of an end user updating their profile page would be blocked if the login end to end test fails.
Testers would have to wait till the login works perfectly before moving on to test the profile update. Some of the bugs that only show up during end to end tests could have being discovered during unit test. 

In summary, unit tests help to discover bugs just faster than integration tests, they also help to isolate bugs and are far more reliable to help devs discover bugs before end to end tests.

   ***c. Discover [Regression Errors](https://en.wikipedia.org/wiki/Software_regression) Faster***

_"A software regression is a software bug which makes a feature stop functioning as intended after a certain event (for example, a system upgrade, system patching or a change to daylight saving time)"_

Once you start refactoring your code for a feature, another service that depends on your newly refactored service would either fail or pass.
TDD would help you reduce regression errors. Once a test that used to pass starts to fail you know you have to fix before pushing to production.


 	***d. Confidence***

Overall confidence your deliverable increases. The specification doesn't just work on your machine but works Production,UAT and any other dev's machine.

Well, there will still be bugs but not as much as with TDD.

### 3. A Sample TDD Project

Using Spring Boot,let's build a sample project using TDD principles. The project would be a simple application to help users signup. 
The dependencies needed for testing are listed below.

```xml
    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.fluentlenium</groupId>
            <artifactId>fluentlenium-core</artifactId>
            <version>0.10.9</version>
        </dependency>

        <dependency>
            <groupId>org.fluentlenium</groupId>
            <artifactId>fluentlenium-assertj</artifactId>
            <version>0.10.9</version>
        </dependency>

        <dependency>
            <groupId>com.jayway.restassured</groupId>
            <artifactId>spring-mock-mvc</artifactId>
            <scope>test</scope>
            <type>jar</type>
        </dependency>

         <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <scope>test</scope>
        </dependency>
```
       

What the dependencies give us?

   1.&nbsp;[Fluentlenium](http://fluentlenium.org/docs/). 
_"FluentLenium helps you writing readable, reusable, reliable and resilient UI functional tests for the browser."_

   2.&nbsp;[RestAssured](http://rest-assured.io/) and [HamCrest](http://hamcrest.org/) give us the opportunity to test rest service and validate.

   3.&nbsp;[Mockito](http://site.mockito.org/) 
_"Mockito is used to mock interfaces so that a dummy functionality can be added to a mock interface that can be used in unit testing. This tutorial should help you learn how to create unit tests with Mockito as well as how to use its APIs in a simple and intuitive way..."_


Let's start

We can break down the project in 2 parts. 
<br/><br/>
  i. Frontend <br/>
 ii. HTTP API to process the request from the frontend.<br/>


1. Create a spring boot project with thymeleaf and h2 dependencies. 

2. Write the first functional test for the signup page.

```java
	@RunWith(SpringJUnit4ClassRunner.class)
	@SpringApplicationConfiguration(classes = AppMain.class)
	@WebAppConfiguration
	@IntegrationTest(value = "server.port:9000")
	public class SignupControllerUITest extends FluentTest{

		public WebDriver webDriver = new FirefoxDriver();
```

The _webDriver_ is what Fluentlenium uses to browse natively as a user would. The _FireFoxDriver_ options works fine because it comes as default. Our current setup requires FireFox 38. 
I won't advice using the ChromeDriver unless you can ship the driver based on OS with your code and switch based on the other developers you work with. AFAIK It requires you set a path to the _ChromeDriver_ which doesn't make the code developer friendly.

Another option is the _HtmlUnitDriver_. However it won't open the browser natively but it would also test the page.


1. Lets create our first test : _Can I see the signup page?_

```java
	@Test
	public void testViewSignupPage() throws Exception {
	    goTo("http://localhost:9000/signup");
	    FluentLeniumAssertions.assertThat(find(".header")).hasText("Signup");
	}
```

This would obviously fail because we don't have the signup page. So we fix it by creating our signup page. So our test would pass.

![_config.yml](/posts/first_red.png) 

To fix this, we'll create a controller for our signup page

```java
    @RequestMapping(value = "/signup",method = RequestMethod.GET)
	public String signup() {
	    return "signup";
	}
```

Test, let it fail then create the view for our signup page in _"/resources/template"_. This is based on the default config for thymeleaf
We'll make sure we have an element the css selector _".header"_ and has text _"Signup"_ to pass our test.

![_config.yml](/posts/first_green.png).


Next we would like to create a form for our signup page. So we create a test that checks our signup page for the form.
This would fail. We would move on to create the signup page to pass that test.

Once passed,we'll create a test that fills our signup form and submits. 

```java
	Assert.assertEquals("SUCCESS: "+USERNAME+" saved.",webDriver.switchTo().alert().getText());
```    

This would give us another failure because we are yet to code our signup page to show any alert although our test expects an alert.
After adding the jquery code to submit the form details to our HTTP API_ with the help of an alert we can know whether it works or not.

Note that [Fluentlenium](http://fluentlenium.org/docs/) offers lots of locators you can use to validate as well. 

After refactoring our code in the view to show a success alert or failure alert we are one step close since we don't get this exception 

	org.openqa.selenium.NoAlertPresentException: No alert is present (WARNING: The server did not provide any stacktrace information)

but this

	org.junit.ComparisonFailure: 
	Expected :SUCCESS : malike saved.
	Actual   :ERROR:405


We refactor to fix this as well. We fix this red by creating the api for saving user data. We create a test for our signup HTTP API in the 
SignupControllerTest to first return 200. 

Then another test to confirm your details get saved once the parameters are passed to the api. This failing test would require us to create
a UserServiceTest class.

We continue with this cycle of creating a test,which would fail,refactor till its passes. Then we move on to the next specification.  


Check out the [source code](https://github.com/malike/spring-boot-tdd). The test cases for the the signup page can be found [here](https://github.com/malike/spring-boot-tdd/blob/master/src/test/java/st/malike/spring/boot/tdd/http/SignupControllerUITest.java)
and the test cases for the api [here](https://github.com/malike/spring-boot-tdd/blob/master/src/test/java/st/malike/spring/boot/tdd/http/SignupControllerTest.java) and the tests for our UserServiceTest [here](https://github.com/malike/spring-boot-tdd/blob/master/src/test/java/st/malike/spring/boot/tdd/service/UserServiceTest.java).

The [Frontend](https://github.com/malike/spring-boot-tdd/blob/master/src/test/java/st/malike/spring/boot/tdd/http/SignupControllerUITest.java) test uses [Fluentlenium](http://fluentlenium.org/docs/) to help us use the browser natively as our end user would. <br/>
The [Controller](https://github.com/malike/spring-boot-tdd/blob/master/src/test/java/st/malike/spring/boot/tdd/http/SignupControllerTest.java) test uses RestAssured to test our HTTP endpoint and HamCrest to help us validate the response.<br/>
The [UserService](https://github.com/malike/spring-boot-tdd/blob/master/src/test/java/st/malike/spring/boot/tdd/service/UserServiceTest.java) test uses Mockito to help us mock UserService and all the services it depends on,which in our case is just UserRepository. <br/>

You can see that although the Frontend,the SignUpController and the UserService tests are different they follow the same TDD guidelines. Breaking into bits and writing the tests before the feature and refactoring till the pass.

Although a bit limited in features,with this example we've been able to create test cases to test our frontend using [Fluentlenium](http://fluentlenium.org/docs/) and also using Mockito,RestAssured and HamCrest test our service and HTTP API. 

After 12 test cases,all green, we've built a signup feature using TDD.

![_config.yml](/posts/all_green.png).



<br>
<br>

	
> Our sample project can be found [here](https://github.com/malike/spring-boot-tdd)






<br>
<br>
**REFERENCES**

[http://www.jamesshore.com/Agile-Book/test_driven_development.html](http://www.jamesshore.com/Agile-Book/test_driven_development.html)
[https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html)
