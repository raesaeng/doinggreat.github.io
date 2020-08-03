---
layout: post
title: "GCP로 wordpress 설치하기 (공인 ip로 연결)"
author: "한결"
categories: blog
---


GCP로 wordpress 설치를 해볼 것이다.  
VM 인스턴스에 웹서버를 구축해준 뒤에 sql과 연동시켜 DB로 사용할 것이다.  
  
<img width="478" alt="스크린샷 2020-08-03 오후 9 37 24" src="https://user-images.githubusercontent.com/69098825/89183010-92714780-d5d1-11ea-9504-2b898daa05e6.png">  
<img width="471" alt="스크린샷 2020-08-03 오후 9 37 36" src="https://user-images.githubusercontent.com/69098825/89183020-94d3a180-d5d1-11ea-967c-ef48f751a43f.png">  
  
<img width="852" alt="스크린샷 2020-08-03 오후 9 38 40" src="https://user-images.githubusercontent.com/69098825/89183111-bb91d800-d5d1-11ea-9a9c-917b5d17c0d4.png">
  
Compute Engine 탭에서 Centos 7버전의 VM 인스턴스를 하나 생성했다.  
ssh 접속하여 필요한 패키지를 설치해준다.  

    yum -y install update
    yum -y install httpd
    yum -y install epel-release
    rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
    yum -y install mod_php72w php72w-cli
    yum -y install php72w-bcmath php72w-gd php72w-mbstring php72w-mysqlnd php72w-pear php72w-xml php72w-xmlrpc php72w-process

먼저 업데이트를 진행해주고 Apache와 php7 버전을 설치해준다.  
php를 yum으로 받으면 낮은버전이라 지정하여 설치해준다.  
설치 전 epel 저장소가 추가되어야 한다.  
추가로 webtaric 저장소를 설치해 주어야 하므로 rpm 명령어로 설치한다.  
  
> mod_php72w : Apache HTTP 서버와 연동을 위한 모듈  
> php-bcmath : bcmath 라이브러리  
> php-gd : gd 그래픽 라이브러리  
> php-mbstring : multi-byte 문자열 처리(한글과 같은 2byte 문자열 처리)  
> php-mysql : MySQL 데이터베이스 지원  
> php-pear : php 확장 라이브러리  
  
    php -v
  
<img width="501" alt="스크린샷 2020-08-03 오후 9 47 25" src="https://user-images.githubusercontent.com/69098825/89183955-02340200-d5d3-11ea-9fcf-fdf044759cb9.png">
  
php가 버전이 맞게 설치되었는지 확인해준다.  
  
    systemctl start httpd
    systemctl enable httpd
    systemctl status httpd
  
실행시켜준다.  
  
  
  
wordpress를 다운받아준다.  
https://wordpress.org/download/  
  
wget로 다운받을 것이기 때문에 wget가 설치되어 있는지 확인해준다.  
설치가 안되어 있다면  
  
    yum -y install wget
    wget https://wordpress.org/latest.tar.gz
  
<img width="226" alt="스크린샷 2020-08-03 오후 9 53 39" src="https://user-images.githubusercontent.com/69098825/89184479-d5341f00-d5d3-11ea-8743-ebe71cd9f148.png">
  
    ls
  
ls 명령어로 다운이 되었는지 확인해준다.  
  
    tar -xvzf latest.tar.gz -C /var/www/html
    chown -R apache: /var/www/html/wordpress
  
다운로드 받은 워드프레스 파일을 tar 명령어로 풀어준다. /var/www/html 위치 지정해준다.  
wordpress 폴더에 권한을 주어야 한다. Apache를 다운받으며 자동으로 Apache 사용자가 생성되어 지정해줄 수 있다.  
  
    ll /var/www/html/
  
<img width="445" alt="스크린샷 2020-08-03 오후 9 55 39" src="https://user-images.githubusercontent.com/69098825/89184653-17f5f700-d5d4-11ea-86af-3ced57636a30.png">
  
권한이 부여됐는지 확인해준다.  
  
권한이 부여 됐으면 가상 서버를 지정해주어야 한다.  
  
    vim /etc/httpd/conf/httpd.conf  
  
으로 들어가 맨 밑에 가상 서버를 지정해줘야 한다.  
깨알 단축키로 shift+G 를 누르면 맨 밑으로 간다.
임의로 serveradmin , servername, severalias 를 지정해준다.  
  
    <VirtualHost *:80>
	ServerAdmin admin@wp.com
	DocumentRoot /var/www/html/wordpress
	ServerName wp.com
	ServerAlias www.wp.com
	ErrorLog /var/log/httpd/tecminttest-error-log
	CustomLog /var/log/httpd/tecminttest-acces-log common
    </VirtualHost>
  
<img width="546" alt="스크린샷 2020-08-03 오후 10 00 16" src="https://user-images.githubusercontent.com/69098825/89185154-bd10cf80-d5d4-11ea-977f-39c24e044223.png">
  
  
sql과 연결하기 위하여 mariadb 를 설치해 주어야 한다.  
  
    yum -y install mariadb mariadb-server
  
![Screenshot from 2020-08-03 14-33-39](https://user-images.githubusercontent.com/69098825/89162306-d99a1100-d5ae-11ea-8a52-7e7087751e81.png)
  
또 selinux를 꺼줘야 접근이 가능하므로 selinux도 꺼준다.  
  
    setenforce 0
  
  
<img width="482" alt="스크린샷 2020-08-03 오후 10 01 45" src="https://user-images.githubusercontent.com/69098825/89185338-0b25d300-d5d5-11ea-9d8c-b89d5fcc3482.png">
<img width="460" alt="스크린샷 2020-08-03 오후 10 02 18" src="https://user-images.githubusercontent.com/69098825/89185342-0c570000-d5d5-11ea-9016-be95e258e485.png">
  
gcp의 sql 탭으로 들어가 인스턴스를 생성한다.  
  
<img width="548" alt="스크린샷 2020-08-03 오후 10 10 00" src="https://user-images.githubusercontent.com/69098825/89185983-1b8a7d80-d5d6-11ea-818c-1434a0a4f51b.png">
  
인스턴스를 생성한 뒤 cloud shell 으로 로그인한다.  
  
    create database 만들데이터베이스이름;
    show databases;
  
<img width="291" alt="스크린샷 2020-08-03 오후 10 12 44" src="https://user-images.githubusercontent.com/69098825/89186274-81770500-d5d6-11ea-97d2-6b5b492c86de.png">
  
데이터 베이스를 생성해주고 잘 만들어 졌는지 확인해준다.  
  
  
<img width="431" alt="스크린샷 2020-08-03 오후 10 19 11" src="https://user-images.githubusercontent.com/69098825/89186873-622ca780-d5d7-11ea-96b6-9133d2d23a3b.png">
  
유저도 생성해준다.  
  
    create user user@'%' identified by 'password';
  
유저를 생성했으면 권한도 부여해준다.  
'%'는 원격접속이 가능함을 뜻한다.  
  
    grant all on database.* to user@'%'
    flush privileges;
  
권한을 부여해주고 적용시켜준다.  
  
  
그러고 나서 웹서버의 아이피 주소로 접속한다.  
그러면 워드프레스 설치 마법사가 뜰 것이다.  
  


