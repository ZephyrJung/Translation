# Java中的数据结构

@Author ZephyrJung

 ![img](http://img.blog.csdn.net/20151124194221498?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## Coolection

- `int size()`

  返回集合中元素的个数，如果超过`Integer.MAX_VALUE`，则返回`Integer.MAX_VALUE`

- `boolean isEmpty()`

  判断是否为空

- `boolean contains(Object o)`

  判断是否包含指定元素，如果元素类型不符，抛出`ClassCastException`异常，如果指定的元素是null并且这个集合不允许包含null元素，则抛出`NullPointerException`

- `Iterator<E> iterator()`

  返回集合元素的游标，不保证顺序

- `Object[] toArray()`

  返回包含集合中所有元素的数组。如果集合限定了元素的游标顺序，那返回的数组也应该是同样的顺序。返回的数组与原集合不存在引用关系，即便集合的实现是通过数组实现的，也要返回一个新的数组。这个方法充当了基于数组和基于集合接口的桥梁

- `<T> T[] toArray(T[] a)`

  :question:

- `boolean add(E e)`

  向集合中添加指定的元素，如果添加成功则返回true，如果已经存在则返回false。支持这一操作的集合可能带有限制条件，如不允许元素为null，在文档中应该明确指出。如果集合拒绝添加了某些特定元素，必须抛出一个异常而非返回false，如此保证在正常调用这个方法后，集合中一定包含这个元素。

- `boolean remove(Object o)`

  删除一个指定元素

- `boolean containsAll(Collection<?> c)`

  如果集合包含给定集合的所有元素，则返回true

- `boolean addAll(Collection<? extends E> c)`

  添加给定集合的所有元素

- `boolean removeAll(Collection<?> c)`

  移除指定集合中的所有元素

- removeIf

  ```java
  default boolean removeIf(Predicate<? super E> filter) {
      Objects.requireNonNull(filter);
      boolean removed = false;
      final Iterator<E> each = iterator();
      while (each.hasNext()) {
          if (filter.test(each.next())) {
              each.remove();
              removed = true;
          }
      }
      return removed;
  }
  ```

  移除集合中符合给定条件的所有元素。@since 1.8

- `boolean retainAll(Collection<?> c)`

  保留给定集合中的元素，也即删除集合中不存在给定集合中的所有元素

- `void clear()`

  移除集合中的所有元素

- `boolean equals(Object o)`

  接口对于equals的实现没有特别的约束。通用的规则是满足对称性，换句话说a.equals(b)和b.equals(a)应该同时满足。List和Set则要求equals的对象必须分别也是List和Set，因此，一个集合如果既没有实现List也没有实现Set，那么当List或Set传入equals方法时必须反会false。基于同样的逻辑，一个类不可能同时实现List接口和Set接口。

- `int hashCode()`

  返回这个集合的哈希值。接口对于哈希值的实现也没有特定的要求，但重写了Object.equals方法时，也必须重写Object.hashCodde方法。在实践中，c1.equals(c2)意味着c1.hashCode==c2.hashCode

- spliterator

  ```java
  @Override
  default Spliterator<E> spliterator() {
      return Spliterators.spliterator(this, 0);
  }
  ```

  :question:

- stream

  ```java
  default Stream<E> stream() {
  	return StreamSupport.stream(spliterator(), false);
  }
  ```

  :question:

- parallelStream

  ```java
  default Stream<E> parallelStream() {
      return StreamSupport.stream(spliterator(), true);
  }
  ```

  :question:

## List extends Collection

:question:List接口继承了Collection，为何需要重写已有的接口？为何有的写了有的没写。。。

List除了继承Collection接口所指定的方法之外，添加了部分重载函数和新得方法

- `boolean addAll(int index,Collection<? extends E> c)`

  从指定位置向List加入给定集合中的所有元素

- replaceAll

  ```java
  default void replaceAll(UnaryOperator<E> operator) {
      Objects.requireNonNull(operator);
      final ListIterator<E> li = this.listIterator();
      while (li.hasNext()) {
          li.set(operator.apply(li.next()));
      }
  }
  ```

  替换所有元素为指定操作的结果。@since 1.8

- sort

  ```java
  default void sort(Comparator<? super E> c) {
      Object[] a = this.toArray();
      Arrays.sort(a, (Comparator) c);
      ListIterator<E> i = this.listIterator();
      for (Object e : a) {
          i.next();
          i.set((E) e);
      }
  }
  ```

  根据给定规则给元素排序。@since 1.8

- `E get(int index)`

  获取指定位置的元素

- `E set(int index,E element)`

  替换指定位置的元素为给定元素

- `E remove(int index)`

  删除指定位置的元素

- `int indexOf(Object o)`

  获取给定元素首次出现的位置，如果没有则返回-1。

- `int lastIndexOf(Object o)`

  获取给定元素最后出现的位置，如果没有返回-1。

- `ListIterator<E> listIterator()`

  返回List元素的游标

- `ListIterator<E> listIterator(int index)`

  返回从指定位置开始的元素的游标

- `List<E> subList(int fromIndex,int toIndex)`

  返回表指定起止位置的视图，包括fromIndex，不包含toIndex，如果起止位置相同，则返回空。当一个List明确要求一个范围内的操作时，可以传递一个子表视图而不是用原整个表，如：`list.subList(from,to).clear()`移除了List中的一定范围内的元素。

可见，List相对于Collection而言，多出了位置的概念。有了位置也就有了顺序，新增的方法一般都跟这个有关。

## Set extends Collection

Set的方法没有新增的，官方定义Set为不包含重复元素的Collection。

## Queue extends Colletion

该接口只重写了Collection的add(E)方法，其余皆为新增：

- `boolean offer(E e)`

   