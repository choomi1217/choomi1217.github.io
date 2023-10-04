---
title:  "[오라클 성능 고도화 원리와 해법] ch1.SQL 분석 도구"

categories:
  - db-performance-advancement
tags:
  - [db-performance-advancement, oracle]

toc: true
toc_sticky: true

breadcrumbs: true

date: 2023-10-03
last_modified_at: 2023-10-04
---


# SQL 분석 도구

- Explain Plan
- Auto Trace
- SQL Trace
- Event Trace

### Explain Plan
- `Explain plan set statement_id ='query1' for select * from emp where empno = 7900;` 
- `SELECT * FROM plan_table WHERE statement_id = 'query1';`
- `SELECT plan_table_output FROM TABLE(DBMS_XPLAN.DISPLAY('plan_table','query1'));`

### Auto Trace
- `set autotrace on`
  - `traceonly` : 실행계획, 통계정보 출력
  - `explain` : 쿼리 수행결과, 실행계획 출력
  - `statistics` : 쿼리 수행결과, 통계정보 출력
- `select * from emp where empno = 7900;`

### SQL Trace
1. 트레이스 수집
- `alter session set sql_trace = true;`
- `select * from emp where empno = 7900;`
- `alter session set sql_trace = false;`
2. 트레이스 파일 찾기
3. 리포트 파일 생성
4. 리포트 파일 분석
