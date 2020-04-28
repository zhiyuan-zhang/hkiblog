---
title: 异步的观察者模式处理后台任务
date: 2018/10/8
tags:
  - java
  - 模板
comments: true
abbrlink: 49197
img:
---


收到任务说要做一个后台监听文件上传接口 

如果有docx文档 需要将任务放在后台进程中

将docx文档转换成pdf文档

首先文档转换不是我们今天要讲的内容 稍后我会开一章来讲

事件监听在spring boot里还是比较容易实现的 有以下几点需要注意

- vo实体需要继承 ApplicationEvent
- 实现接口ApplicationListener里的onApplicationEvent方法
- 使用applicationContext的publishEvent来发布事件
- 最后继承AsyncUncaughtExceptionHandler添加异步报错控制



新建实体 因为我这里只用到 id 所以只需要传入id即可
```java

/**
 * @author: ZHANG.HAO
 * @Description:
 */
public class WordToPdfEvent extends ApplicationEvent {

    Integer id;


    public WordToPdfEvent(Integer id) {
        super(id);
        this.id = id;
    }
}

```


这里需要写你的后台监听到事件后要做的内容 我的是转换文档
```java

/**
 * @Author: ZHANG.HAO
 * @Description:
 */
@Slf4j
@Component
public class handleListener implements ApplicationListener<WordToPdfEvent> {

    @Override
    public void onApplicationEvent(WordToPdfEvent wordToPdfEvent) {

       // do something
    }
}

```


异步发布事件

```java

/**
 * @author: ZHANG.HAO
 * @Description:
 */
@Slf4j
@Service
public class AsyncService {


    @Autowired
    ApplicationContext applicationContext;

    @Async
    public void executorAsyncTask(List<SysFile> list){

            list.forEach(SysFile -> {
                applicationContext.publishEvent(new WordToPdfEvent(SysFile.getId()));
            });
    }

}

```

配置后台异步exception
```java

/**
 * 后台异步exception
 * @author: ZHANG.HAO
 * @Description:
 */
@Slf4j
public class MyAsyncUncaughtExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Autowired
    ISysLogService sysLogService;

    @Override
    public void handleUncaughtException(Throwable ex, Method method, Object... params) {
        log.info("handleUncaughtException ");

        StringJoiner sjr = new StringJoiner(",", "[", "]");
        sjr.add("Exception Cause - " + ex.getMessage()) ;
        sjr.add("Method name - " + method.getName()) ;
        for (Object param : params) {
            sjr.add("Parameter value - " + param) ;
        }
         sysLogService.save(new SysLog(-99,sjr.toString()));

    }


}

```


配置异步连接池
```java

/**
 * @author ZHANG.HAO
 * @Date: 2019-09-09 18:34
 * @Description:
 */
@Component
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(17);
        executor.setMaxPoolSize(42);
        executor.setQueueCapacity(11);
        executor.setThreadNamePrefix("WordToPdfExecutor-");
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new MyAsyncUncaughtExceptionHandler();
    }

}

```







