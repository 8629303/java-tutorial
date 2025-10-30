# MacOS 环境变量配置

> - Mac环境变量配置：https://blog.csdn.net/weixin_45364220/article/details/125517554
> - Mac 环境变量配置：https://blog.csdn.net/Blue_Pepsi_Cola/article/details/134421132



## 1、快速充电 Bash & Zsh

在 MacOS 上，bash 和 zsh 是两种常用的 shell，各自有一套不同的配置文件，用来控制环境变量、别名、路径等自定义配置。下面将详细说明 bash 和 zsh 的配置文件、它们的加载顺序、以及二者的关联与区别。

- bash：macOS Mojave 及更低的版本会使用 bash shell

- zsh：macOS Catalina 及更高的版本会使用 zsh shell

  > zsh 高度兼容 bash，但也有一定的差别「使用 `man zsh`即可查看」

- 查看电脑当前 shell

  ```bash
  echo $SHELL
  ```

- 查看电脑已经安装的 shell

  ```bash
  cat /etc/shells
  ```

- 切换 shell：如果您想更改默认的shell，可以使用`chsh`命令。例如，要将默认 shell 更改为 zsh，可以输入：

  ```bash
  chsh -s /bin/bash
  ```

  然后系统会要求您输入密码以确认更改。请注意，更改默认 shell 可能需要注销并重新登录才能使更改生效。

- Mac 系统下的环境变量—bash

  1. `/etc/profile`：全剧配置，所有用户登录时就会执行一次
  2. `/etc/paths`：类似于`/etc/profile`
  3. `~/.bash_profile`：单用户个人配置，用户登录时执行一次，用户设置的环境变量执行 `.bashrc`。和 `/etc/profile`一样也需要重启后生效。不同的是 `/etc/profile`针对所有用户；`~/.bash_profile`针对当前单个用户
  4. `~/.bash_login`：当每次登陆系统(退出`bash shell`)时，执行该文件
  5. `~/.profile`：针对个人，读取的是个人的配置文件一系列操作「如果用户登录状态，读 `.bash_profile`文件、若`.bash_profile`不存在，则读`.bash_login`，如果前两者都不存在，最后才读`~/.profile`」
  6. `~/.bashrc`：属于当前用户专有的 `shell`文件，当登陆或者打开一个 `shell`窗口时就会被读取，因此无需重启 `shell`窗口

- Mac 系统下的环境变量—zsh

  1. `/etc/zshenv`：为系统上所有用户设置环境变量，在登录shell时首先加载。

  2. `~/.zshenv`：用户特定的环境变量，在登录shell时加载。

  3. `/etc/zprofile`：为系统上所有用户设置环境，登录shell和非登录shell都会加载。

  4. `~/.zprofile`：用户特定的环境，登录shell和非登录shell都会加载。

  5. `/etc/zshrc`：为系统上所有用户设置shell选项，交互式shell和登录shell会加载。

  6. `~/.zshrc`：用户特定的shell选项，交互式shell和登录shell会加载。

  7. `/etc/zlogin`：为系统上所有用户设置登录时的shell函数，登录shell会加载。

  8. `~/.zlogin`：用户特定的登录时的shell函数，登录shell会加载。

  9. `/etc/zlogout`：此文件在登录 shell 登出时加载。应用于显示消息和删除文件。

  10. `~/.zlogout`：用户的退出 shell 配置文件，登录 shell 退出时加载，用于清理工作。

      如下为重点需要配置的两个配置文件

  11. `~/.zprofile`：类似于`~/.bash_profile`，登录时运行，并且允许 `SSH`

  12. `~/.zshrc`：类似于 `~/.bashrc`，针对的是每一个 “终端” `shell 窗口`

看到这儿还不知道这些文件在哪儿？快捷键：**`command + shift + .`**，也可以使用命令行查看：**`ls -la`**。



## 2、Bash 配置文件

### 1、系统级配置文件（适用于所有用户）
- **`/etc/profile`**：系统范围的 Bash 配置文件，适用于所有用户的登录 shell。当你以登录 shell 的方式启动 Bash 时，它会首先加载 `/etc/profile`。
- **`/etc/paths`**：这是 `/etc/profile` 会加载的一个文件，它列出系统的默认 `PATH` 设置。



### 2、用户级配置文件（仅适用于当前用户）

- **`~/.bash_profile`**：用户的登录 shell 配置文件。只在启动登录 shell 时加载。通常用于设置环境变量、路径等。优先级高于 `~/.bash_login` 和 `~/.profile`。
- **`~/.bash_login`**：如果 `~/.bash_profile` 不存在，Bash 会尝试加载这个文件。它通常作为备用的配置文件。
- **`~/.profile`**：如果 `~/.bash_profile` 和 `~/.bash_login` 都不存在，则加载 `~/.profile`。这个文件是一个通用的 shell 登录文件，适用于多种 shell。
- **`~/.bashrc`**：用户的非登录（交互式） shell 配置文件。每次启动非登录 shell 时都会加载。一般用于别名、提示符、命令历史等交互式配置。



### 3、加载顺序（Bash）

1. `/etc/profile`（系统范围）
2. `~/.bash_profile` 或 `~/.bash_login` 或 `~/.profile`（用户范围，按优先级加载）
3. 非登录 shell 时加载 `~/.bashrc`
4. 总结：`/etc/profile ==> /etc/paths ==> ~/.bash_profile ==> ~/.bash_login ==> ~/.profile ~/.bashrc`

通常情况下，登录 shell 会加载 `.bash_profile`，而非登录 shell（如打开一个新终端）会加载 `.bashrc`。



### 4、加载顺序主要要点

MacOS 系统的 Bash Shell  配置文件加载顺序注意点：

*   `/etc/profile`和`/etc/paths`是系统级别的，系统启动就会加载，后面几个是当前用户级的环境变量。后面3个按照从前往后的顺序读取，如果`/.bash_profile`文件存在，则后面的几个文件就会被忽略不读了，如果`/.bash\_profile`文件不存在，才会以此类推读取后面的文件。`~/.bashrc`没有上述规则，它是 Bash Shell 打开的时候载入的。
*   全局设置，一般不建议修改`/etc/profile`和`/etc/bashrc`文件，而去修改`/etc/paths`文件。如果想要对所有用户都生效，则可在`/etc/paths`文件的最后一行加上自己的 App 路径。优先级也是最高的。
*   单个用户生效的配置，一般都是修改`～/.bash_profile`文件，若 Bash Shell 是以 login 方式执行时，才会读取此文件。该文件仅仅执行一次。



### 5、Bash 配置文件检查脚本

这是一个简单的 Shell 脚本，用于检查 Bash 配置文件是否存在，并按顺序打印出来：

```shell
#!/bin/bash

echo "系统级配置文件加载顺序:"
for file in /etc/profile /etc/paths; do
    if [[ -f $file ]]; then
        echo $file
    fi
done
 
echo "用户级配置文件加载顺序:"
for file in ~/.bash_profile ~/.bash_login ~/.profile ~/.bashrc; do
    if [[ -f $file ]]; then
        echo $file
    fi
done
```

运行这个脚本将列出你的系统上 bash 配置文件的加载顺序。记得给这个脚本执行权限，使用`chmod +x script_name`，然后通过`./script_name`运行。



## 3、Zsh 配置文件

### 1、系统级配置文件（适用于所有用户）
- **`/etc/zshenv`**：系统范围的配置文件，适用于所有用户。每次启动 `zsh` 时（无论是否是登录 shell 或交互式 shell）都会加载。通常用于设置系统范围的环境变量。
- **`/etc/zprofile`**：只在登录 shell 时加载。
- **`/etc/zshrc`**：只在交互式 shell 时加载，无论是否是登录 shell。
- **`/etc/zlogin`**：只在登录 shell 启动后加载。
- **`/etc/zlogout`**：只在退出 shell 配置后加载。



### 2、用户级配置文件（仅适用于当前用户）

- **`~/.zshenv`**：用户的环境变量配置文件，每次启动 `zsh` 时都会加载。这个文件通常只用于设置环境变量。
- **`~/.zprofile`**：用户的登录 shell 配置文件，只在登录 shell 时加载。与 Bash 的 `.bash_profile` 类似。
- **`~/.zshrc`**：用户的交互式 shell 配置文件，只在交互式 shell 启动时加载。常用于设置别名、命令提示符等交互性配置。
- **`~/.zlogin`**：用户的登录 shell 启动后加载，类似于 `.zprofile`，但在其后执行。
- **`~/.zlogout`**：用户的退出 shell 配置文件，登录 shell 退出时加载，用于清理工作。



### 3、加载顺序（Zsh）

1. `/etc/zshenv`（系统范围）
2. `~/.zshenv`（用户范围）
3. **登录 shell**：加载 `/etc/zprofile` → `~/.zprofile` → `/etc/zlogin` → `~/.zlogin`
4. **交互式 shell**：加载 `/etc/zshrc` → `~/.zshrc`
5. **退出 shell**：加载 `~/.zlogout` 和 `/etc/zlogout`
6. 总结：`/etc/zshenv → ~/.zshenv → /etc/zprofile → ~/.zprofile → /etc/zlogin → ~/.zlogin → /etc/zshrc → ~/.zshrc → ~/.zlogout~/.zlogout → /etc/zlogout`

与 `bash` 不同，`zsh` 的 `.zshrc` 文件在登录和非登录 shell 启动时，都会在交互模式下加载。



### 4、加载顺序注意点

MacOS 系统的 Zsh Shell  配置文件加载顺序注意点：

- 非登录shell（比如新的终端窗口或者脚本执行）通常只加载`/etc/zshrc`和`~/.zshrc`。
- `.zshrc` 文件在登录和非登录 shell 启动时，都会在交互模式下加载。这点与 Bash 是不同的。



### 5、Zsh 配置文件检查脚本

这是一个简单的 Shell 脚本，用于检查 Zsh 配置文件是否存在，并按顺序打印出来：

```shell
#!/bin/zsh
 
echo "系统级配置文件加载顺序:"
for file in /etc/zshenv /etc/zprofile /etc/zshrc /etc/zlogin /etc/zlogout; do
    if [[ -f $file ]]; then
        echo $file
    fi
done
 
echo "用户级配置文件加载顺序:"
for file in ~/.zshenv ~/.zprofile ~/.zshrc ~/.zlogin ~/.zlogout; do
    if [[ -f $file ]]; then
        echo $file
    fi
done
```

运行这个脚本将列出你的系统上 zsh 配置文件的加载顺序。记得给这个脚本执行权限，使用`chmod +x script_name`，然后通过`./script_name`运行。



## 4、Bash & Zsh 关联与区别

### 1、加载顺序与用途不同
- **登录与非登录 shell**：
  - **Bash**：分为登录 shell 和非登录 shell，登录 shell 会加载 `.bash_profile`，而非登录 shell 加载 `.bashrc`。这意味着在登录后打开的终端和直接打开终端加载的配置文件不同。
  - **Zsh**：同样分为登录和非登录 shell，但 `.zshrc` 通常用于交互式 shell 设置，而 `.zprofile` 和 `.zlogin` 则专门用于登录 shell。



### 2、环境变量与交互配置的区分

- **Bash**：常常将环境变量放在 `.bash_profile` 中，而将交互式设置（如别名和提示符）放在 `.bashrc` 中。
- **Zsh**：`zsh` 中，环境变量通常放在 `.zshenv` 中，而交互式配置放在 `.zshrc` 中，登录 shell 的设置可以放在 `.zprofile` 或 `.zlogin` 中。



### 3、配置文件的灵活性

- **Bash**：不同的 shell 类型加载不同的配置文件，可以通过将 `.bash_profile` 调用 `.bashrc` 来确保配置一致性。
- **Zsh**：`zsh` 的 `.zshrc` 文件在登录和非登录交互式 shell 中都会加载，减少了在两个 shell 配置文件之间复制设置的需求。



### 4、兼容性

- **Bash 与 Zsh 兼容**：如果你从 `bash` 切换到 `zsh`，你可以简单地将 `.bash_profile` 或 `.bashrc` 中的内容迁移到 `zsh` 的 `.zshrc` 或 `.zprofile` 中，但注意 `zsh` 支持更多的高级功能，可以优化配置文件以充分利用这些功能。



### 5、总结

- **Bash** 主要通过 `.bash_profile` 和 `.bashrc` 来分别处理登录和非登录 shell。
- **Zsh** 则通过 `.zshrc` 处理大多数交互式配置，且在登录 shell 中可以使用 `.zprofile` 或 `.zlogin`。
- 两者的文件组织和用途虽类似，但 `zsh` 的配置文件更灵活，特别是在处理交互式和非交互式 shell 时。



## 5、快速充电环境变量

- 环境变量（Environment Variables）一般是指在操作系统中用来指定操作系统运行环境的一些参数，即指明操作系统的重要目录在哪里。如：临时文件夹位置和系统文件夹位置等。
- 对于现在的 Mac 来说，因为默认使用 zsh 作为 Shell，现在 Mac 系统只会显示默认的 zsh 加载的文件。

* 如果没特殊说明，设置 PATH 的语法都为：

  ```bash
  # 中间用冒号隔开
  export PATH=<PATH_1>:<PATH_2>:<PATH_3>:...:<PATH_N>:$PATH
  ```

* 推荐设置用户个人自己的环境变量，尽量不破坏 Mac 系统全局变量（即在`.bash_profile`文件内修改，不要在`/etc/profile 和 /etc/bashrc`文件修改）

* 最后在修改完环境变量的配置文件后需要使其生效，您可以使用以下命令使更改立即生效（无需重启终端）

  ```bash
  source ~/.zshrc
  ```

  或者，如果您修改了`~/.zprofile`或`~/.zlogin`，并希望它们立即生效，您可能需要注销并重新登录，或者重新启动计算机。





## 6、增加环境变量配置

### 1、Zsh Shell 环境变量配置

这里我们使用 zsh 来演示，如果想使用 bash 的话，可以单独修改`.bash_profile`或`.bashrc` ，或者单独修改`.bashrc`也行。

如果发现没有`.zprofile`与`.zshrc`两个文件，可以直接在 `~`目录下创建。由于我们是`.zshrc`所以我们就直接上`.zshrc`的设置。

1、先编辑`~/.zshrc`文件

```bash
vim ~/.zshrc
```

2、添加环境变量【JDK 和 MAVEN】

```bash
# JDK
export JAVA_HOME=/Users/lsx/Java/JDK/jdk-11.jdk/Contents/Home
# MAVEN
export MAVEN_HOME=/Users/lsx/Java/Maven/apache-maven-3.8.8

export PATH=$JAVA_HOME/bin:$PATH
export PATH=$MAVEN_HOME/bin:${PATH}
```

1. 点击`i`进入编辑状态
2. 首先创建一个“export + 自定义名字（例如上述的JAVA_HOME）”
3. 然后使用 export PATH= $PATH : $+自定义名字（JAVA_HOME）。（在一个PATH下添加多个环境变量后面用:号把路径拼接下来，）这样一个环境变量就添加上了。
4. 然后按 Esc 退出编辑状态，使用`:wq`退出。（有时会因为权限问题提示：E45: ‘readonly’ option is set (add ! to override)错误，这时使用`:wq!`强制保存退出就ok）
5. 注意：先定义路径（MAVEN_HOME），后用 PATH 引入（PATH），是从上到下的顺序，要不然就读不出。
   另外一点就是在 zshrc 中我最后添加了${PATH}:这里通过它引用了一些系统基础的命令，如我们刚刚使用 vim，切记一定要加上否则连系统基础命令都会报command not found。

3、保存文件后，执行source命令让其生效

```bash
### 让配置的环境变量生效, 注意修改哪个文件就加载哪个文件
source ~/.zshrc
```

4、补充 Mac 终端命令

- 如果是 vi，则：Esc 退出编辑模式，输入以下命令：
- :wq 保存后退出vi，若为 :wq! 则为强制储存后退出（常用）
- :w 保存但不退出（常用）
- :w! 若文件属性为『只读』时，强制写入该档案
- :q 离开 vi （常用）
- :q! 若曾修改过档案，又不想储存，使用 ! 为强制离开不储存档案。
- :e! 将档案还原到最原始的状态！



### 2、$PATH 与 ${PATH} 区别

在 MacOS 中，${PATH} 和 $PATH 实际上没有区别，都是用来引用环境变量 PATH 的方式。具体来说：

- $PATH 是对环境变量 PATH 的引用，直接输出它的值。
- ${PATH} 是一种更严格的语法，使用大括号将变量名包裹起来。

一般情况下，使用 $PATH 就可以正确引用变量，但当变量名旁边紧接着其他字符时，使用 `${PATH}` 可以避免歧义。例如：

```bash
echo $PATHsuffix   # 会被解释为引用 PATHsuffix 变量，可能不存在
echo ${PATH}suffix # 清楚地引用了 PATH 变量，然后附加 'suffix'
```

总结来说，在大多数情况下，${PATH} 和 $PATH 是等效的，但 `${}` 更适合在复杂情况下使用，以确保变量名的准确解析。

***

6、总结：最快的方法就是添加 export 到 .bashrc 或者 .zshrc 中。但了解整个过程，每个文件的作用还是有一定必要的。







*   



> 3、上述文件的科普

- /etc/paths （全局建议修改这个文件 ）

  编辑 paths，将环境变量添加到 paths文件中 ，一行一个路径

  Hint：输入环境变量时，不用一个一个地输入，只要拖动文件夹到 Terminal 里就可以了。

- /etc/profile （建议不修改这个文件 ）

  全局（公有）配置，不管是哪个用户，登录时都会读取该文件。

- /etc/bashrc （一般在这个文件中添加系统级环境变量）

  全局（公有）配置，bash shell执行时，不管是何种方式，都会读取此文件

- ~/.profile 文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从/etc/profile.d目录的配置文件中搜集shell的设置

  **使用注意**：如果你有对/etc/profile有修改的话必须得重启你的修改才会生效，此修改对每个用户都生效。

- ~/.bashrc 每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.

  **使用注意** 对所有的使用bash的用户修改某个配置并在以后打开的bash都生效的话可以修改这个文件，修改这个文件不用重启，重新打开一个bash即可生效。

- ~/.bash_profile 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该文件被读取.（每个用户都有一个.bashrc文件，在用户目录下）

  **使用注意** 需要需要重启才会生效，/etc/profile对所有用户生效，~/.bash_profile只对当前用户生效。



# MacOS 环境变量配置 export 关键字详解

在 macOS 或任何使用 Zsh 的系统中，当你配置环境变量时，可以通过编辑 `.zshrc` 文件（或 `.bash_profile` 等终端配置文件）来定义环境变量。关于在 `.zshrc` 文件中使用 export 和不使用 export，有以下区别：

## 1、使用 export

export 关键字会将变量导出到**当前 Shell 会话以及所有子 Shell 会话**，也就是说，当前终端的所有子进程都能访问这些被 export 的变量。

示例：

```bash
export MY_VAR="some_value"
```

此时：
- 你定义了一个名为 MY_VAR 的环境变量，并且它可以被当前终端和所有由该终端启动的子进程（如其他脚本或应用程序）访问。

例如：
```bash
$ echo $MY_VAR
some_value

$ zsh  # 打开一个子 Shell
$ echo $MY_VAR  # 子 Shell 中依然可以访问 MY_VAR
some_value
```



## 2、不使用 export

如果只在 `.zshrc` 中定义变量但**不使用 export**，该变量仅在**当前 Shell 会话中可见**，子 Shell 或子进程将无法访问该变量。

示例：

```bash
MY_VAR="some_value"
```

此时：
- 你定义了一个名为 MY_VAR 的变量，但它仅限于当前终端会话。如果你从当前终端启动一个新的终端会话（子 Shell），这个变量将不会被继承。

例如：
```bash
$ echo $MY_VAR
some_value

$ zsh  # 打开一个子 Shell
$ echo $MY_VAR  # 子 Shell 中无法访问 MY_VAR
# (没有输出，因为 MY_VAR 没有被 export)
```



## 3、export 与非 export 的区别总结

| 场景           | 使用 export                                   | 不使用 export                               |
| -------------- | --------------------------------------------- | ------------------------------------------- |
| **当前 Shell** | 变量可用                                      | 变量可用                                    |
| **子 Shell**   | 变量在子 Shell 及所有子进程中都可访问         | 变量在子 Shell 和子进程中不可访问           |
| **适用场景**   | 需要将变量传递给当前 Shell 的子进程或子 Shell | 仅限于当前 Shell 环境的本地变量，不需要传递 |



## 4、何时使用 export？

- **需要将环境变量传递给子进程时**，如启动其他程序、运行脚本等，都需要使用 export。
- **全局配置变量**，比如设置 PATH、JAVA_HOME、PYTHONPATH 等，通常都需要使用 export，以便 Shell 可以在子进程中使用这些环境变量。

例子：

```bash
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
```

这样配置的 PATH 和 JAVA_HOME 将对所有启动的子进程生效。



## 5、何时不需要 export？

- **局部使用变量**时，如果只是在当前 Shell 会话中使用，并不需要子 Shell 或子进程访问这个变量，就可以省略 export。

例子：

```bash
MY_VAR="temporary_value"
```
此时，`MY_VAR` 只在当前 Shell 可用，不会污染子进程的环境。



## 6、常见的环境变量示例

通常，环境变量如 PATH、JAVA_HOME、PYTHONPATH、LANG 等需要通过 export 导出，使其对当前终端会话和所有子进程可用。

设置 PATH 环境变量：

```bash
export PATH="/usr/local/bin:$PATH"
```

设置 Java 环境变量：

```bash
export JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_221.jdk/Contents/Home"
```



总结

- **使用 export**：变量将会成为环境变量，当前 Shell 和子 Shell 都能访问该变量。
- **不使用 export**：变量只在当前 Shell 会话中有效，子 Shell 和子进程无法访问。

通常，对于需要跨进程、跨子 Shell 访问的环境变量（如 PATH、JAVA_HOME 等），应使用 export。对于只在当前会话中使用的变量，则可以不使用 export。

# MacOS 配置环境变量 + 快速切换 JDK 版本

> 2024年最新MacBook苹果电脑安装JDK8、JDK11、JDK17、JDK22教程，配置环境变量 + 快速切换JDK版本：https://blog.csdn.net/Lwehne/article/details/135867512

## 1、官网下载安装及配置「推荐」

从如下官网中找到指定版本 JDK 下载：

- Open JDK「推荐」：https://jdk.java.net/archive/
- Java Archive | Oracle：https://www.oracle.com/java/technologies/downloads/archive/
- Java Archive | Oracle 中国：https://www.oracle.com/cn/java/technologies/downloads/archive/

解压到指定目录即安装完毕。本人安装了 JDK11 和 JDK17。

```bash
 ~  ls ~lsx/Java/JDK
jdk-11.jdk                   openjdk-11_osx-x64_bin.tar
jdk-17.jdk                   openjdk-17_macos-x64_bin.tar
```

1、如果只安装一个版本 JDK，如 JDK11 复制下面代码粘贴到配置文件中即可。

```bash
 ~  vim .zshrc

# JDK Config
export JAVA_HOME=/Users/lsx/Java/JDK/jdk-11.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```

2、如果想安装多个版本，比如像我一样安装 JDK11 和 JDK17，用下面这段代码。

```bash
 ~  vim .zshrc

# JDK Config Start
JAVA_HOME_11=/Users/lsx/Java/JDK/jdk-11.jdk/Contents/Home
JAVA_HOME_17=/Users/lsx/Java/JDK/jdk-17.jdk/Contents/Home

export JAVA_HOME=$JAVA_HOME_11
alias jdk11="export JAVA_HOME=$JAVA_HOME_11 && echo current JDK has switched to openjdk version 11. && java -version"
alias jdk17="export JAVA_HOME=$JAVA_HOME_17 && echo current JDK has switched to openjdk version 17. && java -version"

export PATH=$PATH:$JAVA_HOME/bin
# JDK Config End
```

3、重新加载配置文件让其生效「建议最好重启终端」：

```bash
source ~/.zshrc
```

4、验证 JDK 切换

```bash
 ~  jdk17
current JDK has switched to openjdk version 17.
openjdk version "17" 2021-09-14
OpenJDK Runtime Environment (build 17+35-2724)
OpenJDK 64-Bit Server VM (build 17+35-2724, mixed mode, sharing)
 ~  java -version
openjdk version "17" 2021-09-14
OpenJDK Runtime Environment (build 17+35-2724)
OpenJDK 64-Bit Server VM (build 17+35-2724, mixed mode, sharing)
 ~  jdk11
current JDK has switched to openjdk version 11.
openjdk version "11" 2018-09-25
OpenJDK Runtime Environment 18.9 (build 11+28)
OpenJDK 64-Bit Server VM 18.9 (build 11+28, mixed mode)
 ~  java -version
openjdk version "11" 2018-09-25
OpenJDK Runtime Environment 18.9 (build 11+28)
OpenJDK 64-Bit Server VM 18.9 (build 11+28, mixed mode)
 ~ 
```



## 2、通过 Homebrew 安装及配置

### 1、通过 Homebrew 安装多个版本

安装完 JDK 后，Homebrew 会给出如何配置环境变量的提示。

```bash
# 安装 Java 8
brew install openjdk@8
# 安装 Java 11
brew install openjdk@11
# 安装 Java 17
brew install openjdk@17
```



### 2、查看已安装的 JDK

macOS 将所有安装的 JDK 存放在 /Library/Java/JavaVirtualMachines/ 目录下。你可以使用以下命令查看已安装的所有 JDK 版本：

```bash
ls /Library/Java/JavaVirtualMachines/
```

你还可以使用 java_home 命令来列出系统已安装的 JDK 版本：

```bash
/usr/libexec/java_home -V

# 输出类似：
Matching Java Virtual Machines (3):
    17.0.1 (x86_64) "Oracle Corporation" - "/Library/Java/JavaVirtualMachines/jdk-17.0.1.jdk/Contents/Home"
    11.0.10 (x86_64) "AdoptOpenJDK" - "/Library/Java/JavaVirtualMachines/adoptopenjdk-11.jdk/Contents/Home"
     1.8.0_282 (x86_64) "Oracle Corporation" - "/Library/Java/JavaVirtualMachines/jdk1.8.0_282.jdk/Contents/Home"
```



### 3、配置多版本 JDK 的手动切换

你可以通过修改环境变量 JAVA_HOME 来切换不同的 JDK 版本。

**手动切换 JDK：**使用 java_home 命令指定要使用的 JDK 版本，然后将其赋值给 JAVA_HOME 环境变量。

```bash
# 切换到 Java 8
export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)
# 切换到 Java 11
export JAVA_HOME=$(/usr/libexec/java_home -v 11)
# 切换到 Java 17
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

- 通过 echo $JAVA_HOME 可以查看当前正在使用的 JDK 路径。

**设置默认 JDK：**在 .zshrc 或 .bash_profile 等配置文件中配置 JAVA_HOME，使其在每次打开终端时都自动设置。

```bash
# 设置默认 JDK 版本为 Java 11
export JAVA_HOME=$(/usr/libexec/java_home -v 11)
```



### 4、创建快捷命令来快速切换 JDK

为了方便切换，你可以在 .zshrc 中定义别名或函数来快速切换 JDK 版本。

**在 .zshrc 中添加以下函数：**

```bash
# 切换到 Java 8
jdk8() {
    export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)
    java -version
}

# 切换到 Java 11
jdk11() {
    export JAVA_HOME=$(/usr/libexec/java_home -v 11)
    java -version
}

# 切换到 Java 17
jdk17() {
    export JAVA_HOME=$(/usr/libexec/java_home -v 17)
    java -version
}
```

**使用快捷命令：**「这些函数会根据输入的命令来设置对应的 JAVA_HOME 并显示当前 Java 版本。」

```bash
# 切换到 Java 8
jdk8

# 切换到 Java 11
jdk11

# 切换到 Java 17
jdk17
```

**总结：**

- *安装多个 JDK：通过 Homebrew 或手动下载并安装多个 JDK 版本。
- 手动切换 JDK：使用 export JAVA_HOME=$(/usr/libexec/java_home -v <版本>) 来切换。
- 自动设置默认 JDK：在 .zshrc 中配置 JAVA_HOME，使其自动设置。
- 快速切换 JDK**：通过在 .zshrc 文件中定义快捷命令或函数，方便快速切换 JDK 版本。



# MacOS 安装 Homebrew

## 1、Homebrew 国内安装

苹果电脑安装脚本：

```bash
# 常规安装脚本（完全体 推荐） 
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

```bash
# 极速安装脚本（精简版）
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)" speed
```

苹果电脑卸载脚本：

```bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"
```

在终端输入安装脚本之后，会让你选择下载源：选中科大、清华或者是北京外国语都可以，都是完整版，而且下载速度也都很快。

然后 Homebrew 就会开始安装，过程中如果要求输入密码，那输入开机密码敲回车就行，当最后提示 「需要重启终端 才能使用 Homebrew 」的时候，就表示 Homebrew 已经安装成功了。



## 2、Homebrew 官方安装

Homebrew 简称 brew。官网：https://brew.sh/。

- Homebrew 是一个免费的开源包管理系统，用于 MacOS 和基于 Linux 的系统，简化安装过程。

- 对 MacOS 用户安装 App 很友好，命令式一件安装，方便快捷，且 App 镜像资源丰富。

1、终端中输入如下命令即可安装（由于 Homebrew 是国外软件，下载源基本也在国外，如果下载速度慢，可以[配置使用镜像](https://blog.csdn.net/lwplwf/article/details/79097565)）；

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

2、能安装就能卸载，终端中输入如下命令即可卸载：

```bash
# install 替换成 uninstall 即可
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

3、安装完毕后需要配置环境变量：

```bash
# 如果你使用的是 zsh（默认在 macOS 上）
nano ~/.zshrc
# 如果你使用的是 bash
nano ~/.bash_profile
```

4、在文件中添加以下内容：

```bash
export PATH="/opt/homebrew/bin:$PATH"
```

5、保存并退出编辑器：

- 按 Ctrl + O 保存文件，然后按 Enter 确认。
- 按 Ctrl + X 退出编辑器。

6、重新加载 shell 配置文件：

```bash
# 对于 zsh
source ~/.zshrc
# 对于 bash
source ~/.bash_profile
```

7、你可以通过以下命令检查 brew 是否在 PATH 中：

```bash
which brew
```

8、这应该返回 Homebrew 的路径 `/opt/homebrew/bin/brew`，表示配置成功。



## 3、Homebrew 干净卸载

上面安装教程中提到如何卸载 Homebrew 的命令：

```bash
# 国内镜像卸载
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"
# 官方方式卸载
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

此方式都无法干净卸载电脑中的 Homebrew。如果想干净卸载，可以按照以下步骤进行干净卸载，确保删除所有相关的文件和目录。



### 1、使用 Homebrew 自带的卸载脚本

Homebrew 官方提供了一个卸载脚本，可以自动删除所有 Homebrew 相关的文件和目录。运行以下命令来下载并执行这个脚本：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

这将会卸载 Homebrew，不论是从官方源还是国内镜像源安装的。



### 2、手动卸载（如果脚本未完全清理）

如果脚本没有完全卸载 Homebrew，你可以手动删除 Homebrew 的相关目录和文件。根据你的系统架构，执行以下命令：

1、对于 Intel 架构的 Mac：

```bash
sudo rm -rf /usr/local/Homebrew
sudo rm -rf /usr/local/Cellar
sudo rm -rf /usr/local/Caskroom
sudo rm -rf /usr/local/Frameworks
sudo rm -rf /usr/local/bin/brew
```

2、对于 Apple Silicon 架构的 Mac：

```bash
sudo rm -rf /opt/homebrew
```



### 3、清理 Homebrew 的环境变量

Homebrew 安装时，可能在你的 shell 配置文件（如 `~/.bash_profile`、`~/.zshrc`）中添加了 `brew` 相关的环境变量配置。你可以按照以下步骤手动删除这些配置：

1. 编辑 `~/.bash_profile`、`~/.zshrc` 或 `~/.bashrc` 文件：
   ```bash
   nano ~/.bash_profile
   # 或者 nano ~/.zshrc (取决于你使用的 shell)
   ```

2. 找到包含 `brew` 的配置行，例如：
   ```bash
   export PATH="/usr/local/bin:$PATH"
   export PATH="/opt/homebrew/bin:$PATH"
   ```

3. 删除这些行，保存并退出编辑器。

4. 重新加载配置文件：
   ```bash
   source ~/.bash_profile
   # 或者 source ~/.zshrc
   ```



### 4、删除残留的 Homebrew 文件

Homebrew 可能会留下其他一些日志或缓存文件，你可以进一步清理：

```bash
sudo rm -rf ~/Library/Caches/Homebrew
sudo rm -rf ~/Library/Logs/Homebrew
sudo rm -rf /Library/Logs/Homebrew
sudo rm -rf ~/Library/Preferences/org.homebrew.*
```

***

总结，按照如下步骤这样你就可以干净地卸载通过国内镜像源安装的 Homebrew。

1. 首先运行 Homebrew 官方的卸载脚本。
2. 如果需要，可以手动删除相关目录和环境变量。
3. 确保清理 Homebrew 相关的缓存和日志文件。



## 4、Homebrew 常用命令

1. brew update ：更新 Homebrew 至最新版本

2. brew search 软件包名 ：搜索需要的安装包

3. brew install 软件包名 ：安装指定的安装包

4. brew uninstall 软件包名 ：卸载指定的安装包

5. brew list ：列出已安装的包名

   ```bash
   lsx@MacBook-Air Cellar % brew list
   ==> Formulae
   gettext		libassuan@2	pcre2
   git		libgpg-error	pinentry-mac
   ==> Casks
   iterm2

6. brew upgrade 软件包名 ：升级所有已安装的包

7. brew outdated ：查看所有可更新的包

8. brew doctor ：检测 Homebrew 环境是否健康

9. brew info 软件包名 ：查看包信息

10. brew cleanup ：清理旧版本的包及缓存

11. brew tap ：添加或删除外部tap源

12. brew search --desc 关键词：搜索描述中包含特定关键词的包

13. brew search --cask 软件包名 ：搜索cask包

14. brew install --cask 软件包名 ：安装指定的cask包

15. brew uninstall --cask 软件包名 ：卸载指定的cask包

16. brew services ：管理 Homebrew 服务

17. brew link ：连接包的二进制文件到PATH中

18. brew unlink ：取消连接包的二进制文件

19. brew style ：检查 Homebrew 包的代码风格

20. brew edit ：编辑指定的 Homebrew 包信息和代码



## 5、Homebrew 默认安装路径

1、通过 brew install 安装应用最先是放在/usr/local/Cellar/目录下。

2、有些应用会自动创建[软链接](https://so.csdn.net/so/search?q=软链接&spm=1001.2101.3001.7020)放在/usr/bin或者/usr/sbin，同时也会将整个文件夹放在/usr/local下。

***

在 MacOS 中，Homebrew 的默认安装路径是：

- `/usr/local`：适用于基于 Intel 的 Mac 设备。
- `/opt/homebrew`：适用于基于 Apple Silicon (M1/M2) 的 Mac 设备。

1、具体目录结构：

- **基于 Intel 的 Mac**：
  - Homebrew 本身安装在 `/usr/local/Homebrew`。
  - 软件包（formulae）和工具安装在 `/usr/local/Cellar`。
  - 可执行文件的符号链接位于 `/usr/local/bin`，以确保可以在命令行中直接调用安装的工具。

- **基于 Apple Silicon 的 Mac**：
  - Homebrew 本身安装在 `/opt/homebrew`。
  - 软件包安装在 `/opt/homebrew/Cellar`。
  - 可执行文件的符号链接位于 `/opt/homebrew/bin`。

2、查看 Homebrew 的安装路径。你可以通过以下命令查看当前 Homebrew 的安装路径：

```bash
brew --prefix
```

该命令会输出 Homebrew 的安装目录，例如 `/usr/local` 或 `/opt/homebrew`，取决于你的 Mac 设备架构。





## 6、Homebrew Cask 详解

### 1、Homebrew Cask 如何安装？

Homebrew Cask 是 Homebrew 的一部分，你只需要安装 Homebrew，然后就可以直接使用 Homebrew Cask 来安装图形界面应用程序，无需单独安装 Homebrew Cask。以下是安装 Homebrew 及使用 Cask 的步骤：

1. 安装 Homebrew「如果你还没有安装 Homebrew，可以通过终端运行以下命令进行安装」：

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

    安装成功后，你可以通过运行以下命令确认 Homebrew 是否安装成功：

   ```bash
   brew --version
   ```

2. 使用 Homebrew Cask 安装 GUI 应用程序：

   安装好 Homebrew 后，你可以直接使用 `brew install --cask` 命令来安装图形界面的应用程序。以下是一些常见应用的安装示例：

   ```bash
   # 安装 Google Chrome
   brew install --cask google-chrome
   # 安装 VSCode
   brew install --cask visual-studio-code
   # 安装 Spotify
   brew install --cask spotify
   ```

3. 查看已安装的 Cask 软件「你可以使用以下命令查看通过 Cask 安装的所有软件」：

   ```bash
   brew list --cask
   ```

4. 卸载 Cask 软件「如果需要卸载一个通过 Cask 安装的应用程序，使用以下命令」：

   ```bash
   brew uninstall --cask google-chrome
   ```

1. 通过上述步骤，你就可以使用 Homebrew Cask 来方便地管理 macOS 上的图形界面应用程序了。

***



### 2、如何知道何时使用`--cask`参数？

要知道是否需要使用 `--cask` 参数，通常取决于你想要安装的软件类型：

1. 命令行工具和开发库（例如 `git`、`node`、`python`）通常通过 `brew install` 安装，无需 `--cask` 参数。
2. 图形界面应用程序（GUI 应用）（例如 Google Chrome、Visual Studio Code、Spotify 等）需要通过 `brew install --cask` 来安装。

判断是否需要 `--cask` 参数的几种方式：

1. 检查软件类型：
   - 如果是**命令行工具**或**库**，不需要 `--cask` 参数。
   - 如果是**桌面应用程序**，一般需要加上 `--cask`。

2. 通过 `brew search` 搜索软件：
   - 你可以通过 `brew search` 命令来搜索软件包。如果结果中带有 `cask`，表示这是一个 GUI 应用，需要使用 `--cask` 来安装。
     ```bash
     brew search <软件名>
     brew search google-chrome
     ```
     如果结果显示为 `google-chrome (cask)`，则意味着需要使用 `brew install --cask google-chrome` 来安装。
   
3. 通过 Homebrew 官网或 `brew info` 查看软件详情：
   - 你可以通过访问 [Homebrew Cask 官网](https://formulae.brew.sh/cask/) 或使用 `brew info` 命令查看某个软件包的安装说明。
     ```bash
     brew info <软件名>
     brew info google-chrome
     ```
     该命令会告诉你该软件包是通过 Cask 管理的，意味着你需要使用 `--cask` 参数。

总结：

- 如果你要安装的是命令行工具或开发库，直接使用 `brew install` 即可。
- 如果你要安装的是图形界面的应用程序，则需要使用 `brew install --cask`。

***



### 3、安装图形界面应用未加`--cask`会如何？

如果在安装图形界面应用时没有加 `--cask` 参数，通常会有以下几种情况：

1、Homebrew 找不到对应的命令行工具。如果你尝试安装的是图形界面的应用程序，但没有加 `--cask` 参数，Homebrew 会尝试查找命令行工具或库，通常会提示找不到该软件包。例如：

   ```bash
   brew install google-chrome
   # 输出可能会是
   Error: No available formula with the name "google-chrome"
   ```
2、安装错误的软件。在某些情况下，Homebrew 可能会找到一个同名的命令行工具或库（如果存在）。但是，这不是你想要的图形界面应用程序。例如：

   ```bash
   brew install firefox
   ```
如果存在一个命令行工具或库同名为 `firefox`，Homebrew 会安装该工具，而不是你想要的 Firefox 浏览器 GUI 应用。

3、正确安装命令行工具时无需 `--cask`：如果你确实是在安装命令行工具，则即使不加 `--cask`，也不会有问题。例如：

   ```bash
   brew install git
   ```
这个命令不会需要 `--cask`，因为 `git` 是一个命令行工具，Homebrew 会直接安装它。

解决办法：

- 如果你不确定软件包是否需要 `--cask`，可以先使用 `brew search <软件名>` 来搜索它。如果结果带有 `(cask)`，那么你应该使用 `--cask` 参数进行安装。
- 你还可以通过 `brew info <软件名>` 查看更详细的安装信息，看看该包是否是通过 Cask 管理的。

总结：

如果你没有加 `--cask` 而试图安装图形界面应用程序，通常 Homebrew 会提示找不到该包，或安装错误的软件。解决方法是确认该应用是 GUI 应用，然后加上 `--cask` 参数。



## 7、Homebrew 与 Homebrew Cask 区别

Homebrew 和 Homebrew Cask 是 macOS 下的软件包管理工具，但它们的用途有所不同：

1. Homebrew：
   - 专注于**命令行工具**、**开发库**以及轻量级软件的安装和管理。
   - 用于安装和管理像 `git`、`wget`、`python` 这样的命令行应用或开发依赖。
   - 安装命令是 `brew install package_name`。
   - 适合开发者安装与管理系统工具、编译器、库等。

2. Homebrew Cask：
   - 用于安装和管理**图形界面应用程序**和**大文件**。
   - 适用于安装常用的 GUI 应用程序，如 Google Chrome、Slack、Spotify 等。
   - 安装命令是 `brew install --cask package_name`。
   - 提供方便的方式来安装大多数常见的 macOS 应用程序，而无需手动下载 DMG 或 PKG 文件。

总结：

- Homebrew 用于安装命令行工具、开发库等软件包。
- Homebrew Cask 用于安装带有图形界面的应用程序。

两者都通过 Homebrew 项目管理，但用途不同，可以互补使用。



## 8、Homebrew 安装软件

### 1、安装 Redis

```bash
# 安装
brew install redis

# 启动redis
brew services start redis

# 查看一下版本（复制如下代码到终端运行）：
redis-server

# 查看redis进程
ps axu | grep redis

# 停止redis
brew services stop redis

# 使用的话，在开一个iTerm
(base) ➜  ~ redis-cli -h 127.0.0.1 -p 6379
```

- Redis 的 server 命令在`/opt/homebrew/opt/redis/bin/redis-serve`
- Redis 的 conf 在`/opt/homebrew/etc/redis.conf`



### 2、安装 NVM

`NVM（Node Version Manager）`是一个用于管理 Node.js 版本的工具。它允许您在同一台计算机上安装和切换不同的 Node.js 版本。

```bash
brew install nvm
```

- 查看所有已安装的 NodeJS 版本（nvm安装的）：nvm ls

- nvm install 版本号安装特定版本的 NodeJS
- nvm use 版本号使用指定版本的 NodeJS
- nvm alias default 版本号 命令设置默认版本 NodeJS
- nvm uninstall 版本号 卸载特定的 NodeJS



# MacOS 终端安装与配置

> 1. MacBook配置一套自己喜欢的高效的开发环境：https://blog.csdn.net/u010347226/article/details/82598160
> 2. 为啥人家的命令行终端如此炫酷？原来用了这款137K+Star的神器！https://mp.weixin.qq.com/s/BSACrwGSyMKl4O13Q2NHkw
> 3. 程序员必备终端神器ohmyzsh的安装与配置过程（保姆级）https://mp.weixin.qq.com/s/DflwRPdYTMvkDMIg0Zno_w
> 4. Linux终端环境配置：https://www.geekhour.net/2023/10/21/linux-terminal/

## 1、安装字体「按需安装」

### 1、Nerd 字体安装

终端的一些 iconfont 需要一些特殊字体才能完美显示，推荐使用 Nerd 字体，官网：[nerdfonts.com/](https://nerdfonts.com/)。

powerlevel10k 主题推荐使用 MesloLGS-Nerd 字体，一般在初次安装配置主题的时候会默认提示安装，但是如果没有正常安装的话也可以使用下面的内容来手动安装一下：

1. MesloLGS 字体 ttf 文件下载地址：

   ```bash
   wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf &&
   wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf  &&
   wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf  &&
   wget https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf
   ```

2. 或者 MacOS 也可以使用 Homebrew 来安装

   ```bash
   # Mac homebrew
   brew tap homebrew/cask-fonts &&
   brew install --cask font-<FONT NAME>-nerd-font
   
   e.g.
   brew tap homebrew/cask-fonts
   brew install --cask font-code-new-roman-nerd-font
   ```

3. 安装完成之后再系统设置或者各个软件比如终端或者 VSCode or iTerm2 上把字体设置为`MesloLGS NF`就可以了。

***

### 2、Powerline fonts 字体安装「推荐」

安装特殊字体：Powerline fonts，官网：https://github.com/powerline/fonts。

安装Powerline fonts字体，并在 iTerm2 中设置字体为：Meslo LG M DZ Regular for Powerline即可；安装步骤及命令如下：

```bash
# clone  下载字体
git clone https://github.com/powerline/fonts.git --depth=1
# install 安装字体
cd fonts
./install.sh
# clean-up a bit  安装完成，删除下载的字体
cd ..
rm -rf fonts
# uninstall 卸载字体，不要该字体时再卸载，这个命令备用
./uninstall.sh
```

特殊字体安装完后，为 iTerm2 设置字体：Meslo LG M DZ Regular for Powerline；按照如下的顺序，依次点击：Profiles-Text-Change Font，这里，Font 和 Non-ASCLL Font 都要设置为特殊字体；



## 2、安装及配置 iTerm2

iTerm2 被认为是 MacOS 下最好用的终端工具！

### 1、iTerm2 的安装

- 安装方式一：直接从官网 http://iterm2.com/ 下载并解压，然后把它拖到应用程序里面就行。

- 安装方式二：我们这里直接使用 Homebrew Cask 进行安装，终端输入如下命令：

  ```bash
  brew install --cask iterm2
  ```





### 2、iTerm2 字体配置

上述特殊字体安装完后，为 iTerm2 设置字体：Meslo LG M DZ Regular for Powerline；按照如下的顺序，依次点击：Profiles-Text-Change Font，这里，Font 和 Non-ASCLL Font 都要设置为特殊字体；



### 3、iTerm2 主题配置

1. 先下载`iTerm2`的所有主题「或者直接进入 [Iterm2-color-schemes](https://iterm2colorschemes.com/) 网站下载解压」：

   ```bash
   git clone https://github.com/mbadolato/iTerm2-Color-Schemes --depth=1
   ```

2. 然后按如下顺序打开：

   ```bash
   iTerm2 > Preferences > Profiles > Colors > Color Presets
   ```

3. 就可以点击 `Import` 选项，导入刚下载好的主题。这里本人选择「iTerm2 Solarized Dark」主题。



### 4、修改主题适配自动补全

如果你在使用命令行的时候，发现没有自动补全的结果，可能是主题的颜色区分度不够，可以试着修改下颜色，打开选项中的 Profiles，切换到 Colors，修改 ANSI Colors 中的 Bright 选项颜色：与背景颜色区分开来即可。



## 3、安装及配置 Oh-My-Zsh

### 1、安装及设置 zsh

zsh shell 官网：https://github.com/zsh-users

MacOS 系统中默认的 shell 为 bash shell，但是我们查看 Mac 系统中已经提供的 shell，可以发现系统提供了很多个 shell，现在需要将系统默认使用的 bash shell 改为 zsh shell，终端输入如下命令：

```bash
## 查看本地内置了几种shell：
cat /etc/shells

## 设置zsh为默认shell
chsh -s /bin/zsh
```



### 2、安装 Oh-My-Zsh　

#### 1、如何安装

> 三种方法都行，任选其一

```bash
# 方式一: 原始安装（墙外）GitHub sh 脚本安装
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 方式二: 镜像安装（墙内）：
sh -c "$(curl -fsSL https://install.ohmyz.sh/)"

# 方式三: 手动安装, 还需要手动创建zsh配置文件
git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh --depth=1
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

#### 2、安装过程

```bash
lsx@MacBook-Air ~ % sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
Cloning Oh My Zsh...
remote: Enumerating objects: 1407, done.
remote: Counting objects: 100% (1407/1407), done.
remote: Compressing objects: 100% (1347/1347), done.
remote: Total 1407 (delta 36), reused 1129 (delta 32), pack-reused 0 (from 0)
Receiving objects: 100% (1407/1407), 3.26 MiB | 11.87 MiB/s, done.
Resolving deltas: 100% (36/36), done.
From https://github.com/ohmyzsh/ohmyzsh
 * [new branch]      master     -> origin/master
Branch 'master' set up to track remote branch 'master' from 'origin'.
Already on 'master'
/Users/lsx

Looking for an existing zsh config...
Found /Users/lsx/.zshrc. Backing up to /Users/lsx/.zshrc.pre-oh-my-zsh
Using the Oh My Zsh template file and adding it to /Users/lsx/.zshrc.

         __                                     __   
  ____  / /_     ____ ___  __  __   ____  _____/ /_  
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \ 
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / / 
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/  
                        /____/                       ....is now installed!


Before you scream Oh My Zsh! look over the `.zshrc` file to select plugins, themes, and options.

• Follow us on X: https://x.com/ohmyzsh
• Join our Discord community: https://discord.gg/ohmyzsh
• Get stickers, t-shirts, coffee mugs and more: https://shop.planetargon.com/collections/oh-my-zsh

compinit:527: no such file or directory: /usr/local/share/zsh/site-functions/_brew
➜  ~ 
```



#### 3、安装后的变化

出现上述结果表明安装成功！那么该安装结果给你的系统带来了哪些变化呢？

- 用户家目录下增加了两个内容，其中一个是.zshrc配置文件，该文件内容如下所示：

  ```bash
  # If you come from bash you might have to change your $PATH.
  # export PATH=$HOME/bin:/usr/local/bin:$PATH
  
  # Path to your oh-my-zsh installation.
  export ZSH="$HOME/.oh-my-zsh"
  
  # Set name of the theme to load --- if set to "random", it will
  # load a random theme each time oh-my-zsh is loaded, in which case,
  # to know which specific one was loaded, run: echo $RANDOM_THEME
  # See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
  ZSH_THEME="robbyrussell"
  
  # Set list of themes to pick from when loading at random
  # Setting this variable when ZSH_THEME=random will cause zsh to load
  # a theme from this variable instead of looking in $ZSH/themes/
  # If set to an empty array, this variable will have no effect.
  # ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )
  
  # Uncomment the following line to use case-sensitive completion.
  # CASE_SENSITIVE="true"
  
  # Uncomment the following line to use hyphen-insensitive completion.
  # Case-sensitive completion must be off. _ and - will be interchangeable.
  # HYPHEN_INSENSITIVE="true"
  
  # Uncomment one of the following lines to change the auto-update behavior
  # zstyle ':omz:update' mode disabled  # disable automatic updates
  # zstyle ':omz:update' mode auto      # update automatically without asking
  # zstyle ':omz:update' mode reminder  # just remind me to update when it's time
  
  # Uncomment the following line to change how often to auto-update (in days).
  # zstyle ':omz:update' frequency 13
  
  # Uncomment the following line if pasting URLs and other text is messed up.
  # DISABLE_MAGIC_FUNCTIONS="true"
  
  # Uncomment the following line to disable colors in ls.
  # DISABLE_LS_COLORS="true"
  
  # Uncomment the following line to disable auto-setting terminal title.
  # DISABLE_AUTO_TITLE="true"
  
  # Uncomment the following line to enable command auto-correction.
  # ENABLE_CORRECTION="true"
  
  # Uncomment the following line to display red dots whilst waiting for completion.
  # You can also set it to another string to have that shown instead of the default red dots.
  # e.g. COMPLETION_WAITING_DOTS="%F{yellow}waiting...%f"
  # Caution: this setting can cause issues with multiline prompts in zsh < 5.7.1 (see #5765)
  # COMPLETION_WAITING_DOTS="true"
  
  # Uncomment the following line if you want to disable marking untracked files
  # under VCS as dirty. This makes repository status check for large repositories
  # much, much faster.
  # DISABLE_UNTRACKED_FILES_DIRTY="true"
  
  # Uncomment the following line if you want to change the command execution time
  # stamp shown in the history command output.
  # You can set one of the optional three formats:
  # "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
  # or set a custom format using the strftime function format specifications,
  # see 'man strftime' for details.
  # HIST_STAMPS="mm/dd/yyyy"
  
  # Would you like to use another custom folder than $ZSH/custom?
  # ZSH_CUSTOM=/path/to/new-custom-folder
  
  # Which plugins would you like to load?
  # Standard plugins can be found in $ZSH/plugins/
  # Custom plugins may be added to $ZSH_CUSTOM/plugins/
  # Example format: plugins=(rails git textmate ruby lighthouse)
  # Add wisely, as too many plugins slow down shell startup.
  plugins=(git)
  
  source $ZSH/oh-my-zsh.sh
  
  # User configuration
  
  # export MANPATH="/usr/local/man:$MANPATH"
  
  # You may need to manually set your language environment
  # export LANG=en_US.UTF-8
  
  # Preferred editor for local and remote sessions
  # if [[ -n $SSH_CONNECTION ]]; then
  #   export EDITOR='vim'
  # else
  #   export EDITOR='mvim'
  # fi
  
  # Compilation flags
  # export ARCHFLAGS="-arch x86_64"
  
  # Set personal aliases, overriding those provided by oh-my-zsh libs,
  # plugins, and themes. Aliases can be placed here, though oh-my-zsh
  # users are encouraged to define aliases within the ZSH_CUSTOM folder.
  # For a full list of active aliases, run `alias`.
  #
  # Example aliases
  # alias zshconfig="mate ~/.zshrc"
  # alias ohmyzsh="mate ~/.oh-my-zsh"
  ```

- 另一个变化就是增加了隐藏目录.oh-my-zsh，目录具体内容如下所示：

  ```bash
   ~  ls -la ~/.oh-my-zsh
  total 152
  drwxr-xr-x   23 lsx  staff    736 10  3 14:23 .
  drwxr-xr-x+  52 lsx  staff   1664 10  4 00:07 ..
  -rw-r--r--    1 lsx  staff    139 10  3 14:23 .editorconfig
  drwxr-xr-x   13 lsx  staff    416 10  3 14:23 .git
  drwxr-xr-x    9 lsx  staff    288 10  3 14:23 .github
  -rw-r--r--    1 lsx  staff    109 10  3 14:23 .gitignore
  -rw-r--r--    1 lsx  staff    131 10  3 14:23 .gitpod.Dockerfile
  -rw-r--r--    1 lsx  staff    259 10  3 14:23 .gitpod.yml
  -rw-r--r--    1 lsx  staff     49 10  3 14:23 .prettierrc
  -rw-r--r--    1 lsx  staff   3374 10  3 14:23 CODE_OF_CONDUCT.md
  -rw-r--r--    1 lsx  staff   9211 10  3 14:23 CONTRIBUTING.md
  -rw-r--r--    1 lsx  staff   1142 10  3 14:23 LICENSE.txt
  -rw-r--r--    1 lsx  staff  20699 10  3 14:23 README.md
  -rw-r--r--    1 lsx  staff    955 10  3 14:23 SECURITY.md
  drwxr-xr-x    8 lsx  staff    256 10  3 21:48 cache
  drwxr-xr-x    5 lsx  staff    160 10  3 14:23 custom
  drwxr-xr-x   24 lsx  staff    768 10  3 14:23 lib
  drwxr-xr-x    3 lsx  staff     96 10  3 23:46 log
  -rw-r--r--    1 lsx  staff   7472 10  3 14:23 oh-my-zsh.sh
  drwxr-xr-x  342 lsx  staff  10944 10  3 14:23 plugins
  drwxr-xr-x    3 lsx  staff     96 10  3 14:23 templates
  drwxr-xr-x  145 lsx  staff   4640 10  3 14:23 themes
  drwxr-xr-x    9 lsx  staff    288 10  3 14:23 tools
  ```



#### 4、解决环境变量失效问题

这时时候许多人都以为大功告成，但是你会发现之前你配置了环境变量的东西都不起作用了，比如：JAVA_HOME，MAVEN_HOME 等。这是因为之前配置的环境变量，都是存放在~/.bash_profile文件内，但是他只对之前的bash起作用，而我们现在换成了zsh shell。又或者之前原本是使用 zsh shell，可是在安装 Oh-My-Zsh时，默认把原来的 ~/.zshrc 文件给覆盖了。

解决办法

- 非常简单，只要把 ~/.bash_profile 里面我们配置的环境变量全部 copy 到 ~/.zshrc 文件的最后面并刷新配置文件 source ~/.zshrc 即可。
- 或者安装 Oh-My-Zsh 前提前备份好 ~/.zshrc 文件，安装完毕后合并一下  ~/.zshrc 文件即可。



### 3、如何使用插件（plugin）

1、/Users/username/.oh-my-zsh/plugins 目录里面已经内置了很多的插件，关于每个插件的功能展示可以参考官方文档。如果想要使用某个插件，只需要在 ~/.zshrc 文件中的 plugins 下设置该插件的名字即可！！

2、推荐几个有用的插件：

1. git：定义了很多git相关命令的别名和功能函数脚本
2. aliases：更好用的管理命令别名的工具，例如可以分组显示别名信息
3. tmux：定义了很多tmux相关命令的别名
4. z：目录之间的快速跳转
5. zsh-syntax-highlighting：命令行语法高亮（属于第三方插件，需要自己安装）
6. zsh-autosuggestions：自动补全（属于第三方插件，需要自己安装）
7. web-search：web搜索命令（属于第三方插件，需要自己安装）

3、更多内容参考官方文档：https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins

4、第三方插件如何使用（举个第三方插件的例子）

- 将插件安装在 ~/.oh-my-zsh/custom/plugins 目录下
- 在 ~/.zshrc 文件中的 plugins 下设置该插件的名



### 4、如何使用主题（Theme）

主题官网预览：

- 内置主题：https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
- 外部主题：https://github.com/ohmyzsh/ohmyzsh/wiki/External-themes

GitHub 有公开的所有内置主题样式，外置的也有要自己下载。主题这个因人而异了，一般是初期有新鲜感，过后哪个方便用哪个。常用`philips`,`alien-minimal`,`dst`,`duellj`几个。

1. /Users/usrname/.oh-my-zsh/themes 目录里面已经内置了很多的主题，关于每个主题的效果展示可以参考官方文档。如果想要使用某个主题，只需要在 ~/.zshrc 文件中的 ZSH_THEME 下设置该主题的名字即可！！
2. 修改完该文件，如果想要在当前 Shell 窗口生效，则记得使用 source 命令重新激活一下！要不然就新开启一个 Shell 的标签或者窗口，要不然就关闭并重新打开 Shell 窗口！（后面会推荐使用另一个命令）
3. 官方使用说明：`In order to enable a theme, set ZSH_THEME to the name of the theme in your ~/.zshrc, before sourcing Oh My Zsh; for example: ZSH_THEME=robbyrussell If you do not want any theme enabled, just set ZSH_THEME to blank: ZSH_THEME=""`（不想使用任何主题就设置ZSH_THEME=""即可）
4. 更多内容参考官方文档：https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
5. 第三方主题如何使用（举个第三方主题的例子）
   - 将主题安装在 ~/.oh-my-zsh/custom/themes目录下
   - 在 ~/.zshrc 文件中的 ZSH_THEME 下设置该主题的名字

***

主题配置如下：

```bash
# 配置文件
vim ~/.zshrc
# 默认主题，修改引号内内容即可, 默认为 robbyrussell
ZSH_THEME="agnoster"
```

- agnoster 主题需要 Meslo 字体，否则会出现乱码，点这里下载：https://github.com/powerline/fonts/tree/master/Meslo%20Slashed。
- 安装好字体后，在 iTerm 中应用字体 iTerm -> Preferences -> Profiles -> Text -> Change Font 选择安装好的字体即可。



### 5、如何更新、卸载、配置文件更新如何加载

通过这些命令，你可以非常方便地管理 Oh My Zsh 的配置、插件和主题。

```bash
# 1. 查看更多 omz 命令
# 可以查看更多的omz命令，通过这些命令也可以管理插件、主题等（通过上述的CLI命令管理则不需要重新加载.zshrc文件）
omz help

# 2. 更新
omz update

# 3. 卸载
uninstall_oh_my_zsh

# 4. 使用 omz plugin list 命令列出所有可用的插件：
omz plugin list

# 5. 使用 omz theme list 来列出所有可用的主题：
omz theme list

# 6. 使用 omz theme set <theme_name> 来更改当前主题, 例如，设置 agnoster 主题：
omz theme set agnoster

# 7. 修改完配置文件重新加载「推荐」又称：恢复 Oh My Zsh 的默认配置
# 请避免使用source命令，原因请参考官方文档: https://github.com/ohmyzsh/ohmyzsh/wiki/FAQ#how-do-i-reload-the-zshrc-file
omz reload
```



### 6、Oh-My-Zsh 还有哪些更强大的功能

1、alias：列出系统所有的命令别名（alias 是 Shell 内置命令，在没有配置其他别名的前提下就是 ohmyzsh 内置的所有别名，换句话说就是 ohmyzsh 帮我们设置好了一些别名）

- 如果想自己定义其他的命令别名，可以在 ~./zshrc 文件中进行定义

2、mkcd：创建目录并进入「由 ohmyzsh 创建了该命令」

3、zsh_stats：列出前 20 个使用频率最高的命令「由 ohmyzsh 创建了该命令」

4、take：功能类似于 mkcd 命令「可以完全替代 mkcd 命令」，不过该命令可以识别远程 URL 信息「`something that ends in .git or .tar.(gz|bz2|xz)`」，当给定的参数看起来像是一个 URL 的时候，该命令会下载当前资源并提取到当前目录中，然后还会自动切入到提取目录或者下载目录或者克隆目录「由 ohmyzsh 创建了该命令」使用举例如下所示：

```bash
# 该使用方式并不太常用，使用时将其认为和mkcd一样功能即可
take https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.0-M26/bin/apache-tomcat-11.0.0-M26.tar.gz
```

通过下面操作可以看出会自动下载 Tomcat 文件并解压提取，最后还自动切换到了 conf 目录中。

```bash
 ~/Downloads  take https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.0-M26/bin/apache-tomcat-11.0.0-M26.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12.9M  100 12.9M    0     0  7450k      0  0:00:01  0:00:01 --:--:-- 7449k
 ~/Downloads/apache-tomcat-11.0.0-M26/conf 
```



### 7、Oh-My-Zsh 插件安装详解

#### 1、oh-my-zsh 内置自动补全

- **自动列出目录：**输入 cd 按 tab 键，目录将自动列出，在按 tab 可以切换。
- **自动补全目录：**例如 /opt/local/bin 这个长路径，只需要 cd /o/l/b 按 tab 键自动补全



#### 2、zsh-completions 额外补全插件

> 两种方法都行，二选一

```bash
## 下载安装
git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions

## 添加目录
fpath+=${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions/src

## 激活
source "$ZSH/oh-my-zsh.sh"

## brew下载安装
brew install zsh-completions
```



#### 3、zsh-autosuggestions 历史补全插件

```bash
## 1.手动源码下载安装
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
## vim ~/.zshrc 进去配置 zsh-autosuggestions
plugins=(
    git
    # other plugins...
    zsh-autosuggestions
)

## 2.Homebrew 安装
brew install zsh-autosuggestions
## 然后在 ~/.zshrc 增加
source $(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

> 这里注意：如果使用 iTerm2 的话，可能是主题的颜色区分度不够，所以需要修改主题适配自动补全。



#### 4、zsh-syntax-highlighting 语法高亮插件

命令错误显示红色，直至正确才为绿色，路径正确会添加下划线

```bash
## 下载安装
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

## vim ~/.zshrc 进去配置zsh-syntax-highlighting
plugins=(
    git
    # other plugins...
    zsh-syntax-highlighting
)
```



#### 5、Incremental completion on zsh 实时补全

这个是感觉最有用的，文件多了或者命令记不住的时候：

```bash
## 创建文件夹
mkdir $ZSH_CUSTOM/plugins/incr

## 下载
curl -fsSL https://mimosa-pudica.net/src/incr-0.2.zsh -o $ZSH_CUSTOM/plugins/incr/incr.zsh

## 配置
echo 'source $ZSH_CUSTOM/plugins/incr/incr.zsh' >> ~/.zshrc

## 激活
source ~/.zshrc
```



#### 6、zsh-history-substring-search 插件

> 可以搜索命令历史的插件，使用`Ctrl+R`快捷键触发，模糊搜索历时使用的命令。

- 下载插件到指定目录，使用如下命令即可；

```bash
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
```

- 然后修改配置文件`.zshrc`，在plugins中添加插件`zsh-history-substring-search`；
- 接下来我们就可以通过`Ctrl+R`快捷键触发，然后进行命令搜索补全了。



#### 7、docker 插件

> 自带插件，可以实现 docker 命令补全和自动提示。

- 作为自带插件无需下载，直接修改配置文件`.zshrc`，在plugins中添加插件`docker`；
- 当我们输入 docker 开头的命令时，使用`Tab`键可以出现提示并自动补全。



#### 8、git 插件

> 自带插件，添加了很多git的快捷命令。

- 直接修改配置文件`.zshrc`，在plugins中添加插件`git`；
- 该插件对于 Git 命令提供了非常多的快捷使用方式，比如下面的常用命令；

| 快捷别名 | 命令           |
| :------- | :------------- |
| g        | git            |
| gcl      | git clone      |
| ga       | git add        |
| gc       | git commit     |
| ggp      | git push       |
| ggl      | git pull       |
| gst      | git status     |
| gb       | git branch     |
| glg      | git log --stat |

- 使用快捷命令还是非常方便的！

  ```bash
  # TAB键后的提示
  ~  gcl
  gcl     gclean  gclf
  ```



#### 9、z 插件

> 自带插件，可以快速跳转到上个 cd 的目录下。

- 直接修改配置文件`.zshrc`，在plugins中添加插件`z`，最终配置效果如下；

```bash
plugins=(
    git
    zsh-syntax-highlighting
    zsh-autosuggestions
    zsh-history-substring-search
    docker
    z
)
```

- 我们先切换到`.oh-my-zsh/custom/plugins`目录下，然后再切换到其他目录下，之后直接使用`z plug`命令就可以切换回去了。





### 8、Oh-My-Zsh 配置完整案例

Oh-My-Zsh 的完整配置 ~/.zshrc 文件：

```shell
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="elessar" # set by `omz`

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Uncomment the following line to use case-sensitive completion.
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment one of the following lines to change the auto-update behavior
# zstyle ':omz:update' mode disabled  # disable automatic updates
# zstyle ':omz:update' mode auto      # update automatically without asking
# zstyle ':omz:update' mode reminder  # just remind me to update when it's time

# Uncomment the following line to change how often to auto-update (in days).
# zstyle ':omz:update' frequency 13

# Uncomment the following line if pasting URLs and other text is messed up.
# DISABLE_MAGIC_FUNCTIONS="true"

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# You can also set it to another string to have that shown instead of the default red dots.
# e.g. COMPLETION_WAITING_DOTS="%F{yellow}waiting...%f"
# Caution: this setting can cause issues with multiline prompts in zsh < 5.7.1 (see #5765)
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git aliases tmux z zsh-syntax-highlighting web-search zsh-autosuggestions)

source $ZSH/oh-my-zsh.sh

# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases


# 在Terminal中指定用Sublime打开文件或者目录
alias sublime='open -a /Applications/Sublime\ Text.app'
# 在Terminal中指定用Sublime打开ohmyzsh安装目录
alias ohmyzsh="sublime ~/.oh-my-zsh"
# 在Terminal中指定用Sublime打开zshrc配置文件
alias zshconfig="sublime ~/.zshrc"
# 清空Terminal显示的内容
alias c="clear"
# 修改zprofile文件重新加载生效
alias szprofile="source ~/.zprofile"
# 修改zshrc文件重新加载生效
alias szshrc="source ~/.zshrc"
# 删除文件到废纸篓
alias delete="trash -F"
# git工具修改为英文显示
alias git='LANG=en_GB git'
```





### 9、Oh-My-Zsh 安装最佳实践

#### 1、安装 ohmyzsh

```bash
lsx@MacBook-Air ~ % sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
Cloning Oh My Zsh...
remote: Enumerating objects: 1407, done.
remote: Counting objects: 100% (1407/1407), done.
remote: Compressing objects: 100% (1347/1347), done.
remote: Total 1407 (delta 36), reused 1129 (delta 32), pack-reused 0 (from 0)
Receiving objects: 100% (1407/1407), 3.26 MiB | 11.87 MiB/s, done.
Resolving deltas: 100% (36/36), done.
From https://github.com/ohmyzsh/ohmyzsh
 * [new branch]      master     -> origin/master
Branch 'master' set up to track remote branch 'master' from 'origin'.
Already on 'master'
/Users/lsx

Looking for an existing zsh config...
Found /Users/lsx/.zshrc. Backing up to /Users/lsx/.zshrc.pre-oh-my-zsh
Using the Oh My Zsh template file and adding it to /Users/lsx/.zshrc.

         __                                     __   
  ____  / /_     ____ ___  __  __   ____  _____/ /_  
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \ 
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / / 
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/  
                        /____/                       ....is now installed!


Before you scream Oh My Zsh! look over the `.zshrc` file to select plugins, themes, and options.

• Follow us on X: https://x.com/ohmyzsh
• Join our Discord community: https://discord.gg/ohmyzsh
• Get stickers, t-shirts, coffee mugs and more: https://shop.planetargon.com/collections/oh-my-zsh

compinit:527: no such file or directory: /usr/local/share/zsh/site-functions/_brew
➜  ~ 
```

#### 2、安装 zsh-autosuggestions 插件

```bash
➜  ~ git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-autosuggestions
Cloning into '/Users/lsx/.oh-my-zsh/custom/plugins/zsh-autosuggestions'...
remote: Enumerating objects: 73, done.
remote: Counting objects: 100% (73/73), done.
remote: Compressing objects: 100% (63/63), done.
remote: Total 73 (delta 9), reused 38 (delta 3), pack-reused 0 (from 0)
Unpacking objects: 100% (73/73), done.
```

#### 3、安装 zsh-syntax-highlighting 插件

```bash
➜  ~ git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
Cloning into '/Users/lsx/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting'...
remote: Enumerating objects: 369, done.
remote: Counting objects: 100% (369/369), done.
remote: Compressing objects: 100% (301/301), done.
remote: Total 369 (delta 299), reused 107 (delta 54), pack-reused 0 (from 0)
Receiving objects: 100% (369/369), 182.24 KiB | 2.94 MiB/s, done.
Resolving deltas: 100% (299/299), done.
```

#### 4、配置安装的插件与主题

```bash
➜  ~ vim ~/.zshrc

# 修改主题, 修改引号内内容即可, 默认为 robbyrussell
ZSH_THEME="agnoster"
# 修改plugins标签, 增加插件名称
plugins=(git zsh-syntax-highlighting zsh-autosuggestions)
```

#### 5、设置命令与路径提示符

隐藏用户名和主机名，显示路径的最后一级「修改agnoster主题后，用户名和主机名全路径都会显示出来，这里可以设置隐藏」

```bash
# 隐藏用户名和主机名,如果当前用户和默认用户相同，则不显示用户名和主机名
export DEFAULT_USER=$USER
# 显示路径的最后一级
prompt_dir() {
  prompt_segment blue black '%1~'
}
```

#### 6、重写加载配置让其生效

修改完成后需要重写加载配置才能生效

```bash
# 如下2中方式任选其一
➜  ~ source ~/.zshrc
➜  ~ omz reload
```

#### 7、验证修改后终端的效果

```bash
 ~  cd .oh-my-zsh/custom/plugins
 ~/.oh-my-zsh/custom/plugins   master  pwd
/Users/lsx/.oh-my-zsh/custom/plugins
 ~/.oh-my-zsh/custom/plugins   master 
```



# MacOS 彻底删除应用程序与数据及其配置等

> macOS系统下彻底删除应用程序和其数据、配置等：https://blog.csdn.net/zhiyuan411/article/details/136073065

## 1、使用专用软件卸载应用

例如使用 appCleaner 来卸载应用程序，则会自动清理相关数据和配置等。

如果已经手动删除或卸载掉应用程序，则需要遵循以下步骤来进一步删除数据、配置等相关文件。



## 2、删除应用程序本体

打开 Launchpad，找到要删除的应用程序，长按（或右键点击）它，选择“移到废纸篓”。

或者，直接在“应用程序”目录下删除对应的应用或者应用的文件夹。如果应用自带卸载程序，则推荐执行卸载程序来删除应用。



## 3、清除应用数据文件和支持文件

应用的数据文件和支持文件都存储在：

```bash
/Users/用户名/Library/Application Support
~/Library/Application Support
```

进入该目录，找到对应应用程序的文件夹并删除。



## 4、清除容器目录下的内容

有些现代应用程序使用沙盒机制，其数据可能存储在：

```bash
/Users/用户名/Library/Containers
~/Library/Containers
```

进入该目录，找到对应应用程序的容器文件夹并删除。



## 5、移除偏好设置

应用程序的偏好设置保存在以下文件夹：

```bash
/Users/用户名/Library/Preferences
~/Library/Preferences
```

进入该目录，找到以该应用命名的plist文件并删除。



## 6、清理插件和扩展

如果应用程序安装了插件或者扩展，检查如下目录：

```bash
/Library/Extensions
/Users/用户名/Library/Extensions
~/Library/Extensions
```

进入该目录，找到以该应用命名的相关文件并删除。



## 7、清除漏网之鱼

打开终端，执行以下命令：

```bash
sudo find / -iname "*应用程序名*" 2>/dev/null
```

在找到的结果中，删除不需要的应用程序相关文件。



## 8、删除卸载程序的残留文件总结

鼠标左键点击一下电脑桌面，用组合键“Command+Shift+G”调出“前往文件夹”，复制粘贴下列指令代码，点击进入相应文件夹即可清除该类型程序文件。

1. 程序支持文件夹：“~/Library/Application Support/(应用程序)”
2. 偏好设置文件夹：“~/Library/Preferences/(应用程序)”
3. 程序缓存文件夹：“~/Library/Caches/(应用程序)”
4. 程序插件文件夹：“~/Library/Extensions/(应用程序)”、“/Library/Extensions/(应用程序)”

