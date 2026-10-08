# 온라인 수강생 설문조사 웹사이트
IT과정을 수강하는 학생들이 수강을 하면서 어떤 점이 필요한지 등을 파악해서  
좀 더 개선된 학습환경을 제공하기 위해서 진행되는 설문입니다.

## 3차 버전

FastAPI + HTML/CSS/Vanilla JavaScript + MariaDB(SQLAlchemy/PyMySQL) 프로젝트입니다.
설문 1~10번, DB 기반 선택 목록, 관리자 응답 검색·상세·요약, 전체/팀별 통계,
상담 확인 표시 및 환경·시설 분석을 제공합니다.

### Google Cloud Compute Engine 실행

이 저장소의 최상위가 앱 디렉터리입니다. MariaDB 11.4는 VM에 별도로 설치합니다.
운영 DB는 이전에 준비한 비공개 SQL 백업을 빈 DB에 복원합니다.
실제 응답 백업과 개발용 비밀번호는 이 저장소에 포함하지 않습니다.
`database.sql`은 데이터 없이 새로 시작하는 경우의 스키마·기본 항목 준비용입니다.
이미 운영 데이터를 복원했다면 초기화 스크립트를 다시 실행하지 마세요.

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
cp .env.example .env
# .env에 운영 DB 계정과 비밀번호, 별도 관리자 비밀번호를 설정합니다.
chmod 600 .env
.venv/bin/uvicorn main:app --host 127.0.0.1 --port 8000
```

Nginx와 HTTPS를 통해 접속하고 systemd로 재시작/자동 실행을 구성합니다.
설문 화면은 `/`, 관리자 화면은 `/admin`입니다.
GET `/api/surveys`, `/api/surveys/{id}`, `/api/admin/*`는 관리자 인증이 필요합니다.
POST `/api/surveys`는 기존처럼 설문 저장에 사용합니다.
DB 계정은 앱용 SELECT/INSERT 권한만 부여하고 3306 포트는 공개하지 않습니다.

### Docker 실행 옵션

Dockerfile은 HTTP 포트 환경변수 `PORT`를 지원하며 기본 포트는 8080입니다.
환경변수로 DB 및 관리자 접속 설정을 전달합니다.
컨테이너의 `127.0.0.1`은 VM의 MariaDB가 아니므로 VM에서 host networking을
사용하거나 컨테이너에서 접근 가능한 비공개 DB 주소를 `DB_HOST`로 설정합니다.
MariaDB와 웹 앱은 별도의 실행 프로세스입니다.

### 분석 기준 및 검증

반복 제출을 각각 응답 1건으로 집계합니다. 복수 선택 비율의 분모는 응답 수입니다.
환경·시설 평균은 1~5점 응답만 포함합니다. 미이용/제공 없음/해당 없음과 미응답은 구분합니다.
상담 확인은 추가 상담을 위한 참고 표시이며 학생 순위나 확정 평가가 아닙니다.

```bash
python -m unittest test_analysis -v
```
