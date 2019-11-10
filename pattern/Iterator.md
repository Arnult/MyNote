# 1. Iterator 迭代器

迭代器模式提供一种方法顺序访问一个聚会对象中的各个元素，而又不暴露其内部的表示。

```puml
interface Iterator{
   hashNext()
   next()
   remover()
 }
class ConcreteIterator {
  hasNext()
  next()
  remove()
}
Iterator <|.. ConcreteIterator
Client *--Iterator


 
```
