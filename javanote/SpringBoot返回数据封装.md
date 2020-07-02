# SpringBoot返回数据封装

## 新建一个CommonResult类

```java
/**
 * 通用返回对象
 *
 * @author : wangyufeng
 * @time : 2020-6-8
 */
public class CommonResult<T> {
    private Integer code;
    private String message;
    private T data;
    
    protected CommonResult(){

    }

    protected CommonResult(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    /**
     * 成功返回结果
     *
     * @param <T>  the type parameter
     * @param data the data
     * @return the common result
     */
    public static<T> CommonResult<T> success(T data) {
        return new CommonResult<T>(ResultCode.SUCCESS.getCode(), ResultCode.SUCCESS.getMessage(), data);
    }

    /**
     * 成功返回结果.
     *
     * @param <T>     the type parameter
     * @param data    获取的数据
     * @param message 提示消息
     * @return the common result
     */
    public static <T> CommonResult<T> success(T data, String message) {
        return new CommonResult<T>(ResultCode.SUCCESS.getCode(), message, data);
    }

    /**
     * 失败返回结果.
     *
     * @param <T>       the type parameter
     * @param errorCode 错误码
     * @return the common result
     */
    public static <T> CommonResult<T> failed(IErrorCode errorCode) {
        return new CommonResult<>(errorCode.getCode(), errorCode.getMessage(), null);
    }

    /**
     * 错误返回结果.
     *
     * @param message 提示信息
     */
    public static <T> CommonResult failed(String message) {
        return new CommonResult(ResultCode.FAILED.getCode(), message, null);
    }


    /**
     * 失败返回结果
     */
    public static <T> CommonResult<T> failed() {
        return failed(ResultCode.FAILED);
    }

    /**
     * 参数验证失败返回结果
     *
     * @param message 提示信息
     */
    public static <T> CommonResult<T> validateFailed(String message) {
        return new CommonResult<T>(ResultCode.VALIDATE_FAILED.getCode(), message, null);
    }

    /**
     * 未登录的返回结果
     *
     * @param data 数据
     */
    public static <T> CommonResult<T> unauthorized(T data) {
        return new CommonResult<T>(ResultCode.UNAUTHORIZED.getCode(), ResultCode.UNAUTHORIZED.getMessage(), data);
    }

    /**
     * 未授权返回结果
     *
     * @param data 数据
     */
    public static <T> CommonResult<T> forbidden(T data) {
        return new CommonResult<T>(ResultCode.FORBIDDEN.getCode(), ResultCode.FORBIDDEN.getMessage(), data);
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMessage(){
        return message;
    }

    public void setMessage(String message){
        this.message = message;
    }

    public T getData(){
        return data;
    }

    public void setData(){
        this.data = data;
    }

}

```

## 新建一个ResultCode枚举


```java
/**
 * 常用API操作码
 *
 * @author wangyufeng 
 * @date 2020-06-08
 */
public enum ResultCode implements IErrorCode {
    SUCCESS(200, "操作成功"),
    FAILED(500, "操作失败"),
    VALIDATE_FAILED(404, "参数校验失败"),
    UNAUTHORIZED(401, "暂未登录或token已过期"),
    FORBIDDEN(403, "权限不足");

    private Integer code;
    private String message;

    private ResultCode(Integer code, String message){
        this.code = code;
        this.message = message;
    }

    @Override
    public Integer getCode() {
        return code;
    }

    @Override
    public String getMessage() {
        return message;
    }
}

```
## 新建一个用来封装错误码的类

```java
/**
 * 封装API的错误码
 *
 * @author : wangyufeng 
 * @time : 2020-6-8
 */
public interface IErrorCode {
    /**
     * 获取错误码
     * @return the code
     */
    Integer getCode();

    /**
     * 获取错误信息
     */
    String getMessage();
}
```
