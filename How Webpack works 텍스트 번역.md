# How Webpack works? 텍스트 번역

> https://blog.lyearn.com/how-webpack-works-236f8cc43ae7 번역

## How webpack works?

Webpack is an **event-driven plugin-based compiler.** That means, webpack has a life cycle for bundling the files and each life-cycle step can be imagined as an event. We can add a plugin that will listen to these different events and act accordingly. The default functionalities can also be inserted through plugins i.e. there will be some default plugins handling some core functionalities.

> Webpack은 **이벤트 주도(event-driven) 플러그인-기반(plugin-based) 컴파일러**다. Webpack은 파일을 번들링하는 lifecycle을 가지며 lifecycle의 각 단계는 이벤트라고 생각할 수 있다. 이런 이벤트를 수신하고 이벤트에 따라 동작하는 플러그인을 추가할 수 있다. 기본적인 기능 또한 플러그인을 통해 삽입될 수 있는데, 핵심 기능을 다루는 기본적인 플러그인이 예시에 속한다.

Let’s say the life cycle has these 5 methods:

> compilation-start → resolve → parse → bundle → compilation-end

> lifecycle은 5가지 단계로 나누어볼 수 있다.
>
> 
>
> > 컴파일-시작→모듈 해석→파싱→번들링→컴파일-완료

The plugins can be integrated to listen to these events and do the operation on the source code files. There will be some default plugins integrated to perform the core functionalities.

> 플러그인은 이 이벤트들을 수신하도록 통합될 수 있으며 소스 코드 파일에 연산을 수행한다. 핵심 기능을 수행하기 위해 통합된 기본 플러그인들이 있다.

Any plugin-based architecture can be designed similarly, it has a life cycle for any particular feature, and custom event handlers(which can be imagined as plugins) can be added to act upon at different steps of the life cycle.

> 모든 플러그인-기반 아키텍처는 비슷하게 설계되었다. 특정 기능을 위한 lifecycle이 존재하며, lifecycle의 각 단계에서 동작할 커스텀 이벤트 핸들러(플러그인으로 생각할 수 있다)를 추가할 수 있다.

Let’s now go through the webpack life cycle.

> 이제 webpack lifecycle에 대해 알아보자.

## What happens under the hood in webpack?

Let’s understand some keywords and then connect them to form a whole story.

> 몇몇 키워드를 이해하고 이것들로 전체 이야기를 구성해보자.

- **Compiler:** Just like a normal compiler, it will be **starting and stopping points** in the webpack life cycle. It can be imagined as a **central dispatcher of events.**

  > **컴파일러**: 일반적인 컴파일러처럼 webpack lifecycle에서 시작점이나 종료점이다. **이벤트의 중앙 디스패처**라고 할 수 있다.

- **Compilation AKA The Dependency Graph:** It is like the brain. Through this webpack understands which sources you are using in your code-base. It contains the **dependency graph traversal algorithm**. It is created by the compiler.

  > **컴파일 결과물(=의존성 그래프)**: 뇌와 같다. webpack은 이것을 통해 어떤 소스 코드가 코드베이스에서 사용되는지 이해한다. 이것은 **의존성 그래프 순회 알고리즘**을 포함한다. 또한 컴파일러에 의해 생성된다.

- **Resolver:** It converts the **partial path to the absolute path**. This will be used to check if some file exists and if it does, give us some information.

  > **해석기**: 이것은 **부분 경로를 절대 경로로 변환**한다. 파일이 존재하는지 확인하고, 그렇다면 정보를 제공하는데 사용한다.

- **Module Factory:** Takes successfully resolved request by resolver and creates a **module object** with source files and some information received by the resolver.

  > **모듈 팩토리**: 해석기로부터 성공적으로 해석된 요청을 받고 소스 파일과 해석기로부터 받은 정보들을 사용하여 **모듈 객체**를 생성한다.

- **Parser:** Takes a module object and turns in AST(a tree representation of the source code) to parse. Find all *requires* and *imports* and make a tree to parse for bundling.

  > **파서**: 모듈 객체를 받아 AST(소스 코드의 트리 표현)으로 변환하여 파싱한다. 모든 require와 import를 찾고 번들링하기 위해 파싱할 트리를 만든다.

- **Templates:** It is used for **data binding for the dependency graph**. It binds the tree object to the actual code in the module.

  > **템플릿**: 이것은 **의존성 그래프에 데이터 바인딩할 때 사용**된다. 이것은 트리 객체를 모듈의 실제 코드와 바인딩한다.

![Webpack은 어떻게 그래프를 빌드하는가](https://miro.medium.com/v2/resize:fit:1050/0*-SXPGmftK_urMG5_.png)

(출처: Webpack의 공동-저자인 Sean Larkin가 Munich에서 열린 JS Kongress의 오프닝 키노트에서 사용했던 이미지이다.)

First of all, webpack will look for the **configuration file** and look for the **entry point** mentioned. The [compiler](https://github.com/webpack/webpack/blob/main/lib/Compiler.js) will start the [compilation](https://github.com/webpack/webpack/blob/main/lib/Compilation.js) from that file. A relative file path will be sent to the [resolver](https://github.com/webpack/webpack/blob/main/lib/ResolverFactory.js). The resolver will convert the **relative path** to the **absolute path**. That request will go to the module factory. [Module factory](https://github.com/webpack/webpack/blob/main/lib/ModuleFactory.js) will create a **module object** will some more information like the type of the file, size, absolute path, assigned id, etc.

> 우선, Webpack은 **설정 파일**을 찾고 적혀있는 **진입점**을 찾는다. 상대 경로는 해석기로 보내진다. 해석기는 상대 경로를 절대 경로로 바꾼다. 이 요청은 모듈 팩토리로 보내진다. 모듈 팩토리는 **모듈 객체**를 만드는데, 모듈 객체는 파일의 종류, 크기, 절대 경로, 할당된 id와 같은 정보를 가진다.

Now, according to the file type mentioned in the module object, the compiler will look for the **transpilers** to convert the code into the **browser-readable format**. After converting the code, the [parser](https://github.com/webpack/webpack/blob/main/lib/Parser.js) will **parse the file** and look for the ‘require’ or ‘import’ statements and update the object with the **dependency information** like below:

> 이제, 모듈 객체에 적혀있는 파일의 종류에 따라 컴파일러는 코드를 **브라우저가 읽을 수 있는 형태**로 변환하는 **트랜스파일러**를 찾는다. 코드를 변환한 후에는 파서가 **파일을 파싱**하고 'require'나 'import'문을 찾아 객체에 아래와 같은 **의존성 정보**를 갱신할 것이다.

```json
// example module object
// 예시 모듈 객체
{
  id: 0,
  absolutePath: '/path',
  fileType: '.jsx',
  dependency: [{ id1, relativePath }, { id2, relativePath }]
  // some more information
  // 또다른 정보
}
```

The compiler will parse all the files **recursively** in this manner to finally form a complete [**dependency graph**](https://github.com/webpack/webpack/blob/main/lib/Dependency.js) **of module objects**. Hash-map will also be maintained between the file id, absolute path, and parsed file.

After building the dependency graph, the compiler will **topologically sort** all dependencies.

> ***Topological sorting\****: Topological sorting for Directed Acyclic Graph (DAG) is a linear ordering of vertices such that for every directed edge u v, vertex u comes before v in the ordering.*

> 컴파일러는 위와 같은 방법으로 모든 파일을 **재귀적으로** 파싱하여 **모듈 객체로 이루어진 완전한 의존성 그래프**를 만들 것이다. 파일 id, 절대 경로, 그리고 파싱된 파일 간의 해시-맵도 유지된다.
>
> 의존성 그래프를 빌딩한 후에 컴파일러는 모든 의존성을 **위상 정렬**한다.
>
> > **위상 정렬**: 사이클 없는 방향 그래프(DAG; Directed Acyclic Graph)에 대한 위상 정렬은 정점을 선형적으로 나열하는 것이다. 모든 방향 있는 간선 `<u, v>`에 대하여 정점 `u`가 `v` 앞에 나열된다.

Then webpack [**merges**](https://github.com/webpack/webpack/blob/main/lib/Template.js) all these topologically sorted dependencies with the help of a [**maintained hash-map**](https://github.com/webpack/webpack/blob/main/lib/ChunkGraph.js) to make a **bundle file**. **Minification** or **Tree shaking** can be done on these bundle files now as **post-processing** measures through an event listener which is also called a plugin. That’s it! Bundler has done his job in these simple steps :)

Let’s dry-run the above bundling process for [the following code](https://github.com/KhushilMistry/webpack-testing):

> 그러면 Webpack은 위상 정렬된 모든 의존성을 **해시-맵**을 사용하여 **병합하고** **번들 파일로 만든다**. **경량화**나 **트리 쉐이킹**은 플러그인이라고 불리우는 이벤트 리스너를 통하여 이 번들 파일에 후처리로 이루어진다. 이게 끝이다! 번들러는 이렇게 간단한 과정을 통해 자기 일을 끝마쳤다.
>
> 다음 코드에 위의 번들렁 과정을 실행해보겠다.

```js
// cat.js ES Module
export default "cat";

// bar.js CommonJS
const cat = require("./cat");
const bar = "bar" + cat;
module.exports = bar;

// foo.js ES Module
import catString from './cat';
const fooString = catString + "foo";
export default fooString;

// index.js - entry point
import fooString from './foo';
import barString from './bar';
import './tree.jpeg';
console.log(fooString, barString);
```

*index.js* is an entry for the bundler. The compiler will start the process with this information. It will convert the relative path into the absolute path and make a module object. We don’t need any transpiler here as it is a simple javascript file. Now parser will start parsing the *index.js* file. It contains 3 import statements, so the final object module created for *index.js* will look like this.

> *index.js*는 번들러의 진입점이다. 컴파일러는 이 정보로 과정을 시작할 것이다. 컴파일러는 상대 경로를 절대 경로로 변환하고 모듈 객체를 만든다. 위 코드는 간단한 자바스크립트 파일이기 때문에 다른 트랜스파일러를 필요로 하지 않는다(역주: `.jsx`와 같은 문법은 `babel-loader`과 같은 별도의 트랜스파일러를 사용하여 트랜스파일해야한다는 뜻이다.) 이제 파서가 <u>index.js</u> 파일을 파싱하기 시작한다. 이 파일은 3개의 import문을 가지고 있어 *index.js*에 대해 만들어진 최종 모듈 객체는 다음과 같다.

```json
// example module object for index.js
// index.js에 대한 예시 모듈 객체
{
  "id":0,
  "absolutePath":"$home/index.js",
  "bundledFile": "bundledFile0.js",
  "fileType":".js",
  "dependency":[
    { "id": 1, "path": "./foo" },
    { "id": 2, "path": "./bar" },
    { "id": 3, "path": "./tree.jpeg"}
  ]
}
```

Webpack will now start compiling the dependencies of *index.js*. Webpack will recursively compile all the files until there is no dependency left for any file and form the following dependency graph. Here *tree.jpeg* will require a file loader/transpiler as *NodeJS* can’t understand it.

> Webpack은 이제 <u>index.js</u>의 의존성을 컴파일하기 시작한다. Webpack은 어떤 파일도 의존성이 남아있지 않을 때까지 재귀적으로 모든 파일을 컴파일하고 아래와 같은 의존성 그래프를 만든다. 이 <u>tree.jpeg</u>는 Node.js가 이해할 수 없으므로 파일 로더/트랜스파일러를 요한다.

![의존성 그래프](https://miro.medium.com/v2/resize:fit:1050/0*dFlXoP7tRCJBUOZA.png)

(의존성 그래프이다.)

It will sort all the module objects in topological order and merge their converted chunks to form a [bundle file](https://github.com/KhushilMistry/webpack-testing/blob/master/build/bundle.js) for browsers to run directly.

> Webpack은 모든 모듈 객체를 위상 정렬로 나열하고, 이것들이 변환된 청크를 병합하여 브라우저에서 바로 실행할 수 있는 번들 파일로 만든다.

![위상 정렬된 의존성 그래프](https://miro.medium.com/v2/resize:fit:1050/0*b94HnwrLoJMs8GY_.png)

(위상 정렬된 의존성 그래프이다.)

## Configuration options in webpack

These are the basic configuration options that webpack gives:

> Webpack이 제공하는 기본 설정 옵션들은 다음과 같다:

- **Entry**: We all know webpack makes a dependency graph and the starting point of this graph is known as the entry or entry point. From the starting point of the dependency graph, it will follow all the dependencies to know what it has to bundle.

  > **진입점**: Webpack이 의존성 그래프를 만들고, 그래프의 시작점이 진입점이라는 것은 이제 알 것이다. 의존성 그래프의 시작점으로부터 Webpack은 번들해야 할 모든 의존성을 파악한다.

- **Output**: Output tells webpack where to put the bundles that it had made and what will be its format.

  > **결과물**: 결과물은 Webpack에게 Webpack이 만든 번들을 어디에 넣고, 또 어떤 포맷을 가져야하는지 알려준다.

- **Loaders**: Loaders convert different types of files like images and CSS into a module before adding them to the dependency graph.

  > **로더**: 로더는 이미지와 CSS와 같은 타입의 파일을 의존성 그래프에 추가하기 전 모듈로 변환한다. (역주: 의존성 그래프는 모듈 객체의 그래프이다.)

- **Plugins**: Plugins provide functionality. It can provide much functionality like printing something on running the webpack, minifying, optimization of bundles, etc.

  > **플러그인**: 플러그인은 추가적인 기능을 제공한다. Webpack을 실행하는 도중 무언가를 출력하거나, 번들을 경량화하고 최적화하는 등의 기능을 제공한다.

One can have a question. What’s the difference between loaders and plugins? Loaders work at the individual file level during or before the bundle is generated. Plugins work at the bundle or chunk level and usually work at the end of the bundle generation process.

Let’s look at the configuration file of the webpack to understand the above concepts.

> 한 가지 의문을 가질 수 있다. 로더와 플러그인의 차이는 무엇인가? 로더는 번들이 생성되기 전이나 생성되는 동안 개별 파일에 동작할 수 있다. 플러그인은 번들이나 청크 수준에서 동작하고 대개 번들 생성 과정의 후반부에서 동작한다.
>
> 이 개념을 이해하기 위해 Webpack의 설정 파일을 살펴보자.

```js
// webpack config 
const path = require("path");
const ExamplePlugin = require("./ExamplePlugin.js");
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.join(__dirname, "build"),
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: "babel-loader"
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
        ]
      },
    ]
  },
  plugins: [
    new ExamplePlugin(),
  ]
}

// Example of a loader: DoTranspile.js
// You will see a majority of loaders being transpilers.

const doTranspile = require('do-transpile');
module.exports = function(devlopmentSourceCode) {
    const browserCode = doTranspile(devlopmentSourceCode);
    return browserCode;
}

// Example of a plugin.
// ExamplePlugin.js

class ExamplePlugin {
  apply(compiler) {
    compiler.plugin("afterCompile", (compiler, callback) => {
      console.log("Webpack is Running!!");
      callback();
    })
  }
}

module.exports = ExamplePlugin;
```

Webpack starts the compilation process from the file mentioned in the **entry**. Output bundles get built in the folder mentioned in the **path** value of the **output** config option with the name mentioned in the **fileName** value.

> Webpack은 `entry`에 지정된 파일부터 컴파일 과정을 시작한다. 결과물인 번들은 `output` 설정 옵션의 `path`에 지정된 폴더에 배치되고 `fileName` 값에 지정된 이름으로 저장된다.

**Loaders** are generally used during the **bundling process**, mainly to convert or **translate** the file into a browser-readable format. Here we have added a [babel-loader](https://github.com/babel/babel-loader), [styles-loader](https://www.npmjs.com/package/styles-loader), and [css-loader](https://www.npmjs.com/package/css-loader). The babel-loader helps in converting modern javascript files into the browser readable javascript modules. The styles loader helps in converting CSS frameworks like sass and less to normal CSS modules. The css-loader helps in converting modern CSS features into browser-readable CSS stylings. There are many such loaders available in package-manager. You can also write your own loader like *DoTranspile.js* in the example.

> **로더**는 보통 번들링 과정 동안 사용되는데, 주로 파일을 브라우저가 읽을 수 있는 형태로 **변환**하는데 사용된다. 이 설정 파일에서 추가한 로더로는 [babel-loader](https://github.com/babel/babel-loader), [styles-loader](https://www.npmjs.com/package/styles-loader), 그리고 [css-loader](https://www.npmjs.com/package/css-loader)가 있다. babel-loader는 모던 자바스크립트 파일을 브라우저가 읽을 수 있는 자바스크립트 모듈로 바꾸는데 도움을 준다. style-loader는 sass와 less와 같은 CSS 프레임워크를 일반적인 CSS 모듈로 바꾸는데 도움을 준다. css-loader는 모던 CSS 기능을 브라우저가 읽을 수 있는 CSS 스타일링으로 바꾸는데 도움을 준다. 패키지 매니저에 다양한 로더들이 많다. 위 예시의 `DoTranspile.js`와 같은 자신만의 로더를 작성할 수도 있다.

**Plugins** are used to **tweak the bundle** at pre or post-processing stages, like for logging, modification, etc. There are many such plugins available in package-manager as well like [SourceMapDevToolPlugin](https://webpack.js.org/plugins/source-map-dev-tool-plugin) and [IgnorePlugin](https://webpack.js.org/plugins/ignore-plugin). Check [this](https://webpack.js.org/plugins/) to explore more. Webpack gives [different event hooks](https://webpack.js.org/api/compiler-hooks/) to listen to. You can add your own plugins to act upon bundles like *ExamplePlugin.js*.

> **플러그인**은 전처리나 후처리 단계에서 번들을 조정(tweak)하는데 사용할 수 있다. 예를 들어 로깅, 수정이 있다. 패키지 매니저에는 [SourceMapDevToolPlugin](https://webpack.js.org/plugins/source-map-dev-tool-plugin)과 [IgnorePlugin](https://webpack.js.org/plugins/ignore-plugin) 같은 다양한 플러그인이 많다. [여기](https://webpack.js.org/plugins/)를 눌러 탐색해보시라. Webpack은 [다양한 이벤트를 수신할 수 있는 이벤트 훅](https://webpack.js.org/api/compiler-hooks/)을 제공한다. 위 예시의 `*ExamplePlugin.js`와 같은 번들에 동작할 자신만의 플러그인을 추가할 수도 있다.

## What’s next for bundlers?

One major complaint with webpack is its **performance time is relatively high** which can be frustrating for the development journey as well as **critically slow** sometimes when deploying your app to the production environment.

> Webpack에 대한 주요 불만 한 가지는 **비교적 긴 수행 시간** 때문에 개발 과정에 좌절할 뿐만 아니라 애플리케이션을 프로덕션 환경에서 배포할 때 치명적으로 느릴 수 있다는 것이다.

Webpack is doing so many things in **single-threaded javascript**: compilation, resolving, creating module objects, parsing, and templating. These all are **language-agnostic tasks**. If we use another **lower-level multi-threaded language** to do all these tasks, building time can be improved exponentially. Few modern javascript bundlers are written with such languages as [Turbopack](https://turbo.build/pack)(written in Rust) and [ESBuild](https://esbuild.github.io/)(written in GO) which claim to be faster than webpack.

> Turbopack is an incremental bundler optimized for JavaScript and TypeScript, written in Rust by the creators of Webpack and [Next.js](https://nextjs.org/) at [Vercel](https://vercel.com/).
>
> On large applications, Turbopack updates 700x faster than Webpack.

> Webpack은 **싱글 스레드 자바스크립트**에서 많은 것을 할 수 있다: 컴파일, 해석, 모듈 객체 만들기, 파싱하기, 템플릿으로 작성하기. 이것들은 모두 **언어에 구애받지 않는 작업들**이다. 또다른 **저수준 멀티 스레드 언어**를 사용하기로 했다면, 빌드 시간은 지수적으로 개선될 수 있을 것이다. 몇몇 모던 자바스크립트 번들러들은 그러한 언어들로 쓰여졌는데, Webpack보다 빠르다고 주장하는 Rust로 쓰여진 [Turbopack](https://turbo.build/pack)과 Go로 쓰여진 [ESBuild](https://esbuild.github.io/)가 그것이다.
>
> > Turbopack은 JavaScript와 TypeScript에 최적화된 증분 번들러로, Webpack과 [Vercel](https://vercel.com/)의 [Next.js](https://nextjs.org/)의 제작자가 Rust로 작성했다.
> >
> > 규모가 큰 애플리케이션에서, Turbopack은 Webpack보다 700배 빠르게 업데이트한다.

Webpack is a **battle-tested javascript bundler** but the performance time is still an issue that can be solved by using lower-level multi-threaded language. Let’s see where Webpack is heading in the near future!

> Webpack은 치열한 테스트를 거친 자바스크립트 번들러이지만 수행 시간이 여전히 문제이고 이것은 저수준 멀티 스레드 언어를 사용하여 해결될 수 있다. 가까운 미래에 Webpack은 어디로 향하는지 지켜보자!