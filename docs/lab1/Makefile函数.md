### value函数

---

`value`函数提供了一种在不对变量进行展开的情况下获取变量值的方法。

语法：`$(value VARIABLE)`

功能：**不对变量`VARIABLE`进行任何展开操作**，直接返回变量`VARIABLE`的值。返回`VARIABLE`的值，它是一个变量名，一般不包含`$`（除非计算的变量名）。

返回值：变量`VARIABLE`定义的文本值。

示例如下：

```makefile
FOO = $PATH
BAR = $(PATH)

first_second = Hello
a = first
b = second
c = $($a_$b)

all:
	@echo "1: $(FOO)"
	@echo "2: $(value FOO)"
	@echo "3: $(BAR)"
	@echo "4: $(value FOO)"
	@echo "5: $(c)"
	echo $(value c)
	$(info $(value FOO))
```

输出结果为：

```shell
1: ATH
2: /home/monster/bin:/home/monster/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
3: /home/monster/bin:/home/monster/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
4: /home/monster/bin:/home/monster/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
5: Hello
echo $($a_$b)
$PATH
```

第一行为：`ATH`。这是因为变量`FOO`定义为`$PATH`，所以展开为`ATH`（`$P`为空）。 第二行才是我们需要显示的系统环境变量`PATH`的值（value函数得到变量`FOO`的值为`$PATH`）。

### eval函数

---

函数`eval`会对它的参数进行展开，展开的结果可以包含一个新变量、目标、隐含规则或者明确规则，展开结果作为`makefile`的一部分。此函数的主要功能是根据其参数的关系、结构，对它们进行替换展开。

“eval”函数执行时会对它的参数进行两次展开。第一次展开过程是由**函数本身完成**的，第二次是函数展开后的结果被作为Makefile内容时由**make解析时展开**的。明确这一过程对于使用“eval”函数非常重要。理解了函数“eval”二次展开的过程后。实际使用时，如果在函数的展开结果中存在引用（格式为：\$(x)），那么在函数的参数中应该使用“$$”来代替“$”。因为这一点，所以通常它的参数中会**使用函数“value”来取一个变量的文本值**。（文本值 vs 展开值）。

示例如下:

```makefile
OBJ=a.o b.o c.o d.o main.o

define MA
main:$(OBJ)
	gcc  -g -o main $$(OBJ)
endef

$(info $(call MA))
$(eval $(call MA))
```

运行如下命令：

```shell
touch a.c b.c c.c d.c main.c
make
```

得到如下结果：

```shell
main:a.o b.o c.o d.o main.o
	gcc  -g -o main $(OBJ)
cc    -c -o a.o a.c
cc    -c -o b.o b.c
cc    -c -o c.o c.c
cc    -c -o d.o d.c
cc    -c -o main.o main.c
gcc  -g -o main a.o b.o c.o d.o main.o
### 忽略后面的错误提示
```

从结果看出，第一次执行调用展开，在整个makefile调用代码中去掉了一个`$`（即全部都进行了一次展开）。

接着看下面一个示例：

```makefile
pointer := pointed_value

define foo 
var := 123
arg := $1
$$($1) := ooooo
endef 
  
$(info $(call foo,pointer))
#$(eval $(call foo,pointer))
 
target:
	@echo -----------------------------
	@echo var: $(var), arg: $(arg)
	@echo pointer: $(pointer), pointed_value: $(pointed_value)
	@echo done.
	@echo -----------------------------
```

> 注意`target`下面的命令必须时`Tab`开始，不能有空格开始，否则，由于里面有`:`，在运行`make`的时候会提示`*** multiple target patterns.  Stop.`

运行结果为：

```shell
var := 123
arg := pointer
$(pointer) := ooooo
-----------------------------
var: , arg:
pointer: pointed_value, pointed_value:
done.
-----------------------------
```

`info`函数只是将`$(call foo,pointer)`的返回值，也就是替换后的代码段，打印到标准输出，而并没有执行代码段，因此上述的各个值均为空。`$(call foo, pointer)`就是makefile对`foo`函数进行第一次求值，求值结果仍然是makefile代码段。那么问题就来了。**既然求值出来的结果还是 Makefile 代码，那这段代码又要怎么运行呢？答案就是再包一个 eval, 所以 eval 就是第二次求值了**。

将上面的注释信息进行调整（注释掉`info`，取消`eval`的注释），可以看到如下结果：

```shell
-----------------------------
var: 123, arg: pointer
pointer: pointed_value, pointed_value: ooooo
done.
-----------------------------
```

> 说明：在makefile中，一个`$()`表示引用变量里面的值，而`$$`表示的是一个单独的`$`符号。可以看成序列`$1`，`$2`，`$3`...`$$`，相当于转义字符。

