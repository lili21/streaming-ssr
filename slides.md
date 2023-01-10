---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: prism
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
---

# Streaming SSR

The Lost Art Of Progressive HTML Rendering

---

# 问题I - 静态资源加载被推迟

<img v-click src="/media/csr-ssr.png" />

---

# 问题II - HTML请求耗时由耗时最长的接口决定

<div grid="~ cols-2 gap-4">

<img src="/media/camp.png" />

<v-click>

```js
// loader for $group_id
export const loader = async ({ params }) => {
  const result = await getGroupInfo(params.group_id);
  return json(result)
}

// loader for $group_id/posts
export const loader = async ({ params }) => {
  const result = await getGroupPosts(params.group_id);
  return json(result)
}
```

</v-click>

</div>

<style>
  img {
    width: 200px;
    height: auto;
  }
</style>

---

# SSR问题III - 服务可用性随着接口依赖增多而降低

```jsx
export const loader = async () => {
  const user = await getUserInfo()
  const detail = await getDetail()

  return json({
    user,
    detail
  })
}
```

---

# What is Streaming SSR?

<video controls src="/media/ssr-streaming.mp4" />

<style>
video {
    height: 400px;
    width: auto;
  }
</style>

---

# How does it work?

<div grid="~ cols-2 gap-4">

```js {all|7-11}
const http = require("http");

const server = http.createServer(async (req, res) => {
  res.writeHead(200, {
    "Content-Type": "text/html",
  });
  res.write("<h1>Hello, World</h1>");
  await sleep(1000);
  res.write("<h2>Streaming SSR</h2>");
  await sleep(1000);
  res.write("<h3>Progressive HTML Rendering</h3>");
  res.end();
});

server.listen(3000)
```

<video controls v-click src="/media/streaming-demo.mov" />

</div>

<style>
  video {
    width: 300px;
    height: auto;
  }
</style>

---

# 静态资源加载时机被推迟

```js {all|5-10}
const server = http.createServer(async (req, res) => {
  res.writeHead(200, {
    "Content-Type": "text/html",
  });
  res.write(`
    <head>
      <link rel="stylesheet" href="style.css" />
      <script src="index.js"></script>
    </head>
  `)
  const data = await dataFetching();
  const html = template(data);
  res.write(html)
  res.end();
});
```

---

# Background

<v-clicks>

- [The Lost Art Of Progressive HTML Rendering. Jeff Atwood, 2005](https://blog.codinghorror.com/the-lost-art-of-progressive-html-rendering/)
- [BigPipe: Pipelining web pages for high performance. Facebook, 2009](https://www.facebook.com/notes/10158791368532200/)
- [新浪微博的BigPipe后端实现技术分享. 微博, 2011](https://www.slideshare.net/slawdan/bigpipe1126adev)
- [MarkoJS. eBay, 2013](https://markojs.com/)
- [React18 - Suspense SSR Architecture. 2022](https://github.com/reactwg/react-18/discussions/37)

</v-clicks>

---

# React开启Streaming

<div grid="~ cols-2 gap-4">

```jsx
import { renderToString } from 'react-dom/server';

const html = renderToString(<App />);
```

```jsx
import { renderToPipeableStream } from 'react-dom/server';

const { pipe } = renderToPipeableStream(<App />, {
  onShellReady() {
    response.statusCode = 200;
    response.setHeader('content-type', 'text/html');
    pipe(response);
  },
  onShellError(error) {
    response.statusCode = 500;
    response.setHeader('content-type', 'text/html');
    response.send('<h1>Something went wrong</h1>'); 
  },
  onAllReady() {},
  onError() {
    console.error(error);
  }
});
```

</div>

---
layout: cover
---

# The Defer API

---

<div grid="~ cols-2 gap-4">

<img src="/media/camp.png" />

```js
// loader for $group_id
export const loader = async ({ params }) => {
  const result = await getGroupInfo(params.group_id);
  return json({
    groupInfo: result
  })
}

// loader for $group_id/posts
export const loader = async ({ params }) => {
  const result = await getGroupPosts(params.group_id);
  return json({
    posts: result
  })
}
```

</div>


<style>
  img {
    width: 250px;
    height: auto;
  }
  </style>

---

<div grid="~ cols-2 gap-4">

<div>

```js {11-14}
// loader for $group_id
export const loader = async ({ params }) => {
  const result = await getGroupInfo(params.group_id);
  return json({
    groupInfo: result
  })
}

// loader for $group_id/posts
export const loader = async ({ params }) => {
  const result = await getGroupPosts(params.group_id);
  return json({
    posts: result
  })
}
```
</div>

<div>

```js {11-14}
// loader for $group_id
export const loader = async ({ params }) => {
  const result = await getGroupInfo(params.group_id);
  return json({
    groupInfo: result
  })
}

// loader for $group_id/posts
export const loader = async ({ params }) => {
  const result = getGroupPosts(params.group_id);
  return defer({
    posts: result
  })
}
```

</div>

</div>

---

```jsx
import { Await, useLoaderData } from '@remix-run/react';

export default function Posts() {
  const { posts } = useLoaderData();

  return (
    <Suspense fallback={<div>loading...</div>}>
      <Await resolve={posts} errorElement={<div>posts获取失败</div>}>
        {
          (data) => {/* 接口返回的数据在这里使用 */}
        }
      </Await>
    </Suspense>
  )
}
```

---

```jsx
import { Await, useLoaderData, useAsyncValue } from '@remix-run/react';

export default function Posts() {
  const { posts } = useLoaderData();

  return (
    <Suspense fallback={<div>loading...</div>}>
      <Await resolve={posts} errorElement={<div>posts获取失败</div>}>
        <PostList />
      </Await>
    </Suspense>
  )
}

function PostList() {
  const data = useAsyncValue();
  return <div>post list {data}</div>
}
```

---

```jsx {all|2-5|9|11-18}
export const loader = async () => {
  const result = fetchData();
  return defer({
    data: result
  })
}

export default function Page() {
  const { data } = useLoaderData();
  return (
    <Suspense fallback={<Loading />}>
      <Await resolve={data} errorElement={<ErrorElement />}>
        {
          (_data) => <div>{_data}</div>
        }
      </Await>
    </Suspense>
  )
}
```

---

```jsx
export const loader = async () => {
  const result = fetchData();
  const result2 = await fetchData2();
  return defer({
    data: result,
    data2: result2
  })
}

export default function Page() {
  const { data, data2 } = useLoaderData();

  return (
    <div>
      <div>{data2}</div>
      <Suspense fallback={<Loading />}>
        <Await resolve={data} errorElement={<ErrorElement />}>
          {
            (_data) => <div>{_data}</div>
          }
        </Await>
      </Suspense>
    </div>
  )
}
```

---
layout: cover
---

# 实战

---

# 周战报

<div grid="~ cols-3 gap-4">

<img src="/media/report-1.png" />
<img src="/media/report-2.png" />
<img src="/media/report-3.png" />

</div>

<style>
  img {
    width: 220px;
    height: 100%;
  }
  </style>

---

```js
export const loader = async () => {
  const [baseData, keywords, data, hero, hallOfFrame, finish] =
    await Promise.all([
      getBattleUser(), // 用户信息
      getReport(ReportType.Keywords), // 总结模块，第一屏
      getReport(ReportType.Data), // 段位数据，第二屏
      getReport(ReportType.Hero), // 常用英雄，第三屏
      getReport(ReportType.HallOfFrame), // 名人堂，第四屏
      getReport(ReportType.Finish), // 尾页，第五屏
    ]);
  return json({
    baseData,
    keywords,
    data,
    hero,
    hallOfFrame,
    finish
  })
}
```

---

<img src="/media/weekly-report.png" />

<style>
  img {
    height: 100%;
  }
</style>

---
layout: two-cols
---

<img src="/media/durate.png" />

::right::

<v-click>

<img class="argos" src="/media/argos.png" />

</v-click>


<style>
  img {
    width: 300px;    
  }

  .argos {
    width: 500px;
  }
</style>

---

```js
export const loader = async () => {
  const [baseData, keywords, data, hero, hallOfFrame, finish] =
    await Promise.all([
      getBattleUser(), // 用户信息
      getReport(ReportType.Keywords), // 总结模块，第一屏
      getReport(ReportType.Data), // 段位数据，第二屏
      getReport(ReportType.Hero), // 常用英雄，第三屏
      getReport(ReportType.HallOfFrame), // 名人堂，第四屏
      getReport(ReportType.Finish), // 尾页，第五屏
    ]);
  return json({
    baseData,
    keywords,
    data,
    hero,
    hallOfFrame,
    finish
  })
}
```

---

```js {all|8|2-6}
export const loader = async () => {
  const baseData = getBattleUser() // 用户信息
  const data = getReport(ReportType.Data) // 段位数据，第二屏
  const hero = getReport(ReportType.Hero) // 常用英雄，第三屏
  const hallOfFrame = getReport(ReportType.HallOfFrame) // 名人堂，第四屏
  const finish = getReport(ReportType.Finish) // 尾页，第五屏

  const keywords = await getReport(ReportType.Keywords), // 总结模块，第一屏

  return defer({
    baseData,
    keywords,
    data,
    hero,
    hallOfFrame,
    finish
  })
}
```

---

# 问题

<div grid="~ cols-2 gap-4">

```js
export const loader = async() => {
  // 段位数据，第二屏
  const data = getReport(ReportType.Data)
  // 常用英雄，第三屏
  const hero = getReport(ReportType.Hero)

  return defer({
    data,
    hero
  })
}
```

<div>

<Cls />

</div>

</div>

---

# 解决方案

```jsx
export default function Page() {
  const { data, hero } = useLoaderData();

  return (
    <SuspenseList>
      <Suspense>
        <Await resolve={data}>
           <Data />
        </Await/>
      </Suspense>
       <Suspense>
        <Await resolve={hero}>
           <Hero />
        </Await/>
      </Suspense>
    </SuspenseList>
  )
}

```

---

# 解决方案

```js
export const loader = async () => {
  const otherData = Promise.allSettle([
    getBattleUser(),
    getReport(ReportType.Data), // 段位数据，第二屏
    getReport(ReportType.Hero), // 常用英雄，第三屏
    getReport(ReportType.HallOfFrame), // 名人堂，第四屏
    getReport(ReportType.Finish) // 尾页，第五屏
  ])

  const keywords = await getReport(ReportType.Keywords), // 总结模块，第一屏

  return defer({
    keywords,
    otherData
  })
}
```

---
layout: cover
---

# 效果

---

# 报警数量减少90%

<img src="/media/alarm.png" />

---

# TTFB减少19%

<img src="/media/ttfb.png" />

<style>
  img {
    width: 80%;
  }
  </style>

---

# LCP减少12%

<img src="/media/lcp.png" />

<style>
  img {
    width: 80%;
  }
</style>


---

# 能不能全部用defer呢

```js
export const loader = async () => {
  const otherData = Promise.allSettle([
    getBattleUser(),
    getReport(ReportType.Data), // 段位数据，第二屏
    getReport(ReportType.Hero), // 常用英雄，第三屏
    getReport(ReportType.HallOfFrame), // 名人堂，第四屏
    getReport(ReportType.Finish) // 尾页，第五屏
  ])

  const keywords = await getReport(ReportType.Keywords), // 总结模块，第一屏

  return defer({
    keywords,
    otherData
  })
}
```

---
layout: center
---

<img src="/media/faas.png" />

---

<div grid="~ cols-2 gap-4">

<img src="/media/camp.png" />

```js
// loader for $group_id
export const loader = async ({ params }) => {
  const result = await getGroupInfo(params.group_id);
  return json({
    groupInfo: result
  })
}

// loader for $group_id/posts
export const loader = async ({ params }) => {
  const result = getGroupPosts(params.group_id);
  return defer({
    posts: result
  })
}
```

</div>

<style>
  img {
    width: 250px;
    height: auto;
  }
  </style>
