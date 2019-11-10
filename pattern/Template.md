# 1. Template   模板模式

模板方法模式：在一个方法中定义一个算法的骨架，而将一些步骤**延迟到子类**中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

```puml
abstract AbstractClass {
   templateMethod()
   primitiveOperation1()
   primitiveOperation2()
   hook()
}

class ConcerteClass {
  primitiveOperation1()
  primitiveOperation2()
}

AbstractClass <|.. ConcerteClass
```
