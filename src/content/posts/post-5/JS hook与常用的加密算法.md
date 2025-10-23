
---
title: JS hook与常用的加密算法
published: 2025-10-23
description: ''
image: ''
tags: ['hook', 'code']
category: 代码日常
draft: false
lang: zh-CN
---
    ## Hook
```js
// ========== Hook全局函数 ==========

// Hook eval函数
(function() {
    const originalEval = eval;
    eval = function() {
        console.log("[Hook] eval被调用", arguments);
        debugger;
        const result = originalEval.apply(this, arguments);
        console.log("[Hook] eval调用完成:", result);
        return result;
    };
})();

// Hook JSON.stringify
(function() {
    const originalStringify = JSON.stringify;
    JSON.stringify = function() {
        console.log("[Hook] JSON.stringify被调用:", arguments);
        debugger;
        const result = originalStringify.apply(this, arguments);
        console.log("[Hook] JSON.stringify调用完成:", result);
        return result;
    };
})();

// Hook JSON.parse
(function() {
    const originalParse = JSON.parse;
    JSON.parse = function() {
        console.log("[Hook] JSON.parse被调用:", arguments);
        debugger;
        const result = originalParse.apply(this, arguments);
        console.log("[Hook] JSON.parse调用完成:", result);
        return result;
    };
})();

// ========== Hook Cookie操作 ==========
(function() {
    const targetCookie = 'a1';
    let cookieValue = "";
    
    Object.defineProperty(document, "cookie", {
        get: () => cookieValue,
        set: (value) => {
            console.log("[Hook] Cookie设置值:", value);
            if (value.includes(targetCookie)) {
                debugger;
            }
            cookieValue = value;
            return value;
        }
    });
})();

// ========== Hook HTTP头 ==========
(function() {
    const originalSetHeader = XMLHttpRequest.prototype.setRequestHeader;
    
    XMLHttpRequest.prototype.setRequestHeader = function(header, value) {
        if (header.toUpperCase() === 'X-REQUESTED-WITH'.toUpperCase()) {
            console.log("[Hook] 拦截到特殊请求头:", header, value);
            debugger;
        }
        return originalSetHeader.apply(this, arguments);
    };
})();

// ========== Hook全局属性 ==========
(function() {
    const targetProperty = 'name';
    let propertyValue;
    
    Object.defineProperty(window, targetProperty, {
        set(value) {
            console.log(`[Hook] ${targetProperty}被修改为:`, value);
            debugger;
            propertyValue = value;
        },
        get() {
            return propertyValue;
        },
        enumerable: true,
        configurable: true
    });
})();

// ========== debugger ==========
(function() {
    _Function = Function.prototype.constructor;
    Function.prototype.constructor = function() {
        return _Function.apply(this, arguments[0].replace("debugger", ''));
    };

	_eval = eval;
	eval = function(code) {
        return _eval.apply(this, code.replace("debugger", ''));
    };
})();

// ========== 禁用调试器 ==========
(function() {
    setInterval = function() {};
    setTimeout = function() {};
    console.log("[Hook] 已禁用定时器函数");
})();
```

## 常见的加解密算法

#### 标准库
```js
/* md5标准小写 */
const CryptoJS = require('crypto-js');
const sign = CryptoJS.MD5(myString).toString();

/* sha系列 */
const CryptoJS = require('crypto-js');
str1 = CryptoJS.SHA256('1').toString()
str2 = CryptoJS.SHA1('1').toString()
str3 = CryptoJS.SHA512('1').toString()

// sm系列
const sm3 = require('sm-crypto').sm3;

// base64解密
data = Buffer.from('aHR0cHM6Ly9kYy5zaW11d2FuZy5jb20v', 'base64').toString('binary')

console.log(data)
```

#### JS md5源码

```js
function md5(string) {
  function safeAdd(x, y) {
    var lsw = (x & 0xffff) + (y & 0xffff);
    var msw = (x >> 16) + (y >> 16) + (lsw >> 16);
    return (msw << 16) | (lsw & 0xffff);
  }

  function bitRotateLeft(num, cnt) {
    return (num << cnt) | (num >>> (32 - cnt));
  }

  function md5cmn(q, a, b, x, s, t) {
    return safeAdd(bitRotateLeft(safeAdd(safeAdd(a, q), safeAdd(x, t)), s), b);
  }

  function md5ff(a, b, c, d, x, s, t) {
    return md5cmn((b & c) | (~b & d), a, b, x, s, t);
  }

  function md5gg(a, b, c, d, x, s, t) {
    return md5cmn((b & d) | (c & ~d), a, b, x, s, t);
  }

  function md5hh(a, b, c, d, x, s, t) {
    return md5cmn(b ^ c ^ d, a, b, x, s, t);
  }

  function md5ii(a, b, c, d, x, s, t) {
    return md5cmn(c ^ (b | ~d), a, b, x, s, t);
  }

  function binlMD5(x, len) {
    /* append padding */
    x[len >> 5] |= 0x80 << len % 32;
    x[((len + 64) >>> 9 << 4) + 14] = len;

    var a = 1732584193;
    var b = -271733879;
    var c = -1732584194;
    var d = 271733878;

    for (var i = 0; i < x.length; i += 16) {
      var olda = a;
      var oldb = b;
      var oldc = c;
      var oldd = d;

      a = md5ff(a, b, c, d, x[i], 7, -680876936);
      d = md5ff(d, a, b, c, x[i + 1], 12, -389564586);
      c = md5ff(c, d, a, b, x[i + 2], 17, 606105819);
      b = md5ff(b, c, d, a, x[i + 3], 22, -1044525330);
      a = md5ff(a, b, c, d, x[i + 4], 7, -176418897);
      d = md5ff(d, a, b, c, x[i + 5], 12, 1200080426);
      c = md5ff(c, d, a, b, x[i + 6], 17, -1473231341);
      b = md5ff(b, c, d, a, x[i + 7], 22, -45705983);
      a = md5ff(a, b, c, d, x[i + 8], 7, 1770035416);
      d = md5ff(d, a, b, c, x[i + 9], 12, -1958414417);
      c = md5ff(c, d, a, b, x[i + 10], 17, -42063);
      b = md5ff(b, c, d, a, x[i + 11], 22, -1990404162);
      a = md5ff(a, b, c, d, x[i + 12], 7, 1804603682);
      d = md5ff(d, a, b, c, x[i + 13], 12, -40341101);
      c = md5ff(c, d, a, b, x[i + 14], 17, -1502002290);
      b = md5ff(b, c, d, a, x[i + 15], 22, 1236535329);

      a = md5gg(a, b, c, d, x[i + 1], 5, -165796510);
      d = md5gg(d, a, b, c, x[i + 6], 9, -1069501632);
      c = md5gg(c, d, a, b, x[i + 11], 14, 643717713);
      b = md5gg(b, c, d, a, x[i], 20, -373897302);
      a = md5gg(a, b, c, d, x[i + 5], 5, -701558691);
      d = md5gg(d, a, b, c, x[i + 10], 9, 38016083);
      c = md5gg(c, d, a, b, x[i + 15], 14, -660478335);
      b = md5gg(b, c, d, a, x[i + 4], 20, -405537848);
      a = md5gg(a, b, c, d, x[i + 9], 5, 568446438);
      d = md5gg(d, a, b, c, x[i + 14], 9, -1019803690);
      c = md5gg(c, d, a, b, x[i + 3], 14, -187363961);
      b = md5gg(b, c, d, a, x[i + 8], 20, 1163531501);
      a = md5gg(a, b, c, d, x[i + 13], 5, -1444681467);
      d = md5gg(d, a, b, c, x[i + 2], 9, -51403784);
      c = md5gg(c, d, a, b, x[i + 7], 14, 1735328473);
      b = md5gg(b, c, d, a, x[i + 12], 20, -1926607734);

      a = md5hh(a, b, c, d, x[i + 5], 4, -378558);
      d = md5hh(d, a, b, c, x[i + 8], 11, -2022574463);
      c = md5hh(c, d, a, b, x[i + 11], 16, 1839030562);
      b = md5hh(b, c, d, a, x[i + 14], 23, -35309556);
      a = md5hh(a, b, c, d, x[i + 1], 4, -1530992060);
      d = md5hh(d, a, b, c, x[i + 4], 11, 1272893353);
      c = md5hh(c, d, a, b, x[i + 7], 16, -155497632);
      b = md5hh(b, c, d, a, x[i + 10], 23, -1094730640);
      a = md5hh(a, b, c, d, x[i + 13], 4, 681279174);
      d = md5hh(d, a, b, c, x[i], 11, -358537222);
      c = md5hh(c, d, a, b, x[i + 3], 16, -722521979);
      b = md5hh(b, c, d, a, x[i + 6], 23, 76029189);
      a = md5hh(a, b, c, d, x[i + 9], 4, -640364487);
      d = md5hh(d, a, b, c, x[i + 12], 11, -421815835);
      c = md5hh(c, d, a, b, x[i + 15], 16, 530742520);
      b = md5hh(b, c, d, a, x[i + 2], 23, -995338651);

      a = md5ii(a, b, c, d, x[i], 6, -198630844);
      d = md5ii(d, a, b, c, x[i + 7], 10, 1126891415);
      c = md5ii(c, d, a, b, x[i + 14], 15, -1416354905);
      b = md5ii(b, c, d, a, x[i + 5], 21, -57434055);
      a = md5ii(a, b, c, d, x[i + 12], 6, 1700485571);
      d = md5ii(d, a, b, c, x[i + 3], 10, -1894986606);
      c = md5ii(c, d, a, b, x[i + 10], 15, -1051523);
      b = md5ii(b, c, d, a, x[i + 1], 21, -2054922799);
      a = md5ii(a, b, c, d, x[i + 8], 6, 1873313359);
      d = md5ii(d, a, b, c, x[i + 15], 10, -30611744);
      c = md5ii(c, d, a, b, x[i + 6], 15, -1560198380);
      b = md5ii(b, c, d, a, x[i + 13], 21, 1309151649);
      a = md5ii(a, b, c, d, x[i + 4], 6, -145523070);
      d = md5ii(d, a, b, c, x[i + 11], 10, -1120210379);
      c = md5ii(c, d, a, b, x[i + 2], 15, 718787259);
      b = md5ii(b, c, d, a, x[i + 9], 21, -343485551);

      a = safeAdd(a, olda);
      b = safeAdd(b, oldb);
      c = safeAdd(c, oldc);
      d = safeAdd(d, oldd);
    }
    return [a, b, c, d];
  }

  function binl2hex(binarray) {
    var hexTab = '0123456789abcdef';
    var str = '';
    for (var i = 0; i < binarray.length * 4; i++) {
      str +=
        hexTab.charAt((binarray[i >> 2] >> (i % 4 * 8 + 4)) & 0xf) +
        hexTab.charAt((binarray[i >> 2] >> (i % 4 * 8)) & 0xf);
    }
    return str;
  }

  function str2binl(str) {
    var bin = [];
    var mask = (1 << 8) - 1;
    for (var i = 0; i < str.length * 8; i += 8) {
      bin[i >> 5] |= (str.charCodeAt(i / 8) & mask) << i % 32;
    }
    return bin;
  }

  return binl2hex(binlMD5(str2binl(string), string.length * 8));
}
```

#### 标准AES加密
##### 代码1

```js

const CryptoJS = require('crypto-js');

// AES加密
const key = "bW8Jc63Uz7I1N5R9";
const iv = "0000000000000000";

const kt = {
    encrypt(e) {
        let t = e;
        // 把key和iv解析为 WordArray
        let n = CryptoJS.enc.Utf8.parse(key);
        let a = CryptoJS.enc.Utf8.parse(iv);
        
		// 传递参数：t 为要加密的文本，n 为密钥，a 为 iv
        let o = CryptoJS.AES.encrypt(t, n, {
            iv: a,
            mode: CryptoJS.mode.CBC,
            padding: CryptoJS.pad.Pkcs7
        });

        // 将二进制结果转换为base64作为输出结果
        return o.toString(CryptoJS.enc.Base64);
    },

    decrypt(e) {
	    // 把key和iv解析为 WordArray
        let t = CryptoJS.enc.Utf8.parse(key);
        let n = CryptoJS.enc.Utf8.parse(iv);

        // 传递参数：e 为加密文本 t为key n为iv
        let r = CryptoJS.AES.decrypt(e, t, {
            iv: n,
            mode: CryptoJS.mode.CBC,
            padding: CryptoJS.pad.Pkcs7
        });
		// 将二进制结果转换为utf-8作为输出结果
        return r.toString(CryptoJS.enc.Utf8);
    }
};

// 测试
const originalText = '{"header":{"sendSysname":"lcw-fe-front","msgType":"lcw-fe-front.vue","msgId":"lcw-fe-front32432490121b69cafgwe470ea68565023424hld31","sendTime":"2025/7/30 18:19:25","exts":{}},"body":{"version":"1.1","trnsType":"lcw-fe-front.vue","trnsId":"lcw-fe-front32432490121b69cafgwe470ea68565023424hld31","exts":{"searchType":"","quickQueryWords":"","currentPage":1,"pageSize":20,"prodRegCode":"","prodName":"","orgName":"","prodStatus":"02","orgTypeList":[],"prodCollectMeth":"01,NA","prodAttr":"","prodOperateModeList":[],"prodInvestNatureList":[],"prodRiskLevelList":[],"prodTermCodeList":[],"collCCYList":[],"performanceCompareBaseCap":"","performanceCompareBaseFloor":"","prodSaleZone":"","orderConfig":"","userCode":""}}}';

const encryptedText = kt.encrypt(originalText);
console.log("加密结果:", encryptedText);

const decryptedText = kt.decrypt(encryptedText);
console.log("解密结果:", decryptedText);
```

##### 代码2

```js
const crypto = require('crypto');

function aesEncrypt(plaintext, key, iv) {
  // 确保key和iv是Buffer
  const keyBuf = Buffer.isBuffer(key) ? key : Buffer.from(key, 'utf8');
  const ivBuf = Buffer.isBuffer(iv) ? iv : Buffer.from(iv, 'utf8');
  
  // 创建加密器
  const cipher = crypto.createCipheriv('aes-128-cbc', keyBuf, ivBuf);
  
  // 加密数据
  let encrypted = cipher.update(plaintext, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  return encrypted;
}


function aesDecrypt(encrypted, key, iv) {
  const keyBuf = Buffer.isBuffer(key) ? key : Buffer.from(key, 'utf8');
  const ivBuf = Buffer.isBuffer(iv) ? iv : Buffer.from(iv, 'utf8');
  
  const decipher = crypto.createDecipheriv('aes-128-cbc', keyBuf, ivBuf);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

const key = "bW8Jc63Uz7I1N5R9";
const iv = "0000000000000000";

data = '{"header":{"sendSysname":"lcw-fe-front","msgType":"lcw-fe-front.vue","msgId":"lcw-fe-front32432490121b69cafgwe470ea68565023424hld31","sendTime":"2025/7/30 18:19:25","exts":{}},"body":{"version":"1.1","trnsType":"lcw-fe-front.vue","trnsId":"lcw-fe-front32432490121b69cafgwe470ea68565023424hld31","exts":{"searchType":"","quickQueryWords":"","currentPage":1,"pageSize":20,"prodRegCode":"","prodName":"","orgName":"","prodStatus":"02","orgTypeList":[],"prodCollectMeth":"01,NA","prodAttr":"","prodOperateModeList":[],"prodInvestNatureList":[],"prodRiskLevelList":[],"prodTermCodeList":[],"collCCYList":[],"performanceCompareBaseCap":"","performanceCompareBaseFloor":"","prodSaleZone":"","orderConfig":"","userCode":""}}}'

encrypted = aesEncrypt(data, key, iv)
decrypted = aesDecrypt(encrypted, key, iv)
console.log(decrypted)
```