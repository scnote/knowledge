# knowledge
study document

[参考qs.js](https://github.com/ljharb/qs)
### 安装
> npm install qs

### parse(Object)
#### 参数
##### plainObjects
> 普通对象(非通过function/new xx() 创建的)
```js
assert.deepEqual(qs.parse('foo[bar]=baz'), {
    foo: {
        bar: 'baz'
    }
});
```

##### allowPrototypes
> 允许重写原型链中的属性(plainObjects 同效)不推荐使用
> URI 编码的字符串也可以
```js
var protoObject = qs.parse('a[hasOwnProperty]=b', { allowPrototypes: true });
assert.deepEqual(protoObject, { a: { hasOwnProperty: 'b' } });
```

##### depth
> 设置子对象深度
```
var deep = qs.parse('a[b][c][d][e][f][g][h][i]=j', { depth: 1 });
assert.deepEqual(deep, { a: { b: { '[c][d][e][f][g][h][i]': 'j' } } });
```
> 默认解析到深度为5的子对象
```
var expected = {
    a: {
        b: {
            c: {
                d: {
                    e: {
                        f: {
                            '[g][h][i]': 'j'
                        }
                    }
                }
            }
        }
    }
};
var string = 'a[b][c][d][e][f][g][h][i]=j';
assert.deepEqual(qs.parse(string), expected);
```
##### parameterLimit
> 限制参数数量
```
var limited = qs.parse('a=b&c=d', { parameterLimit: 1 });
assert.deepEqual(limited, { a: 'b' });
```
##### ignoreQueryPrefix
> 忽略查询前置问号
```
var prefixed = qs.parse('?a=b&c=d', { ignoreQueryPrefix: true });
assert.deepEqual(prefixed, { a: 'b', c: 'd' });
```
##### delimiter
> 按指定分隔符解析
```
var delimited = qs.parse('a=b;c=d', { delimiter: ';' });
assert.deepEqual(delimited, { a: 'b', c: 'd' });
```
> 支持正则表达式
```
var regexed = qs.parse('a=b;c=d,e=f', { delimiter: /[;,]/ });
assert.deepEqual(regexed, { a: 'b', c: 'd', e: 'f' });
```
##### allowDots
> 支持解析点符号类型对象
```
var withDots = qs.parse('a.b=c', { allowDots: true });
assert.deepEqual(withDots, { a: { b: 'c' } });
```
##### charset
> 将百分比编码的八位字节解码为 指定字符集
```
var oldCharset = qs.parse('a=%A7', { charset: 'iso-8859-1' });
assert.deepEqual(oldCharset, { a: '§' });
```
##### charsetSentinel 
> 字符集哨兵: 检测错误编码或者不同编码内容
> 指定后, utf8的参数将被忽略,转换成[charset值]/UTF-8 编码类型
> 当同时指定charset与charsetSentinel时,如果字符串中有UTF8字符集,charset指定的字符集将被覆盖为默认字符集(UTF-8)
```
var detectedAsUtf8 = qs.parse('utf8=%E2%9C%93&a=%C3%B8', {
    charset: 'iso-8859-1',
    charsetSentinel: true
});
assert.deepEqual(detectedAsUtf8, { a: 'ø' });

// Browsers encode the checkmark as &#10003; when submitting as iso-8859-1:
var detectedAsIso8859_1 = qs.parse('utf8=%26%2310003%3B&a=%F8', {
    charset: 'utf-8',
    charsetSentinel: true
});
assert.deepEqual(detectedAsIso8859_1, { a: 'ø' });
```
##### interpretNumericEntities
> 解释数字实体
> charsetSentinel 模式下也可生效
```
var detectedAsIso8859_1 = qs.parse('a=%26%239786%3B', {
    charset: 'iso-8859-1',
    interpretNumericEntities: true
});
assert.deepEqual(detectedAsIso8859_1, { a: '☺' });
```
#### 其它示例
```
assert.deepEqual(qs.parse('foo[bar][baz]=foobarbaz'), {
    foo: {
        bar: {
            baz: 'foobarbaz'
        }
    }
});
```

### parse(Array)
#### 基础特性
> 字符串有无指定索引都可转换
> 生成的数组跳过空索引
```
var noSparse = qs.parse('a[1]=b&a[15]=c');
assert.deepEqual(noSparse, { a: ['b', 'c'] });
```
> 空值也将被保留
```
var withEmptyString = qs.parse('a[]=&a[]=b');
assert.deepEqual(withEmptyString, { a: ['', 'b'] });

var withIndexedEmptyString = qs.parse('a[0]=b&a[1]=&a[2]=c');
assert.deepEqual(withIndexedEmptyString, { a: ['b', '', 'c'] });
```
> 可以生成对象数组
```
var arraysOfObjects = qs.parse('a[][b]=c');
assert.deepEqual(arraysOfObjects, { a: [{ b: 'c' }] });
```

#### 参数
##### allowSparse
> 生成的数组中  ,给空索引分配位置
```
var sparseArray = qs.parse('a[1]=2&a[3]=5', { allowSparse: true });
assert.deepEqual(sparseArray, { a: [, '2', , '5'] });
```
##### arrayLimit
> 限制数组大小,默认最大值20; 大于20会将索引值作为KEY生成对象
```
var withMaxIndex = qs.parse('a[100]=b');
assert.deepEqual(withMaxIndex, { a: { '100': 'b' } });
```
##### parseArrays
> 是否转换为数组
```
var noParsingArrays = qs.parse('a[]=b', { parseArrays: false });
assert.deepEqual(noParsingArrays, { a: { '0': 'b' } });

var mixedNotation = qs.parse('a[0]=b&a[b]=c');
assert.deepEqual(mixedNotation, { a: { '0': 'b', b: 'c' } });
```
##### comma
> 使用逗号生成数组 , 无法通过逗号生成对象数组,例:  
```
var arraysOfObjects = qs.parse('a=b,c', { comma: true })
assert.deepEqual(arraysOfObjects, { a: ['b', 'c'] })
```





#### 其它示例
```
var withArray = qs.parse('a[]=b&a[]=c');
assert.deepEqual(withArray, { a: ['b', 'c'] });

var withIndexes = qs.parse('a[1]=c&a[0]=b');
assert.deepEqual(withIndexes, { a: ['b', 'c'] });
```
### Stringifying
#### 基础特性
> 默认 URI 对输出进行编码
```
assert.equal(qs.stringify({ a: 'b' }), 'a=b');
assert.equal(qs.stringify({ a: { b: 'c' } }), 'a%5Bb%5D=c');
```
> 对数组进行编码时,默认为数组添加索引
```
qs.stringify({ a: ['b', 'c', 'd'] });
// 'a[0]=b&a[1]=c&a[2]=d'
```
> 当对象被字符串化时，默认情况下它们使用括号表示法：
```
qs.stringify({ a: { b: { c: 'd', e: 'f' } } });
// 'a[b][c]=d&a[b][e]=f'
```
> 空值将会被忽略,但是在字符串中还会保留key值
```
assert.equal(qs.stringify({ a: '' }), 'a=');
```
> 空对象返回空
```
assert.equal(qs.stringify({ a: [] }), '');
assert.equal(qs.stringify({ a: {} }), '');
assert.equal(qs.stringify({ a: [{}] }), '');
assert.equal(qs.stringify({ a: { b: []} }), '');
assert.equal(qs.stringify({ a: { b: {}} }), '');
```
> undefined值 直接被省略
```
assert.equal(qs.stringify({ a: null, b: undefined }), 'a=');
```


#### 参数
##### encode
> 是否编码
```
var unencoded = qs.stringify({ a: { b: 'c' } }, { encode: false });
assert.equal(unencoded, 'a[b]=c');
```
##### encodeValuesOnly
> 只对值编码
```
var encodedValues = qs.stringify(
    { a: 'b', c: ['d', 'e=f'], f: [['g'], ['h']] },
    { encodeValuesOnly: true }
);
assert.equal(encodedValues,'a=b&c[0]=d&c[1]=e%3Df&f[0][0]=g&f[1][0]=h');
```
##### encoder
> 编码函数
> 只在encode: true的情况下生效
```
var encoded = qs.stringify({ a: { b: 'c' } }, { encoder: function (str) {
    // Passed in values `a`, `b`, `c`
    return // Return encoded string
}})

var encoded = qs.stringify({ a: { b: 'c' } }, { encoder: function (str, defaultEncoder, charset, type) {
    if (type === 'key') {
        return // Encoded key
    } else if (type === 'value') {
        return // Encoded value
    }
}})
```
##### decoder
> 解码函数
```
var decoded = qs.parse('x=z', { decoder: function (str) {
    // Passed in values `x`, `z`
    return // Return decoded string
}})

var decoded = qs.parse('x=z', { decoder: function (str, defaultDecoder, charset, type) {
    if (type === 'key') {
        return // Decoded key
    } else if (type === 'value') {
        return // Decoded value
    }
}})
```
##### indices
> 不为数组添加索引
```
qs.stringify({ a: ['b', 'c', 'd'] }, { indices: false });
// 'a=b&a=c&a=d'
```
##### arrayFormat
> 指定数组编码方式
```
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'indices' })
// 'a[0]=b&a[1]=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'brackets' })
// 'a[]=b&a[]=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'repeat' })
// 'a=b&a=c'
qs.stringify({ a: ['b', 'c'] }, { arrayFormat: 'comma' })
// 'a=b,c'
```
> 当使用 arrayFormat 设置为 'comma' 时，设置commaRoundTrip true/false，以在单项数组上附加 []，以便它们可以在解析中往返。
##### allowDots
> 对象编码时用点来表示
```
qs.stringify({ a: { b: { c: 'd', e: 'f' } } }, { allowDots: true });
// 'a.b.c=d&a.b.e=f'
```
##### addQueryPrefix
> 添加前置问号
```
assert.equal(qs.stringify({ a: 'b', c: 'd' }, { addQueryPrefix: true }), '?a=b&c=d');
```
##### delimiter
> 指定分隔符
```
assert.equal(qs.stringify({ a: 'b', c: 'd' }, { delimiter: ';' }), 'a=b;c=d');
```
##### serializeDate
> 只覆盖 Date 对象的序列化
```
var date = new Date(7);
assert.equal(qs.stringify({ a: date }), 'a=1970-01-01T00:00:00.007Z'.replace(/:/g, '%3A'));
assert.equal(
    qs.stringify({ a: date }, { serializeDate: function (d) { return d.getTime(); } }),
    'a=7'
);
```
##### sort
> 指定排序方法
```
function alphabeticalSort(a, b) {
    return a.localeCompare(b);
}
assert.equal(qs.stringify({ a: 'c', z: 'y', b : 'f' }, { sort: alphabeticalSort }), 'a=c&b=f&z=y');
```
##### filter
> 筛选需要编码的key,以及value的处理
```
function filterFunc(prefix, value) {
    if (prefix == 'b') {
        // Return an `undefined` value to omit a property.
        return;
    }
    if (prefix == 'e[f]') {
        return value.getTime();
    }
    if (prefix == 'e[g][0]') {
        return value * 2;
    }
    return value;
}
qs.stringify({ a: 'b', c: 'd', e: { f: new Date(123), g: [2] } }, { filter: filterFunc });
// 'a=b&c=d&e[f]=123&e[g][0]=4'
qs.stringify({ a: 'b', c: 'd', e: 'f' }, { filter: ['a', 'e'] });
// 'a=b&e=f'
qs.stringify({ a: ['b', 'c', 'd'], e: 'f' }, { filter: ['a', 0, 2] });
// 'a[0]=b&a[2]=d'
```
```
https://domain.com/endpoint?range=30...70

class Range {
    constructor(from, to) {
        this.from = from;
        this.to = to;
    }
}

qs.stringify(
    {
        range: new Range(30, 70),
    },
    {
        filter: (prefix, value) => {
            if (value instanceof Range) {
                return `${value.from}...${value.to}`;
            }
            // serialize the usual way
            return value;
        },
    }
);
// range=30...70
```
##### charset
> 指定编码
```
var numeric = qs.stringify({ a: '☺' }, { charset: 'iso-8859-1' });
assert.equal(numeric, 'a=%26%239786%3B');
```
> 指定编码中不存在的字符,被串化为数字实体
```
var numeric = qs.stringify({ a: '☺' }, { charset: 'iso-8859-1' });
assert.equal(numeric, 'a=%26%239786%3B');
```
### 有关null
> 默认情况下，空值被视为空字符串
```
var withNull = qs.stringify({ a: null, b: '' });
assert.equal(withNull, 'a=&b=');
```
> 不区分是否带等号
```
var equalsInsensitive = qs.parse('a&b=');
assert.deepEqual(equalsInsensitive, { a: '', b: '' });
```

### 其它参数
##### strictNullHandling
> 区分空值和空字符串。在结果字符串中，空值没有 = 符号
```
var strictNull = qs.stringify({ a: null, b: '' }, { strictNullHandling: true });
assert.equal(strictNull, 'a&b=');
```
> 解析时,没有等号的key值,赋值为null
```
var parsedStrictNull = qs.parse('a&b=', { strictNullHandling: true });
assert.deepEqual(parsedStrictNull, { a: null, b: '' });
```
##### skipNulls
> 串化时跳过null值
```
var nullsSkipped = qs.stringify({ a: 'b', c: null}, { skipNulls: true });
assert.equal(nullsSkipped, 'a=b');
```

### 其它示例
```
var primitiveValues = qs.parse('a=15&b=true&c=null');
assert.deepEqual(primitiveValues, { a: '15', b: 'true', c: 'null' });
```
```
var iso = qs.stringify({ æ: 'æ' }, { charset: 'iso-8859-1' });
assert.equal(iso, '%E6=%E6');

var sentinel = qs.stringify({ a: '☺' }, { charsetSentinel: true });
assert.equal(sentinel, 'utf8=%E2%9C%93&a=%E2%98%BA');

var isoSentinel = qs.stringify({ a: 'æ' }, { charsetSentinel: true, charset: 'iso-8859-1' });
assert.equal(isoSentinel, 'utf8=%26%2310003%3B&a=%E6');

var encoder = require('qs-iconv/encoder')('shift_jis');
var shiftJISEncoded = qs.stringify({ a: 'こんにちは！' }, { encoder: encoder });
assert.equal(shiftJISEncoded, 'a=%82%B1%82%F1%82%C9%82%BF%82%CD%81I');

var decoder = require('qs-iconv/decoder')('shift_jis');
var obj = qs.parse('a=%82%B1%82%F1%82%C9%82%BF%82%CD%81I', { decoder: decoder });
assert.deepEqual(obj, { a: 'こんにちは！' });

assert.equal(qs.stringify({ a: 'b c' }), 'a=b%20c');
assert.equal(qs.stringify({ a: 'b c' }, { format : 'RFC3986' }), 'a=b%20c');
assert.equal(qs.stringify({ a: 'b c' }, { format : 'RFC1738' }), 'a=b+c');
```
