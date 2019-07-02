---
title: SpringSocial
date: 2019-06-20 15:39:17
tags: Spring
---

对于一个应用来说，提供对社交网络的支持是很有必要的。应用可以通过与社交网络的集成来迅速积累人气。与社交网络的集成主要有两个方面的功能：第一个是比较简单的用户登录集成，即允许用户使用社交网络网站上的已有账户来登录应用。这样做的好处是可以省去要用户重新注册的流程，同时也可以与用户已有的社交网络建立连接；第二个是深度的集成方式，即允许用户把应用中的相关内容分享到社交网络中。这样做的好处是可以保持用户的粘性，同时推广应用本身。

<!--more-->

# 使用社交网络已有账号登录

与社交网络集成的最基本的功能是允许用户使用已有的账号进行登录。用户除了注册新的账号之外，还可以使用已有的其他网站的账号来登录。这种方式通常称为连接第三方网站。对于用户来说，通常的场景是在注册或登录页面，选择第三方社交网络来进行连接。然后跳转到第三方网站进行登录和授权。完成上述步骤之后，新的用户会被创建。在连接过程中，应用可以从第三方社交网站获取到用户的概要信息，来完成新用户的创建。当新用户创建完成之后，下次用户可以直接使用第三方社交网络的账号进行登录。

## 基本配置

Spring Social 提供了处理第三方网站用户登录和注册的 Spring MVC 控制器（Controller）的实现，可以自动完成使用社交网络账号登录和注册的功能。在使用这些控制器之前，需要进行一些基本的配置。Spring Social 的配置是通过实现 org.springframework.social.config.annotation.SocialConfigurer 接口来完成的。[代码清单 1](https://www.ibm.com/developerworks/cn/java/j-lo-spring-social/#_%E6%B8%85%E5%8D%95%201.%20%E4%BD%BF%E7%94%A8%20Spring%20Social%20%E7%9A%84%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE)给出了实现 SocialConfigurer 接口的 SocialConfig 类的部分内容。注解“@EnableSocial”用来启用 Spring Social 的相关功能。注解“@Configuration”表明该类也同样包含 Spring 框架的相关配置信息。SocialConfigurer 接口有 3 个方法需要实现：
*   addConnectionFactories：该回调方法用来允许应用添加需要支持的社交网络对应的连接工厂的实现。
*   getUserIdSource：该回调方法返回一个 org.springframework.social.UserIdSource 接口的实现对象，用来惟一标识当前用户。
*   getUsersConnectionRepository：该回调方法返回一个 org.springframework.social.connect.UsersConnectionRepository 接口的实现对象，用来管理用户与社交网络服务提供者之间的对应关系。
具体到示例应用来说，addConnectionFactories 方法的实现中添加了由 org.springframework.social.linkedin.connect.LinkedInConnectionFactory 类表示的 LinkedIn 的连接工厂实现。getUserIdSource 方法的实现中通过 Spring Security 来获取当前登录用户的信息。getUsersConnectionRepository 方法中创建了一个基于数据库的 JdbcUsersConnectionRepository 类的实现对象。LinkedInConnectionFactory 类的创建方式体现了 Spring Social 的强大之处。只需要提供在 LinkedIn 申请的 API 密钥，就可以直接使用 LinkedIn 所提供的功能。与 OAuth 相关的细节都被 Spring Social 所封装。
