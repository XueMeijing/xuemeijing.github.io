---
title: github时光机：修改提交历史到1970年
date: 2023-04-09 16:19:03
categories: 
- 有趣
---

在我还没有毕业的时候，我就听说过这样一个关于程序员的段子，某公司想招一个30岁以下，拥有20年工作经验的人。从现在起这不是段子了，只需简单几步，你可以修改你的github提交记录到1970年，成为一个20岁就拥有50年开发经验的老鸟。

## 效果展示

可以打开 https://github.com/XueMeijing 在线体验 (PC端)

![image](https://user-images.githubusercontent.com/35559153/230755702-f35bd2ff-5411-4463-a7cc-b2e711a4aae9.png)

## 修改步骤
注意：不确定是否能回滚到正常状态

1. 创建一个新的名叫1969的仓库 https://github.com/new
![image](https://user-images.githubusercontent.com/35559153/230755895-00b7138c-d2e3-4c7c-83fa-6cd570328415.png)

2. 本地创建一个 index.sh 文件，复制 https://github.com/XueMeijing/1969-script/blob/master/index.sh 内容到本地 index.sh

3. 执行shell
    ```bash
    sh index.sh
    ```

4. 输入 2 选择 ssh 模式（如果你本地已经配置了ssh登录）

5. 复制创建的 1969 仓库的 git 链接 然后回车进行下一步
![image](https://user-images.githubusercontent.com/35559153/230756155-9fa663e0-f9cf-4e8c-9dce-6c3376bd11d5.png)
![image](https://user-images.githubusercontent.com/35559153/230756186-0bbb9b3f-3eed-41e9-a8af-e430243efff2.png)

6. 输入文字或者直接回车进行下一步，输入的文字会出现在 README.md 里

7. 刷新仓库页面，看是否提交成功，如果报错可以翻到下面的常见问题看是否能解决
![image](https://user-images.githubusercontent.com/35559153/230756281-2b84532d-b89c-47ee-b643-96197a6deeb4.png)

8. 回到个人资料页，刷新页面看提交历史的年份是否改变

## 常见问题

1. 执行 shell 失败，报错 badDateOverflow: invalid author/committer line - date causes integer overflow
  - index.sh 设置的年份要大于等于1970年
  - 等于1970年的时候，跟所在时区有关，比如北京时区为+0800，设置 index.sh 的 GIT_AUTHOR_DATE 和 GIT_COMMITTER_DATE 为 "${YEAR}-01-01T08:00:00"

2. 提交成功，但是个人资料页时间轴没改变
  - 我的问题是本地 git 配置的邮箱，和 github 的邮箱不是同一个，需要在 github 设置-> 邮箱中加上本地的邮箱
  - 可能没到 24 小时
  - 更多信息请看 [Common reasons that contributions are not counted](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-profile/managing-contribution-settings-on-your-profile/why-are-my-contributions-not-showing-up-on-my-profile)

3. 时间轴改变到1969年，但是上下滚动的时候有错位异常
  - 选择 Select Activity overview 模式，滚动就正常了
    ![image](https://user-images.githubusercontent.com/35559153/230754120-e5687f95-b3f7-48ca-aa2b-94601e2dac31.png)

## 原理

git 的每次提交有两个时间，分别为 AuthorDate 和 CommitDate ，在一些情况下他们是不一致的，比如使用 git cherry-pick 或者 git commit --amend 后, 再使用 git log --pretty=fuller 可以查看每次提交 AuthorDate 和 CommitDate 的区别。

![image](https://user-images.githubusercontent.com/35559153/230784864-b6db0d2d-74f3-43ee-8275-725a2ce7d16b.png)

你可以每次提交手动设置提交时间，如
```bash
git commit -m 'add something' --date 1970-01-01T08:00:00
```
通过修改提交时间还可以进行其他有趣的玩法，比如下面两个
### 生成随机贡献 https://github.com/Shpota/github-activity-generator
![image](https://user-images.githubusercontent.com/35559153/230758005-29ee8376-ea3c-4d75-b73b-ec419ab3946a.png)

### 自己设置未来炫酷的贡献 https://github.com/empdo/art
![image](https://user-images.githubusercontent.com/35559153/230758057-c0d8e862-1bca-4503-bee3-61f63fe69776.png)



## 结束

俺有个习惯，划水没事的时候就喜欢点开别人的 github 资料页看，看到这位大神 [antfu](https://github.com/antfu) 的时候惊了，她的贡献记录竟然到 1990 年，俺就搜了下发现她写的库 https://github.com/antfu/1990-script , 但是 issues 里和 stars 只有极少数人修改成功了，最近修改成功的一位在今年 1 月。有个哥们还提了一个pr，这哥们更牛逼直接冲到了1969年😅 所以其他人失败可能并不是因为 github 修复了这个问题，经过查文档和调试后发现，我的问题是 github 邮箱里没有保存本地 git 设置的邮箱，添加后就正常了。如果你按照教程修改成功了，欢迎点个star~ https://github.com/XueMeijing/1969-script
