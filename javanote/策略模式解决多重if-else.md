[TOC]

## 方式一：使用枚举+工厂+策略模式(运行单个)

### 使用if-else

假设我们要开发一个支付接口，要对接多种支付方式，通过渠道码区分各种的支付方式。于是定义一个枚举PayEnum，如下：

```java
public enum PayEnum {
    ALI_PAY("ali","支付宝支付"),
    WECHAT_PAY("wechat","微信支付"),
    UNION_PAY("union","银联支付"),
    XIAO_MI_PAY("xiaomi","小米支付");
    /**渠道*/
    private String channel;
    /**描述*/
    private String description;
    PayEnum(String channel, String description) {
        this.channel = channel;
        this.description = description;
    }
    /**以下省略字段的get、set方法*/
}
```

创建一个PayController类，代码如下：

```
@RestController
@RequestMapping("/buy")
public class PayController {
    @Resource(name = "payService")
    private PayService payService;
    /**
    * 支付接口
    * @param channel 渠道
    * @param amount  消费金额
    * @return String 返回消费结果
    * @author Ye hongzhi
    * @date 2020/4/5
    */
    @RequestMapping("/pay")
    public String pay(@RequestParam(name = "channel") String channel,
                      @RequestParam(name = "amount") String amount
    )throws Exception{
        return payService.pay(channel,amount);
    }
}
```

再创建一个PayService接口以及实现类PayServiceImpl

```java
public interface PayService {
    /**
    * 支付接口
    * @param channel 渠道
    * @param amount  金额
    * @return String
    * @author Ye hongzhi
    * @date 2020/4/5
    */
    String pay(String channel,String amount)throws Exception;
}
```

```java
@Service("payService")
public class PayServiceImpl implements PayService {
    private static String MSG = "使用 %s ,消费了 %s 元";
    @Override
    public String pay(String channel, String amount) throws Exception {
        if (PayEnum.ALI_PAY.getChannel().equals(channel)) {
            //支付宝
            //业务代码...
            return String.format(MSG,PayEnum.ALI_PAY.getDescription(),amount);
        }else if(PayEnum.WECHAT_PAY.getChannel().equals(channel)){
            //微信支付
            //业务代码...
            return String.format(MSG,PayEnum.WECHAT_PAY.getDescription(),amount);
        }else if(PayEnum.UNION_PAY.getChannel().equals(channel)){
            //银联支付
            //业务代码...
            return String.format(MSG,PayEnum.UNION_PAY.getDescription(),amount);
        }else if(PayEnum.XIAO_MI_PAY.getChannel().equals(channel)){
            //小米支付
            //业务代码...
            return String.format(MSG,PayEnum.XIAO_MI_PAY.getDescription(),amount);
        }else{
            return "输入渠道码有误";
        }
    }
}
```

效果：

![image-20200810103447494](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200810103447494.png)

这样看，以上代码的确可以实现需求，通过渠道码区分支付方式，可是看到上面那么多达4个的if-else的代码结构，已经开始显示出问题了。假设有更多的支付方式，那么这段代码就要写更多的else if去判断，这显然会不利于代码的扩展，这样会导致这个支付的方法越写越长。

在设计模式六大原则中，其中一个原则叫做**开闭原则，对扩展开放，对修改关闭，应尽量在不修改原有代码的情况下进行扩展**。

基于上面提到的开闭原则，我们可以使用策略模式进行重构。



### 使用策略模式重构代码

定义一个策略接口类PayStrategy

```java
public interface PayStrategy {

    String MSG = "使用%s，消费%s元";

    /**
     * 支付
     *
     * @param channel 通道
     * @param amount  量
     * @return {@link String}* @throws Exception 异常
     */
    String pay(String channel,String amount)throws Exception;
}
```

然后再创建四种策略实现类实现接口

```java
@Component("aliPay")
public class AliPayStrategyImpl implements IPayClient {

    @Override
    public Object pay(String channel, String amount) {
        System.out.println("支付宝支付："+amount+"元");
        return null;
    }
}
```

```java
@Component("wechatPay")
public class WechatPayStrategyImpl implements PayStrategy{
    @Override
    public String pay(String channel, String amount) throws Exception {
        return String.format(MSG, PayEnum.WECHAT_PAY.getDescription(),amount);
    }
}
```

```java
@Component("xiaomiPay")
public class XiaomiPayStrategyImpl implements PayStrategy{
    @Override
    public String pay(String channel, String amount) throws Exception {
        return String.format(PayEnum.XIAOMI_PAY.getDescription(), amount);
    }
}
```

```java
@Component("unionPay")
public class UnionPayStrategyImpl implements PayStrategy{
    @Override
    public String pay(String channel, String amount) throws Exception {
        return String.format(PayEnum.UNION_PAY.getDescription(),amount);
    }
}
```

思路就是通过渠道码，动态获取到集体的实现类，这样就可以实现不需要if else判断。怎么通过渠道码获取实现类呢？

在PayEnum枚举加上BeanName字段，然后增加一个通过渠道码获取BeanName的方法

```java
public enum PayEnum {
    /**
     * 支付宝支付
     */
    ALI_PAY("ali", "支付宝支付", "aliPay"),
    /**
     * 微信支付
     */
    WECHAT_PAY("wechat", "微信支付", "wechatPay"),
    /**
     * 银联
     */
    UNION_PAY("union", "银联支付", "unionPay"),
    /**
     * 小米支付
     */
    XIAOMI_PAY("xiaomi", "小米支付", "xiaomiPay");

    /**
     * 渠道
     */
    private String channel;

    /**
     * 描述
     */
    private String description;

    /**
     * bean的名字策略实现类对应的 beanName
     */
    private String beanName;

    public static PayEnum findPayEnumByChannel(String channel){
        PayEnum[] enums = PayEnum.values();
        for (PayEnum payEnum : enums){
            if (payEnum.getChannel().equals(channel)){
                return payEnum;
            }
        }
        return null;
    }

    PayEnum(String channel, String description, String beanName) {
        this.channel = channel;
        this.description = description;
        this.beanName = beanName;
    }




    public String getChannel() {
        return channel;
    }

    public void setChannel(String channel) {
        this.channel = channel;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getBeanName() {
        return beanName;
    }

    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }
}

```

这时候还差一个获取Spring上下文对象的工具类，于是我们创建一个SpringContextUtil类

```java
@Component
public class SpringContextUtil implements ApplicationContextAware {
    /**
     * 上下文对象实例
     */
    private static ApplicationContext applicationContext;
    /**
     * 获取applicationContext
     */
    private static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
    /**
     * 通过name获取Bean
     * */
    public static Object getBean(String name){
        return getApplicationContext().getBean(name);
    }
    /**
     * 通过name,以及Clazz返回指定的Bean
     * */
    public static <T> T getBean(String name,Class<T> clazz){
        return getApplicationContext().getBean(name,clazz);
    }
    @Override
    @Autowired
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextUtil.applicationContext = applicationContext;
    }
```

接着定义一个工厂类，通过渠道码获取对应的策略实现类

```java
public class PayStrategyFactory {
    /**
     * 通过渠道码获取支付策略具体实现类
     * */
    public static PayStrategy getPayStrategy(String channel){
        PayEnum payEnum = PayEnum.findPayEnumBychannel(channel);
        if(payEnum == null){
            return null;
        }
        return SpringContextUtil.getBean(payEnum.getBeanName(),PayStrategy.class);
    }
}
```

最后我们再改造一下原来的PayServiceImpl的pay方法

```java
@Override
public String pay(String channel, String amount) throws Exception {
    PayStrategy payStrategy = PayStrategyFactory.getPayStrategy(channel);
    if(payStrategy == null){
        return "输入渠道码有误";
    }
    return payStrategy.pay(channel,amount);
}
```

假设需要增加新的支付方式，就不需要再使用else if 去判断，而是在枚举中定义一个新的枚举对象，然后再增加一个策略实现类，实现对应的方法，那就可以很轻松地扩展。也实现了开闭原则。



## 方式二：使用枚举+MAP+策略模式(运行多个)

定义一个支付接口

```java
public interface IPayClient<R> {
    /**
     * 支付
     *
     * @param channel 渠道
     * @param amount  量
     * @return {@link R}
     */
    R pay(String channel, String amount);
}
```

定义支付方式枚举：

```java
public enum PayEnum {
    /**
     * 支付宝支付
     */
    ALI_PAY("ali", "支付宝支付"),
    /**
     * 微信支付
     */
    WECHAT_PAY("wechat", "微信支付"),
    /**
     * 银联
     */
    UNION_PAY("union", "银联支付"),
    /**
     * 小米支付
     */
    XIAOMI_PAY("xiaomi", "小米支付");

    /**
     * 渠道
     */
    private String channel;

    /**
     * 描述
     */
    private String description;



    PayEnum(String channel, String description) {
        this.channel = channel;
        this.description = description;

    }




    public String getChannel() {
        return channel;
    }

    public void setChannel(String channel) {
        this.channel = channel;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

}

```

定义策略接口

根据枚举的支付类型获取bean，调用bean的pay方法

```java
public interface PayStrategy {

    /**
     * lock
     */
    static final Object LOCK = new Object();

    /**
     * Msg map
     */
    static final ConcurrentHashMap<PayEnum, IPayClient<Object>> MSG_MAP = new ConcurrentHashMap<>();

    /**
     * 支付
     *
     * @param channel 通道
     * @param amount  量
     * @param payEnum 支付枚举
     * @return {@link Object}
     * @throws Exception 异常
     */
    public static Object pay(PayEnum payEnum, String channel, String amount) throws Exception{
        return getPayClient(payEnum).pay(channel,amount);
    }

    /**
     * 得到支付客户端
     *
     * @param payEnum 支付枚举
     * @return {@link IPayClient}
     * @throws Exception 异常
     */
    static IPayClient getPayClient(PayEnum payEnum) throws Exception {
        IPayClient<Object> payClient = MSG_MAP.get(payEnum);

        if (Objects.isNull(payClient)) {
            synchronized (LOCK) {
                payClient = MSG_MAP.get(payEnum);

                if (Objects.isNull(payClient)) {
                    Object bean = SpringContextUtil.getBean(payEnum.getChannel());
                    if (Objects.isNull(bean)) {
                        throw new Exception("找不到");
                    }

                    payClient = (IPayClient<Object>) bean;
                    putPayClient(payEnum, payClient);

                }
            }

        }
        return payClient;
    }

    /**
     * 将支付客户端
     *
     * @param payEnum   支付枚举
     * @param payClient 支付客户端
     */
    static void putPayClient(PayEnum payEnum, IPayClient payClient){
        synchronized (LOCK) {
            MSG_MAP.put(payEnum, payClient);
        }
    }
}

```

定义业务的接口

```java
public interface PayService {

    /**
     * 业务支付
     *
     * @param channel 渠道
     * @param amount  量
     * @return {@link String}* @throws Exception 异常
     */
    String payPal(String channel,String amount)throws Exception;

}
```

判断传入的渠道，添加入枚举集合里，遍历集合调用支付

```java
@Service("payService")
public class PayServiceImpl implements PayService {
    private static String MSG = "使用 %s ,消费了 %s 元";
    @Override
    public String payPal(String channel, String amount) throws Exception {

        Set<PayEnum> payEnums = new HashSet<>();

        if (PayEnum.ALI_PAY.getChannel().equals(channel)){
            payEnums.add(PayEnum.ALI_PAY);
        }
        if (PayEnum.WECHAT_PAY.getChannel().equals(channel)){
            payEnums.add(PayEnum.WECHAT_PAY);
        }

        doPay(channel, amount, payEnums);

        return "成功";

    }

    private void doPay(String channel, String amount, Set<PayEnum> payEnums) {
        payEnums.forEach(payEnum -> {
            try {
                PayStrategy.pay(payEnum,channel,amount);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });

    }
}
```

定义实现

**注意@Component的值和枚举类的一致**

```java
@Component("ali")
public class AliPayStrategyImpl implements IPayClient {

    @Override
    public Object pay(String channel, String amount) {
        System.out.println("支付宝支付："+amount+"元");
        return channel+"支付"+amount+"员";
    }
}
```

```java
@Component("union")
public class UnionPayStrategyImpl implements IPayClient {
    @Override
    public String pay(String channel, String amount) {
        System.out.println("银联支付："+amount+"元");
        return channel+"支付"+amount+"员";
    }
}
```

```java
@Component("wechat")
public class WechatPayStrategyImpl implements IPayClient {
    @Override
    public Object pay(String channel, String amount) {
        System.out.println("微信支付："+amount+"元");
        return channel+"支付"+amount+"员";
    }
}
```

```java
@Component("xiaomi")
public class XiaomiPayStrategyImpl implements IPayClient {

    @Override
    public Object pay(String channel, String amount) {
        System.out.println("小米支付："+amount+"元");
        return channel+"支付"+amount+"员";
    }
}
```

结果

![image-20200810114346115](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200810114346115.png)

![image-20200810114356704](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200810114356704.png)

![image-20200810114446528](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200810114446528.png)

![image-20200810114458925](https://kulalasmile.oss-cn-hangzhou.aliyuncs.com/PicGoimage-20200810114458925.png)