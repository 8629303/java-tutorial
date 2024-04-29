> 作者：小P不是Peppa；来源：https://mp.weixin.qq.com/s/VZ09N_aLqtJ1etPfNKcUgA

## 一、正常流程的代码

这个Demo实现数据的简单查询

```java
@RestController
public class TestController {
    @Autowired
    TestService testService;

    @GetMapping("/get")
    public String get() {
        return testService.get();
    }
}

@Service
public class TestService {
    @Autowired
    TestRepository testRepository;
    public String get() {
        return testRepository.get();
    }
}

@Component
public class TestRepository {
    public String get() {
        return "test service";
    }
}
```







## 二、改造动态的创建Sevice

上述demo中，不论controller，service还是dao，都有一个共同点，那就是它们都交由spring容器来管理（@RestController、@Service、@Repository）和依赖注入（@Autowired），在容器刷新完成之后这些Bean已经生成并完成了依赖注入，但是现在有一个问题，我不想让它们这么快生成，而是想在后续用到的时候再生成，并且用完后删除，再用到的时候再创建，这时候应该怎么做？

1、把service改动下

去掉@Service注解，不让容器管理，同时去掉依赖注入，提供set方法方便后续设置dao属性

2、提供一个动态生成、查询、删除service的类和一个对应的controller便于访问

```java
import com.example.demo.dao.WinnersDao;
import com.example.demo.service.WinnersService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class DynamicFactory implements BeanFactoryAware, ApplicationContextAware {

    private ApplicationContext applicationContext;

    private DefaultListableBeanFactory beanFactory;

    public WinnersService dynamicGetWinnerService() {
        WinnersService winnersService = null;
        try {
            winnersService = beanFactory.getBean("winnersService", WinnersService.class);
            log.info("获取到的winnersService为：" + winnersService);
        } catch (Exception e) {
            log.info("dynamicGetWinnerService error==>" + e.getMessage());
        }
        return winnersService;
    }

    /**
     * 动态注册bean
     */
    public void dynamicCreateWinnerService() {
        try {
            /**
             * 创建bean定义，设置属性winnersDao，前提是winnersDao已经被spring管理了
             */
            BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(WinnersService.class);
            beanDefinitionBuilder.addPropertyReference("winnersDao", "winnersDao");
            beanFactory.registerBeanDefinition("winnersService", beanDefinitionBuilder.getRawBeanDefinition());
            log.info("创建winnersService......");
        } catch (Exception e) {
            log.error("dynamicCreateWinnerService error==>" + e.getMessage());
        }
    }

    /**
     * 动态销毁bean
     */
    public void dynamicDestroyBean() {
        try {
            beanFactory.destroySingleton("winnersService");
            beanFactory.removeBeanDefinition("winnersService");
            log.info("销毁winnersService......");
        } catch (Exception e) {
            log.error("dynamicDestroyBean error==>" + e.getMessage());
        }
    }


    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (DefaultListableBeanFactory) beanFactory;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

```java
package com.example.demo.controller;

import com.example.demo.custom.DynamicFactory;
import com.example.demo.service.WinnersService;
import com.example.demo.vo.WinnersVo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/dynamic")
public class DynamicController {

    @Autowired
    private DynamicFactory dynamicFactory;

    @RequestMapping("createBean")
    public String createBean() {
        dynamicFactory.dynamicCreateWinnerService();
        return "createBean success";
    }

    @RequestMapping("destroyBean")
    public String destroyBean() {
        dynamicFactory.dynamicDestroyBean();
        return "destroyBean success";
    }
}
```

3、改造controller，注入DynamicFactory，后续通过它获取service

