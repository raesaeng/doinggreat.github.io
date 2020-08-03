---
layout: post
title: "GCP로 wordpress 설치하기 (공인 ip로 연결)"
author: "한결"
categories: blog
---


GCP로 wordpress 설치를 해볼 것이다.
VM 인스턴스에 웹서버를 구축해준 뒤에 sql과 연동시켜 DB로 사용할 것이다.   

Compute Engine 탭에서 VM 인스턴스를 하나 생성했다.   
ssh 접속하여 필요한 패키지를 설치해준다.

    yum -y install update
    yum -y install httpd
    yum install epel-release
    rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
    yum install mod_php72w php72w-cli
    yum install php72w-bcmath php72w-gd php72w-mbstring php72w-mysqlnd php72w-pear php72w-xml php72w-xmlrpc php72w-process

먼저 업데이트를 진행해주고 Apache와 php7 버전을 설치해준다.
php를 yum으로 받으면 낮은버전이라 지정하여 설치해준다. 설치 전 epel 저장소가 추가되어야 한다.
추가로 webtaric 저장소를 설치해 주어야 한다.

mod_php72w : Apache HTTP 서버와 연동을 위한 모듈
php-bcmath : bcmath 라이브러리
php-gd : gd 그래픽 라이브러리
php-mbstring : multi-byte 문자열 처리(한글과 같은 2byte 문자열 처리)
php-mysql : MySQL 데이터베이스 지원
php-pear : php 확장 라이브러리

    php -v
php가 버전이 맞게 설치되었는지 확인해준다.

    systemctl start httpd
    systemctl enable httpd
    systemctl status httpd
    
실행시켜준다.

    <?php phpinfo(); ?>
아파치랑 연동이 되었는지 확인해본다.


연동이 되었으면 wordpress를 다운받아준다.
https://wordpress.org/download/

wget로 다운받을 것이기 때문에 wget가 설치되어 있는지 확인해준다.
설치가 안되어 있다면
    yum -y install wget
    wget https://wordpress.org/latest.tar.gz

    ls
ls 명령어로 다운이 되었는지 확인해준다.

    tar -xvzf latest.tar.gz -C /var/www/html
    chown -R apache: /var/www/html/wordpress

다운로드 받은 워드프레스 파일을 tar 명령어로 풀어준다. /var/www/html 위치 지정해준다.
wordpress 폴더에 권한을 주어야 한다. Apache를 다운받으며 자동으로 Apache 사용자가 생성되어 지정해줄 수 있다.

    cd /var/www/html; ll; cd -
권한이 부여됐는지 확인해준다.

권한이 부여 됐으면 가상 서버를 지정해주어야 한다.
    vim /etc/httpd/conf/httpd.conf
으로 들어가 가상 서버를 지정해준다.
임의로 serveradmin , servername, severalias 를 지정해준다.

    <VirtualHost *:80>
	ServerAdmin admin@wp.com
	DocumentRoot /var/www/html/wordpress
	ServerName wp.com
	ServerAlias www.wp.com
	ErrorLog /var/log/httpd/tecminttest-error-log
	CustomLog /var/log/httpd/tecminttest-acces-log common
    </VirtualHost>



sql과 연결하기 위하여 mariadb 를 설치해 주어야 한다.

    yum -y install mariadb mariadb-server

![Screenshot from 2020-08-03 14-33-39](https://user-images.githubusercontent.com/69098825/89162306-d99a1100-d5ae-11ea-8a52-7e7087751e81.png)

또 selinux를 꺼줘야 접근이 가능하므로 selinux도 꺼준다.
    setenforce 0 


gcp의 sql 탭으로 들어가 인스턴스를 생성한다.

인스턴스를 생성한 뒤 로그인

    create database 만들데이터베이스이름;
    show databases;

데이터 베이스를 생성해주고 잘 만들어 졌는지 확인해준다.

유저도 생성해준다.

    create user user@'%' identified by 'password';
유저를 생성했으면 권한도 부여해준다.
'%'

    grant all on database.* to user@'%'
    flush privileges;

권한을 부여해주고 적용시켜준다.


그러고 나서 웹서버의 아이피 주소로 접속한다.
그러면 워드프레스 설치 마법사가 뜰 것이다.



