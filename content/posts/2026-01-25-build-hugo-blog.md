---
title: "GitHub Pages와 Hugo로 기술 블로그 구축하고 배포하기"
date: 2026-01-25T22:06:00+09:00
description: "올해도 시작된 N번째 블로그 도전기"
categories: ["Log"]
tags: ["Hugo", "GitHub", "PaperMod", "Blog", "Config"]
draft: false
---

## 1. 블로그를 시작한 이유

지난 몇 년간 기술 블로그를 '시작하고 방치하기'를 수없이 반복해 왔습니다.  
돌이켜보니 꾸준히 써야 한다는 강박과 주제에 대한 모호한 기준 때문에 글쓰기가 점점 과제처럼 느껴졌던 것 같습니다.

특히 타인의 글을 옮겨 적거나 AI가 만든 내용을 그대로 복사해 오곤 했는데, 제 머릿속에는 아무것도 남지 않았고, 자연스레 블로그에 대한 애정도 식어버렸습니다.

그래서 이제는 AI를 초기 구성과 문장을 다듬는 ‘편집 도구’로만 활용하고, 내용은 반드시 저의 고민과 문장으로 직접 채워보려 합니다.

올해는 진짜.. 진짜.. 다를거에요...

## 2. 왜 깃허브 블로그인가?

Tistory나 Velog 같은 편리한 플랫폼도 많지만, 굳이 번거로운 깃허브 블로그(GitHub Pages)를 선택한 이유는 다음과 같습니다. 
- **커스터마이징 자유도**: 다른 플랫폼은 제공되는 틀 안에서만 움직여야 합니다. Velog는 다소 고정적이고, Tistory는 자유도가 높지만 설정 방식이 파편화되어 있어 피곤함을 느꼈습니다. 반면 GitHub 블로그는 UI/UX부터 기능 하나까지 제가 직접 코드로 제어할 수 있습니다.
- **개발자 갬성(?)**: 티스토리는 기술 블로그로 많이 사용되지만 범용 플랫폼의 느낌이 강하고, Velog는 모두가 똑같은 UI를 사용한다는 점이 아쉬웠습니다. **홍대병 말기인 저에게는 이보다 딱 맞는 공간이 없었습니다.**
- **잔디로 증명하는 지속 가능성**: 개발자에게 가장 익숙한 공간은 GitHub입니다. 단순히 글을 쓰는 행위를 넘어, 코드와 함께 커밋(Commit) 데이터로 기록이 남는다는 점은 블로그를 꾸준히 운영하게 만드는 강력한 동기부여가 될 것 같습니다.

깃허브 블로그는 주로 Jekyll, Hexo, Hugo가 있지만, **PaperMod 테마**가 제일 마음에 들어 Hugo를 선택하게 되었습니다.

## 3. 구축 순서별 상세 설명

### STEP 1: Hugo 설치 및 프로젝트 생성

Hugo를 설치하고 새로운 사이트 구조를 만듭니다.

```bash
# 1. Homebrew를 이용한 Hugo 설치
brew install hugo

# 2. 신규 사이트 생성 (설정 파일은 yaml 포맷)
hugo new site blog --format yaml
cd blog
```
### STEP 2: GitHub 저장소 생성 및 로컬 연결

GitHub에서 `<사용자명>.github.io` 저장소를 생성한 뒤 로컬 폴더와 연결합니다.

```bash
git init
git add .
git commit -m "feat: create hugo blog"
git branch -M main
git remote add origin https://github.com/<사용자명>/<사용자명>.github.io.git
git push -u origin main
```
### STEP3: 테마 적용 (Submodule 구성)

[Themes](https://themes.gohugo.io/)에서 원하는 테마를 하나 선택하고 `submodule`로 등록하여 관리의 편의성을 높입니다.

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

```
### STEP 4: 사이트 설정 (`hugo.yaml`

기본적인 블로그 정보와 테마 설정을 진행합니다.

```yaml
baseURL: "https://<사용자명>.github.io/"
languageCode: en-us
title: <사용자명>.github.io
pagerSize: 6
theme: PaperMod
```
### STEP 5: GitHub Pages & Actions 설정
GitHub 저장소의 **Settings > Pages**에서 Build and deployment의 Source를 `GitHub Actions`로 변경합니다. 

![GitHub Pages 설정 화면](/images/blog-setup-01.jpg)

이제 푸시할 때마다 자동으로 빌드됩니다.

### STEP 6: 최종 배포 (Git Commit & Push)
프로젝트 루트에 `.github/workflows/hugo.yaml` 파일을 생성하고 아래 워크플로우를 저장합니다.

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build with Hugo
        run: hugo --gc --minify --baseURL "https://<사용자명>.github.io/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### STEP 7: 로컬 서버 검수 및 최종 푸시

배포 전 `hugo server` 명령어로 마지막 확인을 거친 뒤 서버로 전송합니다.

```bash
# 로컬에서 미리보기 (http://localhost:1313)
hugo server -D

# 모든 변경사항 스테이징 및 커밋
git add .
git commit -m "feat: publish first post"

# 원격 저장소로 푸시
git push origin main
```

## 4. 마치며
이전에는 매번 `deploy.sh` 스크립트를 직접 실행하며 수동으로 빌드와 배포를 관리해야 했는데 이번에 **GitHub Actions**를 도입하면서 `git push` 한 번으로 빌드부터 배포까지 자동화하니까 간단하고 편리해졌습니다.

다음 작업으로는 제 눈에 더 편한 블로그 UI/UX를 하나씩 수정해 봐야겠습니다.
