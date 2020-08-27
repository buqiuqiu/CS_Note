https://blog.csdn.net/zouxinfox/article/details/5848519

确切的说，区域锁(Scoped locking)不是一种锁的类型，而是一种锁的使用模式(pattern)。这个名词是Douglas C. Schmidt于1998年在其论文Scoped Locking提出，并在ACE框架里面使用。但作为一种设计思想，这种锁模式应该在更早之前就被业界广泛使用了。
区域锁实际上是RAII模式在锁上面的具体应用。RAII(Resource Acquisition Is Initialization)翻译成中文叫“资源获取即初始化”，最早是由C++的发明者 Bjarne Stroustrup为解决C++中资源分配与销毁问题而提出的。RAII的基本含义就是：C++中的资源（例如内存，文件句柄等等）应该由对象来管理，资源在对象的构造函数中初始化，并在对象的析构函数中被释放。STL中的智能指针就是RAII的一个具体应用。RAII在C++中使用如此广泛，甚至可以说，不会RAII的裁缝不是一个好程序员。
### 未加区域锁：
先看看下面这段程序，Cache是一个可能被多个线程访问的缓存类，update函数将字符串value插入到缓存中，如果插入失败，则返回-1。
```cpp
Cache *cache = new Cache;
ThreadMutex mutex;
int update(string value)
{
	mutex.lock();
	if (cache == NULL)
	{
		mutex.unlock();
		return -1;
	}
	if (cache.insert(value) == -1)
	{
    mutex.unlock();
		return -1;
	}
	mutex.unlock();
	return 0;
}
```
 

从这个程序中可以看出，为了保证程序不会死锁，每次函数需要return时，都要需要调用unlock函数来释放锁。不仅如此，假设cache.insert(value)函数内部突然抛出了异常，程序会自动退出，锁仍然能不会释放。实际上，不仅仅是return，程序中的goto, continue, break语句，以及未处理的异常，都需要程序员检查锁是否需要显示释放，这样的程序是极易出错的。

同样的道理，不仅仅是锁，C++中的资源释放都面临同样的问题。例如前一阵我在阅读wget源码的时候，就发现虽然一共只有2万行C代码，但是至少有5处以上的return语句忘记释放内存，因此造成了内存泄露。

### 区域锁的实现
但是自从C++有了有可爱的RAII设计思想，资源释放问题就简单了很多。区域锁就是把锁封装到一个对象里面。锁的初始化放到构造函数，锁的释放放到析构函数。这样当锁离开作用域时，析构函数会自动释放锁。即使运行时抛出异常，由于析构函数仍然会自动运行，所以锁仍然能自动释放。一个典型的区域锁
```cpp
class Thread_Mutex_Guard 
{
public:
	Thread_Mutex_Guard (Thread_Mutex &lock)
	: lock_ (&lock) 
	{ 
		// 如果加锁失败，则返回-1
		owner_= lock_->lock(); 
	}

	~Thread_Mutex_Guard (void) 
	{
		// 如果锁获取失败，就不释放
		if (owner_ != -1)
			lock_->unlock ();
	}
private:
	Thread_Mutex *lock_;
	int owner_;
};
```

将策略锁应用到前面的update函数如下

```cpp 
Cache *cache = new Cache;
ThreadMutex mutex;
int update(string value)
{
    Thread_Mutex_Guard (mutex)
	if (cache == NULL)
	{
		return -1;
	}
	If (cache.insert(value) == -1)
	{
		return -1;
	}
	return 0;
}
```

基本的区域锁就这么简单。如果觉得这样锁的力度太大，可以用中括号来限定锁的作用区域，这样就能控制锁的力度。如下

 
{
	Thread_Mutex_Guard guard (&lock);
	...............
	// 离开作用域，锁自动释放
}


### 区域锁的改进方案
上面设计的区域锁一个缺点是灵活行，除非离开作用域，否则不能够显式释放锁。如果为一个区域锁增加显式释放接口，一个最突出的问题是有可能会造成锁的二次释放，从而引发程序错误。

例如
```cpp
{
	Thread_Mutex_Guard guard (&lock);
	If (…)
	{
		//显式释放（第一次释放）
		guard.release();
		// 自动释放(第二次释放)
		return -1;
    }
}
```

为了避免二次释放锁引发的错误，区域锁需要保证只能够锁释放一次。一个改进的区域锁如下：

```cpp 
class Thread_Mutex_Guard 
{
public:
	Thread_Mutex_Guard (Thread_Mutex &lock)
	: lock_ (&lock) 
	{ 
		acquire(); 
	}
	int acquire()
	{
		// 加锁失败，返回-1
		owner_= lock_->lock();
		return owner;
	}
	~Thread_Mutex_Guard (void) 
    {
	    release();
    }
    int release()
	{
		// 第一次释放
		if (owner_ !=  -1)
		{
			owner = -1;
			return lock_->unlock ();
        }
        // 第二次释放
        return 0;
	}
private:
	Thread_Mutex *lock_;
	int owner_;
};
```

可以看出，这种方案在加锁失败或者锁的多次释放情况下，不会引起程序的错误。

### 缺点：
区域锁固然好使，但也有不可避免的一些缺点

(1) 对于非递归锁，有可能因为重复加锁而造成死锁。

(2) 线程的强制终止或者退出，会造成区域锁不会自动释放。应该尽量避免这种情形，或者使用一些特殊的错误处理设计来确保锁会释放。

(3) 编译器会产生警告说有变量只定义但没有使用。有些编译器选项甚至会让有警告的程序无法编译通过。在ACE中，为了避免这种情况，作者定义了一个宏如下

`#define UNUSED_ARG(arg) { if (&arg) /* null */; }`

使用如下：
```
Thread_Mutex_Guard guard (lock_);
UNUSED_ARG (guard);
```

这样编译器就不会再警告了。