---
title: "모바일 반응형 및 본문 가독성 개선"
date: 2026-01-31T11:55:11+09:00
description: "모바일 반응형 설정 및 가독성 증가 설정한 경험을 정리합니다."
categories: ["Log"]
tags: ["Blog", "PaperMod", "Config", "Responsive", "Hugo", "UI/UX"]
draft: false
---

평소에는 노트북으로 블로그를 관리하다 보니 몰랐는데, 아이패드나 휴대폰으로 글을 다시 읽을 때면 묘한 불편함이 느껴졌습니다.  
글자들은 너무 빽빽하고, 링크는 손가락으로 누르기 힘들고...

그래서 모바일 환경에서도 시원시원하게 읽히는 **반응형 스타일과 본문 줄간격 최적화 설정을 해보려** 합니다.

## 1. 줄간격 및 공통 스타일 설정
가독성의 핵심은 '여백'이라고 생각합니다.  
화면 전체에 글자가 가득 차 있으면 눈이 쉽게 피로해지기 때문에, PC와 모바일 모두에 적용되는 공통 간격을 먼저 손봐주었습니다.

### 제목(Heading) 간격 조절
```css
.post-content h1,
.post-content h2,
.post-content h3 {
    margin-top: 2.5rem !important;  /* 위쪽 여백을 넓혀 섹션 구분 명확화 */
    margin-bottom: 1.2rem !important;
    line-height: 1.4;
}

.post-content h2 {
    border-bottom: 1px solid var(--tertiary); /* 구분선으로 시각적 안정감 */
    padding-bottom: 0.3rem;
}
```

### 리스트 아이템 간격
기본 설정에서는 리스트(`ul`, `ol`)의 줄간격이 좁아 답답해 보였습니다.  
이를 `1.6` 정도로 높여 시원하게 배치했습니다.

```css
.post-content ul li,
.post-content ol li {
    margin-top: 0.3em;
    margin-bottom: 0.3em;
    line-height: 1.6; /* 줄간격을 넓혀 가독성 향상 */
}
```

## 2. 모바일 반응형 대응 (768px 이하)

모바일 기기에서 체감되는 불편함을 해결하기 위한 설정입니다.

### 링크 터치 영역 확보

모바일에서는 마우스 커서가 아닌 손가락을 사용합니다.  
따라서 텍스트 링크가 너무 얇으면 클릭하기가 정말 어렵기 때문에 `min-height`를 활용해 눈에 보이지 않는 터치 영역을 확장했습니다.

```css
@media screen and (max-width: 768px) {
    .post-content p a, 
    .post-content blockquote a {
        padding: 2px 0;
        min-height: 44px; /* 최소 터치 높이 확보 */
        display: inline-block;
    }
}
```

### 표(Table)와 코드 블록 최적화
작은 화면에서 표가 깨지거나 코드 블록이 너무 커서 가독성을 해치는 문제를 방지했습니다.
- **표(Table):** 여백을 최소화하여 좁은 화면에서도 최대한 많은 정보를 담도록 설정.
- **코드 블록:** 폰트 크기를 살짝 줄이고(`0.85em`), 가로 스크롤이 부드럽게 작동하도록 설정. 

```css
.post-content pre {
    overflow-x: auto;
    -webkit-overflow-scrolling: touch; /* iOS 등에서 부드러운 스크롤 제공 */
    font-size: 0.85em;
}

.post-content table td {
    padding: 3px 2px !important;
    line-height: 1.2 !important;
}
```

---

설정을 마치고 나니 이제는 어떤 기기로 접속해도 제가 의도한 대로 글이 읽히는 것 같아 마음이 한결 편안합니다.

다음에는 다양한 언어로 번역해 주는 '다국어 변환 토글' 기능을 구현해보려 합니다.
