> SpringBoot读取Resources文件：https://blog.csdn.net/myli92/article/details/129890193

SpringBoot读取文件一般分为两种，一种是获取文件对象（File/URL）另一种是获取文件的流。

# 方式 1： getClass.getResource

```java
private void wayByFile() {
    try {
        URL url = this.getClass().getResource("../../../test.txt");
        String fileName = url.getFile().replaceFirst("/", "");
        // 解决中文目录和文件名乱码问题
        fileName = URLDecoder.decode(fileName, "utf-8");
        List<String> content = Files.readAllLines(Paths.get(fileName));
        System.out.println("方式1 文件内容:" + content);
    } catch (Exception e) {
        System.out.println("方式1 错误:" + e);
    }
}
```



# 方式 2： ResourceUtils 类

org.springframework.util.ResourceUtils

```java
private void way2ByFile() {
    try {
        File file = ResourceUtils.getFile("classpath:test.txt");
        FileInputStream fileInputStream = new FileInputStream(file);
        InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream);
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        System.out.println("方式2 文件内容:" + bufferedReader.readLine());
    } catch (Exception e) {
        System.out.println("方式2 错误:" + e);
    }
}
```



# 方式 3： getClass.getResourceAsStream

```java
private void way3ByStream() {
    try {
        InputStream fileInputStream = ReadResourceApplication.class.getResourceAsStream("../../../test.txt");
        InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream);
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        System.out.println("方式3 文件内容:" + bufferedReader.readLine());
    } catch (Exception e) {
        System.out.println("方式3 错误:" + e);
    }
}
```



# 方式 4： ClassPathResource 类

org.springframework.core.io.ClassPathResource

```java
private void way4ByStream() {
    try {
        ClassPathResource classPathResource = new ClassPathResource("test.txt");
        InputStreamReader inputStreamReader = new InputStreamReader(classPathResource.getInputStream());
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        System.out.println("方式4 文件内容:" + bufferedReader.readLine());
    } catch (Exception e) {
        System.out.println("方式4 错误:" + e);
    }
}
```



# 方式 5： Resource 接口

结合Spring注解，使用org.springframework.core.io.Resource

```java
@Value("test.txt")
Resource resource;

private void way5ByStream() {
    try {
        InputStreamReader inputStreamReader = new InputStreamReader(resource.getInputStream());
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        System.out.println("方式5 文件内容:" + bufferedReader.readLine());
    } catch (Exception e) {
        System.out.println("方式5 错误:" + e);
    }
}
```



# 方式 6：ResourceLoader 接口

```java
@Autowired
ResourceLoader resourceLoader;

private void way6ByStream() {
    try {
        InputStreamReader inputStreamReader = new InputStreamReader(resourceLoader.getResource("test.txt").getInputStream());
        BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
        System.out.println("方式6 文件内容:" + bufferedReader.readLine());
    } catch (Exception e) {
        System.out.println("方式6 错误:" + e);
    }
}
```



# 各种方式总结

1. 这几种方式在工具中都是可以正常读取到的。
2. 当打成war运行时，这几种方式都是可以正常运行的，war会被自动解压成目录然后才运行。
3. 当打成jar时，必须通过流的方式才能正常读取或拷贝文件。方式1和方式2都不能读取到文件，因为文件都在jar包中，没有一个实际的路径，所以会提示文件不存在。

