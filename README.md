#Upgrade the GitLab Chart (from 6.11.x to 7.x.x)
------------
> GitLab Helm Chart 7.0.0부터, 번들된 PostgreSQL 버전이 12.7.0에서 14.8.0으로 업그레이드됨에 따라,
> 7.0.0버전 이후로는 PostgreSQL 12.7.0은 기본적으로 지원되지 않습니다.<br/>
> PostgreSQL 데이터베이스를 업그레이드하기 위해서는 기존 데이터베이스를 백업한 후 새로운 데이터베이스로 복원하는 절차를 따라야 합니다.<br/>
> 이 문서는 해당 과정을, GitLab 공식 문서를 참고하여 작성하였습니다.<br/>
> https://docs.gitlab.com/charts/installation/database_upgrade.html
<br/>

#### 1. 데이터베이스 백업
---
- PostgreSQL 데이터베이스의 내용을 덤프하여 백업하고, 덤프된 데이터베이스 파일을 tar 형식으로 압축하여 MinIO의 gitlab-backups 버킷에 저장합니다.
- 해당 명령어 실행 후 "413 Request Entity Too Large" 에러가 발생한다면<br/>
nginx-ingress-controller deployment의 annotation에 nginx.ingress.kubernetes.io/proxy-body-size: 64m 를 추가
```
GITLAB_RELEASE=v6.11.12
curl -s "https://gitlab.com/gitlab-org/charts/gitlab/-/raw/${GITLAB_RELEASE}/scripts/database-upgrade" | bash -s pre
```
