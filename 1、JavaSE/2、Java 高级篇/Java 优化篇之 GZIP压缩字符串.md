# GZIP 实现压缩字符串

> Java-131-GZIP压缩字符串：https://mp.weixin.qq.com/s/8LHrjuxmXbWKuW9Wm9P0iA

今天看了一个压缩字符串的方式，直接使用的 Java 8 的 API 实现的。

HTTP 中有三种压缩：

- Zlib：常见类型有zip、rar和7z等。非常流行的文件压缩算法.在Linux 平台应用十分广泛。
- Deflate：更高的压缩率，7-zip就是实现的它，它还可以对gzip、PNG、MNG甚至Zip文件进行再次压缩从而得到比zlib压缩更小的大小。
- Gzip：一种无损压缩算法，其基础为Deflate,两者差不多。

我们在HTTP请求头中可以看到支持的压缩算法，支持哪种就会显示哪种：一般都是：Gzip 与 Deflate。

- Deflate 是 LZ77 与哈弗曼编码的一个组合体。
- 基本原理是：对于要压缩的文件，首先使用LZ77算法的一个变种进行压缩，对得到的结果再使用哈夫曼编码（根据情况，使用静态哈弗曼编码或动态哈夫曼编码）的方法进行压缩。

**什么是LZ77呢？**

- LZ77 严格意义上来说不是一种算法，而是一种编码理论。它的核心思路是如果一个串中有两个重复的串，那么只需要知道后面的串与前面串重复的长度和后面串起始字符与前面串起始字符相对于起始位置的距离。
- 来举个例子：比如字符串“不要啊不要啊停”，可以看到重复字符串有两个:“不要啊”。通过LZ77算法可压缩为“不要啊(4,3)停”，其中4表示重复的字符串起始字符(也就是第二个“不”)到第一个重复字符串起始字符(也就是第一个“不”)的距离，3表示重复字符串的长度。
  由此可见，对要压缩的字符串是有要求的，重复度越高的字符串，压缩效果是越好！我通过生成UUID字符串测试了，1000个字符它给我压缩成了950个，啊哈哈，这压了个毛！

**哈夫曼编码是什么呢？**

- 哈夫曼编码是数据结构课程中一种常见的算法。哈夫曼编码使用变长编码表对源符号进行编码，变长编码表通过一种评估来源符号出现概率的方法得到，出现概率较高的字母使用较短的编码，反之出现概率低的使用较长的编码，这样使编码之后的字符串的平均长度、期望值降低，从而达到无损压缩数据的目的。

下面是工具类代码：

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Base64;
import java.util.zip.GZIPInputStream;
import java.util.zip.GZIPOutputStream;

/**
 * 字符串压缩与解压缩
 */
public class CompressUtils {

    /**
     * 使用gzip压缩字符串
     *
     * @param str 要压缩的字符串
     * @return str
     */
    public static String compress(String str) {
        if (str == null || str.length() == 0) {
            return str;
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        GZIPOutputStream gzip = null;
        try {
            gzip = new GZIPOutputStream(out);
            gzip.write(str.getBytes(StandardCharsets.UTF_8));
        } catch (Exception e) {
            e.printStackTrace();
            return str;
        } finally {
            if (gzip != null) {
                try {
                    gzip.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return Base64.getEncoder().encodeToString(out.toByteArray());
    }

    /**
     * 使用gzip解压缩
     *
     * @param compressedStr 要解压缩的字符串
     * @return 解压后的字符串
     */
    public static String unCompress(String compressedStr) {
        if (compressedStr == null || compressedStr.length() == 0) {
            return compressedStr;
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ByteArrayInputStream inputStream = null;
        GZIPInputStream gzipInputStream = null;
        byte[] compressed;
        String decompressed;
        try {
            compressed = Base64.getDecoder().decode(compressedStr);
            inputStream = new ByteArrayInputStream(compressed);
            gzipInputStream = new GZIPInputStream(inputStream);
            byte[] buffer = new byte[1024];
            int offset = -1;
            while ((offset = gzipInputStream.read(buffer)) != -1) {
                out.write(buffer, 0, offset);
            }
            decompressed = out.toString();
        } catch (IOException e) {
            e.printStackTrace();
            return compressedStr;
        } finally {
            if (gzipInputStream != null) {
                try {
                    gzipInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            try {
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return decompressed;
    }

    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            // String str1 = CommonUtil.getRandomStr(1000);
            String str1 = "我自己的报文，你自己整一个";
            System.out.println("压缩前大小:" + str1.length());
            long t1 = System.currentTimeMillis();
            String compress = CompressUtils.compress(str1);
            System.out.println("压缩后大小:" + compress.length() + "|耗时:" + (System.currentTimeMillis() - t1) + "ms->");
            long t2 = System.currentTimeMillis();
            String unCompress = CompressUtils.unCompress(compress);
            System.out.println("解压后大小:" + unCompress.length() + "|耗时:" + (System.currentTimeMillis() - t2) + "ms->");
            System.out.println("对比压缩前后字符串:" + unCompress.equals(str1));
        }
    }
}
```

```
压缩前大小:13
压缩后大小:80|耗时:9ms->
解压后大小:13|耗时:7ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:1ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
压缩前大小:13
压缩后大小:80|耗时:0ms->
解压后大小:13|耗时:0ms->
对比压缩前后字符串:true
```

我们的一个单个请求报文有17K，压缩后就可以剩下5K，耗时也在可接受范围。



#  Gzip + Filter 压缩超大 JSON 对象

> SpringBoot + Filter 实现 Gzip 压缩超大 json 对象，传输耗时大大减少：https://mp.weixin.qq.com/s/lwCGR9d8q4couzz0NEb2qg

## 1、业务背景

是这样的，业务背景是公司的内部系统有一个广告保存接口，需要ADX那边将投放的广告数据进行保存供后续使用。广告数据大概长这样：

- adName：是广告名字

- adTag：是广告渲染的HTML代码，超级大数据库中都是用text类型来存放的，我看到最大的adTag足足有60kb大小

  ```json
  {
      "adName":"",
      "adTag":""
  }
  ```

因此，对于请求数据那么大的接口我们肯定是需要做一个优化的否则太大的数据传输有以下几个弊端：

- 占用网络带宽，而有些云产品就是按照带宽来计费的，间接浪费了钱
- 传输数据大导致网络传输耗时 为了克服这几个问题团队中的老鸟产生一个想法：

> 请求广告保存接口时先将JSON对象字符串进行GZIP压缩，那请求时传入的就是压缩后的数据，而GZIP的压缩效率是很高的，因此可以大大减小传输数据，而当数据到达广告保存接口前再将传来的数据进行解压缩，还原成JSON对象就完成了整个GZIP压缩数据的请求以及处理流程。

其实这样做也存在着弊端：

- 请求变复杂了

- - 接口调用方那边需要对数据进行压缩
  - 接口执行方那边需要对拿到的数据进行解压

- 需要额外占用更多的CPU计算资源

- 可能会影响到原有的其他接口

对于以上几点基于我们公司当前的业务可以这样解决：

- 对于需要占用而外的CPU计算资源来说，公司的内部系统属于IO密集型应用，因此用一些CPU资源来换取更快的网络传输其实是很划算的
- 使用过滤器在请求数据到达Controller之前对数据进行解压缩处理后重新写回到Body中，避免影响Controller的逻辑，代码零侵入
- 而对于改造接口的同时是否会影响到原来的接口这一点可以通过 HttpHeader 的Content-Encoding=gzip属性来区分是否需要对请求数据进行解压缩 那废话少说，下面给出实现方案



## 2、实现思路

前置知识：

- HTTP请求结构以及Content-Encoding 属性
- gzip压缩方式
- Servlet Filter
- HttpServletRequestWrapper
- Spring Boot
- Java 输入输出流

核心代码：

- 创建一个SpringBoot项目，先编写一个接口，功能很简单就是传入一个Json对象并返回，以模拟将广告数据保存到数据库

  ```java
  @Slf4j
  @RestController
  public class AdvertisingController {
  
      @PostMapping("/save")
      public Advertising saveProject(@RequestBody Advertising advertising) {
          log.info("获取内容"+ advertising);
          return advertising;
      }
  }
  
  @Data
  public class Advertising {
      private String adName;
      private String adTag;
  }
  ```

- 编写并注册一个拦截器

  ```java
  @Slf4j
  @Component
  public class GZIPFilter implements Filter {
  
      private static final String CONTENT_ENCODING = "Content-Encoding";
      private static final String CONTENT_ENCODING_TYPE = "gzip";
  
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
          log.info("init GZIPFilter");
      }
  
      @Override
      public void doFilter(ServletRequest servletRequest, 
                           ServletResponse servletResponse, 
                           FilterChain filterChain) throws IOException, ServletException {
          long start = System.currentTimeMillis();
          HttpServletRequest httpServletRequest = (HttpServletRequest)servletRequest;
  
          String encodeType = httpServletRequest.getHeader(CONTENT_ENCODING);
          if (CONTENT_ENCODING_TYPE.equals(encodeType)) {
              log.info("请求:{} 需要解压", httpServletRequest.getRequestURI());
              UnZIPRequestWrapper unZIPRequestWrapper = new UnZIPRequestWrapper(httpServletRequest);
              filterChain.doFilter(unZIPRequestWrapper,servletResponse);
          }
          else {
              log.info("请求:{} 无需解压", httpServletRequest.getRequestURI());
              filterChain.doFilter(servletRequest,servletResponse);
          }
          log.info("耗时：{}ms", System.currentTimeMillis() - start);
      }
  
      @Override
      public void destroy() {
          log.info("destroy GZIPFilter");
      }
  }
  
  @Configuration
  public class FilterRegistration {
  
      @Resource
      private GZIPFilter gzipFilter;
  
      @Bean
      public FilterRegistrationBean<GZIPFilter> gzipFilterRegistrationBean() {
          FilterRegistrationBean<GZIPFilter> registration = new FilterRegistrationBean<>();
          // Filter可以new，也可以使用依赖注入Bean
          registration.setFilter(gzipFilter);
          // 过滤器名称
          registration.setName("gzipFilter");
          // 拦截路径
          registration.addUrlPatterns("/*");
          // 设置顺序
          registration.setOrder(1);
          return registration;
      }
  }
  ```

- 实现RequestWrapper实现解压和写回Body的逻辑

  ```java
  /**
   * JsonString经过压缩后保存为二进制文件 -> 解压缩后还原成JsonString转换成byte[] 写回body中
   */
  @Slf4j
  public class UnZIPRequestWrapper extends HttpServletRequestWrapper {
  
      private final byte[] bytes;
  
      public UnZIPRequestWrapper(HttpServletRequest request) throws IOException {
          super(request);
          try (BufferedInputStream bis = new BufferedInputStream(request.getInputStream());
               ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
              final byte[] body;
              byte[] buffer = new byte[1024];
              int len;
              while ((len = bis.read(buffer)) > 0) {
                  baos.write(buffer, 0, len);
              }
              body = baos.toByteArray();
              if (body.length == 0) {
                  log.info("Body无内容，无需解压");
                  bytes = body;
                  return;
              }
              this.bytes = GZIPUtils.uncompressToByteArray(body);
          } catch (IOException ex) {
              log.info("解压缩步骤发生异常！");
              ex.printStackTrace();
              throw ex;
          }
      }
  
      @Override
      public ServletInputStream getInputStream() throws IOException {
          final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
          return new ServletInputStream() {
  
              @Override
              public boolean isFinished() {
                  return false;
              }
  
              @Override
              public boolean isReady() {
                  return false;
              }
  
              @Override
              public void setReadListener(ReadListener readListener) {
  
              }
  
              @Override
              public int read() throws IOException {
                  return byteArrayInputStream.read();
              }
          };
      }
  
      @Override
      public BufferedReader getReader() throws IOException {
          return new BufferedReader(new InputStreamReader(this.getInputStream()));
      }
  }
  ```

- 附上压缩工具类

  ```java
  public class GZIPUtils {
      public static final String GZIP_ENCODE_UTF_8 = "UTF-8";
  
      /**
       * 字符串压缩为GZIP字节数组
       * @param str
       * @return
       */
      public static byte[] compress(String str) {
          return compress(str, GZIP_ENCODE_UTF_8);
      }
  
      /**
       * 字符串压缩为GZIP字节数组
       * @param str
       * @param encoding
       * @return
       */
      public static byte[] compress(String str, String encoding) {
          if (str == null || str.length() == 0) {
              return null;
          }
          ByteArrayOutputStream out = new ByteArrayOutputStream();
          GZIPOutputStream gzip;
          try {
              gzip = new GZIPOutputStream(out);
              gzip.write(str.getBytes(encoding));
              gzip.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
          return out.toByteArray();
      }
  
      /**
       * GZIP解压缩
       * @param bytes
       * @return
       */
      public static byte[] uncompress(byte[] bytes) {
          if (bytes == null || bytes.length == 0) {
              return null;
          }
          ByteArrayOutputStream out = new ByteArrayOutputStream();
          ByteArrayInputStream in = new ByteArrayInputStream(bytes);
          try {
              GZIPInputStream ungzip = new GZIPInputStream(in);
              byte[] buffer = new byte[256];
              int n;
              while ((n = ungzip.read(buffer)) >= 0) {
                  out.write(buffer, 0, n);
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
          return out.toByteArray();
      }
  
      /**
       * 解压并返回String
       * @param bytes
       * @return
       */
      public static String uncompressToString(byte[] bytes) throws IOException {
          return uncompressToString(bytes, GZIP_ENCODE_UTF_8);
      }
  
      /**
       *
       * @param bytes
       * @return
       */
      public static byte[] uncompressToByteArray(byte[] bytes) throws IOException {
          return uncompressToByteArray(bytes, GZIP_ENCODE_UTF_8);
      }
  
      /**
       * 解压成字符串
       * @param bytes 压缩后的字节数组
       * @param encoding 编码方式
       * @return 解压后的字符串
       */
      public static String uncompressToString(byte[] bytes, String encoding) throws IOException {
          byte[] result = uncompressToByteArray(bytes, encoding);
          return new String(result);
      }
  
      /**
       * 解压成字节数组
       * @param bytes
       * @param encoding
       * @return
       */
      public static byte[] uncompressToByteArray(byte[] bytes, String encoding) throws IOException {
          if (bytes == null || bytes.length == 0) {
              return null;
          }
          ByteArrayOutputStream out = new ByteArrayOutputStream();
          ByteArrayInputStream in = new ByteArrayInputStream(bytes);
          try {
              GZIPInputStream ungzip = new GZIPInputStream(in);
              byte[] buffer = new byte[256];
              int n;
              while ((n = ungzip.read(buffer)) >= 0) {
                  out.write(buffer, 0, n);
              }
              return out.toByteArray();
          } catch (IOException e) {
              e.printStackTrace();
              throw new IOException("解压缩失败！");
          }
      }
  
      /**
       * 将字节流转换成文件
       * @param filename
       * @param data
       * @throws Exception
       */
      public static void saveFile(String filename,byte [] data)throws Exception{
          if(data != null){
              String filepath ="/" + filename;
              File file  = new File(filepath);
              if(file.exists()){
                  file.delete();
              }
              FileOutputStream fos = new FileOutputStream(file);
              fos.write(data,0,data.length);
              fos.flush();
              fos.close();
              System.out.println(file);
          }
      }
  ```



## 3、测试效果

注意一个大坑：千万不要直接将压缩后的`byte[]`当作字符串进行传输，否则你会发现压缩后的请求数据竟然比没压缩后的要大得多!一般有两种传输压缩后的`byte[]`的方式：

- 将压缩后的`byet[]`进行base64编码再传输字符串，这种方式会损失掉一部分GZIP的压缩效果，适用于压缩结果要存储在Redis中的情况
- 将压缩后的`byte[]`以二进制的形式写入到文件中，请求时直接在body中带上文件即可，用这种方式可以不损失压缩效果

Postman测试Gzip压缩数据请求：

- 请求头指定数据压缩方式：
- Body带上压缩后的`byte[]`写入的二进制文件：
- 执行请求，服务端正确处理了请求并且请求size缩小了将近一半，效果还是很不错的，这样GZIP压缩数据的请求的处理就完成了，完整的项目代码在下方：https://gitee.com/wx_1bceb446a4/gziptest

```HTTP
POST:localhost:8080/save
header:
Content-Type:application/json
Content-Encoding:gzip
binory(body type)
上传文件
```

