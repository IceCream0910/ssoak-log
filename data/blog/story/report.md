---
title: Slack으로 신고 기능 빠르게 구현하기
date: '2022-09-12'
tags: ['개발 세션']
draft: false
summary: 'Slack API를 사용해 커뮤니티 신고 기능을 구현하는 방법을 소개합니다.'
---

안녕하세요.
쏙 커뮤니티에는 신고 기능이 있습니다. 부적절한 게시물이나 댓글 옆 느낌표 아이콘을 눌러 신고할 수 있죠.
신고가 접수되면 아래와 같이 Slack을 통해 알림이 오고 신고 내용을 처리할 수 있습니다.
![image](https://i.imgur.com/JL5n7a7.png)


### 신고 기능이 왜 필요함
쏙에도 처음에 신고 기능이 없었습니다.
그런데 말입니다. Play 스토어에 게시할 때 커뮤니티 기능에 신고 기능이 없을 경우 관련 가이드라인에 따라 앱 게시가 승인되지 않더군요.
그래서 빠른 시간 안에 신고 기능을 구현하고자 Slack API를 사용했습니다.

그럼 Slack API를 사용해 웹 상에서 Slack 채널로 메시지를 전송하는 방법을 알아볼까요?

---
우선 Slack에 가입하여 워크스페이스를 생성해주어야 합니다. 이 과정은 생략하겠습니다.

## Slack 워크스페이스에 앱 추가하기
![image](https://i.imgur.com/1lHADY7.png)
좌측에 앱을 눌러 검색창에 `incoming webhook`을 검색해서 추가해줍니다.

![image](https://i.imgur.com/I0qZQpx.png)
Slack에 추가를 눌러주세요.

![image](https://i.imgur.com/mEYeWTt.png)
메시지를 발송할 채널을 선택한 후 `수신 웹후크 통합 추가`를 눌러주세요.

이후 나오는 설정 페이지에서 웹 후크 url을 확인해주세요.
![image](https://i.imgur.com/r4HCUKE.png)

아래 설정에서 프로필 사진이나 이름 등을 설정할 수도 있습니다.

## Javascript와 연결하기
이제 js 상에서 신고 내용을 Slack 채널로 전송해볼까요?
전송하는 코드는 아래와 같습니다.

```js
 var payload = JSON.stringify({
            "fallback": "새로운 신고가 접수되었습니다.",
            "text": "커뮤니티 글에 대한 새로운 신고가 접수되었습니다.",
            "color": "danger",
            "fields": [
                {
                    "title": "게시글 내용 :",
                    "value": "전송할 값",
                    "short": false  // 값이 다른 값과 나란히 표시될 정도로 짧은지를 나타내는 옵션
                },
                {
                    "title": "게시글 링크:",
                    "value": "전송할 값",
                    "short": false
                }
            ]
});

const url = '위에서 발급된 webhook url 입력';

$.post(url, payload).done(function (data) {
     console.log("신고 등록됨");
});
```

url 변수에는 위에서 발급받은 webhook url을 입력해줍니다.
> github에 webhook url이 포함된 소스코드를 게시할 경우 자동으로 새 웹훅 url이 발급되어 기존 url은 사용할 수 없게 됩니다. 따라서 환경변수 등의 방법으로 소스코드 상에 url이 직접 노출되지 않도록 해야합니다.

이렇게 해주면 위에서 봤던대로 payload에서 지정해준 json 형식으로 채널에 메시지가 전송됩니다.
![image](https://i.imgur.com/JL5n7a7.png)

형식은 [Creating rich message layouts 문서](https://api.slack.com/messaging/composing/layouts)를 참고하세요.

---

Slack을 활용해 간단하게 신고 기능을 구현해봤습니다.
이를 사용하면 사용자의 입력을 별도 서버 없이도 쉽게 받아올 수 있어 신고 기능 뿐 아니라 다양한 기능에 활용할 수 있을 것 같습니다.