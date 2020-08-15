## springboot2.X自定义注解实现简化校验

背景：因项目需要，自定义注解实现校验

------

1. 注解接口编写实现

```java
package com.***.ywsp.domain.anotation;
import static java.lang.annotation.ElementType.ANNOTATION_TYPE;
import static java.lang.annotation.ElementType.CONSTRUCTOR;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.TYPE_USE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Documented;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

/**
 * 手机座机校验注解
 * @author linzyc
 *
 */
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
@Repeatable(IsMobileTel.List.class)// 这里需要注意
@Documented
@Constraint(validatedBy = { MobileTelValidator.class })// 这里需要注意
public @interface IsMobileTel {

	String message() default "{mobile or telphone is invalid !!!}";

	Class<?>[] groups() default { };

	Class<? extends Payload>[] payload() default { };

	@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
	@Retention(RUNTIME)
	@Documented
	@interface List {
		IsMobileTel[] value();// 这里需要注意
	}
	
}

```

2. 校验类编写实现

```java
package com.***.ywsp.domain.anotation;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import com.***.ywsp.util.ValidatorUtil;

/**
 * 手机座机校验器
 * @author linzyc
 *
 */
public class MobileTelValidator implements ConstraintValidator<IsMobileTel, String>{
	
	@Override
	public void initialize(IsMobileTel constraintAnnotation) {// 这里注意
	}

	@Override
	public boolean isValid(String value, ConstraintValidatorContext context) {
        /**
        * 调用工具类校验
        */
		return ValidatorUtil.isMobileTel(value);
	}

}

```

3. 工具类编写实现

```java
package com.***.ywsp.util;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.apache.commons.lang3.StringUtils;

/**
 * 校验工具类，后续如有需要补充
 * @author linzyc
 *
 */
public class ValidatorUtil {
	
	private static final Pattern mobile_pattern = Pattern.compile("^[1][3,4,5,8][0-9]{9}$");
	
	private static final Pattern mobile_tel_pattern = Pattern.compile("^((0\\d{2,3}-\\d{7,8})|(1[3584]\\d{9}))$");

	/**
	 * 校验手机号
	 */
	public static boolean isMobile(String s) {
		if(StringUtils.isEmpty(s)) {
			return false;
		}
		Matcher m = mobile_pattern.matcher(s);
		return m.matches();
	}
	
	/**
	 * 校验手机号及座机
	 */
	public static boolean isMobileTel(String s) {
		if(StringUtils.isEmpty(s)) {
			return true;// 因考虑有的手机号非必录，这里针对空的直接不校验，外加@NotBlank控制
		}
		Matcher m = mobile_tel_pattern.matcher(s);
		return m.matches();
	}
}

```

