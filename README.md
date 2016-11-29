# simple-json-schema

## 问题
+ 使用json书写json schema不方便
+ json schema嵌套层次深

## 目标
+ 简化json schema的书写

## 方案

参考React.propTypes, tcomb使用js表达变量的类型、结构、约束。

### 方案1
以简洁的函数链式调用的方式，书写schema。

#### 基本类型
提供5中基本类型`Str`、`Int`、`Num`、`Arr`、`Obj`，通过一个对象ET使用。

```
export default {
    get str() {
        return new Str();
    }
    get Int() {
        return new Int();
    }
    // 其他类型...
}

// 使用
import ET from 'ecom-type';

let schema = {
    name: ET.str
};
```

+ 每种类型对应的校验关键词作为该类型的方法；

```
let schema = {
    name: ET.str.pattern(/xxxx/),
    files: ET.arr.items(ET.str)
};
```

+ schema默认为`required`，如不是必须字段，调用`mayBe`方法；

```
let schema = {
    name: ET.str.pattern(/xxxx/),
    files: ET.arr.items(ET.str).mayBe()
};
```

+ json schema的`string`、`array`校验关键词`maxLength`、`minLength`对应`length`方法;

```
let schema = {
    name: ET.str.length(20, 5),
    files: ET.arr.items(ET.str).mayBe()
};

// length方法签名
function length(maxLength, minLength) {}

```

+ `array`的**unique**对应**uniq**；
+ `number`、`integer`的**max, exclusiveMaximum, min, exclusiveMinmum**对应**range**；

```
let schema = {
    // ET.int.range(100, 16, true)
    // ET.int.range(100, 16)
    // ET.int.range(null, 16)
    // ET.int.range(100)
    age: ET.int.range(100, true, 16, true)
};
```

方案2

+ 定义`Str`, `Int`, `Num`, `Arr`, `Obj`类对应json schema中的`type`的取值；

```
let schema = {
    name: Str(),
    foo: Obj()
};
```

+ 约束字段作为参数传给上述类；

```
let schema = {
    name: Str({
        maxLength: 100,
        minLength: 10,
        pattern: /xxxx.*/,
        required: true
    }),
    foo: Obj({
        properties: {
            key1: Str({minLength: 1}),
            key2: Arr({items: Int({max: 10})})
        }
    })
};

```

+ 定义`enum`, `allOf`, `oneOf`, `not`, `anyOf`函数，返回值为schema，对应json schema中关键字；

```
let schema = {
    gender: enum('男', '女'),
    bar: allOf(Str(), Num(), enum(1, 2))
};
```

+ 定义`format`对象，支持json schema提供的format，开放自定义`format`

```
let schema = {
    birthday: format.date(),
    blog: format.uri()
};
```

### 完整的example

#### js schema

```
let schema = {
    name: Str({
        maxLength: 100,
        minLength: 10,
        pattern: /xxxx.*/,
        required: true
    }),
    gender: enum('男', '女'),
    foo: Obj({
        properties: {
            key1: Str({minLength: 1}),
            key2: Arr({items: Int({max: 10})})
        }
    }),
    bar: allOf(Str(), Num()),
    birthday: format.date()
};
```

#### json schema
```
{
    "id": "",
    "title": "",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "maxLength": 1000,
            "minLength": 10,
            "pattern": /xxx.*/
        },
        "gender": {
            "enum": ['男', '女']
        },
        "foo": {
            "type": "object",
            "properties": {
                "key1": {
                    "type": "string",
                    "minLength": 1
                },
                "key2": {
                    "type": "array",
                    "items": {
                        "type": "integer",
                        "max": 20
                    }
                }
            }
        },
        "bar": {
            "allOf": [
                {"type": "string"},
                {"type": "number"}
            ]
        },
        "birthday": {"format": "date"}
    },
    "required": ["name"]
}
```
