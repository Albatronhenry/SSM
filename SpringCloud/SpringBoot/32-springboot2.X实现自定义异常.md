# springboot2.X实现自定义异常以及@Valid等注解校验实现

背景：统一异常处理，方便前端友好展示；表单校验无需额外判断表单字段

------

## 1. 自定义异常-全局异常注解

> @ControllerAdvice 
>
> @RestControllerAdvice
>
> @ExceptionHandler(Exception.class)

### 1.1 全局异常处理类 CustomExceptionAdvice

```java
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import com.***.result.Result;
import com.***.result.ResultUtil;

import lombok.extern.slf4j.Slf4j;

/**
 * 全局异常处理
 *
 */
@ControllerAdvice
@Slf4j
public class CustomExceptionAdvice {

	@ResponseBody
	@ExceptionHandler(Exception.class)
	public Result customExceptionHandler(Exception e) {
		log.error(">>>>>>>>>>>>>>>>>>自定义异常处理，异常信息：{}", e);
		if(e instanceof BaseException) {
			return ResultUtil.exception(e);
		} else if(e instanceof MethodArgumentNotValidException) {
            // MethodArgumentNotValidException @Valid @notBlank等注解校验异常
	        String errorMsg = getValidMsg(e); 
			return  ResultUtil.error(-1, errorMsg);
		}else {
			return ResultUtil.error("系统异常，请联系管理员");
		}
	}

	public String getValidMsg(Exception e) {
		BindingResult result = ((MethodArgumentNotValidException) e).getBindingResult();
		StringBuilder errorMsg = new StringBuilder() ;	        
		if (result.hasErrors()) {
		    List<FieldError> fieldErrors = result.getFieldErrors();
		    // 每次选取第一个，以便提示更友好
		    FieldError error = fieldErrors.get(0);
		    errorMsg.append(error.getDefaultMessage()).append("!");		   
		}
		return errorMsg.toString();
	}
}

```

### 1.2 BaseException  异常基类

```java
package com.***.common.exception;

import com.***.common.result.ResultEnums;

import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 * 异常基类--继承RuntimeException
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class BaseException extends RuntimeException {

	private static final long serialVersionUID = 9023331168150884791L;
	private Integer code;
	private String msg;
	private Object data;

	public BaseException(ResultEnums status) {
		super(status.getMsg());
		this.code = status.getCode();
		this.msg = status.getMsg();
	}

	public BaseException(ResultEnums status, Object data) {
		this(status);
		this.data = data;
	}

	public BaseException(Integer code, String msg) {
		super(msg);
		this.code = code;
		this.msg = msg;
	}

	public BaseException(Integer code, String msg, Object data) {
		this(code, msg);
		this.data = data;
	}

	public BaseException(String msg) {
		this(-1, msg);
	}

}
```

### 1.3 ResultUtil 封装结果集工具类

```java
package com.***.common.result;

import com.***.common.exception.BaseException;

/**
 * 返回工具类
 */
public class ResultUtil {

    public static Result success() {
        return success(ResultEnums.SUCCESS);
    }

    public static Result success(Object obj) {
        return new Result(ResultEnums.SUCCESS, obj);
    }

    public static Result success(ResultEnums enums) {
        return new Result(enums, null);
    }

    public static Result success(ResultEnums enums, Object obj) {
        return new Result(enums, obj);
    }
    
    public static Result error(Integer code, String msg, Object obj) {
        return new Result(code, msg, obj);
    }

    public static Result error(Integer code, String msg) {
        return new Result(code, msg);
    }

    public static Result error(String msg) {
        return error(-1, msg);
    }
    
    public static Result exception(Exception e) {
    	if(e instanceof BaseException) {
    		return error(-1, ((BaseException) e).getMsg(), e);
    	} else {
    		return unknow();
    	}
    }
    
    public static Result unknow() {
    	return new Result(-1, "系统异常");
    }

}
```

### 1.4 Result 返回结果类

```java
package com.***.common.result;

import lombok.Data;
import lombok.ToString;

import java.io.Serializable;

/**
 * 返回结果
 */
@Data
@ToString
public class Result implements Serializable {

	private static final long serialVersionUID = -2405501630340981356L;

	private Integer code;

    private String msg;

    private Object data;

    private Long timestamp;

    public Result(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
        this.data = null;
        this.timestamp = System.currentTimeMillis();
    }
    
    public Result(Integer code, String msg, Object obj) {
        this.code = code;
        this.msg = msg;
        this.data = obj;
        this.timestamp = System.currentTimeMillis();
    }

    public Result(ResultEnums enums, Object data) {
        this.code = enums.getCode();
        this.msg = enums.getMsg();
        this.data = data;
        this.timestamp = System.currentTimeMillis();
    }
    
    public Boolean ok() {
    	return code == 200;
    }
    
}
```

### 1.5 ResultEnums 通用状态码

```java
package com.***.common.result;

import lombok.Getter;

/**
 * 通用状态码
 */
@Getter
public enum ResultEnums{
    /**
     * 操作成功！
     */
    SUCCESS(200, "操作成功！"),

    /**
     * 操作异常！
     */
    ERROR(500, "操作异常！"),

    /**
     * 退出成功！
     */
    LOGOUT(200, "退出成功！"),

    /**
     * 请先登录！
     */
    UNAUTHORIZED(401, "请先登录！"),

    /**
     * 暂无权限访问！
     */
    ACCESS_DENIED(403, "权限不足！"),

    /**
     * 请求不存在！
     */
    REQUEST_NOT_FOUND(404, "请求不存在！"),

    /**
     * 请求方式不支持！
     */
    HTTP_BAD_METHOD(405, "请求方式不支持！"),

    /**
     * 请求异常！
     */
    BAD_REQUEST(400, "请求异常！"),

    /**
     * 参数不匹配！
     */
    PARAM_NOT_MATCH(400, "参数不匹配！"),

    /**
     * 参数不能为空！
     */
    PARAM_NOT_NULL(400, "参数不能为空！"),

    /**
     * 当前用户已被锁定，请联系管理员解锁！
     */
    USER_DISABLED(403, "当前用户已被锁定，请联系管理员解锁！"),

    /**
     * 用户名或密码错误！
     */
    USERNAME_PASSWORD_ERROR(5001, "用户名或密码错误！"),

    /**
     * token 已过期，请重新登录！
     */
    TOKEN_EXPIRED(5002, "token 已过期，请重新登录！"),

    /**
     * token 解析失败，请尝试重新登录！
     */
    TOKEN_PARSE_ERROR(5002, "token 解析失败，请尝试重新登录！"),

    /**
     * 当前用户已在别处登录，请尝试更改密码或重新登录！
     */
    TOKEN_OUT_OF_CTRL(5003, "当前用户已在别处登录，请尝试更改密码或重新登录！"),

    /**
     * 无法手动踢出自己，请尝试退出登录操作！
     */
    KICKOUT_SELF(5004, "无法手动踢出自己，请尝试退出登录操作！"),

    INCONSISTENT_DATA_VERSION(5003, "数据版本不一致，请刷新页面重试");

    /**
     * 状态码
     */
    private Integer code;

    /**
     * 返回信息
     */
    private String msg;

    ResultEnums(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public static ResultEnums fromCode(Integer code) {
        ResultEnums[] statuses = ResultEnums.values();
        for (ResultEnums status : statuses) {
            if (status.getCode()
                    .equals(code)) {
                return status;
            }
        }
        return SUCCESS;
    }

    @Override
    public String toString() {
        return String.format(" ResultEnums:{code=%s, msg=%s} ", getCode(), getMsg());
    }

}
```



## 2. @Valid @isBlank @isEmpty @isNull等注解使用

### 2.1 注解解释

> @Valid 确保下面注解生效   @Validation
>
> @isBlank  作用于String类型，其他类型可能会实效
>
> @isEmpty 作用于集合
>
> @isNull      作用于基础类型

### 2.2  SaveBaseForm注解类

```java
package com.***.domain.form.acc;

import javax.validation.constraints.NotBlank;

import com.***.domain.form.BaseForm;

import lombok.Data;
import lombok.EqualsAndHashCode;

/**
 *  表单保存基类
 */
@Data
@EqualsAndHashCode(callSuper=false)
public class SaveBaseForm extends BaseForm{
	
	/**
	 * 单据ID，打开申请页面已加载获取
	 */
	@NotNull(message = "单据id不能为空")
	private long id;
	
	/**
	 *  单位名称
	 */
	@NotBlank(message = "单位不能为空")
	private String enName;
	
	/**
	 *  联系人
	 */
    @isEmpty(message = "联系人不能为空")
	private List<String> contactName;
		
}

```

### 2.3 项目使用为 Controller 对应方法上加 @Valid 注解使之生效

```java
/**
	 * 保存/更新
	 * @param form
	 * @return
	 */
	@PostMapping("/save")
	public Result save(@RequestBody @Valid AccSaveBaseForm form) {
    }
```

