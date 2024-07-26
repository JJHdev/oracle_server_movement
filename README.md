오라클 서버를 이전하면서 발생했던 오류들 기록

1. expdp >>  --alldabase로 추출시 기존에 사람들이 사용했던 모든 데이터들이 엮어서 자꾸 에러발생함. (내가 사용하는 chemas만 옮기길 추천함
2. expdp
     해당 내용은 sqlplus에서 sys as sysdba 계정으로 실행할 것
   설명 : 순서로는 오라클에서 dmp 추출시 경로를 오라클에 지정해서 사용해야한다.
    -. Create or Replace directory 경로변수명 AS '실제경로';
   설명 : 기존에 사용하던 유저명으로 사용할것이다. (사용안할거면 새로 만들것)
    -. create user 유저명 identified by 비밀번호;
   설명 : 기존에 사용하던 유저명 or 새로만든 유저로 권한을 부여할것이다.
    -. grant exp_full_database to 유저명;
    -. grant read, write on directory 경로변수명 to 유저명;

   설명: sqlplus에서 나온후 (ip주소는 원격이 아니면 생략가능, 경로변수가 실제로 있어야한다., orcl, xe는 오라클에서 찾을수 있음.)
   expdp sys/비밀번호@IP주소:orcl schemas=스키마명 directory=경로변수명 dumpfile=덤프파일명.dmp
   expdp sys/비밀번호@IP주소:xe schemas=스키마명 directory=경로변수명 dumpfile=덤프파일명.dmp

3. impdp
   해당 내용은 sqlplus에서 sys as sysdba 계정으로 실행할 것 (그리고 새로운 리눅스or 윈도우에서 실행한다 가정)
   설명: 경로변수명 설정
    -. create or replace directory 경로변수명 as '실제경로';
   설명 : 유저생성 (오라클설치하고 실제로 계정이 생성되는지 확인할것, 기존에 사용하던 유저명을 사용하는게 좋음.)
    -. create user 유저명 identified by 비밀번호;
   설명 : 권한 부여
    -. grant imp_full_database to 유저명;
    -. grant read, write on directory 경로변수명 to 유저명;

   설명 : sqlplus에서 나와서 실행할것
     -. impdp 유저명/비밀번호@orcl schemas=추출시사용한스키마명 directory=경로변수명 dumpfile=경로변수위치에추출한dmp파일명.dmp


4. error
  -. 덤프값, 인수값 (ora-39001) 경로설정이 잘못되었던가, 권한이 없던가, 파일이없던가 문제임 -> 실제로 전부 문제없었는데 계정, 경로변수, 파일을 C드라이브최상단 에 실행했더니 해결됨
   
  -. ora-39172, ora-01653 테이블스페이스에 용량이 부족해서 늘려야함.  아래내용대로 실행하였고, 에러로그에 어느 테이블스페이스에서 부족한지 상세히 알려준다.
     그리고 못찾을 경우 기존에 생성된 테이블이름으로 찾을수 있음
----------------------------- 테이블 이름으로 담당 테이블스페이스 찾기 쿼리문 ----------------------------
SELECT 
    segment_name AS table_name,
    tablespace_name
FROM 
    dba_segments
WHERE 
    segment_type = 'TABLE'
    AND segment_name like '%테이블명%';

    
----------------------------- 테이블스페이스 용량 확인및 증설 ----------------------------
    -. SELECT * FROM DBA_TABLESPACES;
    
  SELECT * FROM dba_users;
  
  SELECT value AS db_block_size
  FROM   v$parameter
  WHERE  name = 'db_block_size';
  
  SELECT file_name,
         tablespace_name,
         bytes / 1024 / 1024 AS size_mb,
         maxbytes / 1024 / 1024 AS max_size_mb
  FROM   dba_data_files
  WHERE  tablespace_name = 'USERS';
  
  SELECT 4194303 * 8192 *2 / 1024 / 1024 AS max_size_mb FROM dual;
  
  
  SELECT tablespace_name, file_name, bytes/1024/1024 AS size_mb, autoextensible
  FROM dba_data_files
  WHERE tablespace_name = 'USERS';
  
  ALTER TABLESPACE users
  ADD DATAFILE 'C:\TOOLS\ORADATA\ORCL\USERS04.DBF' SIZE 20G;
  
  ALTER DATABASE DATAFILE 'C:\TOOLS\ORADATA\ORCL\USERS04.DBF' AUTOEXTEND ON NEXT 1G MAXSIZE UNLIMITED;
