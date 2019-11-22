---
title: 同步Gitlab项目到Github
layout: post
tags: git
date: 2019-04-23 00:00:00
description: 之前在 Github 中导入了一个 Gitlab 项目，但是在 Gitlab 项目有更新的时候， Github 没有更新，所以尝试一下。
---




## 步骤

1. 在 Gitlab 项目中选择 *Settings -> Reposiroty -> Mirror a repository*

2. 在 *Git repository URL* 中输入 Github 的 url，但是格式有要求的，

   格式要求是：

   *https://username@gitlab.company.com/group/project.git*

   这里的 **group** 其实就是你自己的 **github** 用户名。

   例如，我的 **github repo url** 是：

   *https://github.com/AdoBeatTheWorld/goserver.git*

   在这里我需要输入的是：

   *https://AdoBeatTheWorld@github.com/AdoBeatTheWorld/goserver.git*

3. 选择 *Mirror Direction* 为 **Push**

4. 在*Password*中填入**Github**的登陆密码

5. 点击**Mirror repository**按钮

6. 会在**Mirrored repositories**中生成一条新的记录

   ![mirrored repos](./mirrorrepos.png)

7. 点击垃圾桶旁边的刷新按钮

8. 刷新页面就可以看到时候刷新成功，刷新成功的话*Last update*会从 **never**  变成 **just now**

9. 此时去看对应的 Github 项目就会发现已经更新了

## 参考

[Gitlab repository_mirroring](https://docs.gitlab.com/ee/workflow/repository_mirroring.html)

[Gitlab Support Ticket 55729](https://gitlab.com/gitlab-org/gitlab-ce/issues/55729)

[Gitlab Support Ticket 4060](https://gitlab.com/gitlab-com/support-forum/issues/4060)