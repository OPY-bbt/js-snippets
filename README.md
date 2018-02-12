# js-snippets

## stableSort
```js
// chrome sort function which built in use different algorithm 
// when the count greater than 10; In this status, the result isn't
// stable
const stableSort = ([...arr], [...keys]) => {
  if (keys.length <= 0) {
    return arr;
  }
  const key = keys.shift();
  const result = arr
      .sort((a, b) => ((a[key] - b[key]) || (a.$index - b.$index)))
      .map((item, $index) => ({ ...item, $index }));
  return stableSort(result, keys);
}

// example
var input = [ 
    { height: 100, weight: 80 },
    { height: 90, weight: 90 },
    { height: 70, weight: 95 },
    { height: 100, weight: 100 },
    { height: 80, weight: 110 },
    { height: 110, weight: 115 },
    { height: 100, weight: 120 },
    { height: 70, weight: 125 },
    { height: 70, weight: 130 },
    { height: 100, weight: 135 },
    { height: 75, weight: 140 },
    { height: 70, weight: 140 } 
];
stableSort(input, ['weight', 'height'];
```

## scroll bottom
```js
// tips:
//微信中web开发，input被弹出键盘挡住最简单解决方案
//在键盘弹出之前，将页面滚动到底部，就可以解决。
const value = document.body.scrollHeight - document.documentElement.clientHeight;
const setTop = (value) => {
  document.documentElement.scrollTop = document.body.scrollTop = value;
}
```

## return top
```js
// void (DOMELEMENT, [int])
export const returnTop = (dom, scrollRate = 4) => {
  let topDistance = dom.scrollTop;
  const top = () => {
    const gap = (0 - topDistance) / scrollRate;
    topDistance += gap;

    if (topDistance < 1) {
      dom.scrollTop = 0;
      return;
    }
    dom.scrollTop = topDistance;
    requestAnimationFrame(top);
  };
  top();
};
```

## getNestedValueSafe
```js
// var (Object, String"a.b.c")
export const getNestedValueSafe = (obj, keys='') =>
  keys
    .split('.')
    .reduce((a, b) => (a !== null && a !== undefined ? a[b] : undefined), obj)
```

## keep blank enter in text
```jsx
// reactElement(String)
const handleBlank = (str) => {
  let tmp = str;
  const res = [];
  while (tmp.includes(' ')) {
    const idx = tmp.indexOf(' ');
    res.push(tmp.slice(0, idx));
    res.push(' ');
    tmp = tmp.slice(idx + 1);
  }
  res.push(tmp);
  return res;
};

export const tranformStringtoHTML = (str) => {
  // handle enter
  let tmp = str;
  const enterArr = [];

  while (tmp.includes('\n')) {
    const idx = tmp.indexOf('\n');
    enterArr.push(handleBlank(tmp.slice(0, idx)));
    // ignore '\n' which index is idx
    tmp = tmp.slice(idx + 1);
  }
  enterArr.push(handleBlank(tmp));
  return (
    <div>
      {
        enterArr.map((s, i) => (
          <div key={i} style={{ minHeight: '1em' }}>
            {
              Array.isArray(s)
                ?
                s.map((ss, ii) => (
                  ss === ' ' ? <span key={ii}>&nbsp;</span> : <span key={ii}>{ss}</span>
                ))
                :
                s
            }
          </div>
        )
        )
      }
    </div>
  );
};
```

## get Url params
```js
// refers: https://www.sitepoint.com/get-url-parameters-with-javascript/
export function getUrlParams(url) {
  const d = decodeURIComponent;
  let queryString = url ? url.split('?')[1] : window.location.search.slice(1);
  const obj = {};
  if (queryString) {
    queryString = queryString.split('#')[0]; // eslint-disable-line
    const arr = queryString.split('&');
    for (let i = 0; i < arr.length; i += 1) {
      const a = arr[i].split('=');
      let paramNum;
      const paramName = a[0].replace(/\[\d*\]/, (v) => {
        paramNum = v.slice(1, -1);
        return '';
      });
      const paramValue = typeof (a[1]) === 'undefined' ? true : a[1];
      if (obj[paramName]) {
        if (typeof obj[paramName] === 'string') {
          obj[paramName] = d([obj[paramName]]);
        }
        if (typeof paramNum === 'undefined') {
          obj[paramName].push(d(paramValue));
        } else {
          obj[paramName][paramNum] = d(paramValue);
        }
      } else {
        obj[paramName] = d(paramValue);
      }
    }
  }
  return obj;
}
```

## 富文本中的图片表单上传
- 一般富文本中图片是base64，还有图片经过裁剪处理后也是base64格式的
- 图片是base64的话 需要转换为blob然后append表单中上传
```js
// get img's url in <img src="">
export const getImgSrc = (str) => {
  // [^xyz] 一个反向字符集。也就是说， 它匹配任何没有包含在方括号中的字符
  const regex = /<img [^>]*src=['"]([^'"]+)[^>]*>/gi;
  const result = [];
  let cur;

  while((cur = regex.exec(str)) != null) {
    result.push(cur[1]);
  }
  return result;
}

// is base64 format?
export const isDataURL = (str) => {
    if (str === null) {
      return false
    }
    const regex = /^\s*data:([a-z]+\/[a-z]+(;[a-z-]+=[a-z-]+)?)?(;base64)?,[a-z0-9!$&',()*+;=\-._~:@/?%\s]*\s*$/i
    return !!str.match(regex)
}

// convert base64url to blob which can be appended in form
export const convertBase64UrlToBlob = (urlData) => {
         
  var bytes=window.atob(urlData.split(',')[1]);        //去掉url的头，并转换为byte  

  // base64 由于 2^{6}=64，所以每6个比特为一个单元
  //处理异常,将ascii码小于0的转换为大于0  
  var ab = new ArrayBuffer(bytes.length); 
  var ia = new Uint8Array(ab);
  for (var i = 0; i < bytes.length; i++) {
    ia[i] = bytes.charCodeAt(i);  
  }

  return new Blob([ab] , {type : 'image/png'});
}

export const post = (resources, form) => {
  const oData = new FormData(form);

  if (isDataURL(resources.pic)) {
    // use append to solve browser compatibility, if you use set will error in safari which version is 10.1.2
    oData.append('picture', utils.convertBase64UrlToBlob(pic_base64));
    // post to server
  }
}
```

## chinese unit convert
```js
// Array[number unit](Number, [Number])
const getDigit = (integer) => {
  let tmp = integer;
  let digit = -1;
  while (tmp >= 1) {
    digit += 1;
    tmp /= 10;
  }
  return digit;
};
const addWan = (integer, number, mutiple, decimalDigit) => {
  const digit = getDigit(integer);
  if (digit > 3) {
    let remainder = digit % 8;
    if (remainder >= 5) {   // ‘十万’、‘百万’、‘千万’显示为‘万’
      remainder = 4;
    }
    return [`${Math.round(number / window.Math.pow(10, remainder + (mutiple - decimalDigit)), 10) / window.Math.pow(10, decimalDigit)}`, '万'];
  } else {
    const tmp = Math.round(number / window.Math.pow(10, mutiple - decimalDigit), 10);
    return [tmp / window.Math.pow(10, decimalDigit), ''];
  }
};

export const addChineseUnit = (number, decimalDigit = 2) => {
  const integer = Math.floor(number);
  const digit = getDigit(integer);
  // ['个', '十', '百', '千', '万', '十万', '百万', '千万'];
  const unit = [];
  if (digit > 3) {
    const multiple = Math.floor(digit / 8);
    if (multiple >= 1) {
      const tmp = Math.round(integer / (10 ** (8 * multiple)));
      const addWanArr = addWan(tmp, number, 8 * multiple, decimalDigit);
      unit.push(addWanArr[1]);
      for (let i = 0; i < multiple; i += 1) {
        unit.push('亿');
      }
      return [addWanArr[0], unit.join('')];
    } else {
      return addWan(integer, number, 0, decimalDigit);
    }
  } else {
    return [number, ''];
  }
};
```

## 动态设置viewport 如果设计图是750px 为标准, 移动端
```js
export const useScaledViewportMeta = () => {
  const phoneWidth = parseInt(window.screen.width, 10);
  const phoneScale = phoneWidth / 750;
  document.write(`<meta name="viewport" content="width=750, user-scalable=no, minimum-scale=${phoneScale
    },maximum-scale=${phoneScale}">`);
};
```