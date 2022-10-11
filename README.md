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

