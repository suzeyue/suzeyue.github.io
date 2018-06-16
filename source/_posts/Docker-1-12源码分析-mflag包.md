---
title: Docker 1.12源码分析 -- mflag包
date: 2017-08-10 15:11:37
tags: 
  - docker
  - golang
---


mflag包实现了命令行中flag的解析。
代码在docker/pkg/mflag/flag.go中。

### 用法
- 第一步：注册flag。
flag的注册主要用到Bool（）、BoolVar（）、Int（）、IntVar（）、String（）、StringVar（）等方法。
这些方法需要对应的数据类型（实现Value接口）：boolValue、intValue、stringValue。。。这些类型代码中以定义好。
同样的我们可以自定义数据类型进行注册解析，此类型必须满足Value接口，然后调用`Var(value Value, names []string, usage string)`方法注册（第一个参数为Value）。

- 第二步：解析flag。
在所有flag注册完毕之后，调用`Parse(arguments []string)`将arguments解析到定义好的flag里。

### 主要的数据结构

#### Value：
Value接口是用于将动态值保存到一个flag里
```
// 如果一个Value接口有IsBoolFlag（）方法且返回值为true，
// 那么命令行解析就会把-name解析为-name=true，而不是使用下一个命令行参数
type Value interface {
   String() string
   Set(string) error
}
```

#### Getter：
```
// Get（）可以获取Value接口中的内容，这里Get（）为什么不是Value接口的一部分，
// 是因为兼容性的问题（Getter接口是在Go1之后出现）。
type Getter interface {
   Value
   Get() interface{}
}
```

#### Flag：
代表一个flag状态
```
type Flag struct {
   Names    []string // flag在命令行的名字（这里是slice，可能有多个名字）
   Usage    string   // flag的帮助信息
   Value    Value    // 需要设置的值（这里Value就是上方的interface）
   DefValue string   // 默认值（Value.String()）
}
```

#### FlagSet：
FlagSet代表一个已注册的flag的集合。FlagSet零值没有名字，采用ContinueOnError错误处理策略
```
ype FlagSet struct {
   // Usage函数在解析flag出现错误时会被调用
   // 这里为什么会是一个函数而不是使用方法，是为了能够自定义错误处理函数
   Usage      func()
   ShortUsage func()

   name             string
   parsed           bool
   actual           map[string]*Flag
   formal           map[string]*Flag
   args             []string // flags之后的参数
   errorHandling    ErrorHandling
   output           io.Writer // nil表示stderr
   nArgRequirements []nArgRequirement
}
```
注意actual和formal，map的value都是*Flag，因为有可能一个flag有很多name（比如-h、--help），这样这些name都会指向同一个Flag，那么只要Parse（）任意一个name，其它的name都会改变，因为它们是同一个Flag。

另外，在本包中定义了很多数据类型：boolValue、intVlaue、int64Value、... 、stringValue、durationValue。这些类型都有String（） string、Set（string） error、Get（） interface｛｝方法，所以他们也就实现了上面的Value接口和Getter接口。
其中，我们需要注意的是boolValue，它有一个方法IsBoolFlag（） bool，所以boolValue同样实现了boolFlag接口，而这个接口在命令行解析时会将-name解析成-name=true，而不需要下一个命令行参数。

### 主要的方法：
FlagSet提供了很多flag的注册方法，分为两种：
1. Bool（），Int（），String（）等会返回一个对应类型的指针，flag的值会存储到指针指向的变量里；
2. BoolVar（），IntVar（），StringVar（）等方法比前一种多一种参数，对应类型的指针p，flag的值保存在p所指向的变量中，这一类没有返回值。

其实Xxx（）方法的实现还是调用了XxxVar（）方法，只不过首先new一个对应类型的指针p，用来当作XxxVar（）的第一个参数。
而这些所有的实现最终会调用Var（）方法。
```
// Var（）方法根据的name和usage定义一个flag，而flag的类型和值根据第一个参数获得
func (fs *FlagSet) Var(value Value, names []string, usage string) {
   // 默认值始终是一个string，不会改变
   flag := &Flag{names, usage, value, value.String()}
   for _, name := range names {
      name = strings.TrimPrefix(name, "#")
      _, alreadythere := fs.formal[name]
      if alreadythere {
         var msg string
         if fs.name == "" {
            msg = fmt.Sprintf("flag redefined: %s", name)
         } else {
            msg = fmt.Sprintf("%s flag redefined: %s", fs.name, name)
         }
         fmt.Fprintln(fs.Out(), msg)
         panic(msg) // Happens only if flags are declared with identical names
      }
      if fs.formal == nil {
         fs.formal = make(map[string]*Flag)
      }
      fs.formal[name] = flag
   }
}
```
首先根据参数初始化了一个Flag并返回了对应的指针flag，flag.DefValue其实是value.String（）。
然后遍历names获取每一个name。将name开始的“#”去除，看*FlagSet fs的formal中是否存在formal[name]，如果存在则引发panic，如果不存在就将此flag添加到fs.formal中。
也就是说，同一个flag如果有多个名字，那么这个flag会在fs.formal中存多次，而key就是names中的不同name，value则是这个flag的指针。

注册好之后就是解析，解析的主要方法是Parse（arguments []string），在方法里调用了parseOne（），我们先来看这个方法。
```
func (fs *FlagSet) parseOne() (bool, string, error) {
   if len(fs.args) == 0 {
      return false, "", nil
   }
   s := fs.args[0]
   if len(s) == 0 || s[0] != '-' || len(s) == 1 {
      return false, "", nil
   }
   if s[1] == '-' && len(s) == 2 { // "--" terminates the flags
      fs.args = fs.args[1:]
      return false, "", nil
   }
   name := s[1:]
   if len(name) == 0 || name[0] == '=' {
      return false, "", fs.failf("bad flag syntax: %s", s)
   }

   // it's a flag. does it have an argument?
   fs.args = fs.args[1:]
   hasValue := false
   value := ""
   if i := strings.Index(name, "="); i != -1 {
      value = trimQuotes(name[i+1:])
      hasValue = true
      name = name[:i]
   }

   m := fs.formal
   flag, alreadythere := m[name] // BUG
   if !alreadythere {
      if name == "-help" || name == "help" || name == "h" { // special case for nice help message.
         fs.usage()
         return false, "", ErrHelp
      }
      if len(name) > 0 && name[0] == '-' {
         return false, "", fs.failf("flag provided but not defined: -%s", name)
      }
      return false, name, ErrRetry
   }
   if fv, ok := flag.Value.(boolFlag); ok && fv.IsBoolFlag() { // special case: doesn't need an arg
      if hasValue {
         if err := fv.Set(value); err != nil {
            return false, "", fs.failf("invalid boolean value %q for  -%s: %v", value, name, err)
         }
      } else {
         fv.Set("true")
      }
   } else {
      // It must have a value, which might be the next argument.
      if !hasValue && len(fs.args) > 0 {
         // value is the next arg
         hasValue = true
         value, fs.args = fs.args[0], fs.args[1:]
      }
      if !hasValue {
         return false, "", fs.failf("flag needs an argument: -%s", name)
      }
      if err := flag.Value.Set(value); err != nil {
         return false, "", fs.failf("invalid value %q for flag -%s: %v", value, name, err)
      }
   }
   if fs.actual == nil {
      fs.actual = make(map[string]*Flag)
   }
   fs.actual[name] = flag
   for i, n := range flag.Names {
      if n == fmt.Sprintf("#%s", name) {
         replacement := ""
         for j := i; j < len(flag.Names); j++ {
            if flag.Names[j][0] != '#' {
               replacement = flag.Names[j]
               break
            }
         }
         if replacement != "" {
            fmt.Fprintf(fs.Out(), "Warning: '-%s' is deprecated, it will be replaced by '-%s' soon. See usage.\n", name, replacement)
         } else {
            fmt.Fprintf(fs.Out(), "Warning: '-%s' is deprecated, it will be removed soon. See usage.\n", name)
         }
      }
   }
   return true, "", nil
}
```
首先是对异常情况的处理。
然后获取fs.args[0]赋值给s。并且这时fs.args = fs.args[1:]，也就是说每解析一次，fs.args都会去除第一个arg。
对s进行解析。因为命令行flag会有三种情况：
- -flag
- -flag=x
- -flag x

那么。如果存在“=”，左边为name，右边为value；如果不存在，那么s直接就是name（name需要去除第一个'-'，value去除“''”等）
```
m := fs.formal
flag, alreadythere := m[name]
```
根据name获取对应的flag。如果没有获取到则返回错误（没定义却使用help时是ErrHelp）。
- 如果flag.Value是boolFlag类型且IsBoolFlag()为真，那么代表这个flag是一个特例，可以不需要value，也就是上面第一种flag情况,这时就将flag的值设为true；
- 如果value存在，那么直接用value赋值，这也是上面第二种flag情况。
- 如果flag.Value不是boolFlag类型，那么就是上面第三种flag情况，而flag的值是下一个参数（如果没有下一个参数则返回错误）。
```
hasValue = true
value, fs.args = fs.args[0], fs.args[1:] //这里需要注意的是fs.args又去除了第一个元素。
```
设置flag.Value的值为value。
将赋值好的flag添加到FlagSet的actual中。
```
fs.actual[name] = flag
```
最后是对“#”开头的name处理，“#”代表弃用。

现在再来看看Parse（arguments []string） error方法。
```
// 从参数arguments中解析注册的flag。必须在所有flag注册之后，访问flag值之前进行。
// -help未注册而使用时，会返回ErrHelp。
func (fs *FlagSet) Parse(arguments []string) error {
   fs.parsed = true
   fs.args = arguments
   for {
      seen, name, err := fs.parseOne()
      if seen {
         continue
      }
      if err == nil {
         break
      }
      if err == ErrRetry {
         if len(name) > 1 {
            err = nil
            for _, letter := range strings.Split(name, "") {
               fs.args = append([]string{"-" + letter}, fs.args...)
               seen2, _, err2 := fs.parseOne()
               if seen2 {
                  continue
               }
               if err2 != nil {
                  err = fs.failf("flag provided but not defined: -%s", name)
                  break
               }
            }
            if err == nil {
               continue
            }
         } else {
            err = fs.failf("flag provided but not defined: -%s", name)
         }
      }
      switch fs.errorHandling {
      case ContinueOnError:
         return err
      case ExitOnError:
         os.Exit(125)
      case PanicOnError:
         panic(err)
      }
   }
   return nil
}
```
首先为FlagSet的parsed赋值为true，代表此FlagSet已被解析过。
然后为FlagSet得args赋值为参数arguments，用来进行接下来的解析（调用parseOne()）。
接下来主要对当error为ErrTry是进行处理。为什么要进行这一步，这主要是因为有可能连写了一些flag（比如：-al，这里将-a和-l进行了连写），需要将连写的flag拆开逐个解析。
```
for _, letter := range strings.Split(name, "") {
   fs.args = append([]string{"-" + letter}, fs.args...)
   seen2, _, err2 := fs.parseOne()
   ......
}
```
将name以字母一个个分割，并进行遍历。
在字母前添加"-"，并将fs.agrs slice与之合并，形成新的fs.args。
调用fs.parseOne()进行解析。
这样flag的解析就完成了。

还有一个比较重要的函数是Merge(dest *FlagSet, flagsets ...*FlagSet) error，作用是将n个FlagSet合并成一个dest FlagSet。如果在合并过程中出现name重复的话，就按照dest的errorHanding处理。

### 主要的变量：
#### CommandLine：
```
// CommandLine是默认命令行flag集，用于解析os.Args。包水平的函数·如BoolVar等都是对其方法的封装。
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)  //FlagSet.name,  FlagSet.errorHanding
这里主要看一下Parse(),可以看到它解析的是os.Args[1:]
func Parse() {
   // Ignore errors; CommandLine is set for ExitOnError.
   CommandLine.Parse(os.Args[1:])
}
```

代码中的其它部分这里就不做分析。
