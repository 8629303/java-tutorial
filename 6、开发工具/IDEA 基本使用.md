> 来源：IDEA的基本使用：让你的IDEA有飞一般的感觉：https://glory.blog.csdn.net/article/details/77868300

# 1、设置 Maven

1. 在 File =》settings =》 搜索 maven
2. Mavan home directory =》设置maven安装包的bin文件夹所在的位置
3. User settings file=》设置setting文件所在的位置
4. Local repository=》设置本地仓库的



# 2、设置代码行宽度

1. 在File =》settings =》Editor =》Code Style
2. 有人会问，如果输入的代码超出宽度界线时，如何让IDE自动将代码换行？有两种方式！
3. 第一种，在上述的“Right margin (columns)”的下方，有“Wrap when typing reaches right margin”选项，选中它，是什么效果呢？
4. 随着输入的字符的增加，当代码宽度到达界线时，IDEA会自动将代码换行。
5. 第一种方式是在输入代码时触发，还有第二种方式，在File =》settings =》Code Style =》Java中，选中“Wrapping and Braces”选项卡，
6. 在“Keep when reformatting”中有一个“Ensure rigth margin is not exceeded”，选中它，是什么效果呢？
7. 从配置项的字面意思很容易理解，在格式化Java代码时，确保代码没有超过宽度界线。
8. 即输入的代码超出界线后



# 3、提示不区分大小写

1. 首先打开File =》setting
2. 然后，输入：sensitive
3. 将右侧的 case sensitive completion 修改为 NONE



# 4、强制更新 Maven Dependencies

1. Intellj 自动载入Mave依赖的功能很好用，但有时候会碰到问题，导致pom文件修改却没有触发自动重新载入的动作，此时需要手动强制更新依赖。
2. 如下：
3. 手动删除Project Settings里面的Libraries内容；
4. 在Maven Project的试图里clean一下，删除之前编译过的文件；
5. 项目右键 =》Maven =》Reimport
6. Ok， 此时发现依赖已经建立！ 



# 5、IDEA 的环境配置默认保存位置

1. IDEA 的环境配置默认保存位置:C:\Users\xxxxxxxxx\.IntelliJIdea14 , xxxxxx代表用户目录
2. 可以对该目录进行备份,一但环境出问题恢复此配置即可.
3. 可以在%IDEA_HOME%/bin/idea.properties中修改该配置路径.



# 6、隐藏不想看到的文件或者文件夹

1. Intellij IDEA 隐藏不想看到的文件或者文件夹（类似eclipse的filter功能）
2. 打开intellij =》File =》Settings =》搜索 File Types =》点击 Ignored Files and Folders
3. 点击Editor =》点击File Types =》点击 Ignored Files and Folders
4. 添加你想隐藏的文件或文件夹
5. 再返回看我创建的项目目录，隐藏的那些文件或文件夹都不见了。



# 7、修改为Eclipse快捷键

File =》Settings =》Keymap =》 Keymaps 改为 Eclipse



# 8、修改默认设置（Settings）

默认设置 等同于 新项目设置修改（即全局设置，无需每次启动打开新项目都设置）

1. 2020 版本：File =》Other Settings =》default setting
2. 2022 版本：全局设置 File =》Other Settings =》Setting for New Projects



# 9、修改智能提示快捷键

1. File =》Settings =》Keymap =》Main menu =》Code =》Completion  =》Basic =》修改为 Ctrl+Alt+Enter
2. 保存时把冲突的Remove掉。
3. File =》Settings =》Keymap =》Editor Actions =》Complete Current Statement =》修改为 Ctrl+;



# 10、查找快捷键冲突问题处理

File =》Settings =》Keymap =》Main menu =》Edit =》Find =》修改Find...和Replace...分别改为Ctrl+F 和Ctrl+R



# 11、显示行号设置

1. File =》Settings =》Editor =》General =》Appearance =》Show Line Numbers 选中
2. 在代码行号区域区域点击右键 =》选择Show Line Numbers



# 12、代码智能提示，忽略大小写

File =》Settings =》Editor =》Code Completion =》把Case sensitive completion设置为None就可以了



# 13、用*标识编辑过的文件 

1. Editor =》General =》Editor Tabs =》Appearance =》Mark modifyied (*)
2. 在IDEA中，你需要做以上设置, 这样被修改的文件会以*号标识出来，你可以及时保存相关的文件。





# 14、关闭自动代码提示

Preferences =》IDE Settings =》Editor =》Code Completion =》Autopopup documentation in (ms)



# 15、改变编辑文本字体大小

File =》settings =》EDITOR COLORS & FONTS =》FONT =》SIZE



# 16、编码字符集设置

1. FILE =》SETTINGS =》FILE ENCODINGS =》IDE ENCODING
2. FILE =》SETTINGS =》FILE ENCODINGS =》Project Encoding
3. FILE =》SETTINGS =》FILE ENCODINGS =》Default encoding for properties files
4. FILE =》SETTINGS =》FILE ENCODINGS =》Transparent native-to-ascii conversion



# 17、Live Template 自定义模板（类&方法注释）

> IDEA自定义JavaDOC注释模板：https://blog.csdn.net/m0_58680865/article/details/128377619

1、选择Live Template：File =》Settings =》Editor =》Live Templates =》Java

2、创建自定义 Template Group：点击右边的 + 号，选择 Template Group，输入名称，比如：Test

3、创建自定义Live Template：选中刚才新建的Test，再次点 + 号，选择Live Template，起名比如cc（类注释），mc（方法注释）

4、设置模板内容：点击Edit variables按钮，选择IDEA提供的表达式，最后Apply

- 类注释：方式-1。变量属性可以点击 Edit variables 编辑

  ```java
  /**
   * @Title:
   * @ClassName: $PackageName$.$ClassName$.java
   * @Description:
   *
   * @Copyright 2016-2018 公司名称 - Powered By 研发中心
   * @author: 作者名
   * @date:  $DATE$ $TIME$
   * @version V1.0
   */
  ```

  其中变量属性为（Edit variables 中设置的值）：

  | Name        | Expression         |
  | ----------- | ------------------ |
  | PackageName | currentPackage()   |
  | ClassName   | className()        |
  | DATE        | date("yyyy-MM-dd") |
  | TIME        | time()             |

- 类注释：方式-2。通过IDEA文件头方式，每次创建文件自动生成

  1. 第一步：File=》Settings=》Editor=》File and Code Templates
  2. 第二步：Includes=》File Header（然后添加如下内容）

  ```java
  /**
   * @Title:
   * @BelongProjecet ${PROJECT_NAME} 
   * @BelongPackage ${PACKAGE_NAME}
   * @Description: 
   *
   * @Copyright 2019 公司名 - Powered By 研发一部
   * @Author: 作者名
   * @Date:  ${DATE} ${TIME}
   * @Version V1.0
   */
  ```

- 方法注释（Live Templates 中）：

  ```java
  /**
   * @Title:  
   * @MethodName: $methodName$ 
  $param$
   * @Return $returns$
   * @Exception $exception$
   * @Description:
   *
   * @author: 作者名
   * @date:  $date$ $time$
   */
  ```

  其中变量属性为（Edit variables 中设置的值）：

  | Name       | Expression                            |
  | ---------- | ------------------------------------- |
  | methodName | methodName()                          |
  | param      | methodName() 或者 使用上面 param 脚本 |
  | returns    | methodReturnType()                    |
  | exception  | expressionType(Expression)            |
  | date       | date("yyyy-MM-dd")                    |
  | time       | time()                                |

  这里本人配置失效了。没找出原因。建议注释还是使用`File and Code Templates`来配置。



5、模板的使用

1. 类文件头部：cc + tab
2. 方法名头部：mc + tab



6、JavaDoc 需要熟知的注释标签：

1. @see 引用类/方法。
2. @author: 作者。
3. @date：日期。
4. @version: 版本号。
5. @throws：异常信息。
6. @param：参数
7. @return： 方法返回值。
8. @since: 开源项目常用此标签用于创建日期 。
9. {@value}: 会使用该值，常用于常量。
10. {@link} 引用类/方法。
11. {@linkplain} 与@link功能一致。



7、IDEA 需要熟知的标签（File and Code Templates 中使用的）

1. ${USER}：当前用户。
2. ${DATE}：当前日期。
3. ${TIME}：当前时间。
4. ${YEAR}：当前年。
5. ${MONTH}：当前月。
6. ${DAY}：当前日。
7. ${HOURS}：当前小时。
8. ${MINUTE}：当前分钟。
9. ${PACKAGE_NAME}：包名。

# 18、配置 Tomcat 参数

Run =》Edit Configurations =》添加Tomcat Server =》VM options

```bash
vm options: -Xms256m -Xmx512m -XX:PermSize=128m -XX:MaxPermSize=256m
```



# 19、调整IDEA启动时的内存配置参数

```bash
%IDEA_HOME%/bin/idea.exe.vmoptions
```

如下配置：第一个是IDEA最小启动内存，我IDEA 2022版本的, 默认128m（可以改成1024m）；第二个是idea最大启动内存，默认1024m（可以改成2048m）；第三个是IDEA代码缓存大小，默认是512m（可以改成1024m），各位可以根据自己电脑配置自行分配。

```
-Xms128m
-Xmx1024m
-XX:ReservedCodeCacheSize=512m
```



# 20、导入Eclipse Web项目Tomcat找不到

导入Eclipse Web项目发布到Tomcat如果找不到，可以在环境配置的Facets增加web支持,在Artifacts中增加项目部署模块名





# 21、打开新文件CPU很高，去除校验即可

每次打开一个新jsp或java文件时,cpu都占用很高,去掉检验即可:

- File =》Settings =》Editor =》Inspections



# 22、开启类修改后自动编译

1. File=》setting=》Buil,Execution,Deployment=》compiler=》Make project automatically
2. 编译错误问题解决：Error:java: Compilation failed: internal java compiler error
3. set 中 Java complier 设置的问题 ，项目中有人用jdk1.6 有人用jdk1.7 版本不一样 会一起这个错误



# 23、提示实现Serializable接口

1. 使用 Eclipse 或 MyEclipse 的同学可能知道，如果 implements Serializable 接口时，会提示你生成 serialVersionUID。
2. 但 Intellij IDEA 默认没启用这个功能。
3. Preferences=》IEditor=》nspections=》Serialization issues=》Serializable class without ’serialVersionUID’，
4. 选中以上后，在你的class中：光标定位在类名前，按 Alt+Enter 就会提示自动创建 serialVersionUID了



# 24、开启演出模式

我们可以使用【Presentation Mode】，将IDEA弄到最大，可以让你只关注一个类里面的代码，进行毫无干扰的coding。

可以使用Alt+V快捷键，谈出View视图，选择Appearance ，然后选择Enter Presentation Mode。

这个模式的好处就是，可以让你更加专注，因为你只能看到特定某个类的代码。可能读者会问，进入这个模式后，我想看其他类的代码怎么办？这个时候，就要考验你快捷键的熟练程度了。你可以使用CTRL+E弹出最近使用的文件。又或者使用CTRL+N和CTRL+SHIFT+N定位文件。

如何退出这个模式呢？很简单，使用ALT+V弹出View视图，选择Appearance ，然后选择Exit Presentation Mode 即可。

但是我强烈建议你不要这么做，因为你是可以在Enter Presentation Mode模式下在IDEA里面做任何事情的。当然前提是，你对IDEA足够熟练。



# 25、Inject Language

如果你使用IDEA在编写JSON字符串的时候，然后要一个一个\去转义双引号的话，就实在太不应该了，又烦又容易出错。在IDEA可以使用Inject Language帮我们自动转义双引号。

- 先将焦点定位到双引号里面，使用Alt + Enter快捷键弹出 inject Language 视图，并选中：**Inject language or reference**。
- 选择后，切记要直接按下Enter回车键，才能弹出 Inject language 列表。在列表中选择 JSON（JSON files）组件。
- 选择完后。鼠标焦点自动会定位在双引号里面，这个时候你**再次使用 Alt + Enter 就可以看到**。
- 选中 Edit JSON Fragment 并回车，就可以看到编辑JSON文件的视图了。然后我们可以输入想要输入的JSON字符串。
- 可以看到IDEA确实帮我们自动转义双引号了。如果要退出编辑JSON信息的视图，只需要使用ctrl+F4快捷键即可。
- Inject Language可以支持的语言和操作多到你难以想象，读者可以自行研究。



# 26、强大的 symbol

如果模糊记得某个方法名字个别字母，想在IDEA里找到，可以直接使用 Ctrl + Shift + Alt + N，使用symbol来查找即可。 



# 27、IDEA 和 Windows 默认快捷键冲突解决

1. 如：Ctrl + Alt + ↑或 Ctrl + Alt + F12
2. **解决方式：在桌面右键 - 图形选项 - 快捷键 - 禁止 就可以了**



# 28、格式化代码时，注释被格式化问题

解决方案：File=》Settings=》Editor=》Code Style=》Java =》将 Enable Javadoc Formating 取消掉



# 29、IDEA 2020版显示 Run Dashboard

1. 启动按钮，点击“Edit Configrations”
2. 左侧，选择Templates
3. 右侧，选择Configurations available in Run Dashboard
4. 点击“+”，选择Spring Boot



# 30、切换大小写

- IDEA 的切换大小写的**默认快捷键**是：Ctrl + Shift + U
- 如果默认快捷键冲突，可以双击Shift，输入：Toggle Case



# 31、IDEA中批量删除代码的注释

先通过IDEA使用：Ctrl + R（或者 Ctrl + Shift + R）

1. 选择正则表达式替换【可选】，选择Regex
2. 可以选择指定文件类型【可选】：File mask：*.java
3. 替换内容：为空就是删除。

```shell
(/\*([^*]|[\r\n]|(\*+([^*/]|[\r\n])))*\*+/|[ \t]*//.*)
```

或者

```shell
(/\*([^*]|[\r\n]|(\*+([^*/]|[\r\n])))*\*+/)
```





# 32、忽略CSS、JS文件报错

- 选择需要处理的文件，按下快捷键：Ctrl + Alt + Shift + H
- 将高亮级别调到**None**即可
- 不消失的话关闭重新打开文件，或重启IDEA



# 33、终端 Terminal 改为 GitBash 或 Cmder

1、IDEA Terminal 替换成 Git Bash

- 在IDEA中，打开settings，设置相应的Bash路径：

- Files=》Settings=》Tools=》Terminal=》Shell path

  ```bash
  C:\Program Files (x86)\Git\bin\bash.exe
  # 或者, 如下方式 请检查路径中，是否正确加了英文双引号，路径是否正确。
  "C:\Program Files (x86)\Git\bin\sh.exe" -login -i
  ```

2、IDEA Terminal 替换成 Cmder

- 增加系统变量：变量名为：**CMDER_ROOT**、变量值为：Cmder 安装主目录

- 打开设置（Ctrl + Alt + S），进入 Plugins，搜索栏搜索 Terminal，查看 Terminal 插件是否打勾选中，如果没有，请打勾

- 进入设置（Ctrl + Alt + S），进入 Tools 字段，再进入 Terminal 字段，在 Shell path 那一栏中，输入你主机 Cmder 的安装位置

  ```bash
  "cmd.exe" /k ""%CMDER_ROOT%\vendor\init.bat""
  ```

- 重新启动 Idea IDE，然后打开 Terminal 查看是否配置正确。

- 如果没有，请检查路径中，是否正确加了英文双引号，路径是否正确。有的老版本，cmd.exe 两边是不需要加双引号的，但是新版本的策略中，都要求加双引号了。

2、解决git commit注释乱码的问题

- 在C:\Program Files\Git\etc\bash.bashrc末尾行追加如下内容：

  ```bash
  export LANG="zh_CN.UTF-8"
  export LC_ALL="zh_CN.UTF-8"
  ```

3、重启：重启IDEA或者关闭当前Terminal的Session连接，然后New Session连接。



# 34、设置 Terminal 背景色与字体颜色

1、设置Terminal背景色

- Terminal背景色默认是白色，在cmder和gitbash中习惯使用了黑色，我这里就修改为黑色背景
- File=》Settings=》Editor=》Console Color=》在右边选择Console=》Background，设置Background

2、设置Terminal字体颜色

- Terminal背景色默认是白色，在cmder和gitbash中习惯使用了黑色，我这里就修改为黑色背景
- File=》Settings=》Editor=》Console Color=》在右边选择Console=》Background，设置Background





# 35、导入的项目转成Maven项目

1. 工程文件下如果没有pom.xml，新建文件pom.xml，并填写好内容。
2. 在 pom.xml 文件上右键：Add as Maven Project。



# 36、一次启动多个SpringBoot应用

1. 选择任意一个SpringBoot的启动类或者点击Run标签，然后选择 Edit Configurations...
2. 左上角选择 +，添加选择 Compound
3. 修改Name，把对应的启动类添加进来即可
4. 在Run Dashboard中显示多个启动类
   - idea2020以下版本用RunDashboard
     - 启动按钮，点击“Edit Configrations”
     - 左侧，选择Templates
     - 右侧，选择Configurations available in Run Dashboard
     - 点击“+”，选择Spring Boot
   - idea2020以后版本改功能被services替代
     - 在Services窗口（ALT + 8）中，选择Run Configuration Type
     - 然后选择Compound或者SpringBoot即可



# 37、IDEA 查看类结构

可以查看类结构，包括类中的属性、方法、内部类等

1. 方法一：选中类名或在当前类任何位置，使用快捷键 Ctrl + F12，但是只能用一次，每次用都要按 Ctrl + F12
2. 方式二：选中类名，在编辑器的左上角点击 Structure（ALT + 7）
3. 方式二：选中类名，在 Project（ALT + 1）的右上角点击设置，然后点击 Show Members，会在类文件下显示树结构方法名





# 38、IDEA删除多余空行

先通过IDEA使用：Ctrl + R（或者 Ctrl + Shift + R）

1. 选择正则表达式替换【可选】，选择Regex

2. 可以选择指定文件类型【可选】：File mask：*.java

3. 输入如下正则表达式：

   ```
   ^\s*\n
   ```

4. 替换全部 Replace All



# 39、代码部分格式化

有时候我们写一些代码的时候，格式化后，会使代码格式变得不美观、不易读，这里在看一些源码的时候发现有这么一种操作，随手记下

在IDEA中将Formatter markers打开：

- File=》Settings=》Code Style=》Formatter Control 中开启 Enable formatter markers in comments

像以下这样，`@formatter:off`开启关闭格式化，`@formatter:on`重新开启格式化：

```java
// @formatter:off
http
    .authorizeRequests().expressionHandler(expressionHandler)
    .and()
    .addFilterBefore(resourcesServerFilter, AbstractPreAuthenticatedProcessingFilter.class)
    .exceptionHandling()
    .accessDeniedHandler(accessDeniedHandler)
    .authenticationEntryPoint(authenticationEntryPoint);
// @formatter:on
```



# 40、下载JAR关联的源码

步骤：

1. File=》Settings=》Maven=》Importing（ 勾选上 Sources 和 Documentation）
2. 右键项目的pom.xml=》Maven=》Download Xxx
   - Download Sources
   - Download Documentation
   - Download Sources And Documentation



# 41、Version Controller 不显示

> 这种问题只会发生在IDEA2020版本之前，IDEA2020版本之后无需设置。

方式一：VCS=》Enable Version Control Integration 即可以显示Version Controller窗口

方式二：使用 VCS 操作弹出窗口（Alt + ` 或 VCS | VCS 操作弹出窗口），选择 Enable Version Control Integration



# 42、将WEB-INF\lib下JAR包添加到项目中

1. 打开Project Structure【可以使用快捷键：Ctrl + Alt + Shift + S】
2. 左侧选中Modules，在Dependecies中，点击右侧“+”号，选择：JARS or directories...
3. 选择WEB-INF\lib下的Jar包添加
4. 左侧选中Artifacts，然后点击-项目名:war exploded ，在Avaliable Elements中将刚刚添加的JAR包put into /WEB-LIF/lib



# 43、设置JAVA中的常量类等号对齐

1. Setting=》code style=》Java
2. Wrapping and baces=》Group declarations=》Align fields in columns 和 Align variables in columns（勾上即可）

```java
private int    f1     = 1;
private String field2 = "";
```

可以查看Java代码中变量与只都已经对齐了。



44、插入代码模板（Ctrl + J）

在IDEA中，通过代码模板，可以快速创建main函数，for循环等

- main 函数：psvm 也就是public static void main的首字母。

- for 循环：

  ```java
  itar 生成array for代码块
  itco 生成Collection迭代
  iten 生成enumeration遍历
  iter 生成增强for循环
  itit 生成iterator 迭代
  itli 生成List的遍历
  ittok 生成String token遍历
  ```

  示例：生成10次循环

  ```java
  // 正序：10.fori
  for (int i = 0; i < 10; i++) {
              
  }
  ```

  ```java
  // 反序：10.forr
  for (int l = 10; l > 0; l--) {
              
  }
  ```

- Ctrl + J：插入现有的代码模板

- Ctrl + Alt + J：用动态模板环绕



# 44、代码块折叠

IDEA 为我们，提供了如下方式

鼠标左键选中需要折叠的代码块儿=》右键=》Folding=》Fold Selecton/Remove region（或者Fold Code Block）

Folding 中可以看到有6个折叠选项，它的快捷键都不太好用。可以右击 “Add Keyboard Shortcut” 自定义你的快捷键。下面就分别说一下这6个选项所代表的意义。

1. Expand：展开子层

   Expand：展开折叠的代码，如果里面有多层且之前没有展开过，需要一层一层的展开。如果之前展开过且是它相当于折叠的回退键。也就是说你这个代码折叠之前是怎么样，就展开是什么样。(对Java,xsd都有效)

2. Collapse：递归折叠

   Collapse：折叠代码，以选中的代码开始，一层一层的往父层折叠直到全部折叠起来。并不会影响父层的平层，也就是只是影响它的父、祖父及以上的级别，不会影响叔级别。(java,xsd都有效)

3. Expand All：展开所有折叠

   Expand All：展开所有的折叠代码。也就是完全展开，不会出现折叠代码。(java,xsd都有效)

4. Collapse All：折叠所有代码

   Collapse All：折叠所有代码块。会把所有展开代码全部折叠起来。只要有下层代码块都会折叠起来。(java,xsd都有效)

5. Fold Selection/Remove region：折叠/展开交替

   折叠/展开选中当前代码块。先折叠代码块到父级，然后再展开代码块，然后再折叠代码块到父级….(java,xsd都有效)

6. Fold Code Block：递归折叠

   Collapse：折叠代码，以选中的代码开始，一层一层的往父层折叠直到全部折叠起来。并不会影响父层的平层，也就是只是影响它的父、祖父及以上的级别，不会影响叔级别。(对Java代码,对xsd无效)。





# 45、控制台中文乱码解决

第一步：修改IDEA中的vmoptions配置文件：进入Intellij idea安装目录，找到bin文件夹下面：idea64.exe.vmoptions和idea.exe.vmoptions这两个文件。分别在这两个文件中添加：**-Dfile.encoding=UTF-8**

第二步：修改 idea中的FileEncodings：File=》Settings=》Editor=》FileEncodings 中将 GlobalEncoding、ProjectEncoding 和 Default encoding for properties都配置成UTF-8

第三步：修改Tomcat中的VM options：

1. 在Tomcat图标下拉框，选择Edit Configurations
2. 在VM options项中添加：-Dfile.encoding=UTF-8

第四步：重启 IDEA





# 46、多线程调试，进入线程run或call方法

IDEA 多线程调试设置：

1. 在断点上右键
2. 选择Thread,然后点Done(建议选择Thread后点击make default把此模式设置为默认模式
3. 开启Debug模式，就可以进入多线程了
4. 当进入run或者call方法时，idea会有个提示框（idea2018.1、idea2020.1），点击Switch thread
5. 这时候就进入具体的线程了



# 47、打包可执行JAR包，并指定JDK编译版本

Idea：Java文件打包可执行的JAR文件，并指定JDK编译版本：https://glory.blog.csdn.net/article/details/89277741



# 48、双斜杠注释改成紧跟代码头

1. File =》settings =》Code Style =》Java中，选中“Code Generation”选项卡
2. 勾选 Comment Code 下面的 “Line comment at first column” 与 “Add a space at comment start”



# 49、设置 Ctrl + 鼠标左键双击创建类 的快捷键

File =》settings =》Keymap =》Main Menu =》File =》File Open Actions =》Open Project Actions =》new =》new Group (1) =》Java Class，右键选中 ”add Mouse Shortcut“ 选项卡，然后设置 Ctrl + 鼠标双击 即可。 