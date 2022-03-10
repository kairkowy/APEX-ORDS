오라클 APEX & ORDS 설치 및 구성 예제
===============
이 예제는 APEX 21.2과 ORDS standalone 21.4.1을 사용하는 예제임. mod_plsql, Embedded PLSQL 게이트웨이를 사용해야 하는 경우에는 APEX 20.1 이전 버전을 사용해야 함.

참고자료
 + Oracle Application ExpressRelease 21.1 Installation Guide
https://docs.oracle.com/en/database/oracle/application-express/21.1/htmig/apex-installation-requirements.html#GUID-02BE4A34-B631-412C-8A82-EB92DABBACE0

 + Oracle REST Data ServicesRelease 21.1 Installation, Configuration, and Development Guide

   https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/21.1/aelig/installing-REST-data-services.html#GUID-A16BCCA2-8081-4062-A635-9F7C36FC394F

작성자/작성일 : 고운용 / 2022. 3.

예제 환경

 + 운영체제 : Oracle Linux 7.9
 + DB 버전 : Oracle EE 19.12, PDB only
 + APEX 버전 : 21.2
 + ORDS 버전 : 21.4.1
 + Oracle JDK 17.0.2
 + 서버 IP : 192.168.0.23
 + Hostname : wokot450.kr.oracle.com

## 오라클 APEX 설치

### APEX 설치 요구사항
* Oracle database 12.1 이후 버전의 Oracle EE, SE, XE 에디션에서 지원
* Oracle RAC 환경 지원
* Memory_target 사이즈 최소 300MB 이상 필요
* Oracle XML DB 기능 필요(DBCA로 DB생성시 자동 인스톨 됨.)
* 지원되는 브라우져 : Chrome, Mozilla, Safari, Edge
* 웹 리스너(서버) : 오라클 웹로직 서버, Oracle HTTP 서버, Apache 또는 ORDS 19.x (standalone 구성시) 이후 버전

### APEX 설치

#### APEX 다운로드 및 압축 풀기
* apex_21.1.zip 다운로드
  https://www.oracle.com/tools/downloads/apex-downloads.html

```shell
unzip apex_21.2.zip
cd apex
ls
apexins1.sql         apexins_nocdb.sql           appins.sql          apxdvins_cdb.sql    apxpatch.sql        apxrtins1.sql         apxrtins.sql      coreins4.sql  images
apexins2.sql         apexins.sql                 apxappcon.sql       apxdvins_nocdb.sql  apxremov1.sql       apxrtins2.sql         apxsilentins.sql  coreins5.sql  install2022-03-03_13-47-32.log
apexins3.sql         apex_rest_config_cdb.sql    apxchpwd.sql        apxdvins.sql        apxremov2.sql       apxrtins3.sql         builder           coreins.sql   LICENSE.txt
apexins_adb.sql      apex_rest_config_core.sql   apxdevrm_cdb.sql    apxdwngrd.sql       apxremov_cdb.sql    apxrtins_cdb.sql      core              dbcsconf.sql  load_trans.sql
apexins_cdb.sql      apex_rest_config_nocdb.sql  apxdevrm_nocdb.sql  apxpatch_cdb.sql    apxremov_nocdb.sql  apxrtins_cdb_upg.sql  coreins2.sql      dbcsins.sql   utilities
apexins_cdb_upg.sql  apex_rest_config.sql        apxdevrm.sql        apxpatch_nocdb.sql  apxremov.sql        apxrtins_nocdb.sql    coreins3.sql      devins.sql
```
### APEX DB 환경 구성

APEX는 Full 개발자 환경과 런타임 환경을 각각 지원함. 런타임 환경은 별도로 Full 개발자 환경으로 전환이 가능함


#### Full 개발자 환경인 경우

Full 개발자 환경인 경우에는 apexins.sql을 실행하고 런타임 환경인 경우에는 apexrins.sql을 실행함

```sql
SYS 계정으로
SQL>@apexins.sql SYSAUX SYSAUX TEMP /i/  
```
####  Runtime 환경인 경우

```sql
SYS 계정으로
SQL>@apexrtins.sql SYSAUX SYSAUX TEMP /i/

...set_appun.sql
PL/SQL procedure successfully completed.
...
...
...
Next Steps:

 1. Create your Instance Administration Account
    Run the apxchpwd.sql script

 2. Configure the APEX_PUBLIC_USER Account
    See the Installation Guide for details

 3. Configure RESTful Services
    Run the apex_rest_config.sql script

timing for: Phase 3 (Switch)
Elapsed: 00:00:09.10
timing for: Complete Installation
Elapsed: 00:06:42.73
```

* 환경 구성 SQL 파일 실행과정에서 APEX에 필요한 DB계정이 생성됨.
   + APEX_210200) : APEX 스키마와 메터정보 소유자(이 계정은 버전별로 계정명이 변경되는 것 같음)
   + FLOWS_FILES : APEX 업로드 파일들의 소유자
   + APEX_PUBLIC_USER : ORDS, HTTP Server, mod_sql 관련 구성에 사용되는 유저
* ORDS(RESTFul) 서비스 구성된 경우 추가되는 DB 계정.
   + APEX_REST_PUBLIC_USER : APEX에 저장된 RESTfule 서비스 정의 호출용 계정
   + APEX_LISTENR : APEX에 저장된 RESTful 서비스 정의를 쿼리하는 데 사용되는 계정.

#### APEX 인스턴스 관리자 계정 생성

APEX 인스턴스 관리자는 APEX 워크스페이스 프로비저닝 관리, APEX 기능 및 인스턴스 구성, 보안 관리 등을 책임지는 APEX 관리 수퍼유저임.

인스턴스 관리자 계정 생성을 위해 apxchpwd.sql 을 실행 함.
 - account : admin   ---  Default 계정이며 다른 계정 이름을 사용 가능함.
 - email address : admin@...    -- 이메일 계정 입력.
 - password : Welcome1#  --- 특수문자 포함 해야함

```sql
SYS계정으로

SQL>@apxchpwd.sql

Enter the administrator's username [ADMIN]
User "ADMIN" does not yet exist and will be created.
Enter ADMIN's email [ADMIN] admin@kr.oracle.com
Enter ADMIN's password []
Created instance administrator ADMIN.
```

* apxchpwd.sql 역할
  + 첫 APEX 인스턴스 관리자(admin) 생성 및 비밀번호 변경
  + Runtime 환경을 개발환경으로 변환 할때 비밀번호 변경
  + 인스턴스 관리자(admin) 비밀번호 변경
  + 인스턴스 관리자 계정 언락킹(unlocking)

#### Restart (http 데몬)

기존에 HTTP서비스 중인 환경에서는 HTTP 데몬을 재기동 해줘야 함

#### APEX_PUBLIC_USER 계정 환경설정

"apex_public_user"는 디폴트 계정이며 초기 구성할 떄는 계정 활성화 및 비밀번호 변경을 해야 함.

```sql
SQL>ALTER USER APEX_PUBLIC_USER ACCOUNT UNLOCK ;
SQL>ALTER USER APEX_PUBLIC_USER IDENTIFIED BY Welcome1;
```

#### ORDS RESTful 서비스 환경 구성(Must)
APEX를 처음 설치 할 경우에는 apex_rest_config.sql 반드시 실행해야 함.
APEX_LISTENER, APEX_REST_PUBLIC_USER 계정 및 비밀번호 생성이 수행됨

```sql
SYS 계정으로
SQL>@apex_rest_config.sql

Enter a password for the APEX_LISTENER user              []
Enter a password for the APEX_REST_PUBLIC_USER user              []
...set_appun.sql
...setting session environment
...create APEX_LISTENER and APEX_REST_PUBLIC_USER users
...grants for APEX_LISTENER and ORDS_METADATA user


```

* 이 예제에서는 WEB 서비스를 Oracle REST Data Service(ORDS) standalone를 사용할 것임.
* Apache 또는 Oracle HTTP를 사용할 경우에는 ORDS 설치 가이드를 참고 바람.




ORDS standalone 서비스 설치 및 구성
---------

### ORDS 설치 파일 다운로드 및 압축해제

최신 또는 원하는 버전의 ords 다운로드
https://www.oracle.com/technetwork/developer-tools/rest-data-services/downloads/index.html

* ords-21.4.1.025.0904.zip 파일 다운로드 (ords.version.number.zip)

### 디렉토리 생성 및 압축 해제

다운로드 파일을 압축해제 함.

```shell
unzip ords-21.4.1.025.0904.zip -d ./ords
cd ords
ls
docs  examples  FUTC.txt  index.html  installer  license.txt  ords.war  params
```
* ordw.war 파일 확인
  + context root를 /ords 로 하고싶을 때는 ords.war로 /apex로 하고 싶을 경우는 ords.war를 apex.war로 rename 하면 됨

### 이미지 디렉토리 복사

APEX가 설치된 디렉토리에서 apex/images 디렉토리를 ords 서비스가 설치 된 디렉토리로 복사

```shell
mkdir -p ords/apex/images
cp -r apex/images/ ./ords/apex/
```

### Static 파일 지원 환경 설정

Static 파일을 이용하여 ORDS Standalone 모드를 설치하려면 APEX 신규 설치 후에는 다음 SQL을 실행해야 함.
  + ords_installer_privileges.sql는 지정된 데이터베이스 사용자에게 ORDS 설치, 업그레이드, 검증 및 제거할 수 있는 권한을 제공하는 스크립트임.
  + 여기서는 "ORDS" 계정을 사용할 예정이기 때문에 먼저 계정을 생성하고 나서 설치 SQL을 실행 함.

```sql  
SYS 계정
SQL>create user ords identified by Welcome1 default tablespace users quota unlimited on users;
SQL>grant connect, resource to ords;
SQL>@./ords/installer/ords_installer_privileges.sql ords
Grant succeeded.
...
...
...
PL/SQL procedure successfully completed.
```

### ORDS 설치
ORDS 설치는 커맨드 라인으로 Step-by-Step으로 설치 할 수 있으나, 여기서는 parameter 파일을 이용해서 설치를 진행 할 것임

#### parameter 파일 수정

파일 위치 : $ORDS_HOME/params/ords_params.properfies

parameter 파일 샘플
```ords_parameter

### ORDS config
db.connectionType=basic
db.hostname=wokot450.kr.oracle.com
db.port=1521
db.servicename=pdb18.kr.oracle.com
db.sid=pdb18
## DB connection : tns type
#db.connectionType=tns
#db.tnsDirectory=/home/oracle/oracle_19c/network/admin
#db.tnsAliasName=pdb18
## DB connection : custom type
#db.connectionType=customurl
#db.customURL=jdbc:oracle:thin:@//wokot450.kr.oralce.com:1521/pdb18

db.username=APEX_PUBLIC_USER
migrate.apex.rest=false
rest.services.ords.add=true
user.public.password=Welcome1
schema.tablespace.default=SYSAUX
schema.tablespace.temp=TEMP
standalone.http.port=8080
standalone.mode=true
standalone.static.images=/home/woko/apex/images
standalone.use.https=false
user.tablespace.default=SYSAUX
user.tablespace.temp=TEMP
bequeath.connect=false

## SQl Developer Web
feature.sdw=true
restEnabledSql.active=true
database.api.enabled=true
## Database API
database.api.enabled=true

### APEX config
db.username=APEX_PUBLIC_USER
db.password=Welcome1
plsql.gateway.add=true
rest.services.apex.add=true
user.apex.listener.password=Welcome1
user.apex.restpublic.password=Welcome1
user.public.password=Welcome1

```

#### Static 파일을 이용한 ORDS 설치

ORDS는 Oracle JAVA 1.8 이상의 버전을 필요로 함.

```shell
java -jar ords.war install --parameterFile /home/woko/ords/params/ords_params.properties
This Oracle REST Data Services instance has not yet been configured.
Please complete the following prompts


Enter the location to store configuration data: /home/woko/ords/config
Requires to login with administrator privileges to verify Oracle REST Data Services schema.

Enter the administrator username:ords
Enter the database password for ords:
Confirm password:
Passwords do not match, try again
Enter the database password for ords:
Confirm password:
Connecting to database user: ords url: jdbc:oracle:thin:@//wokot450.kr.oracle.com:1521/pdb18.kr.oracle.com

Retrieving information.
2022-03-03T05:37:07.710Z INFO        reloaded pools: []
Installing Oracle REST Data Services version 21.4.1.r0250904
... Log file written to /home/woko/ords_install_core_2022-03-03_143707_00840.log
... Verified database prerequisites
... Created Oracle REST Data Services proxy user
... Created Oracle REST Data Services schema
... Granted privileges to Oracle REST Data Services
... Created Oracle REST Data Services database objects
... Log file written to /home/woko/ords_install_datamodel_2022-03-03_143724_00260.log
... Log file written to /home/woko/ords_install_scheduler_2022-03-03_143725_00398.log
... Log file written to /home/woko/ords_install_apex_2022-03-03_143727_00635.log
Completed installation for Oracle REST Data Services version 21.4.1.r0250904. Elapsed time: 00:00:21.141

2022-03-03T05:37:29.077Z INFO        HTTP and HTTP/2 cleartext listening on host: localhost port: 8080
2022-03-03T05:37:29.099Z INFO        Disabling document root because the specified folder does not exist: /home/woko/orda/config/ords/standalone/doc_root
2022-03-03T05:37:29.885Z INFO        Configuration properties for: |apex||
db.servicename=pdb18.kr.oracle.com
error.keepErrorMessages=true
db.hostname=wokot450.kr.oracle.com
db.password=******
cache.directory=/tmp/apex/cache
debug.debugger=false
security.requestValidationFunction=wwv_flow_epg_include_modules.authorize
log.logging=false
cache.duration=days
security.maxEntries=2000
cache.expiration=7
resource.templates.enabled=true
error.maxEntries=50
db.port=1521
debug.printDebugToScreen=false
jdbc.DriverType=thin
jdbc.MinLimit=1
jdbc.statementTimeout=900
cache.procedureNameList=
jdbc.InactivityTimeout=1800
jdbc.MaxLimit=10
misc.defaultPage=apex
cache.maxEntries=500
database.api.enabled=true
db.username=APEX_PUBLIC_USER
cache.type=lru
jdbc.MaxConnectionReuseCount=1000
restEnabledSql.active=true
security.disableDefaultExclusionList=false
jdbc.MaxStatementsLimit=10
cache.caching=false
feature.sdw=true
misc.compress=
security.validationFunctionType=plsql
jdbc.InitialLimit=3
cache.monitorInterval=60
log.maxEntries=50
db.connectionType=basic

2022-03-03T05:37:29.887Z WARNING     *** jdbc.MaxLimit in configuration |apex|| is using a value of 10, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:29.887Z WARNING     *** jdbc.InitialLimit in configuration |apex|| is using a value of 3, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:30.899Z INFO        Configuration properties for: |apex|pu|
db.servicename=pdb18.kr.oracle.com
error.keepErrorMessages=true
db.hostname=wokot450.kr.oracle.com
db.password=******
cache.directory=/tmp/apex/cache
debug.debugger=false
security.requestValidationFunction=wwv_flow_epg_include_modules.authorize
log.logging=false
cache.duration=days
security.maxEntries=2000
cache.expiration=7
resource.templates.enabled=true
error.maxEntries=50
db.port=1521
debug.printDebugToScreen=false
jdbc.DriverType=thin
jdbc.MinLimit=1
jdbc.statementTimeout=900
cache.procedureNameList=
jdbc.InactivityTimeout=1800
jdbc.MaxLimit=10
misc.defaultPage=apex
cache.maxEntries=500
database.api.enabled=true
db.username=ORDS_PUBLIC_USER
cache.type=lru
jdbc.MaxConnectionReuseCount=1000
restEnabledSql.active=true
security.disableDefaultExclusionList=false
jdbc.MaxStatementsLimit=10
cache.caching=false
feature.sdw=true
misc.compress=
security.validationFunctionType=plsql
jdbc.InitialLimit=3
cache.monitorInterval=60
log.maxEntries=50
db.connectionType=basic

2022-03-03T05:37:30.900Z WARNING     *** jdbc.MaxLimit in configuration |apex|pu| is using a value of 10, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:30.900Z WARNING     *** jdbc.InitialLimit in configuration |apex|pu| is using a value of 3, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:31.180Z INFO        Configuration properties for: |apex|al|
db.servicename=pdb18.kr.oracle.com
error.keepErrorMessages=true
db.hostname=wokot450.kr.oracle.com
db.password=******
cache.directory=/tmp/apex/cache
debug.debugger=false
security.requestValidationFunction=wwv_flow_epg_include_modules.authorize
log.logging=false
cache.duration=days
security.maxEntries=2000
cache.expiration=7
resource.templates.enabled=true
error.maxEntries=50
db.port=1521
debug.printDebugToScreen=false
jdbc.DriverType=thin
jdbc.MinLimit=1
jdbc.statementTimeout=900
cache.procedureNameList=
jdbc.InactivityTimeout=1800
jdbc.MaxLimit=10
misc.defaultPage=apex
cache.maxEntries=500
database.api.enabled=true
db.username=APEX_LISTENER
cache.type=lru
jdbc.MaxConnectionReuseCount=1000
restEnabledSql.active=true
security.disableDefaultExclusionList=false
jdbc.MaxStatementsLimit=10
cache.caching=false
feature.sdw=true
misc.compress=
security.validationFunctionType=plsql
jdbc.InitialLimit=3
cache.monitorInterval=60
log.maxEntries=50
db.connectionType=basic

2022-03-03T05:37:31.181Z WARNING     *** jdbc.MaxLimit in configuration |apex|al| is using a value of 10, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:31.182Z WARNING     *** jdbc.InitialLimit in configuration |apex|al| is using a value of 3, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:31.395Z INFO        Configuration properties for: |apex|rt|
db.servicename=pdb18.kr.oracle.com
error.keepErrorMessages=true
db.hostname=wokot450.kr.oracle.com
db.password=******
cache.directory=/tmp/apex/cache
debug.debugger=false
security.requestValidationFunction=wwv_flow_epg_include_modules.authorize
log.logging=false
cache.duration=days
security.maxEntries=2000
cache.expiration=7
resource.templates.enabled=true
error.maxEntries=50
db.port=1521
debug.printDebugToScreen=false
jdbc.DriverType=thin
jdbc.MinLimit=1
jdbc.statementTimeout=900
cache.procedureNameList=
jdbc.InactivityTimeout=1800
jdbc.MaxLimit=10
misc.defaultPage=apex
cache.maxEntries=500
database.api.enabled=true
db.username=APEX_REST_PUBLIC_USER
cache.type=lru
jdbc.MaxConnectionReuseCount=1000
restEnabledSql.active=true
security.disableDefaultExclusionList=false
jdbc.MaxStatementsLimit=10
cache.caching=false
feature.sdw=true
misc.compress=
security.validationFunctionType=plsql
jdbc.InitialLimit=3
cache.monitorInterval=60
log.maxEntries=50
db.connectionType=basic

2022-03-03T05:37:31.395Z WARNING     *** jdbc.MaxLimit in configuration |apex|rt| is using a value of 10, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:31.396Z WARNING     *** jdbc.InitialLimit in configuration |apex|rt| is using a value of 3, this setting may not be sized adequately for a production environment ***
2022-03-03T05:37:35.426Z INFO        Oracle REST Data Services initialized
Oracle REST Data Services version : 21.4.1.r0250904
Oracle REST Data Services server info: jetty/9.4.44.v20210927


```
* ORDS standalone은 Jetty 서비스가 패키딩 되어 서비스가 됨.

#### Ctrl+c 로 프로세스를 종료 시킴

#### Document root 디렉토리 생성

```shell
mkdir -p /home/woko/ords/config/ords/standalone/doc_root
```

### ORDS standalone 서비스 확인

* 테스트 파일 생성
```shell
cd /home/woko/ords/config/ords/standalone/doc_root
vi hello.txt
hi~ APEX
```

* ords standalone 서비스 시작
```shell
java -Dorg.eclipse.jetty.server.Request.maxFormContentSize=3000000 -Duser.timezone=UTC -jar ords.war standalone
```

* 브라우저에서
http://192.168.0.23:8080/hello.txt

     hi~ APEX 확인 ~~~ 해주세요


### 네트워크 서비스 활성화

* APEX에서 PDF 리포팅, 웹서비스 사용 또는 이메일 전송 등을 위해서는 네트워크 서비스를 활성화해야 함.
* APEX_210200 유저에 대해 모든 호스트 연결 권한을 부여하게 됨.
* 12c 이후 부터는 "APPEND_HOST_ACE" 사용을 권함.
* DBMS_NETWORK_ACL_ADMIN의 CREATE_ACL, ASSIGN_ACL,ADD_PRIVILEGE 및 CHECK_PRIVILEGE 프로시저는 12이후 부터 미지원 됨

### 접속 권한 부여
1. 모든 host에 허용할 경우
```sql
SYS 계정으로
BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host => '*',
        ace => xs$ace_type(privilege_list => xs$name_list('connect'),
                           principal_name => 'APEX_210200',
                           principal_type => xs_acl.ptype_db));
END;
/
```
2. local host에만 허용할 경우
```sql
SYS 계정으로 실행
BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
        host => 'localhost',
        ace => xs$ace_type(privilege_list => xs$name_list('connect'),
                           principal_name => 'APEX_210100',
                           principal_type => xs_acl.ptype_db));
END;
/
```

### 언어 설정
* 여기서는 생략함 . 참조 문서 참조


## APEX 워크스페이스 생성 예제
* APEX 관리 화면을 실행하고 개발자 계정 생성 예제를 진행 함

```
http://192.168.0.23:8080/ords/apex_admin

계정: admin
패스워드 : ********

```
<img src="./images/create_workspace_main.JPG" width="80%" height="80%" title="APEX Admin 메인화면" alt="Admin_Main"></img>

<img src="./images/create_workspace_1.JPG" width="80%" height="80%" title="APEX Admin 메인화면" alt="Admin_Main"></img>

<img src="./images/create_workspace_2.JPG" width="80%" height="80%" title="APEX Admin 메인화면" alt="Admin_Main"></img>

<img src="./images/create_workspace_3.JPG" width="80%" height="80%" title="APEX Admin 메인화면" alt="Admin_Main"></img>

<img src="./images/create_workspace_4.JPG" width="80%" height="80%" title="APEX Admin 메인화면" alt="Admin_Main"></img>



지금까지 APEX 설치를 모두 끝냈고 admin 로그인과 워크스페이스 만드는 예제를 수행했습니다.

다음은 설치된 ORDS 서비스를 이용한 샘플 예제를 알아보겠습니다. 이 샘플 예제에는 클라이언트 자격인증을 위한 OAuth 기능도 포함합니다.

## ORDS 서비스 예제

### 계정 및 데이터 생성

* ORDS 서비스 예제는 ordstest 계정을 가용 함.
+ 계정명 : ordstest
+ 예제 테이블명 : emp

```sql

create user ordstest identified by Welcome1 default tablespace users quota unlimited on users;
grant connect, resource to ordstest;

conn ordstest/Welcome1@pdb18

create table emp (
empno number(4,0),
ename varchar2(10 byte),
job varchar2(9 byte),
mgr number(4,0),
hiredate date,
sal number(7,2),
comm number(7,2),
deptno number(2,0),
constraint pk_emp primary key (empno)
);

insert into emp  values(7839, 'KING', 'PRESIDENT', null, to_date('17-11-1981','dd-mm-yyyy'), 5000, null, 10);
insert into emp  values(7698, 'BLAKE', 'MANAGER', 7839,  to_date('1-5-1981','dd-mm-yyyy'),  2850, null, 30);
insert into emp  values(7782, 'CLARK', 'MANAGER', 7839,  to_date('9-6-1981','dd-mm-yyyy'),  2450, null, 10);
insert into emp  values(7566, 'JONES', 'MANAGER', 7839,  to_date('2-4-1981','dd-mm-yyyy'),  2975, null, 20);
insert into emp  values(7788, 'SCOTT', 'ANALYST', 7566,  to_date('13-JUL-87','dd-mm-rr') - 85,  3000, null, 20);
insert into emp  values(7902, 'FORD', 'ANALYST', 7566,  to_date('3-12-1981','dd-mm-yyyy'), 3000, null, 20);
insert into emp  values(7369, 'SMITH', 'CLERK', 7902,  to_date('17-12-1980','dd-mm-yyyy'),  800, null, 20);
insert into emp  values(7499, 'ALLEN', 'SALESMAN', 7698,  to_date('20-2-1981','dd-mm-yyyy'),  1600, 300, 30);
insert into emp  values(7521, 'WARD', 'SALESMAN', 7698,  to_date('22-2-1981','dd-mm-yyyy'),  1250, 500, 30);
insert into emp  values(7654, 'MARTIN', 'SALESMAN', 7698,  to_date('28-9-1981','dd-mm-yyyy'),  1250, 1400, 30);
insert into emp  values(7844, 'TURNER', 'SALESMAN', 7698,  to_date('8-9-1981','dd-mm-yyyy'),  1500, 0, 30);
insert into emp  values(7876, 'ADAMS', 'CLERK', 7788,  to_date('13-JUL-87', 'dd-mm-rr') - 51,  1100, null, 20);
insert into emp  values(7900, 'JAMES', 'CLERK', 7698,  to_date('3-12-1981','dd-mm-yyyy'),  950, null, 30);
insert into emp  values(7934, 'MILLER', 'CLERK', 7782,  to_date('23-1-1982','dd-mm-yyyy'),  1300, null, 10);

commit;
```

### 리소스 생성   

#### 스키마 활성화

```sql
ordstest 계정으로

begin
  ords.enable_schema;
  commit;
end;
/
```

#### 서비스 리소스 생성

```sql
begin
ords.create_service(
  p_module_name => 'examples.employees' ,
  p_base_path => '/examples/employees/',
  p_pattern => '.' ,
  p_items_per_page => 7,
  p_source => 'select * from emp order by empno desc');
  commit;
end;
/

col source_type for a30
col source for a40
SELECT id, template_id, source_type, method, source
FROM   user_ords_handlers ;
        ID TEMPLATE_ID SOURCE_TYPE                              METHOD     SOURCE
---------- ----------- ---------------------------------------- ---------- ----------------------------------------
     10088       10087 json/collection                          GET        select * from emp order by empno desc

-- exec ords.delete_module('examples.employees');
```

#### 리소스 확인

cURL 명령어로 리소스 확인

```shell
curl -i http://192.168.0.23:8080/ords/ordstest/examples/employees/

{"items":[{"empno":7934,"ename":"MILLER","job":"CLERK","mgr":7782,"hiredate":"1982-01-23T00:00:00Z","sal":1300,"comm":null,"deptno":10},{"empno":7902,"ename":"FORD","job":"ANALYST","mgr":7566,"hiredate":"1981-12-03T00:00:00Z","sal":3000,"comm":null,"deptno":20},{"empno":7900,"ename":"JAMES","job":"CLERK","mgr":7698,"hiredate":"1981-12-03T00:00:00Z","sal":950,"comm":null,"deptno":30},{"empno":7876,"ename":"ADAMS","job":"CLERK","mgr":7788,"hiredate":"1987-05-23T00:00:00Z","sal":1100,"comm":null,"deptno":20},{"empno":7844,"ename":"TURNER","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-08T00:00:00Z","sal":1500,"comm":0,"deptno":30},{"empno":7839,"ename":"KING","job":"PRESIDENT","mgr":null,"hiredate":"1981-11-17T00:00:00Z","sal":5000,"comm":null,"deptno":10},{"empno":7788,"ename":"SCOTT","job":"ANALYST","mgr":7566,"hiredate":"1987-04-19T00:00:00Z","sal":3000,"comm":null,"deptno":20}],"hasMore":true,"limit":7,"offset":0,"count":7,"links":[{"rel":"self","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/"},{"rel":"describedby","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/metadata-catalog/examples/employees/"},{"rel":"first","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/"},{"rel":"next","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/?offset=7"}]}

```
### 액세스 제어

#### 권한(Privilege) 생성 및 확인
 + "example.employees" privilige 와 'HR_Administrator' 를 생성
 + user_ords_privileges 메타정보로 Privilege Id, name을 확인

```sql
ordstest 계정으로

begin
  ords.create_role('HR_Administrator');     

  ords.create_privilege(
      p_name => 'example.employees',
      p_role_name => 'HR_Administrator',
      p_label => 'Employee Data',
      p_description => 'Provide access to employee HR data');
  commit;
end;
/
-- exec ords.delete_role('HR Administrator');
-- exec ords.delete_privilege('example.employees');

col name for a30
select id,name from user_ords_privileges where name = 'example.employees';
    ID NAME     
------ --------------------
10260 example.employees

```

### 권한(privilege)와 리소스 연결


```sql
ordstest 계정으로
begin
  ords.create_privilege_mapping(
        p_privilege_name => 'example.employees',
        p_pattern => '/examples/employees/*');       
        commit;
        end;
/
col name format a40
col pattern for a40
select privilege_id, name, pattern from user_ords_privilege_mappings;

PRIVILEGE_ID NAME                 PATTERN          
------------ --------------------  ---------------------
10260        example.employees    /examples/employees/*
```

#### 리소스 보호 확인

* example.employees 권한으로 리소스(/examples/employees/)가 보호되는지 확인 하는 단계임

```shell

curl -i http://192.168.0.23:8080/ords/ordstest/examples/employees/

HHTTP/1.1 401 Unauthorized
Content-Type: application/problem+json
Content-Length: 183

{
    "code": "Unauthorized",
    "message": "Unauthorized",
    "type": "tag:oracle.com,2020:error/Unauthorized",
    "instance": "tag:oracle.com,2020:ecid/w4UbuYdZgYcyPtP86tRUrg"
}
```

example.employees privilege에 의해 리소스 액세스가 보호되고 있음을 확인할 수 있음.


## OAuth 자격인증을 통한 서비스 보호

* 실제 어플리케이션 환경에서는 클라이언트 인증이 필요함. 어플리테이션 환경에서 사용 가능한 자격인증을 위해 OAuth 인증 예제를 다룰 것임.

* OAuth 자격인증 종류
 + cookie-based authentication : RESTful API와 동일 출처에서 호스팅 되는 웹 어플리케이션에서 RESTful API로 액세스 할 경우에 사용 가능
 + authorization code authentication : 3rd party 어플리케이션에 클라이언트 자격증명을 안전하게 유지할 수 있는 자체 웹 서버가 있는 경에 사용 가능
 + implicit authentication : 3rd party 응용프로그램에 자격증명을 안전하게 유지할 수 있는 웹 서버가 없는 경우에 사용 가능. 이 인증방식은 single-page-based 애플리케이션에 유용. Implicit 방식에서는 refresh token을 발급할 수 없어 사용자에게 인증요청 메시지가 더 자주 표시됨
 + client credentials authentication : 3rd party 애플리케이션에 RESTful API에 대한 직접 액세스 권한을 부여해야 하는 경우에 사용됨. 3rd party 응용프로그램은 서버 기반 응용프로그램이어야 함. Client credentials 인증 방식은 파일에 남기 저장 되기 때문에 애플리케이션과 클라이언트 자격증명을 같이 사용하지 말것.

 + Native 모바일 또는 데스크탑 애플리케이션에서 Authorization  또는 Implicit 인증 방식을 사용할 필요가 있음. 인증 승인 프로세스가 끝 날때 웹 브라우저에소 액세스 토큰을 캡처 해야 함.

### Cookie-based(file-based) 자격인증

#### 계정, 권한 및 롤 생성

* file-based 사용자 레퍼지토리를 사용하여 examples.employees privilege에 액세스하는 데 필요한 HR_Administrator 역할이 있는 테스트 계정 생성
 + 계정명 : hr_admnin
 + 권한명 :examples.employees
 + 롤명 : HR_Administrator

* 이 계정, 권한 및 롤은 다른 인증방식 예제에서도 동일하게 사용됨.

```shell
java -jar ords.war user hr_admin HR_Administrator
Enter a password for user hr_admin:
Confirm password for user hr_admin:
2022-03-03T08:15:36.083Z INFO        Created user: hr_admin in file: /home/woko/ords/config/ords/credentials
```

#### 사용자 사인 민 서비스 확인
* 웹브라우저에서 URL 입력 및 리턴 메시지 확인
* 401 에러가 나면서 "sign in" 링크 화면이 나타남. 클릭하고 사용자명 hr_admin, 패스워드 ***** 입력
* 페이지가 http://192.168.0.23:8080/ords/ordstest/examples/employees/ 으로 리다이렉션 되면서 JSON 문서가 표시됨

```url
http://192.168.0.23:8080/ords/ordstest/examples/employees/

401 Unauthorized

sign_in 링크 클릭
id: hr_admin
Password : 패스워드

JSON 문서 표시

```
<img src="./images/401_Unauthorized.JPG" width="80%" height="80%" title="Sign_In for OAuth" alt="SignIn"></img>


<img src="./images/oauth_after_signin_id.JPG" width="40%" height="40%" title="Sign_In for OAuth account" alt="SignIn_account"></img>

<img src="./images/json_service.JPG" width="40%" height="40%" title="JSON Service view" alt="JSONService"></img>

* cURL로 확인하는 방법

```shell
curl -i --user hr_admin:Welcome1 http://192.168.0.23:8080/ords/ordstest/examples/employees/

HTTP/1.1 200 OK
Content-Type: application/json
X-ORDS_DEBUG: false
ETag: "wSRZUqeyCwnAN0kso57DAVpBZKQesYpvrUAJiwS0045+Anmq36nXxGXl8/zZUwXKGLOmSObBIUO8ez6Z36vNUg=="
Transfer-Encoding: chunked

{"items":[{"empno":7934,"ename":"MILLER","job":"CLERK","mgr":7782,"hiredate":"1982-01-23T00:00:00Z","sal":1300,"comm":null,"deptno":10},{"empno":7902,"ename":"FORD","job":"ANALYST","mgr":7566,"hiredate":"1981-12-03T00:00:00Z","sal":3000,"comm":null,"deptno":20},{"empno":7900,"ename":"JAMES","job":"CLERK","mgr":7698,"hiredate":"1981-12-03T00:00:00Z","sal":950,"comm":null,"deptno":30},{"empno":7876,"ename":"ADAMS","job":"CLERK","mgr":7788,"hiredate":"1987-05-23T00:00:00Z","sal":1100,"comm":null,"deptno":20},{"empno":7844,"ename":"TURNER","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-08T00:00:00Z","sal":1500,"comm":0,"deptno":30},{"empno":7839,"ename":"KING","job":"PRESIDENT","mgr":null,"hiredate":"1981-11-17T00:00:00Z","sal":5000,"comm":null,"deptno":10},{"empno":7788,"ename":"SCOTT","job":"ANALYST","mgr":7566,"hiredate":"1987-04-19T00:00:00Z","sal":3000,"comm":null,"deptno":20}],"hasMore":true,"limit":7,"offset":0,"count":7,"links":[{"rel":"self","href":"http://192.168.0.23:8080/ords/ordstest/examples/employees/"},{"rel":"describedby","href":"http://192.168.0.23:8080/ords/ordstest/metadata-catalog/examples/employees/"},{"rel":"first","href":"http://192.168.0.23:8080/ords/ordstest/examples/employees/"},{"rel":"next","href":"http://192.168.0.23:8080/ords/ordstest/examples/employees/?offset=7"}]}

```

### OAuth Client credentials 자격인증
* Client credential OAuth flow로 "Client_Credentials_Example" 이름의 클라이언트 등록 후 "example.employees" 권한에 액세스 하는 예제임.
* 크라이언트 생성 후 user_ords_client 메타정보에서 등록된 클라이언트의 ID와 client_Secret 값 확인하는 쿼리를 수행함.
* "CLIENT_ID"와 "CLIENT_SECRET" 값은 OAtuth 클라이언트에 대한 시크릿 크리덴셜(비밀자격코드)임. 이 자격 코드는 비밀번호와 같이 안전하게 관리해야 함.

#### 클아이언트 생성

* 클라이언트 등록은 OAuth에서 제공하는 DB package를 사용 함.

```sql
ordstest 계정으로

begin
 oauth.create_client(
      p_name => 'Client_Credentials_Example',
      p_grant_type => 'client_credentials',
      p_privilege_names => 'example.employees',
      p_support_email => 'support@example.com');
 commit;
end;
/

-- exec oauth.delete_client('Client_Credentials_Example');

select client_id,client_secret from user_ords_clients where name = 'Client_Credentials_Example';

CLIENT_ID                        CLIENT_SECRET
-------------------------------- --------------------------------
Zm6Z-XMm_LTOrgGMLxtT7Q..         Cj6HfFG-syaUOXy-5NsbMQ..


```

#### 클라이언트에 롤 할당 및 확인

* "Client_Credentials_Example" 클라이언트에게 "HR_Administrator" 롤을 부여함.
* "user_ords_client_roles" 메타정보레서 클라이언트에 부여된 롤을 확인 할 수 있음.

```sql
ordstest 계정으로
begin
 oauth.grant_client_role(
     'Client_Credentials_Example',
     'HR_Administrator');
 commit;
end;
/

select * from user_ords_client_roles where client_name = 'Client_Credentials_Example';

 CLIENT_ID CLIENT_NAME                                 ROLE_ID ROLE_NAME
---------- ---------------------------------------- ---------- ----------------------------------------
     10093 Client_Credentials_Example                    10089 HR_Administrator

```

#### 액세스 토큰 얻기

* OAuth 프로토콜은 rfc6749-4.4 Client Credentials를 사용하여 토큰 엔드포인트 URL로 액세스 토큰을 얻는 방식임.
* cURL command
 + curl -i --user Client_ID:Client_Secret --data "grant_type=client_credentials" http://hostname/ords/"schema_name"/oauth/token
   - Cliend_ID, Client_Secret 값을 user_ords_clients 조회해서 얻은 값으로 대체

```shell
curl -i --user Zm6Z-XMm_LTOrgGMLxtT7Q..:Cj6HfFG-syaUOXy-5NsbMQ.. --data "grant_type=client_credentials" http://wokot450.kr.oracle.com:8080/ords/ordstest/oauth/token

HTTP/1.1 200 OK
Content-Type: application/json
X-ORDS_DEBUG: false
X-Frame-Options: SAMEORIGIN
Transfer-Encoding: chunked

{"access_token":"IVGVP-GrXzQJLUIBM4uN_w","token_type":"bearer","expires_in":36000}

```

#### 액세스 토큰으로 리소스 액세스 여부 확인

* cURL command
 + curl -i -H"Authorization: Bearer accessToken" https://example.com/ords/ordstest/examples/employee
   - "accessToken" 값은 위에서 얻은 access_token결과 값임(IVGVP-GrXzQJLUIBM4uN_w)

```shell
curl -i -H"Authorization: bearer IVGVP-GrXzQJLUIBM4uN_w" http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/

HTTP/1.1 200 OK
Content-Type: application/json
X-ORDS_DEBUG: false
ETag: "AISxJYbbwDrH0ImLXHnjO7jEOe1WpDQKlYfqD5fh9rdRHj8znUcz7V7lqWBz8x7ZD0NnOf3T3RCLtTM9xknmoQ=="
Transfer-Encoding: chunked

{"items":[{"empno":7934,"ename":"MILLER","job":"CLERK","mgr":7782,"hiredate":"1982-01-23T00:00:00Z","sal":1300,"comm":null,"deptno":10},{"empno":7902,"ename":"FORD","job":"ANALYST","mgr":7566,"hiredate":"1981-12-03T00:00:00Z","sal":3000,"comm":null,"deptno":20},{"empno":7900,"ename":"JAMES","job":"CLERK","mgr":7698,"hiredate":"1981-12-03T00:00:00Z","sal":950,"comm":null,"deptno":30},{"empno":7876,"ename":"ADAMS","job":"CLERK","mgr":7788,"hiredate":"1987-05-23T00:00:00Z","sal":1100,"comm":null,"deptno":20},{"empno":7844,"ename":"TURNER","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-08T00:00:00Z","sal":1500,"comm":0,"deptno":30},{"empno":7839,"ename":"KING","job":"PRESIDENT","mgr":null,"hiredate":"1981-11-17T00:00:00Z","sal":5000,"comm":null,"deptno":10},{"empno":7788,"ename":"SCOTT","job":"ANALYST","mgr":7566,"hiredate":"1987-04-19T00:00:00Z","sal":3000,"comm":null,"deptno":20}],"hasMore":true,"limit":7,"offset":0,"count":7,"links":[{"rel":"self","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/"},{"rel":"describedby","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/metadata-catalog/examples/employees/"},{"rel":"first","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/"},{"rel":"next","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/?offset=7"}]}

```

### Authorization code 자격 인증

* 이 인증 방식은 모바일 또는 데스크탑 클라이언트 인증 방식으로 유용함.

#### 클라이언트 등록

```sql
ordstest 계정으로

begin
  oauth.create_client(
        p_name => 'Authorization Code Example',
        p_grant_type => 'authorization_code',
        p_owner => 'Example Inc.',
        p_description => 'Sample for demonstrating Authorization Code Flow',
        p_redirect_uri => 'http://192.168.0.23:8080/auth/code/example/',
        p_support_email => 'support@example.org',
        p_support_uri => 'http://192.168.0.23:8080/support',
        p_privilege_names => 'example.employees'
        );
  commit;
end;
/

-- exec oauth.delete_client('Authorization Code Example');

PL/SQL procedure successfully completed.

select id, client_id, client_secret from user_ords_clients where name = 'Authorization Code Example';
I
        ID CLIENT_ID                        CLIENT_SECRET
---------- -------------------------------- --------------------------------
     10124 PKCerjw1BoBZ3kBk-BIOow..         qqyDQmriAjU7wotiw0ULoA..


```

#### OAuth authorization code 얻기

* cURL command
 + http://example.com/ords/ordstest/oauth/auth?response_type=code&client_id=cliendId&state=uniqueRando
   - "clientId"는 메타정보의 "CLIENT_ID" 값이며, state 값은 유니크한 값으로 입력

```http
http://192.168.0.23:8080/ords/ordstest/oauth/auth?response_type=code&client_id=PKCerjw1BoBZ3kBk-BIOow..&state=38au7fds99c110856546

401 Uauthorized

Access to this resource is protected. Please sign in to access this resource.

```
* "sign in" 링크 클릭
* id : hr_admin, password : 패스워드 입력
승인 화면 에서 "승인" 클릭 하면

웹브라우저에 다음과 같이 리다이렉션 URL이 나옴. 이 URL의 "code" 값과 state 값은 잘 보관 되어야 함

http://192.168.0.23:8080/auth/code/example/?code=R_agMqsPATf0sZqvbPg1Hg&state=38au7fds99c110856546

##### OAuth 액세스 토큰 얻기
* cURL command
 + curl --user clientId:clientSecret --data"grant_type=authorization_code&code=authorizationCode" https://example.com/ords/
ordstest/oauth/token
* 위 액세스 토큰 얻기 단계의 리다이이렉션 URL의 code 겂과 state값을 이용하여 액세스 토큰을 얻을 수 있음
 * "authorizationCode"를 code값으로 대체함

```shell
curl --user PKCerjw1BoBZ3kBk-BIOow..:qqyDQmriAjU7wotiw0ULoA.. --data "grant_type=authorization_code&code=R_agMqsPATf0sZqvbPg1Hg" http://192.168.0.23:8080/ords/ordstest/oauth/token

{"access_token":"7II7_mRBStje4Pb87RsZLQ","token_type":"bearer","expires_in":36000,"refresh_token":"abw8kXBX_6tBYnk8RcgE8g"}
```

#### 액세스 토큰으로 리소스 액세스 확인

* cURL command
 + curl -i -H"Authorization: Bearer accessToken" https://example.com/ords/ordstest/examples/employe
   - "accessToken"을 액세스 토큰 값으로 대체

```shell
curl -i -H"Authorization: Bearer 7II7_mRBStje4Pb87RsZLQ" http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/

HTTP/1.1 200 OK
Content-Type: application/json
X-ORDS_DEBUG: false
ETag: "xnytuXSDAtS6ooMHPiLhNDFVOjOKhh/95+TP+0wvBkYEObmycZxFaRyq183b9mdF5ip/wH5qgGqcqxrUJyvtyg=="
Transfer-Encoding: chunked

{"items":[{"empno":7934,"ename":"MILLER","job":"CLERK","mgr":7782,"hiredate":"1982-01-22T15:00:00Z","sal":1300,"comm":null,"deptno":10},{"empno":7902,"ename":"FORD","job":"ANALYST","mgr":7566,"hiredate":"1981-12-02T15:00:00Z","sal":3000,"comm":null,"deptno":20},{"empno":7900,"ename":"JAMES","job":"CLERK","mgr":7698,"hiredate":"1981-12-02T15:00:00Z","sal":950,"comm":null,"deptno":30},{"empno":7876,"ename":"ADAMS","job":"CLERK","mgr":7788,"hiredate":"1987-05-22T14:00:00Z","sal":1100,"comm":null,"deptno":20},{"empno":7844,"ename":"TURNER","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-07T15:00:00Z","sal":1500,"comm":0,"deptno":30},{"empno":7839,"ename":"KING","job":"PRESIDENT","mgr":null,"hiredate":"1981-11-16T15:00:00Z","sal":5000,"comm":null,"deptno":10},{"empno":7788,"ename":"SCOTT","job":"ANALYST","mgr":7566,"hiredate":"1987-04-18T15:00:00Z","sal":3000,"comm":null,"deptno":20}],"hasMore":true,"limit":7,"offset":0,"count":7,"links":[{"rel":"self","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/"},{"rel":"describedby","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/metadata-catalog/examples/employees/"},{"rel":"first","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/"},{"rel":"next","href":"http://wokot450.kr.oracle.com:8080/ords/ordstest/examples/employees/?offset=7"}]}
```

#### Refresh token을 사용한 세션 연장

* refresh token을 사용하여 새로 얻은 토큰 값을 사용하여 새로운 라이프타임을 가진 새 액세스 토큰을 만들수 있음

* cURL command
 + curl --user clientId:clientSecret --data “grant_type=refresh_token&refresh_token=refreshToken" https://example.com/ords/ordstest/oauth/token

```shell
curl --user PKCerjw1BoBZ3kBk-BIOow..:qqyDQmriAjU7wotiw0ULoA.. --data "grant_type=refresh_token&refresh_token=abw8kXBX_6tBYnk8RcgE8g" http://192.168.0.23:8080/ords/ordstest/oauth/token

{"access_token":"y-ML-L_183LW42qLomDZBw","token_type":"bearer","expires_in":36000,"refresh_token":"C0ma7_nktcgo4pdtJqEFDg"}

```

### Implicit flow 자격인증

#### 클라이언트 등록 및 확인

```sql
begin
 oauth.create_client(
    p_name => 'Implicit Example',
    p_grant_type => 'implicit',
    p_owner => 'Example Inc.',
    p_description => 'Sample for demonstrating Implicit Flow',
    p_redirect_uri => 'http://example.org/implicit/example/',
    p_support_email => 'support@example.org',
    p_support_uri => 'http://example.org/support',
    p_privilege_names => 'example.employees'
    );
 commit;
end;
/

-- exec oauth.delete_client('Implicit Example');

select id, client_id, client_secret from user_ords_clients where name = 'Implicit Example';

        ID CLIENT_ID                        CLIENT_SECRET
---------- -------------------------------- --------------------------------
     10156 arHcF55_4E_8fliKMJuNgQ..

```

#### 액세스 토큰 얻기

* URL command

 * https://example.com/ords/ordstest/oauth/auth?response_type=token&client_id=cliendId&state=uniqueRandomValue

```html
http://192.168.0.23:8080/ords/ordstest/oauth/auth?response_type=token&client_id=arHcF55_4E_8fliKMJuNgQ..&state=8fs3rfsa3004rewfsaf
```

#### 사용자 인증 및 액세스 확인

```html
http://192.168.0.23:8080/implicit/example/#token_type=bearer&access_token=LL9Oa8s1qV0Sg1rlRCkuKw&expires_in=36000&state=8fs3rfsa3004rewfsaf
```


```shell
curl -i -H "Authorization: Bearer LL9Oa8s1qV0Sg1rlRCkuKw" http://192.168.0.23:8080/ords/ordstest/examples/employees/

HTTP/1.1 200 OK
Content-Type: application/json
X-ORDS_DEBUG: false
ETag: "pwRtxkfQVmwqIGvk1IY8v9ZUGP1Yxk0dXJHkKWiPI+OmonMd8QjR8F5b5k3gykL7XO9lD25Z6QLNtgjEXe99NA=="
Transfer-Encoding: chunked

{"items":[{"empno":7934,"ename":"MILLER","job":"CLERK","mgr":7782,"hiredate":"1982-01-22T15:00:00Z","sal":1300,"comm":null,"deptno":10},{"empno":7902,"ename":"FORD","job":"ANALYST","mgr":7566,"hiredate":"1981-12-02T15:00:00Z","sal":3000,"comm":null,"deptno":20},{"empno":7900,"ename":"JAMES","job":"CLERK","mgr":7698,"hiredate":"1981-12-02T15:00:00Z","sal":950,"comm":null,"deptno":30},{"empno":7876,"ename":"ADAMS","job":"CLERK","mgr":7788,"hiredate":"1987-05-22T14:00:00Z","sal":1100,"comm":null,"deptno":20},{"empno":7844,"ename":"TURNER","job":"SALESMAN","mgr":7698,"hiredate":"1981-09-07T15:00:00Z","sal":1500,"comm":0,"deptno":30},{"empno":7839,"ename":"KING","job":"PRESIDENT","mgr":null,"hiredate":"1981-11-16T15:00:00Z","sal":5000,"comm":null,"deptno":10},{"empno":7788,"ename":"SCOTT","job":"ANALYST","mgr":7566,"hiredate":"1987-04-18T15:00:00Z","sal":3000,"comm":null,"deptno":20}],"hasMore":true,"limit":7,"offset":0,"count":7,"links":[{"rel":"self","href":"http://192.168.0.23:8080/ords/ordstest/examples/employees/"},{"rel":"describedby","href":"http://192.168.0.23:8080/ords/ordstest/metadata-catalog/examples/employees/"},{"rel":"first","href":"http://192.168.0.23:8080/ords/ordstest/examples/employees/"},{"rel":"next","href":"http://192.168.0.23:8080/ords/ordstest/examples/employees/?offset=7"}]}
```


> 참고
> ords 서비스 기동 쉘스크립트 예제
>
>  > startords shell 예제

```shell
vi startords
#!/bin/bash
LOGFILE=/home/woko/ords/logs/ords-`date +"%Y""%m""%d"`.log
export JAVA_OPTIONS="-Dorg.eclipse.jetty.server.Request.maxFormContentSize=3000000 -Duser.timezone=UTC"
nohup java ${JAVA_OPTIONS} -jar ords.war standalone >> $LOGFILE 2>&1 &
echo "if you want to View, using tail -f $LOGFILE"
```
>  > stopords shell 예제

```
vi stopords
#!/bin/bash
kill `ps -ef | grep ords.war | awk '{print $2}'`
```
>

Good job~~~ 수고하셨습니다.
