# Gitee登录替代微博登录

1、微博开放平台审核身份认证时间过长，改用Gitee登录作为第三方登录

2、预先准备

2-1、在 Gitee 的设置-数据管理-第三方应用中创建应用，设置应用回调地址为 http://auth.gulimall.com/oauth2/gitee/success

2-2、在 ums_member 表中增加字段

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1698892463965-d23fefe2-5c1a-488e-bc09-2b7c4c30e798.png)

2-3、在 Nacos 的 gulimall-auth-server 模块的配置文件中添加相关配置

```java
gitee:
  oauth:
    client-id: 
    client-secret: 
    redirect-uri: http://auth.gulimall.com/oauth2/gitee/success
```

2-4、在 gulimall-common 中创建 SocialUserTo 实体类用于远程调用

```java
package com.atguigu.common.to;
import lombok.Data;
@Data
public class SocialUserTo {
    private String accessToken;
    private String tokenType;
    private Long expiresIn;
    private String refreshToken;
    private String scope;
    private Long createdAt;
    private String id;
    private String login; //用户用户名
    private String name; //用户昵称
    private String  avatarUrl;
    private String bio; //用户自我介绍
    private String email;
}
```

2-5、编写远程调用接口

```java
@PostMapping("/member/member/oauth2/login")
R login(@RequestBody SocialUserTo to);
```

3、参考 https://gitee.com/api/v5/oauth_doc，修改登录页面 login.html

```java
<li>
    <a href="https://gitee.com/oauth/authorize?client_id={client_id}&redirect_uri={redirect_uri}&response_type=code">
        <img style="width: 25px; height: 25px" src="/static/login/JD_img/gitee.png"/>
        <span>Gitee</span>
    </a>
</li>
```

3-1、client_id 在创建应用后自动生成，redirect_uri 为应用回调地址

3-2、图标可以直接存入 nginx 中，也可以使用官网图片 https://gitee.com/static/images/logo-black.svg

4、生成 accessToken 后，会根据应用回调地址调用 gulimall-auth-server 模块中的 gitee() 方法

```java
@Controller
public class OAuth2Controller { 
    @Value("${gitee.oauth.client-id}")
    private String clientId;
    @Value("${gitee.oauth.client-secret}")
    private String clientSecret;
    @Value("${gitee.oauth.redirect-uri}")
    private String redirectUri;
    @Autowired
    private MemberFeignService memberFeignService;

    @GetMapping("/oauth2/gitee/success")
    public String gitee(@RequestParam("code") String code) throws Exception {
        Map<String, String> headers = new HashMap<>();
        Map<String, String> querys = new HashMap<>();
        querys.put("grant_type", "authorization_code");
        querys.put("code", code);
        querys.put("client_id", clientId);
        querys.put("redirect_uri", redirectUri);
        querys.put("client_secret", clientSecret);
        Map<String, String> bodys = new HashMap<>();
        HttpResponse responsePost = HttpUtils.doPost("https://gitee.com", "/oauth/token", "POST", headers, querys, bodys);
        if(responsePost.getStatusLine().getStatusCode() == 200){
            String json = EntityUtils.toString(responsePost.getEntity());
            SocialUserTo socialUserTo = JSON.parseObject(json, SocialUserTo.class);
            if(socialUserTo != null && StringUtils.hasLength(socialUserTo.getAccessToken())){
                querys.clear();
                querys.put("access_token", socialUserTo.getAccessToken());
                HttpResponse responseGet = HttpUtils.doGet("https://gitee.com", "/api/v5/user", "GET", headers, querys);
                if(responseGet.getStatusLine().getStatusCode() == 200){
                    json = EntityUtils.toString(responseGet.getEntity());
                    SocialUserTo user = JSON.parseObject(json, SocialUserTo.class);
                    BeanUtils.copyProperties(user, socialUserTo, "accessToken", "tokenType", "expiresIn", "refreshToken", "scope", "createdAt");
                    R r = memberFeignService.login(socialUserTo);
                    if(r.getCode() == 0){
                        json = JSON.toJSONString(r.get("data"));
                        MemberRespVo vo = JSON.parseObject(json, MemberRespVo.class);
                        return "redirect:http://gulimall.com";
                    }
                }
            }
        }
        return "redirect:http://auth.gulimall.com/login.html";
    }
}
```

4-1、@Value("${}") 中的配置名必须与配置文件中的完全一致，否则无法正常解析

4-2、HttpUtils.doPost()、HttpUtils.doGet() 中的第一个参数为 host，必须写完整地址，否则报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1698893615933-5538d504-8cd6-457f-8978-709605adebee.png)

4-3、因为向 Gitee 申请令牌时，返回的信息中并不包含用户id，参考 https://gitee.com/api/v5/swagger#/getV5User 获取授权用户资料

4-4、socialUserTo 中为令牌相关信息，user 中为用户相关信息，在合并两个对象时，BeanUtils.copyProperties() 携带的 String 参数为忽略的参数，避免信息被覆盖为 null

5、远程调用 gulimall-member 中的方法

```java
@PostMapping("/oauth2/login")
public R login(@RequestBody SocialUserTo to){
    MemberEntity member = memberService.login(to);
    if(member != null){
        return R.ok().put("data", member);
    }
    return R.error(BizCodeEnum.LOGIN_EXCEPTION.getCode(), BizCodeEnum.LOGIN_EXCEPTION.getMsg());
}

@Override
public MemberEntity login(SocialUserTo to) {
    String uid = to.getId();
    MemberEntity member = this.baseMapper.selectOne(new QueryWrapper<MemberEntity>().eq("social_uid", uid));
    if(member == null){ 
        member = new MemberEntity();
        member.setUsername(to.getLogin());
        member.setNickname(to.getName());
        member.setEmail(to.getEmail());
        member.setHeader(to.getAvatarUrl());
        member.setSign(to.getBio());
        member.setSocialUid(uid);
        MemberLevelEntity memberLevelEntity = memberLevelDao.getDefaultLevel();
        member.setLevelId(memberLevelEntity.getId());
        this.baseMapper.insert(member);
    }
    redisTemplate.opsForValue().set(MemberConstant.OAUTH_GITEE + uid, to.getAccessToken(), to.getExpiresIn(), TimeUnit.SECONDS);
    return member;
}
```

5-1、因为 Gitee 设置令牌过期时间为1天，因此选择将令牌放在 Redis 而非数据库中，需要提前在 Nacos 中配置 Redis 信息，否则报错

![img](https://cdn.nlark.com/yuque/0/2023/png/2836791/1698895028441-0a2a4c9f-3881-4d6f-a2c2-bdd17492d0f3.png)











































