---
title: "Strategy Pattern"
date: 2022-08-20T15:59:13+08:00
type: page
---

```
Define a family of algorithms, encapsulate each one, and make them interchangeable.
```

策略模式就是应对**同一类型**的问题，存在**多种独立处理方式**的场景。

### 场景特征

- 多种处理方式独立解决同一种问题
- 自由切换实现方式/算法
- 屏蔽实现细节

### 优缺点

- 符合开闭原则，避免使用多重条件转移语句，提高算法保密性、安全性
- 客户端必须知道所有的策略，策略类维护难度大

最简单的就是 if/else，switch/case ，高级语言都有实现，其他场景: 支付方式、阶梯价格、优惠方案，Spring 初始化（策略之间继承）。。。

我的天，之前图床使用 typora+picgo+sm.ms，很长时间没写，今天发现突然 sm.ms 因为网络原因不能用了。。。，又折腾到了七牛云，还好不多，明天在迁移一下，有空看看七牛 qshell 怎么写的，这么多功能，有没有什么设计模式。。。



### ![](https://image.liguangchang.cn/log_image/strategy-pattern.png)



### 角色

- Context 操作策略的上下文环境，屏蔽高层模块对实际策略的直接访问
- Algorithm 策略的抽象定义，控制策略行为
- ConcreteStrategy 具体策略角色，具体的策略实现

### 简单实现

```java
public class Client {
    public static void main(String[] args) {
        SimpleStrategy strategy = new ConcreteStrategyFoo();
        Context strategyContext = new Context(strategy);
        strategyContext.algorithm();
	      //I`m strategy foo.
    }

    interface SimpleStrategy {
        void algorithm();
    }

    static class Context {
        private SimpleStrategy strategy;

        public Context(SimpleStrategy strategy) {
            this.strategy = strategy;
        }

        public void algorithm() {
            this.strategy.algorithm();
        }
    }

    //concrete
    static class ConcreteStrategyFoo implements SimpleStrategy {

        @Override
        public void algorithm() {
            System.out.println("I`m strategy foo.");
        }
    }

    static class ConcreteStrategyBar implements SimpleStrategy {

        @Override
        public void algorithm() {
            System.out.println("I`m strategy bar.");
        }
    }
}
```

### context

- map 作为context

```java
public interface PromotionStrategy {
    void promotionAlgorithm();

    class PromotionStrategyFoo implements PromotionStrategy {
        @Override
        public void promotionAlgorithm() {
            System.out.println("promotion strategy foo.");
        }
    }

    class DefaultPromotionStrategy implements PromotionStrategy {
        @Override
        public void promotionAlgorithm() {
            System.out.println("promotion strategy default.");
        }
    }

    class PromotionStrategyBar implements PromotionStrategy {
        @Override
        public void promotionAlgorithm() {
            System.out.println("promotion strategy bar.");
        }
    }
}
```

```java
public class PromotionStrategyFactory {
    private static Map<String, PromotionStrategy> psMap = Maps.newHashMap();

    static {
        psMap.put(PromotionKey.FOO, new PromotionStrategy.PromotionStrategyFoo());
        psMap.put(PromotionKey.BAR, new PromotionStrategy.PromotionStrategyBar());
    }

    public static final PromotionStrategy emptyStrategy = new PromotionStrategy.DefaultPromotionStrategy();

    private PromotionStrategyFactory() {
    }

    public static PromotionStrategy getStrategy(String strategyKey) {
        PromotionStrategy promotionStrategy = psMap.get(strategyKey);
        return promotionStrategy == null ? emptyStrategy : promotionStrategy;
    }

    private interface PromotionKey {
        String FOO = "foo";
        String BAR = "bar";
    }
}
```



```java
public class TestPromotionStrategy {
    public static void main(String[] args) {
        PromotionStrategyContext sFoo = new PromotionStrategyContext(new PromotionStrategy.PromotionStrategyFoo());
        PromotionStrategyContext sBar = new PromotionStrategyContext(new PromotionStrategy.PromotionStrategyBar());
        sFoo.doPromotion();
        sBar.doPromotion();

        String promotionKey = "foo";
        PromotionStrategy strategy = PromotionStrategyFactory.getStrategy(promotionKey);
        strategy.promotionAlgorithm();
    }
}
```

- Bean context

```java
public class PromotionStrategyContext {
    private PromotionStrategy strategy;

    public PromotionStrategyContext(PromotionStrategy ps) {
        this.strategy = ps;
    }

    public void doPromotion() {
        strategy.promotionAlgorithm();
    }
}
```

```java
public class TestPromotionStrategy {
    public static void main(String[] args) {
        PromotionStrategyContext sFoo = new PromotionStrategyContext(new PromotionStrategy.PromotionStrategyFoo());
        PromotionStrategyContext sBar = new PromotionStrategyContext(new PromotionStrategy.PromotionStrategyBar());
        sFoo.doPromotion();
        sBar.doPromotion();
    }
}
```

### 支付策略

```java

public class PaymentStrategy {
    public static final String wx = "WX_PAYMENT";
    public static final String zfb = "ZFB_PAYMENT";
    public static final String default_payment = "CASH_PAYMENT";
    private static Map<String, Payment> paymentStrategy = Maps.newHashMap();

    static {
        paymentStrategy.put(wx, new Payment.WxPayment());
        paymentStrategy.put(zfb, new Payment.ZfbPayment());
        paymentStrategy.put(default_payment, new Payment.CashPayment());
    }

    public static Payment get(String paymentRoute) {
        if (!paymentStrategy.containsKey(paymentRoute)) {
            return paymentStrategy.get(default_payment);
        }
        return paymentStrategy.get(paymentRoute);
    }
}
```

```java
public abstract class Payment {
    public abstract String getRoute();

    public PaymentResult pay(String uid, double amount) {
        if (queryBalance(uid) < amount) {
            return new PaymentResult(500, "支付失败", "余额不足");
        }
        return new PaymentResult(200, "支付成功:余额:" + (queryBalance(uid) - amount), String.valueOf(amount));
    }

    public abstract double queryBalance(String uid);

    static class CashPayment extends Payment {
        @Override
        public String getRoute() {
            return "现金支付";
        }

        @Override
        public double queryBalance(String uid) {
            return 100;
        }
    }

    static class WxPayment extends Payment {
        @Override
        public String getRoute() {
            return "微信支付";
        }

        @Override
        public double queryBalance(String uid) {
            return 523;
        }
    }

    static class ZfbPayment extends Payment {
        @Override
        public String getRoute() {
            return "支付宝支付";
        }

        @Override
        public double queryBalance(String uid) {
            return 532;
        }
    }

    class PaymentResult {
        private int code;
        private String msg;
        private String data;

        public PaymentResult(int code, String msg, String data) {
            this.code = code;
            this.msg = msg;
            this.data = data;
        }

        @Override
        public String toString() {
            return msg + " 成功支付金额: " + data + "!";
        }
    }
}
```

```java
public class Cashier {
    private String uid;
    private String orderNo;
    private double amount;

    public Cashier(String uid, String orderNo, double amount) {
        this.uid = uid;
        this.orderNo = orderNo;
        this.amount = amount;
    }

    public Payment.PaymentResult pay() {
        return pay(PaymentStrategy.default_payment);
    }

    public Payment.PaymentResult pay(String paymentRoute) {
        Payment payment = PaymentStrategy.get(paymentRoute);
        System.out.println("welcome to advanced intelligent cashier system!\n支付方式:" + payment.getRoute());
        System.out.println("本次交易金额: " + this.amount + "\n开始扣款...\n等待系统确认...");
        return payment.pay(this.uid, this.amount);
    }
}
```

```java
public class TestCashier {
    public static void main(String[] args) {
        Cashier order = new Cashier("1", "123", 23.5);
        System.out.println(order.pay(PaymentStrategy.wx));
    }
}
welcome to advanced intelligent cashier system!
支付方式:微信支付
本次交易金额: 23.5
开始扣款...
等待系统确认...
支付成功:余额:499.5 成功支付金额: 23.5!
```

### go 实现策略

```go
import (
	"fmt"
	"testing"
)

type Strategy interface {
	sendMsg()
}

type FooStrategy struct {
}

func (f FooStrategy) sendMsg() {
	fmt.Println("fooStrategy sendMsg.")
}

type BarStrategy struct {
}

func (b BarStrategy) sendMsg() {
	fmt.Println("barStrategy sendMsg.")
}

type StrategyType interface {
	FooStrategy | BarStrategy
}

var m = map[string]Strategy{
	"Foo": &FooStrategy{},
	"Bar": &BarStrategy{},
}

func TestStrategy(t *testing.T) {
	strategy, ok := m["Foo"]
	if ok {
		strategy.sendMsg()
	}
}
//fooStrategy sendMsg.
```

