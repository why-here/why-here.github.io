### 翻译自 [Fixed Point Arithmetic on the ARM](http://infocenter.arm.com/help/topic/com.arm.doc.dai0033a/DAI0033A_fixedpoint_appsnote.pdf)

### 介绍

本应用笔记介绍了如何使用ARM C编译器和ARM或Thumb汇编器编写高效的定点算术代码。 由于ARM内核是一个整数处理器，因此必须使用整数算术模拟所有浮点操作。 使用定点算法而不是浮点数将大大提高许多算法的性能。

本文档包含以下部分：

- 定点运算原理：描述了定点运算所需的数学概念。
- 例子：给出了为信号处理和图形处理编写定点代码的例子，这是两种最常见的用途。
- C编程：涵盖了在C中实现定点代码的实际细节。示例程序与一组宏定义一起给出。
- 汇编程序：介绍汇编程序示例。

### 定点算法原理

在计算算术中，可以使用一对整数（n，e）来近似表示分数：分别为尾数和指数。这对整数表示分数 $n2^{-e}$ 。指数 e 可以被认为是放置二进制小数点之前必须移动 n 的位数。比如

| Mantissa (n) | Exponent (e) |   Binary   | Decimal |
| :----------: | :----------: | :--------: | :-----: |
|   01100100   |      -1      | 011001000. |   200   |
|   01100100   |      0       | 01100100.  |   100   |
|   01100100   |      1       | 0110010.0  |   50    |
|   01100100   |      2       | 011001.00  |   25    |
|   01100100   |      3       | 01100.100  |  12.5   |
|   01100100   |      7       | 0.1100100  | 0.78125 |

如果 e 是一个变量，保存在寄存器中，并且在编译时未知，则（n，e）被称为浮点数。如果预先知道e，则在编译时，（n，e）被称为固定点数。 通过存储尾数，可以将固定点数存储在标准整数变量（或寄存器）中。

对于定点数，指数e通常用字母q表示。以下小节介绍如何对两个定点数执行基本的算术运算，两个算数为 $a=n2^{-p}$ 和 $b=m2^{-q}$ , 结果表示为 $c=k2^{-r}$ ,这里 p，q 和 r 是固定的常数指数。

在计算时，我们只需通过两个尾数 n m 计算得到 k 。其中会使用到他们的指数进行转换。实际存储数字时也只存储尾数。指数则由另一个对应的数字存储。

#### 指数变化

对定点数执行的最简单的操作是改变指数。 为了将指数从 p 改变为 r（执行操作c = a），可以通过简单的移位从 n 计算尾数 k 。

因为：$n2^{-p} =  n2^{r – p} * 2^{-r}$ 所以

$k = n << (r-p) $  $ if (r>=p)$

$k = n >> (p-r)$  $if (p>r)$

#### 加法和减法

要执行操作c = a + b，首先将a和b转换为与c具有相同的指数r，然后尾数相加。 该方法由等式如下：

$n2^{-r} + m2^{-r} = (n + m)2^{-r}$

减法类似

#### 乘法

乘积c = ab可以使用简单的整数乘法执行。 等式如下：

$ab = n2^{–p} * m2^{–q} =  (nm)2^{–(p + q)}$

因此乘积n * m是指数为p + q的答案的尾数。 要将答案转换为具有指数r，请执行如上所述的移位。比如

$k = (n*m) >> (p+q-r)$  $if (p + q >= r)$

#### 除法

除法 c=a/b 也可以通过简单的整数除法执行，公式是：

$a/b = (n2^{-p})/(m2^{-q}) = (n/m)2^{q-p}=(n/m)2^{r+q-p}2^{-r}$

为了不丢失精度，必须在除以m之前执行 $2^{r+q-p}$ 乘法运算。比如，

$k = (n<<(r+q-p))/m$  $if(r+q>=p)$

#### 平方根

平方根的公式如下：

$\sqrt{a} = \sqrt{n2^{-p}} = \sqrt{n2^{2r-p}}*2^{-r}$

换一种说法，$c=\sqrt{a} $ 执行 $k=isqr(n<<2r-p)$ 这里 isqr 是整数的平方根函数。

#### 溢出

上述等式说明如何仅使用整数运算对定点形式的数字执行加法，减法，乘法，除法和根提取的基本操作。但是，为了使答案精确地存储在（$\pm2^{-r}$）表示的结果中，您必须确保计算中不会发生溢出。如果指数没有被仔细选择，每一个左移，加/减或乘法都会产生溢出并导致无意义的答案。

以下“真实世界”情况的示例表明如何选择指数。

### 实例

#### 信号处理

在信号处理中，数据通常由分数值组成，范围为-1到+1。 此示例使用指数q和32位整数尾数（以便每个值可以保存在一个ARM寄存器中）。 为了能够将两个数字相乘而不会溢出，您需要2q<32或q<=15。 在实践中，通常选择q = 14，因为这允许与几个积累相乘而没有溢出风险。

在编译时固定q = 14。 然后0xFFFFC000表示-1，0x00004000表示+1和0x00000001表示$2^{-14}$，最精确的除数。

假设x，y是两个q = 14尾数，n，m是两个整数。 通过应用上面的公式，您可以得到基本操作，其中 a 为正常的整数：

| Operation  | Code to produce mantissa of answer in q=14 format |
| :--------: | :-----------------------------------------------: |
|   x + y    |                       x + y                       |
|   x + a    |                    x + (a<<14)                    |
|     xa     |                        x*a                        |
|     xy     |                     (x*y)>>14                     |
|    x/a     |                        x/a                        |
|    a/b     |                     (a<<14)/b                     |
|    x/y     |                     (x<<14)/y                     |
| $\sqrt{x}$ |                    sqr(x<<14)                     |

#### 图像处理

在这个例子中，（x，y，z）坐标以八位小数信息（q = 8）以小数形式存储。 如果x存储在一个32位寄存器中，那么x的最低字节给出小数部分，最高的三部分是整数部分。

要计算距离原点的距离d，以q = 8的形式：$d =\sqrt{x^2 +y^2 + z^2}$

如果您直接应用上述公式并将所有中间答案以q = 8的形式保存，则会得到以下代码：

```
x = (x*x)>>8      square x
y = (y*y)>>8      square y
z = (z*z)>>8      square z
s = x+y+z         sum of squares
d = sqrt(s<<8)    the distance in q=8 form
```

或者，如果您将中间答案保持为q = 16格式，则减少班次数并提高准确度：

```
x = x*x      square of x in q=16 form
y = y*y      square of y in q=16 form
z = z*z      square of z in q=16 form
s = x+y+x    sum of squares in q=16 form
d = sqr(s)   distance d in q=8 form
```

#### 总结

- 如果你以q形式相加两个数字，它们将保持q-form。
- 如果你用q形式乘以两个数字，答案是2q形式。
- 如果你采用q形式的数字的平方根，答案是q / 2形式。
- 要从q-form转换为r-form，你需要向左移（r-q）或向右移（q-r），取决于q和r中哪一个更大
- 要获得最佳精度结果，请选择最大的q，并保证中间计算不能溢出。

### C 程序

可以使用标准整数算术运算在C中对定点算术进行编程，并且当需要时（通常在确保答案仍然是q形式之前或之后的操作之前或之后）使用移位来改变q-形式。

为了使编程更容易阅读，已经定义了一组C宏。 下面的示例程序定义了这些宏并说明了它们的用法：

注：编译需要在末尾加上 -lm 选项 ，如 `gcc fixed.c -o fixed -lm`

```c
/* A Simple C program to illustrate the use of Fixed Point Operations */
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
/* DEFINE THE MACROS */
/* The basic operations perfomed on two numbers a and b of fixed
point q format returning the answer in q format */
#define FADD(a,b) ((a)+(b))
#define FSUB(a,b) ((a)-(b))
#define FMUL(a,b,q) (((a)*(b))>>(q))
#define FDIV(a,b,q) (((a)<<(q))/(b))
/* The basic operations where a is of fixed point q format and b is
an integer */
#define FADDI(a,b,q) ((a)+((b)<<(q)))
#define FSUBI(a,b,q) ((a)-((b)<<(q)))
#define FMULI(a,b) ((a)*(b))
#define FDIVI(a,b) ((a)/(b))
/* convert a from q1 format to q2 format */
#define FCONV(a, q1, q2) (((q2)>(q1)) ? (a)<<((q2)-(q1)) : (a)>>((q1)-(q2)))
/* the general operation between a in q1 format and b in q2 format
returning the result in q3 format */
#define FADDG(a,b,q1,q2,q3) (FCONV(a,q1,q3)+FCONV(b,q2,q3))
#define FSUBG(a,b,q1,q2,q3) (FCONV(a,q1,q3)-FCONV(b,q2,q3))
#define FMULG(a,b,q1,q2,q3) FCONV((a)*(b), (q1)+(q2), q3)
#define FDIVG(a,b,q1,q2,q3) (FCONV(a, q1, (q2)+(q3))/(b))
/* convert to and from floating point */
#define TOFIX(d, q) ((int)( (d)*(double)(1<<(q)) ))
#define TOFLT(a, q) ( (double)(a) / (double)(1<<(q)) )
#define TEST(FIX, FLT, STR) { \
  a = a1 = randint(); \
  b = bi = a2 = randint(); \
  fa = TOFLT(a, q); \
  fa1 = TOFLT(a1, q1); \
  fa2 = TOFLT(a2, q2); \
  fb = TOFLT(b, q); \
  ans1 = FIX; \
  ans2 = FLT; \
  printf("Testing %s\n fixed point answer=%f\n floating point answer=%f\n", \
  STR, TOFLT(ans1, q), ans2); \
}
int randint(void) {
  int i;
  i=rand();
  i = i & 32767;
  if (rand() & 1) i=-i;
  return i;
}
int main(void) {
  int q=14, q1=15, q2=7, q3=14;
  double fa, fb, fa1, fa2;
  int a,b,bi,a1,a2;
  int ans1;
  double ans2;
  /* test each of the MACRO's with some random data */
  TEST(FADD(a,b), fa+fb, "FADD");
  TEST(FSUB(a,b), fa-fb, "FSUB");
  TEST(FMUL(a,b,q), fa*fb, "FMUL");
  TEST(FDIV(a,b,q), fa/fb, "FDIV");
  TEST(FADDI(a,bi,q), fa+bi, "FADDI");
  TEST(FSUBI(a,bi,q), fa-bi, "FSUBI");
  TEST(FMULI(a,bi), fa*bi, "FSUBI");
  TEST(FDIVI(a,bi), fa/bi, "FSUBI");
  TEST(FADDG(a1,a2,q1,q2,q3), fa1+fa2, "FADDG");
  TEST(FSUBG(a1,a2,q1,q2,q3), fa1-fa2, "FSUBG");
  TEST(FMULG(a1,a2,q1,q2,q3), fa1*fa2, "FMULG");
  TEST(FDIVG(a1,a2,q1,q2,q3), fa1/fa2, "FDIVG");
  printf("Finished standard test\n");
  /* the following code will calculate exp(x) by summing the
  series (not efficient but useful as an example) and compare it
  with the actual value */
  while (1) {
    printf("Please enter the number to be exp'ed (<1): ");
    scanf("%lf", &fa);
    printf(" x = %f\n", fa);
    printf(" exp(x) = %f\n", exp(fa));
    q = 14; /* set the fixed point */
    a = TOFIX(fa, q); /* get a in fixed point format */
    a1 = FCONV(1, 0, q); /* a to power 0 */
    a2 = 1; /* 0 factorial */
    ans1 =0; /* series sum so far */
    for (bi=0 ; bi<10; bi++) { /* do ten terms of the series */
      int j;
      j = FDIVI(a1, a2); /* a^n/n! */
      ans1 = FADD(ans1, j);
      a1 = FMUL(a1, a, q); /* increase power of a by 1 */
      a2 = a2*(bi+1); /* next factorial */
      printf("Stage %d answer = %f\n", bi, TOFLT(ans1, q));
    }
  }
  return 0;
}
```

### 使用汇编编程

