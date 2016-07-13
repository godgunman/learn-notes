#### Java编程规范


##### 源文件结构

- 文件格式为 `UTF-8`；
- 顶级类的名字与源文件的名字相同；
- 只允许一个顶级类声明；

##### 命名规范

- 类（包括 `class`、`interface`和 `enum`）采用大骆驼峰命名——`UpperCamelCase`：
- 非 `final` 的变量、方法、参数采用小骆驼峰命名——`lowerCamelCase`；
- 静态变量，字母全部大写，并以下划线分割单词——`CONSTANT_CASE`；
- 包的字母应该全部小写——`com.linkin.app`。

##### 行检查

- 限制行的长度为 `100` 个字符；
- 一行一个语句；
- 每行只声明一个变量；
- 操作符如 `.`、`+`、`-` 等后还有语句时，若需要断行，则把这些符号作为下一行的首字符，而不是当前行的末尾字符：

  ```java
  // wrong
  new Date().
          setTime(1111111);

  // right
  new Date()
          .setTime(1111111);

  // wrong
  String greeting = "This" +
          "is" +
          "a" +
          "long" +
          "sentence" +
          ".";

  // right
  String greeting = "This"
          + "is"
          + "a"
          + "long"
          + "sentence"
          + ".";
  ```

##### 缩进

- 禁止使用 `TAB` 作为缩进符号，统一采用 `4` 个空白字符代替 `TAB` 符号；
- 除数组分行定义时采用 `8` 个空白字符外，其他情况下，缩进统一为 `4` 个空白字符；
- 注释与它对应的代码，缩进应该保持一致：

  ```java
  //      this is resource id (wrong)
          int resId = 0;

          // this is resource id (right)
          int resId = 0;
  ```

##### package & import

- `package` 和 `import` 语句不换行；
- `import` 不要使用通配符 `*`；

##### switch 结构

- `switch` 结构中，必须有 `default` 的分支，即使这个分支不执行任何操作（不执行任何操作的话，应该写上相应的注释说明）：

  ```java
  // wrong
  switch (input) {
  	case 1:
  		prepareOne();
  		break;
  	case 2:
  		prepareTwo();
  		break;
  	case 3:
  		handleThree();
  		break;
  }

  // right
  switch (input) {
  	case 1:
  		prepareOne();
  		break;
  	case 2:
  		prepareTwo();
  		break;
  	case 3:
  		handleThree();
  		break;
  	default:
  		// ignored
  }
  ```

- `switch` 结构中，如果 `case` 分支中没有包含 `break`、`return`、`throw` 或者 `continue`，则需要加上 `fall through` 的注释（以表示并不是开发者遗漏了 `break` 语句，而是`有意为之`）：

  ```java
  switch (input) {
  	case 1:
  	case 2:
  		prepareOneOrTwo();
  		// fall through
  	case 3:
  		handleOneTwoOrThree();
  		break;
  	default:
  		handleLargeNumber(input);
  }
  ```

##### 大括号 `{}`

- 大括号与`if`, `else`, `for`, `do`, `while` 等语句一起使用，即使只有一条语句（或是空），也应该把大括号写上；
- 对于非空块和块状结构，大括号遵循 `K&R风格`：
  - 左大括号前不换行；
  - 左大括号后换行；
  - 右大括号前换行；
  - 如果右大括号是一个语句、函数体或类的终止，则右大括号后换行； 否则不换行。例如，如果右大括号后面是`else`、`catch` 或逗号`,`，则不换行：

    ```java
    return new MyClass() {
      @Override public void method() {
        if (condition()) {
          try {
            something();
          } catch (ProblemException e) {
            recover();
          }
        }
      }
    };
    ```

##### 空格

- 分隔任何保留字与紧随其后的左括号 `(`，如 `if`、`for`、`switch`、`catch` 等；
- 分隔任何保留字与其前面的右大括号 `}`，如 `else`、`catch` 等；
- 在任何左大括号 `{` 前，但以下两个情况例外：

  ```java
  @SomeAnnotation({a, b})

  String[][] x = {{"foo"}};
  ```

- 在任何二元或三元运算符的两侧，以及包括以下的情况：

  ```java
  <T extends Foo & Bar>

  catch (FooException | BarException e)

  for(String name : names)
  ```
- 在`,` 、 `:` 、 `;` 及右括号 `) `后；
- 如果在一条语句后做注释，则双斜杠 `//` 两边都要空格；

##### 其它

- 必须以 Java 风格来定义数组：

  ```java
  // C style
  int numbers[] = {1, 2, 3};

  // Java style
  int[] numbers = {1, 2, 3};
  ```

- 修饰符按照以下的顺序出现：

  ```java
  public
  protected
  private
  abstract
  static
  final
  transient
  volatile
  synchronized
  native
  strictfp
  ```
- 不要有多余的修饰符，例如：接口中的方法不必使用 `public`、`abstract` 修饰；
- 长整型应该用 `L` 来标识，而不是 `l`；
- 当一个类有多个构造方法，或多个同名方法，这些方法应该按顺序出现在一起，中间不要放进其它方法；

##### 更新记录

> 2016-03-18

- 初版。