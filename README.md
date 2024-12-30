
# 1 前言


​ 本文将介绍 GLSL 中数据类型、数组、结构体、宏、运算符、向量运算、矩阵运算、函数、流程控制、精度限定符、变量限定符（in、out、inout）、函数参数限定符等内容，另外提供了一个 include 工具，方便多文件管理 glsl 代码，实现代码的精简、复用。


​ Unity 中 Shader 介绍详见 → [【Unity3D】Shader常量、变量、结构体、函数](https://github.com)，渲染管线介绍详见 → [【OpenGL ES】渲染管线](https://github.com):[FlowerCloud机场节点订阅](https://dahelaoshi.com)。


# 2 数据类型


## 2\.1 基本数据类型


### 2\.1\.1 基本数据类型




| 类型 | 说明 | 案例 |
| --- | --- | --- |
| **void** | 空类型，即不返回任何值 | void fun() |
| **bool** | 布尔类型 true、false | bool a \= true; |
| **int** | 带符号的整数 | int a \= 0; |
| **float** | 带符号的浮点数 | float a \= 1\.0; float b \= 2\.; |
| **vec2、vec3、vec4** | 2 维、3 维、4 维浮点数向量 | vec2 a \= vec2(1\., 2\.); vec2 b \= vec2(1\.); // ⇔ vec2(1\., 1\.) vec3 c \= vec3(a, 3\.); // ⇔ vec3(1\., 2\., 3\.) |
| **bvec2、bvec3、bvec4** | 2 维、3 维、4 维布尔向量 | bvec2 a \= bvec2(true, false); bvec2 b \= bvec2(true); // ⇔ bvec2(true, true) bec3 c \= bvec3(a, true); // ⇔ bvec3(true, false, true) |
| **ivec2、ivec3、ivec4** | 2 维、3 维、4 维整数向量 | ivec2 a \= ivec2(1, 2\); ivec2 b \= ivec2(1\); // ⇔ ivec2(1, 1\) ivec3 c \= ivec3(a, 3\); // ⇔ ivec3(1, 2, 3\) |
| **mat2、mat3、mat4** | 2x2、3x3、4x4 浮点数矩阵 (列向量) | mat2 a \= mat2(1\., 2\., 3\., 4\.); mat2 b \= mat2(1\.); // ⇔ mat2(1\., 1\., 1\., 1\.) |
| **sampler2D** | 2D 纹理 | sampler2D sampler; |
| **samplerCube** | 盒纹理 | samplerCube sampler; |


​ 说明：mat2、mat3、mat4 中的元素都是按照列向量的顺序排列的，即 mat2 m \= mat2(m11, m21, m12, m22\) 对应的公式如下。


![img](https://img2024.cnblogs.com/blog/3135663/202412/3135663-20241229154325404-1402468398.png)


### 2\.1\.2 向量分量访问


​ GLSL 中的向量（vec2、vec3、vec4）可以表示一个空间坐标 (x, y, z, w)，也可以表示一个颜色 (r, g, b, a)，还可以表示一个纹理坐标 (s, t ,p, q)，所以 GLSL 提供了多样的分量访问方式。



```
vec4 v = vec4(1.0, 2.0, 3.0, 4.0);
float x1 = v.x; // 1.0
float x2 = v.r; // 1.0
float x3 = v.s; // 1.0
float x4 = v[0]; // 1.0

vec3 xyz = v.xyz; // vec3(1.0, 2.0, 3.0)
vec3 stq = v.stq; // vec3(1.0, 2.0, 3.0)
vec3 rgb = v.rgb; // vec3(1.0, 2.0, 3.0)
vec3 abc = vec3(v[0], v[1], v[2]); // vec3(1.0, 2.0, 3.0)

```

### 2\.1\.3 数据类型转换


​ GLSL 可以使用构造函数进行显式类型转换。



```
// 0或0.0转换为false, 非0转换为true
bool a1 = bool(1.0); // true
bool a2 = bool(0); // false

// true转换为1或1.0, false转换为0或0.0
int a3 = int(true); // 1
float a4 = float(false); // 0.0

int a5 = int(2.0); // 2
float a6 = float(1); // 1.0

```

## 2\.2 数组


​ GLSL 只支持一维数组。



```
// float 数组
float[3] a = float[] (1.0, 2.0, 3.0);
float b[3] = float[] (1.0, 2.0, 3.0);
float c[3] = float[3] (1.0, 2.0, 3.0);

// vec 数组
vec2[2] d = vec2[] (vec2(0.0), vec2(1.0));

```

## 2\.3 结构体



```
struct light {
    vec4 color;
    vec3 pos;
};
const light lgt = light(vec4(1.0), vec3(0.0));

```

​ 说明：结构体中的字段不可用 const 修饰。


## 2\.4 内置变量




| **范围** | **变量** | **说明** |
| --- | --- | --- |
| 顶点着色器的 Output 变量 | highp vec4 **gl\_Position**; | 顶点坐标信息 |
| 顶点着色器的 Output 变量 | mediump float **gl\_PointSize**; | 顶点大小 (只在 GL\_POINTS 图元模式下有效) |
| 片元着色器的 Input 变量 | mediump vec4 **gl\_FragCoord**; | 片元在屏幕空间的坐标，假设屏幕宽高分别为 width、height x: 片元的x坐标，值域 \[0, width \- 1] y: 片元的x坐标，值域 \[0, height \- 1] z: 片元的深度坐标，值域 \[0, 1] w: 总是 1，通常用于透视除法 |
| 片元着色器的 Input 变量 | bool **gl\_FrontFacing**; | 标志当前图元是否是正面图元的一部分 |
| 片元着色器的 Output 变量 | mediump vec4 **gl\_FragColor**; | 设置当前片点的颜色 |


## 2\.5 宏


​ 与 C 语言一样，GLSL 中也可以通过 \#define 定义宏，如下。



```
#define PI 3.14159265359

```

​ 另外，GLSL 也提供了一些内置宏。



```
__LINE__ // 当前源码中的行号
__VERSION__ // 当前glsl版本号, 如: 300
GL_ES // 当前运行环境是否是 OPGL ES, 1 表示是
GL_FRAGMENT_PRECISION_HIGH // 当前系统的片元着色器是否支持高浮点精度, 1表示支持

```

# 3 运算符


## 3\.1 基础运算符




| 优先级 (越小越高) | 运算符 | 说明 | 结合性 |
| --- | --- | --- | --- |
| 1 | **()** | 聚组: a \* (b \+ c) | N/A |
| 2 | **\[]** **()** **.** **\+\+ \-\-** | 数组下标 方法参数: fun(arg1, arg2\) 属性访问 自增 (a\+\+) / 自减 (a\-\-) | L \- R |
| 3 | **\+\+ \-\-** **\+ \-** **!** | 自增 (\+\+a) / 自减 (\-\-a) 正 (\+a) 负 (\-a) 号 取反 (!a) | R \- L |
| 4 | **\*** **/** **%** | 乘法 / 除法运算 整数求余运算 (浮点数求余用 mod 函数) | L \- R |
| 5 | **\+ \-** | 加法 / 减法运算 | L \- R |
| 7 | **\< \> \<\= \>\=** | 关系运算符 | L \- R |
| 8 | **\=\= !\=** | 相等性运算符 | L \- R |
| 12 | **\&\&** | 逻辑与 | L \- R |
| 13 | **^^** | 逻辑排他或 (用处基本等于 !\=) | L \- R |
| 14 | **\|\|** | 逻辑或 | L \- R |
| 15 | **? :** | 三目运算符 | L \- R |
| 16 | **\= \+\= \-\= \*\= /\=** | 赋值和复合赋值 | L \- R |
| 17 | **,** | 顺序分配运算 | L \- R |


​ 说明：GLSL 中没有隐式类型转换，因此任何表达式左右两侧的类型必须一致，以下表达式都是错误的。



```
// 以下代码运行时会报错
int a = 2.;
int b = 1. + 2;
float c = 2;
float d = 2. + 1;
bool e = 0;
vec2 f = vec2(1., 2.) * 2;

```

## 3\.2 向量运算符



```
// 标量与向量运算
vec2 a = vec2(1., 2.) + 3.; // vec2(4., 5.)
vec2 b = 3. + vec2(1., 2.); // vec2(4., 5.)
vec2 c = vec2(1., 2.) * 3.; // vec2(3., 6.)
vec2 d = 3. * vec2(1., 2.); // vec2(3., 6.)

// 向量与向量运算
vec2 e = vec2(1., 2.) + vec2(3., 4.); // vec2(4., 6.)
vec2 f = vec2(1., 2.) * vec2(3., 4.); // vec2(3., 8.)

```

## 3\.3 矩阵运算符



```
// 标量与矩阵运算
mat2 a = mat2(1.) + 2.; // mat2(3.)
mat2 b = 2. + mat2(1.); // mat2(3.)
mat2 c = mat2(1.) * 2.; // mat2(2.)
mat2 d = 2. * mat2(1.); // mat2(2.)

// 向量与矩阵运算
vec2 e = vec2(1., 2.) * mat2(1., 2., 3., 4.); // vec2(5., 11.)
vec2 f = mat2(1., 2., 3., 4.) * vec2(1., 2.); // vec2(7., 10.)

// 矩阵与矩阵运算(矩阵对应元素运算)
mat2 g = mat2(1.) + mat2(2.); // mat2(3.)
mat2 h = matrixCompMult(mat2(1.), mat2(2.)); // mat2(2.)

// 矩阵与矩阵运算(矩阵乘法)
mat2 i = mat2(1., 2., 3., 4.) * mat2(5., 6., 7., 8.); // mat2(23., 34., 31., 46.)
mat2 j = mat2(5., 6., 7., 8.) * mat2(1., 2., 3., 4.); // mat2(19., 22., 43., 50.)

```

![img](https://img2024.cnblogs.com/blog/3135663/202412/3135663-20241229154325388-1247803515.png)


# 4 函数


## 4\.1 自定义函数


​ GLSL 允许在程序的最外部声明函数，函数不能嵌套、不能递归调用，且必须声明返回值类型（无返回值时声明为 void）在其他方面 GLSL 函数与 C 语言函数非常类似。



```
vec4 getPosition() { 
    vec4 pos = vec4(0.,0.,0.,1.);
    return pos;
}

void doubleSize(inout float size) {
    size = size * 2.0;
}

```

## 4\.2 内置函数


​ **1）数值运算**



```
sign(x)、abs(x) // 符号、绝对值
min(a, b)、max(a, b) // 最值函数
ceil(x)、floor(x)、round(x) // 取整函数
fract(x) // 取小数部分
mod(x, y) // 取余数
sqrt(x)、pow(x)、inversesqrt(x) // 幂函数, inversesqrt(x)=1/sqrt(x)
exp(x)、exp2(x) // 指数函数(e^x、2^x)
log(x)、log2(x) // 对数函数
degrees(x)、radians(x) // 角度转换函数
sin(x)、cos(x)、tan(x)、asin(x)、acos(x)、atan(x) // 三角函数
sinh(x)、cosh(x)、tanh(x) // 双曲线函数
clamp(x, min, max) // 将x约束在min和max之间, 超过边界就取边界值
smoothstep(min, max, x) // 平滑比例, 公式: k=saturate((x-min)/(max-min)), y=k*k*(3-2*k)
mix(a, b, f) // 混合, 公式: y=(1-f)*a+f*b
step(a, b) // 如果a>b, 返回0; 如果a<=b, 返回1; 当a、b是向量时, 每个分量独立判断, 如: step(fixed2(1,1),fixed(0,2))=(0,1)

```

​ 说明：以上函数输入的可以是：float、vec2、vec3、vec4，且可以逐分量操作；对于整数求余运算，只能用 %；对于浮点数求余运算，只能用 mod。


​ **2）逻辑运算**



```
bvec z = lessThan(x, y) // 逐分量比较x < y, 将结果写入z的对应位置
bvec z = lessThanEqual(x, y) // 逐分量比较x <= y, 将结果写入z的对应位置
bvec z = greaterThan(x, y) // 逐分量比较x > y, 将结果写入z的对应位置
bvec z = greaterThanEqual(x, y) // 逐分量比较x >= y, 将结果写入z的对应位置
bvec z = equal(x, y) // 逐分量比较x == y, 将结果写入z的对应位置
bvec z = notEqual(x, y) // 逐分量比较x != y, 将结果写入z的对应位置
bvec y = not(x) // bool矢量的逐分量取反
bool y = any(x) // 如果x的任意一个分量是true, 则结果为true
bool y = all(x) // 如果x的所有分量是true, 则结果为true

```

​ **3）向量运算**



```
distance(pos1, pos2) // 计算pos1与pos2之间的距离
length(vec) // 计算向量的模长
normalize(vec) // 计算向量的单位向量
dot(v1, v2) // 向量点乘
cross(v1, v2) // 向量叉乘
reflect(i, n) // 根据入射向量和法线向量, 计算反射向量(i和n不需要归一化)
refract(i, n, ratio); // 根据入射向量、法线向量、折射率比值, 计算折射向量(i和n需要归一化, ratio为入射介质折射率/折射介质折射率, 或sin(折射角)/sin(入射角))

```

​ **4）矩阵运算**



```
// 矩阵与矩阵运算(矩阵对应元素运算)
mat2 a = mat2(1.) + mat2(2.); // mat2(3.)
mat2 b = matrixCompMult(mat2(1.), mat2(2.)); // mat2(2.)

// 矩阵与矩阵运算(矩阵乘法)
mat2 c = mat2(1., 2., 3., 4.) * mat2(5., 6., 7., 8.); // mat2(23., 34., 31., 46.)
mat2 d = mat2(5., 6., 7., 8.) * mat2(1., 2., 3., 4.); // mat2(19., 22., 43., 50.)

```

![img](https://img2024.cnblogs.com/blog/3135663/202412/3135663-20241229154325372-229745723.png)


​ **5）纹理查询函数**



```
vec4 texture(sampler2D sampler, vec2 coord);
vec4 texture2D(sampler2D sampler, vec2 coord);
vec4 texture2DProj(sampler2D sampler, vec3 coord);
vec4 texture2DProj(sampler2D sampler, vec4 coord);
vec4 textureCube(samplerCube sampler, vec3 coord);

```

# 5 流程控制


​ GLSL 的流控制与 C 语言非常相似，主要有 if、for、while、continue、break，不同的是 GLSL 中多了 discard。使用 discard 会退出片元着色器，不会执行后面的操作，片元也不会写入帧缓冲区。



```
for (int i = 0; i < 10; i++) {
    sum += a[i];
    if (sum > 5.0)
        break;
}

while (i < 10) {
    sum += a[i];
    if (i % 3 == 1)
        continue;
    i++;
}

do {
    sum += a[i];
    if (sum > 5.0)
        discard;
    i++;
} while (i < 10)

```

# 6 限定符


## 6\.1 精度限定符


​ 片元着色器中，对于浮点数 GLSL 有 highp（高）、mediump（中）、lowp（低） 三种精度。



```
lowp float color;
varying mediump vec2 Coord;
lowp ivec2 foo(lowp mat3);
highp mat4 m;

```

​ 在片元着色器的第一行加上 precision highp float，表示设定了默认的精度，所有没有显式表明精度的变量都会按照默认精度处理。



```
precision highp float;

```

​ 通过判断系统环境，来选择合适的精度。



```
#ifdef GL_ES
#ifdef GL_FRAGMENT_PRECISION_HIGH
precision highp float;
#else
precision mediump float;
#endif
#endif

```

## 6\.2 变量限定符




| 限定符 | 说明 |
| --- | --- |
| **none** | 默认的变量限定符，可省略，可读写。 |
| **const** | 修饰的变量为只读类型，变量在定义时必须初始化。 |
| **attribute** | 只能在顶点着色器中使用，修饰全局只读变量，一般用于修饰顶点属性，如：顶点的位置、法线、纹理坐标、颜色等。 |
| **uniform** | 修饰全局只读变量，一般用于修饰程序传递给 shader 的变量，如：屏幕宽高比、光源位置等。 |
| **varying** | 修饰需要进行光栅化插值的变量，在片元着色器中是只读的，如：纹理坐标等。 |



```
// 顶点着色器
attribute vec4 a_position;
attribute vec2 a_texCoord0;
varying vec2 v_texCoord0;

void main() {
    gl_Position = vec4(a_position, 1.0);
    v_texCoord0 = a_texCoord0;
}

// 片元着色器
precision highp float;
uniform sampler2D u_texture;
varying vec2 v_texCoord0;

void main() {
  gl_FragColor = texture(u_texture, v_texCoord0);
}

```

## 6\.3 函数参数限定符


​ 函数的参数默认以拷贝的形式传递，即值传递，我们可以为参数添加限定符实现引用传递，GLSL 中提供的参数限定符如下。




| 限定符 | 说明 |
| --- | --- |
| in | 默认的限定符，参数是值传递，在函数中可读写。 |
| out | 参数是引用传递，在函数中只能写（write\-only）。 |
| inout | 参数是引用传递，在函数中可读写（read\-write）。 |


​ 除了函数的参数可以用 in、out、inout 修饰，全局变量也可以用这些限定符修饰，如下。



```
// 顶点着色器
in vec3 a_position;
in vec2 a_texCoord0;
out vec2 v_texCoord0;

void main() {
    gl_Position = vec4(a_position, 1.0);
    v_texCoord0 = a_texCoord0;
}

// 片元着色器
precision highp float;
uniform sampler2D u_texture;
in vec2 v_texCoord0;
out vec4 o_fragColor;

void main() {
    o_fragColor = texture(u_texture, v_texCoord0);
}

```

# 7 include


​ GLSL 中没有提供 \#include 功能，如以下代码会运行报错。



```
#include 

```

​ 要想实现一个 glsl 文件依赖另一个 glsl 文件，可以使用以下工具加载 glsl 字符串。


​ ShaderUtils.java



```
import android.content.Context;

import java.util.HashSet;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * Shader工具类
 * @author little fat sheep
 */
public class ShaderUtils {
    // 匹配: #include  或 #include “xxx”, 并且不以"//"开头
    private static final String INCLUDE_REGEX = "(?m)^(?!//\\s*)#include\\s+(<([^>]+)>|\"([^\"]+)\")";

    /**
     * 通过asset加载shader
     * @param vertexShader 顶点着色器路径, 如: "shaders/origin_vert.glsl"
     * @param fragmentShader 片元着色器路径, 如: "shaders/origin_frag.glsl"
     */
    public static String[] loadShader(Context context, String vertexShader, String fragmentShader) {
        String vertex = StringUtils.loadString(context, vertexShader);
        String fragment = StringUtils.loadString(context, fragmentShader);
        String vertex1 = replaceIncludeFiles(context, vertex);
        String fragment1 = replaceIncludeFiles(context, fragment);
        return new String[] { vertex1, fragment1 };
    }

    /**
     * 通过资源id加载shader
     * @param vertexShader 顶点着色器资源id, 如: R.raw.origin_vertex
     * @param fragmentShader 片元着色器资源id, 如: R.raw.origin_fragment
     */
    public static String[] loadShader(Context context, int vertexShader, int fragmentShader) {
        String vertex = StringUtils.loadString(context, vertexShader);
        String fragment = StringUtils.loadString(context, fragmentShader);
        String vertex1 = replaceIncludeFiles(context, vertex);
        String fragment1 = replaceIncludeFiles(context, fragment);
        return new String[] { vertex1, fragment1 };
    }

    /**
     * 将shader字符串中#include的文件替换为文件内容
     */
    public static String replaceIncludeFiles(Context context, String shaderContent) {
        Pattern pattern = Pattern.compile(INCLUDE_REGEX);
        HashSet set = new HashSet<>(); // 用于去掉重复的include
        return replaceIncludeFiles(context, shaderContent, pattern, set);
    }

    /**
     * 将shader字符串中#include的文件替换为文件内容(include的文件中可能也有include, 需要递归调用)
     */
    private static String replaceIncludeFiles(Context context, String shaderContent, Pattern pattern, HashSet set) {
        Matcher matcher = pattern.matcher(shaderContent);
        StringBuffer sb = new StringBuffer(shaderContent.length());
        while (matcher.find()) {
            String angleBrackets = matcher.group(2); // 尖括号内的路径
            String quotationMarks = matcher.group(3); // 引号内的路径
            String file = angleBrackets != null ? angleBrackets : quotationMarks;
            if (set.contains(file)) {
                matcher.appendReplacement(sb, ""); // 删除重复的include
            } else {
                set.add(file);
                String includeShader = StringUtils.loadString(context, file); // 加载include文件中的字符串
                String quoteShader = Matcher.quoteReplacement(includeShader); // 替换字符串中的转义字符
                String wholeShader = replaceIncludeFiles(context, quoteShader, pattern, set); // 递归替换字串中的include
                matcher.appendReplacement(sb, wholeShader); // 将字符串添加到sb中
            }
        }
        matcher.appendTail(sb);
        return sb.toString();
    }
}

```

​ StringUtils.java



```
import android.content.Context;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

/**
 * 字符串工具类
 * @author little fat sheep
 */
public class StringUtils {
    /**
     * 根据资源路径读取字符串
     * @param assetPath 资源路径, 如: "shaders/origin_vert.glsl"
     */
    public static String loadString(Context context, String assetPath) {
        String str = "";
        try (InputStream inputStream = context.getAssets().open(assetPath)) {
            str = loadString(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return str;
    }

    /**
     * 根据资源id读取字符串
     * @param rawId 资源id, 如: R.raw.origin_vertex
     */
    public static String loadString(Context context, int rawId) {
        String str = "";
        try (InputStream inputStream = context.getResources().openRawResource(rawId)) {
            str = loadString(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return str;
    }

    private static String loadString(InputStream inputStream) {
        StringBuilder sb = new StringBuilder();
        try (BufferedReader br = new BufferedReader(new InputStreamReader(inputStream))) {
            String line;
            while ((line = br.readLine()) != null) {
                sb.append(line).append("\n");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return sb.toString();
    }
}

```

​ 声明：本文转自[【OpenGL ES】GLSL基础语法](https://github.com)。


