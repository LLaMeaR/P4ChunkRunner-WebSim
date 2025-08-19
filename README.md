<!-- TITLE -->
<h1 align="center">P4ChunkRunner Web Simulator</h1>

<p align="center">
  <em>Anchor 기반 Hash-CDC(Content-Defined Chunking) 시각화 시뮬레이터</em><br/>
  <sub>Sharding 대비 변경 전파를 줄이고, 의미 단위를 보존하는 청킹을 브라우저에서 실험해보세요.</sub>
</p>

<p align="center">
  <a href="https://llamear.github.io/P4ChunkRunner-WebSim/">
    <img alt="Live Demo" src="https://img.shields.io/badge/Live%20Demo-Open%20in%20Browser-2ea44f?style=for-the-badge">
  </a>
  <a href="#-features">
    <img alt="Features" src="https://img.shields.io/badge/AnchorCDC-Visualizer-1f6feb?style=for-the-badge">
  </a>
  <a href="#-how-it-works">
    <img alt="How it works" src="https://img.shields.io/badge/Explanation-How%20it%20works-8250df?style=for-the-badge">
  </a>
</p>

<p align="center">
  <a href="https://llamear.github.io/P4ChunkRunner-WebSim/">
    <img src="docs/preview.png" alt="Preview" width="900">
  </a>
</p>

> ℹ️ `docs/preview.png`는 예시 경로입니다. 스크린샷을 `docs/` 폴더에 추가하거나 경로를 수정하세요.

---

## 📌 Overview

**P4ChunkRunner Web Simulator**는 <strong>Anchor-CDC</strong> 방식으로 문자열 키 시퀀스를 청크로 나누는 과정을
브라우저에서 인터랙티브하게 확인할 수 있는 웹 앱입니다.  
해시 접두사로 고정 분할하는 **Sharding**과 달리, 앵커 조건을 만족하는 지점에서만 경계를 형성하여

- 불필요한 diff 확산을 줄이고,
- 의미 단위(semantic unit)의 분절을 완화하며,
- **Tail Append / Contiguous Delete** 시나리오에서 변경 범위를 안정적으로 국소화합니다.

---

## 🚀 Live Demo

- **바로 실행:** https://llamear.github.io/P4ChunkRunner-WebSim/  
  설치/빌드 없이 브라우저에서 즉시 실행됩니다.

---

## 🗂️ Repository Layout

```text
.
├── index.html     # 앱 엔트리 (UI)
├── main.js        # Anchor-CDC 로직 + 시각화 동작
├── style.css      # 레이아웃/스타일
└── README.md
```

<details>
<summary>ASCII 버전(문자 인코딩 이슈가 있을 때 사용)</summary>

```
.
|-- index.html     # 앱 엔트리 (UI)
|-- main.js        # Anchor-CDC 로직 + 시각화 동작
|-- style.css      # 레이아웃/스타일
`-- README.md
```
</details>

---

## 🧩 Features

- **Anchor-CDC 시각화**: 키 시퀀스에서 앵커가 잡히는 지점에만 경계 생성
- **파라미터 실험**: `maskBits`, `minEntries` 등 핵심 파라미터를 즉시 조정
- **해시/앵커 표시**: 키별 해시·앵커 충족 여부를 시각적으로 확인
- **Diff 감쇠 체험**: Tail Append / Contiguous Delete에서 변경 전파 감소 확인
- **브라우저 단일 페이지**: GitHub Pages로 간편 배포

---

## 🧪 Try It (샘플 키)

아래와 같이 붙여넣어 보세요. (한 줄에 한 키)

```
Event_001
Event_002
Event_003
...
Event_040
```

- **Tail Append**: 끝에 `Event_041 ~ Event_050` 추가  
  → 대부분 기존 청크는 유지, 말단 일부만 재형성
- **Contiguous Delete**: `Event_011 ~ Event_020` 삭제  
  → 인접 청크만 영향을 받고, 전체 재분할이 일어나지 않음

---

## 🛠️ Quick Start (Local)

```bash
# 1) Clone
git clone https://github.com/llamear/P4ChunkRunner-WebSim.git
cd P4ChunkRunner-WebSim

# 2) Run locally
#  - 방법 A: index.html을 브라우저로 더블클릭
#  - 방법 B: 간단 서버 (Python 예시)
python -m http.server 8080
# open http://localhost:8080
```

> GitHub Pages는 정적 호스팅이므로 별도 빌드가 필요 없습니다.

---

## 🧠 How it works

Anchor-CDC는 **키의 해시**를 사용해 **앵커(Anchor) 조건**을 만족하는 지점에서만 경계를 세웁니다.

- 해시 예시: FNV-1a 64-bit (구현은 `main.js`)
- 마스크 계산:  
  `mask = (1UL << maskBits) - 1`
- **앵커 조건:**  
  `(hash & mask) == mask  →  경계 형성`
- **minEntries:**  
  지나친 경계 빈도를 방지하기 위한 최소 청크 크기 하한

<details>
<summary><strong>Anchor-CDC vs Sharding 요약</strong></summary>

**Sharding (해시 접두사로 균등 분할)**  
- ✅ 단순, 구현/스케일링 용이  
- ❌ 중간 삽입/삭제/리네임 시 여러 파일로 변경 전파가 확산 → 리뷰/머지 비용↑

**Anchor-CDC (내용 정의 경계)**  
- ✅ 내용(키 시퀀스)에 의해 경계가 안정적으로 결정 → 변경 전파 감쇠  
- ✅ Tail Append/Contiguous Delete에서 교체 파일 수와 변경율 감소  
- ⚠️ 파라미터(`maskBits`, `minEntries`)에 대한 이해 필요
</details>

---

## 🎛️ Parameters

- **maskBits**: 앵커 빈도 제어 (값↑ → 앵커 드묾 → 청크 커짐)
- **minEntries**: 최소 청크 크기 (너무 잘게 쪼개지는 것 방지)

> 일반적으로 `maskBits`는 데이터 길이/변동 패턴에 따라 조절하고,  
> `minEntries`는 팀의 리뷰 단위/머지 정책에 맞춰 정합니다.

---

## 🖼️ Screenshots

- `docs/preview.png` / `docs/preview.gif`에 이미지를 추가하면 위 미리보기가 표시됩니다.
- 캡션 예시:
  - *Figure 1.* 초기 시퀀스에서 앵커(●) 지점에만 경계 형성
  - *Figure 2.* Tail Append 후 말단 청크만 재형성

---

## 🗺️ Roadmap

- [ ] Sharding 비교 뷰(토글)
- [ ] 해시 함수 선택 (FNV-1a / xxHash 등)
- [ ] 난수 키 생성기 & 재현 가능한 Seed
- [ ] 청크 메타데이터 JSON Export

---

## 🤝 Contributing

이슈·아이디어·버그 리포트를 환영합니다.  
PR 시에는 재현 스텝과 스크린샷을 함께 남겨주시면 도움이 됩니다.

---

## 📄 License

루트의 `LICENSE` 파일을 참고하세요.

---

## 🌍 English (Short)

**P4ChunkRunner Web Simulator** visualizes Anchor-based Hash CDC in the browser.  
Boundaries form only at anchor positions (`(hash & mask) == mask`), keeping diffs localized under **Tail Append** and **Contiguous Delete**—unlike hash-prefix sharding that often spreads changes.

- **Live Demo:** https://llamear.github.io/P4ChunkRunner-WebSim/  
- **Files:** `index.html`, `main.js`, `style.css`
