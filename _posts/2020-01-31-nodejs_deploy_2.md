---
title: 'Nodejs 배포하기 2편 - Pm2 Deploy'
last_modified_at: 2020-01-31
categories: ['nodejs']
tags: ['nodejs', 'jenkins', 'git', 'pm2']
published : true
---

## Step0. 참고사이트
- [NEMBV 21 운영과 배포 방법](https://fkkmemi.github.io/nembv/nembv-21-deploy-web/){:target="_blank"}

## Step1. ecosystem.config.js 생성
- [Pm2 Deploy docs](https://pm2.keymetrics.io/docs/usage/deployment/){:target="_blank"}
    ```javascript
    module.exports = {
        apps: [
            {
                name: 'hook',
                script: 'server/hook.js',
                instances: '1',
                exec_mode: 'cluster',
                log_date_format: 'YYYY-MM-DD HH:mm:ss',
                wait_ready: true,
                listen_timeout: 3000
            }
        ],
        deploy: {
            production: {
                user: 'infomax',
                host: 'localhost',  // ['server1', 'server2'] // 운영서버 리스트
                ref: 'origin/master',
                repo: 'git@bitbucket.org:TEAM/PROJECT.git', // git 주소
                path: '/home/user/XXXXXXXX',    // 프로젝트 폴더 경로
                'post-deploy': 'yarn --cwd ./server'    // 빌드시 실행 
            }
        }
    };
    ```

## Step2. host가 다른 서버면?
- SSH 프라이빗 키 방식으로 하거나 비밀번호 없이 접근 가능하도록 설정
- 원격 하려는 서버에서 작업
    ```shell
    ls ~/.ssh/id_rsa.pub  # 파일이 존재하는지 체크

    # 없으면 
    # ssh-keygen -t rsa  # 키 생성

    ssh-copy-id -i ~/.ssh/id_rsa.pub [user]@[host] # 해당서버에 등록~
    ```


## Step3. 첫 셋팅하기
- 실행 후 해당 경로에 잘복사 되었는지 확인
    ``` shell
    pm2 deploy ecosystem.config.js production setup
    ```

- jenkins를 이용중이라면 첫 Build시 Execute shell에서 실행
    ![{{"/assets/images/nodejs_deploy/4.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/4.png" | absolute_url}})

## Step4. 배포하기
```
# 서버 리로드시 
pm2 deploy ecosystem.config.js production exec "pm2 startOrReload ecosystem.config.js"

# 서버 리로드 필요 없을시 
pm2 deploy ecosystem.config.js production 

# 롤백 할 때
pm2 deploy ecosystem.config.js production ref $ROLLBACK_TAGS exec "pm2 startOrReload ecosystem.config.js"
```