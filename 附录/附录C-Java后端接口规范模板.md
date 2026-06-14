# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 附录C：Java 后端接口规范模板

> 以下代码基于 Spring Boot 框架，面向有 Java Web 开发基础的读者。

### C.1 统一响应格式 Result\<T\>

```java
package com.example.common.result;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import java.io.Serializable;

/**
 * 统一响应结果封装
 *
 * @param <T> 数据泛型
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result<T> implements Serializable {

    private static final long serialVersionUID = 1L;

    /** 状态码 */
    private Integer code;

    /** 提示信息 */
    private String message;

    /** 响应数据 */
    private T data;

    /** 时间戳 */
    private Long timestamp;

    // ==================== 成功响应 ====================

    public static <T> Result<T> success() {
        return success(null);
    }

    public static <T> Result<T> success(T data) {
        return success("操作成功", data);
    }

    public static <T> Result<T> success(String message, T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage(message);
        result.setData(data);
        result.setTimestamp(System.currentTimeMillis());
        return result;
    }

    // ==================== 失败响应 ====================

    public static <T> Result<T> error() {
        return error(500, "操作失败");
    }

    public static <T> Result<T> error(String message) {
        return error(500, message);
    }

    public static <T> Result<T> error(Integer code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        result.setTimestamp(System.currentTimeMillis());
        return result;
    }

    // ==================== 常用状态码常量 ====================

    public static final int SUCCESS = 200;
    public static final int BAD_REQUEST = 400;
    public static final int UNAUTHORIZED = 401;
    public static final int FORBIDDEN = 403;
    public static final int NOT_FOUND = 404;
    public static final int INTERNAL_ERROR = 500;

    /**
     * 判断是否成功
     */
    public boolean isSuccess() {
        return SUCCESS == this.code;
    }
}
```

**响应示例：**

```json
// 成功响应
{
    "code": 200,
    "message": "操作成功",
    "data": {
        "id": 1,
        "name": "张三"
    },
    "timestamp": 1700000000000
}

// 失败响应
{
    "code": 400,
    "message": "参数错误：用户名不能为空",
    "data": null,
    "timestamp": 1700000000000
}
```

---

### C.2 分页响应格式 PageResult\<T\>

```java
package com.example.common.result;

import lombok.Data;
import java.io.Serializable;
import java.util.List;

/**
 * 分页响应结果封装
 *
 * @param <T> 数据泛型
 */
@Data
public class PageResult<T> implements Serializable {

    private static final long serialVersionUID = 1L;

    /** 当前页码 */
    private Long pageNum;

    /** 每页条数 */
    private Long pageSize;

    /** 总记录数 */
    private Long total;

    /** 总页数 */
    private Long totalPages;

    /** 当前页数据 */
    private List<T> records;

    /** 是否有下一页 */
    private Boolean hasNext;

    /** 是否有上一页 */
    private Boolean hasPrevious;

    /**
     * 构建分页结果（配合 MyBatis-Plus 的 IPage 使用）
     */
    public static <T> PageResult<T> of(Long pageNum, Long pageSize, Long total, List<T> records) {
        PageResult<T> result = new PageResult<>();
        result.setPageNum(pageNum);
        result.setPageSize(pageSize);
        result.setTotal(total);
        result.setRecords(records);
        result.setTotalPages((total + pageSize - 1) / pageSize);
        result.setHasNext(pageNum < result.getTotalPages());
        result.setHasPrevious(pageNum > 1);
        return result;
    }

    /**
     * 从 MyBatis-Plus IPage 转换
     */
    public static <T> PageResult<T> of(com.baomidou.mybatisplus.core.metadata.IPage<T> page) {
        PageResult<T> result = new PageResult<>();
        result.setPageNum(page.getCurrent());
        result.setPageSize(page.getSize());
        result.setTotal(page.getTotal());
        result.setRecords(page.getRecords());
        result.setTotalPages(page.getPages());
        result.setHasNext(page.getCurrent() < page.getPages());
        result.setHasPrevious(page.getCurrent() > 1);
        return result;
    }
}
```

**分页响应示例：**

```json
{
    "code": 200,
    "message": "操作成功",
    "data": {
        "pageNum": 1,
        "pageSize": 10,
        "total": 55,
        "totalPages": 6,
        "records": [
            { "id": 1, "name": "张三" },
            { "id": 2, "name": "李四" }
        ],
        "hasNext": true,
        "hasPrevious": false
    },
    "timestamp": 1700000000000
}
```

---

### C.3 全局异常处理 GlobalExceptionHandler

```java
package com.example.common.exception;

import com.example.common.result.Result;
import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindException;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.multipart.MaxUploadSizeExceededException;

import javax.servlet.http.HttpServletRequest;
import java.util.stream.Collectors;

/**
 * 全局异常处理器
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    // ==================== 自定义业务异常 ====================

    @ExceptionHandler(BusinessException.class)
    public Result<?> handleBusinessException(BusinessException e, HttpServletRequest request) {
        log.warn("业务异常 [{}]: {}", request.getRequestURI(), e.getMessage());
        return Result.error(e.getCode(), e.getMessage());
    }

    // ==================== 参数校验异常 ====================

    /**
     * 处理 @Valid 校验异常（@RequestBody 参数）
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<?> handleValidException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining("; "));
        log.warn("参数校验失败: {}", message);
        return Result.error(Result.BAD_REQUEST, message);
    }

    /**
     * 处理 @Validated 校验异常（表单参数）
     */
    @ExceptionHandler(BindException.class)
    public Result<?> handleBindException(BindException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining("; "));
        log.warn("参数绑定失败: {}", message);
        return Result.error(Result.BAD_REQUEST, message);
    }

    // ==================== 文件上传异常 ====================

    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public Result<?> handleMaxUploadSizeExceededException(MaxUploadSizeExceededException e) {
        log.warn("文件大小超限: {}", e.getMessage());
        return Result.error(Result.BAD_REQUEST, "文件大小超出限制");
    }

    // ==================== 其他异常 ====================

    @ExceptionHandler(Exception.class)
    public Result<?> handleException(Exception e, HttpServletRequest request) {
        log.error("系统异常 [{}]: ", request.getRequestURI(), e);
        return Result.error(Result.INTERNAL_ERROR, "系统繁忙，请稍后重试");
    }
}
```

**自定义业务异常类：**

```java
package com.example.common.exception;

import lombok.Getter;

/**
 * 自定义业务异常
 */
@Getter
public class BusinessException extends RuntimeException {

    private final Integer code;

    public BusinessException(String message) {
        super(message);
        this.code = 400;
    }

    public BusinessException(Integer code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(String message, Throwable cause) {
        super(message, cause);
        this.code = 400;
    }
}
```

**使用示例：**

```java
@Service
public class UserServiceImpl implements UserService {

    @Override
    public UserVO getUserById(Long id) {
        User user = userMapper.selectById(id);
        if (user == null) {
            throw new BusinessException("用户不存在");
        }
        return convertToVO(user);
    }
}
```

---

### C.4 JWT 工具类 JwtUtil

```java
package com.example.common.util;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.Map;

/**
 * JWT 工具类
 */
@Slf4j
@Component
public class JwtUtil {

    /** 密钥（建议配置在 application.yml 中，至少 256 位） */
    @Value("${jwt.secret:mySecretKeyForJwtTokenGenerationMustBeLongEnough123456}")
    private String secret;

    /** 过期时间（毫秒），默认 7 天 */
    @Value("${jwt.expiration:604800000}")
    private Long expiration;

    /** 生成密钥 */
    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
    }

    /**
     * 生成 JWT Token
     *
     * @param subject   主题（通常是用户ID）
     * @param claims    自定义声明
     * @return JWT Token
     */
    public String generateToken(String subject, Map<String, Object> claims) {
        Date now = new Date();
        Date expireDate = new Date(now.getTime() + expiration);

        return Jwts.builder()
                .claims(claims)                    // 自定义声明
                .subject(subject)                  // 主题
                .issuedAt(now)                     // 签发时间
                .expiration(expireDate)            // 过期时间
                .signWith(getSigningKey())         // 签名
                .compact();
    }

    /**
     * 生成 JWT Token（简化版）
     */
    public String generateToken(String subject) {
        return generateToken(subject, null);
    }

    /**
     * 解析 JWT Token
     *
     * @param token JWT Token
     * @return Claims
     */
    public Claims parseToken(String token) {
        try {
            return Jwts.parser()
                    .verifyWith(getSigningKey())
                    .build()
                    .parseSignedClaims(token)
                    .getPayload();
        } catch (ExpiredJwtException e) {
            log.warn("JWT Token 已过期: {}", e.getMessage());
            throw new BusinessException(401, "Token 已过期，请重新登录");
        } catch (UnsupportedJwtException e) {
            log.warn("不支持的 JWT Token: {}", e.getMessage());
            throw new BusinessException(401, "无效的 Token");
        } catch (MalformedJwtException e) {
            log.warn("格式错误的 JWT Token: {}", e.getMessage());
            throw new BusinessException(401, "无效的 Token");
        } catch (SecurityException e) {
            log.warn("JWT Token 签名验证失败: {}", e.getMessage());
            throw new BusinessException(401, "无效的 Token");
        } catch (IllegalArgumentException e) {
            log.warn("JWT Token 参数非法: {}", e.getMessage());
            throw new BusinessException(401, "无效的 Token");
        }
    }

    /**
     * 从 Token 中获取用户ID（subject）
     */
    public String getUserId(String token) {
        return parseToken(token).getSubject();
    }

    /**
     * 验证 Token 是否有效
     */
    public boolean validateToken(String token) {
        try {
            parseToken(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 判断 Token 是否过期
     */
    public boolean isTokenExpired(String token) {
        try {
            Claims claims = parseToken(token);
            return claims.getExpiration().before(new Date());
        } catch (ExpiredJwtException e) {
            return true;
        }
    }
}
```

**JWT 拦截器：**

```java
package com.example.common.interceptor;

import com.example.common.util.JwtUtil;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * JWT 认证拦截器
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class JwtAuthInterceptor implements HandlerInterceptor {

    private final JwtUtil jwtUtil;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        // 放行 OPTIONS 请求（预检请求）
        if ("OPTIONS".equalsIgnoreCase(request.getMethod())) {
            return true;
        }

        String token = request.getHeader("Authorization");
        if (token == null || !token.startsWith("Bearer ")) {
            sendError(response, 401, "未登录或 Token 缺失");
            return false;
        }

        token = token.substring(7);
        if (!jwtUtil.validateToken(token)) {
            sendError(response, 401, "Token 无效或已过期");
            return false;
        }

        // 将用户ID存入请求属性
        String userId = jwtUtil.getUserId(token);
        request.setAttribute("userId", userId);

        return true;
    }

    private void sendError(HttpServletResponse response, int code, String message) {
        response.setStatus(code);
        response.setContentType("application/json;charset=UTF-8");
        try {
            response.getWriter().write(
                String.format("{\"code\":%d,\"message\":\"%s\",\"data\":null}", code, message)
            );
        } catch (Exception e) {
            log.error("写入错误响应失败", e);
        }
    }
}
```

**拦截器注册：**

```java
package com.example.config;

import com.example.common.interceptor.JwtAuthInterceptor;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@RequiredArgsConstructor
public class WebMvcConfig implements WebMvcConfigurer {

    private final JwtAuthInterceptor jwtAuthInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtAuthInterceptor)
                .addPathPatterns("/api/**")              // 拦截路径
                .excludePathPatterns(                    // 排除路径
                        "/api/auth/login",
                        "/api/auth/register",
                        "/api/public/**",
                        "/api/doc.html",
                        "/api/swagger-resources/**"
                );
    }
}
```

---

### C.5 跨域配置 CorsConfig

```java
package com.example.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 跨域配置
 *
 * 注意：如果使用了 Spring Security，还需要在 SecurityConfig 中配置 CORS
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                // 允许的源（生产环境应配置具体域名）
                .allowedOriginPatterns("*")
                // 允许的请求方法
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH")
                // 允许的请求头
                .allowedHeaders("*")
                // 是否允许携带凭证（Cookie）
                .allowCredentials(true)
                // 预检请求缓存时间（秒）
                .maxAge(3600);
    }
}
```

> **小程序开发注意：** 小程序端发起的 `wx.request` 不受浏览器同源策略限制，因此跨域配置主要是为了方便前端 H5 调试和 Swagger 文档访问。

---

### C.6 接口文档规范（Swagger 注解使用示例）

#### Maven 依赖

```xml
<!-- SpringDoc OpenAPI（Spring Boot 3.x） -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>

<!-- 或使用 Knife4j 增强（国内推荐） -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.3.0</version>
</dependency>
```

#### Controller 层注解示例

```java
package com.example.controller;

import com.example.common.result.PageResult;
import com.example.common.result.Result;
import com.example.entity.dto.UserLoginDTO;
import com.example.entity.dto.UserRegisterDTO;
import com.example.entity.vo.UserVO;
import com.example.service.UserService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.Parameters;
import io.swagger.v3.oas.annotations.tags.Tag;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@Tag(name = "用户管理", description = "用户注册、登录、信息管理等接口")
@RestController
@RequestMapping("/api/user")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @Operation(summary = "用户登录", description = "通过微信 code 登录或账号密码登录")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "登录成功",
            content = @Content(schema = @Schema(implementation = UserVO.class))),
        @ApiResponse(responseCode = "401", description = "认证失败"),
        @ApiResponse(responseCode = "400", description = "参数错误")
    })
    @PostMapping("/login")
    public Result<UserVO> login(@Valid @RequestBody UserLoginDTO loginDTO) {
        UserVO userVO = userService.login(loginDTO);
        return Result.success(userVO);
    }

    @Operation(summary = "用户注册")
    @PostMapping("/register")
    public Result<Void> register(@Valid @RequestBody UserRegisterDTO registerDTO) {
        userService.register(registerDTO);
        return Result.success();
    }

    @Operation(summary = "获取用户信息")
    @Parameter(name = "id", description = "用户ID", required = true, example = "1")
    @GetMapping("/{id}")
    public Result<UserVO> getUserById(@PathVariable Long id) {
        UserVO userVO = userService.getUserById(id);
        return Result.success(userVO);
    }

    @Operation(summary = "分页查询用户列表")
    @Parameters({
        @Parameter(name = "pageNum", description = "页码", example = "1"),
        @Parameter(name = "pageSize", description = "每页条数", example = "10"),
        @Parameter(name = "keyword", description = "搜索关键词")
    })
    @GetMapping("/list")
    public Result<PageResult<UserVO>> getUserList(
            @RequestParam(defaultValue = "1") Long pageNum,
            @RequestParam(defaultValue = "10") Long pageSize,
            @RequestParam(required = false) String keyword) {
        PageResult<UserVO> page = userService.getUserList(pageNum, pageSize, keyword);
        return Result.success(page);
    }

    @Operation(summary = "上传头像")
    @PostMapping("/avatar")
    public Result<String> uploadAvatar(
            @Parameter(description = "头像文件", required = true)
            @RequestParam("file") MultipartFile file) {
        String avatarUrl = userService.uploadAvatar(file);
        return Result.success(avatarUrl);
    }

    @Operation(summary = "更新用户信息")
    @PutMapping("/{id}")
    public Result<Void> updateUser(@PathVariable Long id, @RequestBody UserVO userVO) {
        userService.updateUser(id, userVO);
        return Result.success();
    }

    @Operation(summary = "删除用户")
    @Parameter(name = "id", description = "用户ID", required = true)
    @DeleteMapping("/{id}")
    public Result<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return Result.success();
    }
}
```

#### DTO / VO 注解示例

```java
package com.example.entity.dto;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.*;
import lombok.Data;

@Schema(description = "用户登录请求")
@Data
public class UserLoginDTO {

    @Schema(description = "微信登录 code", example = "0a1B2c3D")
    private String code;

    @Schema(description = "手机号", example = "13800138000")
    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;

    @Schema(description = "密码", example = "123456")
    @Size(min = 6, max = 20, message = "密码长度为 6-20 位")
    private String password;
}

@Schema(description = "用户注册请求")
@Data
public class UserRegisterDTO {

    @Schema(description = "用户名", example = "张三", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "用户名不能为空")
    @Size(max = 20, message = "用户名不能超过 20 个字符")
    private String username;

    @Schema(description = "手机号", example = "13800138000", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "手机号不能为空")
    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;

    @Schema(description = "密码", example = "123456", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "密码不能为空")
    @Size(min = 6, max = 20, message = "密码长度为 6-20 位")
    private String password;

    @Schema(description = "确认密码", example = "123456")
    private String confirmPassword;
}

@Schema(description = "用户信息响应")
@Data
public class UserVO {

    @Schema(description = "用户ID", example = "1")
    private Long id;

    @Schema(description = "用户名", example = "张三")
    private String username;

    @Schema(description = "头像URL", example = "https://example.com/avatar/1.jpg")
    private String avatar;

    @Schema(description = "手机号", example = "138****8000")
    private String phone;

    @Schema(description = "注册时间", example = "2024-01-01 12:00:00")
    private String createTime;
}
```

#### Swagger 配置

```java
package com.example.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import io.swagger.v3.oas.models.Components;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("微信小程序后端 API 文档")
                        .description("基于 Spring Boot 3.x 的微信小程序后端接口文档")
                        .version("1.0.0")
                        .contact(new Contact()
                                .name("开发团队")
                                .email("dev@example.com")))
                // 全局 JWT 认证
                .addSecurityItem(new SecurityRequirement().addList("Bearer Token"))
                .components(new Components()
                        .addSecuritySchemes("Bearer Token",
                                new SecurityScheme()
                                        .type(SecurityScheme.Type.HTTP)
                                        .scheme("bearer")
                                        .bearerFormat("JWT")
                                        .description("输入 JWT Token")));
    }
}
```

#### application.yml 配置

```yaml
# application.yml
springdoc:
  swagger-ui:
    path: /swagger-ui.html
    tags-sorter: alpha
    operations-sorter: method
  api-docs:
    path: /v3/api-docs

# Knife4j 配置（如果使用 Knife4j）
knife4j:
  enable: true
  setting:
    language: zh_cn
```

---

