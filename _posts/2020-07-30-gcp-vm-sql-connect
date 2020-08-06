---
layout: post
title: "GCP VM인스턴스와 SQL 연결하기"
author: "한결"
categories: blog
---
  
  
클라우드 sql db서버를 생성하여 웹서버 인스턴스와 연결해준다.
  
웹서버는 이미 구축되어 있고 sql만 연결하면 됨.
  
​

@db (cloud sql)  instance_name:dbtest, root:dkagh2.

$ gcloud sql connect dbtest --use r=root --quiet 

로그인 후

Create database dbtest;

데이터베이스를 생성해준다. 

create user dbadmin@’%’ identified by ‘dkagh1.^^’;

유저와 암호를 생성하고

grant all on dbtest.* to dbadmin@’%’;

모든 권한을 부여해준다.

flush privileges;

show grants for dbadmin@’%’;

맞게 부여됐는지 확인한다.

​

​

SQL - 연결 탭으로 들어가 네트워크 추가하여 웹서버 네트워크를 추가해준다.

접근 허용 네트워크 추가 (0.0.0.0/0 -> 특정서버IP)

​

​

@web (VM인스턴스)

sudo yum install mysql

mysql -u dbadmin -p -h [db_external_ip]34.68.60.207

db익스터널 아이피는 sql의 외부 아이피를 입력해준다. sql 메인에 나와있음

exit

​

​

​

 proxy 사용하는 경우 : cloud sql API -library

wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64

​

프록시 파일을 다운받은 후 

​

실행 권한 주기위해 

IAM 서비스 계정 및 JSON 키 생성해준다.

IAM 서비스 계정 탭에서 서비스 계정 키 생성하기로 키를 생성한다.

​

프록시 연결을 위한 방법은 두가지가 있다.

하나는 tcp 3306 포트 이용하는 것이고 하나는 Unix socket을 이용하는 것이다.

먼저 tcp 3306 포트 이용하는 방식은 

​

$ ./cloud_sql_proxy.linux.amd64 -instances=[db연결이름]=tcp:3306  -credential_file=[/home/gmailaccount/key json파일] & 

​

db연결이름은 sql 메인의 연결이름

/home/내구글아이디/key.json 파일을 생성하여준다.

다운받은 json 파일을 vi /home/내구글아이디/key.json을 생성하여 복사 붙여넣기하여 json 파일을 생성해줌

&를 넣는 이유 백그라운드 동작을 시키기 위해서 [백그라운드 포그라운드 개념 복습하기]

​

$ ps -ef | grep cloud_sql 

동작확인하는 것을 확인해준다.

$ mysql -u root -p --host 127.0.0.1 

인스턴스에서 내 자신 호스트아이피를 이용해 접속이 가능한지 확인해본다.

​

​

Unix socket이용

    $ mkdir /cloudsql

$ chmod 777 /cloudsql

$ ./cloud_sql_proxy.linux.amd64 -dir=/cloudsql -instances=[db연결이름] -credential_file=[key json파일]

$ ps -ef | grep cloud sql

$ mysql -u root -p -S /cloudsql/[소켓명]

​

​

​

​

​

​

프락시를 서비스 유닛으로 생성해서 관리하기  : cloudsqlproxy.service

​

$ sudo vim ~/cloudsqlproxy.service

//-----------------------------cloudsqlproxy.service  (unix socket)

[Unit]

Description=Google Cloud SQL Proxy

After=syslog.target network.target auditd.service

​

[Service]

ExecStart=/opt/cloudsqlproxy/cloud_sql_proxy -dir=/cloudsql -instances=[DB인스턴스연결이름] -credential_file=[key json파일]

ExecStop=/bin/kill -TERM $MAINPID

​

[Install]

WantedBy=multi-user.target

-----------------------------cloudsqlproxy.service//

​

//-----------------------------cloudsqlproxy.service  (tcp3306)

[Unit]

Description=Google Cloud SQL Proxy

After=syslog.target network.target auditd.service

​

[Service]

ExecStart=/opt/cloudsqlproxy/cloud_sql_proxy -instances=[DB인스턴스연결이름]=tcp:3306 -credential_file=[key json파일]

ExecStop=/bin/kill -TERM $MAINPID

​

[Install]

WantedBy=multi-user.target

-----------------------------cloudsqlproxy.service//

​

​

​

$ sudo cp ./cloudsqlproxy.service /etc/systemd/system/

$ sudo systemctl daemon-reload

$ sudo systemctl start cloudsqlproxy

$ sudo systemctl status cloudsqlproxy  (verify error)

​
