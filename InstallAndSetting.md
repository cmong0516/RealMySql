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
세션 범위의 시스템 변수중 MySQL 서버의 설정파일에 명시해 초기화 할수 있는 변수는 대부분 범위가 'Both' 라고 명시되어 있는데 MySQL 서버가 기억하고 있다가 클라이언트와의 커넥션이 생성되는 순간 해당 커넥션의 기본값으로 사용한다.


### 정적 변수와 동적 변수
>MySQL 서버의 시스템 변수는 서버가 기동중인 상태에서 변경 가능한지에 따라 동적변수와 정적 변수로 구분된다.
1. 정적변수 : 디스크에 저장되어 있는 설정파일을 변경하는 경우
2. 동적변수 : 이미 기동중인 서버의 메모리에 있는 MySQL 서버의 시스템 변수를 변경하는 경우

- 디스크에 저장되어 있는 설정파일을 변경했을 경우 MySQL 서버를 재시작 해야 적용된다.
- 이미 기동중인 시스템 변수를 변경하는 경우 현재 기동중인 인스턴스에서만 유효하다. 영구히 적용하길 원한다면 my.cnf 파일을 변경하여 재시작 해도 그 설정이 유지되게 해야한다.
- set 명령어로 변경할수 있는건 동적변수이다.

### SET PERSIST

> 동적 변수는 set global 명령으로 변경하여 즉시 서버에 적용할수 있다.
하지만 이렇게 저장한 시스템 변수가 재시작 하고나면 적용되어 있지 않는다.
동적 변수를 MySQL 의 설정파일에 적용시키기 위한 명령어가 MySQL 8 버전에서 추가되었는데 set persist 이다.

- set persist 명령어는 세션 변수에는 적용되지 않는다
- set persist 명령은 자동으로 global 시스템 변수의 변경으로 인식한다.
- 현재 서버에는 변경된 내용을 저장하지 않고 다음 재시작부터 적용하길 원한다면 set persist_only 명령어를 사용한다.
- set persist , set persist_only 명령어를 사용하면 json 포맷의 mysqld-auto.cnf 파일이 생성되며 시스템 변수의 이름,설정값 , 수정자,수정일 등이 기록된다.
- set persist, set persist_only 명령으로 추가된 시스템 변수를 삭제할 경우 mysqld-auto.cnf 파일을 직접 변경하다가 MySQL 서버가 시작되지 못할수도 있는데 안전하게 reset persist 명령어를 사용하자.
