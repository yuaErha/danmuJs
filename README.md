弹幕插件
#### 油猴插件: 爬取主流视频网站弹幕显示到第三方播放器

支持弹幕列表
| 名称 | 是否支持 |
| ----- |:----:|
|腾讯视频|√|
|爱奇艺|√|
|优酷视频|√|
|芒果TV|√|

### 效果图
![demo_1](./images/demo.png)

本项目是使用js爬取主流视频网站弹幕并解析数据, 然后将弹幕插入到第三方(~~盗版~~)网址的播放器中, 从而实现白嫖~~~
 
使用
========

弹幕解析
--------
腾讯解析

```JavaScript
async tencentParse(url, offset = 0) {
  const { vid } = await xhr({ method: 'GET', url }).then(resp => JSON.parse(resp.responseText.match(/var VIDEO_INFO = (\{.*?\})/m)[1]))
  const targetId = await xhr({
    method: 'GET',
    url: 'http://bullet.video.qq.com/fcgi-bin/target/regist?otype=json&vid=' + vid
  }).then(resp => resp.responseText.match(/targetid=(\d+)/m)[1])
  // const timestamp = Math.floor(timepoint / 30)
  // const offset = 30 * timestamp + 15
  const danmuUrl = 'http://mfm.video.qq.com/danmu?timestamp=' + offset + '&target_id=' + targetId
  return xhr({ method: 'GET', url: danmuUrl })
    .then(resp => {
      let comments = JSON.parse(resp.responseText)['comments']
      return comments.map(
        item =>
          new Object({
            content: item['content'],
            name: item['opername'],
            time: item['timepoint']
          })
      )
    })
    .catch(err => [])
}
```
爱奇艺解析
------------

```JavaScript
async iqiyiParse(url, offset = 0) {
const { tvId } = await xhr({ method: 'GET', url }).then(resp => JSON.parse(resp.responseText.match(/:page-info='(.*?)'/m)[1]))
// const offset = Math.ceil(timepoint / 60 / 5)
const danmuUrl = `https://cmts.iqiyi.com/bullet/${String(tvId).substr(String(tvId).length - 4, 2)}/${String(tvId).substr(String(tvId).length - 2, 2)}/${tvId}_300_${offset}.z`
return xhr({
  method: 'GET',
  url: danmuUrl,
  responseType: 'arraybuffer'
})
  .then(resp => {
    const responseArray = new Uint8Array(resp.response)
    const responseXMl = new TextDecoder().decode(pako.ungzip(responseArray)).replace(/&#\d{2};/g, '')
    const parser = new DOMParser()
    const xmlDoc = parser.parseFromString(responseXMl, 'text/xml')
    return Array.from(xmlDoc.querySelectorAll('bulletInfo')).map(node => {
      const item = xml2json(node)
      return new Object({
        content: item['content'],
        name: item['userInfo']['name'] || '',
        time: item['showTime']
      })
    })
  })
  .catch(err => [])
}
```
优酷解析
------------

```JavaScript
async youkuParse(url, offset = 0) {
  const { videoId, seconds } = await xhr({ method: 'GET', url }).then(resp => {
    let data = resp.responseText.match(/\}\s*window\.PageConfig\s*=\s*([\s\S]*?);\s*var/m)[1]
    eval(`data = ${data}`)
    return data
  })
  const danmuUrl = `https://service.danmu.youku.com/list?mat=${offset}&ct=1001&iid=${videoId}`
  console.log(danmuUrl)
  await xhr({ method: 'GET', url: danmuUrl }).then(resp => {
      console.log(resp.responseText)
  })
  console.log(videoId)
}
```
芒果解析
------------

```JavaScript
async mgtvParse(url, offset = 0) {
  const splits = url.split('/')
  const cid = splits[4]
  const vid = splits[5].split('.')[0]
  // const offset = Math.floor(timepoint / 60)
  const danmuUrl = `https://galaxy.bz.mgtv.com/rdbarrage?version=3.0.0&vid=${vid}&cid=${cid}&time=${60 * 1000 * offset}`
  return xhr({ method: 'GET', url: danmuUrl })
    .then(resp => {
      const comments = JSON.parse(resp.responseText)['data']['items']
      return comments.map(
        item =>
          new Object({
            content: item['content'],
            name: '',
            time: item['time']
          })
      )
    })
    .catch(err => [])
}
```
Video 监听
------------

```JavaScript
// 监听Document以及frame下的 video元素
function ready(selector, fn) {
  const docRoot = window.document.documentElement
  if (!docRoot) return false
  const listenNodeList = []
  // 获取MutationObserver，兼容低版本的浏览器
  const MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver
  // 创建Observer
  const Observer = new MutationObserver(mutations => {
    docRoot.querySelectorAll(selector).forEach(item => {
      if (listenNodeList.includes(item)) {
      } else {
        listenNodeList.push(item)
        fn(item)
        Observer.disconnect()
      }
    })
  })
  // 获取dom元素
  Observer.observe(docRoot, {
    childList: true,
    subtree: true
  })
}
```

版本更新
========

1.0
---
实现弹幕爬取播放功能
 
