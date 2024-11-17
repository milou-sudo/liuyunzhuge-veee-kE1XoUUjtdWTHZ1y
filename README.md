v阅读目录* [v搭建架构](https://github.com)
* [v自定义验证码文本生成器](https://github.com)
* [v前后端验证码实现流程扩展](https://github.com)
* [v源码地址](https://github.com)
v博客前言


> [Kaptcha](https://github.com)是谷歌开源的一个可高度配置的比较老旧的实用验证码生成工具。它可以实现：(1\)验证码的字体/大小颜色；(2\)验证码内容的范围(数字，字母，中文汉字)；(3\)验证码图片的大小，边框，边框粗细，边框颜色(4\)验证码的干扰线验证码的样式(鱼眼样式、3D、 普通模糊)。



[回到顶部](https://github.com)## v搭建架构


 添加maven引用 


```
        <dependency>
            <groupId>com.github.pengglegroupId>
            <artifactId>kaptchaartifactId>
            <version>2.3.2version>
        dependency>
```


 创建Kaptcha配置类 


```
/**
 * @Author chen bo
 * @Date 2023/12
 * @Des
 */
@Configuration
public class KaptchaConfig {
    @Bean
    public DefaultKaptcha getDefaultKaptcha() {
        com.google.code.kaptcha.impl.DefaultKaptcha defaultKaptcha = new com.google.code.kaptcha.impl.DefaultKaptcha();
        Properties properties = new Properties();
        properties.put("kaptcha.border", "no");
        properties.put("kaptcha.textproducer.font.color", "black");
        properties.put("kaptcha.image.width", "200");
        properties.put("kaptcha.image.height", "50");
        properties.put("kaptcha.textproducer.font.size", "25");
        properties.put("kaptcha.session.key", "verifyCode");
        properties.put("kaptcha.textproducer.char.space", "5");
        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);

        return defaultKaptcha;
    }
}
```


此处配置的类可参考下方的配置表格：




| 常量 | 描述 | 默认值 |
| --- | --- | --- |
| kaptcha.border | 图片边框，合法值：yes , no | yes |
| kaptcha.border.color | 边框颜色，合法值： r,g,b (and optional alpha) 或者 white,black,blue. | black |
| kaptcha.border.thickness | 边框厚度，合法值：\>0 | 1 |
| kaptcha.image.width | 图片宽 | 200 |
| kaptcha.image.height | 图片高 | 50 |
| kaptcha.producer.impl | 图片实现类 | com.google.code.kaptcha.impl.DefaultKaptcha |
| kaptcha.textproducer.font.size | 文本实现类 | com.google.code.kaptcha.text.impl.DefaultTextCreator |
| kaptcha.textproducer.font.size | 字体大小 | 40px. |
| kaptcha.textproducer.font.color | 字体颜色，合法值： r,g,b 或者 white,black,blue. | black |
| kaptcha.textproducer.char.space | 文字间隔 | 2 |
| kaptcha.noise.impl | 干扰实现类 | com.google.code.kaptcha.impl.DefaultNoise |
| kaptcha.noise.color | 干扰 颜色，合法值： r,g,b 或者 white,black,blue. | black |
| kaptcha.obscurificator.impl | 图片样式： 水纹com.google.code.kaptcha.impl.WaterRipple 鱼眼com.google.code.kaptcha.impl.FishEyeGimpy 阴影com.google.code.kaptcha.impl.ShadowGimpy | com.google.code.kaptcha.impl.WaterRipple |
| kaptcha.background.impl | 背景实现类 | com.google.code.kaptcha.impl.DefaultBackground |
| kaptcha.background.clear.from | 背景颜色渐变，开始颜色 | light grey |
| kaptcha.background.clear.to | 背景颜色渐变， 结束颜色 | white |
| kaptcha.textproducer.char.length | 验证码长度 | 5 |


 创建controller 


```
/**
 * @Author chen bo
 * @Date 2023/12
 * @Des
 */
@RestController
@RequestMapping("/demo")
@Slf4j
public class ImageController {
    @Autowired
    private DefaultKaptcha defaultKaptcha;

    @RequestMapping(path = "/kaptcha", method = RequestMethod.GET)
    public void getKaptcha(HttpServletResponse response, HttpSession session) {
        String text = defaultKaptcha.createText();
        BufferedImage image = defaultKaptcha.createImage(text);
        // 线上环境这个验证码肯定是要存redis里的，redis的key还需要设置一个合理的过期时间
        session.setAttribute("kaptcha", text);
        response.setContentType("image/png");
        try {
            ServletOutputStream os = response.getOutputStream();
            ImageIO.write(image, "png", os);
        } catch (IOException e) {
            log.error("响应验证码失败:" + e.getMessage());
        }
    }

    @CrossOrigin
    @RequestMapping(path = "/login", method = RequestMethod.POST)
    public String login(HttpSession session,  String kaptcha) {
        if(kaptcha.equals(session.getAttribute("kaptcha"))){
            return kaptcha + "验证码正确";
        }else{
            return kaptcha + "验证码错误";
        }
    }
}
```


 添加登录页面 


```
DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Titletitle>
    <script>
        function refresh_kaptcha() {
            var path = "http://localhost:8301/demo/kaptcha?p=" + Math.random();
            document.getElementById("kaptcha").src=path;
        }

        function login() {
            var xhr = new XMLHttpRequest;

            xhr.open('post', 'http://localhost:8301/demo/login');
            // post请求的参数要放在send方法中作为参数的 - 必须的字符串
            // post请求要带参数必须在send之前设置 头信息
            xhr.setRequestHeader('content-type', 'application/x-www-form-urlencoded')
            // 数据在传送之前需要进行编码
            xhr.send('kaptcha=' + document.getElementById("kaptcha_value").value)
            xhr.onreadystatechange = function () {
                // 监听请求状态的变化 readystate (1-5 1准备发送 2 发送完成 3 发送完成数据准备接收 4数据
                if (xhr.readyState === 4) {
                    if (xhr.status >= 200 && xhr.status < 300) {
                        var res = xhr.responseText;
                        //res = JSON.parse(res)
                        alert(res);
                    }
                }
            }
        }
    script>
head>
<body>
<h3>请登录h3>
<div class="col-sm-4">
    <input type="text" placeholder="请输入用户名" name="username" required="required"/>
    <br/>
    <input type="password" placeholder="请输入密码" name="password" required="required"/>
    <br/>
    <span style="display: inline">
            <input type="text" name="请输入验证码" id="kaptcha_value" placeholder="验证码" required="required"/>
            <img src="http://localhost:8301/demo/kaptcha" id="kaptcha" style="width:100px;height:50px;" class="mr-2"/>
            <a href="javascript:refresh_kaptcha();" class="font-size-12 align-bottom">刷新验证码a>
    span>
    <br/>
    <button type="submit" onclick="login()">登录button>
div>
body>
html>
```


 效果图 
![请叫我头头哥](https://img2023.cnblogs.com/blog/506684/202310/506684-20231018200337589-1484066735.png)


[回到顶部](https://github.com)## v自定义验证码文本生成器


 创建自定义文本生成器 


```
/**
 * @Author chen bo
 * @Date 2023/12
 * @Des
 */
public class CustomTextCreator extends DefaultTextCreator {

    private static final String[] Number = "0,1,2,3,4,5,6,7,8,9,10".split(",");
    @Override
    public String getText()
    {
        int result;
        Random random = new Random();
        int x = random.nextInt(10);
        int y = random.nextInt(10);
        StringBuilder suChinese = new StringBuilder();
        int randomOperand = (int) Math.round(Math.random() * 2);
        if (randomOperand == 0) {
            result = x * y;
            suChinese.append(Number[x]);
            suChinese.append("*");
            suChinese.append(Number[y]);
        } else if (randomOperand == 1) {
            if (!(x == 0) && y % x == 0) {
                result = y / x;
                suChinese.append(Number[y]);
                suChinese.append("/");
                suChinese.append(Number[x]);
            } else {
                result = x + y;
                suChinese.append(Number[x]);
                suChinese.append("+");
                suChinese.append(Number[y]);
            }
        } else if (randomOperand == 2) {
            if (x >= y) {
                result = x - y;
                suChinese.append(Number[x]);
                suChinese.append("-");
                suChinese.append(Number[y]);
            } else {
                result = y - x;
                suChinese.append(Number[y]);
                suChinese.append("-");
                suChinese.append(Number[x]);
            }
        } else {
            result = x + y;
            suChinese.append(Number[x]);
            suChinese.append("+");
            suChinese.append(Number[y]);
        }
        suChinese.append("=?@").append(result);
        return suChinese.toString();
    }
}
```


 更新Kaptcha配置类 


```
/**
 * @Author chen bo
 * @Date 2023/12
 * @Des
 */
@Configuration
public class KaptchaConfig {
    //    @Bean
//    public DefaultKaptcha getDefaultKaptcha() {
//        com.google.code.kaptcha.impl.DefaultKaptcha defaultKaptcha = new com.google.code.kaptcha.impl.DefaultKaptcha();
//        Properties properties = new Properties();
//        properties.put("kaptcha.border", "no");
//        properties.put("kaptcha.textproducer.font.color", "black");
//        properties.put("kaptcha.image.width", "200");
//        properties.put("kaptcha.image.height", "50");
//        properties.put("kaptcha.textproducer.font.size", "25");
//        properties.put("kaptcha.session.key", "verifyCode");
//        properties.put("kaptcha.textproducer.char.space", "5");
//        Config config = new Config(properties);
//        defaultKaptcha.setConfig(config);
//
//        return defaultKaptcha;
//    }
    @Bean(name = "captchaProducerMath")
    public DefaultKaptcha getKaptchaBeanMath() {
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        Properties properties = new Properties();
        // 是否有边框 默认为true 我们可以自己设置yes，no
        properties.setProperty("kaptcha.border", "yes");
        // 边框颜色 默认为Color.BLACK
        properties.setProperty("kaptcha.border.color", "105,179,90");
        // 验证码文本字符颜色 默认为Color.BLACK
        properties.setProperty("kaptcha.textproducer.font.color", "blue");
        // 验证码图片宽度 默认为200
        properties.setProperty("kaptcha.image.width", "160");
        // 验证码图片高度 默认为50
        properties.setProperty("kaptcha.image.height", "60");
        // 验证码文本字符大小 默认为40
        properties.setProperty("kaptcha.textproducer.font.size", "35");
        // KAPTCHA_SESSION_KEY
        properties.setProperty("kaptcha.session.key", "kaptchaCodeMath");
        // 验证码文本生成器
        properties.setProperty("kaptcha.textproducer.impl", "com.kaptcha.demo.util.CustomTextCreator");
        // 验证码文本字符间距 默认为2
        properties.setProperty("kaptcha.textproducer.char.space", "3");
        // 验证码文本字符长度 默认为5
        properties.setProperty("kaptcha.textproducer.char.length", "6");
        // 验证码文本字体样式 默认为new Font("Arial", 1, fontSize), new Font("Courier", 1,
        // fontSize)
        properties.setProperty("kaptcha.textproducer.font.names", "Arial,Courier");
        // 验证码噪点颜色 默认为Color.BLACK
        properties.setProperty("kaptcha.noise.color", "white");
        // 干扰实现类
        properties.setProperty("kaptcha.noise.impl", "com.google.code.kaptcha.impl.NoNoise");
        // 图片样式 水纹com.google.code.kaptcha.impl.WaterRipple
        // 鱼眼com.google.code.kaptcha.impl.FishEyeGimpy
        // 阴影com.google.code.kaptcha.impl.ShadowGimpy
        properties.setProperty("kaptcha.obscurificator.impl", "com.google.code.kaptcha.impl.ShadowGimpy");
        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
}
```


 创建CustomController 


```
/**
 * @Author chen bo
 * @Date 2023/12
 * @Des
 */
@RestController
@Slf4j
@RequestMapping("/custom")
public class CustomController {

    @Autowired
    private Producer producer;

    public static final String DEFAULT_CODE_KEY = "random_code_";

    /**
     * @MethodName createCaptcha
     * @Description  生成验证码
     * @param httpServletResponse 响应流
     * @Author hl
     * @Date 2022/12/6 10:30
     */
    @GetMapping("/create")
    public void createCaptcha(HttpServletResponse httpServletResponse, HttpSession session) throws IOException {
        // 生成验证码
        String capText = producer.createText();
        String capStr = capText.substring(0, capText.lastIndexOf("@"));
        String result = capText.substring(capText.lastIndexOf("@") + 1);
        BufferedImage image = producer.createImage(capStr);
        // 保存验证码信息
        String randomStr = UUID.randomUUID().toString().replaceAll("-", "");
        System.out.println("随机数为:" + randomStr);
        //redisTemplate.opsForValue().set(DEFAULT_CODE_KEY + randomStr, result, 3600, TimeUnit.SECONDS);
        session.setAttribute("kaptcha", result);
        // 转换流信息写出
        FastByteArrayOutputStream os = new FastByteArrayOutputStream();
        try {
            ImageIO.write(image, "jpg", os);
        } catch (IOException e) {
            log.error("ImageIO write err", e);
            httpServletResponse.sendError(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        // 定义response输出类型为image/jpeg类型，使用response输出流输出图片的byte数组
        byte[] bytes = os.toByteArray();
        //设置响应头
        httpServletResponse.setHeader("Cache-Control", "no-store");
        //设置响应头
        httpServletResponse.setHeader("randomstr",randomStr);
        //设置响应头
        httpServletResponse.setHeader("Pragma", "no-cache");
        //在代理服务器端防止缓冲
        httpServletResponse.setDateHeader("Expires", 0);
        //设置响应内容类型
        ServletOutputStream responseOutputStream = httpServletResponse.getOutputStream();
        responseOutputStream.write(bytes);
        responseOutputStream.flush();
        responseOutputStream.close();
    }
}
```


 效果图 
![请叫我头头哥](https://img2023.cnblogs.com/blog/506684/202310/506684-20231018200429233-274152950.png)


![请叫我头头哥](https://img2023.cnblogs.com/blog/506684/202310/506684-20231018200437481-1402175427.png)


[回到顶部](https://github.com)## v前后端验证码实现流程扩展


1. 前端向后端请求验证码。
2. 后端通过谷歌开源工具Kaptcha生成图形验证码（实际是5个随机字符），缓存到redis，key键可以是 业务\+用户id，value值就是那5个随机字符。设置TTL为2分钟。
3. 后端将图形验证码转base64编码字符串，将该字符串返回给前端。
4. 前端解析base64编码的字符串，即可在页面上显示图形验证码。
5. 用户输入密码与验证码后提交表单到后端。
6. 后端根据业务和用户id组成的key键到redis查找缓存的验证码信息，会有如下情况：
	* 如果redis返回为空，则通知前端验证码失效，需要重新获取验证码。
	* 如果redis返回不为空，但是不相等，说明验证码输入错误。则删除redis中对应验证码缓存，通知前端验证码错误，需要重新获取验证码。
	* 如果redis返回不为空，并且相等，则校验成功，就删除redis中对应验证码缓存，并在mysql中修改密码。最后通知前端修改成功。


其他参考/学习资料：


* [https://github.com/dreamOfHua/p/3532776\.html](https://github.com):[milou加速器](https://xinminxuehui.org)
* [https://blog.csdn.net/weixin\_43296313/article/details/128207045](https://github.com)


[回到顶部](https://github.com)## v源码地址


[https://github.com/toutouge/javademosecond/tree/master/kaptcha\-demo](https://github.com/toutouge/javademosecond/tree/master/kaptcha-demo "请叫我头头哥")



 
 作　　者：**[请叫我头头哥](https://github.com/toutou/ "请叫我头头哥")**
 
 出　　处：[https://github.com/toutou/](https://github.com/toutou/ "请叫我头头哥")
 
 关于作者：专注于基础平台的项目开发。如有问题或建议，请多多赐教！
 
 版权声明：本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文链接。
 
 特此声明：所有评论和私信都会在第一时间回复。也欢迎园子的大大们指正错误，共同进步。或者[直接私信](http://msg.cnblogs.com/msg/send/%E8%AF%B7%E5%8F%AB%E6%88%91%E5%A4%B4%E5%A4%B4%E5%93%A5 "请叫我头头哥")我
 
 声援博主：如果您觉得文章对您有帮助，可以点击文章右下角**【推荐】**一下。您的鼓励是作者坚持原创和持续写作的最大动力！
 
 




![](http://images.cnblogs.com/cnblogs_com/toutou/682006/o_guanbi.png)* [搭建架构](https://github.com)
* [自定义验证码文本生成器](https://github.com)
* [前后端验证码实现流程扩展](https://github.com)
* [源码地址](https://github.com)
