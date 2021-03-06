# SVA 简单语法总结

## 第一章 SVA 介绍

**断言是设计属性的描述**

断言又被称为“监视器”“检验器”

作为一种调试技术手段，用在设计和验证中

**为什么使用 SVA**

1. verilog 是一种过程语言
2. verilog 是一种冗长的语言
3. 由于 verilog 的过程性，因此在一定情况下，verilog 的检验器甚至可能无法捕捉到所有被触发事件
4. verilog 没有提供内嵌的的机制来提供功能覆盖率数据

**采用 SVA 的原因**

1. sva 是一种描述性语言，可以很好地描述时序
2. 提供了内嵌函数来测试特定的情况，
3. 提供了一些构造来自动收集功能覆盖率数据

**sva 检验失败时会自动打印相关信息**

**systemverilog 是基于事件的执行模式，在每个 time slot，许多事件按照安排的顺序发生。这个事件的列表按照安排的顺序发生**

---

### 断言的评估和执行

1. 预备（preponed) : 采样断言变量，而且信号和变量的值不能改变，这样确保在 time slot 开始时采样到稳定的值
2. 观察（obversed）： 对所有的属性表达式求值
3. 响应（reactive）：调度评估属性成功或失败的代码

---

> 关于 time slot 以及事件调度详细请参看《systemverilog 3.1a LRM》

**_断言分类_**

1. 并发断言
   `a_cc: assert property(@(posedge clk) not (a && b));`
   1. 基于时钟周期
   2. 在时钟边沿根据调用的变量的采样值计算测试表达式
   3. **变量的采样在预备阶段完成，表达式的计算在调度器的观察阶段完成**
   4. 可以在 procedural block、module 、interface、program 中出现
   5. 可以在静态和动态验证中出现
2. 及时断言

   1. 基于模拟事件的语义
   2. 测试表达式的求值就像在过程块中的其他 verilog 表达式一样**本质不是时序相关，而是立即求值**
   3. 必须出现在过程块中
   4. 只能用于动态模拟

   > `always_comb begin a_ia : assert ( a&&b ); end `

**区分即时断言和并发断言的关键词是 property**

**SVA 使用 sequence 来表示事件**

```systemverilog
sequence name_of_sequence ;
    <test expression>;
endsequence
```

**SVA 使用 property 来表示更加复杂的有序行为**

```systemverilog
property name_of_property ;
    <test expression>;
endproperty
```

**SVA 使用 assert 来检查属性**

```systemverilog
 assertion_name : assert property(property_name);
```

### 边缘表达式

**SVA 内嵌了边缘表达式，以便用户监视信号从一个时钟周期到另一个时钟周期的跳变**这使得用户可以检验边沿敏感信号

1. `$rose(boolean expression or signal name );`
2. `$fell(boolean expression or signal name );`
3. `$stable(boolean expression or signal name );`

### 逻辑关系的序列

### 序列表达式

**在序列定义中定义形参，相同的序列可以被重用到设计中具有相似行为的信号上**

```
    sequence s3_lib(a,b);
        a || b ;
    endsequence
```

_一些在设计中常见的通常的属性可以被开发成一个库以便于重用。比如，one-hot 状态机检查，等效性检查等都适合放在这样的检验器库中_

### 时序关系的序列

有时间，我们关心的是检查需要几个时钟周期才能完成的事件，也就是所谓的时序检查  
在 SVA 中，时钟周期延迟用##来表示，如##3 表示 3 个时钟周期

### SVA 中的时钟定义

一个 sequence 或者 property 在模拟中并不起什么作用，它们必须别 assert 才能发挥作用
在 sequence、property、assert 中都可以定义时钟

建议：**在 property 中定义时钟，保持 sequence 独立与时钟**是一种良好的编码风格
断言一个序列并不一定需要定义一个独立的属性。因为断言语句调用属性，在断言的语句中可以直接调用被检查的表达式
_注意时钟定义重复的情况_

### 禁止属性

使用关键词**not**

### action block 执行块

可以使用断言陈述的之执行块（action block）来打印自定义的成功或失败信息.

```systemverilog
assertion_name :
    assert property(property_name)
        <success message>;
    else
        <fail message>;
```

**可以在 action block 中实现控制模拟环境收集功能覆盖率数据等功能**

### 蕴含操作符(implication)

被定义为：当检查的起始点不是有效时，忽略这次检查
蕴含等效于一个 if-then 结构。蕴含的左边叫作**先行算子(antecedent)**，右边叫作**后续算子(consequent)**。先行算子是约束条件。当先行算子成功时，后续算子才会被计算。如果先行算子不成功，那么整个属性就默认地被认为成功。这叫作**空成功 (vacuous success)**。**蕴含结构只能被用在属性定义中，不能在序列中使用**

1. 交叠蕴含（overlapped implication)
   - 交叠蕴含用符号|->表示。如果先行算子匹配，在**同一个时钟周期**计算后续算子表达式
2. 非交叠蕴含(Non-overlapped implication)
   - 非交叠蕴含用符号|=>表示。如果先行算子匹配，那么在**下一个时钟周期**计算后续算子表达式。后续算子表达式的计算总是有一个时钟周期的延迟

#### 后续算子带固定延迟的蕴含

```systemverilog
property p10;
    @(posedge clk) a |-> ##2 b;
endproperty
```

**先行算子可以使用序列的定义**也就是对先行算子的限制不是特别严格

### SVA 检验器的时序窗口

SVA 允许使用时序窗口来匹配后续算子（时序窗口表达式左手边的值必须小于右手边的值,左手边的值可以是 0。如果它是 0，表示后续算子必须在先行算子成功的那个时钟边沿开始计算。

```systemverilog
property p12;
    @(posedge clk) (a && b) |-> ##[1:3] c;
endproperty
a12 ： assert property(p12);
```

每声明一个时序窗口，就会在每个时钟沿上触发多个线程来检查所有可能的成功。
在任何一个时钟上升沿只能有一个有效地开始，但是可以有很多有效的结束

### 重叠的时序窗口

先行算子和后续算子可以从同一个时钟边沿开始

### 无限的时序窗口

在时序窗口的窗口上限可以用符号"$"定义，这表明时序没有上限。这叫作**可能性(eventuality)**运算符。检验器不停地检查表达式是否成功直到模拟结束

会对性能产生巨大的影响，**不推荐使用**
如果有一个有效地开始，但是后续算子在模拟结束前依然无效，则这些检查被报告为**未完成检验（incomplete check）**

### ended 结构

将多个序列以序列的起始点作为同步点，来组合成时间上连续的检查。SVA 提供了另一种使用**序列的结束点**作为同步点的连接机制,这种机制通过给序列名字追加关键字 ended 来表示
关键词“ended”保存了一个布尔值，值的真假取决于序列是否在特定的时钟边沿匹配检验。这个 s.ended 的布尔值只有在相同时钟周期有效。

### 使用参数的 SVA 检验器

可以在 SVA 检验器中使用参数

### 使用选择运算符的 SVA 检验器

`? : `

---

- [ ] **使用 true 表达式的 SVA 检验器**
      可以在时间上延长 SVA 检验器

### $past 构造

可以得到信号在几个时钟周期之前的值。在默认情况下，它提供信号在前一个时钟周期的值。

```systemverilog
$past(signal_name,number of clock cycles);
# instance
property p19;
@(posedge clk) ( c&&d ) |->
    ($past(( a&&b ),2) == 1'b1);
endproperty
a19: assert property(p1);
```

#### 带门控时钟的$past 构造

```systemverilog
$past(signal_name,number of clock cycles,gating signal);
```

只有当门控时钟信号的值为真时才检查后续算子的情况

## 重复算子

1. 连续重复（consecutive repetition）
   允许用户表明信号或者序列将在指定数量的时钟周期内都连续地匹配,**信号的每次匹配之间都有一个时钟周期的隐藏延迟**
   ```systemverilog
   signal or sequence [*n];
   //a[*3] 等价于 a ##1 a ##1 a
   ```
   可用于序列、延迟窗口、可以和可能性运算符结合
2. 跟随重复（go to repetition）
   允许用户表明一个表达式将匹配达到指定的次数，而且不一定在连续的时钟周期上发生,这些匹配可以是间歇性的，而且不一定在连续的时钟周期上发生，跟随重复的主要要求是被检验重复的表达式的最后一个匹配应该发生在整个序列匹配结束之前
   ```systemverilog
   signal [->n]
   start ##1 a[->3] ##1 stop ;
   /*
       这个序列需要信号“a”的匹配(即信号“a”的第三次，也就是最后一次重复的匹配)正好发生在“stop”成功之前。换句话说，信号“stop”在序列的最后一个时钟周期匹配，而且在前一个时钟周期，信号“a”有一次匹配。
   */
   ```
3. 非连续重复（non-consecutive repetition）
   与跟随重复相似，除了它并不要求信号的最后一次重复匹配发生在整个序列匹配前的那个时钟周期
   ```systemverilog
   signal or sequence [=n];
   ```

---

**在跟随重复和非连续重复中只允许使用表达式，不能使用序列**

---

### "and"构造

and 可以逻辑地组合两个序列。两个序列必须具有相同的起始点，但是可以具有不同的结束点。检验的起始点是第一个序列成功时的起始点，而检验的结束点是使得属性最终成功的另一序列成功时的点

### "intersect"构造

两个序列必须在相同时刻开始且在相同时刻结束,即两个序列的长度必须相等

### "or"构造

### "first_match"构造

> 任何时候使用了逻辑运算符(如“and”和“or”)的序列中指定了时间窗，就有可能出现同一个检验具有多个匹配的情况。“first_match”构造可以确保只用第一次序列匹配，而丢弃其他的匹配

### "throughout"构造

> implication 是允许定义前提条件的一项技术,含**只在时钟边沿检验前提条件一次**，然后就开始检验后续算子部分，因此它**不检测先行算子是否一直保持为真**。

```systemverilog
    (expression) throughout (sequence definition)
```

**在整个检查过程中，expression 应该保持为真**

### "within"构造

> within 构造允许在一个序列中定义另一个序列
> `seq1 within seq2 `
> 这表示 seq1 在 seq2 的开始到结束的范围内发生，且序列 seq2 的开始匹配点必须在 seq1 的开始匹配点之前发生，序列 seq1 的结束匹配点必须在 seq2 的结束匹配点之前结束

## 内建的系统函数

1. $onehot(expression) 换句话说，就是在任意给定的时钟沿，表达式只有一位为高。
2. $onehot0(expression) 检验表达式满足“zero one-hot”，换句话说，就是在任意给定的时钟沿，表达式只有一位为高或者没有任何位为高
3. $isunknown(expression)—— 检验表达式的任何位是否是 X 或者 Z。
4. $countones(expression)—— 计算向量中为高的位的数量

## disable iff 构造

在某些设计情况中，如果一些条件为真，则我们不想执行检验。换句话说，这就像是一个异步的复位，使得检验在当前时刻不工作。SVA 提供了关键词“disable iff”来实现这种检验器的异步复位

```systemverilog
disable iff (expression) < property definition>
```

当 expression 为真时，我们不想检验 property definition

### 使用 intersect 控制序列的长度

```systemverilog
property p35;
(@(posedge clk) 1[*2:5] intersect
                (a ##[1:$] b ##[1:$] c));
endproperty

a35： assert property(p35);
```

这可以使用带 1[*2:5]的 intersect 运算符来加以约束。这个 intersect 的定义检查从序列的有效开始点(信号“a”为高)，到序列成功的结束点(信号“c”为高)，一共经过 2~5 个时钟周期

### 在属性中使用形参

可以用定义形参的方式来重用一些常用的属性
SVA 允许使用属性的形参来定义时钟。这样，属性可以应用在使用不同时钟的相似设计模块中

```systemverilog
property arb (a，b，c，d);
    @(posedge clk)
    ($fell(a) ##[2:5] $fell(b)) |->
    ##1 ($fell(c) && $fell(d)) ##0
    (!c&&!d) [*4] ##1 (c&&d) ##1 b;
endproperty
```

### 嵌套的蕴含

```systemverilog
`define free (a && b && c && d)
property p_nest;
    @(posedge clk) $fell(a) |->
    ##1 (!b && !c && !d) |->
    ##[6：10]
    `free;
endproperty

a_nest： assert property(p_nest);
```

### 在蕴含中使用 if/else

SVA 允许在使用蕴含的属性的后续算子中使用“if/else”语句。

```systemverilog
property p_if_else;
@(posedge clk)
    ($fell(start) ##1 (a||b)) |->
    if(a)
        (c[->2] ##1 e)
    else
          (d[->2] ##1 f);
endproperty

a_if_else： assert property(p_if_else);
```

### SVA 中的多时钟定义

SVA 允许序列或者属性使用多个时钟定义来采样独立的信号或者子序列。SVA 会自动地同步不同信号或子序列使用的时钟域。  
**当在一个序列中使用了多个时钟信号时，只允许使用“##1”延迟构造**

**使用“##0”会产生混淆，即在信号“a”匹配后究竟哪个时钟信号才是最近的时钟。这将引起竞争，因此不允许使用。使用##2 也不允许，因为不可能同步到时钟“clk2”的最近的上升沿。**

**禁止在两个不同时钟驱动的序列之间使用交叠蕴含运算符。因为先行算子的结束和后续算子的开始重叠，可能引起竞争的情况，这是非法的。**

可以使用非交叠蕴含(|=>)

### matched 构造

任何时候如果一个序列定义了多个时钟，构造“matched”可以用来监测第一个子序列的结束点。

```systemverilog
sequence s_a;
    @(posedge clk1) $rose(a);
endsequence
sequence s_b;
    @(posedge clk2) $rose(b);
endsequence
property p_match;
    @(posedge clk2) s_a.matched |=> s_b;
endproperty

a_match： assert property(p_match);
```

理解“matched”构造的使用方法的关键在于，**被采样到的匹配的值一直被保存到另一个序列最近的下一个时钟边沿**

### expect 构造

与 verilog 中的等待构造相似，关键的区别在于 expect 语句等待的是属性的成功检验，expect 构造后面的语句是作为一个阻塞语句来执行的。

```systemverilog
initial begin
    @(posedge clk);
    #2ns cpu_ready = 1'b1 ;
    expect ( (@posedge clk) ##[1:16 ] memory_ready == 1'b1 )
    $display("Hand shake successful\n");
    else begin
    $display("Hand shake failed： exiting\n")
    $finish();
    end

    for(i=0; i<64; i++) begin
        send_packet();
        $display("PACKET %0d sent\n"，i);
    end
end
```

### 使用局部变量的 SVA

在序列或者属性中可以定义局部变量，而且可以对这种变量赋值

```systemverilog
property p_local_var1 ;
int var1 ;
@(posedge clk)
( $rose(enable1),var1 = a ) |->
##4 （ aa == (var1*var1*var1) ;
endproperty
```

可以在 SVA 中保存和操作局部变量

### 在序列匹配时调用子程序

SVA 可以在序列每次成功匹配时调用子程序。同一序列中定义的局部变量可以作为参数传给这些子程序。对于序列的每次匹配，子程序调用的执行与它们在序列定义中的顺序相同

- [ ] 这里说的子程序是什么意义？ (在书中可以看到这里的子程序可以是 sequence)

### 将 SVA 与设计链接

1. 在 module 定义中内建或者内联检验器( SVA 代码可以内建在 module 定义的任何地方)
2. 将检验器与 module、module instance 等绑定
   如果用户决定将 SVA 检验器与设计代码分离，那么就需要建立一个独立的检验器模块。定义独立的检验器模块，增强了检验器的可重用性

```systemverilog
module mutex_chk(a,b,clk);
input logic a,b,clk;
property p_muxtex;
    @(posedge clk) not (a && b);
endproperty

a_muxtex: assert property(p_muxtex);
endmodule
```

**定义检验器模块时，它是一个独立的实体。检验器用来检验一组通用的信号，检验器可以与设计中任何的 module 或者 instance 绑定**

```systemverilog
bind <module_name or instance_name>
    <checker name ><checker instance name >
    <design signals> ;
// eg
bind inline muxtex_chk i2 (a,b,clk);
```

- [ ] 与 module bind 相比 instrance bind 有什么区别？

与检验器绑定的设计信号可以包含绑定实例中的任何信号的跨模块引用(cross module reference)

### SVA 与功能覆盖率

功能覆盖是按照设计规范来衡量验证状态的一个标准，可以分为两类：

1. 协议覆盖
2. 测试计划覆盖

断言可以用来获得有关协议覆盖的穷举信息。sva 提供了关键词**cover**来实现这个功能

```systemverilog
 <cover_name> : cover property(property_name);
```

cover 语句的结果包含下面的信息：

1. 属性被尝试检验的次数。
2. 属性成功的次数。
3. 属性失败的次数。
4. 属性空成功的次数。

就像断言(assert)语句一样，覆盖(cover)语句可以有执行块。在一个覆盖成功匹配时，可以调用一个函数(function)或者任务(task)，或者更新一个局部变量
