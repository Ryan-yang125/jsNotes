# 1.数据类型

Null, undefined, boolean, number, string, symbol(es6),bigint(es6),object

## 1.1 number和bigint

### number

js以64位浮点数(IEEE754)的形式储存number

![618px-IEEE_754_Double_Floating_Point_Format.svg](https://gitee.com/untermrad/picture-bed/raw/master/upic/618px-IEEE_754_Double_Floating_Point_Format.svghwfMa4.png)

转换规则如下：
$$
6.5 = 110.1_2 = 1.101*2^4 = -1^0*(1+0.101)*2^{1025-1023}=(-1)^{sign}*(1+fraction)*2^{exponent-1023}
$$
所以$sign=0,exponent=1025=10000000001_2,fraction=1010000..000_2$

为什么是$exponent-1023$ ?

因为$2^{11} = 2048$表示范围为$[0,2047]$，而指数部分可能为负，所以使用$e-1023$表示范围转换为$[-1022,1023]$,其中$e \in (0,2047)$，因此js能表示的normal的范围为$[2^{-1022},2^{1023}]$

当$e == 0$ 

- $f!=0$，表示subnormal，即比$2^{-1022}$更小的正常数,计算方法为:

$$
{\displaystyle (-1)^{\text{sign}}\times 2^{1-1023}\times 0.{\text{fraction}}=(-1)^{\text{sign}}\times 2^{-1022}\times 0.{\text{fraction}}}
$$

- $f==0$,表示$0 ||-0$,符号表示趋向的方向

当$e==2047$时

- $f == 0$,表示*Infinity*和*-Infinity*
- $f!=0$表示*NaN*

### bigint

Es6出现的bigint解决了number只能准确表示$[-(2^{53}-1),2^{53}-1]$之间的整数的问题。

当超过这个范围时，number会出现下面这种情况

![截屏2020-11-18 22.15.20](https://gitee.com/untermrad/picture-bed/raw/master/upic/%E6%88%AA%E5%B1%8F2020-11-18%2022.15.204qBOzJ.png)

这是因为*fraction*只有52位，当超过这个范围时，低位会由于空间不足被舍去。

## 1.2 null 和 undefined

undefined 在一个变量声明且未赋值时由编译器自动赋值，null需要手动赋值表示故意缺少的对象值

```javascript
null == undefined//true
null === undefined//false
typeof(null)//object
typeof(undefined)//undefined
```

About `typeof()`：

First we know  the value of js is stored with a type tag(at the lowest bits) in memory:

- 000: **object.** The data is a reference to an object.
- 1: **int**. The data is a 31 bit signed integer.
- 010: **double.** The data is a reference to a double floating point number.
- 100: **string.** The data is a reference to a string.
- 110: **boolean.** The data is a boolean.

Two values were special:

- `undefined` (`JSVAL_VOID`) was the integer $-2^{30}$ (a number outside the integer range).
- `null` (`JSVAL_NULL`) was the machine code NULL pointer. Or: an object type tag plus a reference that is zero.我们可以认为null是引用次数为0的对象

Then the engine's code for `typeof`:

<img src="https://gitee.com/untermrad/picture-bed/raw/master/upic/%E6%88%AA%E5%B1%8F2020-11-18%2017.10.03fFQNqu.png" alt="截屏2020-11-18 17.10.03" style="zoom:50%;" />

在检查（1）*undefined*的时候

```c
  #define JSVAL_IS_VOID(v)  ((v) == JSVAL_VOID)
```

在(2)*object*的时候，检查*type tag*，由于*null pointer*在内存中以$0x00$的形式存放，且不可被调用，因此判断为*object*

同时可以得到typeof的输出值:

- number
- string
- boolean
- undefined
- object
-  [Js.md](Js.md) function
- symbol(es6)
- Bitint(es6)

**补充：**

我们经常会这样使用`if...else`

```javascript
if(obj){doSomething}
if(!obj){doOtherThing}
```

为什么不管是null还是undefined都不会调用呢？

这里涉及到js中的真假值和强制类型转换

JS中7个*falsy*:

**false(boolean),+0(number),-0(number),null(null),undefined(null),''(string),NaN(number)**

除此以外全部为真值

if语句判断表达式的结果是*True还是false*，若传入非boolean类型，根据其真/假值强制转换为boolean

因此我们可以用`!!obj`根据真假值性质强制转换为boolean型

## 1.3 Symbol

