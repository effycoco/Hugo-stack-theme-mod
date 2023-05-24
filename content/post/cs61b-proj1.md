---
title: "记录完成CS61b Project1过程中遇到的问题"
description:
date: 2023-05-24T18:19:19+08:00
image:
categories:
slug: cs61b-proj1
tags:
math:
license:
hidden: false
comments: true
draft: false
---

1a 12.28 - 12.30

1b 01.03 - 01.05

1c 03.05 - 03.08

# 1a

学完第三周 DLList 后可做本部分。

这部分的任务创建 `LinkedListDeque.java`，也就是 DLList，可以由课上讲的 SLList 改成。同时还应该使用并丰富 `LinkedListDequeTest.java`。

## 1(solved)

怎样让 sentinel 的 prev 和 next 指向自身？我想要实现的结构

![](https://raw.githubusercontent.com/effycoco/images/main/img/%E6%88%AA%E5%B1%8F2022-12-28%2009.17.17.png)

my try

```java
sentinel = new TNode(null, TNode sentinel, TNode sentinel);
sentinel = new TNode(null, TNode this, TNode this);
```

显然不对。

先不管了。

我犯了一个错误，传递 arguments 时不需要写 type，因此上述代码语法错误。问题本身也想到了解决方案：创建 sentinel node 时先把 prev, next 设为 null，创建好后再 reassign 自身给这两个 pointer，如下

```java
sentinel = new TNode(null, null, null);
sentinel.prev = sentinel;
sentinel.next = sentinel;
```

顺便一提，project1 slides 里的一句话对我看懂和自己画这种示意图帮了大忙，"Any time I drew an arrow in class that pointed at an object, the pointer was to the ENTIRE object, not a particular field of an object." 箭头的指向是整个 node, 而非 node 的某个 field。箭头的来处是 node 的 field。

## 2(solved)

是否应该在新建 node 时指定 next 和 prev，即将这两个作为创建时需接收的参数。此问题和 1 相关。相关代码

```java
 // when define node class
public class ItemNode{
	   // declare variables needed: item, prev, next
	   ...
	   // node constructor
	   public ItemNode(Type i, ItemNode p, ItemNode n ){
		   item = i;
		   prev = p;
		   next = n;
	   }
}
	// when intialize the sentinel node， - 略，see 问题1

```

我感觉是需要的，因为 SLList 里在这两个地方都指定了 next。

最后证明确实需要。

## 3(solved)

当 node 的 item 是 int 时，我们创建 sentinel 时可以将其 item 设为任意整数，反正也不会用到；但当 node 的 item 可能是任意类型时，创建 sentinel 时要将其初始化为什么值? `null` 吗? 此问题依然与问题 1 相关。相关代码见问题 1 和 2。

最后证明就是用 `null`。

## 碎碎念

感觉直接写 generic type 适用的版本要比写只适用于 int 的版本难。我先试试把 SLList 改成 generic 版本，然后又遇到了问题 3。然后我发现可以用 `null`。改完了。决定还是直接写 generic 版本。现在我需要忽略问题 1 继续写 LinkedListDeque 本身。对照 SLList 过程中解决了问题 1。然后写出了 `addFirst` 和 `addLast` 两个方法，没遇到什么障碍，但画图捋思路的过程好有成就感，忍不住放张草稿纸自我欣赏。

![deque示意图.jpg](https://raw.githubusercontent.com/effycoco/images/main/img/IMG_8122.jpg)

写完这两个方法我也确认了问题 2，原本的代码可行。然后就继续写剩下的方法。

写 `removeFirst` 的时候想到一个垃圾回收问题（见问题 4），很快想通了。该问题可能看起来像废话文学，但这种自问自答可以让我理解更透彻。写 `equals` 时遇到问题 5，也很快解决了。

至此就把 `LinkedListDeque` 初版本写完了，一开始是不知从何下手，尝试写时被前三个问题卡住，解决之后后面写得都比较顺利。但这不是最终的版本，肯定有 edge case 没有考虑到，接下来我要使用 starter code 里的 test code 做 test。

使用 test 发现的问题有（记录是为了学习怎么写 testcode）：

1. remove 时忘记 `size--`，发现这点多亏 addRemoveTest 在 list 创建时、调用 `addFirst` 后、调用 `removeFirst` 后都使用了 isEmpty 来检查是否与预期一致。
2. 忘记考虑 list 为空的情况。发现这点是因为 starter code 有 `removeEmptyTest`。

好吧，它给的初始 testcode 确实不足，还需要自己继续写。让我重读一遍 spec，一个可能不值得说的心得就是做项目时要经常看 spec（做项目前通读一遍）。其实 testcode 文件是不算分数的，注册学生有 autograder，但老师限制了提交次数，目的就是让学生自己检查代码而不是依赖 autograder。这倒是方便了我这种非注册学生，只要会自己写 testcode，上课的体验和所学就不会比注册学生差。本来想逃避的，还是写吧。对照着 lab3 以及课上给出的 SLList 的 testcode，应该写得出来。

我不确定是要给每个 test 多加几个 case、多调用几个方法，然后再写 randomizedTest，还是直接增加一个 randomizedTest 就可以呢。先试后者。写了 randomizedTest，把除 `equals` 外的所有方法都用上了，N 设为 5000 没出问题，应该就没问题了。如果没有已知正确的版本怎么测试 `equals` 呢，或许我可以 import API 自带的 DLList，然后按 lab03 教的写 random comparison test。但是这门课还没告诉我怎么用 Java 自带的 list 结构呢，虽然已经在 Head First 里学过了，还是偷懒先不做了。

接下来该做 1b 了。

## 4 垃圾回收 (solved)

Remember that the Java garbage collector will “delete” things for us if and only if there are no pointers **to** that object.

这个问题就是已知当没有任何 reference 指向某个 object 时，这个对象就会被清理；那以下两种情况 object 会被清理吗?

    a. 当object A有其他object的reference时，视觉上就是A的某个field指向不会被清理的另一个object。 答案是A仍然会被清理，因为没有对象A的reference，也就无法访问对象A内部的reference了。因此，根本无需考虑从A出发的箭头，只需考虑是否有指向A的箭头。

    b. 当object A 的某个field是object，并且有reference指向这个object。答案也是A会被清理。因为A的某个field是object并不意味着这个object在A内部，这个object是独立于A的，只是有来自A的箭头指向这个object，因此这个问题本质上和第一个是同一个问题。

## 5 `instanceof` (solved)

（这其实就是 Casting 问题，后来学了，做这个项目时还没学到）

在 `if(o instanceof LinkedListDeque)` 里面使用 o 的方法（作为一个 LinkedListDeuque instance 会有的方法）会报错，也就是说还是只能在 o 上调用 object 方法而不能调用 LinkedListDeuque instance 的方法。

(digress, instance 拥有的方法和 fields/variables 好像有个总称，叫什么来着？查到了，是 _members_，可以说 class members, instance members, members of the object).

这个问题问 chatGPT 解决了，好像有点作弊啊（当然不算，我只是更快速精准定位到所需信息嘛）。得到的关键信息是 the `instanceof` operator does not allow you to access the members of the object directly. In order to access the members of the object, you need to cast the object to the specific class type first.

贴一下完整的问题和回答：

问：

java question. o is a instance of Object, and DLListDeque is a class, which have method size(). I wrote following code, `if(o instanceof DLListDeque){int a = o.size()}` but I got error saying Cannot resolve method 'size' in 'Object'. I wonder why

答：

The `instanceof` operator is used to determine whether an object is an instance of a particular class or one of its subclasses. It returns a boolean value indicating whether the object is an instance of the specified class.

However, the `instanceof` operator does not allow you to access the members of the object directly. In order to access the members of the object, you need to cast the object to the specific class type first.

For example, in your case, you can modify your code as follows:

```java
if (o instanceof DLListDeque) {
    DLListDeque deque = (DLListDeque) o;
    int a = deque.size();
}
```

This will cast the `o` object to a `DLListDeque` object, which allows you to access its `size` method.

Alternatively, you can also check the class of the object using the `getClass` method and then invoke the method using reflection:

```java
if (o.getClass() == DLListDeque.class) {
    Method sizeMethod = DLListDeque.class.getMethod("size");
    int a = (int) sizeMethod.invoke(o);
}
```

However, using reflection can be slower than using a cast, so it's generally better to use a cast if you know the type of the object at compile time.

# 1b

这部分需要写 `ArrayDeque.java`，实质上是 Array-based list。并写测试代码。

和课上的 AList 不同的是这次要让 `addFirst`, `removeFirst` 也 take constant time.

因此，`ArrayDeque`的 first 实际上在数组中间的位置，要让数据的两端都有空位，将普通数组变成首尾相连的结构。

先根据 slides 里提供的结构图示，将它转换为代码。遇到问题 1，解决。先不考虑 resize 把各个方法写出来了。然后考虑 resize 的情况也顺利写出来了。接下来写 test code。边写边改了 real code 里的一些 bug。最后写了一个 randomizedTest，把 N 设为 5000 时测试花了 5 秒，比 LinkedListDeque 时间长很多，应该是 resize 带来的。嗯 1b 现在就完成啦。依然不确定我的测试代码是不是找出了所有 bug。

## 1

有两个 pointer, `nextFirst` and `nextLast`，一个在 addFirst 时向前移，一个在 addLast 时向后移。但当其中一个移到末端后，若 array 还没填满，需要将该 pointer 移到另一端，也就是说将 array 视为一个首尾相接的循环结构；更具体地说，当 nextFirst=0、nextLast<length-1，即数据填满了前端，尾端有空位，addFirst 后应将 nextFirst 设为 length-1。如何实现？

本来想着一开始设置 nextFirst=0, nextLast=1 应该会简化情况，后来发现没差。但逻辑不难，不过还是保留这个问题吧：

```java
public void addFirst(T item){
    if(size==items.length){
        resize(size*2);
    }
    items[nextFirst] = item;
    size++;
    if(nextFirst==0){
        nextFirst = items.length-1;
    }else{
        nextFirst--;
    }
}
```

## 2 Negative Size

通过自动评分机检查出来这个问题，出在 removeLast 方法里，没考虑到 size 为 0 的情况。先在 Test 文件里加一个相应测试，code 里最简单的解决办法是加

```java
if(size==0){
    return null;
}
```

但我想看看不这样的话问题出在哪里。
如果不加这一条件，在 size 为 0 时，会返回正确的结果，但 size 会减 1 成为-1，从而使后续运算出现问题。就还是在最前面加上这个条件好了。removeFirst 也要。

# 1c

## Iterable

天呐终于可以开始做这部分了。

首先需要使 1a,1b 写好的两个 deque Iterable。对应 Reading6.1-6.4、lecture11 的内容。

照葫芦(6.3 ArraySet)画瓢写好了 ArrayDeque，使其 Iterable。并写了测试，通过。

照另一个葫芦(disc05 OHQueue)写 LinkedListDeque，这个不一样的地方比较多，有点绕。首先是有 sentinel 时如何判断 hasNext()，一般的 linkedlist 可以根据`==null`判断，但这个结构用 sentinel 代替了 null，应该用`.equals(sentinel)`判断是否还有下一个。想清楚这点就很快写好了。写了测试并通过。

这两个文件正好代表两种集合，array 和 linked list，前者写 iterator 时采用 index，后者需创建 instance variable 代表当前 node，作为 pointer。后者的 constructor 需接收一个 node 参数代表第一个 node(或 sentinel)。

## `MaxArrayDeque`

接下来该写`MaxArrayDeque`了，是`ArrayDeque`的 subclass。需要用 Comparator，我怎么记得 61b 没讲过这个，是可选教材 Head First Java 讲的，薛定谔的选读？不过即使没讲也应该会自己查才行啦。虽然想直接做，但感觉无从下手，趁此机会先把 hf 讲这部分的笔记给做了。做完了。继续做这个。

没懂，constructor 的要求"creates a `MaxArrayDeque` with the given `Comparator`"是什么意思？`Comparator`不是单独的一个 class 吗，ArrayDeque 的构造器是要创造一个空数组，并且给几个 instance var 赋值。`MaxArrayDeque`肯定也是从零创造，不会在创造之初就要给已有集合排序……。懂了，意思应该是把给定的 comparator 设为默认排序方式？可是默认排序不是要靠元素 class implements Comparable 来设吗，要用 comparator 的时候是一定要把它作为参数传递的啊，难道要转换？给你一个 comparator，转换成 comparable 用相同方式排序？但我又没写元素 class。啊不懂，先跳过。

然后要写个返回最大值的方法，用之前给的 comparator，哦其实上个问题就只是拿个变量储存给的 comparator 吧哈哈。但是怎么用呢，在 hf 里用的是`List.sort(Comparator c)`，之所以成立是因为 List 的 API 定义了这个方法，但自定义的 ArrayDeque 没有，array 也没有。还有一个用法是`Collections.sort(List list, Comparator c)`，搜了一下 List 不能替换成 Array，所以要怎么用呢。头晕，去睡了，醒来先跳过这个。

下一个方法也是返回最大值，但不是用创建时给的 comparator，而是传入 comparator 为参数。跳过。

先读完这部分的说明，还是没有头绪的话就跳过这节。倒是对写测试有点头绪，写好了一个测试。
写完了 Deque，再继续这个，啊又搜了一遍 61b 的电子书压根没讲。网上搜索的主要困难是实际用时直接使用 java 内建的 ArrayDeque class，这个是从头写，内建的有些继承关系是自制版没有的。不过我知道这个课设计从头自制 Java 内建类的意义就在于弄清楚使用某一特性时是类的哪部分继承关系和成员起了作用。

嘿嘿写完了，测试也通过。我知道之前哪里想的不对了，`List.sort(Comparator c)`是排序时用的，这里并不需要，这里只需找最大值，而无需改变数列的顺序，所以只需利用 Comparator Interface 声明的方法 compare(a,b)就可以了。就只需把找数组最大值时用的`if(a<b)`改为`if(c.compare(a,b)<=0)`。（其实这点是问 AI 该怎样写一个构造器接收 comparator 的 ArrayDeque 从它给的答案中意识到的。）

接下来该做 GH 了，明天再说。

## Deque Interface

这部分就是加个接口，让 ArrayDeque 和 LinkedListDeque 都实现这个接口。其中 isEmpty()和 equals()在两个 class 中的实现逻辑相同，所以应提取出来写在 Deque 接口。这里遇到一个问题是默认方法不能重写上层 class 的非默认方法，也就是说：

```java
public interface Deque{
	// okay， because isEmpty() is a new method declared by this interface
	default boolean isEmpty() {
	    return size()==0;
	}
	// wrong, 提示Default method 'equals' overrides a member of 'java.lang.Object'
	// 不能将equals()设为默认方法，默认方法只能是这个接口声明的新方法
	default boolean equals(Object o){
		// 略
	}
}
```

也就是说 `isEmpty()`和 `equals()`虽然都需要在 interface 里 generalize，但应采用不同的处理方式。前者只需如以上代码所示，将具体实现写在 default method 里，但对 Object 方法无法这样做，所以仍应在每个 concrete class 中重写 Object 的 equals()，只是在看类型是要看是不是属于某接口，而非具体类，这样就能在同接口不同类之间比较了。不过这样各类的 equals 方法重复了，不知道有没有更好的做法。(反正我一开始以为要想个办法让 equals 的 body 写在 interface 里，具体类里面的 equals 删掉，结果一直没找到，绕了点远路。)

啊现在再回过头做 MaxArrayDeque。

# Guitar Hero

读题有些费脑筋，大概懂了后开始做。
写好 GuitarString，测试通过 1/4，应该是哪里理解错，明天再看吧。
啊啊啊啊后面好几天都没有写码，今天终于试着再捡起来。先从再读一遍题和 start code 和 test code 开始吧。

1. 将 deque 的每一个元素替换为随机值（-0.5 ～ 0.5，也就是白噪音）
2. 去掉 deque 的第一个元素，计算(d1+d2)/2\*DECAY，把计算结果添加到 deque 末端，同时。这个结果称为 `newDouble`
3. 播放上一步得到的 `newDouble`，方式是`StdAudio.play(-0.9)`。然后重复第二步。

class `GuitarString` 要实现的就是步骤 1、2。步骤 3 将由 `GuitarString`的 client 完成。
在构造器中用零填充命名为 buffer 的 deque。给出了 buffer 需要的长度`capacity`，这步应该没问题。
然后另外三个函数我就不太懂了。
`pluck`要做的是步骤 1，由于使用的结构是 deque，将 0 全部替换为随机值实际上是先移除一个再添加一个，先 removeFirst 再 addLast(不知道先 removeFirst 再 addFirst 是不是一样)。这个方法写的应该也没问题。`tic`和 `sample`我就有些混乱。`tic`是要完成步骤 2，`sample`是要获得步骤 2 中被去掉的首位元素，但有一个地方重叠的，sample 需返回首位元素，也就是步骤 2 和方法`tic`中首先被去掉的元素。感觉这点在题目中没写清楚，第一版代码，我以为需要在 sample 中也 removeFirst()，然后在 tic 方法中先 call `sample`方法，将返回值用于接下来的计算。实际上两个方法互不干扰，sample 返回的是 getFirst()，两个方法不应关联，应能分别调用。

总之，debug 成果就是方法之间应独立，不存在内部调用，`sample`方法只需 get，不应 modify。

啊我被卡住之后 7 天没写代码，结果 2 小时就解决了耶。

有个思考题，是考虑用 ArrayDeque 和 LinkedListDeque 的差异，在 GuitarString class 中只涉及 deque 的三个方法，`get(0)`, `removeFirst`, and `addLast`，ArrayDeque 的三个方法都是常数，ListDeque 后两个也是，`get`虽然是 O(N)，但`get(0)`是常数，所以两个结构在这里几乎没有区别。（但 AI 说虽然 performance 没有区别，ArrayDeque 还是更好，因为从两头添加或删除元素都是常数时间。可是 LinkedListDeque 也是啊，查了一下 API，有 ArrayDeque 没有 LinkedListDeque，所以应该是 AI 只考虑了原生的 `ArrayDeque`和 `LinkedList`。AI 嘴好硬啊）

但接着就遇到了问题，`GuitarHeroLite`运行时应该出现一个图形交互界面，结果是空白的。`TTFAF`倒是可以正常运行。给的提示全是以 UpdateRecents 开头的，搜了一下好像和代码无关。先不管了。

至此这个项目就算做完了，还落下一点不计分的，因为和 GuitarHeroLite 相关，也没法做。
对它的原理还是不太懂，有空再研究。

# Gradescope

用上了自动评分机，提交到 Project1:checkpoint 在最后一步遇到问题，测试 ArrayDeque 的 negative size

```
Test Failed!
    java.lang.ArrayIndexOutOfBoundsException: Index -1 out of bounds for length 8
    at deque.ArrayDeque.removeLast:94 (ArrayDeque.java)
    at AGTestArrayDeque.negSizeTest:177 (AGTestArrayDeque.java)
```

好吧，这点确实没考虑到。返工解决。见 1b - negative size 小节。
