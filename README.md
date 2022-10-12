This is a starter template for [Learn Next.js](https://nextjs.org/learn).

# 새로 배운것

> 출처: https://nextjs.org/docs

*****

# Link Component
- enables client-side navigation
- client-side navigation
    - 페이지 전환이 자바스크립트를 통해 이루어지는데, 브라우저를 통해 이루어지는 것보다 빠르다.
    - 브라우저를 통한 전환은 전통적 `<a>` 태그
- 심지어, `Link`를 쓰면 Next.js가 자동적으로 링크되는 페이지의 코드를 prefetch -> 백그라운드에서 이미 도착할 페이지가 로딩될것! 훨씬 빠름.

### 정리
You create routes as files under pages and use the built-in Link component. No routing libraries are required.

# Script Component

```
<Script
    src= "https://connect.facebook.net/en_US/sdk.js"
    strategy="lazyOnload"
    onLoad={() =>
        console.log(`script loaded correctly, window.FB has been populated`)
    }
/>
```

- strategy: 서드파티 스크립트가 로드되어야 할 시점
    - lazyOnload: load this script lazily during browser idle time
- onLoad: used to run any JS code immediately after the script has finished loading. 

# CSS Modules
- `파일명.module.css` 형식이어야 사용 가능
- 자동으로 유니크한 클래스명을 생성하므로 CSS 모듈을 사용한다면 클래스 이름 충돌 걱정할 필요 없음
- Next.js의 code splitting -> CSS 모듈에 잘 적용된다
    - 각 페이지마다 최소 양의 CSS가 로드되게 한다.
    - 결고적으로, 더 작은 번들 사이즈
- CSS 모듈은 빌드 시점에 자바스크립트 번들에서 추출되고, Next.js에 의해 `.css` 파일로

# Global Styles
- CSS 모듈은 component-level style에 유용
- 모든 페이지에 적용되길 원하는 CSS -> global CSS

## App Component
- `App` is the top-level component which will be common across all the different pages

## Adding Global CSS
- global CSS 파일을 추가하는 법: `pages/_app.js`에서 파일을 임포트한다.
- global CSS는 `pages/_app.js` 밖에서는 임포트 될 수 없다.
    - global CSS가 페이지의 모든 엘리먼트에 영향을 주기때문
    - e.g., 홈페이지에서 `/posts/first-post` 페이지로 이동할 때, 홈페이지의 글로벌 스타일이 `/posts/first-post`에도 영향을 미칠 것이다.

*****

# Pre-rendering and Data Fetching
## Pre-rendering
- Next.js는 기본으로 모든 페이지를 `pre-render`
    - = client-side에서 할 필요 없이, Next.js는 각 페이지에 대해 미리 HTML을 만들어 둔다.
    - better performance, better SEO
    - 생성된 HTML은 그 페이지에 필요한 최소한의 자바스크림트로 이루어짐
    - 브라우저가 페이지를 로드하면, JS코드가 실행되고, 그제서야 페이지가 완전히 인터랙티브해진다! (이 과정을 `hydration` 이라고 부름)

## 2 Forms of Pre-rendering
1. Static Generation
    - 빌드 시점에 HTML을 pre-render, 각 요청 시 이 HTML을 재사용
2. Server-side rendering
    - 각 요청마다 HTML을 생성

> In development mode (when you run npm run dev or yarn dev), pages are pre-rendered on every request. This also applies to Static Generation to make it easier to develop. When going to production, Static Generation will happen once, at build time, and not on every request.

## 언제 Static Generation을 쓰고 언제 Server-side Rendering을 쓰는가
- Static Generation
    - 가능하다면 사용하는 것을 권장
    - CDN으로 HTML 전달 -> 서버가 각 요청마다 페이지를 렌더하는 것보다 훨씬 빠름
    - 예시
        - Marketing pages
        - Blog posts
        - E-commerce product listings
        - Help and documentation
    - 고려할 점
        - 유저의 요청에 앞서서 이 페이지를 미리 렌더할 수 있나? -> Yes -> Static generation
    - 적절하지 않은 경우
        - 유저의 요청보다 먼저 페이지를 렌더할 수 없는 경우
        - 페이지가 자주 업데이트되는 데이터를 가지거나, 각 요청마다 페이지 내용이 변화할 떄 -> `Server Side Rendering`

## Static Generation with Data
![img](https://nextjs.org/static/images/learn/data-fetching/static-generation-with-data.png)

- Static generation with Data using `getStaticProps`
    - Next.js에서 페이지 컴포넌트를 export하면, `getStaticProps`라는 `async` 펑션도 export 가능
        - `getStaticProps` runs at build time in production, and
        - **Inside the function, you can fetch external data and send it as props to the page**

    - `getStaticProps`: Next.js야 이 페이지 data dependency 있으니까 너 이 페이지 빌드 타임에 pre-render할 때, data 먼저 해결해라
    - (development mode에서는 `getStaticProps`가 매 요청마다 실행된다.)

    ```
    export default function Home(props) { ... }

    export async function getStaticProps() {
    // Get external data from the file system, API, DB, etc.
    const data = ...

    // The value of the `props` key will be
    //  passed to the `Home` component
    return {
        props: ...
    }
    }
    ```

    ![img](https://nextjs.org/static/images/learn/data-fetching/index-page.png)    


## getStaticProps Details
### Fetch External API or Query Database
- fetched data from other sources (external API endpoint)
```
export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from an external API endpoint
  const res = await fetch('..');
  return res.json();
}
```

- query DB directly
```
import someDatabaseSDK from 'someDatabaseSDK'

const databaseClient = someDatabaseSDK.createClient(...)

export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from a database
  return databaseClient.query('SELECT posts...')
}
```

- 이것들이 가능한 이유 -> `getStaticProps`는 only **runs on the server-side**
    - never run on the client-side
    - won't even be included in the JS bundle for the browser
    - You can write code such as direct DB queries without them being sent to browsers

### Development Vs. Production
- Development
    - `npm run dev` or `yarn dev`
    - `getStaticProps` runs on every request
- Production
    - `getStaticProps` runs at build time
    - However, this behavior can be enhanced using the `fallback` key returned by `getStaticPaths`
- 빌드타임에 실행되게 되어있으므로, 당신은 **query Parameter**나 **HTTP headers**같은 request time에만 사용 가능한 데이터를 쓰지 못할 것이다.

### Only Allowed in a Page
- `getStaticProps`는 오직 page로부터 export 될 수 있다. 페이지가 아닌 파일에서는 export 불가능.
- 이런 제한이 있는 이유는 React는 페이지가 렌더되기 전에 모든 필요 데이터를 갖고 있어야 하기 때문이다.

### What If I Need to Fetch Data at Request Time?
- Static Generation은 build time에 딱 한 번 일어나기 때문에 자주 바뀌어야 하는 데이터나 유저의 요청에 반응해야 하는 것에는 적절하지 않다.
- 데이터가 바뀌는 이런 상황에서는 **Server-side Rendering**을 사용할 수 있다. 
- 그니까 `getStaticProps`는 client side라는 것

## Fetching Data at Request Time
> 빌드 타임이 아닌, 리퀘스트 타임에 데이터를 가져와야 한다면 Server-Side Rendering!

### Using `getServerSideProps`
```
export async function getServerSideProps(context) {
  return {
    props: {
      // props for your component
    },
  };
}
```
- `getServerSideProps`가 리퀘스트 타임에 호출되므로 parameter인 `context`는 request specific parameters를 포함한다.
- 반드시 데이터가 request time에 fetch되어야 하는 페이지를 pre-render 할 때만 `getServerSideProps`를 사용해라
    - Time to first byte (TTFB)가 `getStaticProps`를 사용할 때보다 느릴 것
    - 서버가 매 요청마다 결과를 계산해야 하고, 그 결과는 extra configuration 없이는 CDN으로 캐싱되지 않을 것이기 때문이다.

## Client-side Rendering
- If you **do not** need to pre-render the data Client-side rendering 전략 취할 수도 있다.
    - Statically generate (pre-render) parts of the page that do not require external data
    - When the page loads, fetch external data from the client using JS and populate the remaning parts.

- Private, user-specifig pages에 적합 (SEO 필요 없을 때)

![img](https://nextjs.org/static/images/learn/data-fetching/client-side-rendering.png)

### SWR
- client side에서 data fetching할 때 쓰라고 Next.js가 만든 React hook
- cacing, revalidation, focus tracking, reteching on interval, ...
- 관심 있으면 찾아보기

*****

# Dynamic Routes
> 블로그 데이터마다 URL이 있어서, 포스트를 열람했으면 좋겠다!

## Page Path Depends on External Data
![img](https://nextjs.org/static/images/learn/dynamic-routes/page-path-external-data.png)
- Next.js allows you to statically generate pages with paths that depend on external data -> enables **dynamic URLs** in Next.js
- Graphic Summary
    ![img](https://nextjs.org/static/images/learn/dynamic-routes/how-to-dynamic-routes.png) 


## Implement getStaticPaths
- getAllPostIds function (lib/posts.js)
```
export function getAllPostIds() {
  const fileNames = fs.readdirSync(postsDirectory);

  // Returns an array that looks like this:
  // [
  //   {
  //     params: {
  //       id: 'ssg-ssr'
  //     }
  //   },
  //   {
  //     params: {
  //       id: 'pre-rendering'
  //     }
  //   }
  // ]
  return fileNames.map((fileName) => {
    return {
      params: {
        id: fileName.replace(/\.md$/, ''),
      },
    };
  });
}
```
    - 중요: 반환되는 리스트는 array of objects(not strings)
    - 각각의 object는 `params` key를 가져야 하고, `id` key를 가진 object를 value로 해야 한다.
    - 우리가 file name에서 `[id]`를 사용하고 있기 때문
    - 이렇게 하지 않으면 `getStaticPaths` 실패

- getStaticPaths (pages/posts/[id].js)
```
import { getAllPostIds } from '../../lib/posts';

export async function getStaticPaths() {
  const paths = getAllPostIds();
  return {
    paths,
    fallback: false,
  };
}
```

## Dynamic Routes Details
### Development vs. Production
- Development
    - `npm run dev` or `yarn dev`
    - `getStaticPaths` runs on every request
- Production
    - `getStaticPaths` runs at build time

### Fallback
```
export async function getStaticPaths(){
    const paths = getAllPostIds();
    return{
        paths,
        fallback: false,
    }
}
```
- `fallback`의 의미
    - `fallback: false`
        - `getStaticPaths`가 리턴하지 않았던 경로 -> **404page**
    - `fallback: true`
        - `getStaticPaths`의 behavior 변화
        - `getStaticPaths`로 리턴받은 paths들은 build time에 HTML로 렌더
        - build time에 생성되지 않은 paths -> 404 page 뜨지 않음.
        대신, Next.js가 그런 path에 처음 접근했을 때, 그 페이지의 "fallback" version을 띄움.
        - In the background, Next.js will statically generate the requested path. Subsequent requests to the same path will serve the generated page, just like other pages pre-rendered at build time.
    - `fallback: blocking`
        - New paths will be server-side rendered with `getStaticProps`, and cached for future requests so it only happends once per path

### Catch-all Routes
- Dynamic routes can be extended to catch all paths by adding `...`
    - `pages/posts/[...id].js` matches `/posts/a`, but also `/posts/a/b`, `/posts/a/b/c` and so on.

- If you do this, in getStaticPaths, you must return an array as the value of the id key like so:
```
return [
  {
    params: {
      // Statically Generates /posts/a/b/c 🐶
      id: ['a', 'b', 'c'],
    },
  },
  //...
];
```

- And `params.id` will be an array in getStaticProps:
```
export async function getStaticProps({ params }) {
  // params.id will be like ['a', 'b', 'c']
}
```


### Router
- Next.js router -> import 'useRouter' hook from 'next/router'

### 404 Pages
- To create a custom 404 page, create `pages/404.js`

*****

# API Routes

## Creatine API Routes
- API endpoint를 만들 수 있게 해줌
- `pages/api` 에 function을 만들면 됨
- can be deployed as Serverless Functions (also known as lambdas)

### localhost:3000/api/hello
- pages/api/hello.js 생성
```
export default function handler(req, res) {
  res.status(200).json({ text: 'Hello' });
}
```

## API Routes Details
### Do Not Fetch an API Route from `getStaticProps` or `getStaticPaths`
- 대신, `getStaticProps`, `getStaticPaths` 내부에 서버 사이드 코드를 직접 작성하라
- 아니면 helper function을 호출

- `getStaticProps`, `getStaticPaths`는 서버사이드에서만 실행되고 클라이언트 사이드에서는 실행 X
- 또, 이런 함수들은 브라우저를 위한 JS 번들에는 포함되지 않을 것 (DB 쿼리에 쓰기 좋음)

### Good Use Case: Handling From INput
- 폼 작성 뒤 `POST` -> DB에 저장하는 코드를 작성
- API route code는 클라이언트 번들에 포함이 되지 않으므로 안전하게 서버사이드 코드를 작성할 수 있다.

### Dynamic API Routes
- API routes들도 다이나믹할 수 있다.