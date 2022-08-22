---
title: "Chain Pattern"
date: 2022-08-21T14:04:35+08:00
type: page
---

```
Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.
```

请求会被责任链中链头到链尾的节点依次判断处理。

### 场景特征

- 多个节点可以处理同一个请求，处理节点在运行时决定
- 不明确指定接受者的情况下，请求多个节点中的一个
- 可以动态制定一组节点来处理请求
- Filter filter chain，netty pipeline

### 优缺点

- 请求和处理解耦
- 节点处理只需关注自己处理，不处理转发到下一个节点
- 请求在链路上传递，请求发送者不需要知道链路结构
- 符合开闭原则，易于扩展，链路结构灵活，可以通过改变链路结构动态调整
- 链路过长或者处理时间过长，影响整体性能
- ⚠️存在**循环引用会造成死循环**

![](/Users/lgc/code/github/images/design-pattern/chain-pattern.png)

### 角色

- Handler 处理器方法约束，维护一个 next handler 
- ConcreteHandler 具体的处理实现

### 简单实现

```java
public class TestSimpleChain {
    public static void main(String[] args) {
        Handler foo = new ConcreteHandlerFoo();
        Handler bar = new ConcreteHandlerBar();
        foo.setNextHandler(bar);
        foo.handleReq("bar");
    }

    static abstract class Handler {
        protected Handler nextHandler;

        public void setNextHandler(Handler nextHandler) {
            this.nextHandler = nextHandler;
        }

        public abstract void handleReq(String req);
    }

    static class ConcreteHandlerFoo extends Handler {

        @Override
        public void handleReq(String req) {
            if (req.equals("foo")) {
                System.out.println("foo handler req.");
                return;
            }
            System.out.println("foo.handler done, do nothing.");
            if (this.nextHandler != null) {
                this.nextHandler.handleReq(req);
            }
        }
    }

    static class ConcreteHandlerBar extends Handler {

        @Override
        public void handleReq(String req) {
            if (req.equals("bar")) {
                System.out.println("bar handler req.");
                return;
            }
            System.out.println("bar.handler done, do nothing.");
            if (this.nextHandler != null) {
                this.nextHandler.handleReq(req);
            }
        }
    }
}
#
foo.handler done, do nothing.
bar handler req.
```

责任链在链结构较长时，串联整个链会变得繁琐，可以通过 builder 模式来进行优化，其实一开始是准备看 go 的 method chain，翻了翻就看到了设计模式。

### builder-chain

```java


public abstract class LoginHandler<T> extends BuildChainHandler<T> {
    public void next(LoginHandler<T> next) {
        this.chain = next;
    }

    public abstract void check(Member member);

    static class LoginValidateHandler<T> extends LoginHandler<T> {
        @Override
        public void check(Member member) {
            if (StringUtils.isBlank(member.getLoginName()) || StringUtils.isBlank(member.getLoginPass())) {
                System.out.println("please login in");
                return;
            }
            System.out.println("login pre check success");
            chain.check(member);
        }
    }

    static class LoginCheckHandler<T> extends LoginHandler<T> {
        @Override
        public void check(Member member) {
            member = MemberService.checkExists(member.getLoginName(), member.getLoginPass());
            if (null == member) {
                System.out.println("user not exist.");
                return;
            }
            System.out.println("user:" + member.getLoginName() + " login.");
            chain.check(member);
        }
    }

    static class LoginAuthHandler<T> extends LoginHandler<T> {
        @Override
        public void check(Member member) {
            if (!member.getRoleName().equals("admin")) {
                System.out.println("admin auth fail.,no operation");
                return;
            }
            System.out.println("welcome admin " + ":" + member.getLoginName());
        }

    }
}
```

```java

public class TestBuilderChain {
    public static void main(String[] args) {
        BuildChainHandler.Builder<? extends LoginHandler> builder = new BuildChainHandler.Builder();
        builder.addHandler(new LoginHandler.LoginValidateHandler()).
                addHandler(new LoginHandler.LoginCheckHandler()).
                addHandler(new LoginHandler.LoginAuthHandler());
        BuildChainHandler<? extends LoginHandler> buildChain = builder.build();
        buildChain.check(new Member("spider", "12345"));
    }
}

```

