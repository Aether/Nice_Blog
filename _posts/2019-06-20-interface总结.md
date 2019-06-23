---
layout: post
title: "Java中-able总结"
subtitle: ""
date: 2019-06-20
author: Aether
category: coding
tags: CODE SOFTWARECONSTRUCTION JAVA
finished: true
---

## Java中-able总结

- Comparable & Comparator
- Iterable & Iterator
- Observable & Observer
- Throwable
- Cloneable
- Runnable

### Comparable & Comparator

#### Comparable
需override`compareTo()`方法
~~~
class Person implements Comparable<Person> {
   @Override
   public int compareTo(Person o) {
     if (...) return -1;
     else if (...) return 1;
     else return 0;
   }
}
~~~

#### Comparator
`Comparator`是策略模式`strategy design pattern`，可使用不同的Comparator对象实现不同策略的排序。
需override`compare()`方法
~~~
class Cmp implements Comparator {
  @Override
  public int compare(Object arg0, Object arg1) {
    Person a = (Person) arg0;
    Person b = (Person) arg1;
    if (...) return -1;
    else if (...) return 1;
    else return 0;
  }
}
~~~
调用方法：
~~~
Person[] p = new Person[4];
Arrays.sort(p);//默认排序
Arrays.sort(p, new Cmp());//自定义排序
~~~

#### Summary
- 顾名思义，某个class实现`Comparable`可以使该class的对象可比较：`a.compareTo(b)`
- `Comparator`为比较器和一种比较策略，内部需override`compare()`方法

### Iterable & Iterator

#### Iterator
Iterator的定义：包含`hasNext()`，`next()`和`remove()`方法。
~~~
public interface Iterator {  
　　boolean hasNext();  
　　Object next();  
　　void remove();  
}
~~~
Iterator的使用：
~~~
Iterator iter = l.iterator();
while(iter.hasNext()){
    String str = (String) iter.next();
    System.out.println(str);
}
~~~

#### Iterable

Iterabler的定义：包含`iterator()`方法。
~~~
public interface Iterable<T> {
  Iterator<T> iterator();
}
~~~
Iterable的使用：override`iterator()`方法
~~~
public class ArrayMap<K, V> implements Iterable<K> {
  @Override
  public Iterator<K> iterator() {
      return new KeyIterator();
  }
}

public class KeyIterator implements Iterator<K> {
  @Override
  public boolean hasNext() {
      ...
  }
  @Override
  public K next() {
      ...
  }
}
~~~

### Observable & Observer

#### Observer
~~~
public interface Observer {
  void update(Observable o, Object arg);
}
~~~
#### Observerable
~~~
public class NumObservable extends Observable {
  private int data = 0;
  public int getData() {
     return data;
  }
  public void setData(int i) {
     data = i;
     setChanged();
     notifyObservers();
  }
}
~~~
被观察者`NumObserable`执行了`notifyObservers()`后，观察者执行`update()`方法。
~~~
public class NumObserver implements Observer {

  @Override
  public void update(Observable o, Object arg) {
    NumsObservable myObserable = (NumsObservable) o;
    System.out.println("Data has changed to " + myObserable.getData());
  }

}
~~~

### Throwable
This is a **class**, not interface.
~~~
class MyException_another extends Throwable {
   private static final long serialVersionUID = 1L;
   public MyException_another() {
     super();
   }
   public MyException_another(String msg) {
     super(msg);
   }
   public MyException_another(String msg, Throwable cause) {
     super(msg, cause);
   }
   public MyException_another(Throwable cause) {
     super(cause);
   }
}
~~~

### Cloneable
需重写`clone()`方法
~~~
public class Test implements Cloneable {
  @Override
  protected Object clone() throws CloneNotSupportedException {
      ...
  }
}
~~~

### Runnable

需重写`run()`方法
~~~
class MyThread1 implements Runnable {
  @Override
  public synchronized void  run() {
    ...
  }
}
~~~
