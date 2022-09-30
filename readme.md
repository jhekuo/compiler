# 2021编译技术实验记录

## 简介

SysY 语言是本次课程要实现的编程语言，是 C 语言的一个子集。每个 SysY 程序的源码存储在一个扩展名为 sy 的文件中。该文件中有且仅有一个名为 main 的主函数定义，还可以包含若干全局变量声明、常量声明和其他函数定义。SysY 语言支持 int 类型和元素为 int 类型且按行优先存储的多维数组类型，其中 int 型整数为32位有符号数；const 修饰符用于声明常量。

SysY 语⾔通过`getint`与`printf`函数完成 IO 交互，函数⽤法已在⽂法中给出，需要同学们⾃⼰实现。

- **函数**：函数可以带参数也可以不带参数，参数的类型可以是 int 或者数组类型；函数可以返回 int 类型的值，或者不返回值(即声明为void类型)。当参数为 int 时，按值传递；⽽参数为数组类型时，实际传递的是数组的起始地址，并且形参只有第⼀维的⻓度可以空缺。函数体由若⼲变量声明和语句组成。 
- **变量/常量声明**：可以在⼀个变量/常量声明语句中声明多个变量或常量，声明时可以带初始化表达式。所有变量/常量要求先定义再使⽤。在函数外声明的为全局变量/常量，在函数内声明的为局部变量/常量。 
- **语句**：语句包括赋值语句、表达式语句(表达式可以为空)、语句块、if 语句、while 语句、break 语句、continue 语句、return 语句。语句块中可以包含若⼲变量声明和语句。 
- **表达式**：⽀持基本的算术运算(`+、-、*、/、%`)、关系运算(`==、!=、<、>、<=、>=`)和逻辑运 算(`!、&&、||`)，⾮ 0 表示真、0 表示假，⽽关系运算或逻辑运算的结果⽤ 1 表示真、0 表示假。算符的优先级和结合性以及计算规则(含逻辑运算的“短路计算”)与 C 语⾔⼀致。


## 文法

详细文法见[![1](https://img.shields.io/badge/repo-miniSysY-9cf?logo=github)](https://github.com/BUAA-SE-Compiling/miniSysY-tutorial/blob/master/miniSysY.md)  [![2](https://img.shields.io/badge/pdf-%E6%96%87%E6%B3%95%E5%AE%9A%E4%B9%89%E8%AF%B4%E6%98%8E-9cf?logo=gitbook)](https://github.com/imingx/Compiler/blob/main/docs/2021%E7%BC%96%E8%AF%91%E6%8A%80%E6%9C%AF%E5%AE%9E%E9%AA%8C%E6%96%87%E6%B3%95%E5%AE%9A%E4%B9%89%E5%8F%8A%E7%9B%B8%E5%85%B3%E8%AF%B4%E6%98%8E.pdf)

```
  编译单元 CompUnit → {Decl} {FuncDef} MainFuncDef // 1.是否存在Decl 2.是否存在FuncDef

  声明 Decl → ConstDecl | VarDecl // 覆盖两种声明

  常量声明 ConstDecl → 'const' BType ConstDef { ',' ConstDef } ';' // 1.花括号内重复0次 2.花括号内重复多次

  基本类型 BType → 'int' // 存在即可

  常数定义 ConstDef → Ident { '[' ConstExp ']' } '=' ConstInitVal // 包含普通变量、一维数组、二维数组共三种情况

  常量初值 ConstInitVal → ConstExp
    | '{' [ ConstInitVal { ',' ConstInitVal } ] '}' // 1.常表达式初值 2.一维数组初值 3.二维数组初值

  变量声明 VarDecl → BType VarDef { ',' VarDef } ';' // 1.花括号内重复0次 2.花括号内重复多次

  变量定义 VarDef → Ident { '[' ConstExp ']' } // 包含普通变量、一维数组、二维数组定义
    | Ident { '[' ConstExp ']' } '=' InitVal

  变量初值 InitVal → Exp | '{' [ InitVal { ',' InitVal } ] '}'// 1.表达式初值 2.一维数组初值 3.二维数组初值

  函数定义 FuncDef → FuncType Ident '(' [FuncFParams] ')' Block // 1.无形参 2.有形参

  主函数定义 MainFuncDef → 'int' 'main' '(' ')' Block // 存在main函数

  函数类型 FuncType → 'void' | 'int' // 覆盖两种类型的函数 

  函数形参表 FuncFParams → FuncFParam { ',' FuncFParam } // 1.花括号内重复0次 2.花括号内重复多次

  函数形参 FuncFParam → BType Ident ['[' ']' { '[' ConstExp ']' }] // 1.普通变量 2.一维数组变量 3.二维数组变量

  语句块 Block → '{' { BlockItem } '}' // 1.花括号内重复0次 2.花括号内重复多次

  语句块项 BlockItem → Decl | Stmt // 覆盖两种语句块项

  语句 Stmt → LVal '=' Exp ';' // 每种类型的语句都要覆盖
    | [Exp] ';' //有无Exp两种情况
    | Block 
    | 'if' '( Cond ')' Stmt [ 'else' Stmt ] // 1.有else 2.无else
    | 'while' '(' Cond ')' Stmt
    | 'break' ';' | 'continue' ';'
    | 'return' [Exp] ';' // 1.有Exp 2.无Exp
    | LVal = 'getint''('')'';'
    | 'printf' '('FormatString {',' Exp} ')'';' // 1.有Exp 2.无Exp

  表达式 Exp → AddExp 注：SysY 表达式是int 型表达式 // 存在即可

  条件表达式 Cond → LOrExp // 存在即可

  左值表达式 LVal → Ident {'[' Exp ']'} //1.普通变量 2.一维数组 3.二维数组

  基本表达式 PrimaryExp → '(' Exp ')' | LVal | Number // 三种情况均需覆盖

  数值 Number → IntConst // 存在即可

  一元表达式 UnaryExp → PrimaryExp | Ident '(' [FuncRParams] ')' // 三种情况均需覆盖,函数调用也需要覆盖FuncRParams的不同情况
    | UnaryOp UnaryExp // 存在即可

  单目运算符 UnaryOp → '+' | '−' | '!' 注：'!'仅出现在条件表达式中 // 三种均需覆盖

  函数实参表 FuncRParams → Exp { ',' Exp } // 1.花括号内重复0次 2.花括号内重复多次 3. Exp需要覆盖数组传参和部分数组传参

  乘除模表达式 MulExp → UnaryExp | MulExp ('*' | '/' | '%') UnaryExp // 1.UnaryExp 2.* 3./ 4.% 均需覆盖

  加减表达式 AddExp → MulExp | AddExp ('+' | '−') MulExp // 1.MulExp 2.+ 3.- 均需覆盖

  关系表达式 RelExp → AddExp | RelExp ('<' | '>' | '<=' | '>=') AddExp // 1.AddExp 2.< 3.> 4.<= 5.>= 均需覆盖

  相等性表达式 EqExp → RelExp | EqExp ('==' | '!=') RelExp // 1.RelExp 2.== 3.!= 均需覆盖

  逻辑与表达式 LAndExp → EqExp | LAndExp '&&' EqExp // 1.EqExp 2.&& 均需覆盖

  逻辑或表达式 LOrExp → LAndExp | LOrExp '||' LAndExp // 1.LAndExp 2.|| 均需覆盖

  常量表达式 ConstExp → AddExp 注：使用的Ident 必须是常量 // 存在即可

  格式化字符 FormatChar → %d

  普通字符 NormalChar → 十进制编码为32,33,40-126的ASCII字符

  字符 Char → FormatChar | NormalChar 

  格式化字符串 FormatString → '"'{ Char }'"'
```

## 测试样例

```
const int array[2] = {1,2};

int main(){
    int c;
    c = getint();
    printf("output is %d, %d",c, array[0]);
    return 0;
}
```

```
// {Decl}------------------------------
int g_a = 1613;
int g_b, g_c;
int g_arr_a[2], g_arr_b[2][2];
int g_arr_c[2] = {1, 2};
int g_arr_d[2][2] = {{1, 2}, {1, 2}};
const int const_a = 1613;
const int const_b = 1613, const_c = 1613;
const int const_arr_a[2] = {1, 2};
const int const_arr_b[2][2] =  {{1, 2}, {1, 2}};

// {ConstExp}--------------------------
const int const_d = 2 + 2 - 2;
const int const_e = 3 * 3 / 3 % 3;
const int const_f = -(4);
const int const_g = +(5);
const int const_h = const_a;

// {FuncDef}---------------------------
void func_1(){return;}
void func_2(int a){int b;b = a;return;}
void func_3(int a, int b){return;}
void func_4(int arr1[], int arr2[][2], int arr3[ ]){
    arr1[1] = 1;
    arr2[1][1] = 1;
    return;
}

int func_5(){return 1;}

// {Cond}---------------------------
void func_6(int a, int b){
    func_2(a);
    func_3(a, b);
    func_4(g_arr_c, g_arr_d, g_arr_d[0]);
    if(a == b) {return;}
    if(a != b);
    if(a > b);
    if(a < b);
    if(a <= b);
    if(a >= b);
    if(!a);
    return;
}

int main()
{
    int n, i;
    //scanf("%d", &n);
    n = getint();
    i = 1;
    1;;{}

    if( i >= 1) i = 1;
    if( i < 1)  i = 1; 
    else i = 4;

    while( i > 1)
    {
        i = i - 1;
        if(i > 1)
            continue;
        else
            break;
    }
    // {printf}--------------------------
    printf("19373136\n");
    printf("i is %d, n is %d\n", i, n);
    printf("! ()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[]^_`abcdefghijklmnopqrstuvwxyz{|}~\n");
    return 0;
}
```

## 步骤

- [00\_文法解读][00_文法解读]
    - [文法解读测试用例][文法解读测试用例]

- [01\_词法分析][01_词法分析]

- [02\_语法分析][02_语法分析]
    - [语法分析测试用例][语法分析测试用例]

- [03\_错误处理][03_错误处理]
    - [错误处理测试用例][错误处理测试用例] 

- [期中考试][期中考试]


## 提交信息

```
(1) type:
feat		增加特性
fix	        修复错误
docs		修改文档
style		更改空格、缩进、样式，不改变代码逻辑
refactor	代码重构
perf		优化相关，提升性能和体验
test		增加测试用例
build		依赖相关的内容
ci              ci配置相关，对k8s, docker的配置文件修改
chore		改变构建流程、增加依赖库和工具
revert		回滚版本

(2) scope: commit影响的范围
(3) subject: commit目的的简短描述，以第一人称现在时，小写字母开头，不加句号

type(scope): subject

some useful template:

docs(readme): 增加内容/improve content
🎉init commit
增加项目文件用feat
random message: "`curl -s http://whatthecommit.com/index.txt`"

git commit的message可以换行，在输入第一个"后，输入回车并不会直接执行命令。
在第二行输入的message称为body，最后还有foot。
```

## 参考

1. [`unique_ptr`][unique_ptr], [`shared_ptr`][share_ptr]
2. [TryC - a small interpreter written by C][tryC - a small interpreter written by C]
3. [My First Language Frontend with LLVM Tutorial][My First Language Frontend with LLVM Tutorial]




[期中考试]: https://github.com/imingx/Compiler/tree/midtermExam "期中考试"

[share_ptr]: http://www.cplusplus.com/reference/memory/shared_ptr/ "share_ptr"
[00_文法解读]: https://github.com/imingx/Compiler/tree/00_%E6%96%87%E6%B3%95%E8%A7%A3%E8%AF%BB "00_文法解读"
[01_词法分析]: https://github.com/imingx/Compiler/tree/01_Lexer	"01_词法分析"
[02_语法分析]: https://github.com/imingx/Compiler/tree/02_Parser  "02_语法分析"
[文法解读测试用例]: https://github.com/imingx/Compiler/tree/00_%E6%96%87%E6%B3%95%E8%A7%A3%E8%AF%BB_testFile "文法解读测试用例"
[语法分析测试用例]: https://github.com/imingx/Compiler/tree/02_Parser_testFile "语法分析测试用例"
[03_错误处理]: https://github.com/imingx/Compiler/tree/03_ErrorHandler "03_错误处理"
[错误处理测试用例]: https://github.com/imingx/Compiler/tree/03_ErrorHandler_testFile "错误处理测试用例"
[unique_ptr]: https://blog.csdn.net/shaosunrise/article/details/85158249 "unique_ptr"
[My First Language Frontend with LLVM Tutorial]: https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html "My First Language Frontend with LLVM Tutorial"
[tryC - a small interpreter written by C]: https://github.com/imingx/tryC "tryC - a small interpreter written by C"
