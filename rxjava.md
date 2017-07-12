## 使用lambda 

1. 在根build.gradle 中

    	dependencies {
	        classpath 'com.android.tools.build:gradle:2.3.2'
	        classpath 'me.tatarka:gradle-retrolambda:3.2.0'
   		 }

2. app build.gradle 中

		apply plugin: 'me.tatarka.retrolambda'

		android{
			...
			 compileOptions {
			        sourceCompatibility JavaVersion.VERSION_1_8
			        targetCompatibility JavaVersion.VERSION_1_8
			    }
			...
			}

3. 简单使用

		 btn_1 = (Button) findViewById(R.id.btn_1);
		 btn_1.setOnClickListener(v -> Toast.makeText(this, "我被点击了", Toast.LENGTH_SHORT).show());

		 Observable.just(1,2,3,4,5)
                        .map(integer -> "this is " + integer)
               			.subscribe(s -> Logger.d(s)); // 当传入的参数就是下一个方法要使用的参数时，可以写成下面这样
						.subscribe(Logger::d);



## Rxjava
### 转换类操作符
1. map (将一种流转换成另外一种流)

		Observable.just(1,2,3,4,5)
                        .map(integer -> "this is " + integer)
                        .subscribe(Logger::d);
2. flatmap (一对多的转换，并且转换之后的流是可观测的)

		ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            arrayList.add(i);
        }
        Observable.just(arrayList)
                .flatMap(Observable::from)
                .subscribe(Logger::d);

3. concatMap (解决了flatMap的交叉问题)

		使用flatmap 时 :
		String[][] arr = new String[10][10];
                for (int i = 0; i < 10; i++) {
                    for (int j = 0; j < 10; j++) {
                        arr[i][j] = i + "," + j;
                    }
                }
                Observable.from(arr)
                        .flatMap(Observable::from)
                        .subscribeOn(Schedulers.io())
                        .observeOn(AndroidSchedulers.mainThread())
                        .subscribe(s -> Log.e("mytag",s));
		
		输出 ： ... 7,8  7,9  9,0  9,1  9,2 ... 8,0  8,1 ....

		上面数据出现了交叉现象，即不是按照from 进来的顺序在进行flatmap 变换，顺序不可控制

		使用 concatMap 就能不会出现交叉现象
	      Observable.from(arr)
	                .concatMap(Observable::from)
	                .subscribeOn(Schedulers.io())
	                .observeOn(AndroidSchedulers.mainThread())
	                .subscribe(Logger::d);

4. flatMapIterable  (flatMapIterable和flatMap几乎是一样的，不同的是flatMapIterable它转化的多个Observable是使用Iterable作为源数据的。)

5. switchMap 

	> switchMap()和flatMap()很像，除了一点：每当源Observable发射一个新的数据项（Observable）时，它将取消订阅并停止监视之前那个数据项产生的Observable，并开始监视当前发射的这一个。

	    Observable.from(arr)
	            .switchMap(Observable::from)
	            .subscribeOn(Schedulers.io())
	            .observeOn(AndroidSchedulers.mainThread())
	            .subscribe(Logger::d);

	输出 0,1 ... 1,5 9,0 .....  
	当后面有新的数据源出来时，前面的将被取消，只能保证最后一个数据源是完整被订阅的，前面的数据源是不可控的，可能走了一半，也可能没开始

6. scan 
	>  scan()对一个序列的数据应用一个函数，并将这个函数的结果发射出去作为下个数据应用合格函数时的第一个参数使用。

		Observable.just(1, 2, 3, 4, 5)
                        .scan((integer, integer2) -> {
                            Logger.e("integer = " + integer + " integer2 =" + integer2);
                            return integer + integer2;
                        })
                        .map(integer -> {
                            Logger.e("map 中的数据" + integer);
                            return integer;
                        })
                        .subscribe(integer -> Logger.e("输出的数据"+integer));
		输出结果:

		map 中的数据1
		输出的数据1
		
		integer = 1 integer2 =2
		map 中的数据3
		输出的数据3
		
		integer = 3 integer2 =3
		map 中的数据6
		输出的数据6
		
		integer = 6 integer2 =4
		map 中的数据10
		输出的数据10
		
		integer = 10 integer2 =5
		map 中的数据15
		输出的数据15


		从输出结果来看，数据源第一次到 scan 时并没有执行里面的代码，而是直接进入到map 中，并且最终被订阅的数据是1 , 第二次，则进入sacn 中，并且第一个参数是上次计算的结果 1 ,第二个参数是第二次传入的参数


### 过滤类操作符

1. filter
	> 用来过滤观测序列中我们不想要的值，只返回满足条件的值
	
		Observable.just(1, 2, 3, 4, 5)
                        .filter(integer -> integer > 3)
                        .map(integer -> integer+"")
                        .subscribe(Logger::e);
		输出  4  5 
2. take
	> 用一个整数n作为一个参数，从原始的序列中发射前n个元素
	
		Observable.just(1, 2, 3, 4, 5)
	                        .take(3)
	                        .map(integer -> integer+"")
	                        .subscribe(Logger::e);

		输出 1  2  3

3. takeLast
	> 用一个整数n作为参数，只不过它发射的是观测序列中后n个元素

		Observable.just(1, 2, 3, 4, 5)
                        .takeLast(3)
                        .map(integer -> integer+"")
                        .subscribe(Logger::e);
		输出  3  4  5

4. takeUntil

	> takeUntil(Observable)订阅并开始发射原始Observable，同时监视我们提供的第二个Observable。如果第二个Observable发射了一项数据或者发射了一个终止通知，takeUntil()返回的Observable会停止发射原始Observable并终止。

		Observable.interval(300,TimeUnit.MILLISECONDS)
                        .takeUntil(Observable.interval(900,TimeUnit.MILLISECONDS))
                        .map(aLong -> aLong+"")
                        .subscribe(Logger::e);

		输出 0 1
		分析 ： 原始Observable 每300 毫秒发射一次数据，当takeUtil  中的Obserble 开始发射数据时，停掉原来的Observable

	> 当takeUtil 接收一个Func1 时，就表示终止条件

		   Observable.interval(300,TimeUnit.MILLISECONDS)
                        .takeUntil(aLong -> aLong > 5)
                        .map(aLong -> aLong+"")
                        .subscribe(Logger::e);
			输出 0 1 2 3 4 5 6
			注意，这里输出了6，即终止条件也会发射. 注意这里是终止条件，要与 filter 区别,这里表示 大于5就终止，并且包含令他终止的条件6， 在filter 中表示留下大于5的数据，丢弃小于等于 5 的数据


5. skip
	> 忽略Observable发射的前n项数据

		Observable.interval(300,TimeUnit.MILLISECONDS)
	                        .takeUntil(aLong -> aLong > 5)
	                        .skip(3)
	                        .map(aLong -> aLong+"")
	                        .subscribe(Logger::e);
		输出 3 4 5 6 

6. skipLast
	> 忽略发射的后n项数据

		Observable.interval(300,TimeUnit.MILLISECONDS)
                        .takeUntil(aLong -> aLong > 5)
                        .skipLast(3)
                        .map(aLong -> aLong+"")
                        .subscribe(Logger::e);
		输出 0 1 2 3

7. elementAt
	> 用来获取元素Observable发射的事件序列中的第n项数据，并当做唯一的数据发射出去。

		Observable.interval(300,TimeUnit.MILLISECONDS)
                        .takeUntil(aLong -> aLong > 5)
                        .elementAt(4)
                        .map(aLong -> aLong+"")
                        .subscribe(Logger::e);
		输出 4  注意，这里数据是从 0 开始的

8. distinct
	> 去重
	
		Observable.just(1,2,3,4,5,5,6,8,4,2)
		                        .distinct()
		                        .map(integer -> integer+"")
		                        .subscribe(Logger::e);
		输出 1,2,3,4,5,6,8

		  ArrayList<UserBean> list = new ArrayList<>();
            list.add(new UserBean(1,"张三"));
            list.add(new UserBean(1,"李四"));
            list.add(new UserBean(1,"王五"));
            list.add(new UserBean(2,"啦"));
            list.add(new UserBean(3,"s"));
            list.add(new UserBean(3,"计划"));

            Observable.from(list)
                    .distinct(UserBean::getGroup) // 判断重复的条件
                    .map(UserBean::getName)
                    .subscribe(Logger::e);
			输出  张三，啦，s，计划

9. distinctUntilChanged
	> 判断当前数据和前一个发射的数据是否相同,如果相同就不发射

	 	Observable.just(1, 1, 2, 3, 2, 3, 3, 5, 6, 6)
                        .distinctUntilChanged()
                        .map(integer -> integer + "")
                        .subscribe(Logger::e);
		输出 ： 1,2,3,2,3,5,6

		ArrayList<UserBean> list = new ArrayList<>();
                list.add(new UserBean(1, "张三"));
                list.add(new UserBean(1, "李四"));
                list.add(new UserBean(1, "王五"));
                list.add(new UserBean(2, "啦"));
                list.add(new UserBean(3, "s"));
                list.add(new UserBean(3, "我"));
                list.add(new UserBean(2, "切片"));
                list.add(new UserBean(3, "计划"));

                Observable.from(list)
                        .distinctUntilChanged(UserBean::getGroup)
                        .map(UserBean::getName)
                        .subscribe(Logger::e);
			输出 ： 张三 啦  s  切片 计划

10. first
	> 只取满足条件的第一个，如果没有条件直接取第一个

		ArrayList<UserBean> list = new ArrayList<>();
                list.add(new UserBean(1, "张三"));
                list.add(new UserBean(1, "李四"));
                list.add(new UserBean(1, "王五"));
                list.add(new UserBean(2, "啦"));
                list.add(new UserBean(3, "s"));
                list.add(new UserBean(3, "我"));
                list.add(new UserBean(2, "切片"));
                list.add(new UserBean(3, "计划"));

                Observable.from(list)
                        .first(userBean -> userBean.getGroup() == 2)
                        .map(UserBean::getName)
                        .subscribe(Logger::e);
			输出  啦

11. last 
	> 与first 相反 只发射满足条件的最后一个

12. Debounce

### 组合类操作符
1. merge 
	> 将两个Observable发射的事件序列组合并成一个事件序列，就像是一个Observable发射的一样。你可以简单的将它理解为两个Obsrvable合并成了一个Observable，合并后的数据是无序的


			ArrayList<String> list = new ArrayList<>();
                list.add("A");
                list.add("B");
                list.add("C");
                list.add("D");
                list.add("E");
                list.add("F");
                list.add("G");
                list.add("H");
                Observable<String> stringObservable = Observable.interval(300, TimeUnit.MILLISECONDS)
                        .map(aLong -> list.get(aLong.intValue()))
                        .take(list.size());
                Observable<Long> longObservable = Observable.interval(500, TimeUnit.MILLISECONDS)
                        .take(20);
                Observable.merge(stringObservable, longObservable)
                        .subscribeOn(Schedulers.io())
                        .subscribe(new Subscriber<Serializable>() {
                            @Override
                            public void onCompleted() {

                            }

                            @Override
                            public void onError(Throwable e) {

                            }

                            @Override
                            public void onNext(Serializable serializable) {
                                if (serializable instanceof String) {
                                    Logger.e("我是String::" + serializable);
                                } else if (serializable instanceof Long) {
                                    Logger.e("我是Long::" + serializable);
                                }
                            }
                        }); 


2. startWith
	> 用于在源Observable发射的数据前插入数据,并且可以调节线程

 		Observable.just(1, 2, 3, 4, 5)
                        .subscribeOn(Schedulers.io()) 
                        .filter(integer -> {
                            Logger.e("subscribe"+integer);
                            return true;
                        })
                        .observeOn(AndroidSchedulers.mainThread())
                        .startWith(-1, -2, -3)  // 在源之前发射，并且可以指定到主线程
                        .filter(integer -> {
                            Logger.e("startWith"+integer);
                            return true;
                        })
                        .observeOn(AndroidSchedulers.mainThread())
                        .map(integer -> integer + "")
                        .subscribe(Logger::e);

		输出 : startWith-1  startWith-2  startWith-3  subscribe1 subscribe2 -1 -2 -3 subscribe3 subscribe4 subscribe5 1 2 3 4 5 
3. concat
	> 用于将多个obserbavle发射的的数据进行合并发射，concat严格按照顺序发射数据，前一个Observable没发射完是不会发射后一个Observable的数据的

		// 还是merge 那个例子 只是换成了concat
		// 与上面两种的区别
		// 1. merge 无序，concat 有序
		// 2. startWith只能在发射之前插入数据
		 ArrayList<String> list = new ArrayList<>();
                list.add("A");
                list.add("B");
                list.add("C");
                list.add("D");
                list.add("E");
                list.add("F");
                list.add("G");
                list.add("H");
                Observable<String> stringObservable = Observable.interval(300, TimeUnit.MILLISECONDS)
                        .map(aLong -> list.get(aLong.intValue()))
                        .take(list.size());
                Observable<Long> longObservable = Observable.interval(500, TimeUnit.MILLISECONDS)
                        .take(20);
                Observable.concat(stringObservable, longObservable)
                        .subscribeOn(Schedulers.io())
                        .subscribe(new Subscriber<Serializable>() {
                            @Override
                            public void onCompleted() {

                            }

                            @Override
                            public void onError(Throwable e) {

                            }

                            @Override
                            public void onNext(Serializable serializable) {
                                if (serializable instanceof String) {
                                    Logger.e("我是String::" + serializable);
                                } else if (serializable instanceof Long) {
                                    Logger.e("我是Long::" + serializable);
                                }
                            }
                        });
			输出： A B C D .... 1 2 3 ...

4. zip
	> 用来合并两个Observable发射的数据项，根据Func2函数生成一个新的值并发射出去。当其中一个Observable发送数据结束或者出现异常后，另一个Observable也将停止发射数据


		  ArrayList<String> list = new ArrayList<>();
                list.add("A");
                list.add("B");
                list.add("C");
                list.add("D");
                list.add("E");
                list.add("F");
                list.add("G");
                list.add("H");
                Observable<String> stringObservable = Observable.interval(300, TimeUnit.MILLISECONDS)
                        .map(aLong -> list.get(aLong.intValue()))
                        .take(list.size());
                Observable<Long> longObservable = Observable.interval(500, TimeUnit.MILLISECONDS)
                        .take(20);
                Observable.zip(stringObservable, longObservable, (s, along) -> s + "-----" + along)
                        .subscribe(Logger::e);

			输出 A---1  B---2 .....  H---9

5. CombineLatest (没看懂)

6. SwitchOnNext
7. join