---
layout: single
title: "[Git] 브랜치 관련 명령어 정리"
categories: Git
tags:
  - Git
toc: true
toc_sticky: true
author_profile: false
date: 2023-11-20
last_modified_at: 2023-11-20
sidebar:
  nav: docs
---
  
# Git 브랜치 명령어 정리
### 로컬 브랜치 목록 조회

~~~
git branch
~~~
### 원격 브랜치 목록 조회
~~~powershell
git branch -r
git branch -a
~~~
### 로컬 브랜치 삭제
~~~
git branch -d {브랜치 이름}
~~~
### 원격 브랜치 삭제
~~~powershell
git push origin --delete {원격 브랜치 이름}
~~~

