### Order-Only前提条件

---

makefile 中的生成规则格式为：

```makefile
target : normal-prerequisites | order-only-prerequisites
[TAB]command1
[TAB]command2
[TAB]...
[TAB]commandN
```

正常前提条件的作用：

- 在 `target` 目标下的命令被执行前，所有`正常前提条件`的生成命令都需要被执行。
- 任何一个前提目标 `normal-prerequisites`比生成目标`target`新时，生成目标都被认为太旧而需要被重新生成。

命令前提条件的作用：

- 执行某个或某些规则，不引起生成目标被重新生成。

示例如下：

```makefile
LIBS=lib.c
foo: foo.c | $(LIBS)
	touch foo
	@echo "order"
```

运行如下过程：

```shell
touch foo.c lib.c
make
vim lib.c #修改lib.c的内容
make
vim foo.c #修改foo.c的内容
make
```

会得到如下结果：

```shell
---第一次make---
touch foo
order

---第二次make---
make: 'foo' is up to date.

---第三次make---
touch foo
order
```

可以看到 `lib.c` 的修改不会影响 `foo` 的重新生成。