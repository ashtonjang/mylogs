---
title: 'Nodejs 배포하기 1편 - Jenkins, Git 설정하기'
last_modified_at: 2020-01-30
categories: ['nodejs']
tags: ['nodejs', 'jenkins', 'git', 'pm2']
published : true
---

## Step0. 참고사이트
- [SpringBoot2로 Rest api 만들기(13) – Jenkins 배포(Deploy) + Git Tag Rollback](https://daddyprogrammer.org/post/2697/springboot2-jenkins-deploy-gittag-rollback/){:target="_blank"}

## Step1. Jenkins → 새로운 Item
- Enter an item name : 프로젝트 네임
- Freesttyle project 선택

![{{"/assets/images/nodejs_deploy/1.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/1.png" | absolute_url}})


## Step2. 프로젝트 구성
### General → 이 빌드는 매개변수가 있습니다.
- Git Parameter 추가
    - Name : **ROLLBACK_TAGS**
        - 아래 소스코드 관리에서 Git 추가할 때 파라미터 이름이랑 동일해야 함
    - Description :  롤백인경우 TAG선택, 선택하지 않으면 master 브랜치 최신 리비전으로 배포
    - Parameter Type : **Tag**
    - Default Value : **origin/master**
        - 입력값 없으면 최신 리비전으로 배포하기 위해 설정
- String Parameter (안해도 됨)
    - 매개변수 명 : TAG_ID
    - Default Value : Notitle
    - 설명 : TAG를 생성할때 추가로 ID를 붙일때 사용합니다.

![{{"/assets/images/nodejs_deploy/2.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/2.png" | absolute_url}})


## Step3. 소스 코드 관리
### Git
- Repositories
    - Repository URL : git 주소를 입력
    - Credentials : git에 인증 정보를 입력
- Branches to build
    - Branch Specifier (blank for 'any') : ${ROLLBACK_TAGS}
        - 위에 추가 했던 **Git Parameter Name 입력**

![{{"/assets/images/nodejs_deploy/3.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/3.png" | absolute_url}})


## Step4. 빌드 유발
- Bitbucket 에서 Webhook을 발생 시켜서 배포 시 추가


## Step5. Build
### Execute shell -> Command
- Nodejs 를 빌드하기 위한 Shell Script...
- pm2 deploy를 이용

```shell
echo $BUILD_TYPE
echo $ROLLBACK_TAGS

# 첫 빌드환경 셋팅임으로 첫 한번만 실행해야 함
#pm2 deploy ecosystem.config.js production setup

# 빌드하기 서버 폴더가 있을 경우 pm2 재시작. 아니면 pm2에 server 폴더 watch 걸면 됨.
lists=`git diff --name-only HEAD~`
if [[ $lists == *"server/"* ]]; then
    pm2 deploy ecosystem.config.js production ref $ROLLBACK_TAGS exec "pm2 startOrReload ecosystem.config.js"
else
    pm2 deploy ecosystem.config.js production ref $ROLLBACK_TAGS
fi
```

![{{"/assets/images/nodejs_deploy/4.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/4.png" | absolute_url}})

## Step6. 빌드 후 조치
### Git Publisher
- Push Only If Build Succeeds : check.
- Tags :
    - Tag to push : Release-$BUILD_ID-$TAG_ID
    - Tag message :
        - TAG INFO
        BuildNo - ${BUILD_NUMBER}
        BuildTag - ${BUILD_TAG}
    - Create new tag : check
    - Target remote name : origin

![{{"/assets/images/nodejs_deploy/5.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/5.png" | absolute_url}})

### Slack Notifications
- 성공 후 슬랙 발송을 위해.

![{{"/assets/images/nodejs_deploy/6.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/6.png" | absolute_url}})


## Step7. Test
- 좌측 메뉴 Build with Parameters
- Pm2 설정까지 하고나서 실행
- 빌드하고 롤백 테스트

![{{"/assets/images/nodejs_deploy/7.png" | absolute_url}}]({{"/assets/images/nodejs_deploy/7.png" | absolute_url}})