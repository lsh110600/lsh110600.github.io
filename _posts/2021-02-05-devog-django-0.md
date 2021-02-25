---
layout: post
title: '[Django] Django 세팅'
subtitle: 'Django 세팅'
categories: devlog
tags: Django 
comments: true
---

### DB확인

python manage.py shell
from django_app.models import Sgp30
sgp30 = Sgp30.objects.values() 

time_list = [sgp30[_]['collect_time'].strftime("%Y%m%d_%H%M%S") for _ in range(len(sgp30))]
co2_list = [int(sgp30[_]['co2']) for _ in range(len(sgp30))]    
tvoc_list = [int(sgp30[_]['tvoc']) for _ in range(len(sgp30))]   



### DB 상태 수정
mysql> describe bme280;
+--------------+-------------+------+-----+---------+-------+
| Field        | Type        | Null | Key | Default | Extra |
+--------------+-------------+------+-----+---------+-------+
| collect_time | datetime    | NO   |     | NULL    |       |
| humidity     | varchar(12) | YES  |     | NULL    |       |
| temperature  | varchar(12) | YES  |     | NULL    |       |
| pressure     | varchar(12) | YES  |     | NULL    |       |
| altitude     | varchar(12) | YES  |     | NULL    |       |
+--------------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)

- primary key 추가 [출처](http://www.tcpschool.com/mysql/mysql_constraint_primaryKey)

```sql
mysql> alter table sgp30
    -> modify column collect_time datetime PRIMARY KEY
    -> ;
```

- primary key 삭제

```sql
mysql> alter table sgp30 drop primary key;
```

- id 만들기 (row num)

alter table veml7700 add id int PRIMARY KEY auto_increment;

- 컬럼 첫번째 위치로 이동
ALTER TABLE 테이블명 MODIFY COLUMN 컬럼명 자료형 FIRST;

```sql
mysql> alter table veml7700 MODIFY COLUMN id int FIRST;
```

alter table veml7700 modify id int not null auto_increment;



- collect_time PRI 제거 후 id 입력
mysql> describe sgp30
    -> ;
+--------------+-------------+------+-----+---------+----------------+
| Field        | Type        | Null | Key | Default | Extra          |
+--------------+-------------+------+-----+---------+----------------+
| collect_time | datetime    | NO   |     | NULL    |                |
| co2          | varchar(12) | YES  |     | NULL    |                |
| tvoc         | varchar(12) | YES  |     | NULL    |                |
| id           | int(11)     | NO   | PRI | NULL    | auto_increment |
+--------------+-------------+------+-----+---------+----------------+



mysql> describe sgp30;
+--------------+-------------+------+-----+---------+-------+
| Field        | Type        | Null | Key | Default | Extra |
+--------------+-------------+------+-----+---------+-------+
| id           | int(11)     | NO   | PRI | NULL    |       |
| collect_time | datetime    | NO   |     | NULL    |       |
| co2          | varchar(12) | YES  |     | NULL    |       |
| tvoc         | varchar(12) | YES  |     | NULL    |       |
+--------------+-------------+------+-----+---------+-------+
4 rows in set (0.01 sec)


## 유용한 장고 ORM 메소드 정리 사이트
https://velog.io/@matisse/Django-%EA%B0%9D%EC%B2%B4-method-%EC%A7%91%EC%A4%91-%ED%83%90%EA%B5%AC



## DB 설정 변경 시 장고에서 싱크 맞추기

$ python manage.py inspectdb > dashboard/models.py
$ python manage.py makemigrations
$ python manage.py migrate



-- 다시 처음부터 --


## 장고에서 대시보드 만들기
가상환경 생성 / 접속

접속 : source dashboard/bin/activate

https://github.com/Nels885/django_sb_admin_2.git

python manage.py migrate
python manage.py runserver 8080

## DB 연동

django_sb_admin_2/sbadmin/setings.py 에서 아래 코드 추가/수정


```python
import pymysql
pymysql.install_as_MySQLdb()

DATABASES = {
    
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'sensor_db',
        'USER': 'lsh110600',
        'PASSWORD': '1234',
        'HOST': 'localhost',
        'PORT': '3306',
    }   
}
```

$ python manage.py inspectdb > dashboard/models.py
$ python manage.py makemigrations
$ python manage.py migrate



 var ctx = document.getElementById("myAreaChart");
                        var myLineChart = new Chart(ctx, {




