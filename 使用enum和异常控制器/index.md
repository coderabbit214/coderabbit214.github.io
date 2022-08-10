# 使用Enum和异常控制器


# 使用Enum和异常控制器

> 目的：避免定义各种异常类
>
> 基础：已经做了统一返回处理

1. Enum

   ```java
   public enum ResultCodeEnum {
   
       /**
        * 成功
        */
       SUCCESS(200, "操作成功"),
   
       /**
        * 系统错误
        */
       ERROR(500, "系统错误"),
   
       /**
        * 操作失败
        */
       FAILED(101, "操作失败"),
   
   
       /**
        * 参数错误
        */
       PARAM_ERROR(103, "参数错误"),
   
       /**
        * 参数错误-已存在
        */
       INVALID_PARAM_EXIST(104, "请求参数已存在"),
   
       /**
        * 参数错误
        */
       INVALID_PARAM_EMPTY(105, "请求参数为空"),
   
       /**
        * 参数错误
        */
       PARAM_TYPE_MISMATCH(106, "参数类型不匹配"),
   
       /**
        * 参数错误
        */
       PARAM_VALID_ERROR(107, "参数校验失败"),
   
       /**
        * 参数错误
        */
       ILLEGAL_REQUEST(108, "非法请求");
   
   
       public int code;
   
       public String msg;
   
       ResultCodeEnum(int code, String msg) {
           this.code = code;
           this.msg = msg;
       }
   
       public int getCode() {
           return code;
       }
   
       public void setCode(int code) {
           this.code = code;
       }
   
       public String getMsg() {
           return msg;
       }
   
       public void setMsg(String msg) {
           this.msg = msg;
       }
   }
   ```

2. 自定义业务异常类

   ```java
   public class BusinessException extends RuntimeException {
   
       private int code;
   
       private String msg;
   
       public BusinessException() {
           this.code = ResultCodeEnum.FAILED.code;
           this.msg = ResultCodeEnum.FAILED.msg;
       }
   
       public BusinessException(String message) {
           this.code = ResultCodeEnum.FAILED.code;
           this.msg = message;
       }
   
       public BusinessException(ResultCodeEnum resultCodeEnum){
           this.code = resultCodeEnum.code;
           this.msg = resultCodeEnum.msg;
       }
   
       public BusinessException(int code, String msg) {
           this.code = code;
           this.msg = msg;
       }
   
       public BusinessException(Throwable cause) {
           super(cause);
       }
   
       public BusinessException(String message, Throwable cause) {
           super(message, cause);
       }
   
       public String getMsg() {
           return msg;
       }
   
       public void setMsg(String msg) {
           this.msg = msg;
       }
   
       public int getCode() {
           return code;
       }
   
       public void setCode(int code) {
           this.code = code;
       }
   }
   ```

3. 异常控制器

   ```java
   @ControllerAdvice
   public class RestResultExceptionHandler {
   
       @ResponseBody
       @ExceptionHandler(value = RuntimeException.class)
       public RestResult<?> handleRunTimeException(RuntimeException e) {
           return RestResultUtils.failedWithMsg(500, e.getMessage());
       }
   
   
       /**
        * 自定义业务异常-BusinessException
        */
       @ResponseBody
       @ExceptionHandler(value = {BusinessException.class})
       public AjaxResult handlerCustomException(BusinessException exception) {
           return AjaxResult.error(exception.getCode(), exception.getMsg());
       }
   
   }
   ```

   

