---
title: "Giscus 댓글 연동 및 사이드바 ToC 적용"
date: 2026-01-29T21:19:57+09:00
description: "PaperMod 테마의 기본 ToC 커스텀하고, Giscus 댓글 시스템을 연동하는 방법을 정리합니다."
categories: ["Log"]
tags: ["Hugo", "PaperMod", "Giscus", "CSS"]
draft: false
---

이번 포스팅에서는 GitHub Discussions를 활용한 댓글 시스템 **Giscus** 연동과, **사이드바 ToC** 구성을 정리했습니다.  
사실 댓글 시스템에는 다양한 선택지가 있었지만, 대부분 Giscus를 사용하고 있고 GitHub 생태계와 가장 잘 맞는다는 점 때문에 저도 선택하게 되었습니다.

## 1. Giscus 댓글 시스템 연동
Giscus는 GitHub의 **Discussions** API를 이용하기 때문에, 리포지토리 자체의 설정을 먼저 변경해야 합니다.

### 1-1. GitHub 및 Giscus 기본 설정
1. **Discussions 활성화**: 블로그 리포지토리의 `Settings > General > Features`에서 **Discussions**를 체크합니다.
2. **Giscus 앱 설치**: [GitHub Giscus App](https://github.com/apps/giscus)을 설치하고 블로그 리포지토리에 권한을 부여합니다.
3. **설정값 추출**: [giscus.app](https://giscus.app/ko)에서 저장소명(`유저명/저장소명`)을 입력하고 `data-repo-id`, `data-category-id`를 복사해둡니다.

### 1-2. 테마 동기화 스크립트 (`comments.html`)
단순 삽입만 하면 블로그의 다크/라이트 모드 전환 시 댓글창 테마가 따로 노는 문제가 발생합니다.  
이를 해결하기 위해 `postMessage`를 이용한 동기화 로직을 추가했습니다. 
```js
/* layouts/partials/comments.html */
function getGiscusTheme() {
    const theme = localStorage.getItem("pref-theme");
    // 블로그 테마 상태에 따라 giscus의 protanopia 테마 매칭
    const darkTheme = "dark_protanopia";
    const lightTheme = "light_protanopia";

    if (theme === "dark") return darkTheme;
    if (theme === "light") return lightTheme;
    return window.matchMedia('(prefers-color-scheme: dark)').matches ? darkTheme : lightTheme;
}

function setGiscusTheme() {
    const iframe = document.querySelector('iframe.giscus-frame');
    if (!iframe) return;
    
    // iframe에 메시지를 보내 실시간으로 테마 변경
    iframe.contentWindow.postMessage({
        giscus: { setConfig: { theme: getGiscusTheme() } }
    }, 'https://giscus.app');
}

// 블로그의 테마 토글 버튼 클릭 시 setGiscusTheme 실행
document.addEventListener('DOMContentLoaded', () => {
    const themeToggle = document.querySelector('#theme-toggle');
    if (themeToggle) themeToggle.addEventListener('click', setGiscusTheme);
});
```

## 2. 사이드바 ToC 커스텀 적용
PaperMod는 기본적으로 ToC가 본문 중앙 상단에 위치하여, 스크롤을 내리면 목차가 사라지는 단점이 있습니다.  
이를 **우측 사이드바 고정형(Sticky/Fixed)**으로 개선했습니다.

### 2-1. Hugo 및 CSS 설정
`hugo.yaml`에서 `ShowToc: true`를 확인한 후, `assets/css/extended/toc.css`에 아래 내용을 추가합니다.

```css
/* PC 화면 (1350px 이상)에서만 사이드바 적용 */
@media screen and (min-width: 1350px) {
    .toc {
        position: fixed !important;
        top: 150px;             /* 상단 헤더와의 간격 */
        left: calc(50% + 420px); 
        width: 280px;           /* 사이드바 적정 너비 */
        background: none !important;
        border: none !important;
    }

    /* 목차 제목 스타일 (Table of Contents) */
    .toc details summary {
        cursor: pointer;
        font-weight: 600;
        font-size: 0.95rem;
        color: var(--primary);  /* 테마 기본 포인트 컬러 */
        margin-bottom: 15px;
    }

    /* 가이드 라인 최적화 */
    .toc .inner {
        margin-left: 7px;
        padding-left: 15px;
        border-left: 1px solid var(--tertiary); /* 세로 가이드 라인 */
    }

    /* 링크 가독성 및 호버 효과 */
    .toc a {
        color: var(--secondary);
        display: block;
        padding: 6px 0;
        font-size: 13.5px;
        transition: all 0.2s ease; /* 부드러운 전환 */
    }

    .toc a:hover {
        color: var(--primary);
        transform: translateX(4px); /* 오른쪽으로 살짝 이동 */
    }
}
```

#### 주요 개선 포인트
1. 고정 위치(Fixed): position: fixed를 사용하여 스크롤 시에도 항상 우측에 위치하게 함.
2. 반응형(1350px): 화면이 좁은 모바일에서는 본문 상단에 유지하고, 넓은 데스크탑 화면에서만 사이드바로 이동.
3. 시각적 가이드: border-left를 활용해 H2, H3 계층 구조를 선으로 연결하여 가독성 향상.

## 마무리
확실히 댓글과 목차가 생기니까 좀 더 블로그다워진 거 같습니다.  
목차(ToC)는 글이 길어질 때 도움이 될 거 같고, 댓글 창은 사실 누가 달아주실까 싶긴 하지만... 없으면 또 허전하니까 추가해봤습니다.

다음 포스팅에서는 **전체 글 간격과 모바일 반응형 설정**을 해보려 합니다.
