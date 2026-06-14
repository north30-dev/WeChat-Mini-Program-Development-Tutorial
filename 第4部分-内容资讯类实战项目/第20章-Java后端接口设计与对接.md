# 微信小程序开发入门教程

> **适用读者**：具有 Java Web 开发经验（Spring Boot / Spring MVC）的后端开发者
> **学习目标**：从零掌握微信小程序开发，能够独立完成内容资讯类小程序项目
> **教程特色**：Java 视角对比、完整实战项目、企业级开发规范

---

## 第20章 Java后端接口设计与对接

本章将搭建完整的 Spring Boot 后端，为小程序提供 RESTful API。对于 Java 开发者来说，这是最熟悉的部分——Controller、Service、Mapper 三层架构。

### 20.1 Spring Boot 项目搭建

**pom.xml 核心依赖**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.18</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>news-app-server</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- MyBatis-Plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.5</version>
        </dependency>

        <!-- MySQL -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.11.5</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Hutool工具类 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.25</version>
        </dependency>

        <!-- OkHttp（用于调用微信API） -->
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>4.12.0</version>
        </dependency>
    </dependencies>
</project>
```

**application.yml 配置**：

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/news_app?useUnicode=true&characterEncoding=utf8mb4&serverTimezone=Asia/Shanghai
    username: root
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# 微信小程序配置
wechat:
  appid: your_appid
  secret: your_secret

# JWT配置
jwt:
  secret: your_jwt_secret_key_at_least_256_bits_long_for_security
  expiration: 604800  # 7天（秒）
```

### 20.2 统一响应格式

```java
package com.example.newsapp.common;

import lombok.Data;
import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;

/**
 * 统一API响应格式
 * 类比：前端 Axios 拦截器中判断的 response.data 结构
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Result<T> {

    private int code;
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        return new Result<>(0, "success", data);
    }

    public static <T> Result<T> success() {
        return new Result<>(0, "success", null);
    }

    public static <T> Result<T> error(int code, String message) {
        return new Result<>(code, message, null);
    }

    public static <T> Result<T> error(String message) {
        return new Result<>(-1, message, null);
    }
}
```

```java
package com.example.newsapp.common;

import lombok.Data;
import java.util.List;

/**
 * 分页响应对象
 * 与前端 request.js 中 result.records / result.total 对应
 */
@Data
public class PageResult<T> {
    private List<T> records;
    private long total;
    private int page;
    private int pageSize;

    public PageResult(List<T> records, long total, int page, int pageSize) {
        this.records = records;
        this.total = total;
        this.page = page;
        this.pageSize = pageSize;
    }
}
```

### 20.3 跨域配置

```java
package com.example.newsapp.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 跨域配置
 * 注意：小程序线上环境不受浏览器同源策略限制，但开发工具调试时需要
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOriginPatterns("*")
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }
}
```

> **Java开发者注意点**：小程序的 `wx.request` 在真机上不受浏览器同源策略限制（它不是在浏览器中运行的），但在微信开发者工具中调试时，底层使用的仍然是 Chromium 内核，因此需要配置 CORS。另外，小程序要求后端必须配置 **HTTPS** 和 **域名白名单**（在微信公众平台 -> 开发 -> 开发管理 -> 服务器域名中配置），开发阶段可以在开发者工具中勾选"不校验合法域名"。

### 20.4 JWT 工具类

```java
package com.example.newsapp.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long expiration;

    private SecretKey key;

    @PostConstruct
    public void init() {
        this.key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
    }

    /**
     * 生成JWT Token
     * @param userId 用户ID
     * @param openid 微信openid
     */
    public String generateToken(Long userId, String openid) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", userId);
        claims.put("openid", openid);

        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + expiration * 1000))
                .signWith(key)
                .compact();
    }

    /**
     * 解析Token
     */
    public Claims parseToken(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    /**
     * 从Token中获取用户ID
     */
    public Long getUserId(String token) {
        Claims claims = parseToken(token);
        return Long.valueOf(claims.get("userId").toString());
    }

    /**
     * 判断Token是否过期
     */
    public boolean isTokenExpired(String token) {
        try {
            Claims claims = parseToken(token);
            return claims.getExpiration().before(new Date());
        } catch (Exception e) {
            return true;
        }
    }
}
```

### 20.5 认证拦截器

```java
package com.example.newsapp.interceptor;

import com.example.newsapp.util.JwtUtil;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.Map;

/**
 * JWT认证拦截器
 * 类比：Spring Security的Filter，但更轻量
 */
@Slf4j
@Component
public class AuthInterceptor implements HandlerInterceptor {

    @Autowired
    private JwtUtil jwtUtil;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        // OPTIONS预检请求直接放行
        if ("OPTIONS".equalsIgnoreCase(request.getMethod())) {
            return true;
        }

        String token = request.getHeader("Authorization");
        if (token != null && token.startsWith("Bearer ")) {
            token = token.substring(7);
        }

        if (token == null || token.isEmpty() || jwtUtil.isTokenExpired(token)) {
            response.setStatus(401);
            response.setContentType("application/json;charset=UTF-8");
            Map<String, Object> result = new HashMap<>();
            result.put("code", 401);
            result.put("message", "未授权，请先登录");
            new ObjectMapper().writeValue(response.getOutputStream(), result);
            return false;
        }

        // 将用户ID存入请求属性
        Long userId = jwtUtil.getUserId(token);
        request.setAttribute("userId", userId);

        return true;
    }
}
```

```java
package com.example.newsapp.config;

import com.example.newsapp.interceptor.AuthInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private AuthInterceptor authInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor)
                .addPathPatterns("/api/**")            // 拦截所有API
                .excludePathPatterns(                   // 排除不需要登录的接口
                        "/api/auth/login",
                        "/api/articles/list",
                        "/api/articles/detail/*",
                        "/api/articles/search",
                        "/api/articles/banners",
                        "/api/categories"
                );
    }
}
```

### 20.6 实体类

```java
package com.example.newsapp.entity;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

@Data
@TableName("articles")
public class Article {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String title;
    private String summary;
    private String content;       // HTML格式
    private String coverUrl;
    private Long categoryId;
    private String author;
    private Integer viewCount;
    private Integer likeCount;
    private Integer commentCount;
    private Boolean isTop;
    private Boolean isBanner;
    private Integer status;       // 0-草稿 1-已发布 2-下架
    private LocalDateTime publishedAt;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;
}

@Data
@TableName("users")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String openid;
    private String nickname;
    private String avatarUrl;
    private String phone;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;
}

@Data
@TableName("comments")
public class Comment {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long articleId;
    private Long userId;
    private Long parentId;
    private Long replyUserId;
    private String content;
    private Integer likeCount;
    private Integer status;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
}

@Data
@TableName("favorites")
public class Favorite {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long userId;
    private Long articleId;
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
}
```

### 20.7 核心Controller实现

#### AuthController —— 用户登录

```java
package com.example.newsapp.controller;

import com.example.newsapp.common.Result;
import com.example.newsapp.entity.User;
import com.example.newsapp.mapper.UserMapper;
import com.example.newsapp.util.JwtUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Value("${wechat.appid}")
    private String appid;

    @Value("${wechat.secret}")
    private String secret;

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private JwtUtil jwtUtil;

    @Autowired
    private ObjectMapper objectMapper;

    /**
     * 微信小程序登录
     * 流程：小程序wx.login()获取code -> 后端用code换取openid -> 查询/创建用户 -> 返回JWT
     */
    @PostMapping("/login")
    public Result<Map<String, Object>> login(@RequestBody LoginRequest request) {
        String code = request.getCode();
        if (code == null || code.isEmpty()) {
            return Result.error("code不能为空");
        }

        try {
            // 1. 调用微信 code2Session 接口
            String url = String.format(
                "https://api.weixin.qq.com/sns/jscode2session?appid=%s&secret=%s&js_code=%s&grant_type=authorization_code",
                appid, secret, code
            );

            OkHttpClient client = new OkHttpClient();
            Request httpRequest = new Request.Builder().url(url).build();
            Response httpResponse = client.newCall(httpRequest).execute();
            String body = httpResponse.body().string();
            JsonNode jsonNode = objectMapper.readTree(body);

            String openid = jsonNode.get("openid").asText();
            String sessionKey = jsonNode.get("session_key").asText();

            if (openid == null || openid.isEmpty()) {
                return Result.error("微信登录失败：" + jsonNode.get("errmsg").asText());
            }

            // 2. 查询或创建用户
            User user = userMapper.selectOne(
                new LambdaQueryWrapper<User>().eq(User::getOpenid, openid)
            );
            if (user == null) {
                user = new User();
                user.setOpenid(openid);
                user.setNickname("微信用户");
                userMapper.insert(user);
            }

            // 3. 生成JWT Token
            String token = jwtUtil.generateToken(user.getId(), user.getOpenid());

            // 4. 返回结果
            Map<String, Object> data = new HashMap<>();
            data.put("token", token);
            Map<String, String> userInfo = new HashMap<>();
            userInfo.put("nickname", user.getNickname());
            userInfo.put("avatarUrl", user.getAvatarUrl());
            data.put("userInfo", userInfo);

            return Result.success(data);

        } catch (Exception e) {
            log.error("微信登录异常", e);
            return Result.error("登录失败：" + e.getMessage());
        }
    }

    @Data
    static class LoginRequest {
        private String code;
    }
}
```

> **Java开发者注意点**：微信登录的核心流程是 `code2Session`——小程序端调用 `wx.login()` 获取临时 `code`，后端用 `code` + `appid` + `secret` 调用微信服务器换取 `openid` 和 `session_key`。这类似于 OAuth2 的授权码模式，但更简单。**注意**：`session_key` 非常重要（用于数据解密），不要返回给前端，也不要泄露。

#### NewsController —— 文章接口

```java
package com.example.newsapp.controller;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.newsapp.common.PageResult;
import com.example.newsapp.common.Result;
import com.example.newsapp.entity.Article;
import com.example.newsapp.mapper.ArticleMapper;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/articles")
public class NewsController {

    @Autowired
    private ArticleMapper articleMapper;

    /**
     * 获取文章列表（分页 + 分类筛选）
     * 对应前端：pages/index/index.js 的 loadArticles()
     *
     * GET /api/articles/list?page=1&pageSize=10&categoryId=1
     */
    @GetMapping("/list")
    public Result<PageResult<Article>> list(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int pageSize,
            @RequestParam(required = false) Long categoryId) {

        Page<Article> pageParam = new Page<>(page, pageSize);
        LambdaQueryWrapper<Article> wrapper = new LambdaQueryWrapper<>();

        // 只查询已发布的文章
        wrapper.eq(Article::getStatus, 1);

        // 分类筛选
        if (categoryId != null && categoryId > 0) {
            wrapper.eq(Article::getCategoryId, categoryId);
        }

        // 置顶优先，然后按发布时间倒序
        wrapper.orderByDesc(Article::getIsTop)
               .orderByDesc(Article::getPublishedAt);

        // 排除正文内容（列表不需要）
        wrapper.select(Article.class, info -> !info.getColumn().equals("content"));

        Page<Article> result = articleMapper.selectPage(pageParam, wrapper);

        return Result.success(new PageResult<>(
            result.getRecords(),
            result.getTotal(),
            (int) result.getCurrent(),
            (int) result.getSize()
        ));
    }

    /**
     * 获取文章详情
     * 对应前端：pages/detail/detail.js 的 loadArticle()
     *
     * GET /api/articles/detail/123
     */
    @GetMapping("/detail/{id}")
    public Result<Article> detail(@PathVariable Long id) {
        Article article = articleMapper.selectById(id);
        if (article == null || article.getStatus() != 1) {
            return Result.error("文章不存在");
        }

        // 阅读量 +1
        article.setViewCount(article.getViewCount() + 1);
        articleMapper.updateById(article);

        return Result.success(article);
    }

    /**
     * 搜索文章
     * 对应前端：pages/search/search.js 的 doSearch()
     *
     * GET /api/articles/search?keyword=Spring&page=1&pageSize=20
     */
    @GetMapping("/search")
    public Result<PageResult<Article>> search(
            @RequestParam String keyword,
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "20") int pageSize) {

        if (StringUtils.isBlank(keyword)) {
            return Result.error("搜索关键词不能为空");
        }

        Page<Article> pageParam = new Page<>(page, pageSize);
        LambdaQueryWrapper<Article> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(Article::getStatus, 1)
               .and(w -> w
                   .like(Article::getTitle, keyword)
                   .or()
                   .like(Article::getSummary, keyword)
               )
               .orderByDesc(Article::getPublishedAt);

        // 搜索结果也排除正文
        wrapper.select(Article.class, info -> !info.getColumn().equals("content"));

        Page<Article> result = articleMapper.selectPage(pageParam, wrapper);

        return Result.success(new PageResult<>(
            result.getRecords(),
            result.getTotal(),
            (int) result.getCurrent(),
            (int) result.getSize()
        ));
    }

    /**
     * 获取Banner推荐文章
     * 对应前端：pages/index/index.js 的 loadBanners()
     *
     * GET /api/articles/banners
     */
    @GetMapping("/banners")
    public Result<List<Article>> banners() {
        LambdaQueryWrapper<Article> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(Article::getStatus, 1)
               .eq(Article::getIsBanner, true)
               .orderByDesc(Article::getPublishedAt)
               .last("LIMIT 5");

        // Banner只需要部分字段
        wrapper.select(Article::getId, Article::getTitle,
                       Article::getCoverUrl, Article::getSummary);

        List<Article> banners = articleMapper.selectList(wrapper);
        return Result.success(banners);
    }
}
```

> **Java开发者注意点**：`wrapper.select(Article.class, info -> !info.getColumn().equals("content"))` 是 MyBatis-Plus 的字段排除语法，在列表查询中排除 `content` 大字段，避免传输冗余数据。这类似于 JPA 的 `@Query("SELECT new ...")` 或原生 SQL 中只 SELECT 需要的列。前端 `request.js` 中 `resolve(res.data.data)` 取的就是 `Result<T>` 中 `data` 字段的值。

#### CommentController —— 评论接口

```java
package com.example.newsapp.controller;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.example.newsapp.common.Result;
import com.example.newsapp.entity.Comment;
import com.example.newsapp.entity.User;
import com.example.newsapp.mapper.CommentMapper;
import com.example.newsapp.mapper.UserMapper;
import lombok.Data;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import java.util.*;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/comments")
public class CommentController {

    @Autowired
    private CommentMapper commentMapper;

    @Autowired
    private UserMapper userMapper;

    /**
     * 获取评论列表（树形结构）
     * 对应前端：pages/detail/detail.js 的 loadComments()
     *
     * GET /api/comments/123
     */
    @GetMapping("/{articleId}")
    public Result<List<CommentVO>> list(@PathVariable Long articleId) {
        LambdaQueryWrapper<Comment> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(Comment::getArticleId, articleId)
               .eq(Comment::getStatus, 1)
               .orderByDesc(Comment::getCreatedAt);

        List<Comment> allComments = commentMapper.selectList(wrapper);

        // 构建用户Map
        Set<Long> userIds = allComments.stream()
            .map(c -> {
                HashSet<Long> ids = new HashSet<>();
                ids.add(c.getUserId());
                if (c.getReplyUserId() != null) ids.add(c.getReplyUserId());
                return ids;
            })
            .reduce(new HashSet<>(), (a, b) -> { a.addAll(b); return a; });

        Map<Long, User> userMap = new HashMap<>();
        if (!userIds.isEmpty()) {
            userMapper.selectBatchIds(userIds).forEach(
                u -> userMap.put(u.getId(), u)
            );
        }

        // 组装VO并构建树
        List<CommentVO> voList = allComments.stream()
            .map(c -> toVO(c, userMap))
            .collect(Collectors.toList());

        List<CommentVO> tree = buildTree(voList);
        return Result.success(tree);
    }

    /**
     * 发布评论
     * 对应前端：pages/detail/detail.js 的 submitComment()
     *
     * POST /api/comments
     */
    @PostMapping
    public Result<Void> create(@RequestBody CommentRequest request,
                               HttpServletRequest httpRequest) {
        Long userId = (Long) httpRequest.getAttribute("userId");

        Comment comment = new Comment();
        comment.setArticleId(request.getArticleId());
        comment.setUserId(userId);
        comment.setContent(request.getContent());
        comment.setParentId(request.getParentId());
        comment.setReplyUserId(request.getReplyUserId());
        comment.setLikeCount(0);
        comment.setStatus(1);

        commentMapper.insert(comment);

        // 更新文章评论数
        // （此处省略ArticleMapper的updateCommentCount调用）

        return Result.success();
    }

    /**
     * 点赞评论
     */
    @PostMapping("/like/{id}")
    public Result<Void> like(@PathVariable Long id) {
        Comment comment = commentMapper.selectById(id);
        if (comment != null) {
            comment.setLikeCount(comment.getLikeCount() + 1);
            commentMapper.updateById(comment);
        }
        return Result.success();
    }

    // --- 私有方法 ---

    private CommentVO toVO(Comment c, Map<Long, User> userMap) {
        CommentVO vo = new CommentVO();
        vo.setId(c.getId());
        vo.setContent(c.getContent());
        vo.setLikeCount(c.getLikeCount());
        vo.setCreatedAt(c.getCreatedAt().toString());

        User user = userMap.get(c.getUserId());
        vo.setNickname(user != null ? user.getNickname() : "匿名用户");
        vo.setUserAvatar(user != null ? user.getAvatarUrl() : "");

        if (c.getReplyUserId() != null) {
            User replyUser = userMap.get(c.getReplyUserId());
            vo.setReplyNickname(replyUser != null ? replyUser.getNickname() : "");
        }

        return vo;
    }

    private List<CommentVO> buildTree(List<CommentVO> all) {
        Map<Long, List<CommentVO>> childrenMap = all.stream()
            .filter(c -> c.getParentId() != null)
            .collect(Collectors.groupingBy(CommentVO::getParentId));

        return all.stream()
            .filter(c -> c.getParentId() == null)
            .peek(c -> c.setChildren(childrenMap.getOrDefault(c.getId(), new ArrayList<>())))
            .collect(Collectors.toList());
    }

    // --- DTO ---

    @Data
    static class CommentRequest {
        private Long articleId;
        private String content;
        private Long parentId;
        private Long replyUserId;
    }

    @Data
    static class CommentVO {
        private Long id;
        private Long parentId;
        private String nickname;
        private String userAvatar;
        private String replyNickname;
        private String content;
        private Integer likeCount;
        private String createdAt;
        private List<CommentVO> children;
    }
}
```

#### FavoriteController —— 收藏接口

```java
package com.example.newsapp.controller;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.example.newsapp.common.Result;
import com.example.newsapp.entity.Article;
import com.example.newsapp.entity.Favorite;
import com.example.newsapp.mapper.ArticleMapper;
import com.example.newsapp.mapper.FavoriteMapper;
import lombok.Data;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/favorites")
public class FavoriteController {

    @Autowired
    private FavoriteMapper favoriteMapper;

    @Autowired
    private ArticleMapper articleMapper;

    /**
     * 添加收藏
     */
    @PostMapping("/add")
    public Result<Void> add(@RequestBody FavoriteRequest request,
                            HttpServletRequest httpRequest) {
        Long userId = (Long) httpRequest.getAttribute("userId");

        // 检查是否已收藏
        Long count = favoriteMapper.selectCount(
            new LambdaQueryWrapper<Favorite>()
                .eq(Favorite::getUserId, userId)
                .eq(Favorite::getArticleId, request.getArticleId())
        );
        if (count > 0) {
            return Result.success(); // 已收藏，直接返回
        }

        Favorite favorite = new Favorite();
        favorite.setUserId(userId);
        favorite.setArticleId(request.getArticleId());
        favoriteMapper.insert(favorite);

        return Result.success();
    }

    /**
     * 取消收藏
     */
    @PostMapping("/remove")
    public Result<Void> remove(@RequestBody FavoriteRequest request,
                               HttpServletRequest httpRequest) {
        Long userId = (Long) httpRequest.getAttribute("userId");

        favoriteMapper.delete(
            new LambdaQueryWrapper<Favorite>()
                .eq(Favorite::getUserId, userId)
                .eq(Favorite::getArticleId, request.getArticleId())
        );

        return Result.success();
    }

    /**
     * 检查是否已收藏
     */
    @GetMapping("/check/{articleId}")
    public Result<Boolean> check(@PathVariable Long articleId,
                                 HttpServletRequest httpRequest) {
        Long userId = (Long) httpRequest.getAttribute("userId");

        Long count = favoriteMapper.selectCount(
            new LambdaQueryWrapper<Favorite>()
                .eq(Favorite::getUserId, userId)
                .eq(Favorite::getArticleId, articleId)
        );

        return Result.success(count > 0);
    }

    /**
     * 获取收藏列表
     */
    @GetMapping("/list")
    public Result<List<FavoriteVO>> list(HttpServletRequest httpRequest) {
        Long userId = (Long) httpRequest.getAttribute("userId");

        List<Favorite> favorites = favoriteMapper.selectList(
            new LambdaQueryWrapper<Favorite>()
                .eq(Favorite::getUserId, userId)
                .orderByDesc(Favorite::getCreatedAt)
        );

        List<FavoriteVO> voList = favorites.stream().map(f -> {
            FavoriteVO vo = new FavoriteVO();
            vo.setId(f.getId());
            vo.setArticleId(f.getArticleId());
            vo.setCreatedAt(f.getCreatedAt().toString());

            Article article = articleMapper.selectById(f.getArticleId());
            if (article != null) {
                vo.setTitle(article.getTitle());
                vo.setCoverUrl(article.getCoverUrl());
            }
            return vo;
        }).collect(Collectors.toList());

        return Result.success(voList);
    }

    /**
     * 获取收藏数量
     */
    @GetMapping("/count")
    public Result<Long> count(HttpServletRequest httpRequest) {
        Long userId = (Long) httpRequest.getAttribute("userId");
        Long count = favoriteMapper.selectCount(
            new LambdaQueryWrapper<Favorite>().eq(Favorite::getUserId, userId)
        );
        return Result.success(count);
    }

    @Data
    static class FavoriteRequest {
        private Long articleId;
    }

    @Data
    static class FavoriteVO {
        private Long id;
        private Long articleId;
        private String title;
        private String coverUrl;
        private String createdAt;
    }
}
```

### 20.8 分类接口

```java
package com.example.newsapp.controller;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.example.newsapp.common.Result;
import com.example.newsapp.entity.Category;
import com.example.newsapp.mapper.CategoryMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/api/categories")
public class CategoryController {

    @Autowired
    private CategoryMapper categoryMapper;

    @GetMapping
    public Result<List<Category>> list() {
        List<Category> categories = categoryMapper.selectList(
            new LambdaQueryWrapper<Category>()
                .orderByAsc(Category::getSortOrder)
        );
        return Result.success(categories);
    }
}
```

> **Java开发者注意点**：以上所有 Controller 的路径都以 `/api` 开头，这与前端 `request.js` 中 `BASE_URL = 'https://your-domain.com/api'` 对应。后端接口返回的 `Result<T>` 结构 `{ code: 0, message: "success", data: {...} }` 与前端 `request.js` 中 `res.data.code === 0` 的判断逻辑一一对应。这种契约式的接口设计，与你在 Spring Boot 中为 Web 前端编写 REST API 完全一致。

---

