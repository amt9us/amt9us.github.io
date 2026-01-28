---
title: PaperMod 테마 상세 설정하기
date: 2026-01-28T17:14:20+09:00
description: "Hugo PaperMod 테마의 hugo.yaml 상세 설정 정리"
categories: ["Log"]
tags: ["Hugo", "PaperMod", "Config"]
draft: false
---

GitHub Action을 활용해 포스팅 자동화 환경까지 구축을 했으니, 이제 밋밋한 블로그를 조금 꾸며보려고 합니다.
**PaperMod** 테마는 미니멀한 디자인 뒤에 생각보다 강력하고 다양한 기능을 가지고 있습니다.
`hugo.yaml` 파일을 하나씩 뜯어보며, 필요한 기능들을 어떻게 활성화하고 제어하는지 정리해 보겠습니다.

## hugo.yaml 설정
PaperMod의 설정은 크게 기본 정보, 테마 기능(`params`), 그리고 메뉴 설정으로 나뉩니다.
### 1. 기본 시스템 설정
```yaml
baseURL: "https://<username>.github.io/"
languageCode: en-us
title: <title>
pagerSize: 6
theme: PaperMod
```
- **baseURL**: 블로그의 실제 주소입니다. GitHub Pages 주소를 넣으면 됩니다.
- **pagerSize**: 메인 페이지에서 한 번에 보여줄 포스트 개수입니다.
### 2. 사이트 정체성 및 메타 데이터
```yaml
env: production
title: amt9us.github.io
description: "실무 경험과 개인적인 학습 기록을 정리하는 공간입니다."
keywords: [Backend, Devlog, Blog]
author: amt9us
images: ["/images/default-og.jpg"]
DateFormat: "January 2, 2006"
defaultTheme: auto
disableThemeToggle: false
```
- **env**: 배포 환경임을 명시합니다.
- **description & keywords**: 검색 엔진(Google, Naver 등)이 내 블로그를 파악할 때 사용하는 아주 중요한 정보(SEO)입니다.
- **defaultTheme & disableThemeToggle**: 시스템 설정에 따라 다크/라이트 모드가 자동으로 바뀌게(`auto`) 설정했습니다.

### 3. 포스팅 가독성을 위한 디테일
```yaml
ShowReadingTime: true
ShowShareButtons: true
ShowPostNavLinks: true
ShowBreadCrumbs: true
ShowCodeCopyButtons: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
ShowToc: true
TocOpen: true
disableSpecial1stPost: false
disableScrollToTop: false
comments: true
hidemeta: false
```
이 부분은 글을 읽는 독자의 편의성을 결정합니다.
- **TocOpen**: 목차를 항상 펼쳐둡니다.
- **ShowCodeCopyButtons**: 코드 블록에 복사 버튼을 추가합니다.
- **ShowReadingTime & WordCount**: 이 글을 읽는 데 시간이 얼마나 소요될지 미리 알려줍니다.

### 4. 검색 기능 (Fuse.js)
```yaml
fuseOpts:
  isCaseSensitive: false
  shouldSort: true
  location: 0
  distance: 1000
  threshold: 0.4
  minMatchCharLength: 2
  limit: 10
  keys: ["title", "summary", "content"]
```
- **threshold**: 값이 낮을수록 검색어와 정확히 일치해야 합니다. 0.4 정도가 오타를 적당히 허용하면서도 정확한 결과를 뽑아줍니다.
- **keys**: 검색어가 제목(`title`), 요약(`summary`), 본문(`content`) 모두에서 검색되도록 설정했습니다.

### 5. 상단 네비게이션 메뉴 및 페이지 활성화
```yaml
menu:
  main:
    - identifier: search
      name: Search
      url: /search/
      weight: 5
    - identifier: archives
      name: Archives
      url: /archives/
      weight: 10
    - identifier: tags
      name: Tags
      url: /tags/
      weight: 20
    - identifier: categories
      name: Categories
      url: /categories/
      weight: 30
```
> **⚠️ 주의사항**: `hugo.yaml`에 메뉴를 추가했다고 해서 페이지가 자동으로 생기지는 않습니다. 각 URL에 매칭되는 마크다운 파일을 `content` 폴더 안에 직접 만들어줘야 404 에러가 나지 않습니다.

#### **페이지 생성을 위한 필수 작업**
각 기능을 정상적으로 사용하려면 아래와 같이 파일을 생성하고 `layout`을 지정해줘야 합니다.
1. **검색 페이지 (`content/search.md`)**
```markdown
---
title: "Search"
layout: "search" # PaperMod가 제공하는 전용 검색 레이아웃 사용
summary: "search"
placeholder: "찾으시는 포스팅의 키워드를 입력하세요..."
---
```

2. **아카이브 페이지 (`content/archives.md`)**
```markdown
---
title: "Archive"
layout: "archives" # 월별/연도별로 글을 모아주는 레이아웃
url: "/archives/"
summary: "archives"
---
```

### 6. 코드 하이라이트 (Markup & Highlight)
```yaml
pygmentsUseClasses: true
markup:
  highlight:
    noClasses: false
    codeFences: true
    guessSyntax: true
    lineNos: true
    anchorLineNos: false
    style: github
```
- **lineNos**: 코드 라인 번호를 표시합니다.
- **style: github**: 코드 스테일 테마입니다.
- **guessSyntax**: 코드 언어를 명시하지 않아도 자동으로 분석해 색을 입혀줍니다.

---

설정을 마치고 나니 밋밋하던 블로그가 제법 봐줄 만한 모습으로 변했습니다.

하지만 아직 완벽하진 않습니다.
글 사이의 간격이라든지, 코드 블록의 둥글기(rounding), TOC의 위치나 접기 기능 등 조금 더 디테일하게 손보고 싶은 부분들이 눈에 띄기 시작했습니다.

다음 포스팅에서는 **댓글 기능**과, CSS를 활용한 **디테일한 TOC 커스텀**에 대해 다뤄보겠습니다.
