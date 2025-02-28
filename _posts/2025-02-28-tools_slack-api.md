---
title: slack api를 사용하여 slack으로 메세지 보내기
date: 2025-02-28 10:24:32 +0900
categories: [tools]
tags: [slack, webhook]
---

> slack api를 사용하여 benchmark 등 프로그램의 실행 결과를 slack으로 전송하는 방법에 대해서 기록합니다. 오래 걸리는 benchmark의 실행 결과만을 slack으로 전달받기 위해 매우 간단하게 설정했습니다. 유용한 slack api를 발견할 경우 계속해서 업데이트 할 계획입니다.
{: .prompt-info }

## slack app 생성

1. 브라우저에서 slack 로그인 및 <https://api.slack.com/apps> 접속

2. 우상단의 **Create New App** 클릭

   - pop-up에서 **From scratch** 선택 
   - App Name 설정 및 app을 추가할 워크스페이스 선택
   - **Create App** 클릭

3. 좌측 사이드바의 **OAuth & Permissions**에서 **Scopes** 설정. Bot이 특정 채널에 메세지를 보내기 위해 `incoming-webhook`, `chat:write`, `groups:write` 권한 필요

   <img src="assets/img/tools/slack_api_scopes.png" alt="slack_api_scopes.png" style="zoom: 50%;" />

4. **OAuth & Permissions**의 **OAuth Tokens**에서 **Install to workspace** 클릭

   - app을 게시할 채널 선택 및 허용

5. 좌측 사이드바에서 **Incoming Webhooks** 선택

   - 생성된 Webhook URL을 curl을 사용하여 테스트

```bash
curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' Webhook_URL
```

   - 성공적으로 잘 실행되면 준비 완료



## Shell Script 생성

프로그램의 실행 결과를 slack으로 전송하기 위해 다음과 같은 스크립트를 생성했습니다.

```sass
# !/bin/bash

payload='{"text":""}'
payload=$(echo "$payload" | jq -r --arg cmd "From $HOSTNAME: $*" '.text = $cmd')

curl -X POST -H 'Content-type: application/json' --data "$payload" Webhook_URL > /dev/null 2>&1
```
{: file='~/slack.sh'}

아래와 같이 사용하여 프로그램의 return value를 slack으로 전송받을 수 있습니다. (`$?`는 직전 프로그램의 return value를 의미합니다.)

```bash
$ your_program; ~/slack.sh $?
```


