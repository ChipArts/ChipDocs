# SystemVerilog Coding Style

## 〇、前言

-   第一部分：**基础语法**

    该部分主要介绍了一些代码书写规定和基础的电路描述方法。该部分除常见的语法外，还提供了一些在新版SystemVerilogy语言中支持的新语法，文档中列出的语法均经过最新的仿真和综合工具测试。

-   第二部分：**高级语法**

    该部分为进阶语法介绍，主要介绍SystemVerilog中引入用于封装和做通用设计的相关语法。该部分内容建议有一定基础的设计人员采用。由于EDA工具支持不够完善，虽然我们在最新工具下测试了文档中介绍的相关语法，但是不排除目前该部分语法可能存在隐藏的Bug。采用该部分推荐的语法可以极大简化电路设计，尤其是顶层互联的相关代码。同时合理的利用相关语法可以实现一定程度的基础电路抽象，极大提高电路可读性。

-   第三部分：**特殊用法**

    该部分介绍为了简化代码、进行快速设计采用的相关语法。使用方法比较激进，建议酌情采用。该部分主要介绍了两种特殊的设计方法：基于宏定义的模板例化方法 和 基于宏电路库的设计方法。详细介绍了这两种方法的设计动机、设计原理和设计规范。

-   第四部分：**Bug**

    该部分介绍了目前已知的高级语法在不同EDA工具中的Bug。也列出了希望EDA工具和SystemVerilog未来能够支持的特性。

-   第五部分：**附录**

## 一、基础语法

### 1 命名规范

#### 1.1 文件命名

1.  每个文件中只包含一个module、class、package，文件名与文件内内容名称相同。

    ```systemverilog
    module TestModule;
    ...
    endmodule : TestModule
    ```

​	上述代码保存在 **TestModule.sv** 文件中。

2.  头文件根据需要命名，以svh作为后缀。例如：**StructDef.svh** 。

3.  电路库文件以 **库名 + .sv** 命名。 例如：**CircuitLib.sv** 文件中包含以下内容：

    `````systemverilog
    module CircuitLib_ModuleAaa
    ...
    module CircuitLib_ModuleBbb
    ...
    module CircuitLib_ModuleCcc
    ...
    `````

#### 1.2 module、class、package、function、task命名

1.  以 **大驼峰** 格式命名，package以Pkg结尾。

    ```systemverilog
    //AaBbCc.sv
    module AaBbCc;
      ...
    endmodule : AaBbCc
    
    class ClassDemo;
      ...
    endclass
    
    //XxxYyyPkg.sv
    package XxxYyyPkg;
      ...
    endpackage : XxxYyyPkg
    
    function logic ... FuncDemo(input ...)
      ...
    endfunction
    
    task TaskDemo (input ...);
      ...
    endtask
    ```

2.  若module属于某个电路库，module命名格式：**电路库名称 + ‘_’ + module名称**。

    `````systemverilog
    // CircuitLib.sv
    module CircuitLib_Max
      ...
    endmodule : CircuitLib_Max
    `````

3.  module例化使用前缀（**’U_’** 或 **‘Ux_’**）+ **ModuleName**

    ```systemverilog
    Adder U_Adder(...);
    Adder U1_Adder(...);
    Adder U2_Adder(...);
    ```

#### 1.3 信号命名

##### 1.3.1 命名风格

-   所有信号采用 **下划线命名法** 命名。

    ```systemverilog
    logic dat;
    wire  mem_w_data;
    ```

##### 1.3.2 后缀

```
module DemoModule
(
  input  data1_i,
  inout  data2_b,
  output data3_o
);

  logic dat0,dat_temp;
  logic dat_r,dat_l;

  ...

endmodule : DemoModule
```

1.  后缀用于标志信号的特殊用途或特殊含义，以一个小写字母写在信号名后。
2.  端口信号：

    -   input: **i** (input port)， 例如：**xxx_yyy_i**

    -   inout: **b** (bi-directional port)， 例如：**xxx_yyy_b**

    -   output: **o** (output port)， 例如：**xxx_yyy_o**

3.  寄存器：**r** (register) ，例如：**xxx_yyy_r** 。

4.  锁存器：**l** (latch) ，例如：**xxx_yyy_l** 。

5.  低有效：**n** (negative)，例如：**xxx_yyy_n** 。

6.  异步信号：**a** (asynchronous)，例如：**xxx_yyy_a** 。

7.  后缀顺序：i/b/o > a > n > r，端口信号可以只写方向，不写其他后缀。

##### 1.3.3 前缀

1.  前缀用于在不改变信号名情况下表示信号属性变化，命名方式：**前缀 + ‘_’ + 信号名** 。所有后缀都可以用于前缀。例如：

    ```systemverilog
    assign dat = ...;
    always_ff (posedge clk) begin
      r_dat <= dat;
    end
    ```

​	在上面的示例代码中，r_dat表示保存dat的寄存器的输出信号。使用 ‘_’ 后的名称可以在代码中直接找到该信号。

2.   寄存器下一个时钟周期的值：**f** (following)，例如：

     ```systemverilog
     logic [7:0] w_data_r,f_w_data_r;
     assign f_w_data_r = ... ;
     always_ff (posedge clk) begin
       w_data_r <= f_w_data_r;
     end
     ```

3.   信号取反：**i** (invert)，例如：**assign i_xxx_yyy = ~xxx_yyy**;

4.   同步后信号：**s** (synchronous)，例如：**s_wen_a**。

#### 1.4 参数、宏命名

```systemverilog
`define DEMO_MACRO 1
parameter  P_PARAM_A    = 2;
localparam P_PARAM_B    = 3;
localparam P_PARAM_A_LG = $clog2(PARAM_A);
parameter  type aaa_bbb_t  = logic [3:0];

module Demo
#(P_PARAM_A = 1,
  P_PARAM_B = 2,
parameter type
  aaa_bbb_t = logic [3:0],
localparam
  P_PARAM_C = 3
)(
...
);

endmodule : Demo
```

1.  由于 **参数** 和 **宏** 表示常数，与普通信号不同，因此所有字母全部大写(因为全大写字符串在编辑器中高亮与普通字符串不同)，以便于信号进行区分。只有传递数据类型的参数可以包含小写字母。除type参数以外，其他参数定义(parameter和localparam)以 **‘P_’** 开始。
2.  单词间用 ‘_’ 隔开。
3.  若某参数 **P_PARAM_A** 是 **P_PARAM_B** 的对数，可以写成 **P_PARAM_B_LG**，例如： **P_PARAM_B_LG = $clog2(P_PARAM_B)**。
4.  在端口中定义顺序为: **parameter** > **parameter type** > **localparam**。
5.  **localparam** 如果不会在端口定义中使用，可以在代码正文中定义。
6.  定义类型以 **‘_t’** 作为后缀，类型名用 **下划线命名法** 命名。

#### 1.4 特殊注释命名

-   文件中如果有待实现功能或待完善的注释，使用 **TODO** 标注。如果有BUG，使用 **FIXME** 标注。IDE中一般有插件可以列出文档中所有标注的位置。

### 2 格式规范

#### 2.1 文件头

每个设计文件都要包含文件头，端口处定义的参数和IO的完整注释要写在文件头中（防止代码中多行注释影响代码可读性），代码中可添加简要注释。在一个电路库文件中若包含多个单元，每个单元需要一个单独的文件头。文件头格式如下：

```systemverilog
// ==============================================================================
// Copyright (c) 2014-{YEAR} All rights reserved
// ==============================================================================
// Author  : your name <your email>@email.com
// File    : {FILE}
// Create  : {DATE} {TIME}
// Revise  : {RDATE} {RTIME}
// Editor  : {EDITER}
// Version : {VERSION}
// Description :
//    ...
//    ...
// Parameter   :
//    ...
//    ...
// IO Port     :
//    ...
//    ...
// Modification History:
//   Date   |   Author   |   Version   |   Change Description
// -----------------------------------------------------------------------------
// 19-06-02 |        |     0.1     |    Original Version
// ...
// ==============================================================================
```

#### 2.2 代码格式

##### 2.2.1 通用格式

1.  代码缩进 **禁止使用Tab** ，一律使用 **2空格** 。

2.  分隔符（如逗号、分号）后需要添加一个空格，运算符前后需要空格。

3.  begin 在当前行末尾，不重新开启一行, begin前添加一个空格。end 与 else 写在同一行。

    ```systemverilog
    always_comb begin
      if(...) begin
        ...
      end else if(...) begin
        ...
      end else begin
        ...
      end
    end
    ```

3.   语句间可以有1个或多个空格。多余一个空格可以方便对齐和查看(便于使用对齐插件查看代码)。例如下面的代码中, code 2 的定义在开启对其插件的情况下有更好的可读性，因为在wire之后多插入一个空格可以令下一行信号定义正确的对齐。

     ```
     // code 1
       wire testWire_1, testWire_2, testWire_3,
            testWire_4, testWire_5, testWire_6;
     // code 2
       wire  testWire_1, testWire_2, testWire_3,
             testWire_4, testWire_5, testWire_6;
     ```

4.   重要的block，及包含信号定义的block，需要添加 **block name** 。所有 **module**， **interface**， **package** 和 **有名字的block** 主要添加对应的 **ending name**。**block name** 和 **ending name** 之前的 **‘:’** 前后都需要添加空格。

     ```systemverilog
     module DemoModule();
       always_comb begin : DemoBlock
         ...
       end : DemoBlock
     endmodule : DemoModule
     ```

5.   单行注释前若有字符，至少需要隔两个空格

     `````
     sv语句 + 至少两个空格 + 单行注释
     `````

##### 2.2.2 module端口格式

端口格式定义如下：
```systemverilog
import DemoAaaPkg::*;       // 引用package，单独一行，前后无空格。
import DemoBbbPkg::*;       // 多个package写在不同的行中。
							// 空一行
module DemoLib_ModuleXxxYyy #( // 以"#("结尾，'#'前有空格
parameter					// 以 'parameter' 开头
  P_A = "_",                // 其他parameter在新的行中定义，定义前需要2个空格进行缩进。
  P_B = "_",
localparam                  // 若存在local parameter，localparam在新的一行中定义，前后无空格。
  P_B_LG = $clog2(P_B),     // local parameter定义格式与parameter相同。
  P_C = P_A - 1
)(                          // 在新的行中写 '参数定义右括号' 和 '端口定义左括号'。
  input        clk     ,
  input        rst_n   ,    // 端口在新行中定义，2个空格缩进。
  input        en_i    ,    // 端口定义顺序：input, inout, output。
  input        dat_i   ,    // 同方向端口定义顺序：clock, reset, enable, data。
  inout        vld_b   ,    // 端口和参数定义结尾的逗号分隔符可以对齐也可以不对齐。
  output logic dat_o        // 代码中端口部分参数和信号后可添加简要注释，完整注释在文件头中添加。
);                          // 端口定义 右括号 及 分号 单独一行，前后无空格。
  ...
endmodule : DemoLib_ModuleXxxYyy //单独一行，前后无空格。添加 ending name。':' 前后各有一个空格。


module DemoLib_Aaa (        // 如果没有模块中没有参数，直接定义左括号，'('前有空格。
  input  clk,
  input  rst_n,
  input  dat_i,
  output dat_o
);
  ...
endmodule : DemoLib_Aaa
```

如果模块端口较多，且不同端口连接模块不同，可以按照连接关系对端口进行分组：

```systemverilog
module DemoGroupIO (
  // function A IO
  input  a1,
  input  a2,
  output a3,
  // function B IO
  input  b1,
  output b2,
  // function C IO
  input c1,
  input c2
);
  ......
endmodule : DemoGroupIO
```

##### 2.2.3 module例化格式

1.  模块例化时，参数在例化时通过 **‘#()’** 直接传递，尽量避免使用 **defparam**，因为在最新的标准中，已经不推荐使用defparam定义参数。模块例化可以在同一行完成，也可以分多行完成。示例代码：

    ```systemverilog
    Adder
      U_Adder(
        .a(a),
        .b(b),
        .o(o)
      );
    
    Sub #(
      .type_A(logic [3:0]),
      .type_B(logic [3:0])
    ) U_Sub(
        .a(a),
        .b(b),
        .o(o)
      );
    
    And #(.width(8)) U_And(.a(a), .b(b), .o(o));
    And #(8) U_And(a, b, o);
    ```

2.   单行例化格式:

     ```
     2空格 + module名 + 1空格 + #( + 参数列表 + ) + 1空格 + 实例化名 + ( + 端口列表 +);
     ```

     端口列表的每一个','后有一个空格

3.   有参数例化格式:

     ```
     2空格 + module名 + 1空格 + #(
     4空格 +   第一个参数 ，
               ...
     4空格 +   第N个参数)
     2空格 + ) + 1空格 + 实例化名(
     6空格        端口连接...
     4空格 +    );
     ```

4.   无参数例化格式:

     ```
     2空格 + module名
     4空格 +    实例化名(
     6空格 +       端口连接...
     4空格 +    );
     ```

5.   **参数传递** 及 **端口连接** 格式

     -   尽量避免使用 **‘.\*’** 方式连接，容易引起隐藏的Bug。如果对代码非常熟悉，且很有必要的情况下可以使用 ‘.*’ 进行端口连接，使用这种方式需要在注释中说明连接了哪些端口。

     -   使用 **‘.port(signal)’** 连接信号和端口。

     -   若信号和端口命名相同，可以使用 **‘.port’** 方式连接。比如：带有寄存器的模块连接时，时钟和复位端口连接可使用 ‘.clk,.rst,’。

     -   连接顺序：input, inout, output。

     -   同方向端口顺序：clock, reset, clear, enable, data。

6.   若需要使用不指定端口，按顺序连接的方式，按照如下格式书写并用注释进行标注：

     ```systemverilog
     // Instantiation in a single line.
     ModuleName #(parameter_1) U1_ModuleName(signal_1,signal_2,signal_3,signal_4,signal_5);
     
     // Instantiation in multiple lines.
     // Ports are defined in different lines according to their direction.
     ModuleName #(
         P_A
     ) U2_ModuleName(
         signal_1,signal_2,  //input
         signal_3,           //inout
         signal_4,signal_5   //output
       );
     
     // Instantiation in multiple lines.
     // Each port is defined in its own line.
     ModuleName
       U3_ModuleName(
         signal_1, //port_1
         signal_2, //port_2
         signal_3, //port_3
         signal_4, //port_4
         signal_5  //port_5
       );
     ```

7.   完整示例代码：

     ```systemverilog
     module DemoModule (
       input        [3:0] dat_a_i,
       input        [3:0] dat_b_i,
       output logic [4:0][4:0] dat_sum_o,
       output logic [7:0] dat_mult_o
     );
     
       Mult #(
           .type_A(logic [3:0]),
           .type_B(logic [3:0]),
           .WIDTH($bits(datMult))
       ) U_Mult(
           .dat_a_i(dat_a_i),
           .dat_a_i(dat_b_i),
           .dat_o(dat_mult_o)
         );
     
       Adder
         U_Adder(
           .dat_a_i(dat_a_i),
           .dat_a_i(dat_b_i),
           .dat_o(dat_sum_o[0])
         );
     
       Adder
         U0_Adder(
           .dat_a_i(dat_a_i),
           .dat_a_i(dat_b_i),
           .dat_o(dat_sum_o[1])
         );
     
       Adder U1_Adder(dat_a_i, dat_b_i, dat_sum_o[2]);
     
       Adder
         U2_Adder(
           dat_a_i, dat_b_i,  //input
           dat_sum_o[3]       //output
         );
     
       Adder
         U3_Adder(
           dat_a_i,   //iDatA
           dat_b_i,   //iDatB
           dat_sum_o[4]  //oDat
         );
     
     endmodule : DemoModule
     ```

### 3 设计规范

#### 3.1 信号定义

```systemverilog
logic [3:0] w_dat, r_dat;
logic [3:0] w_dat_r;
wire  [3:0] w_dat_o = w_dat;
assign w_dat = ...;
always_ff(posedge clk)begin
  w_dat_r <= w_dat;
end

always_comb begin
  r_dat = func(...);
end
```

1.  所有信号使用 **logic** 定义。
2.  在定义时直接赋值的信号使用wire类型。因为logic不支持定义时赋值。
3.  组合逻辑电路表达式中包含function，使用always_comb赋值。因为assign赋值时，使用function可能引起仿真器bug。
4.  同向结构化信号，尽量使用struct定义。struct类型可以通过 **parameter type** 在不同模块间传递。

#### 3.2 位宽定义及固定值赋值

1.  MSB写在左侧，LSB写在右侧，LSB(Least Significant Bit)是“最低有效位”。MSB(Most Significant Bit)是“最高有效位”。
2.  LSB最好从0开始，如果有特殊需求，LSB可以从非零值开始，比如总线对齐：**logic [31:2] bus_addr;**
3.  固定值赋值使用以下方式：

    -   0赋值使用：**‘0**，例如：**assign dat = ‘0;**

    -   全1赋值值使用：**‘1**，例如：**assign dat = ‘1;**

    -   某确定值使用：**位宽 + ‘b/d/h/o + 数值**，例如：**assign dat = 8’d1;**

    -   使用cast操作符 **‘()** 进行参数化信号赋值：**参数 + ‘(带格式数值)**，例如：

        **assign dat = WIDHT’(‘d3);** 或 **assign dat = $bits(dat)’(3’b101);**

4.   当两个信号位宽有相关性，使用$bits()代替parameter定义信号位宽，这样可以使电路更利于复用，减少位宽对应参数变化引起的问题。

     ```systemverilog
     logic             en;
     logic [WIDTH-1:0] xx;
     // Use like this.
     logic [$bits(xx)*2-1:0] yy;
     assign yy[$bits(xx)-1:0] = {$bits(xx){en}} & xx;
     
     // Dont use like this!!
     logic [WIDTH*2-1:0] yy;
     assign yy[WIDTH-1:0] = {WIDTH{en}} & xx;
     ```

5.   尽量使用 **[位置+:位宽]** 或 **[位置-:位宽]** 方式赋值。

     ```systemverilog
     // Use like this.
     assign xx = dat[16+:8];
     assign yy = dat[16-:8];
     
     // Dont use like this!!
     assign xx = dat[23:16];
     assign yy = dat[15:8];
     ```

6.   使用packed方式定义多维向量信号。例如：**logic \[2:0][7:0] dat;**

7.   使用系统函数进行位宽相关计算。

     -   $bits：计算向量信号或struct信号的位宽。

     -   $size：计算当前向量中一共有多少组信号。

     -   \$signed/$unsigned：有符号/无符号位宽扩展

     -   $clog2：计算log2(x), 可以使用在端口定义中。

     -   $high：获取信号最高位。**使用时需要小心，信号有可能不是第0位开始**。

     -   $low： 获取信号最低位。


8.   如果定义MSB在左侧，LSB在右侧，从0开始的信号，比如：[7:0] 或 [31:0]，推荐使用预定义的宏进行位宽定义，可以增强可读性，降低bug出现几率。

     -   'w()：用 **参数**、**宏定义** 或 **计算式** 定义信号位宽。

     -   'b()：根据一个已知信号的定义 **相同位宽** 的信号。

     -   用法示例:

         ```systemverilog
         `define w(__width__)  (__width__):0
         `define b(__signal__) (__signal__):0
         
         logic [`w(P_WIDTH)] dat_a;
         logic [`w(P_WIDTH)]['b(datA)] dat_b;
         ```

#### 3.3 组合逻辑电路设计规范

1.  在设计中使用data mask写法：**yy = {$bits(xx){en}} & xx;**

    -   综合生产的电路简洁高效，Bug少。配合Onehot信号使用，效果极好。
    -   可以用来代替三目运算符 ()?: ，实现更好的性能。

        `````systemverilog
        // Use like this.
        assign dat  = {$bits(a){(x==2'd1 && y==2'd1)}} & a;
                      {$bits(b){(x==2'd2 && y==2'd2)}} & b;
                      {$bits(c){(x==2'd3 && y==2'd3)}} & c;
                      {$bits(d){(x==2'd4 && y==2'd4)}} & d;
        
        // Dont use like this!!
        assign dat  = (x==2'd1 && y==2'd1)? a :
                      (x==2'd2 && y==2'd2)? b :
                      (x==2'd3 && y==2'd3)? c : d;
        `````

2.   使用操作符：**inside**。

     ```systemverilog
     assign dat_en = dat inside {2'd1, 2'd3};
     ```

3.   使用操作符：**{<<N{Signal}}** 或 **{>>N{Signal}}** 。

     ```systemverilog
     {<<M{N}}    //从N中按顺序流出大小为M的块，从最右边的块开始，到最左边的块结束。
     {>>M{N}}    //从N中按顺序流出大小为M的块，从最左边的块开始，到最右边的块结束。
     
     assign a = {<<2{b}};
     assign b = {>>2{c}};
     ```

4.   使用操作符：**==?** 或 **!=?**。

     ```systemverilog
     assign dat_en = (a ==? 3'b1?1) & (b !=? 3'b??1);
     ```

5.   使用操作符：**’( )**。

     ```systemverilog
     logic [7:0][2:0] a;
     typedef logic [2:0][7:0] dat_t;
     dat_t b;
     assign b = dat_t'(a);
     ```

6.   使用操作符：**>>>**。**该操作符必须对signed类型信号是用，否则计算结果错误**。

     ```systemverilog
     logic signed [7:0] a,b;
     assign a = b >>> 4;
     ```

7.   组合逻辑使用 **assign** 和 **always_comb** 块。在always_comb块中，使用 **‘=’** 赋值。

8.   在组合逻辑中，if尽可能只与else搭配， **极不推荐使用else if** 。如果有多判断条件存在，使用case语句。简单的 if()…else… 语句综合生成无优先级电路。而 if()… else if()… else… 语句在大多数情况下会综合出带优先级逻辑。为避免过度使用else if引入不必要的逻辑路径，禁止在逻辑电路中使用该语法。需要复杂带优先级判断的情况使用if…else进行嵌套，这样的规定可以显式的表明优先级，同时提醒设计人员在设计之初就考虑电路优化。

     ```systemverilog
     always_comb begin
       if(...)begin
         ...
       end else begin // no else if(..) !!
         ...
       end
     end
     ```

9.   case 语句用法规范。

     -   case条件如果互斥，使用：**unique case(xxx) inside** 或 **unique case(1’b1)**
     -   case条件若非互斥，使用：**priority case(xxx) inside** 或 **priority case(1’b1)**
     -   设计中，尽量使用 **unique case** 。综合后生成无优先级电路，priority生成带优先级电路。
     -   case条件复杂，需要在判断条件后添加注释说明判断条件含义。
         -   default规范:
             -   若有一个固定默认值，则default为固定值。
             -   若case条件已经是full case，则default替换为：’**// full case**’。注意在full case的情况下，不要写default，不然综合器会发现无法进入该条件，报warning。
             -   若不存在default，则禁止使用case语句。这种情况使用case有可能造成后仿与前仿行为不一致。
     -   case最多允许嵌套2层。
     -   尽量避免使用太长的case语句。如果逻辑过于复杂，建议拆分逻辑实现。

     ```systemverilog
     always_comb
       unique case(xx) inside
         2'b00: dat0 = ...;
         2'b01: dat0 = ...;
         2'b10: dat0 = ...;
         2'b11: dat0 = ...;
         // full case
       endcase
     
     always_comb
       priority case(xx) inside
         2'b0?: dat1 = ...;
         2'b10: dat1 = ...;
         default: dat1 = '0;
       endcase
     
     always_comb
       unique case(1'b1)
         a==2'b00 && b==2'b00: begin // Add comments for case 1 condition!
           dat2 = ...;
         end
         a==2'b11 && b==2'b11: begin // Add comments for case 2 condition!
           dat2 = ...;
         end
         default: dat2 = '0;
       endcase
     ```

10.   在always块中对多个信号进行条件赋值时，必须在所有条件下对每个信号赋值。设计时可采用下一方法中任意一种：

      -   在条件赋值前给信号默认值，在条件赋值时对部分信号赋值。

      -   在所有条件分支中写明所有信号赋值。

      ```systemverilog
      // Assignment with default value
      always_comb begin
        a = '0;
        b = '0;
        if(x) begin
          a = '1;
        end else begin
          b = '1;
        end
      end
      
      // Assign value to all signals in each condition
      always_comb begin
        if(x) begin
          c = '1;
          d = '0;
        end else begin
          c = '0;
          d = '1;
        end
      end
      
      // Code write as below is wrong. It will generate latches.
      always_comb begin
        if(x) begin
          m = '1;
        end else begin
          n = '1;
        end
      end
      ```

#### 3.4 时序电路设计规范

1.  寄存器设计使用：**always_ff**， 赋值符号：**<=**
2.  锁存器设计使用：**always_latch**， 赋值符号：**=**
3.  寄存器设计时，信号顺序遵循：reset > clear > enable > assignment。**信号保持的else不要写**。

```systemverilog
always_ff (posedge clk) begin
  if(!rst_n)
    dat <= P_INI_DAT;
  else
    if(clr)
      dat <= P_INI_DAT;
    else
      if(en)
        if(a)
          dat <= datA;
        else
          dat <= datB;
      //else  <---------- Dont write the assignment for data keeping!
      //  dat <= dat; <-- Dont write the assignment for data keeping!
      ...
end
```

4.   在IC设计中，使用 **异步低有效** 复位。标准单元对于这种复位方式支持更好。
5.   在FPGA设计中，使用 **同步高有效** 复位。
6.   尽量简化寄存器块中逻辑判断电路的复杂度，需要复杂逻辑的场景中，先使用组合逻辑电路计算寄存器数据，再保存到寄存器中。
7.   推荐使用module对寄存器进行封装，在需要寄存器电路时直接调用。

#### 3.5 参数定义规范

1.  只有可能在例化时传递的参数才可以定义为parameter，其他模块内部的信号都要定义为localparam。
2.  localparam可以定义在端口处，也可以在代码内有需要的地方再定义。
3.  如果一个信号类型在模块端口或内部多次使用，则可以在module起始位置定义信号的type。
4.  verilog参数默认无类型, 会根据实际传递参数的不同。参数定义时，仅使用以下俩种类：
    -   无类型字符串。不带类型的字符串参数，可以通过传递不同的名字，使得同一模块有不同实现。尤其适用于硬核IP的例化（比如：SRAM）。该类型参数必须在其他参数前定义。字符串定义使用双引号 **”**（sv中字符串不支持单引号）。在定义字符串参数时，应尽量减少字母数量，防止综合后，在module名中加入过长的字符串。
    -   固定值参数。sv支持多维参数，且只有packed类型的多维参数可以使用{}运算符统一赋值，因此所有固定值参数要显式定义类型。由于int类型不支持packed数组，因此可以统一定义参数类型 **intp：typedef logic [31:0] intp;**。所有参数使用intp定义。
5.  如果参数必须在例化时手动传入，则在参数定义时，不要设置默认值。
6.  如果需要定义动态参数矩阵，需要先定义矩阵维度参数。设计过程中注意不要超出参数范围。代码如下所示：

```
module TestModule #(
  P_STRING = "TestModule"     ,
  intp P_NUM    = 2           ,
  intp P_ADDRS  = {0,1}       ,
  intp P_WIDTH
)(
  ...
);
  ...
endmodule : TestModule

module tb;
  TestModule #(
      .P_STRING("U_Test")
      .P_NUM(4)
      .P_ADDRS({32'd0,32'd1,32'd2,32'd3}),
      .P_WIDTH(32'd32) // P_WIDTH is necessary, since it has no default value.
) U_Test(
      ...
  );
  ...
endmodule : tb
```

#### 3.6 例化设计规范

-   见一、2.2.3

#### 3.7 FSM设计规范

TODO

### 4 仿真规范

#### 4.1 信息打印规范

1.  参数检查等功能可以放置在initial块中。发现代码错误的情况使用 **$error**, 不使用**$display**,因为**$error**在打印信息时可以提示错误位置。添加宏定义控制变量 **CHECK_ERR_EXIT**。开启该宏后，发生错误时可以停止仿真。
1.  TODO



## 二、高级语法

### 1 参数化电路设计规范

1.  可以不显示声明 **generate** 和 **endgenerate**。不声明在标准中允许，且没有任何影响。

2.  给必要的生成块 **添加block name** ，带有block name的块需要添加对应的 **ending name**。若在生成语句中调用module或定义信号，该module或信号通过block name访问。若没有显示定义block name，仿真器或综合器会自动生成。**在使用生成块时，尽量给所有的生成块命名。**

3.  互斥的block可以使用 **相同的block name**。例如：

    ```systemverilog
    if(...) begin : BlkName
      ...
    end else begin : BlkName
      ...
    end
    ```

4.  所有在生成语句中使用的所有变量都是 **固定值** (参数 或 宏)，不是电路中的信号。

5.  条件电路生成使用宏定义实现，用以区分电路中信号的 if 判断。

    ```
    `define gen_if   if
    `define gen_elif else if
    `define gen_else else
    
    `gen_if(P_PARAM_A == 1) begin : dat
      assign dat1 = xx;
    end `gen_else begin: dat
      assign dat1 = yy;
    end
    
    always_comb begin
      `gen_if(P_PARAM_A == 1) begin
        dat2 = xx;
      end `gen_else begin
        dat2 = yy;
      end
    end
    ```

6.  生成块中for循环写法：**for(genvar i = 0; i < xx; i++)**

7.  always中for循环写法：**for(int i = 0; i < xx; i++)**

8.  for循环的边界判断尽量使用系统函数根据信号进行自动推断。循环变量自加可以使用 **‘i++’** 计算符。

9.  always中如果需要遍历一个向量内的所有信号，使用foreach循环实现：**foreach(dat[i])**

    ```systemverilog
    // Define signal outside the loop generate block is recommended.
    logic [P_WIDTH-1:0] dat1;
    // It is recommended to use $bits(dat1) instead of P_WIDTH.
    for(genvar i = 0; i < $bits(dat1); i++) begin : dat1_gen
      assign dat1[i] = xx[i];
    end : dat1_gen
    
    // Use P_WIDTH is also allowed.
    for(genvar i = 0; i < P_WIDTH; i++) begin : dat2_gen
      wire dat2 = xx[i];  // The signal dat2 can only be accessed by block name: dat2Block.
    end : dat2_gen
    
    logic [P_WIDTH-1:0] dat3, dat4;
    always_comb begin
      for(int i = 0; i < $bits(dat3); i++) begin
        dat3[i] = xx[i];
      end
    end
    
    always_comb begin
      foreach(dat_a_i[i]) begin
        dat4[i] = dat_a_i[i];
      end
    end
    ```

### 2 struct 用法规范

```systemverilog
// Define a struct signal directly.
struct packed{
  logic [7:0] dat;
} dat1_st;

// Define a struct type and define a struct signal by the new type.
typedef struct packed{
  logic [7:0] dat;
} Demo2St;
Demo2St dat2_st;

// Parameterized struct definition by macro.
`define TypedefDemoSt(width) \
  typedef struct packed{\
    logic [width-1:0] dat;\
  }
`define TypeDemoSt(width) \
  struct packed{\
    logic [width-1:0] dat;\
  }

// Define struct type and struct variable by macro.
`TypedefDemoSt(8) Demo3St;
Demo3St dat3_st;
`TypeDemoSt(8) dat4_st;

// Type convert by casting operating.
assign dat3_st = dat3_st'(dat4_st);
```

1.  同方向有相关性信号，推荐使用struct定义。
2.  结构体定义必须使用packed形式。
3.  直接使用struct定义在不同位置的变量会被EDA工具认为是两个不同变量。当需要在多处定义相同struct时，使用typedef形式定义类型，类型名用 **大驼峰** 命名法，结尾用 **St** 作为后缀。
4.  struct定义的变量用 **下划线** 命名法，**st** 作为后缀。
5.  使用宏实现参数化struct定义，建议同时定义 typedef 和 非typedef 两种方式。两种宏分别以：**Typedef** 和 **Type** 作为前缀，使用 **大驼峰** 命名法，**St** 作为后缀。(SystemVerilog标准中使用virtual class实现参数化struct定义，该语法尚未被部分EDA工具支持。)
6.  struct 可以使用 **‘( )** 操作符。
7.  union定义方式与struct相同，变量后缀为 **un** 。

### 3 package 用法规范

```systemverilog
package BasicPkg;
  parameter P_A = 1;

  function automatic logic [3:0] DatAnd(input [3:0] in1, in2)
    return in1 & in2;
  endfunction
endpackage
```

1.  有相关性的信号、参数、数据类型、函数可以集合在一起定义在一个package内。
2.  package以 **大驼峰** 方式命名，以 **Pkg** 作为名称结尾。
3.  package内的定义都不支持参数化。(SystemVerilog标准中尚不支持)
4.  package中定义的function必须 **包含automatic** 声明，否则综合时会报错。

### 4 interface 用法规范

```systemverilog
interface DemoItf #(
    P_A = 1,
    P_B = 2,
    ...
);

  logic [3:0] dat_vec;  // All signal defined in 'logic'.
  logic [1:0] dat;
  logic       dat_vec0, dat_vec1, dat_vec2, dat_vec3;
  typedef struct packed{logic dat1; logic [1:0] dat2;} DataSt;
  
  assign dat_vec0 = dat_vec[0];
  assign dat_vec1 = dat_vec[1];   // Only bit selection/extension is allowed.
  assign dat_vec2 = dat_vec[2];
  assign dat_vec3 = dat_vec[3];

  function automatic void Codec;  // 'automatic' is necessary.
    dat = {(dat_vec3|dat_vec2), (dat_vec3|dat_vec1)};
  endfunction
  
  function automatic logic BiggerThanOne;
    return {(dat > 2'd1), dat};
  endfunction

  modport DatVecOut(output dat_vec);
  modport DatIn(input dat, import BiggerThanOne); // import function in modport.
  modport Unit(input dat_vec0, dat_vec1, dat_vec2, dat_vec3, output dat, import Codec);

endinterface : TestItf

module TestItfUnit
(
  TestItf.Unit dat_if_b
);
  dat_if_b.Codec();
endmodule : TestItfUnit

module ModuleBb
(
  TestItf.DatIn dat_if_i,
  output logic result_o
);
  typedef dat_if_i.DataSt DatSt;  // Use typedef in interface.
  DatSt data_st;
  assign data_st = dat_if_i.BiggerThanOne();   // Use function in interface.
  assign result_o = data_st.dat1;
endmodule : ModuleBb
```

1.  interface名称定义使用 **Itf** 作为后缀，信号定义使用 **If** 作为后缀。内部信号使用 **logic** 或 **struct** 定义。
2.  modport名称采用**大驼峰**命名
3.  用于模块间互联的interface中，值允许存在 **位选择、位截取、位扩展** 逻辑电路，不能存在任何会生成具体器件的逻辑电路。
    -   interface中实现的电路逻辑在综合后会直接出现在例化interface的module中，这种写法不利于综合、后端流程。因此不允许直接在interface中实现具体电路。
    -   位选择、位截取、位扩展逻辑并不存在实际电路，只是改变连接关系，不影响其他流程。

3.   实现与interface相关性较高的逻辑，通过以下两种方式：

     -   单独实现一个module。

         设计一个单独的module，将interface作为接口，将逻辑放置在module内。对于复杂的电路实现，推荐这种方式。

     -   在interface中设计 **function** 或 **task**。

         1.  将需要实现的功能设计在interface的 **function** 或 **task** 中。
         2.  通过modport将 function 或 task 直接 **import** 到module中。
         3.  在module中直接调用。
         4.  function 或 task 可以直接访问interface里的信号，不需要通过端口传递。
         5.  建议只使用function，不使用task。在function中不要放置带复杂的逻辑。

4.   可以在interface中typedef数据类型，通过interface将数据类型引入到module中。
5.   标准中允许在module中直接访问interface中的parameter，该功能目前尚未被全部EDA工具支持。
6.   减少在interface中的input信号数量，尤其是会参与计算的信号。在测试中遇到过相关EDA工具Bug。
7.   interface在端口定义和信号连接时必须 **指定modport** 。否则综合会提示信号未使用warning。
8.   通过 **interface + modport + 参数化设计** 可以实现verilog可变端口数量。
9.   利用interface可以进行电路封装，将通用电路在interface内实现，可选电路使用function实现。在使用时，根据需要调用对应的function。在此情况下interface内可以包含实际电路。



## 三、特殊语法

### 1 基于宏模板的电路设计方法

在进行电路设计时，很多特定的电路有固定的描述方法。为了提高电路的复用性，可以通过设计电路库的方式提供基础常见的电路模块。但是通用电路模块为了保持通用性，需要引入大量参数，导致电路调用时除了复杂的端口连接，还需要参数传递，而参数传递发生错误在很多情况下只会出现warning，很难进行debug。通过模板的方式可以简化电路描述中很多代码书写，部分实现自动位宽匹配，比如：例化时指定位宽、可参数化代码中的信号扩展等等。此外可以在可以在电路模板中加入assert用于进行静态验证。

使用宏的设计方法，要注意不要重复定义宏。为了方便定义，需要在一个文件中单独定义错误检验宏: **__DefErr__** 。

```systemverilog
`ifdef __DefErr__
  Macro Define Error: __DefErr__ has already been defined!!
`else
  `define __DefErr__(Str) Macro Define Error: Str has already been defined!!
`endif
```

为了方便设计，可以使用以及定义好的通用宏定义文件：CommonDefine.svh

#### 1.1 基于宏的模板例化方法。

利用Verilog进行电路设计时，大部分参数与可以通过接口连接的信号进行推算。例如：数据位宽、地址对应的memory深度等等。因此在该设计方法中，使用宏作为module模板，减少需要显式传递的参数数量。宏模板定义代码如下：

```systemverilog
// Macro template defination for CircuitLib_Adder.
`ifdef CircuitLib_Adder
  `__DefErr__(CircuitLib_Adder)
`else
  `define CircuitLib_Adder(UnitName, B_TYPE_MT, dat_a_i_MT, dat_b_i_MT, dat_o_MT) \
CircuitLib_Adder  #(
  .WIDTH_A($bits(iDatA_MT)),   \
  .WIDTH_O($bits(oDat_MT)),    \
  .TYPE_B(B_TYPE_MT)           \
) UnitName(                    \
    .dat_a_i(dat_a_i_MT),      \
    .dat_b_i(dat_b_i_MT),      \
    .dat_o(dat_o_MT)           \
  )
`endif
module CircuitLib_Adder #(
  P_WIDTH_A = "_",  // width of dat_a_i
  P_WIDTH_O = "_",  // width of dat_o
parameter type
  B_TYPE    = "_"   // data type of iDatB
)(
  input  [P_WIDTH_A-1:0] dat_a_i,
  input  B_TYPE          dat_b_i,
  output [P_WIDTH_O-1:0] dat_o
);
  assign dat_o = dat_a_i + dat_b_i;
endmodule: CircuitLib_Adder
```

按照规范设计module。在module定义上声明宏模板。宏模板格式：

1.  重定义检查。该写法不符合verilog语法，因此在编译时，无论编译选项如何设置，只要发现重复宏定义就会报Error。
2.  定义宏模板，宏模板定义第一行无空格，结尾直接使用 ‘**\\**’ 换行。
3.  宏对应的module在新一行中直接按照例化格式书写。 **此处代码缩进以行首为准，不以上一层define为准，便于EDA工具展开宏后进行代码调试。**
4.  参数中先定义例化名称，再定义type参数，最后定义输入、输出端口。
5.  输入输出端口与端口名称相同，增加 ‘_MT’ 后缀(Macro Template)。
6.  完成module定义。
7.  定义module时，参数在两个 ‘**//**’ 间标注计算方法。在后面的 ‘**//**’后写注释。若某参数与端口无关则不标注计算方法。(该写法用于宏自动生成)
8.  除最后一行外，其他行宏以 ‘**\\**’ 结尾。(多行宏定义标准写法)
9.  结束条件编译。

10.   宏模板例化时可参考无端口声明module例化方式。

#### 1.2 基于宏的电路设计方法

由于目前调用module进行电路设计有诸多限制(不能在interface中使用等等)，而标准中规定的参数化function还有很多EDA工具无法支持，因此需要使用宏对需要封装的电路进行设计，以实现类似参数化function功能。(若EDA工具更新对参数化function的支持，则不再使用此方法)

```systemverilog
`ifdef CircuitLib_MaskM
  `__DefErr__(CircuitLib_MaskM)
`else
  `define CircuitLib_MaskM(en,dat) ({$bits(dat){en}} & dat)
`endif

`ifdef CircuitLib_OnehotM
  `__DefErr__(CircuitLib_OnehotM)
`else
`define CircuitLib_OnehotM(dat_i, dat_i) \
foreach(dat_o[i])begin           \
    dat_o[i] = (dat_i[i] == i);  \
end                              \
`endif
```

定义方式与基于宏的例化相似。定义宏前要检查是否出现重定义错误。若没有重定义，则定义宏电路。宏电路以 **‘M’** 作为后缀。其他定义方式与前述相同。**此处电路描述代码缩进以行首为准，不以上一层define为准，便于EDA工具展开宏后进行代码调试。定义时，第一行无缩进，其他行在缩进两次基础上再根据需要缩进** 只有在 **以下两种情况下** 推荐使用宏定义进行电路设计：

-   **单行宏** ：当前电路需要在一行代码内实现，即要实现类似function中return效果。
-   **多行宏** ：当前电路可能会在always_comb块、function中使用。

宏电路设计方法只适用于常用基础电路，复杂电路必须使用module实现。对于所设计的宏电路，必须在文档中明确标识该宏适用于哪种场景。对于同一功能，可能同时存在module实现和宏实现，此时优先使用module来完成电路设计。基于宏的电路模块调用方式如下：

```systemverilog
module Test;
...

  logic [width-1:0] dat_vec;
  always_comb begin
    `CircuitLib_OnehotM(dat,dat_vec);
  end
  logic [width-1:0] final_dat;
  assign final_dat = `CircuitLib_MaskM(en,dat_vec);

endmodule: Test
```

#### 1.3 注意事项

由于SystemVerilog语言本身的语法缺失，只能采用宏进行电路设计。利用宏模板设计方法后，设计电路代码看起来很像编程语言中的函数调用。此处必须要注意：**宏模板设计是利用宏在电路中实例化一个标准电路，不是函数调用，与编程语言中的函数调用有本质区别。**

### 2 基于宏电路库的设计方法

Verilog/SystemVerilog中没有基于库、包的设计方法，也没有对应的库、包管理方法。不利于设计复用。因此我们在宏模板基础上，利用宏进行电路库管理。对于一个设计好的电路库(CircuitLib)，包含三个文件，文件均以电路库名称命名，后缀名不同，所有文件放置在同一个以库名命名的文件夹中：

-   CircuitLib.svh

    头文件：包含该电路库中通用的数据类型、宏等。为了实现类似import的包管理，需要在该文件中定义宏缩写声明。该文件中需要包含当前库需要调用的其他电路库。

-   CircuitLib.vm

    宏电路文件：三、1.2中规定的基于宏的电路设计模块都要在该文件中定义。该文件不是电路库的必须文件。

-   CircuitLib.sv

    标准电路文件：所有package，interface，module都要定义在该文件中。

#### 2.1 宏电路文件

所有宏电路都定义在同一个宏电路文件中，定义方式与3.1.2中相同。如下示例代码中展示了CircuitLib电路库的宏电路文件(CircuitLib.vm)。该文件中定义了一个MaskM宏，一个OnehotM宏。

```systemverilog
`ifdef CircuitLib_MaskM
  `__DefErr__(CircuitLib_MaskM)
`else
  `define CircuitLib_MaskM(en,dat) ({$bits(dat){en}} & dat)
`endif

`ifdef CircuitLib_OnehotM
  `__DefErr__(CircuitLib_OnehotM)
`else
`define CircuitLib_OnehotM(dat_i, dat_i) \
foreach(dat_o[i])begin           \
    dat_o[i] = (dat_i[i] == i);  \
end                              \
`endif
```

#### 2.2 标准电路文件

所有package、interface和module都定义在标准电路文件中。在文件内定义顺序为 **package -> interface -> module** , 同优先级下，按首字母排序,由于package内部可能有依赖关系，若存在依赖关系，以依赖关系为准。若是几个module(package、interface)有一定相关性(属于同一类型不同配置 或 一同构成一个大IP)，可以在库内分成不同的section。示例代码如下：

```systemverilog
//section: DemoSection++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// package

package CircuitLib_DemoPkg;
  typedef logic [3:0] type_Dat;
endpackage: CircuitLib_DemoPkg

// interface

interface CircuitLib_InvOutItf;
  logic [3:0] dat;
endinterface: CircuitLib_InvOutItf

// module
///////////////////////////////////////////////////////////////////////////////
// Module name : CircuitLib_Inv
// Author      : 
// Date        : 2019-06-20
// Version     : 0.1
// Description :
//    ...
//    ...
// Modification History:
//   Date   |   Author   |   Version   |   Change Description
//==============================================================================
// 19-06-02 |            |     0.1     |    Original Version
// ...
////////////////////////////////////////////////////////////////////////////////
`ifndef Disable_CircuitLib_Inv
`ifdef CircuitLib_Inv
  `__DefErr__(CircuitLib_Inv)
`else
  `define CircuitLib_Inv(UnitName, dat_i_MT, dat_o_MT) \
CircuitLib_Inv  #(             \
  .WIDTH($bits(iDat_MT))       \
) UnitName(                  \
    .dat_i(dat_i_MT),        \
    .dat_o(dat_o_MT)         \
  )
`endif
module CircuitLib_Inv #(
  WIDTH = "_"  //$bits(iDat)//
)(
  input  [WIDTH-1:0] dat_i,
  output [WIDTH-1:0] dat_o
);
  assign dat_o = ~dat_i;
endmodule: CircuitLib_Inv
`endif
//endsection: DemoSection+++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

标准电路文件中，电路代码规范与文档中其他部分介绍相同。由于所有module都定义在同一个文件中，为了方便电路改动，增加模块编译开关。在示例代码中，CircuitLib_Inv模块定义前增加编译开关：**`ifndef Disable_CircuitLib_Inv** 。在工程中如果需要自己重新实现该模块，可以使用该宏命令屏蔽此模块，用重新设计的代码进行替换。

给每一个宏、package、interface、module增加 **注释头** (类似文件头), demo中为了简化代码，只定义了CircuitLib_Inv模块的注释头。定义格式与文件头类似。

**section**定义方式：

-   起始：’//’ + ‘section: ‘ + SectionName + ‘+++++++…+++++’
-   结束：’//’ + ‘endsection:’ + SectionName + ‘+++++++…+++++’

#### 2.3 头文件

宏库头文件书写格式如下所示。

```systemverilog
`define CircuitLib_MacroDef(ImportName, DefName)                      \
  `ifdef ImportName``DefName                                              \
    Macro Define Error: ImportName``DefName has already been defined!!    \
  `else                                                                   \
    `define ImportName``DefName `CircuitLib_``DefName                 \
  `endif
`define CircuitLib_PackageDef(ImportName, DefName)                    \
  `ifdef ImportName``DefName                                              \
    Macro Define Error: ImportName``DefName has already been defined!!    \
  `else                                                                   \
    `define ImportName``DefName CircuitLib_``DefName                  \
  `endif
`define CircuitLib_InterfaceDef(ImportName, DefName)                  \
  `ifdef ImportName``DefName                                              \
    Macro Define Error: ImportName``DefName has already been defined!!    \
  `else                                                                   \
    `define ImportName``DefName CircuitLib_``DefName                  \
  `endif
`define CircuitLib_ModuleDef(ImportName, DefName)                     \
  `ifdef ImportName``DefName                                              \
    Macro Define Error: ImportName``DefName has already been defined!!    \
  `else                                                                   \
    `define ImportName``DefName `CircuitLib_``DefName                 \
  `endif
////////////////////////////////////////////////////////////////////////////////////////

`define Use_CircuitLib(ImportName)                 \
  `CircuitLib_MacroDef(ImportName, MaskM)          \
  `CircuitLib_MacroDef(ImportName, Onehot)         \
  `CircuitLib_PackageDef(ImportName, DemoPkg)      \
  `CircuitLib_InterfaceDef(ImportName, InvOutItf)  \
  `CircuitLib_ModuleDef(ImportName, Inv)

`define Unuse_CircuitLib(ImportName) \
  `undef ImportName``MaskM               \
  `undef ImportName``Onehot              \
  `undef ImportName``DemoPkg             \
  `undef ImportName``InvOutItf           \
  `undef ImportName``Inv

////////////////////////////////////////////////////////////////////////////////////////
```

-   文件分为两部分：

    第一部分为通用宏定义，可以用宏直接定义不同的module等。CircuitLib_MacroDef：用于定义 **宏** 和 **模板类型**。CircuitLib_PackageDef：用于定义 **package**。CircuitLib_InterfaceDef：用于定义 **interface**。CircuitLib_ModuleDef：用于定义 **module**。这四个定义宏中，公共部分为电路库名称，建立新库时，需要将该部分内所有 **CircuitLib** 替换为新库名称。第二部分为宏库的具体定义。定义格式：**Use_CircuitLib(ImportName)**。CircuitLib 为库名称。ImportName为在module内调用时使用的缩写。当一个module内使用多个库时，该缩写可以用于找到电路库名称。由于宏定义是全局有效，为了避免互相干扰，需要在宏库使用完毕后将已定义的宏进行 **undefine**。因此用相同的方法定义Unuse宏。

#### 2.4 宏库使用方法

假设已经存在 CircuitLib 电路库中的相关文件。库的使用可以作用于一个 **module(interface)** 或者一个 **文件**，例子如下：

```systemverilog
// file A, `Use_XxxLib @ beginning of the module, `Unuse_XxxLib @ end of the module.
`Use_CircuitLib(z)
import `zDemoPkg::*;
module TestModule (
  input               en,
  input  DatType      dat_i,
  output logic [15:0] dat_o
);

  `zInvOutItf dat_out();
  `zInv(U_Inv,iDat,dat_out.dat);

  always_comb begin
    `zOnehot(dat_out_oh,dat_out.dat);
  end
  assign dat_o = `zMaskM(en,dat_out_oh);

endmodule: TestModule

`Unuse_CircuitLib(z)

// file B， `Use_XxxLib @ beginning of the file, `Unuse_XxxLib @ end of the file.
`Use_CircuitLib(z)
...
...
`Unuse_CircuitLib(z)
```

在module下一行import之前引用宏库：`Use_CircuitLib(z)

-   该语句结尾无 **;** 。
-   括号内 **z** 为宏库名缩写，与 python 中 import … as 类似。
-   此时，库内任意元素的调用，以 **z** 开头，为了表示更加清晰，可以增加下划线 **z_**。
-   若当前module只使用了一个宏库，则括号内可以指定缩写也 **可以为空** ，此时直接调用元素即可。
-   无论缩写内容是什么，宏都会扩展为全名，比如：**`zInv -> CircuitLib_Inv**，因此在仿真、综合中相关内容都是以该库元素全名显示。
-   在endmodule前 **Unuse** 相应的库：**`Unuse_CircuitLib(z)**。

## 四、Bug

-   暂无

## 五、附录

### 1 信号命名缩写惯例

本章节中规定了信号缩写的建议用法，建议在使用信号过程中多写全称，如果需要缩写，请参考列表中的规定。

| 1W   | 2w   | 3w   | 4w   | 5w    | name                   |
| ---- | ---- | ---- | ---- | ----- | ---------------------- |
| -    | -    | ack  | -    | -     | acknowledge            |
| -    | -    | -    | addr | -     | address                |
| -    | -    | -    | -    | alloc | allocate               |
| -    | -    | -    | avlb | -     | available              |
| -    | -    | -    | asyn | -     | asynchronous           |
| -    | -    | buf  | -    | -     | buffer                 |
| -    | -    | blk  | -    | -     | block                  |
| -    | bj   | -    | -    | -     | branch & jump          |
| -    | -    | cap  | -    | -     | capacity               |
| -    | -    | clr  | -    | -     | clear                  |
| -    | ck   | clk  | -    | -     | clock                  |
| -    | ch   | -    | -    | -     | channel                |
| -    | cs   | -    | -    | -     | chip select            |
| -    | -    | cmp  | -    | -     | compare                |
| -    | -    | cmd  | -    | -     | command                |
| -    | -    | cfg  | -    | -     | config, configuration  |
| -    | -    | -    | ctrl | -     | control                |
| -    | -    | cnt  | -    | -     | counter                |
| -    | -    | -    | curr | -     | current                |
| -    | cs   | -    | -    | -     | current state          |
| -    | -    | dat  | -    | -     | data                   |
| -    | -    | dst  | -    | -     | destination            |
| -    | -    | div  | -    | -     | division               |
| -    | en   | -    | -    | -     | enable                 |
| -    | eq   | -    | -    | -     | equal                  |
| -    | -    | err  | -    | -     | error                  |
| -    | -    | -    | exec | -     | execute                |
| -    | -    | -    | extd | -     | extend                 |
| -    | -    | fnl  | -    | -     | final                  |
| -    | -    | fls  | -    | -     | flush                  |
| -    | -    | fwd  | -    | -     | forward                |
| -    | -    | frm  | -    | -     | frame                  |
| -    | -    | frz  | -    | -     | freeze                 |
| -    | -    | -    | freq | -     | frequency              |
| -    | -    | img  | -    | -     | image                  |
| -    | -    | imm  | -    | -     | immediate              |
| -    | -    | idx  | -    | -     | index                  |
| -    | -    | -    | init | -     | initial                |
| -    | -    | ins  | -    | -     | instruction            |
| -    | -    | int  | -    | -     | interrupt              |
| -    | -    | inv  | -    | -     | inveter                |
| -    | -    | len  | -    | -     | length                 |
| -    | -    | mst  | -    | -     | master                 |
| -    | -    | mem  | -    | -     | memory                 |
| -    | -    | msg  | -    | -     | message                |
| -    | -    | mid  | -    | -     | middle                 |
| -    | -    | mul  | -    | -     | multiplication         |
| -    | -    | nxt  | -    | -     | next                   |
| -    | ns   | -    | -    | -     | next state             |
| -    | -    | num  | -    | -     | number                 |
| -    | oh   | -    | -    | -     | onehot                 |
| -    | op   | -    | -    | -     | operation              |
| -    | -    | pkg  | -    | -     | package                |
| -    | -    | pkt  | -    | -     | packet                 |
| -    | -    | ptr  | -    | -     | pointer                |
| -    | -    | -    | prev | -     | previous               |
| -    | -    | -    | prim | -     | primary                |
| -    | -    | -    | prio | -     | priority               |
| r    | rd   | -    | -    | -     | read                   |
| -    | -    | rdy  | -    | -     | ready                  |
| -    | -    | reg  | -    | -     | register               |
| -    | -    | rst  | -    | -     | reset                  |
| -    | -    | -    | rslt | -     | result                 |
| -    | rx   | rcv  | -    | -     | receive                |
| -    | -    | rls  | -    | -     | release                |
| -    | -    | req  | -    | -     | require                |
| -    | -    | res  | -    | -     | resource               |
| -    | -    | -    | resp | -     | response               |
| -    | -    | ret  | -    | -     | return                 |
| -    | -    | sel  | -    | -     | select                 |
| -    | -    | slv  | -    | -     | slave                  |
| -    | -    | src  | -    | -     | source                 |
| -    | -    | stl  | -    | -     | stall                  |
| -    | -    | -    | stat | -     | state                  |
| -    | -    | std  | -    | -     | standard               |
| -    | -    | sum  | -    | -     | summary                |
| -    | -    | sub  | -    | -     | subtraction            |
| -    | -    | syn  | -    | -     | synchronous            |
| -    | -    | sys  | -    | -     | system                 |
| -    | -    | tgt  | -    | -     | target                 |
| -    | -    | tmp  | -    | -     | temporary              |
| -    | tx   | -    | trsm | -     | transmit, transmission |
| -    | -    | tmp  | -    | -     | template               |
| -    | -    | -    | updt | -     | update                 |
| -    | -    | usr  | -    | -     | user                   |
| -    | -    | vld  | -    | -     | valid                  |
| -    | -    | val  | -    | -     | value                  |
| -    | -    | vec  | -    | -     | vector                 |
| w    | wr   | -    | -    | -     | write                  |