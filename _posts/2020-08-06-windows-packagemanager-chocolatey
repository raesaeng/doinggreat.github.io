---
layout: post
title: "윈도우용 패키지 매니저 chocolatey"
author: "한결"
categories: blog
---

## chocolatey(choco) - Windows 용 Package Manager

[Chocolatey ](https://chocolatey.org/) 는 Mac OS X 의 [Homebrew](http://brew.sh/index_ko.html) 나 Linux 의 yum, apt-get 같은 역할을 수행하는 [NuGet](https://www.nuget.org/) 기반의 윈도우용 패키지 관리자이다. 관리자 권한이 꼭 필요하므로 powershell 으로 접근하여 사용해야 한다.

https://chocolatey.org/packages 에서 원하는 패키지를 검색해서 편리하게 복사해 설치할 수 있다.



* cmd 관리자 권한으로 시작하기

```
powershell

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```



<img width="1181" alt="스크린샷 2020-08-06 오후 9 33 42" src="https://user-images.githubusercontent.com/69098825/89532835-64883f00-d82d-11ea-987e-960a5ab48c93.png">



* chocolatey로 google chrome 다운로드 하기

```
choco install googlechrome -y
```



* AWS CLI 다운로드하기

```
choco install awscli -y
```



* AWS CMD 명령어

```
aws configure
aws ec2 describe-instances
```

  
