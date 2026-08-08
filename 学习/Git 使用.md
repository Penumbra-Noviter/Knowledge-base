---
type: study
tags: [git]
date: 2026-08-06
source: 外部整理（AI 生成，未验证）
status: unverified
---

进入项目目录  cd "D:\Desktop\Craft\Profit Calculator"

配置 Git 身份

  git config --global user.email "你的GitHub邮箱"
  
  git config --global user.name "Penumbra-Noviter"
  
初始化仓库      git init

添加所有文件到暂存区   git add .

创建初始提交   git commit -m "初始提交：收益计算器 PySide6 版本"

在 GitHub 上创建仓库  
创建完成后，GitHub 会显示一个仓库地址，格式类似：“https://github.com/你的用户名/profit-calculator.git”

关联远程仓库并推送   git remote add origin https://github.com/你的用户名/profit-calculator.git

推送代码   git push -u origin main

 ![[Pasted image 20260729031517.png]]

