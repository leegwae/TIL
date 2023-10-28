# How Webpack works

Webpack은 여러 개의 모듈과 의존성을 브라우저에서 실행할 수 있는 단일 파일로 번들하는 JavaScript 번들러이다. Webpack이 어떻게 동작하는지 알아보자.

## Webpack module

Webpack에서 모든 파일은 모듈이다. Webpack은 자바스크립트로 작성된 컴파일러로, 기본적으로 자바스크립트와 JSON 파일만 이해할 수 있으므로 다른 종류의 파일을 Webpack 모듈(자바스크립트)로 변환한다. Webpack 모듈은 다음과 같은 방법으로 의존성을 나타낼 수 있다. [출처](https://webpack.js.org/concepts/modules/)

- ES6 import문
- CJS `require()` 문
- AMD `define`과 `require` 문
- css/sass/less의 `@import`문
- 스타일시트의 `url(...)`
- HTML `<img src='...'>`

Webpack은 진입점으로 지정된 파일부터 시작하여 의존성 그래프를 재귀적으로 빌드한 후 최종적으로 하나의 파일로 번들한다.

## Webpack lifecycle

Webpack은 내부적으로 플러그인을 기반으로 한 컴파일러로, 파일을 번들링하기 위한 라이프사이클을 가진다. 라이프사이클의 각 단계는 이벤트라고 할 수 있고, 플러그인은 이벤트 리스너라고 할 수 있다.

Webpack 라이프사이클은 다섯 단계로 나누어볼 수 있다.

1. 컴파일 시작(compilation-start)
2. 해석(resolve)
3. 파싱(parse)
4. 번들링(bundle)
5. 컴파일 완료(compilation-end)

### 컴파일 시작

**컴파일러(compiler)**가 설정 파일에서 진입점을 찾아 컴파일을 시작한다.

### 해석

**해석기(resolver)**가 상대 경로를 절대 경로로 변환한다. **모듈 팩토리(module factory)**가 해석기로부터 파일에 대한 정보를 받아 **모듈 객체(module object)**를 만든다. 모듈 객체는 파일의 크기, 절대 경로, 종류, id 등의 정보를 포함한다. 그리고 나서 컴파일러는 파일의 종류에 적합한 트랜스파일러를 찾고, **트랜스파일러(transpiler)**는 코드를 브라우저가 읽을 수 있는 형태로 변환한다.

### 파싱

**파서(parser)**는 트랜스파일된 파일을 파싱하고 찾은 `require`나 `import`문을 사용해 모듈 팩토리부터 받은 모듈 객체의 의존성 정보를 갱신한다.

```javascript
{
  id: 0,
  absolutePath: '/path',
  fileType: '.jsx',
  dependency: [{ id1, relativePath }, { id2, relativePath }]
}
```

컴파일러는 재귀적으로 이러한 파싱 과정을 반복하여 모듈 객체로 만들어진 **의존성 그래프(dependency graph)**를 만든다.

### 번들링

컴파일러는 의존성 그래프를 위상 정렬한 후, 청크(chunk)로 변환하고 다시 청크를 병합하여 **번들 파일(bundle file)**을 만든다.

### 컴파일-완료

번들 파일에 후처리(예: 경량화나 트리 쉐이킹)을 수행한다.

## Core concepts

[Webpack의 주요 개념](https://webpack.js.org/concepts/)은 다음과 같다. 이들은 Webpack 설정 파일에서 자세히 설정할 수 있다.

- entry: 의존성 그래프를 빌드할 진입점으로 사용할 모듈이다.
- output: 컴파일의 결과물인 번들 파일이다.
- loader: 주로 트랜스파일러의 역할을 수행한다. 번들링 과정 중에서 자바스크립트가 아닌 파일을 모듈로 변환한다.
- plugin: 주로 번들 파일에 대해 전처리기나 후처리기의 역할을 수행한다. 번들 과정에서의 로깅, 번들 파일에 대한 최적화(경량화, 트리 쉐이킹), 에셋 관리 및 환경 변수 주입 등을 수행한다.

## 어떻게 할 것인가?

Webpack은 자바스크립트뿐만 아니라 다양한 종류의 의존성을 번들할 수 있다. 개발 서버 및 HMR, 번들 파일에 대한 최적화(트리 쉐이킹, 지연 로딩, 코드 스플리팅, 경량화) 등을 지원한다. 오랫동안 사용되어온만큼 생태계도 크고 안정적이다.

그러나 설정이 번거롭고, ESM 출력을 지원하지 않는다는 단점이 있다. 인터프리터/싱글 스레드 자바스크립트로 작성되었으므로 다른 언어에 비해 느리다. 또한 변경 사항이 생기면 진입점부터 컴파일을 전부 새로 시작해야하므로 빌드 시간이 느리다. 이런 관계로 `esbuild-loader`를 사용하여 자바스크립트를 트랜스파일하는 시간을 줄이거나, 아예 저수준 멀티스레드 언어를 사용하는 번들러 혹은 bundless 기반의 번들러(ESM 기반 개발 서버 사용; 개발 환경에서는 개발 서버가 번들하지 않고 브라우저에 모듈을 제공)로 갈아탈 수도 있다.



## 참고

- https://blog.lyearn.com/how-webpack-works-236f8cc43ae7
- https://youtu.be/H3g0BdyVVxA
- https://craigtaub.dev/under-the-hood-of-web-bundlers
- https://webpack.js.org/
- https://ui.toast.com/fe-guide/ko_BUNDLER
- https://fe-developers.kakaoent.com/2022/220707-webpack-esbuild-loader/
- https://medium.com/naver-place-dev/javascript-%EB%B2%88%EB%93%A4%EB%9F%AC%EC%9D%98-%EC%9D%B4%ED%95%B4-4-webpack-%EB%B0%8F-%EB%8B%A4%EB%A5%B8-%EB%B2%88%EB%93%A4%EB%9F%AC%EB%93%A4-e5158e94ef60
- https://www.alibabacloud.com/blog/is-webpack-packaging-too-slow-try-the-bundleless-mode_597148

