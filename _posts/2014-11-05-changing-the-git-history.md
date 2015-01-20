---
layout: post
title:  "Git修改提交历史的author和emial"
date:   2014-11-05
categories: jekyll update
---

修改上一次提交的用户名和email可以用以下命令：

	git commit --amend --author="Author Name <email@address.com>"

修改整个项目的name和email可以参考[github](https://help.github.com/articles/changing-author-info/)

大体过程如下：

1. 打开终端，克隆裸仓库

	```
	git clone --bare https://github.com/user/repo.git && cd repo.git
	```

2. 复制粘贴下面的脚本，修改`OLD_EMAIL`, `CORRECT_NAME`, `CORRECT_EMAIL`

	{% gist 90f1d1da584cfee2355d %}

3. 运行脚本

4. 推送到远端

	```
	git push --force --tags origin 'refs/heads/*'
	```

5. 删除文件

	```
	cd .. && rm -rf repo.git
	```