# GitLab 차트 PostgreSQL 업그레이드 가이드 (from 6.11.x to 7.0.0)
> GitLab Helm Chart 7.0.0부터, 번들된 PostgreSQL 버전이 12.7.0에서 14.8.0으로 업그레이드됨에 따라,
> 7.0.0버전 이후로는 PostgreSQL 12.7.0은</br>
> 기본적으로 지원되지 않습니다.<br/>
> PostgreSQL 데이터베이스를 업그레이드하기 위해서는 기존 데이터베이스를 백업한 후 새로운 데이터베이스로 복원하는 절차를 따라야 합니다.<br/>
> 해당 과정을, GitLab 공식 문서를 참고하여 작성하였습니다.<br/>
> https://docs.gitlab.com/charts/installation/database_upgrade.html

#### 1. 데이터베이스 백업
```
GITLAB_RELEASE=v6.11.12
curl -s "https://gitlab.com/gitlab-org/charts/gitlab/-/raw/${GITLAB_RELEASE}/scripts/database-upgrade" | bash -s pre
```
- 위 명령어는 다음의 작업을 수행합니다.
    - PostgreSQL 데이터베이스의 내용을 덤프하여 백업하고, 덤프된 데이터베이스 파일을 tar 형식으로 압축하여 MinIO의 gitlab-backups 버킷에 저장
 - 위 명령어 실행 후 "413 Request Entity Too Large" 에러가 발생한다면
    - nginx-ingress-controller deployment의 annotation에 nginx.ingress.kubernetes.io/proxy-body-size: 64m 를 추가
<br/>

기존의 PostgreSQL 데이터 삭제
```
kubectl delete statefulset RELEASE-NAME-postgresql
kubectl delete pvc data-RELEASE_NAME-postgresql-0
```
<br/>

#### 2. GitLab 업그레이드
- GitLab Chart 버전 7.3.5로 업그레이드, 업그레이드 시 --set gitlab.migrations.enabled=false 옵션 추가 
    - 업그레이드 후 sidekiq, webservice 파드가 정상작동하지 않아도 무시
    - toolbox 파드에 대한 업그레이드가 완료되었고, 정상작동하는지 확인
```
kubectl rollout status -w deployment/RELEASE_NAME-toolbox
```
<br/>

#### 3. 데이터베이스 복원
```
GITLAB_RELEASE=v6.11.12
curl -s "https://gitlab.com/gitlab-org/charts/gitlab/-/raw/${GITLAB_RELEASE}/scripts/database-upgrade" | bash -s post
```
- 위 명령어는 다음의 작업을 수행합니다.
    - webservice, sidekiq, gitlab-exporter deployments 의 replicas를 0으로 scale down
    - 백업했던 데이터베이스 복원
    - 데이터베이스 마이그레이션
    - scale down 했던 deployments 정상화
<br/>

- 이전의 과정에서 sidekiq, webservice 파드가 정상작동하지 않았다면
    - --set gitlab.migrations.enabled=false 옵션 제거 후 동일 버전(7.3.5) 업그레이드
    - 정상화되었는지 확인
<br/>

> - 7.3.5 이후 추천하는 업그레이드 경로
>     - 7.7.7, 7.11.4, 8.0.2
