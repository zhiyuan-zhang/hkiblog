---
title: 责任链模式
date: 2019/10/8
categories:
  - 设计模式
  - java
typora-root-url: ../
typora-copy-images-to: ../static
abbrlink: 29384
---

```java

public class Servlet_Main {
    public static void main(String[] args) {
        Request request = new Request();
        request.str = "开始 进入: <> 1 ";
        Response response = new Response();
        response.str = "结束 退出: ";

        FilterChain chain = new FilterChain();
        chain.add(new HTMLFilter()).add(new SensitiveFilter());
        chain.doFilter(request, response);
        System.out.println(request.str);
        System.out.println(response.str);

    }
}

interface Filter {
    void doFilter(Request request, Response response, FilterChain chain);
}

class HTMLFilter implements Filter {
    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {

        // 进入的时候做的业务逻辑
        request.str = request.str.replaceAll("<", "[").replaceAll(">", "]") + "HTMLFilter()";

        // 调用下一个chain
        chain.doFilter(request, response);

        // 完成后做业务逻辑
        response.str += "--HTMLFilter()";

    }
}

class Request {
    String str;
}

class Response {
    String str;
}

class SensitiveFilter implements Filter {

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {

        // 进入之前做的业务逻辑
        System.out.println("拦截器2");
        request.str = request.str.replaceAll("1", "2") + " SensitiveFilter()";
        // 去找下一个chain
        chain.doFilter(request, response);

        // 之后的逻辑处理
        response.str += "--SensitiveFilter()";

    }
}



class FilterChain {
    List<Filter> filters = new ArrayList<>();
    int index = 0;

    public FilterChain add(Filter f) {
        filters.add(f);
        return this;
    }

    public void doFilter(Request request, Response response) {
        if(index == filters.size()) {
            // 做了某些业务后需要返回
            System.out.println("做了某些业务后需要返回");
            return;

        }
        Filter f = filters.get(index);

        index ++;

        f.doFilter(request, response, this);
    }
}


```

