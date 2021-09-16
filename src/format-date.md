```javascript
export function formatDate (timespan, fmt = 'yyyy-MM-dd HH:mm') {
  if (!Number.parseInt(timespan)) {
    return timespan
  }
  const date = new Date(timespan)
  if (/(y+)/.test(fmt)) {
    fmt = fmt.replace(RegExp.$1, (date.getFullYear() + '').substr(4 - RegExp.$1.length))
  }
  let o = {
    'M+': date.getMonth() + 1,
    'd+': date.getDate(),
    'H+': date.getHours(),
    'm+': date.getMinutes(),
    's+': date.getSeconds(),
    'f+': date.getMilliseconds(),
    'w+': zhDay(date.getDay())
  }
  for (let k in o) {
    if (new RegExp(`(${k})`).test(fmt)) {
      let str = o[k] + ''
      fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? str : pad(str))
    }
  }
  return fmt
}

export function pad (n, z = 2) {
  return ('00' + n).slice(-z)
}

export function zhDay (w) {
  const arr = ['日', '一', '二', '三', '四', '五', '六']
  return arr[w]
}
```

``` javascript
const now = Date.now()
formateDate(now); // 2021-09-16 10:00

formateDate(now, 'yyyy/MM/dd HH:mm:ss 周w') // 2021/09/16 10:00:00 周四
```
