# RealMySql

# MySQL
## 왜 MySQL 인가 ?
>MySQL 과 오라클을 비교해 본다면 MySQL 의 경쟁력은 가격이다.

## 어떤 DBMS 를 사용해야 좋은가 ?
>1.안정성

>2.성능과 기능

>3.커뮤니티나 인지도

를 고려하여 자신이 가장 잘 사용할수 있는것을 선택

## 설치

# MySQL 설치방법

1.리눅스 서버의 Yum 인스톨러로 설치
- rpm 파일 다운로드
> 1.Yum 인스톨러를 이용하려면 MySQL 소프트웨어 리포지토리를 등록해야 하는데 https://dev.mysql.com/downloads/repo/yum/ 에서 각 운영체제의 버전에 맞는 rpm 파일을 다운로드
- sudo rpm -Uvh {다운로드한 레포}
- 다운로드된 레포 확인
> ls-alh /etc/yum.repos.d/*mysql*
- Yum 인스톨러 명령어를 이용해 설치 가능한 MySQL 목록 확인
> sudo yum search mysql-community
- MySQL 설치명령어 입력
> ex)sudo yum install mysql-community-server-8.0.21


2.리눅스 서버에서 Yum 인스톨러 없이 rpm 파일로 설치
- RPM 패키지로 직접 설치하려면 설치에 필요한 RPM 파일들을 직접 다운로드 해야한다
> https://dev.mysql.com/downloads/mysql/ 에서 원하는 버전 선택후 직접 다운로드
> <br>Development Libraries<br>
Shared Libraries<br>
Compatibility Libraries<br>
MySQL Configuration<br>
MySQL server<br>
Client Utilites<br>
- 의존관계 순으로 설치한다.

3. DMG 패키지로 설치
- (macOS 기준) macOS 인스톨러로 설치하려면 설치에 필요한 DMG 패키지 파일들을 직접 다운로드 해야한다.
>https://dev.mysql.com/downloads/mysql/ 에서 운영체제 선택후 DMG 패키지 파일을 다운로드.
- 다운로드한 DMG파일을 실행하여 설치를 진행
> 설치를 진행하다 보면 사용자 인증방식을 선택하게 된다.
1. Use Strong Password Encryption(인터넷을 경유해서 MySQL 서버를 접속한다면 Use Strong Password Encryption 선택)
2. Use Legacy Password Encryption (MySQL 서버를 사설 네트워크 에서만 사용한다면 Use Legacy Password Encryption 선택)
- Configuration 에서 비밀번호 설정.
> MySQL 서버를 설치할때 /usr/local/mysql 하위에 데이터 디렉토리 , 로그 파일들이 생성되고 관리자 모드로 실행하기 때문에 관리자 비밀번호를 설정해야 한다.

### MySQL 실행
(설치가 완료되면 자동으로 실행된다.)
> ps -ef | grep mysqld 명령어를 실행시켜 보면 파일들이 /usr/local/mysql 하위 디렉토리에 설치되어있고 MySQL 이 실행되어진걸 확인할수 있다.


1. MySQL 서버 시작
> sudo /user/local/mysql/support-files/mysql.server start

2. MySQL 서버 종료
> sudo /user/local/mysql/support-files/mysql.server stop

## MySQL 시스템 변수
> MySQL 서버는 접속된 사용자를 제어하기 위해 어떤 값들을 저장하고 있는데 이 값들을 시스템 변수 라고 한다.
> 시스템 변수를 확인하기 위한 명령어 : SHOW GLOBAL VARIABLES;

### 스시템 변수의 5가지 속성과 의미
1. Cmd-Lone : MySQL 서버의 명령행 인자로 설정될수 있는지 여부
2. Option file : MySQL 의 설정 파일인 my.cnf 로 제어할수 있는지 여부
3. System Var : 시스템 변수인지 아닌지를 나타냄.
4. Var Scorp : 시스템 변수의 적용 범위
5. Dinamic : 시스템 변수가 동적인지 정적인지 구분하는 변수.


### 글로벌 변수와 세션 변수

- 글로벌 변수
> 글로벌 범위의 시스템 변수는 하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수를 의미하며 주로 SQL 서버 자체에 관한 설정이다.

- 세션변수
> 세션 범위의 시스템 변수는 MySQL 클라이언트가 MySQL 서버에 접속할때 부여하는 옵션의 기본값을 제어하는데 사용된다.



# 3.1 사용자 식별

>'mong'@'127.0.0.1' 

MySQL 의 사용자는 사용자의 접속지점 도 계정의 일부이다.
따라서 계정을 언급할 때는 아이디와 호스트를 함께 명시해야 한다.

>'mong'@'%'

MySQL 서버가 기동중인 로컬 호스트에서 mong 이라는 아이디로 접속할 때만 사용될수 있는 계정이다.만약 사용자 계정에 다음과 같이 등록되어 있다면 다른 컴퓨터에서는 접속이 불가능하다.
만약 모든 외부 컴퓨터에서 접속이 가능하게 하려면 호스트 부분을 % 로 대체한다.

> 'mong'@'127.0.0.1'  ,  'mong'@'%'

동일한 아이디가 있을경우 MySQL 서버가 해당 사용자의 인증을 위해 어떤 계정을 선택할까 ?

> MySQL 은 범위가 가장 작은것을 우선적으로 선택한다. 위의 두 아이디중 범위가 작은것은 127.0.0.1 로컬 이기 때문에 이 아이를 먼저 선택한다.

# 3.2 사용자 계정 관리

## 3.2.1 시스템 계정과 일반 계정

> SYSTEM_USER 권한을 가지고 있느냐 없느냐에 따라 시스템 계정과 일반계정 으로 구분한다.

### 시스템 계정
> - 계정관리
- 다른 세션 또는 그 세션에서 실행중인 쿼리를 강제 종료
- 스토어드 프로그램 생성시 DEFINER 를 타 사용자로 설정

### MySQL 내장 계정
> MySQL 서버에는 내장된 계정들이 있는데 내부적으로 각기 다른 목적으로 사용되므로 삭제하지 않도록 주의한다.
1. 'mysql.sys'@'localhost' : MySQL 8.0 부터 내장된 sys 스키마의 객체들의 DEFINER 로 사용되는 계정
2. 'mysql.session'@'localhost' : MySQL 플로그인이 서버로 접근할때 사용되는 계정
3. 'mysql.infoschema'@'localhost' : information_schema 에 정의된 뷰의 DEFINER 로 사용되는 계정
위 세 계정은 처음부터 잠겨있는 상태이므로 의도적으로 풀지 않는한 보안걱정은 하지 않아도 된다.

## 3.2.2 계정 생성
>5.7 버전까지는 grant 명령으로 권한의 부여와 동시에 계정 생성이 가능했지만 8.0 부터 계정 생성은 create user , 권한 부여는 grant 로 구분해서 실행하도록 변경되었다.
계정을 생성할때 다음 옵션들을 설정할수 있다.
- 계정의 인증방식과 비밀번호
- 비밀번호 관련 옵션
- 기본 역할
- SSL 옵션
- 계정 잠금 여부

> create user 'user'@'%'
	identified with 'mysql_native_password by 'password'
    require none
    password expire interval 30 day
    account unlock
    password history defualt
    password reuse interval defualt
    password require current defualt;
    
### 1. idntified with

>사용자의 인증 방식과 비밀번호를 설정한다.
identified with 뒤에는 반드시 인증방식을 명시해야 하는데 기본 인증방식을 사용하려면 'password'

- Native Pluggable Authentication : 비밀번호에 대한 해시 값을 저장해두고 클라이언트가 보낸 해시값과 일치하는지 판별

- Caching SHA-2 Pluggable Authentication : 암호화 해시값 생성을 위해 SHA-2 알고리즘을 사용한다. 해시값을 구하는데 많은 시간이 소모되어 해시 결과값을 메모리에 캐시해서 사용한다. 이 인증방식을 사용하려면 SSL/TLS/RSA키페어 를 반드시 사용해야 하는데 이를 위해 클라이언트에 접속할때 SSL 옵션을 활성화해야한다.

- PAM Pluggable Authentication : 유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할수 있게 해주는 인증방식으로 엔터프라이즈 에디션에서만 사용가능

- LDAP Pluggable Authentication : LDAP 를 이용한 외부인증을 사용할수 있게 해주는 인증방식으로 엔터프라이즈 에디션에서만 사용가능

> Native Authentication 이 기본 인증방식으로 사용됐었지만 Caching SHA-2 방식으로 변경되었다.
이방식은 SSL/TLS/RSA키페어 가 필요로 하기 때문에 Native Authentication 을 사용하고자 한다면 MySQL 설정을 변경하자

> set global default_authentication_plugin="mysql_native_password"


### 2. REQUIRE
>MySQL 서버에 접속할때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다.
만약 설정하지 않을경우 비암호화 채널로 연결한다.

### 3. PASSWORD EXPIRE
> 비밀번호의 유효기간을 설정하는 옵션이며 별도로 설정하지 않을경우 default_password_lifetime 시스템 변수에 저장된 값으로 설정된다.
개발자나 데이터 베이스 관리자의 비밀번호는 유효기간을 설정하는것이 보안상 안전하지만 응용 프로그램 접속용 계정에 유효기간을 설정하는 것은 위험할수 있으니 주의하자.

- password expire : 계정 생성과 동시에 비밀번호의 만료 처리
- password expire naver : 계정 비밀번호의 만료기간 없음
- password expire default : default_password_lifetime시스템 변수에 저장된 기간으로 설정
- password expire interval n day : 비밀번호의 유효기간을 오늘부터 n 일자로 설정

### 4. PASSWORD HISTORY
> 한번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션
password history default : 한번 사용한 비밀번호 재사용 불가
password history n : 비밀번호 이력을 최근 n 개 까지만 저장하여 n개만 재사용 불가

### 5. PASSWORD REUSE INTERVAL
> 한번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션으로 설정하지 않으면 password_reuse_interval 시스템 변수의 기간으로 설정된다.
password reuse interval default : password_reuse_interval 변수사용
password reuse interval n day : n일후 비밀번호 재사용 가능

### 6. PASSWORD REQUIRE
> 비밀번호가 만료되어 새로운 비밀번호로 변경할때 현재 비밀번호를 필요로 할지 결정하는 옵션으로 설정하지 않으면 password_require_current 시스템 변수값으로 설정된다.
password require current : 비밀번호를 변경할때 현재 비밀번호 입력
password require optional : 비밀번호를 변경할때 현재 비밀번호 입력하지 않음
password require default : 시스템 변수값으로 설정

### 7. ACCOUNT LOCK/UNLOCK
> 계정 생성시 ,정보 변경시 계정을 사용하지 못하게 잠글지 여부를 결정
ACCOUNT LOCK : 계정을 사용하지 못하게 잠금
ACCOUNT UNLOCK : 잠긴 계정을 다시 사용가능 상태로 만듬

# 비밀번호 관리

## 3.3.1 고수준 비밀번호
> MySQL 서버의 비밀번호는 유효기간이나 이력관리를 통한 관리뿐만 아니라 비밀번호를 쉽게 유추할수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정할수 있다.
비밀번호 유효성 체크 규칙을 적용하려면 validate_password 컴포넌트를 이용하면 된다.

### validate_password 컴포넌트 설치
> install component 'file://conponent_validate_password';

### 설치된 컴포넌트 확인
> select * from mysql.component;

### 비밀번호 정책

> LOW : 비밀번호 길이만 검증
MIDIUM : 비밀번호의 길이를 검증하여 숫자,대소문자 , 특수문자 배합을 검증
STRONG : MIDIUM + 금칙어 포함여부 검증

#### 금칙어 파일 등록
> set global validate_password.dictionary_file='{fileName}';
set global validate_password.policy='STRONG';

### 3.3.2 이중 비밀번호
> 일반적으로 많은 응용프로그램 서버들이 공용으로 데이터베이스를 사용하기 때문에 서버의 계정 정보는 응용 프로그램 서버로부터 공용으로 사용되는 경우가 많다.
이런 구현 특징 때문에 데이터베이스 서버의 계정 정보는 쉽게 변경하기 어려웠는데 데이터베이스 계정의 비밀번호는 서비스가 실행중인 상태로 변경이 불가능했다.
이를 해결하기 위해 8버전 이후 계정의 비밀번호로 2개의 값을 동시에 사용하는 기능을 추가했다.
이중 비밀번호는 Primary , Secondary 로 구분된다.
이중 비밀번호를 사용하려면 기존 비밀번호 변경 구문에 retain current password 옵션을 추가하면 된다.

> alter user 'root@'localhost' identified by 'old_password';
alter user 'root@'localhost' identified by 'new_password' retain current password;

> 첫 명령어가 실행되면 root 계정의 Primary 는 old_password 가 되고 세컨더리는 비어있다.
두번째 명령어가 실행되면 old_password 는 Secondary 가 되고 new_password 가 Primary 가 되며 두개중 어느것을 사용해도 로그인 된다.

>alter user 'root'@'localhost' discard old password;
MySQL 서버에 접속하는 모든 응용 프로그램이 재시작 완료되면 위의 명령어로 Secondary 비밀번호는 보안을 위해 삭제하는것이 좋다.

# 권한
>권한은 글로벌 권한과 객체단위의 권한으로 구분된다.
객체권한 : grant 명령어로 권한을 부여할때 반드시 특정 객체를 명시해야 한다.
글로벌 : grant 명령어로 권한을 부여할때 특정 객체를 명시하지 말아야 한다.
