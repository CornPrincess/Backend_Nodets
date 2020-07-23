1.Math.abs(-2147483648) = -2147483648 ，因为数据溢出（integer overflow）

2.为什么数组的起始索引是0而不是1:

> This convention originated with machine-language programming, where the ad- dress of an array element would be computed by adding the index to the address of the beginning of an array. Starting indices at 1 would entail either a waste of space at the beginning of the array or a waste of time to subtract the 1.

可以在命令行使用 `-enableassertions` 标志（简写为-ea）来启用断言

为什么要区别原始类型和引用类型，为什么不只用引用类型？ 因为性能，原始类型更接近计算机硬件支持的数据类型，因此使用它们的程序比使用引用类型的程序运行地更快

和Java引用一样，可以把指针看做是机器地址，在许多编程语言中，指针是一种原始数据类型。在Java中创建引用的办法只有一种（new），且改变引用的方法也只有一种（赋值语句），Java的引用被称为安全指针，因为Java保证每个引用都会指向特定类型的对象。

Java中可以通过反射来改变String的值，可见[例子](https://algs4.cs.princeton.edu/12oop/MutableInteger.java.html)
