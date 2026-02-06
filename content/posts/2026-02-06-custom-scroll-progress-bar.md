---
title: "블로그 스크롤 진행 바(Progress Bar) 만들기"
date: 2026-02-06T21:49:42+09:00
description: "스크롤 진행 바(Scroll Progress Bar) 구현 경험을 정리합니다."
categories: ["Log"]
tags: ["JavaScript", "Hugo", "Web", "UX"]
draft: false
---

평소 다른 분들의 기술 블로그를 읽다 보면, 화면 최상단에서 스크롤을 따라 부드럽게 차오르는 **진행 바(Progress Bar)**가 유독 눈에 띄곤 했습니다.  
긴 글을 읽을 때 내가 어느 지점쯤 머물고 있는지 직관적으로 보여주니 심리적인 안정감과 함께 완독에 큰 도움이 되더라고요?

제 블로그에도 이런 디테일이 있으면 좋겠다는 생각이 들어, 이번 기회에 직접 심플하게 구현해 보았습니다.
## 1. HTML & CSS (구조와 디자인)

먼저 진행 바가 위치할 컨테이너와 실제 바의 디자인을 잡아줍니다.
```html
<style>
    /* 진행 바 컨테이너: 최상단 고정 */
    #progress-container {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 4px; /* 바의 두께 */
        display: block;
        background: transparent;
        z-index: 10000; /* 콘텐츠보다 항상 위에 표시 */
    }

    /* 진행 상태를 나타내는 바 */
    #progress-bar {
        width: 0%; /* 시작은 0% */
        height: 100%;
        background-color: var(--primary); /* 테마 기본 포인트 컬러 활용 */
        transition: width 0.1s ease-out; /* 부드러운 애니메이션 효과 */
    }
</style>

<div id="progress-container">
    <div id="progress-bar"></div>
</div>
```
- `var(--primary)`로 현재 테마의 기본 강조 색상을 가져왔습니다.

## 2. JavaScript (스크롤 위치 계산)
사용자의 스크롤 위치를 실시간으로 추적해 너비(`width`)를 업데이트하는 로직입니다.
```js
<script>
    window.addEventListener('scroll', () => {
        // 현재 스크롤된 위치 계산
        const winScroll = window.pageYOffset || document.documentElement.scrollTop;
        
        // 실제로 스크롤 가능한 전체 높이 계산
        // (전체 문서 높이 - 현재 브라우저 창의 높이)
        const height = document.documentElement.scrollHeight - window.innerHeight;
        
        if (height > 0) {
            // 위치를 백분율로 환산하여 너비에 적용
            const scrolled = Math.min((winScroll / height) * 100, 100);
            document.getElementById("progress-bar").style.width = scrolled + "%";
        }
    });
</script>
```

## 💡 주요 포인트

- **정확한 계산**: `scrollHeight`에서 `innerHeight`를 빼주어야 사용자가 페이지 최하단에 도달했을 때 정확히 바가 100% 가득 찹니다.
- **부드러운 움직임**: CSS의 `transition` 속성을 통해 스크롤 시 바가 끊기지 않고 매끄럽게 늘어납니다.
- **Hugo 최적화**: 모든 페이지가 아닌, 포스트 본문에서만 작동하도록 `{{- if .IsPage -}}` 조건문을 감싸서 불필요한 리소스 소모를 방지했습니다.

---

다양한 기능을 가진 블로그들을 보며 '나도 해보고 싶다'는 마음으로 시작했는데, 직접 분석하고 구현해 보니 기대했던 것보다 더 재미있고 유익한 시간이었습니다.
 
앞으로도 매력적인 기능들을 발견하면 적극적으로 분석하고 적용해 보려 합니다.
