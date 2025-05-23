# **1. 번들링의 개념과 목적**

## 번들링이란 무엇인가?

- 사용자에게 웹애플리케이션을 제공하기 위해 html, css, js로 작성된 여러 코드들을 묶는 과정
- 빌드의 한 과정중에 하나임
- 단순히 묶기만 하는것 보다 압축을 한다고 생각하면 좋음

## 번들링이 필요한 이유 (브라우저 환경, 요청 수 최소화 등)

- 원본 프로그램보다 파일이 작아져 실행속도, 로딩속도가 빨라져 성능 상 이점이 있음
- 압축한 파일이 압축 해제 전까지 파일 조작이 안되는 것 처럼 번들링한 웹앱도 사용자가 임의로 조작이 불가능해 보안적인 측면에서도 좋음
- 번들링을 통해 파일을 묶으면 서버에 요청하는 파일 수가 줄어들어 브라우저 실행 속도를 개선 시킬 수 있음

| 상황 | 설명 |
| --- | --- |
| 번들링 X | `main.js`, `header.js`, `footer.js`, `utils.js`... 수십 개 요청 필요 |
| 번들링 O | 모든 JS를 하나로 묶은 `bundle.js` 1개만 요청 |
- 여러 모듈 시스템 (require, import)을 하나로 통합시켜 브라우저에서 호환성을 확보함

## SPA, MPA에서의 번들링 차이

| 항목 | SPA | MPA |
| --- | --- | --- |
| **초기 로딩 시 번들** | 하나의 큰 JS 번들 (`bundle.js`) 로 시작 | 페이지마다 필요한 JS만 로드 |
| **코드 스플리팅** | 필요 시 라우팅에 따라 분할 로드 (lazy loading 등) | 각 HTML마다 별도 번들 생성 가능 |
| **번들 크기** | 처음 로딩 시 무거울 수 있음 | 페이지마다 작지만 요청이 잦음 |
| **리소스 공유** | 모든 기능이 한 앱에 통합 → 공통 모듈 재사용 | 페이지마다 코드가 분리될 수 있어 중복 가능 |
| **빌드 전략** | 전체 앱을 한 번에 번들링 → 효율적인 캐싱 필요 | 페이지 단위로 독립 번들링 가능 |

# **2. 모듈 시스템의 이해**

## JavaScript 모듈 시스템의 역사

Javascript의 모듈 시스템은 시간에 따라 다양한 환경과 요구 조건에 따라 변화해왔다.

### **전역 스코프 방식 (Global Scope)**

- **시기**: JavaScript 초창기
- **특징**: 모든 코드가 전역 스코프에서 실행되어 변수 충돌과 의존성 관리가 어려움
- **문제점**: 대규모 애플리케이션에서 코드 관리가 복잡해지고, 모듈화가 불가능

### CommonJS

- **시기**: 2009년, 서버 사이드 JavaScript 환경을 위해 등장
- **특징**: `require()`로 모듈을 불러오고, `module.exports`로 모듈을 내보냄
- **사용 환경**: 주로 Node.js에서 사용
- **단점**: 동기식 로딩 방식으로 인해 브라우저 환경에서는 비효율적

```jsx
// math.js
module.exports.add = function(a, b) {
  return a + b;
};

// app.js
const math = require('./math');
console.log(math.add(2, 3)); // 출력: 5
```

### AMD (Asynchronous Module Definition)

- **시기**: 브라우저 환경에서 비동기 모듈 로딩을 위해 등장
- **특징**: `define()` 함수를 사용하여 모듈과 의존성을 정의
- **사용 환경**: RequireJS와 같은 라이브러리에서 사용
- **장점**: 비동기 로딩으로 페이지 로딩 속도를 향상시킬 수 있음
- **단점**: 문법이 복잡하고, ECMAScript 표준이 아님

```jsx
define(['dep1', 'dep2'], function(dep1, dep2) {
  return function() {};
});
```

### **UMD (Universal Module Definition)**

- **시기**: CommonJS와 AMD의 호환성을 위해 등장
- **특징**: 다양한 환경에서 모듈을 사용할 수 있도록 설계
- **사용 환경**: 다양한 환경에서 사용될 수는 있으나 ESM이 나온 후 레거시 환경이나 특정 호환성을 위해 사용
- **장점**: 다양한 환경에서 동일한 모듈 사용 가능
- **단점**: 문법이 복잡하고, 정적 분석이 어려워 최적화 기법을 적용하기 어려움

```jsx
(function(root, factory) {
  if (typeof define === 'function' && define.amd) {
    define([], factory);
  } else if (typeof module === 'object' && module.exports) {
    module.exports = factory();
  } else {
    root.myModule = factory();
  }
}(this, function() {
  return {};
}));
```

### ESM (ECMAScript Modules) ⇒ 유일한 JS 공식 표준 모듈 시스템

- **시기**: 2015년, ECMAScript 6(ES6)에서 표준으로 도입
- **특징**: `import`와 `export` 키워드를 사용하여 모듈을 정의하고 불러옴
- **사용 환경**: 최신 브라우저와 Node.js에서 지원
- **예시**:
    
    ```jsx
    // math.js
    export function add(a, b) {
      return a + b;
    }
    
    // app.js
    import { add } from './math.js';
    console.log(add(2, 3)); // 출력: 5
    ```
    
- **장점**: 정적 분석이 가능하여 트리 셰이킹(Tree Shaking)과 같은 최적화가 용이
- **단점**: CommonJS와의 호환성 문제가 있음

# 3. 번들러의 최적화 기법

### 트리 셰이킹(Tree Shaking)

- Tree Shaking은 사용하지 않는 코드들을 제거하기 위해 Javascipt 컨텍스트에서 일반적으로 사용되는 용어
- 사용되지 않는 코드를 제거해 번들 크기를 줄이는 최적화 기법
- ES6 모듈에서만 효과적, CommonJS 에서는 동적 로딩 특성으로 인해 트리 셰이킹이 어려움

### 사이드 이펙트(Side Effect)

- 모듈이 로드될 때 실행되어 외부 상태를 변경하는 코드를 말함. 
ex) 전역 변수 수정, DOM 조작, 콘솔 출력
- 외부 상태를 변경하는 코드로 사이드이펙트가 발생하는 코드는 트리 셰이킹 대상에서 제외되어야 함.

특정 파일만 사이드 이펙트를 가지는 경우

```json
{
  "sideEffects": ["./src/polyfill.js", "*.css"]
}
```

프로젝트 전체가 사이드 이펙트가 없는 경우

```json
{
  "sideEffects": false}
}
```

# **4. 주요 번들러 소개 및 비교**

### Webpack

2012년에 등장한 가장 널리 사용되는 번들러로, 복잡한 프로젝트에 적합한 높은 유연성과 확장성을 제공

- 특징
    - 플러그인과 로더를 통한 다양한 파일 형식 지원
    - 코드 스플리팅, 트리 셰이킹, HMR 등 다양한 최적화 기능 제공
    - 복잡한 설정으로 인해 초기 학습 곡선이 높을 수 있음
- 사용 사례: 대규모 애플리케이션, 복잡한 빌드 요구사항이 있는 프로젝트

### Vite

2020년에 Vue.js의 창시자인 Evan You가 개발한 번들러로, 개발 환경에서의 빠른 속도와 간단한 설정이 강점

- 특징
    - 개발 시에는 ES 모듈을 활용하여 빠른 HMR과 빠른 서버 시작을 지원
    - 프로덕션 빌드 시에는 Rollup을 사용하여 최적화된 번들 생성 (하이브리드 방식)
    - 간단한 설정과 플러그인 시스템을 통해 다양한 프레임워크와의 통합 용이
- 사용사례: 현대적인 프레임워크를 사용하는 프로젝트, 빠른 개발 환경이 필요한 경우

### Rollup

라이브러리 번들링에 최적화된 번들러로 ES 모듈 중심으로 설계

- 특징
    - 작은 번들 크기와 효율적인 트리 셰이킹 지원
    - 플러그인을 통한 확장성 제공
    - 앱보다는 라이브러리 번들링에 더 적합

### **Parcel**

설정이 거의 필요 없는 제로 설정 번들러, 빠른 프로토타이핑과 소규모 프로젝트에 적합

- 특징
    - 자동 코드 분할, HMR, 트리 셰이킹 등 기본 제공
    - 다양한 파일 형식에 대한 기본 지원
    - 복잡한 설정이 필요한 경우에는 유연성이 떨어질 수 있음
- 사용 사례: 소규모 프로젝트, 빠른 개발이 필요한 경우

# 5. Source Map과 디버깅

### Source Map

- 브라우저가 번들된 코드와 원본 소스 코드 간의 관계를 이해할 수 있도록 매핑 정보를 제공하며, 디버깅을 용이하게 하는 도구
- .map으로 끝나는 파일이 소스맵
- 개발자는 브라우저의 개발자 도구에서 원본 소스를 기반으로 디버깅이 가능함

### 디버깅

- 배포된 코드가 번들링 및 난독화되어 있을 경우, 소스맵을 통해 이러한 오류를 원본 코드의 위치와 매핑하여 문제를 해결 할 수 있음
- 외부 패키지나 모듈에서 발생하는 문제를 분석할 때, 소스 맵이 제공되면 해당 라이브러리의 원본 코드를 확인하고 디버깅 할 수 있음

