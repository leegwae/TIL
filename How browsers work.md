# How browsers work

> 브라우저는 서버에게 렌더링에 필요한 자원을 요청하고, 서버로부터 응답을 받은 자원을 파싱하여 렌더링한다.

**렌더링(rendering)**이란 브라우저가 서버에서 받은 자원들(HTML, XML, PDF, 이미지)을 화면에 그리는 과정 전반을 의미한다. 여기서는 HTML 렌더링을 중심으로 서술한다.

## Main Components of Browser

**브라우저(browser)**는 다음과 같은 요소로 이루어진다.

![Browser components](https://web-dev.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/PgPX6ZMyKSwF6kB8zIhB.png?auto=format)

1. 사용자 인터페이스(User Interface): 브라우저가 요청한 페이지를 그린 창을 제외한 모든 것이다. 주소창, 북마크 메뉴 등이 속한다.
2. 브라우저 엔진(Browser engine): 사용자 인터페이스와 렌더링 엔진 사이의 동작을 제어한다.
3. 렌더링 엔진(Rendering engine): 사용자가 요청한 자원을 화면에 표시한다. 크롬과 사파리는 Webkit 엔진을, 파이어폭스는 Gecko 엔진을 렌더링 엔진으로 사용한다.
4. 통신(Networking): HTTP 요청과 같은 네트워크 호출에 사용된다.
5. UI 백엔드(UI backend): 기본적인 위젯(combo box나 window)을 그릴 때 사용한다.
6. 자바스크립트 인터프리터(JavaScript Interpreter): 자바스크립트 엔진. 자바스크립트 코드를 평가하고 실행한다.
7. 데이터 스토리지(Data storage): 쿠키처럼 로컬에 데이터를 저장한다. 브라우저에서 제공하는 스토리지로 localStorage, IndexedDB, WebSQL, FileSystem이 있다.



## Critical Rendering Path

**CPR(Critical Rendering Path)**은 브라우저가 HTML, CSS, JavaScript를 화면에 픽셀로 나타내기까지의 일련의 과정을 뜻한다. CPR은 다음 네 단계로 나타낼 수 있다.

1. 파싱(Parsing)
2. 렌더 트리(Render Tree) 구축
3. 레이아웃(Layout)
4. 페인팅(Painting)

렌더링 엔진은 더 나은 사용자 경험을 위해 HTML이 모두 파싱될 때까지 기다리지 않고 구축한 렌더 트리부터 배치하고 페인팅한다. 페이지의 모든 요소가 한 번에 그려지는 것이 아니라 시간을 두고 계속 그려지는 것은 이 때문이다.

CPR은 일반적으로 초당 60회 정도의 주기로 계산한다.

## Parsing

### 일반적인 파싱의 개념

문서를 파싱하는 것은 문서를 코드가 이해할 수 있는 구조인 파스 트리(parse tree/(abstract) syntax tree)로 변환하는 것이다.

**파싱(parsing)**은 크게 두 과정으로 나누어 볼 수 있다.

1. **어휘 분석(lexical analyze)** 단계에서는 데이터를 유효한 토큰으로 분해한다. **토큰(token)**이란 의미를 가지는 최소 단위로, 곧 어휘 분석의 단위이다. 즉, 입력 데이터를 토큰의 배열로 만든다.
2. **구문 분석(syntax analyze)** 단계에서는 토큰에 언어의 구문 규칙(syntax rule)을 적용하여 트리로 구조화한다. 즉, 어휘 분석에서 만든 토큰의 배열로 파스 트리를 만든다.

**파서(parser)**는 파싱을 수행한다. 파서는 어떤 파싱 단계를 수행하느냐에 따라 크게 두 종류로 구분할 수 있다.

1. **어휘 분석기(lexical analyzer)**는 파싱에서 어휘 분석 단계를 담당한다. 어휘 분석기를 다음 두 개의 요소가 구성된 것으로 보기도 한다.
   1. **토크나이저(tokenizer)**는 데이터를 토큰으로 분해한다. 보통 공백을 기준으로 쪼갠다.
   2. **렉서(lexer; lexical analyzer)**는 토큰의 유효한 의미를 분석한다.
2. **파서(parser)**는 파싱에서 구문 분석 단계를 담당한다. 파서는 구문 규칙에 따라 문서 구조를 분석하여 파스 트리를 구축한다.

### 일반적인 파싱으로 HTML을 파싱할 수 없는 이유

일반적인 파서로는 HTML을 파싱할 수 없는데 그 이유는 아래와 같다.

첫번째, 브라우저가 HTML 오류에 관용적이다. 오류가 발생해도 브라우저가 수정해준다. 두번째, HTML 파싱이 JavaScript, CSS의 다운로드와 파싱, 실행에 의해 중단될 수 있다. 세번째, 스크립트가 `document.write()`를 사용하는 경우 DOM에 변경이 생기며 이 경우 처음부터 HTML을 다시 파싱한다.

### HTML Parsing

![DOM construction process](https://web-dev.imgix.net/image/C47gYyWYVMMhDmtYSLOWazuyePF2/cSL20piziX7XLekCPCuD.png?auto=format)

HTML을 파싱하는 것은 HTML을 DOM 트리로 변환하는 것이다. DOM 트리는 DOM(Document Object Model) 노드로 구성되어있다. HTML5 명세에 따르면 HTML 파싱 알고리즘은 토큰화 단계와 트리 구축 단계로 구성된다.

1. 브라우저는 서버로부터 응답받은 바이트 데이터를 응답 헤더의 `Content-Type`에 지정된 인코딩 방식을 사용하여 문자열로 변환한다. HTML의 경우, 대개 `Content-Type: text/html; charset=utf-8`으로 명시될 것이다. (로컬 디스크로부터 불러오는 경우 `.html` 확장자로 HTML 문서라고 판단하고 `<meta>` 태그의 `charset` 어트리뷰트에 지정된 인코딩 방식을 사용한다. [출처](https://stackoverflow.com/questions/4696499/meta-charset-utf-8-vs-meta-http-equiv-content-type))

2. **토큰화(tokenization)**는 파싱에서 어휘 분석 단계이다. HTML 문서를 나타내는 문자열을 HTML 토큰으로 분해한다. HTML 토큰에는 시작 태그(start tag), 종료 태그(end tag), 속성 이름과 속성 값들이 있다.
   토크나이저는 토큰화를 수행하는 파서이다. 토크나이저는 토큰을 인식하여 트리 생성자에게 넘기는데 이 과정은 HTML 문서가 끝날 때까지 반복된다.

3. **트리 구축(tree construction)**는 파싱에서 구문 분석 단계이다. HTML 토큰들을 노드(Node)로 변환하고, 노드들을 구성하여 DOM 트리를 만든다.

   트리 생성자(tree constructor)는 트리 구축을 수행하는 파서이다. 트리 생성자는 토크나이저로부터 전달 받은 일련의 토큰을 처리하여 DOM 트리에 삽입하는데 이 과정은 마지막 토큰을 받을 때까지 반복된다.

<img src="https://html.spec.whatwg.org/images/parsing-model-overview.svg" alt="img" style="width: 50%;" />

일반적으로 네트워크 요청으로 가져온 HTML 문서는 위와 같은 알고리즘으로 파싱된다. 여기서 *Script Execution* 분기는 다음을 뜻한다. 랜더링 엔진은 HTML을 파싱하는 중 클래식 스크립트 태그(`<script>`)가 나타나면 HTML 파싱은 멈춘다. `src` 속성이 지정되어있는 경우 해당 스크립트를 다운로드한다. 스크립트의 파싱과 실행이 끝나면 HTML 파싱이 재개된다. 이때 스크립트가 `document.write()`와 같은 API를 사용하면 DOM 트리가 수정되므로 이 경우엔 아예 HTML을 처음부터 다시 파싱한다. 지연 스크립트(`<script defer>`)를 사용하면 스크립트의 파싱과 실행 시점을 조절할 수 있다.

### CSS Parsing

![DOM tree](https://web-dev.imgix.net/image/C47gYyWYVMMhDmtYSLOWazuyePF2/keK3wDv9k2KzJA9QubFx.png?auto=format)

CSS를 파싱하는 것은 CSS를 CSSOM 트리로 변환하는 것이다. CSSOM 트리는 CSSOM(CSS Object Model) 노드로 이루어져있다.

CSS 역시 HTML과 같은 프로세스를 사용한다. 바이트는 문자로, 문자는 토큰으로, 토큰은 노드로 변환되고 노드가 모여 CSSOM 트리를 구성한다.

렌더링 엔진은 HTML을 파싱하는 중 `<link>` 태그나 `<style>` 태그를 만나면 HTML 파싱은 멈춘다. `<link>` 태그의 경우 `href` 속성에 지정된 CSS를 다운로드받는다. 이후 CSS의 파싱이 완료되면 HTML 파싱이 재개된다.

## Render tree construction

**렌더 트리(Render tree)**는 DOM 트리와 CSSOM 트리를 결합한 트리로 화면에 렌더링되는 노드만으로 구성된다. Webkit은 렌더 트리의 노드를 렌더러(renderer) 또는 렌더 객체(render object)라고 부른다. Gecko에서는 프레임 트리(frame tree)라고 하며 그 노드를 프레임(frame)이라고 한다.

렌더링 트리를 구성하는 방법은 아래와 같다.

1. DOM 트리의 루트부터 각 노드를 순회한다. `<script>`, `<meta>`, `<head>` 등의 태그로 만든 노드와 `"display: none;"`과 같이 CSS로 숨겨진 노드는 렌더 트리에서 생략된다.
2. 각 노드에 적절히 대응하는 CSSOM 규칙을 찾아 적용한다.
3. 노드를 컨텐츠와 계산된 스타일과 함께 내보낸다.

고로 DOM 트리와 렌더 트리는 완벽하게 일치하지 않는다.

## Layout

렌더러의 크기와 위치를 계산하는 것을 Webkit에서는 **레이아웃(layout),** Gecko에서는 리플로우(reflow)라고 한다. 레이아웃은 루트 렌더러부터 재귀적으로 실행된다. 레이아웃은 일반적으로 왼쪽에서 오른쪽으로 또는 위에서 아래로 흐른다.

레이아웃은 렌더 트리가 수정될 때마다 발생한다. 노드 수가 많을수록 레이아웃이 더 오래 걸린다. 레이아웃이 병목될 경우 스크롤 또는 애니메이션에 버벅거림이 발생할 수 있다.

## Painting

**페인팅(painting)**은 렌더 트리의 노드를 실제 화면에 픽셀로 그리는 것이다. 처음으로 로드될 때는 전체 화면을 그리고 그 이후부터는 최적화로 변경이 필요한 영역만 다시 그린다.

페인팅은 매우 빠른 프로세스이지만 그래도 레이아웃과 더불어 소요 시간을 고려해야한다.

## Rerendering

**리렌더링(rerendering)**은 리플로우와 리페인트가 실행되는 것이다. 리플로우(reflow)는 레이아웃이 변경된 경우 레이아웃을 다시 계산하는 것을 말한다. 리페인트(repaint)는 변경된 렌더 트리를 다시 페인트를 하는 것을 말한다. 리플로우 이후 리페인트가 발생하나 레이아웃에 변경이 없다면 리페인트만 실행된다.

리렌더링은 다음과 같은 이유로 다시 발생할 수 있다. 

- 자바스크립트에 의한 노드 추가 또는 삭제
- 브라우저 창의 리사이징으로 인한 뷰포트 크기 변경
- HTML 요소의 레이아웃 변경을 발생시키는 스타일 변경

리렌더링은 비용이 많이 드므로 리렌더링이 빈번히 발생하지 않도록 주의한다. 리렌더링 횟수를 줄이기 위해 React, Vue와 같은 라이브러리는 Virtual DOM을 사용하여 변경 사항을 한 번에 묶어 반영한다.

## JavaScript Parsing

기본적으로 렌더링 엔진은 HTML 파싱 도중 `<script>` 태그를 만나면 HTML 파싱을 중단하고 자바스크립트 엔진에 제어권을 넘긴다. 자바스크립트 엔진은 자바스크립트 코드를 파싱하고 실행한 후 다시 렌더링 엔진으로 제어권을 넘긴다. 렌더링 엔진은 중단 지점부터 다시 HTML 파싱을 재개한다.

자바스크립트 엔진은 다음과 같은 단계로 자바스크립트 코드를 파싱한다.

1. 토큰화는 파싱에서 어휘 분석 단계이다. 자바스크립트 코드를 나타내는 문자열을 자바스크립트 토큰으로 분해한다. 자바스크립트 토큰에는 예약어, 식별자, 리터럴, 구분자 등이 있다.
2. 파싱은 파싱에서 구문 분석 단계이다. 토큰을 구문 분석하여 AST(Abstract Syntax Tree)를 만든다. 

이후 자바스크립트 엔진은 다음과 같은 단계로 자바스크립트 코드를 실행한다.

1. 바이트코드 변환: AST를 바이트코드로 변환한다.
2. 자바스크립트 실행: 자바스크립트 인터프리터가 자바스크립트를 실행한다.

V8 자바스크립트 엔진은 자주 사용하는 코드를 TurboFan 컴파일러를 사용해 최적화된 머신 코드로 컴파일하여 성능을 최적화한다.

### 파싱과 실행 시점 정하기

![img](https://v8.dev/_img/modules/async-defer.svg)

클래식 스크립트의 파싱과 실행은 HTML의 파싱을 블로킹한다. `src`로 자바스크립트 파일을 로드하는 `<script>` 태그에 한해서 속성을 추가해 스크립트의 파싱과 실행 시점을 정할 수 있다.

- 클래식 스크립트: 스크립트 파싱이 HTML을 블로킹한다. 스크립트 파싱이 완료되자마자 스크립트를 실행한다.
- `async` 스크립트: 스크립트 파싱이 HTML 파싱을 블로킹하지 않는다. 스크립트 파싱이 완료되자마자 스크립트를 실행한다.
- `defer` 스크립트: 스크립트 파싱이 HTML 파싱을 블로킹하지 않는다. HTML 파싱이 완료되자마자 스크립트를 실행한다. (HTML 파싱이 완료되고 DOM 트리가 완성된 즉시 발생하는 `DOMContetLoaded` 이벤트는 `defer` 스크립트 실행 이후 발생한다.)

## 참고

- [Kruno: How browsers work | JSUnconf 2017](https://youtu.be/0IsQqJ7pwhw)
- https://github.com/alex/what-happens-when
- https://web.dev/howbrowserswork/
- https://d2.naver.com/helloworld/59361
- https://html.spec.whatwg.org/multipage/parsing.html#parsing
- [Critical rendering path - Crash course on web performance (Fluent 2013) ](https://youtu.be/PkOBnYxqj3k)
- https://web.dev/critical-rendering-path/
- https://m.post.naver.com/viewer/postView.nhn?volumeNo=8431285&memberNo=34176766
- https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path
- https://wormwlrm.github.io/2021/03/27/How-browsers-work.html#%EA%B0%80%EC%83%81-dom
- 모던 자바스크립트 Deep Dive 38장 브라우저의 렌더링 과정
- https://v8.dev/features/modules#defer