### **1. 번들링의 개념과 목적**

* **번들링이란?**

  * 여러 개의 JS, CSS, 이미지 파일 등을 하나 혹은 소수의 파일로 묶는 작업
  * 보통 `main.js`, `vendor.js`, `style.css` 등으로 출력

* **왜 필요한가?**

  * HTTP 요청 수 최소화 → 초기 로딩 성능 향상
  * 모듈 시스템을 지원하지 않는 브라우저 호환
  * 코드 압축, 난독화 등 최적화를 위한 사전 작업

* **SPA vs MPA에서의 차이**

  * SPA: 초기 번들 용량이 크기 때문에 `code splitting`, `lazy loading` 필수
  * MPA: 페이지별 분리된 번들 생성 → 페이지 진입마다 다른 자원 로딩

---

### **2. 모듈 시스템의 이해**

* **CommonJS**: `require()` 기반, Node.js용

* **AMD**: 브라우저 비동기 로딩용

* **UMD**: CommonJS + AMD 통합

* **ESM (ES Modules)**: `import/export` 사용, 브라우저도 지원

* **ESM의 등장 배경**

  * JS 표준으로 모듈을 통합 → 브라우저 네이티브 지원
  * Vite처럼 번들 없는 개발환경 가능

* **Tree Shaking이란?**

  * 사용되지 않는 코드를 제거하는 최적화 기법 (ESM 기반에서만 동작)
  * sideEffects: `package.json`에서 명시 필요

---

### **3. 주요 번들러 소개 및 비교**

| 번들러         | 특징                     | 적합한 상황              |
| ----------- | ---------------------- | ------------------- |
| **Webpack** | 설정 자유도 높고 생태계 풍부       | 대규모, 커스텀 빌드         |
| **Vite**    | 빠른 개발 서버, Rollup 기반 빌드 | React, Vue 프로젝트에 최적 |
| **Rollup**  | 라이브러리 번들링 최적           | npm 라이브러리 제작 시      |
| **Parcel**  | 설정 없이 시작 가능            | 빠른 프로토타입 제작         |

* **추천 조합 예시**

  * React + TypeScript → Vite, Webpack
  * 라이브러리 → Rollup
  * 학습/소규모 → Parcel

---

### **4. Webpack의 구조와 동작 원리**

* **Entry / Output**: 진입점 파일과 출력 파일 정의
* **Loaders**: `.js`, `.ts`, `.css`, `.svg` 등 파일 처리
* **Plugins**: 번들링 외 작업 (압축, HTML 생성 등)
* **Module Resolution**: `resolve.alias`, `extensions`
* **DevServer / HMR**: 개발 중 빠른 핫 리로드
* **Code Splitting**: `import()`로 lazy load
* **Mode 설정 차이**

  * dev: 소스맵, 빠른 빌드, 미압축
  * prod: 압축, 최적화 적용

---

### **5. Vite의 구조와 Webpack과의 차이**

* **Vite는 번들러인가?**

  * 개발 시: `native ESM` 기반 dev server
  * 빌드 시: Rollup 사용 → 진짜 번들러는 Rollup

* **장점**

  * 즉시 서버 시작 (pre-bundling)
  * 빠른 HMR (모듈 단위 변경만 반영)
  * 파일 기반 routing 등 직관적

* **핵심 개념**

  * `optimizeDeps`: ESM 기반 dependency 사전 번들
  * `pre-bundling`: esbuild로 의존성 한 번에 처리

---

### **6. 번들 크기 최적화 전략**

* **Tree Shaking 적용법**

  * `sideEffects` 설정, named import 사용

* **Dynamic import & Code Splitting**

  ```ts
  const MyComp = React.lazy(() => import('./MyComp'));
  ```

* **Lazy Load, prefetch/preload**

  ```html
  <link rel="preload" href="/main.js" as="script" />
  <link rel="prefetch" href="/next.js" as="script" />
  ```

* **External 설정 (CDN 활용)**

  * Webpack externals / Vite optimizeDeps.exclude

* **Asset 최적화**

  * WebP, AVIF 등 이미지 포맷
  * asset modules (`type: asset/resource`)

---

### **7. Source Map과 디버깅**

* **Source Map이란?**

  * 요약 : 번들된 코드 → 원본 코드로 매핑
  * 번들된/압축된 JS 코드와 원본 소스 코드(TypeScript, ES6, JSX 등)를 매핑(mapping) 해주는 파일
  * 브라우저 개발자 도구에서 디버깅 시 원래 작성한 코드로 보이게 해주는 핵심 기술
  * 예: main.js → main.js.map → main.js에서 발생한 에러를 실제 TS/JSX 위치로 추적 가능

* **devtool 옵션 예시**

  * dev: `cheap-module-source-map`
  * prod: `hidden-source-map`

* **배포 전략**

  * 요약 : source map을 서버에 따로 업로드하고 사용자에게는 제공 X
  * 보안 이슈: 소스맵을 배포하면 내부 로직, 주석, 변수명이 노출될 수 있음
  * 전략 1: hidden-source-map 사용
    * 브라우저에서는 보이지 않지만, 오류 추적 도구(Sentry 등)에서는 활용 가능
  * 전략 2: 소스맵 파일은 서버에 업로드하되, 접근 경로를 보호
    * .map 파일은 S3 등 외부 저장소에 업로드
    * 접근은 인증된 툴(Sentry, Datadog 등)만 가능하게 제한
  * 전략 3: nosources-source-map + 로컬에 원본 저장
    * 예외적으로 원본 코드는 내부 개발자만 추적 가능

---

### **8. Babel, SWC, esbuild와의 관계**

* **트랜스파일링 vs 번들링**

  * 트랜스파일: 최신 JS → 브라우저 호환 JS
  * 번들링: 여러 파일 → 하나로 병합

* **Babel**

  * 유연하고 강력한 플러그인 기반, 다소 느림

* **SWC / esbuild**

  * Rust / Go 기반 → 초고속 빌드
  * Vite, Storybook, Next.js 등에 도입 중

---

### **9. 실무 세팅 예시 (React 기준)**

* **Webpack**

  * `babel-loader`, `ts-loader`, `html-webpack-plugin`, `MiniCssExtractPlugin`
  * `.env` 파일은 `dotenv-webpack` 등으로 로드

* **Vite**

  * `vite.config.ts`에서 `define`, `resolve.alias`, `plugins` 설정
  * Tailwind + PostCSS 조합도 간단

* **빌드 최적화**

  * Webpack: `TerserPlugin`, `minimize: true`
  * Vite: `build.minify: 'terser'`
