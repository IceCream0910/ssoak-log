---
title: 웹인 듯 웹 아닌 앱 같은 너 - PWA 편
date: '2022-09-12'
tags: ['개발 세션']
draft: false
summary: '앱처럼 동작하는 웹. 프로그레시브 웹앱을 이용해 웹앱을 구현하는 방법을 소개합니다.'
---

안녕하세요.
쏙은 웹 기반으로 개발되었지만 마치 앱처럼 작동합니다.
브라우저에서 웹 앱으로 설치하여 실제 앱처럼 홈화면에 추가할 수도 있죠.
이게 어떻게 가능할까요? 웹으로 만드는 앱, PWA에 대해 알아보겠습니다.

## PWA(Progressiv Web App)
프로그레시브 웹 앱으로, HTML, CSS, JS와 같은 웹기술로 만드는 앱을 말합니다.
HTML, CSS, JS로 만든 웹앱을 모던한 웹 브라우저 APIs와 결합해서 크로스 플랫폼에서 동작하는 애플리케이션을 쉽게 만들 수 있습니다.
따라서 PWA는 플랫폼(ios, android...)별로 앱을 따로 만드는 것보다 쉽고 빠르게 개발할 수 있죠. 실제 트위터, 스타벅스, 핀터레스트 등의 앱도 모두 PWA 기술을 사용하고 있습니다.

### 장점
- 다양한 앱스토어에 출시하기 위한 과정을 거치지 않아도 됨

- 웹 기술을 사용해서 다양한 플랫폼을 지원하는 서비스를 개발할 수 있음

- 기존의 웹사이트를 사용하기 때문에 추가로 유지보수에 용이함

- 웹브라우저를 통해 설치없이 바로 바탕화면에 앱 아이콘 추가가 가능함

등이 있지만
네이티브 앱처럼 플랫폼의 모든 API를 사용하지는 못하기 때문에, 100% 네이티브 앱처럼 동작하기는 어렵다는 한계도 있습니다.

근데..
#### 네이티브 앱이 뭐죠
네이티브 앱 은 iOS는 swift, 안드로이드는 Java 처럼 해당 플랫폼에 맞는 프로그래밍 언어를 통해 만드는 앱을 말합니다.
특정 플랫폼에서 제공하는 다양한 API를 제공할 수 있으므로 사용자에게 매우 다양한 기능을 제공할 수 있고 안정적인 성능을 낼 수 있다는 장점이 있습니다.
그러나 플랫폼 별로 따로 개발해야 하기 때문에 개발비용이 큽니다.

본론으로 돌아가서, 그럼 이 PWA를 만들려면 어떻게 해야 할까요?

## Manifest.json 생성
소스코드 디렉토리에 `manifest.json` 파일을 생성해줍니다.
manifest.json 파일은 json 포맷 파일로서, 모든 웹 익스텐션이 포함하고 있어야 하는 파일입니다.
```json
{
  "name": "앱 이름",
  "short_name": "짧은 앱 이름",
  "icons": [
    {
      "src": "/icon_x128.png",
      "type": "image/png",
      "sizes": "128x128"
    },
    {
      "src": "/icon_x144.png",
      "type": "image/png",
      "sizes": "256x256"
    },
    {
      "src": "/icon_x512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": "/",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#f2f2f2"
}
```
이런 식으로 앱의 정보를 명시해주는 파일이라고 생각하면 됩니다.

- name: 웹 앱의 이름입니다.
- short_name: 홈 화면에 표시할 약식 이름입니다.
- description: 앱이 무엇을 하는지 설명하는 간략한 문장입니다.
- icons: 아이콘들의 정보(URL, 크기, 타입)입니다. 사용자의 기기에 적합한 것을 선택할 수 있도록 여러 개를 추가하시기 바랍니다.
- start_url: 앱이 시작할 때 실행할 초기 문서입니다.
- display: 앱을 표시하는 방식입니다(전체화면(fullscreen), 독립형(standalone), minimum-ui, browser)
- theme_color: 운영 체제에 의해 사용될 UI를 위한 주요 색상입니다.
- background_color: 스플래시 화면과 설치하는 동안 사용될 배경 색상입니다.

## html과 연결
manifest.json을 생성했다면 이제 html 파일과 연결해야겠죠.
```html
<link rel="manifest" href="/manifest.json" crossorigin="use-credentials" />
```

## Services Worker
Service Worker는 브라우저와 네트워크 사이의 가상 프록시입니다. 
웹 사이트의 자원을 적절히 캐싱(임시 저장)하거나 사용자의 기기가 오프라인일 때 이를 사용할 수 있도록 구현할 수 있게 해줍니다.

`service-worker.js` 파일을 만들어주겠습니다.

```js
const CACHE_NAME = "cache-v1";
// 캐싱할 파일
const FILES_TO_CACHE = [
    "index.html",
    "offline.html"
];

// 상술한 파일 캐싱
self.addEventListener("install", (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME).then((cache) => cache.addAll(FILES_TO_CACHE))
    );
});

// CACHE_NAME이 변경되면 오래된 캐시 삭제
self.addEventListener("activate", (event) => {
    event.waitUntil(
        caches.keys().then((keyList) =>
            Promise.all(
                keyList.map((key) => {
                    if (CACHE_NAME !== key) return caches.delete(key);
                })
            )
        )
    );
});

// 요청에 실패하면 오프라인 페이지 표시
self.addEventListener("fetch", (event) => {
    if ("navigate" !== event.request.mode) return;

    event.respondWith(
        fetch(event.request).catch(() =>
            caches
                .open(CACHE_NAME)
                .then((cache) => cache.match("offline.html"))
        )
    );
});
```

다시 html에서 service-worker를 연결해줘야 합니다.
```html
<script>
if ("serviceWorker" in navigator) {
  window.addEventListener("load", () => {
    navigator.serviceWorker.register("service-worker.js");
  });
}
jQuery.browser = {};
(function () {
  jQuery.browser.msie = false;
  jQuery.browser.version = 0;
  if (navigator.userAgent.match(/MSIE ([0-9]+)\./)) {
    jQuery.browser.msie = true;
    jQuery.browser.version = RegExp.$1;
  }
})();
</script>

```

이제 html페이지를 브라우저로 연 후 F12 개발자도구를 켜줍니다.
Application 탭에 아래와 같이 service worker가 활성화된 것을 확인할 수 있습니다.

![image](https://i.imgur.com/iFm7Fs2.png)

이렇게 하면 PWA 웹앱을 구현하게 됩니다.
사용자가 좀 더 쉽게 웹앱을 추가할 수 있도록 안내해볼까요?

## PWA 설치 버튼 만들기
PWA가 지원되는 사이트의 경우 모바일에서는 브라우저 자체에서 설치 배너가 표시되고 PC에서도 주소창에 설치 버튼이 생깁니다.
그런데 사용자들은 PWA 웹 앱을 설치할 수 있다는 것을 대부분 알지 못하기 때문에 버튼을 눌러 설치할 수 있도록 만들어보겠습니다.

간단합니다. 다음과 같이 Javascript 코드를 작성해주세요.
```js
var deferredPrompt;

window.addEventListener('beforeinstallprompt', function (e) {
    console.log('beforeinstallprompt Event fired');
    e.preventDefault();
    deferredPrompt = e;
    return false;
});

const installWebApp = document.getElementById('installWebApp'); //버튼 요소 지정

installWebApp.addEventListener('click', function () { //버튼 클릭 이벤트
    if (deferredPrompt !== undefined) {
        // 설치 팝업 띄우기
        deferredPrompt.prompt();

        // 사용자 응답 확인
        deferredPrompt.userChoice.then(function (choiceResult) {

            console.log(choiceResult.outcome);

            if (choiceResult.outcome == 'dismissed') {
                console.log('설치 취소함');
            }
            else {
                console.log('설치 완료');
            }

            deferredPrompt = null;
        });
    } else { //이미 설치 or 미지원 환경
        alert('지원되지 않는 브라우저이거나 이미 설치되어 있습니다. 브라우저는 크롬 또는 사파리를 권장합니다.')
    }
});
```

![image](https://i.imgur.com/5H2n0Ih.png)
근사하군요.