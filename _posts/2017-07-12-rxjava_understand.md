---
layout: post
title: "对 RxJava 的一些理解"
description: ""
category: 
tags: [rxjava]
---

谈谈我对 RxJava 的一些理解 (以 1.x 为例)。陆陆续续看了一两年的 ReactiveX 这种编程思想，文章看了很多，RxJava 的源码看过，也简单用过 RxSwift，因为一直没有用到实际项目中，所以只能说是初略掌握了。以下是我的一些个人理解，用词不是很精确，也不一定正确。

### Rx 与普通的流式编程的区别

首先，什么是普通的流式编程，其实这个名字是我自己想的 (来源是 Java 的 Stream API)，我也不知道该用一个什么专业名词来称呼，不过看下面的例子你就能明白我想要表达的意思。

用 JavaScript / Ruby / Swift 实现的流式编程方法：

    // JavaScript
    let r = [1,2,3,4,5,6,7,8,9].filter(i=>i>5).map(i=>i*10).reduce((a,b)=>a+b, 0)

    // Ruby
    r = [1,2,3,4,5,6,7,8,9].select {|i| i>5}.map {|i| i*10}.reduce(0) {|a,b| a+b}

    // Swift
    var r = [1,2,3,4,5,6,7,8,9].filter {$0>5}.map {$0*10}.reduce(0) {$0+$1}

比较新的现代语言，都给像数组这种可迭代的对象，赋予了 filter / map / reduce 等这些常见的操作，它们内部帮我们实现了循环，使我们只需集中在实现最关键的逻辑上。有了这些操作符以后，基本就和 for 循环 say goodbye 了。

所以，当你看到 Rx 中的 fitler / map / reduce 这些方法时，会不会有这种疑问，咦，它们和上面的 filter / map / reduce 是不是一样啊，特别是我们可以用相似的方法调用来得到相同的结果，就更加令人疑惑了。如下所示。

    Integer[] numbers = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9};
    Observable.from(numbers)
            .filter(integer) -> { return integer > 5; })
            .map((integer) -> { return integer * 10; })
            .reduce(0, (int1, int2) -> { return int1 + int2; })
            .subscribe((integer) -> { Log.i(TAG, "result: " + integer); });

同样得到 300 的结果。

但是，可以说，Rx 和上面的流式编程，形似而神不似，内部实现逻辑可以说是截然不同的。

举个形象但不是非常精确的例子，有 2 种做菜的方式。

第一种方式：

1. 拿到了所有做菜的材料，先全部洗干净了，剔除太小或变质的材料。(对应 filter 操作)
1. 然后把所有的材料都全部切好。(对应 map 操作)
1. 最后用这些材料一起做出菜来。(对应 reduce 操作)

这是常见的流式编程的方式。

第二种做菜的方式，以青椒炒肉为例：

1. 先取肉，洗，切，炒。(对应 肉的 filter -> map -> reduce 操作)
1. 等肉炒得差不多了，我们再取青椒，洗，切，炒，最后和肉一起出锅。(对应青椒的 filter -> map -> reduce 操作)

这是 Rx 的方式。

(你可以在这两种方式的实现代码中输出 log 来验证)。

可见，常见的流式编程法，每一步都是对数据进行整体操作的，操作后得到另外一堆数据，然后整体作为下一步的输入。

而 Rx，是在源头，将数据源中的每一个数据，单独发射出去，使之每一个数据都单独地走一遍整个流程。

因而对应到它们的内部实现上：

1. 流式编程法，每一个单独的步骤，在内部都重新对整体数据进行一次循环，因此它要做很多次小循环，但整体的流程只走一遍。
1. Rx，只在数据最源头，对整体数据进行一次循环，取出每一个单独的数据，让它去走一遍整体流程，因此，它只做一次循环，但整个的流程会走很多遍。

对于 Rx 来说，如果数据源不是固定的，那么它可以监听这个数据来源，当有数据到来时，它就可以把它发射出去把整个流程走一遍。这就是我用来理解 Rx 的观察者模式和响应式编程的方法。

Rx 比常见流式编程的抽象程度更高，它不光抽象了对数据的操作，还抽象了线程切换操作。

### Observabl(Rx) / Promise / Array / Builder 的比较

这里的 Array 指的是上面所说的支持 filter / map / reduce 操作的 Array。

以上这几种方式的共同点是它们可以实现链式调用，原因是它们的方法将返回同类型的实例引用，但返回的这个实例引用，有可能是一个全新的实例，也有可能是原来的实例。

1. 返回值的区别

   - Observabl / Promise 返回新的实例，Observable 返回新的 Observable 对象，Promise 返回新的 Promise 对象
   - Array, filter / map 返回新的数组，但 reduce 返回非数组，并且到这里后链式调用就没法继续了
   - Builder，返回自身引用

1. 是否延迟执行

   - Promise 对象，当它被 new 出来后，其中的方法体是立即执行的，不管后面有没有用 `.then()` 加上回调，执行的结果被存储在对象之中，如果后面再用 `.then()` 加上回调，之前的结果会马上传递给新的回调。
   - Rx 的 Observable 对象，当它被 new 出来后，其中的方法体，只有 `.subscribe()` 方法被执行的那一刻才会执行，否则，只要没有调用 `.subscribe()`，其中的变换就永远不会执行，这和 Promise 是完全不一样的。

### 对 flatMap 的理解

我觉得可以从两个角度来理解。

角度一：

我们上面说到，Rx 的 Observable，每个变换的返回值都应该是一个新的 Observable 对象。以 map 变换为例：

    Integer[] numbers = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9};
    Observable.from(numbers)
            .map((integer) -> { return integer * 10; })

此例中，map 变换中的方法体，返回值是 Integer 类型，并不是我们想要的 Observable，因此你可以理解成 map 内部要把这个返回值重新包装成 Observable (虽然实际并不完全是这样)，此例中，最终的返回值会包装成 `Observable<Integer>`。

但是如果这个方法，它的返回值就是 Observable 呢，那么是不是就可以省去这层封装了呢，我想答案是的。但这时候，就不能再用 map 变换了，必须用 flatMap 变换，flatMap 与 map 相比，其实内部就是少了一层 Observable 的再次封装，就是相当于你替 map 手动干了这个活。

另一个意思就是，filter 抽象的是 T -> boolean 的操作，map 抽象的是 T -> R 的操作，而 flatMap 抽象的是 T -> Observable 的操作。只要是返回值是 Observable 的方法，都可以用 flatMap 变换。

用 flatMap 实现上例中和 map 相同的效果：

    Integer[] numbers = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9};
    Observable.from(numbers)
            .flatMap((integer) -> { return Observable.just(integer * 10); })

(Promise 有着相同的实现原理，在 `Promise.then(func)` 方法中注册的方法，如果返回值是非 Promise，那么内部把返回值包装成 Promise，如果返回值已经是 Promise，那么就直接返回。)

T -> Observable 更常见的一个地方是，用 Retrofit 定义的返回值为 Observable 的 API 请求，比如

    @GET("/users/{id}")
    Observable<User> getUserInfo(@Path("id") int id);

那么我们就应该用 flatMap 去取得这个 api 请求的结果，如下所示：

    Integers[] userIDs = new Integer[]{100, 120};
    Observable.from(userIDs)
            .flatMap((userID) -> { return apiService.getUserInfo(userID); })
            .map((userInfo) -> {...})

角度二：

虽然上例中，我们用 flatMap 替代 map 实现了相同的效果，但实际这是很愚蠢的做法，没有把 flatMap 用在真正需要的地方。这些抽象的目的就是让我们集中在实现最核心逻辑上，map 帮你做了把返回值包装成 Observable 的操作，你却还非得手动去做这个与核心逻辑无关的操作。

那么什么时候才是使用 flatMap 的正确时机呢，就是不得不用，其它变换都实现不了的时候。从字面上看，flat，拍扁，怎么可以算是拍扁，把对 array 的操作转换成对其中每一个元素的操作，就是一种拍扁。

上面的代码中，第一步就是通过 `Observable.from(numbers)`，把 array 转换成 Observable，这个 Observable 把 array 中的每一个元素都单独发射出去，进行后面的各种变换。array -> Observable，这就是 flatMap 最典型，最经典的用法。

`Observable.from(array)`，我们把一个数组转换成对其中每一个元素进行操作，如果这其中的每一个元素，又有自己的一个 array 或 list，我们想对这之中的每一个元素进行操作时，那么毫无疑问，这就是 flatMap 的用武之地了。

举个例子，我们有一些作者，这些作者每个人都出版了一些书，我们想看一下中国作者们写的书中，属于计算机类的那些书的评分。来看一下伪代码：

    Integer[] authorIDs = new Integer[]{1, 2, 3, 4, 5, 6, 7, 8, 9};
    Observable.from(authorIDs)                                   // 1
              .map(id -> apiService.getAuthorInfo(id))           // 2
              .filter(author -> author.country.equals("China"))  // 3
              .flatMap(author -> Observable.from(author.books))  // 4*
              .filter(book -> book.type.equals("computer"))      // 5
              .map(book -> book.rating)                          // 6

1. 把作者 ID 列表转换成对每一个作者 ID 进行操作的 Observable
1. 使用 map 变换，通过 id 得到单个作者的信息
1. 使用 filter 变换，过滤作者，只有中国的作者才可以继续下面的流程
1. 这时候我们拿到了作者信息，每个作者都有一个 bookList，我们没有办法继续对一个 list 进行操作，我们要的是把这个 list 取出来，对其中的每一本书继续进行操作，很显然，这是一个将 list 拍扁的过程，我们可以用 Observable.from(bookList) 来实现，而变换操作符当然是 flatMap。把第 4 步和第 1 步联系着看，可以加深你的理解。
1. 我们取到了单本 book，接着就可以过滤出计算机类的书，只有计算机类的书才可以继续后面的流程
1. 取得 book 的评分

而假设如果每一本书都有多个出版商，你想对这其中的每一个出版商进行一些操作，比如知道它们的名字，哈哈，那你知道该怎么办了，继续用 flatMap 啊。

    ...
    .flatMap(author -> Observable.from(author.books))
    .flatMap(book -> Observable.from(book.publishers))
    .map(publisher -> publisher.name)

总结一下就是，只要是 T -> Observable 的变换，就应该用 flatMap。
