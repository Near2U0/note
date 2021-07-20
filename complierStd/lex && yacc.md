[toc]

# lex语法



## example

这段lex程序的目的是：输入几行字符串，输出行数，单词数和字符的个数

```c

/*******************************************
* Name        : test.l
* Date        : Mar. 11, 2014
* Blog        : http://www.cnblogs.com/lucasysfeng/
* Description : 一个简单的lex例子,输入几行字符串，
*               输出行数，单词数和字符的个数。
*******************************************/
 

/* 第一段 */ 
%{
    int chars = 0;
    int words = 0;
    int lines = 0;
%}

/* 第二段 */  
%%
[a-zA-Z]+  { words++; chars += strlen(yytext); }
\n         { chars++; lines++; }
.          { chars++; }
%%

/* 第三段 */  
main(int argc, char **argv)
{
    yylex();
    printf("%8d%8d%8d\n", lines, words, chars);
}

```

## 编译运行

```shell
#flex test.l

#gcc lex.yy.c –lfl

#./a.out

#然后输入一段文字，按ctrl+d结束输入，则会输出行数，单词数和字符的个数。
```



![](..\images\compileStud\lex_1.jpg)

## lex语法格式说明

1. %%把文件分为3段，第一段是c和lex的全局声明，第二段是规则段，第三段是c代码。
2. 第一段的c代码要用%{和%}括起来，第三段的c代码不用。
3. 第二段规则段，`[a-zA-Z]+ \n`  . 是正则表达式，{}内的是c编写的动作。





# yacc的语法

## BNF范式（巴科斯范式）



BNF是John Backus 在20世纪90年代提出的用以简洁描述一种编程语言的语言。

基本结构为：

```SHELL
<non-terminal> ::= <replacement>
```

non-terminal意为非终止符，就是说我们还没有定义完的东西，还可以继续由右边的replacement，也就是代替物来进一步解释、定义。

举个例子：

在中文语法里，一个句子一般由“主语”、“谓语”和“宾语”组成，主语可以是名词或者代词，谓语一般是动词，宾语可以使形容词，名词或者代词。那么“主语”、“谓语”和“宾语”就是非终止符，因为还可以继续由“名词”、“代词”、“动词”、“形容词”等替代。

```shell
例1. <句子> ::= <主语><谓语><宾语>

例2. <主语> ::= <名词>|<代词>

例3. <谓语>::=<动词>

例4. <宾语>::=<形容词>|<名词>|<代词>

例5. <代词>::=<我>

例6. <动词>::=<吃>

例7. <动词>::=<喜欢>

例8. <名词>::=<车>

例9. <名词>::=<肉>
```

如上，在::=左边的就是non-terminal非终止符，右边的就是replacement，可以是一系列的非终止符，如例1中的replacement便是后面例234左边的非终止符，也可以是终止符，如例56789的右边，找不到别的符号来进一步代替。

因此，终止符永远不会出现在左边。一旦我们看到了终止符，这个描述过程就结束了。



## example

xx.l 文件

```c
%{
#include <stdio.h>
#include "y.tab.h"
%}
%%
[0-9]+                  return NUMBER;
heat                    return TOKHEAT;
on|off                  return STATE;
target                  return TOKTARGET;
temperature             return TOKTEMPERATURE;
\n                      /* ignore end of line */;
[ \t]+                  /* ignore whitespace */;
%%
```

这个l文件主要是参数y文件定义的各种token，大家可以看到它的subroutines部分为空，因为该词法分析器的结果直接输出到语法分析器，因此不需要额外的函数。**下面的y文件都依赖于该l文件**。 一个y文件(语法文件)同样包含definitions、rules、subroutines三个部分，每部分同样通过双百分号(%%)分割。各个部分的作用l文件的对应部分也基本一致。 一个简单的y文件例子，test.y。

test.y 文件

```c
%{
#include <stdio.h>
#include <string.h>
void yyerror(const char *str);
%}
%token NUMBER TOKHEAT STATE TOKTARGET TOKTEMPERATURE
%% 
commands: /* empty */
        | commands command
        ;

command:					/*command 可以由heat_swith 或者 target_set组成*/
        heat_switch
        |
        target_set
        ;

heat_switch:			/* 最终的终止符， heat_switch 可以由token （TOKHEAT STATE）组成 */
        TOKHEAT STATE
        {
                printf("\tHeat turned on or off\n");
        }
        ;

target_set:			/* 最终的终止符*/
        TOKTARGET TOKTEMPERATURE NUMBER
        {
                printf("\tTemperature set\n");
        }
        ;
%%
void yyerror(const char *str)
{
        fprintf(stderr,"error: %s\n",str);
}
int yywrap()
{
        return 1;
}  
main()
{
        yyparse();
} 

```

该y文件的definitions部分声明了一个函数，并定义了一系列标记(TOKEN)。然后在rules部分定义了四个模式序列对应(语句)的动作，其中commands是一个递归定义。最后在subroutines部分定义了一个c语言main函数,读取文件，并实现yywrap并返回1表示停止解析。这个y文件实现了以下功能 ``` 输入：heat on 输出：Heat turned on or off 输入：target temperature 22 输出：New temperature set! ``` ###3.lex与yacc结合 也许你已经注意到了，l文件的definitions部分往往要包含#include "y.tab.h"。而y.tab.h是yacc对y文件编译后产生的c源文见。因此y文件必须限于l文件进行编译成c源文件，然后将l文件产生的c文件和y文件产生的c文件编译连接生产语法解析器。具体步骤见图：

![img](..\images\compileStud\lex_yacc.jpg)








参考：

https://www.cnblogs.com/qiumingcheng/p/14628334.html

http://foio.github.io/lex-yacc/

BNF:https://www.zhihu.com/question/27051306



yacc:https://blog.csdn.net/xiaowei_cqu/article/details/7764913



