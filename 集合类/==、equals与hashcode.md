==与equals的区别？

==: 它的作用是判断两个对象的地址是否相等，即判断两个对象是不是同一个对象。

equals(): 它的作用也是判断两个对象是否相等。

如果类没有覆盖 equals() 方法，则通过 equals() 比较该类的两个对象时，等价于通过 == 比较这两个对象。

如果类覆盖了 equals() 方法，可以覆盖 equals() 方法来比较两个对象的内容是否相等，如果内容相等则返回 true。

String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较对象的内存地址，而 String 的 equals 方法比较的是对象的值。



equals与hashcode？

如果两个对象相等，则hashcode一定也是相同的；

两个对象有相同的hashcode值，它们不一定是相等的。

一般地，equals() 方法被覆盖过，则 hashCode 方法也必须被覆盖。