---
uuid: b0657f40-b8f9-11e9-a5ae-7f2af82a4fbe
author: xiongzhaoyang
email: xiongzhaoyang315@gmail.com
github: https://github.com/xzy0315
avatar: https://avatars0.githubusercontent.com/u/14714560?s=460&v=4
title: 分分钟搞定node爬虫
date: 2019-08-07 17:56:54
tags: Node 爬虫
---

> 教大家如何用 node 写一个简单的爬虫程序

### 爬虫是什么

网络爬虫，是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本。

顾名思义，爬虫就是从互联网上爬取我们想要的数据，也可以叫它为数据挖掘。

在大数据时代，数据就是第一生产力。例如我们想买到低价的机票，可能需要时不时的去各个网站上去看实时的价格。但如果我们掌握了爬虫，我们可以编写脚本定时的去爬取各个网站上实时的机票价格，然后设置一个阈值，当到达这个价格的时候就给自己发通知，这样就可以抢到我们心仪价格的机票了~

<br />

### 准备工作

用 node 做爬虫，我们需要一些工具（npm包）来帮助我们更好的去爬取数据。

* [request](https://www.npmjs.com/package/request) 常用的网络请求库
* [iconv-lite](https://www.npmjs.com/package/iconv-lite) 用于将 GBK 编码转成 utf8
* [cheerio](https://www.npmjs.com/package/cheerio) 用于解析HTML编码的字符串，并可通过类jquery的方式来获取

<br />

### 网站常见的数据渲染方式

目前网站中的数据基本是通过两种方式展示的。一种是服务端渲染，一种是前端渲染。所以接下来的实战会包含这两种渲染方式的爬取。

* 服务端渲染，数据和页面的HTML在一起的。我们直接通过请求页面的URL，就可以拿到 HTML 编码的字符串，然后再去解析就可以拿到我们想要的数据。
* 前端渲染，一般通过接口来获取数据并渲染，这种情况我们只要找出获取数据的接口，就可以通过直接调用接口来获取到我们想要的数据。

<br />

### 爬虫实战

#### 爬取百度风云榜的电影热搜榜（服务端渲染）

> [百度风云榜地址](http://top.baidu.com/category?c=1)

```javascript
import request from 'request-promise'
import cheerio from 'cheerio'
import iconv from 'iconv-lite'

async function task () {
  // 风云榜 - 娱乐
  const pageUrl = 'http://top.baidu.com/category?c=1'
  let html = await request({
    url: pageUrl,
    encoding: null,
  })
  // 不设置 encoding: null 的话，其内部会自动帮我们将 buffer 转成 string
  // html = html.toString('utf8')
  html = iconv.decode(html, 'gbk')
  // 解析 HTML 编码的数据
  const $ = cheerio.load(html)

  $('.box-cont').each((_, item) => {
    const boxCont = $(item)
    const title = boxCont.find('.hd .title a').text()
    console.log(title)

    Array.from(boxCont.find('.list-title')).forEach((item, idx) => {
      console.log(idx + 1, $(item).attr('title'))
    })
    console.log('\n')
  })
}
```
<br />

#### 爬取京东商品评论接口并分析热销的型号（前端渲染）
> [iphonexs max 评论区地址](https://item.jd.com/100000287113.html#comment)

```javascript
import request from 'request-promise'
import iconv from 'iconv-lite'

// 颜色
const colorMap: any = {}
// 内存
const memoryMap: any = {}
// 版本
const editionMap: any = {}
let count = 1

async function task () {
  // 爬取 10 页评论
  for (let i = 1; i <= 10; i += 1) {
    await analyseComment(i)
  }

  console.log(colorMap)
  console.log(memoryMap)
  console.log(editionMap)
}

/**
 * 分析评论
 * @param {number} [page=1] 分析的页码
 */
async function analyseComment (page = 1) {
  // jd 评论接口
  const pageUrl = 'https://sclub.jd.com/comment/productPageComments.action'
  let data = await request({
    url: pageUrl,
    headers: {
      // 模拟浏览器
      'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36',
      // 来源字段必须存在，否则返回数据为空。
      referer: 'https://item.jd.com/100000287113.html',
    },
    qs: {
      productId: 100000287113,
      score: 0,
      sortType: 5,
      page,
      pageSize: 10,
      isShadowSku: 0,
      rid: 0,
      fold: 1,
    },
    encoding: null,
  })
  // 编码转换
  data = iconv.decode(data, 'gbk')
  // 转换成 JSON
  data = JSON.parse(data)

  const { comments } = data

  comments.forEach((comment: any) => {
    // 分析颜色，内存和型号(版本)
    const color = comment.productColor
    const memory = comment.productSales[0].saleValue
    const edition = comment.productSize

    colorMap[color] = colorMap[color] ? colorMap[color] + 1 : 1
    memoryMap[memory] = memoryMap[memory] ? memoryMap[memory] + 1 : 1
    editionMap[edition] = editionMap[edition] ? editionMap[edition] + 1 : 1

    console.log(`--第${count}条评论分析成功--`)
    count += 1
  })

  console.log(`--第${page}页评论分析成功--`)

  // 避免请求频繁被封 IP >_<
  await new Promise(r => setTimeout(r, 2000))
}

task()
```