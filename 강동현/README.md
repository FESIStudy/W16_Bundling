## 📦 번들링이란?
번들링은 **여러 개의 자바스크립트 파일, CSS, 이미지, 모듈** 등을 **하나 또는 여러 개의 최적화된 파일로 묶는 과정**입니다

### 왜 번들링이 필요할까?
1. 브라우저는 모듈 시스템을 완벽하게 지원하지 않거나 느릴 수 있음 → ESM(ES Module)을 직접 쓰면 성능 저하
1. 수백 개의 파일 요청 = 느린 로딩 → 네트워크 요청 줄이기 위해 묶음 필요
1. 코드 최적화 → 죽은 코드 제거(Tree shaking), 압축(Minification) 등
1. 개발 편의성 → 모듈 단위 개발 후 자동 병합

## 🛠️번들러란?
여러 개의 모듈/파일을 **브라우저가 이해할 수 있는 형태로 묶고 최적화하는 도구**

- 입력: JS, CSS, 이미지 등
- 처리: 트랜스파일(Babel, TypeScript), Tree Shaking, 코드 분할, 압축
- 출력: 하나 또는 여러 개의 **번들 파일**

## 🔍 주요 번들러 비교: Webpack, Vite, ESBuild
|항목|Webpack|Vite|ESBuild|
|-|-|-|-|
|언어 기반|JavaScript|JavaScript|Go|
|번들링 방식|	전통적 번들링 (빌드 시 전체 분석)|개발 서버: ESM 기반|프로덕션 빌드: Rollup 사용|	트랜스파일 + 번들링|
|속도|느린 편 (캐시 필요)|개발: 매우 빠름빌드: Rollup 기반으로 중간|가장 빠름|
|설정|복잡하지만 유연|간단하고 직관적|간단하지만 기능 제한|
|트리 쉐이킹|	지원|	지원|	지원|
|HMR |지원|	지원 (느림)|	매우 빠름	제한적|
|사용 목적|	대규모 프로젝트|	최신 웹앱 개발|	툴 제작, 트랜스파일러 역할|
|대표 사용처|	CRA, Next.js, 기업용 프로젝트|	Vite 프로젝트, Nuxt 3, SvelteKit| Vite 내부 엔진, SWC 대체|

## ⚡ Vite가 빠른 이유
Vite는 **개발 서버**와 **프로덕션 빌드**를 **완전히 다르게 처리**합니다.

**🧪 개발 환경 (Development)**
- **ESM 기반**으로 실행 (즉시 실행 가능)
- import 된 모듈만 실시간 로드
- 트랜스파일만 빠르게 하고 번들링은 생략
- esbuild를 사용하여 TS/Babel 처리를 빠르게 함
> ➡️ 전체 앱을 번들링하지 않고, 필요한 모듈만 로드하기 때문에 개발 속도가 빠름

**🏗️ 빌드 환경 (Production)**
- 내부적으로는 Rollup을 사용하여 전체 번들링
- Tree Shaking, Code Splitting 등을 적용


## 🔁 Vite의 HMR(Hot Module Replacement) 작동 방식
### HMR이란?
**코드 수정 시** 전체 페이지를 새로고침하지 않고 **변경된 부분만 브라우저에 반영**하는 기술
### Webpack HMR vs Vite HMR
|항목|Webpack HMR|Vite HMR|
|-|-|-|
|구조|전체 번들을 다시 평가|**모듈 단위로 변경 감지**|
|속도|느림 (전체 재컴파일 발생)|**즉시 반영됨 (부분 업데이트)**|
|구현 방식|JS로 구현됨|esbuild로 빠르게 트랜스파일|
|캐시 사용|제한적|브라우저 캐시 적극 사용|
### Vite의 동작 과정
1. 소스 파일이 변경되면 Vite는 어떤 모듈이 수정되었는지 감지
1. 변경된 모듈만 브라우저로 다시 보내고 실행
1. 브라우저는 해당 모듈을 갱신하고, 전체 페이지 새로고침 없이 업데이트

> ➡️ 결과적으로, 개발자가 파일을 저장하자마자 브라우저에 실시간 반영됨
> ➡️ 개발 경험이 매우 향상되고 빠름

## 🌲 트리 쉐이킹(Tree Shaking)
### ✅ 개념

**사용되지 않는 코드를 제거**하여 번들 크기를 줄이는 최적화 기법
### ✅ 비유

**나무(tree)** 에서 **사용하지 않는 가지(코드)** 를 **흔들어(shake)** 떨어뜨려 버린다고 해서 Tree Shaking이라는 이름이 붙었어요.

### ✅ 전제 조건
- ES Module (ESM) 형식 (import/export)을 사용할 것
> require() (CommonJS)는 제대로 지원되지 않음
- 번들러가 정적 분석 가능해야 함 (즉, 코드 흐름을 예측할 수 있어야 함)

### ✅ 예시
```js
// util.js
export function add(a, b) {
  return a + b;
}
export function multiply(a, b) {
  return a * b;
}

// main.js
import { add } from './util.js';
console.log(add(2, 3));
```
- 위 코드에서 `multiply()`는 사용되지 않으므로, 번들 시 제거됨.
- 트리 쉐이킹을 지원하는 번들러는 `add`만 포함된 결과를 생성함.
### ✅ 장점
- 번들 크기 감소
- 불필요한 코드 제거로 성능 향상
### ❗ 주의할 점
- 부작용(side effect)이 있는 코드는 제거되지 않을 수 있음
- package.json에서 sideEffects:false 설정이 있어야 확실히 적용됨

## 🔀 코드 스플리팅
### ✅ 개념
**필요할 때만 특정 코드**를 로드하도록 코드를 여러 개의 번들(청크)로 나누는 기법
### ✅ 목적
- **초기 로딩 속도 개선**
- 사용자 요청 시 필요한 코드만 **동적 로드**
### ✅ 방식
1. static import-based splitting
  - 번들러가 import()를 만나면 자동으로 청크 분리
2. Dynamic import
  - 코드 실행 시점에 모듈을 로딩

### ✅ 예시
```js
document.getElementById('btn').addEventListener('click', async () => {
  const { showModal } = await import('./modal.js');
  showModal();
});
```
- 초기 로딩 시 modal.js는 로딩되지 않음
- 사용자가 버튼을 클릭했을 때만 해당 모듈이 로드됨
### ✅ 장점
- 초기 페이지 로딩 속도 향상
- 불필요한 코드 지연 로딩

### 📦 Webpack, Vite 등에서 지원 방식
- Webpack: import() → 자동 코드 스플리팅
- Vite: import() → Rollup을 통해 청크로 분리
- React.lazy, Vue async component 등도 코드 스플리팅 활용

## 📦 Webpack 번들 사이즈 최적화
### ✅ 트리 쉐이킹 활성화
- ESM(import/export)을 사용해야 Webpack이 정적 분석 가능
- Webpack은 production 모드에서 자동으로 
### ✅ 트리 쉐이킹 적용
```js
// webpack.config.js
mode: 'production'
```
```json
//package.json
{
  "sideEffects": false
}
```
> 전역 영향이 있는 코드(ex. polyfill)는 개별적으로 제외 설정 필요

### ✅ 코드 스플리팅
- 자동 코드 스플리팅
```js
// 사용자가 특정 액션을 할 때만 모듈 로딩
button.addEventListener('click', async () => {
  const { heavyFunction } = await import('./heavy.js');
  heavyFunction();
});
```
- Webpack 설정에서 청크 분리
```js
//webpack.config.js
optimization: {
  splitChunks: {
    chunks: 'all', // 모든 모듈 대상으로 코드 스플리팅
  },
}
```
### ✅ 외부 라이브러리 외부화
```js
// webpack.config.js
externals: {
  lodash: '_',
}
```
```html
<!-- HTML에서 CDN으로 직접 로딩 -->
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
```
- 번들에 lodash를 포함하지 않게 되어 크기 감소

### ✅ Tree-shakable 라이브러리만 사용하기
lodash → lodash-es

moment.js → dayjs

chart.js → lightweight chart libs like `uPlot`

### 정적 자원 최적화(이미지, 폰트 등)
- image-webpack-loader, url-loader, file-loader
- SVG를 component로 변환 (@svgr/webpack)

### ✅ Minification (압축)
```js
//webpack.config.js
optimization: {
  minimize: true,
  minimizer: [new TerserPlugin({
    terserOptions: {
      compress: {
        drop_console: true, // console.log 제거
      },
    },
  })],
}
```

### ✅ Bundle Analyzer 사용
```sh
npm install --save-dev webpack-bundle-analyzer
```
```js
//webpack.config.js
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
//...
plugins: [new BundleAnalyzerPlugin()],
```
- 시각적으로 어떤 모듈이 얼마나 큰지 분석 가능

## 정리
### 웹 번들링
> 여러 개의 자바스크립트 파일, CSS, 이미지, 모듈 등을 하나 또는 여러 개의 최적화된 파일로 묶는 과정
### 번들러
> 브라우저가 이해할 수 있는 형태로 묶고 최적화하는 도구
- Webpack: 설정 자유도 높고 플러그인 많음, 느리지만 확장성 최고
- Vite: ESM 기반 개발서버 + Rollup 빌드. 빠르고 간편하며 현대적인 개발 환경
- ESBuild: Go 기반 초고속 빌드 도구. Vite 내부에서도 사용됨

### 트리 쉐이킹
- **불필요한 코드 제거**하여 빌드 시 **번들 사이즈 줄임**
### 코드 스플리팅
-	코드를 나눠서 지연 로딩 및 필요한 경우에만 작동하여 **초기 로딩 속도 향상**

### webpack 번들링 최적화 종류
- Tree shaking
- Code splitting
- Externalize
- Tree-shakable
- 정적 자원 최적화
- Minification
- Bundle Analyzer 사용하여 분석
