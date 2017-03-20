---
title: RxJava实际应用一

date: 2016-11-22 19:50:15
categories:
- Android技术
tags:
- 框架
---

RxJava一直不知庐山真面目，不知道他的强大到底在哪里，针对此，根据实际使用来说明一下，其实初学RxJava只要把握两点： ***观察者模式*** 和 ***异步***。

# RxJava的操作符
RxJava操作符是在事件传递过程中做的一些鬼斧神工的操作

## Map操作

应用示例：比如被观察者产生的事件中只有图片文件路径,但是在观察者这里只想要bitmap,那么就需要类型变换

```Java
Observable.create(new Observable.just(getFilePath()))
				//使用map操作来完成类型转换
				.map(new Func1<String, Bitmap>() {
					@Override
					public Bitmap call(String s) {
						return ImageUtils.getImageByFile(s);
					}
				})
				.subscribe(new Subscriber<Bitmap>() {
					@Override
					public void onCompleted() {
						
					}

					@Override
					public void onError(Throwable e) {

					}

					@Override
					public void onNext(Bitmap bitmap) {
						imageView.setImageBitmap(bitmap);
					}
				});
```
 - 实际上在使用map操作时，new Func1() 就对应了类型的转你方向，String是原类型，Bitmap是转换后的类型。在call()方法中，输入的是原类型，返回转换后的类型

读取文件，创建bitmap可能是一个耗时操作，那么就应该在子线程中执行，主线程应该仅仅做展示。那么线程切换一般就会是比较复杂的事情了。但是在Rxjava中，是非常方便的。

```Java
Observable.create(new Observable.just(getFilePath()))
				//制定了被观察者执行的线程环境
				.subscribeOn(Schedulers.newThread())
				//将接下来执行的线程环境指定为io线程
				.observeOn(Schedulers.io())
				//使用map操作来完成类型转换
				.map(new Func1<String, Bitmap>() {
					@Override
					public Bitmap call(String s) {
						return ImageUtils.getImageByFile(s);
					}
				})
				//将后面执行的线程环境切换为主线程
				.observeOn(AndroidSchedulers.mainThread())
				.subscribe(new Subscriber<Bitmap>() {
					@Override
					public void onCompleted() {

					}

					@Override
					public void onError(Throwable e) {

					}

					@Override
					public void onNext(Bitmap bitmap) {
						imageView.setImageBitmap(bitmap);
					}
				});
```

由上面的代码可以看到，使用操作符将事件处理逐步分解，通过线程调度为每一步设置不同的线程环境，完全解决了你线程切换的烦恼。可以说线程调度+操作符，才真正展现了RxJava无与伦比的魅力。

如上，线程调度只用到subscribeOn（）和observeOn（）两个方法。对于初学者，只需要掌握两点：

subscribeOn（）它指示Observable在一个指定的调度器上创建（只作用于被观察者创建阶段）。只能指定一次，如果指定多次则以第一次为准

observeOn（）指定在事件传递（加工变换）和最终被处理（观察者）的发生在哪一个调度器。可指定多次，每次指定完都在下一步生效。

## flatmap操作
应用场景：查找一个学校每个班级的每个学生，并打印出来。

如果用老办法：先读出所有班级的数据，循环每个班级。再循环中再读取每个班级中每个学生，然后循环打印出来。

这种想法，虽然没毛病，就是嵌套得有点多。

那让我们看看RxJava是怎么做的：

```Java
private SchoolClass[] mSchoolClasses=new SchoolClass[2];

		Observable.from(mSchoolClasses)
				.flatMap(new Func1<SchoolClass, Observable<Student>>() {
					@Override
					public Observable<Student> call(SchoolClass schoolClass) {
						//将Student列表使用from方法一个一个发出去
						//将每个班级的所有学生作为一列表包装成一列Observable<Student>
						return Observable.from(schoolClass.getStudents());
					}
				})
				.subscribe(new Action1<Student>() {
					@Override
					public void call(Student student) {
						mText.append("打印单个学生信息：\n");
						mText.append("name:"+student.name+"    age: "+student.age+"\n");
					}
				});


		class SchoolClass{
			Student[] stud;
			public SchoolClass(Student[] s){
				this.stud=s;
			}
			public Student[] getStudents(){
				return  stud;
			}
		}

		class Student{
			String name;
			String age;
			public Student(String name,String age){
				this.name=name;
				this.age=age;
			}
		}

```

RxJava此时对嵌套嘿嘿一笑。其实flatmap操作符将每个Observable产生的事件里的信息再包装成新的Observable传递出来.那么它是怎么破除嵌套难题？

因为FlatMap可以再次包装新的Observable,而每个Observable都可以使用from(T[])方法来创建自己，这个方法接受一个列表，然后将列表中的数据包装成一系列事件。


# 异步（线程调度）

| 调度器类型        | 效果          |
| ------------- | -----:|
| Schedulers.computation()     | 用于计算任务，如事件循环或回调处理，不用用于IO操作，默认线程数等于处理器的数量 | 
| Schedulers.from(execulor)    | 使用指定的Executor作为调度器      |  
| Schedulers.immediate() | 在当前线程立即开始执行任务      |  
| Schedulers.io() | 用于IO密集型任务，如异步阻塞IO操作，这个调度器的线程池会更具需要增长；对于普通的计算任务，请使用computation（）；Schedulers.io()默认是一个CachedThreadScheduler，很像一个有线程缓存的新线程调度器      |  
| Schedulers.newThread() | 为每一个任务创建一个新线程     |  
| Schedulers.trampoline() | 当其他排队的任务完成后，在当前线程排队开始执行      |  


