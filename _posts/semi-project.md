### 1. VM 인스턴스 생성

wordpress 설치를 위해 apache와 php, mariadb 를 설치해준다. 

![01_instance](/Users/gyeol/Downloads/세미프로젝트캡쳐/01_instance.PNG)

![02_instance](/Users/gyeol/Downloads/세미프로젝트캡쳐/02_instance.PNG)



* Apache, MariaDB 설치

```
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
yum install mariadb mariadb-server
```



* PHP 7 설치

```
yum install epel-release
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum install mod_php72w php72w-cli
yum install php72w-bcmath php72w-gd php72w-mbstring php72w-mysqlnd php72w-pear php72w-xml php72w-xmlrpc php72w-process
php -v
```

>mod_php72w : Apache HTTP 서버와 연동을 위한 모듈
>php-bcmath : bcmath 라이브러리
>php-gd : gd 그래픽 라이브러리
>php-mbstring : multi-byte 문자열 처리(한글과 같은 2byte 문자열 처리)
>php-mysql : MySQL 데이터베이스 지원
>php-pear : php 확장 라이브러리

![89010160-03ea9500-d349-11ea-8ded-bd23b30204e3](/Users/gyeol/Desktop/89010160-03ea9500-d349-11ea-8ded-bd23b30204e3.png)

​	PHP가 잘 설치되었는지 php -v 명령어로 확인해준다.





### 2. SQL 인스턴스 생성하기

![03_db](/Users/gyeol/Downloads/세미프로젝트캡쳐/03_db.PNG)



   * 데이터 베이스 만들기

  워드프레스에서 사용할 데이터베이스를 만들어준다. db_0804로 만들어 주었다.

![04_db](/Users/gyeol/Downloads/세미프로젝트캡쳐/04_db.PNG)





### 3. Wordpress 설치하기

   

   워드프레스 홈페이지에서 다운로드 받아 풀어준다. 권한 설정 후 지장한 경로에 위치하고 있는지 확인해준다.

   

   ```
   wget "http://wordpress.org/latest.tar.gz"
   tar -xvzf latest.tar.gz -C /var/www/html
   chown -R apache: /var/www/html/wordpress
   ```



![05_db](/Users/gyeol/Desktop/89010532-bb7fa700-d349-11ea-8fbb-afd178b5fb57.png)

```
vim /etc/httpd/conf/httpd.conf
```

​	httpd.conf 으로 편집 모드로 들어가 가상서버 설정을 진행해준다.

```
<VirtualHost *:80>
	ServerAdmin admin@wp.com
	DocumentRoot /var/www/html/wordpress
	ServerName wp.com
	ServerAlias www.wp.com
	ErrorLog /var/log/httpd/tecminttest-error-log
	CustomLog /var/log/httpd/tecminttest-acces-log common
</VirtualHost>
```

​	임의의 Server 이름과 주소를 지정해준다.



### 4. 프록시 연결하기

   

   프록시를 연결하기 위해 SELinux를 정지 시켜준다.

```
setenforce 0
```

​	구글 프록시를 다운받은 뒤 실행권한을 부여해준다. 만들어둔 SQL 인스턴스와 연동시켜준다. 

​	프록시를 실행하며 mysql 포트인 3306 포트를 지정해준다.

```
wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy && chmod +x cloud_sql_proxy
export SQL_CONNECTION=[SQL 인스턴스 연결 이름]
./cloud_sql_proxy -instances=$SQL_CONNECTION=tcp:3306 &
```

​	

![89010831-4e204600-d34a-11ea-8569-da9da9c0a4b5](/Users/gyeol/Desktop/89010831-4e204600-d34a-11ea-8569-da9da9c0a4b5.png)

​	Ready for new connections 커넥션 준비가 완료되었다.



### 5. Wordpress 설치하기

   

   워드프레스 설치를 진행해준다. 만들어둔 데이터 베이스 정보를 기입한다. 

   프록시 연결했기 때문에 호스트는 127.0.0.1로 연결해준다.

   ![89011199-ed453d80-d34a-11ea-922d-1b784b90e429](/Users/gyeol/Desktop/89011199-ed453d80-d34a-11ea-922d-1b784b90e429.png)

   

![06_wp](/Users/gyeol/Downloads/세미프로젝트캡쳐/06_wp.PNG)

나머지 정보를 입력해주고 나면 워드프레스 사용이 가능하다.

​	

### 6. 인스턴스 템플릿 생성하기

   인스턴스 템플릿 생성을 위해 스냅샷 생성 후 이미지 생성하여 부팅 디스크를 만들어준다.

![07_snap](/Users/gyeol/Downloads/세미프로젝트캡쳐/07_snap.PNG)

   * VM 인스턴스 스냅샷 생성 후 스냅샷을 소스로 이미지 생성

![08_image](/Users/gyeol/Downloads/세미프로젝트캡쳐/08_image.PNG)

   * 이미지를 베이스로 부팅디스크를 생성

![09_temp](/Users/gyeol/Downloads/세미프로젝트캡쳐/09_temp.PNG)

   * 인스턴스 템플릿 생성

![10_temp](/Users/gyeol/Downloads/세미프로젝트캡쳐/10_temp.PNG)



### 7. 인스턴스 그룹 생성.

   

   부하분산을 확인하기 위해 리전별로 인스턴스 그룹을 생성해준다. 

   asia, us, eu 각각 인스턴스 그룹을 생성해 주었다. 

   측정항목을 설정해준다.

   

![11_temp](/Users/gyeol/Downloads/세미프로젝트캡쳐/11_temp.PNG)

![12_group](/Users/gyeol/Downloads/세미프로젝트캡쳐/12_group.PNG)



### 8. ip 고정 주소 예약

   

   VPC 네트워크 탭에서 고정 주소를 예약해준다. 모든 리전이 사용할 ip이기 때문에 전역으로 설정한다.

   

![13_staticip](/Users/gyeol/Downloads/세미프로젝트캡쳐/13_staticip.PNG)



### 9. 부하분산기 생성하기.

   

​	백엔드, 호스트, 프론트엔드 서비스를 생성하여 부하분산기를 만들어준다.

   * 백엔드 서비스 생성

  모든 그룹을 다 추가해주고 Cloud CDN 사용 설정을 체크해준다.

![14_healthcheck](/Users/gyeol/Downloads/세미프로젝트캡쳐/14_healthcheck.PNG)



   * 프런트 엔드 서비스 생성

  ip 주소에 예약해두었던 ip 고정 주소를 넣어준다.

![15_front](/Users/gyeol/Downloads/세미프로젝트캡쳐/15_front.PNG)

​	설정 검토 후 완료되었으면 부하분산기를 생성해준다.



### 9. rsync 키 생성하기

   

   인스턴스를 편리하게 관리하기 위하여 rsync를 적용시켜준다.

   rsync를 사용함으로써 변경된 데이터가 다른 리전에도 바로 적용시켜 관리할 수 있다.

   다음 명령어로 키 생성한다.

```
ssh-keygen -t rsa
```



![16_key](/Users/gyeol/Downloads/세미프로젝트캡쳐/16_key.PNG)



### 10. 스토리지 버킷 생성하기.

​    

​	스토리지 버킷을 생성 후 앞서 생성해둔 ssh 키를 스토리지 버킷에 넣어주어 다른 인스턴스로의 접속이 가능하게 할 것이다. 

​	스토리지 탭에서 버킷을 생성해준다.

​	gsutil 명령어를 사용해 ssh 키를 버킷으로 복사해준다.    



    // EU 34.89.13.2
    gsutil cp ./id_rsa.pub gs://ssh_bucket_tmdgh0701



![17_key](/Users/gyeol/Downloads/발표/17_ssh.PNG)


​    

​	버킷안에 저장된 key 를 볼 수 있다.


​    

![스크린샷 2020-08-04 오후 2.05.55](/Users/gyeol/Desktop/스크린샷 2020-08-04 오후 2.05.55.png)


​    

​	버킷으로 옮겨진 ssh 키를 다른 리전에 위치한 인스턴스로 가져온다.
​ 

```
// Asia 34.84.111.196
gsutil cp gs://ssh_bucket_tmdgh0701/id_rsa.pub ./
cat id_rsa.pub >> authrized_keys
```



​	ssh 키를 공유함으로서 rsync 가 적용되는 것을 볼 수 있다.

​	워드프레스에서 수정, 변경한 데이터가 자동으로 전송된다.


​    

![18_rsync](/Users/gyeol/Downloads/발표/18_rsync.PNG)



​	crontab 명령어를 사용하여 주기적으로 저장할 수 있도록 설정해준다.
​ 
​   

```
rsync -alvzrt --delete -e tmdgh0701@34.84.111.196:/var/www/html/wordpress/ /var/www/html/wordpress/
```



![19_crontab2](/Users/gyeol/Desktop/엽서/19_crontab2.jpg)



### 11. 부하분산 테스트

​	서버 부하 테스트 프로그램을 이용하여 구축한 워드프레스 서버에 트래픽을 줄 것이다.

​	이후 로드 밸런싱이 진행되는지 GCP 내의 그래프로 확인해 볼 것이다.



​	사용할 프로그램은 Apache JMeter 으로 HTTP Request의 Number of Threads (users)를 1000 입력해주었다.



![21_jmeter](/Users/gyeol/Downloads/21_jmeter.PNG)



​	로그 기록 탭에서 cloud HTTP 부하 분산기에서 확인하면 로그가 이렇게 보여진다. 

![20_log](/Users/gyeol/Downloads/20_log.PNG)



* Apache JMeter에서의 그래프

![21_jmeter_graph](/Users/gyeol/Downloads/21_jmeter_graph.PNG)



* GCP 그래프 확인 

  급격하게 치솟다가 떨어지는 그래프를 볼 수 있다. 

![22_chart](/Users/gyeol/Downloads/22_chart.PNG)

![22_graph](/Users/gyeol/Downloads/22_graph.PNG)



* 부하 분산기로 인한 인스턴트 생성 감소 확인

  리전별로 각 1개였던 인스턴스 수량이 변경되고 있는 것을 확인할 수 있다. eu는 증가하여 2개의 인스턴스가 존재하고 us는 트래픽이 줄어들어 2개였던 인스턴스가 1개로 감소되고 있다.

![instance](/Users/gyeol/Desktop/instance.png)



* 로그 기록 탭에서 로그뷰어 GCE 자동 크기 조절기를 선택하면 로그를 볼 수 있다.

  인스턴스 그룹 istance-group-us 가 newSize: 2 oldSize: 1 인 것을 확인할 수 있다. 

![image-20200804155309233](/Users/gyeol/Library/Application Support/typora-user-images/image-20200804155309233.png)



