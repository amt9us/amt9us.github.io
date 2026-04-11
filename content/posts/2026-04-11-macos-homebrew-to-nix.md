---
title: "macOS 패키지 관리를 Homebrew에서 Nix Flake + Home Manager로 이관한 과정"
date: 2026-04-11T14:16:20+09:00
description: "개발 환경의 반복 세팅 문제를 해결하기 위해 Homebrew 기반 환경을 Nix Flake + Home Manager 기반 선언형 구조로 마이그레이션한 경험"
categories: ["Log"]
tags:
  - nix
  - home-manager
  - flake
  - homebrew
  - dotfiles
draft: false
---

기존에는 개발 환경을 Homebrew 중심으로 구성하고, 언어 버전 관리만 mise로 분리하여 사용해 왔습니다.  
하지만 최근 새로운 서버에 Nix Flake와 Home Manager를 도입해 보며, 어떤 환경에서든 동일한 상태를 유지해 주는 Nix의 강력한 재현성을 몸소 체험하게 되었습니다.

평소 개발 환경을 구축하고 최적화하는 과정 자체는 즐기는 편이지만, 역설적으로 새로운 장비나 서버를 세팅할 때마다 매번 같은 환경을 반복해서 재구성하는 일은 점점 비효율적인 소모전처럼 느껴졌습니다.

이에 서버와 로컬 환경을 최대한 간단하게 통일하고자 파편화되어 있던 기존 방식을 과감히 정리하고,  
이제는 Nix Flake와 Home Manager를 기반으로, 단 한 줄의 명령어로 어제 쓰던 환경을 100% 재현할 수 있는 **선언적 통합 환경**으로의 전환을 시도했습니다.

## 🧭 1. Nix + Flake + Home Manager 설치

가장 먼저 Nix를 설치하고 Flake 기능을 활성화합니다.

```bash
curl -L https://nixos.org/nix/install | sh
```

- Flake 활성화

```bash
mkdir -p ~/.config/nix
echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf
```

이후 Home Manager를 flake 기반으로 사용합니다.

## 🗂️ 2. 프로젝트 구조 (The Structure)

환경이 복잡해질수록 구조가 중요해집니다.

```bash
~/nix-config
├── flake.nix              # 전체 entry point
├── home.nix               # 유저 환경 조립
├── modules/               # 기능 단위 모듈
│   ├── zsh.nix
│   ├── git.nix
│   ├── files.nix
│   ├── btop.nix
│   └── gh.nix
└── config/                # 실제 앱 설정 파일
    ├── lazygit/
    ├── alacritty/
    ├── mise/
    └── zellij/
```

### 💡 구조 설계 의도
- `flake.nix` → 전체 시스템 진입점
- `home.nix` → 사용자 환경 조립 중심
- `modules/` → 기능 단위로 책임 분리
- `config/` → 실제 앱 설정 원본 저장소

## 📦 3. Brew → Nix로 패키지 마이그레이션

기존 Homebrew로 설치하던 CLI 도구들을 Nix로 옮겼습니다.

```nix
home.packages = with pkgs; [
  git lazygit fd fzf ...
  mise jq uv 
];
```

## 🧩 4. dotfiles → Nix 관리 전환

기존에는 stow 또는 수동 symlink로 관리하던 설정을 Nix로 이동했습니다.

```bash
xdg.configFile = {
  "lazygit".source = ../config/lazygit;
  "alacritty".source = ../config/alacritty;
  "cava".source = ../config/cava;
  "zellij".source = ../config/zellij;
};
```

이제 설정은 단순히 “파일 복사”가 아니라 “선언된 상태로 항상 유지되는 구조”가 됩니다.

## 🚀 5. 적용 방법

모든 변경은 아래 한 줄로 적용됩니다.

```bash
cd ~/nix-config
home-manager switch --flake .
```

## 🔁 AS-IS / TO-BE

### 🔴 AS-IS (기존 환경)

- 패키지 관리: Homebrew 기반 수동 설치
- 언어 버전 관리: mise로 일부 분리
- dotfiles 관리: stow 또는 수동 symlink
- 환경 재현: 불가능 (환경마다 미세하게 다름)

**❗ 문제점**

- 새로운 머신 세팅 시 반복 작업 발생
- 시간이 지날수록 환경이 조금씩 달라짐 (drift)
- 서버 / 로컬 환경의 일관성 부족

### 🟢 TO-BE (Nix 환경)

- 패키지 관리: Nix Flake 기반 선언형 관리
- 사용자 환경: Home Manager
- dotfiles: modules + config 구조로 코드화
- 환경 재현: 단일 명령어로 100% 동일 환경 복원

**✅ 개선점**

- git clone + switch 한 번으로 환경 복원
- 서버 / 로컬 / 신규 맥 동일 환경 유지
- “설치”가 아니라 “재현”으로 전환

---

Nix Flake와 Home Manager를 도입하면서 가장 크게 바뀐 점은  
“환경을 설치한다”는 개념에서 “환경을 재현한다”는 개념으로 전환된 것이었습니다.

처음에는 단순히 도구를 바꾸는 수준이라고 생각했지만,  
지금은 개발 환경 자체를 코드로 다룬다는 점이 훨씬 더 큰 변화였습니다.

앞으로는 새로운 머신이나 서버를 구성할 때도  
이 구조를 기반으로 동일한 환경을 계속 유지해 나갈 예정입니다.
