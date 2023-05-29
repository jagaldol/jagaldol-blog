---
layout: single
title: "리눅스 서버(AWS 등)의 MySQL을 원격으로 사용하기"
date: 2023-05-30 00:52:00 +0900
# last_modified_at: 2023-05-12 03:57:00 +0900
categories:
    - development
---

이번에는 서버에 MySQL을 설치 및 사용자를 추가하여 다른 컴퓨터에서도 서버의 MySQL에 접근 가능하도록 해봤어요. 제가 AWS 프리티어로 우분투 서버 인스턴스를 대여 중인데 아직 아무 프로그램도 안 올려놔서 데이터베이스라도 서버에서 돌릴려고요.🐎

## 설치

먼저 ssh로 서버에 로그인을 합니다.

```bat
> ssh -i "aws-password.pen" ubuntu@[ip 주소]
```

MySQL부터 깔아야겠죠?

```sh
$ sudo apt-get update
$ sudo apt-get install mysql-server
```

## MySQL 비밀번호 변경

설치는 했는데 아무리 `mysql` 명령어를 넣어도 진입이 안되는 거에요. 알고보니까 aws가 기본 유저로 ubuntu가 되어있는데, mysql에 유저를 root로 해야하더라고요.

```sh
$ mysql -u root -p
password:
```

그리고 비밀번호도 넣어야하는데 설치할 때 분명 비밀번호 같은건 입력하지 않았는데 모르겠는 상황!! 일단 진입은 `mysql -u root -p`로 하면 돼요. 하지만 비밀번호를 알아야해요.

```sh
$ sudo mysql
```

만능 `sudo`를 붙이면 뚫고 진입이 됩니다🤣 진입해서 mysql db의 user 테이블을 수정해서 비밀번호를 변경할게요.

일단, user 테이블을 확인합니다.

```sql
mysql> show databases;

mysql> use mysql;
mysql> select host, user, plugin, authentication_string from user;
+-----------+------------------+-----------------------+--------------------------+
| host      | user             | plugin                | authentication_string
        |
+-----------+------------------+-----------------------+--------------------------+
| %         | hyejun           | caching_sha2_password |                          |
| localhost | debian-sys-maint | caching_sha2_password |                          |
| localhost | mysql.infoschema | caching_sha2_password |                          |
| localhost | mysql.session    | caching_sha2_password |                          |
| localhost | mysql.sys        | caching_sha2_password |                          |
| localhost | root             | caching_sha2_password |
        |
+-----------+------------------+-----------------------+--------------------------+
```

비밀번호는 제가 지워놓은 거고 이렇게 뜰겁니다. 비밀번호는 암호화가 되어있어서 확인할 수 없어요. 따로 명령어로 초기화할께요.

```sql
mysql> alter user 'root'@'localhost' identified by '';
mysql> FLUSH PRIVILEGES;
```
root는 서버에서만 접속 가능하니 비밀번호를 없애버릴게요. `FLUSH PRIVILEGES;`는 사용자에 대한 정보 변경했을 때 해주셔야 반영이 돼요.

## 새로운 유저 생성

이제 외부에서 접속할 새로운 유저 데이터를 생성합니다.

```sql
mysql> CREATE USER '[새로 만들 유저 이름]'@'%' IDENTIFIED BY '[유저 비밀번호]';
mysql> GRANT ALL PRIVILEGES ON *.* TO '[새로 만들 유저 이름]'@'%';
mysql> FLUSH PRIVILEGES;
```

유저 이름 뒤에 ip 대신 `'%'`라는 문자열이 붙었는데요. 이건 모든 아이피를 허용한다는 뜻입니다.

`*.*`의 의미는 원래 `database.table`로 특정 데이터베이스(schema)의 특정 테이블을 허용하도록 할 수 있는데 여기서는 모든 데이터베이스의 모든 테이블을 허용하도록 하는 코드에요.

마찬가지로 `FLUSH PRIVILEGES;`로 반영해줍니다.

## 방화벽 해제

이제 user를 만들었으니 접속이 될 줄알았으나\... 호락호락하지 않네요.

```bat
> mysql -h [아이피 주소] -u [만든 아이디] -p
Enter password: [만든 비밀번호]
```

이러면 되야하는데 10060 error가 나왔어요.

```bat
Enter password: ****
ERROR 2003 (HY000): Can't connect to MySQL server on '[아이피주소]:3306' (10060)
```

찾아보니까 AWS에서 방화벽을 열어야했어요. AWS의 보안그룹에서 인바운드 규칙에서 MYSQL 3306포트를 0.0.0.0으로 수정해줍니다.

AWS의 방화벽 열어두셔야하는 건 요정도에요. 저는 현재 이렇게 열어놨어요.

* 인바운드 - SSH(22포트), HTTP(80포트), HTTPS(443포트), MYSQL(3306포트)
* 아웃바운드 -  SSH(22포트), HTTP(80포트), HTTPS(443포트)

## 이번엔 10061 Error?

이젠 될 줄 알았는데 이번엔 10061 에러가 나버리네요. 진짜 멘탈 박살 날뻔했지만 어떻게 다 해결했습니다.

```bat
Enter password: ****
ERROR 2002 (HY000): Can't connect to MySQL server on '[아이피주소]:3306' (10061)
```

서버의 mysql 설정에서 3306포트를 전체로 열어줘야해요.

다시 ssh로 서버에 접속합니다.

```sh
$ netstat -lntp
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:33060           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      -
tcp6       0      0 :::22                   :::*                    LISTEN      -
tcp6       0      0 :::80                   :::*                    LISTEN      -
```

저는 현재 수정한 상태라 3306포트가 `0.0.0.0`으로 나오는데, 에러가 발생했다만 `127.0.0.1:3306`으로 되어있을 거에요. config 파일을 열어서 수정해야합니다.

목표는 `bind-address = 127.0.0.1`을 `bind-address = 0.0.0.0`으로 수정하는 겁니다.

```sh
$ vi /etc/mysql/my.cnf
```

여기에 있을 수도 있다는데 저는 다른 파일로 연결돼있었어요.

```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

저 directory로 이동해봅니다.

```sh
$ cd /etc/mysql/conf.d
$ ls
mysql.cnf  mysqldump.cnf
```

여기에 있는 `mysql.cnf`과 `mysqldump.cnf`는 의미없는 값만 있었어요.

```sh
$ cd /etc/mysql/mysql.conf.d
$ ls
mysql.cnf  mysqld.cnf
```

드디어 여기서 `mysqld.cnf`에 설정값들이 들어있는걸 찾았습니다.

```sh
vi mysqld.cnf
```

여기서 bind-address와 mysqlx-bind-address를 모두 0.0.0.0으로 변경했어요.

```
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```

이제 저장하고 mysql을 restart 해볼께요.

```sh
$ service mysql restart
```

## 성공!



## 👀참고
* [[Ubuntu] 우분투에 MySQL 설치하기 \| ㅇㅅㅇ.devlog](https://velog.io/@seungsang00/Ubuntu-%EC%9A%B0%EB%B6%84%ED%88%AC%EC%97%90-MySQL-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)
* [Linux : MySQL 사용자 비밀번호 변경 방법, 예제, 명령어 \| 쵸코쿠키의 연습장](https://jjeongil.tistory.com/1484)
* [Mysql 계정 비밀번호 변경 | yebali.log](https://velog.io/@yebali/Mysql-Mysql-%EA%B3%84%EC%A0%95-%EB%B9%84%EB%B0%80%EB%B2%88%ED%98%B8-%EB%B3%80%EA%B2%BD)
* [MySQL 8 인증 플러그인 암호화 방식 변경 \| AllThatLinux](https://atl.kr/dokuwiki/doku.php/mysql_8_%EC%9D%B8%EC%A6%9D_%ED%94%8C%EB%9F%AC%EA%B7%B8%EC%9D%B8_%EC%95%94%ED%98%B8%EB%B0%A9%EC%8B%9D_%EB%B3%80%EA%B2%BD)
* [MySQL 원격 접속 허용 \| ZETAWIKI](https://zetawiki.com/wiki/MySQL_%EC%9B%90%EA%B2%A9_%EC%A0%91%EC%86%8D_%ED%97%88%EC%9A%A9)
* [[MySQL 8.0] MySQL 유저 원격접속 허용하기 \| i-mini](https://1mini2.tistory.com/87)
* [우여곡절 MySql 원격접속 하기ㅠ.ㅠ \| wpdlzhf159.log](https://velog.io/@wpdlzhf159/MySql-%EC%9B%90%EA%B2%A9%EC%A0%91%EC%86%8D-%ED%95%98%EA%B8%B0)
* [[AWS] : AWS Ec2 Ubuntu Mysql 외부 접속하기 \| 노력을 쌓는 개발자 오주현](https://ohju.tistory.com/315)
* [MYSQL(MariaDB)에서 외부접근이 되지않을때(Feat. Can’t Connect To MySQL Server On ‘192.168.X.X'(10061) \| 달소씨의 하루](https://blog.dalso.org/it/4260)
