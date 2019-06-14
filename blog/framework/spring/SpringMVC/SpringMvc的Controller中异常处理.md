<!-- SpringMvc的Controller中异常处理 -->

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Spring中默认的异常处理](#spring中默认的异常处理)
- [使用`@ResponseStatus`处理自定义异常](#使用responsestatus处理自定义异常)
- [使用`try {…} catch` 手动捕获异常](#使用try-catch-手动捕获异常)
- [使用`@ExceptionHandler`处理自定义异常](#使用exceptionhandler处理自定义异常)
- [使用`@ControllerAdvice`注解处理所有的异常](#使用controlleradvice注解处理所有的异常)
- [总结](#总结)
- [参考](#参考)

<!-- /TOC -->


# Spring中默认的异常处理

在Spring中，有一些异常会默认映射为HTTP状态码，不需要程序处理。下表列出Spring的默认处理异常：

|Spring异常 | HTTP状态码|
|---|---|
BindException | 400 - Bad Request
ConversionNotSupportedException | 500 - Internal Server Error
HttpMediaTypeNotAcceptableException | 406 - Not Acceptable
HttpMediaTypeNotSupportedException | 415 - Unsupported Media Type
HttpMessageNotReadableException | 400 - Bad Request
HttpMessageNotWritableException | 500 - Internal Server Error
HttpRequestMethodNotSupportedException | 405 - Method Not Allowed
MethodArgumentNotValidException | 400 - Bad Request
MissingServletRequestParameterException | 400 - Bad Request
MissingServletRequestPartException | 400 - Bad Request
NoSuchRequestHandlingMethodException | 404 - Not Found
TypeMismatchException | 400 - Bad Request

上表中的异常会由Spring自身抛出，如果`DispatcherServlet`处理过程中或执行校验时出现问题时则直接返回。例如，如果`DispatcherServlet`无法找到适合处理请求的控制器方法，那么将会抛出`NoSuchRequestHandlingMethodException`异常，最终的结果就是产生404状态码的响应（Not Found）。

# 使用`@ResponseStatus`处理自定义异常

但对于应用程序内部抛出的自定义异常，它就不能处理了。比如service中抛出了一个自定义异常（`NullOrgException`），我们对`NullOrgException`异常添加`@ResponseStatus`注解，对其指定状态码即可。

如下面的代码：

```java
package com.rebecca.springmvc.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

/**
 * 自定义异常
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/14 17:04
 * @Modified By:
 */

@ResponseStatus(value = HttpStatus.NO_CONTENT, reason = "No Content")
public class NullOrgException extends RuntimeException {
}

```


# 使用`try {…} catch` 手动捕获异常

定义了上述异常后，只要应用程序中有抛出`NullOrgException`异常，就会被捕获并映射为对应的状态码。那么如果程序不仅仅需要状态码，还要包含所产生的错误，那该怎么办呢？此时的话，我们就不能将异常视为HTTP错误了，而是要按照处理请求的方式来处理异常了。

```java
package com.rebecca.springmvc.controller.exception;

import com.rebecca.springmvc.org.bean.Org;
import com.rebecca.springmvc.org.service.OrgService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
@RequestMapping("test/exception")
public class ExceptionTestController {
    private Logger logger = LoggerFactory.getLogger(ExceptionTestController.class);

    @Autowired
    private OrgService service;

    @RequestMapping(value = "orgs", method = RequestMethod.GET)
    @ResponseBody
    public List<Org> getOrgs ()  {
        List<Org> orgs = null;
        try {
            orgs = service.getOrgs();
        } catch (NullOrgException e) {
            logger.error("无组织机构相关数据！",e);
        }
        return orgs;
    }
}
```

比如上面代码中的`NullOrgException`异常，在Spring中没有默认映射，那最简单的办法就是`try {…} catch (NullOrgException e) {…}` 。


# 使用`@ExceptionHandler`处理自定义异常

上述`try {…} catch`方式不利于代码维护，好在Spring提供了一种机制，可以用`@ExceptionHandler`注解将异常映射为HTTP状态码。下面是使用`@ExceptionHandler`注解后的方式：

```java
package com.rebecca.springmvc.controller.exception;

import com.rebecca.springmvc.org.bean.Org;
import com.rebecca.springmvc.org.service.OrgService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
@RequestMapping("test/exception")
public class ExceptionTestController {
    private Logger logger = LoggerFactory.getLogger(ExceptionTestController.class);

    @Autowired
    private OrgService service;

    @RequestMapping(value = "orgs", method = RequestMethod.GET)
    @ResponseBody
    public List<Org> getOrgs ()  {
        List<Org> orgs = service.getOrgs();
        return orgs;
    }

    @ExceptionHandler(NullOrgException.class)
    public String handleNullOrgException() {
        return "无组织机构相关数据！";
    }
}
```

上面代码我们在`handleNullOrgException()`方法上添加了`@ExceptionHandler`注解，当程序抛出`NullOrgException`异常时，将会委托该方法来处理。它的返回值是String，你也可以改为其它的返回类型以满足应用程序需要。对于用`@ExceptionHandler`注解标注的方法来说，它能处理同一个控制器(Controller)中所有处理器方法所抛出的异常。

为了避免在多个控制器中编写重复的`@ExceptionHandler`注解方法，我们会创建一个基础的控制器类，所有控制器类要扩展这个类，从而 **继承** 通用的`@ExceptionHandler`方法。

# 使用`@ControllerAdvice`注解处理所有的异常

前面我们说使用`@ExceptionHandler`注解只能处理一个控制器(Controller)中所有处理器方法所抛出的异常，那么有没有一种方法不用集成就能够处理所有控制器中处理器方法所抛出的异常呢？

答案是：有！从Spring 3.2开始，这肯定是能够实现的，我们只需将其定义到控制器通知类中即可。

在Spring 3.2之后，为这类问题引入了一个新的解决方案：控制器通知。

控制器通知（controller advice）是指任意带有`@ControllerAdvice`注解的类。

这个类会包含一个或多个如下类型的方法：
1. `@ExceptionHandler`注解标注的方法；
2. `@InitBinder`注解标注的方法；
3. `@ModelAttribute`注解标注的方法。

在带有`@ControllerAdvice`注解的类中，上述的这些方法会运用到整个应用程序所有控制器中带有`@RequestMapping`注解的方法上。`@ControllerAdvice`注解本身已经使用了`@Component`，因此`@ControllerAdvice`注解所标注的类将会自动被组件扫描获取到，和有`@Component`注解的类一样。`@ControllerAdvice`最为实用的一个场景就是将所有的`@ExceptionHandler`方法收集到一个类中，这样所有控制器的异常就能在一个地方进行统一处理。例如，我们想将`NullOrgException`的处理方法用到整个应用程序的所有控制器上。如下的程序清单展现的`AppWideExceptionHandler`就能完成这一任务，这是一个带有`@ControllerAdvice`注解的类。下面代码使用`@ControllerAdvice`，为所有的控制器处理异常：

```java
package com.rebecca.springmvc.controller.exception;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

/**
 * 控制器通知类
 * @Author: Rebecca
 * @Description:
 * @Date: Created in 2019/6/14 16:30
 * @Modified By:
 */
@ControllerAdvice  // 定义控制器类
public class AppWideException {

    // 定义异常处理方法
    @ExceptionHandler(NullOrgException.class)
    public String handleNullOrgException() {
        return "无组织机构相关数据！";
    }
}
```

现在，如果任意的控制器方法抛出了DuplicateSpittleException，不管这个方法位于哪个控制器中，都会调用这个duplicateSpittleHandler()方法来处理异常。我们可以像编写@RequestMapping注解的方法那样来编写@ExceptionHandler注解的方法。如程序清单7.10所示，它返回“error/duplicate”作为逻辑视图名，因此将会为用户展现一个友好的出错页面。


# 总结

Spring中的异常处理：
1. 特定的Spring异常将会自动映射为指定的HTTP状态码；
1. *异常* 上可以添加`@ResponseStatus`注解，从而将其映射为某一个HTTP状态码；
1. 在 *方法* 上可以添加`@ExceptionHandler`注解，使其用来处理异常。
1. 将`@ExceptionHandler`注解的异常方法提取到`@ControllerAdvice`注解的类中，整个应用程序生效（该方式只适用于Spring3.2+）。

处理异常的最简单方式就是将其映射到HTTP状态码上，进而放到响应之中。

# 参考

《Spring实战第4版》
