---
date: '2023-11-03T11:05:16+08:00'
lastmod: '2023-11-03T11:05:16+08:00'
draft: true
title: bibi_gengine 使用入门
slug: bibi-gengine-getting-started
description: '- `gengine` 是一款基于 golang 和 AST (抽象语法树)开发的规则引擎, `gengine` 支持的语法是一种自定义的'
tags:
- AI
categories:
- AI
comment: false
toc: false
---

> ### [gengine简介](https://github.com/bilibili/gengine/wiki/gengine%E7%AE%80%E4%BB%8B#gengine%E7%AE%80%E4%BB%8B)
>
> - gengine是一款基于golang和AST(抽象语法树)开发的规则引擎,gengine支持的语法是一种自定义的DSL
> - gengine于2020年7月由哔哩哔哩(bilibili.com)授权开源
> - gengine现已应用于B站风控系统、流量投放系统、AB测试、推荐平台系统等多个业务场景
> - 你也可以将gengine应用于golang应用的任何需要规则或指标支持的业务场景



## 快速入门

### gengine简介

- `gengine` 是一款基于 golang 和 AST (抽象语法树)开发的规则引擎, `gengine` 支持的语法是一种自定义的 DSL
- `gengine` 于2020年7月由哔哩哔哩 (`bilibili.com`)授权开源
- `gengine` 现已应用于 B 站风控系统、流量投放系统、AB 测试、推荐平台系统等多个业务场景
- 你也可以将 `gengine` 应用于 `golang` 应用的任何需要规则或指标支持的业务场景

###  相似工具对比

|              | gengine                                             | gopher-lua                                 |
| ------------ | --------------------------------------------------- | ------------------------------------------ |
| 执行模式     |                                                     |                                            |
| 规则便携难度 |                                                     |                                            |
| 功能         | 功能较少，适合简单的逻辑处理                        | 支持 lua 语法，功能强大                                           |
| 性能         |                                                     |                                            |
| github stars |                                                     |                                            |
| 代码更新频率 |                                                     |                                            |
| 文档         | 全面，完整 https://github.com/bilibili/gengine/wiki | 少 https://github.com/yuin/gopher-lua/wiki |

### 安装

```bash
go get github.com/bilibili/gengine v1.5.3
```

最新tag: [https://github.com/bilibili/gengine/tags](https://github.com/bilibili/gengine/tags)

### 一个例子

```go
package example

import (
	"fmt"
	"time"

	"github.com/bilibili/gengine/builder"
	"github.com/bilibili/gengine/context"
	"github.com/bilibili/gengine/engine"
)

const compare_user_rule1 = `
rule "compare user.Name rule" "user name"  salience 0
begin
	if User1.Name == User2.Name{
		return true
	}else{
		return false
	}
end
rule "compare user.Age rule" "user age"  salience 0
begin
	if User1.Age == User2.Age{
		return true
	}else{
		return false
	}
end
`

// 定义想要注入的结构体
type User struct {
	Name string
	Age  int64
	Male bool
}


func CompareUser(user1, user2 *User, rule string) map[string]interface{} {
	dataContext := context.NewDataContext()
	//注入初始化的结构体
	dataContext.Add("User1", user1)
	dataContext.Add("User2", user2)

	//init rule engine
	ruleBuilder := builder.NewRuleBuilder(dataContext)

	start1 := time.Now().UnixNano()
	//构建规则
	err := ruleBuilder.BuildRuleFromString(rule) //string(bs)
	end1 := time.Now().UnixNano()

	fmt.Printf("[Info] rules num:%d, load rules cost time:%d\n", len(ruleBuilder.Kc.RuleEntities), end1-start1)

	if err != nil {
		fmt.Printf("[Error] err:%s \n", err)
	} else {
		eng := engine.NewGengine()
		start := time.Now().UnixNano()
		//执行规则
		err := eng.Execute(ruleBuilder, true)
		end := time.Now().UnixNano()
		if err != nil {
			fmt.Printf("[Error] execute rule error: %v\n", err)
		}
		fmt.Printf("[Info] execute rule cost %d ns\n", end-start)
		resultMap, _ := eng.GetRulesResultMap()
		fmt.Printf("[Info] execute rule result %+v\n", resultMap)
		return resultMap
	}
	return nil
}
```

#### 示例解释

- `User` 是需要被注入到 `gengine` 中的结构体；结构体在注入之前需要被初始化；结构体需要以指针的形式被注入, 否则无法在规则中改变其属性值
- `compare_user_rule1` 是以字符串定义的具体规则
- `dataContext` 用于接受注入的数据 (结构体、方法等)
- `ruleBuilder` 用于编译字符串形式的规则
- `engine` 接受 `ruleBuilder`, 并以用户选定的执行模式执行加载好的规则
- `GetRulesResultMap()` 返回 `compare_user_rule` 1 中每个规则的结果

## 语法

### gengine DSL语法

```go
const rule = `
rule "rulename" "rule-describtion" salience  10
begin

//规则体

end`
```

如上,gengine DSL完整的语法块由如下几个组件构成:

- 关键字rule,之后紧跟"规则名称"和"规则描述",规则名称是必须的,但规则描述不是必须的. 当**一个gengine实例中有多个规则时,"规则名"必须唯一**,否则当有多个相同规则名的规则时,编译好之后只会存在一个
- 关键字 `salience`, 之后紧跟一个整数, 表示的规则优先级, 它们为非必须. 
	- 数字越大, 规则优先级越高; 
	- 当用户没有显式的指明优先级时, 规则的优先级未知, 如果多个规则的优先级相同，那么在执行的时候，相同优先级的规则执行顺序未知
- 关键字 `begin` 和 end` `包裹的是规则体, 也就是规则的具体逻辑

#### 注释

- 支持规则内的单行注释,注释以双斜杠(//)开头
### 支持的数据类型

- string
- bool
- int, int8, int16, int32, int64
- uint, uint8, uint16, uint32, uint64
- float32, float64

#### 数据结构
gengine主要支持4种数据结构,分别是golang的struct,以及基础类型的map、array和slice, 为了保持简洁、以及语言的无关性, 支持这4种结构的如下操作:
##### struct

**注入struct必须是以指针的形式注入,否则无法在规则中改变结构体的属性值**,注入结构体示例如下:

```go
const rule = `
rule "name test" "struct" salience 10
begin
	// User 是 注入的变量
	// println 是注入的函数
	println(User.Age)
end
`
user := &User{
	Name: "Calo",
	Age:  0,
	Male: true,
}

dataContext := context.NewDataContext()
//inject
dataContext.Add("User", user)
```

##### map

`gengine` 只能基于 `key` 对 `map` 取值或设置值。

```go
const rule = `
rule "name test" "map" salience 10
begin
	// Users 是 注入的变量
	Users["Calo"] = 20
	Users["Jack"] = 29
end
`
users := make(map[string]int, 0)

dataContext := context.NewDataContext()
//inject
dataContext.Add("Users", users)
```

##### 数组

`gengine` 只能基于 `index` 对数组取值或设置值, 且注入的数组, 必须为**指针数组**, 或者附着与结构体指针的数组, **当以非指针形式的数组注入 `dataContext`. 则只能用 index 取值**, 而不能设置值

```go
const rule = `
rule "name test" "array" salience 10
begin
	// AS 是 注入的变量
	AS.MI[1] = 20
	AS.MM[2] = 21
	println(AS.MM[1])
	AA.MM[2] = 21
	println(AA.MM[1])
	println(AA.MM[10]) // 数组越界

	AA[1] = 21  // 可读，可写
	AAReadOnly[1] = 22 // 不可设置值，会报错
end
`

//define
type AS struct {
     MI *[3]int
     MM [4]int
}
    
//init
AS := &AS{
   	MI: &[3]int{},
   	MM: [4]int{},
  }
    
//define
var AA [2]int
AA = [2]int{1, 2}
    
dataContext := context.NewDataContext()
dataContext.Add("PrintName",fmt.Println)
dataContext.Add("AS", AS)
//single array inject, must be ptr
dataContext.Add("AA", &AA)
//single array inject, readonly
dataContext.Add("AAReadOnly", AA)
```
##### slice

```go
const rule = `
rule "name test" "slice" salience 10
begin
	// SS 是 注入的变量
	SS.MI[1] = 20
	SS.MM[2] = 21
	println(SS.MM[1])
	println(SS.MM[10]) // 数组越界

	S[1] = 21  // 可读，可写
	SReadOnly[1] = 22 // 不可设置值，会报错
end
`

//define
type SS struct {
     MI *[]int
     MM []int
}
    
//init
SS := &SS{
   	MI: &[]int{1,2,3,4},
   	MM: []int{4,2,6,7,9},
  }
    
//define
var S = []int{1,2}

    
dataContext := context.NewDataContext()
dataContext.Add("PrintName",fmt.Println)
dataContext.Add("SS", SS)
//single array inject, must be ptr
dataContext.Add("S", &S)
//single array inject, readonly
dataContext.Add("SReadOnly", S)
```

#### nil

不支持 nil，如果需要，可以使用内置函数 `isNil()` 代替

```go
var sliRules = `
rule "1"
begin

//基础类型int，不存在返回默认值
a = contextInt[1]
println("a---> ", a)
println("int isNil  a--->", isNil(a)) // 基础类型永远是false
println("array isNil  contextInt--->", isNil(contextInt)) // 基础类型永远是false

i = contextMap["2"]
println("i---> ", i)
println("ptr map isNil contextMap--->",isNil(contextMap))

end
```

- `isNil` 的参数只有一个, 且可以是任意类型的; 其返回值是 `true` 或 `false`
- 当你有不确定其返回值是否为 `nil`, 而导致规则不可正常执行时, 都可以使用 `isNil` 函数来判别一下
- 基础类型永远都不是 nil, 都是其对应的默认值
- 非基础类型可能为nil,具体测试详见如下
### 支持的运算

- 支持完整数值之间的加 `(+)`、减 `(-)`、乘 `(*)`、除 `(/)` 四则运算, 以及字符串之间的加法
- 完整的逻辑运算`(&&、 ||、 !)`
- 支持比较运算符: 等于 `(==)`、不等于 `(!=)`、大于 `(>)`、小于 `(<)`、大于等于 `(>=)`、小于等于 `(<=)`
- 支持`+=, -=, *=, /=`
- 支持小括号
- 优先级:括号, 非, 乘除, 加减, 逻辑运算`(&&,||)` 依次降低

### 变量

* 用户自定义变量无需申明类型
- **规则内定义的变量,只对当前规则可见,对其他规则不可见**(局部变量)
- **使用dataContext注入的(变量)数据,对加载到gengine中的所有规则均可见**(全局变量)

#### 变量声明

```go
const params_rule = `
rule "name test" "i can" salience 10
begin

// 声明变量
x = 100
z = 200
isOld = x > 10 + 20

// 使用变量
println(isOld)
println(!true)

end
`
```

#### 使用规则外变量

```go

func Test(user *User) {
	const rule1 = `
	rule "name test" "i can"  salience 0
	begin
		// User 是 注入的变量
		// println 是注入的函数
		println(User.Age)
		println(User.GetAge())
	end
	`
	dataContext := context.NewDataContext()
	//注入初始化的结构体
	dataContext.Add("User", user)
	//用户注入需要在规则代码中使用的API
	dataContext.Add("println", fmt.Println)

	//init rule engine
	ruleBuilder := builder.NewRuleBuilder(dataContext)

	//构建规则
	err := ruleBuilder.BuildRuleFromString(rule) //string(bs)
}

```

#### rule 特殊变量

* `@name`：- 在规则体中使用@name,指代的是当前规则名,@name在规则执行时,会被解析为规则名字符串(规则内的名字感知)
* `@id`：- 在规则体中使用@id含义是,如果当前的规则名是可以转化为整数的字符串,则@id就是规则名的整数值,如果规则名字符串不可转化为整数，则@id的值为0.这个是为了方便用户以规则名作为整型参数
* `@desc`：- 在规则体内获知当前规则的描述信息,类型为字符串
* `@sal`：- 在规则体内获知当前规则的优先级信息，类型为int64

### 控制流

#### if ... else

```go
const rule_else_if_test =`
rule "elseif_test" "test"
begin

a = 5
if a < 1 {
	println("a < 1")
} else if a >= 1 && a <6 {
    if a <= 3{
        println("a >= 1 && a<=3")
    }else{
        println("a > 3 && a <6")
    }
} else
	println("a >= 6")
}
end`
```

#### for

##### range number
```go
const forRule = `
rule "number array test" "m dec"
begin

// for statment
for i=1;i<=5;i=i+1{
	println("for i = ", i)
}

end
`
```
##### Range map
```go
const forRule = `
rule "map array test" "m dec"
begin

// forRange map
debugKey := "keylist"
forRange key := mapInfo{
	if key == "key1" {
		debugKey += "|" + key + ":" + mapInfo[key]
	}
}
println("debugKey is ",debugKey)
end
`
```
##### Range slice

```go
const forRule = `
rule "array test" "m dec"
begin

// forRange slice
forRange key := testSlice {
	println("slice = ", key, testSlice[key])
}

for i:=0; i< len(ss); i=i+1{
	ss[i].S // 这种写法会报错
	s = ss[i]
	s.S     //  需要先赋值给s，这样正常
}

end
`
type Request struct {
	S string
	I int
}

func getLen(rs []*Request)int{
	return len(rs)
}

//struct
ss := []*Request{
	Request{
		S: "a", //此处有值，和无值，最终判别nil是不一样的
	},
	Request{
		S: "b",
	},
}

dataContext := context.NewDataContext()
//结构体
dataContext.Add("ss", ss)
dataContext.Add("len", getLen)
```

### 函数

gengine 不支持直接在 rule 中写函数，在 rule 中使用的函数需要在规则外声明，然后注入到 rule 中，

```go
func Test(user *User) {
	const rule1 = `
	rule "name test" "i can"  salience 0
	begin
		// User 是 注入的变量
		// println 是注入的函数
		println("hello world")
	end
	`
	dataContext := context.NewDataContext()
	//用户注入需要在规则代码中使用的API
	dataContext.Add("println", fmt.Println)

	//init rule engine
	ruleBuilder := builder.NewRuleBuilder(dataContext)

	//构建规则
	err := ruleBuilder.BuildRuleFromString(rule) //string(bs)
}
```

为了尽量保证规则的书写和golang语法的无关性,**规则体内可以调用具有多返回的函数或方法,但当使用自定义变量接受函数、方法返回值时,只支持但返回一个返回数据的函数和方法调用**(也是间接建议用户为自己的业务实现设置适当的默认值,而不是直接报错)

#### 内置函数

- 为了处理 golang 语言的 nil, 简化用户使用, gengine 目前提供了一个唯一的内置函数 isNil ()
- isNil的参数只有一个,且可以是任意类型的;其返回值是true或false
- 当你有不确定其返回值是否为nil,而导致规则不可正常执行时,都可以使用isNil函数来判别一下

使用方法参考数据类型 `nil` 部分。

### 并发

#### 规则内并发

- 在很多业务场景下, 一个规则, 需要很多的数据指标支持, 因为数据量的缘故, 这些数据指标往往不可能存储在本地.
- 因此,如果在规则内顺序地从网络上取很多个指标时,会严重拖慢规则的执行性能.
- 基于此考量,**gengine 支持规则内的并发函数调用或赋值语句**. 其语法相当简单, 使用 conc{}语句块来包裹需要并发调用的接口即可.

```go
const rule_conc_statement  = `
rule "conc_test" "test" 
begin
	conc  { // this should be used in time-consuming operation, such as the operation contains network connection (get data from remote based on network) 
		println("hihihi")
		a = 3.0
		b = 4
		c = 6.8
		d = "hello world"
        e = "you will be happy here!"
		sout("heheheheh")
	}
	println(a, b, c, d, e)
end`

//define
func Sout(str string)  {
	println("----",str)
}

//inject
dataContext.Add("println",fmt.Println)
dataContext.Add("sout", Sout)
```

- conc{}语句块内支持放置方法、函数、赋值语句
- conc{}语句块内可以为空,或有一个或多个方法、函数或赋值语句
- conc{}语句块为空时,不产生任何影响,当仅有一条语句时,退化为顺序执行,当多于2条或2条以上的语句时,会进行并发执行
- 用户需要自己来保证在conc{}中的调用的api是线程安全的
- 多执行几次测试,你会发现conc{}内的输出顺序每次都不一样

### pool

## 实践

BuildRuleFromString 是一个耗时操作，不建议每次都创建一个新的 gengine。

```go
var GengineMaps = make(map[string]*ReconRuleGengine, 0)

func GetGengine(ctx lhscontext.BasicContext, ruleName, ruleStr string) (*ReconRuleGengine, error) {
	if g, ok := GengineMaps[ruleName]; ok {
		return g, nil
	}
	//初始化 注入api，请确保注入的API属于并发安全
	apis := make(map[string]interface{})
	apis["println"] = fmt.Println
	apis["Sprintf"] = fmt.Sprintf
	g, err := NewRuleGengine(10, 20, 1, ruleName, apis)
	if err != nil {
		fmt.Printf("new rule Gengine fail | lErr: %s", err)
		return nil, err
	}
	GengineMaps[ruleName] = g
	return g, nil
}

// 业务规则
const service_rules string = `
rule "1" "1"
begin
	println("rule 1...", room.GetAttention())
end 

rule "2" "2"
begin
	println("rule 2...", room.GetNum())
end
`

// 特定的场景服务
type Room struct {
}

func (r *Room) GetAttention( /*params*/ ) int64 {
	// logic
	return 100
}

func (r *Room) GetNum( /*params*/ ) int64 {
	//logic
	return 111
}

// 业务接口
type ReconRuleGengine struct {
	//gengine pool
	Pool *engine.GenginePool
}

// NewRuleGengine 初始化 gengine apiOuter这里最好仅注入一些无状态函数，方便应用中的状态管理
func NewRuleGengine(poolMinLen, poolMaxLen int64, em int, rulesStr string, apiOuter map[string]interface{}) (*ReconRuleGengine, error) {
	pool, err := engine.NewGenginePool(poolMinLen, poolMaxLen, em, rulesStr, apiOuter)
	if err != nil {
		return nil, err
	}

	ge := &ReconRuleGengine{Pool: pool}
	return ge, nil
}

// Execute
func (ms *ReconRuleGengine) Execute(ruleName string, room *Room) (map[string]interface{}, error) {
	//基于需要注入接口或数据,data这里最好仅注入与本次请求相关的结构体或数据，便于状态管理
	data := make(map[string]interface{})
	data["Base"] = baseData
	data["Input"] = dto

	err, resultMap := ms.Pool.ExecuteSelectedRules(data, []string{ruleName})
	if err != nil {
		println(fmt.Sprintf("pool execute rules error: %+v", err))
		return nil, err
	}

	return resultMap, nil
}

```

## 相关链接 ：

1. [gengine简介](https://github.com/bilibili/gengine/wiki/gengine%E7%AE%80%E4%BB%8B)
2. [B 站新一代 golang 规则引擎的设计与实现](https://xie.infoq.cn/article/40bfff1fbca1867991a1453ac)
