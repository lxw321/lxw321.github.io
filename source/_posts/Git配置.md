---
title: Git配置&&一台计算机上连接多个Github账户
date: 2024-01-08 19:55:47
tags: ["Git", "GitHub"]
---

### Git 配置文件

Git 有多个配置文件，它们的位置、作用和优先级如下：

1. 全局配置文件：位于用户主目录下的`.gitconfig`文件。它对当前用户的所有仓库生效。

   - 位置：`~/.gitconfig`（Linux 和 macOS）或`C:\Users\<用户名>\.gitconfig`（Windows）
   - 作用：存储全局级别的 Git 配置，如用户名、邮箱等。
   - 优先级：最低优先级，可被后面的配置文件覆盖。

2. 仓库配置文件：位于 Git 仓库的`.git/config`文件。

   - 位置：每个 Git 仓库都有自己的`.git`目录，其中包含`config`文件。
   - 作用：存储当前仓库的配置信息，覆盖全局配置。
   - 优先级：高于全局配置文件，但低于下面提到的本地配置文件。

3. 本地配置文件：位于当前工作目录下的`.git/config`文件。

   - 位置：在任意的 Git 工作目录中，执行`git init`命令后会创建`.git`目录，其中包含`config`文件。
   - 作用：存储当前工作目录的配置信息，覆盖全局和仓库配置。
   - 优先级：最高优先级，覆盖全局和仓库配置文件。

这些配置文件使用 INI 文件格式，可以通过手动编辑它们或使用 Git 命令进行配置。例如，可以使用以下命令设置全局用户名和邮箱：

```
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

要查看当前的 Git 配置，可以使用以下命令：

```
git config --list
```

该命令会显示所有级别（全局、仓库、本地）的配置项及其值。

### 生成 SSH 密钥

生成主账号的 SSH 密钥

```
ssh-keygen -t rsa -C "your_email@example.com"
```

生成另一个账户 work_user1 的 SSH 密钥

```
ssh-keygen -t rsa -C "your_email@example.com" -f "id_rsa_work_user1"
```

### 将新的 SSH 密钥添加到相应的 GitHub 账户中

复制公钥 ` ~/.ssh/id_rsa.pub`，然后登录你的个人 GitHub 账户：

- 转到 `Settings`
- 在左边的菜单中选择 `SSH and GPG keys`
- 点击 `New SSH key`，提供一个合适的标题，并将密钥粘贴在下面的方框中
- 点击 `Add key` - 就完成了！

对于工作账户，使用相应的公钥（` ~/.ssh/id_rsa_work_user1.pub`），在 GitHub 工作账户中重复上述步骤。

### 开启 ssh-agent

在 Windows 中： 打开计算机管理，OpenSSH Authentication Agent 设置为自动启动。

![](C:\Users\ASUS\AppData\Roaming\marktext\images\2024-01-04-23-54-31-e76cc997-2646-45ad-846e-28497c56c2d3.png)

![](C:\Users\ASUS\AppData\Roaming\marktext\images\2024-01-04-23-55-55-3146513a-365b-4414-aa2c-7c9ca69eb700.png)

<img src="file:///C:/Users/ASUS/AppData/Roaming/marktext/images/2024-01-04-23-56-26-416a7b72-1a3f-452e-9609-8b808a66a9d5.png" title="" alt="" width="399">

### 在 ssh-agent 上注册新的 SSH 密钥

为了使用这些密钥，我们必须在我们机器上的 **ssh-agent** 上注册它们。

```bash
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_work_user1
```

### 编写 SSH 配置文件

在这里，我们实际上是为不同的主机添加 SSH 配置规则，说明在哪个域名使用哪个身份文件。SSH 配置文件将在 **~/.ssh/config** 中。如果有的话，请编辑它，否则我们可以直接创建它。在 `~/.ssh/config` 文件中为相关的 GitHub 账号做类似于下面的配置项：

```bash
# Personal account, - the default config
Host github.com
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa

# Work account-1
Host github.com-work_user1
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa_work_user1
```

“work_user1” 是工作账户的 GitHub 用户 ID。<mark>重要</mark>

上面的配置要求 ssh-agent：

- 使用 **id_rsa** 作为任何使用 **@github.com** 的 Git URL 的密钥
- 对任何使用 **@github.com-work_user1** 的 Git URL 使用 **id_rsa_work_user1** 密钥

### 为本地仓库设置 git remote url

一旦我们克隆/创建了本地的 Git 仓库，确保 Git 配置的用户名和电子邮件正是你想要的。GitHub 会根据提交（commit）描述所附的电子邮件 ID 来识别任何提交的作者。

要列出本地 Git 目录中的配置名称和电子邮件，请执行 `git config user.name` 和 `git config user.email`。如果没有找到，可以进行更新。

```bash
git config user.name "User 1"   // Updates git config user name
git config user.email "user1@workMail.com"
```

### 克隆仓库

注意：如果我们在本地已经有了仓库，那么请查看第 7 步。

现在配置已经好了，我们可以继续克隆相应的仓库了。在克隆时，注意我们要使用在 SSH 配置中使用的主机名。

仓库可以使用 Git 提供的 clone 命令来克隆：

```
git clone git@github.com:personal_account_name/repo_name.git
```

工作仓库将需要用这个命令来进行修改：

```bash
git clone git@github.com-work_user1:work_user1/repo_name.git
```

这个变化取决于 SSH 配置中定义的主机名。@ 和 : 之间的字符串应该与我们在 SSH 配置文件中给出的内容相匹配。

### 对于本地存在的版本库

**如果我们有已经克隆的仓库：**

列出该仓库的 Git remote，`git remote -v`

检查该 URL 是否与我们要使用的 GitHub 主机相匹配，否则就更新 remote origin URL。

```bash
git remote set-url origin git@github.com-worker_user1:worker_user1/repo_name.git
```

确保 @ 和 : 之间的字符串与我们在 SSH 配置中给出的主机一致。

**如果你要在本地创建一个新的仓库：**

在项目文件夹中初始化 Git `git init`。

在 GitHub 账户中创建新的仓库，然后将其作为 Git remote 添加给本地仓库。

```bash
git remote add origin git@github.com-work_user1:work_user1/repo_name.git
```

确保 @ 和 : 之间的字符串与我们在 SSH 配置中给出的主机相匹配。

推送初始提交到 GitHub 仓库：

```bash
git add .
git commit -m "Initial commit"
git push -u origin main
```

参考：[如何用 SSH 密钥在一台机器上管理多个 GitHub 账户 (freecodecamp.org)](https://www.freecodecamp.org/chinese/news/manage-multiple-github-accounts-the-ssh-way/)

[ssh agent 详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/126117538)
