---
layout: post
title: C++ Singleton 以及 Double-Checked Locking Pattern(DCLP)
date: 2016-10-03 16:27:37
updated: 2016-10-04 00:27:37
tags: ["Singleton", "C++"]
---

  从之前 wordpress 博客搬运过来的，格式有点糟 = ,=
<!-- more -->

 <h3>最简单的实现</h3>
 <pre class="lang:c++ decode:true">template&lt;typename T&gt;
 class Singleton {
   static T&amp; getInstance() {
     static T instance;
     return instance;
   }
 };</pre>
 此实现在单线程下是没有任何问题的，但在 C++ 03 多线程环境则不行。<code>static T instance; </code>在第一次调用时才会进行初始化，而在多线程中会发生于初始化有关的 race conditions，根据《Effective C++》，一个消除的方法是在程序的单线程启动阶段手工调用所有<code>getInstance();</code>。不过，C++ 11 标准 6.7 节提到：
 <blockquote>An implementation is permitted to perform early initialization of other block-scope variables with static or thread storage duration under the same conditions that an implementation is permitted to statically initialize a variable with static or thread storage duration in namespace scope (3.6.2). Otherwise such a variable is initialized the first time control passes through its declaration; such a variable is considered initialized upon the completion of its initialization. If the initialization exits by throwing an exception, the initialization is not complete, so it will be tried again the next time control enters the declaration. <strong>If control enters the declaration concurrently while the variable is being initialized, the concurrent execution shall wait for completion of the initialization.</strong></blockquote>
 <blockquote>Footnote: The implementation must not introduce any deadlock around execution of the initializer.</blockquote>
 即在 C++ 11 中，<code>instance</code>是线程安全的。因此，在 C++ 11 多线程环境中在保证所有对<code>instance</code>的使用都是线程安全的前提下，使用此实现是安全的（然而由于其极其简单以至于有些人并不认为是个单例的样子 o .o）。
 另外，GCC提供了 <code>--fno-threadsafe-statics</code> 编译选项用于去除局部静态变量初始化过程线程安全的保证：
 <blockquote>Do not emit the extra code to use the routines specified in the C++ ABI for thread-safe initialization of local statics. You can use this option to reduce code size slightly in code that doesn't need to be thread-safe.</blockquote>
 当然，将<code>instance</code>作为类的静态成员函数也是一种做法。唔，或许到这里就不必往下看了，C++ 11 已经做得足够好了。但是，如果使用局部静态对象的话，其析构时间是不确定的，当其他静态对象析构时有可能会访问这个已经析构的对象，从而出现 Bug 。因此，另一个做法是使用指针来实现 Singleton，<code>getInstance()</code>返回实例的指针 --《设计模式》一书返回的就是指针吖，于是就有一下内容。
 <h3>Double-Checked Locking Pattern (DCLP) -- 双重检查锁定模式</h3>
 先说 C++ 03 的情况，即函数局部静态变量初始化的线程安全没有保证，一个可能实现如下：
 <pre class="lang:c++ decode:true ">template&lt;typename T&gt;
 class Singleton {
   static T* getInstance() {
     static T* instance = new T;
     return instance;
   }
 };
 // 相当于：
 template&lt;typename T&gt;
 class Singleton {
   static T* getInstance() {
     if(pInstance  == NULL) {
       pInstance  = new T;
     }
     return pInstance;
   }
   static T* pInstance;
 };
 
 template&lt;typename T&gt;
 T* Singleton&lt;T&gt;::pInstance = NULL;</pre>
 多线程环境下可能发生如下情况：
 Thread1: 读取<code>pInstance</code>，为<code>NULL</code>
 Thread2: 读取<code>pInstance</code>，为<code>NULL</code>
 Thread1: 调用 <code>new T;</code>
 Thread2: 调用 <code>new T;</code>
 明显不行。
 若是进行加锁：
 <pre class="lang:c++ decode:true ">static T* getInstance() {
   Lock lock;  // RAII
   if(pInstance  == NULL) {
     pInstance  = new T;
   }
   return pInstance;
 }</pre>
 除了第一次进行初始化时发挥作用外，其余每次调用<code>getInstance()</code>时，都会导致不必要的开销。
 如果将函数内的局部静态成员移动到
 于是有人研究出 DCLP 进行改进，以将降低比不要的开销：
 <pre class="lang:c++ decode:true ">static T* getInstance() {
   if(pInstance  == NUL) {
     Lock lock;  // RAII
     if(pInstance  == NULL) {
       pInstance  = new T;
     }
   }
   return pInstance;
 }</pre>
 看起来很巧妙，但实际上由于C++编译器生成的代码以及CPU乱序执行的问题，这样的实现是有问题的，```pInstance = new T;```，实际包含以下步骤：
 1. 分配内存用于保存对象
 2. 在分配的内存上构造对象
 3. 将 <code>pInstance </code>指针指向分配的内存地址
 但是，有可能编译出来的代码相当于：
 <pre class="lang:c++ decode:true ">static T* getInstance() {
   if(pInstance  == NUL) {
     Lock lock;  // RAII
     if(pInstance  == NULL) {
       pInstance =                   // 3
         operator new(sizeof(T));    // 1
       new (pInstance) T;            // 2
     }
   }
   return pInstance;
 }</pre>
 也就是有可能线程1在未构造完对象时，先将内存地址赋值到<code>pInstance</code>，然后线程2并发调用<code>getInstance()</code>得到一个尚未构造好的对象的地址。
 
 一个解决方案是使用CPU提供的 barrier 指令：
 <pre class="lang:c++ decode:true ">T* T::instance () {
   if (pInstance == NULL) {
     Lock lock;
     if (pInstance== NULL) {
       T *tmp = new T;
       ... // 插入 memory barrier
       pInstance = tmp;
   }
   return pInstance;
 }</pre>
 然而，要为各个CPU、编译器平台写不同的 barrier 指令是件麻烦的事，要是用 C++ 11 实现 DCLP 会不会简单点？答案是肯定的。
 <pre class="lang:c++ decode:true ">std::atomic&lt;Singleton*&gt; Singleton::m_instance;
 std::mutex Singleton::m_mutex;
  
 Singleton* Singleton::getInstance() {
     Singleton* tmp = m_instance.load(std::memory_order_relaxed);
     std::atomic_thread_fence(std::memory_order_acquire);
     if (tmp == nullptr) {
         std::lock_guard&lt;std::mutex&gt; lock(m_mutex);
         tmp = m_instance.load(std::memory_order_relaxed);
         if (tmp == nullptr) {
             tmp = new Singleton;
             std::atomic_thread_fence(std::memory_order_release);
             m_instance.store(tmp, std::memory_order_relaxed);
         }
     }
     return tmp;
 }</pre>
 而经过在VS2015上对“最简单的实现”反汇编跟踪，可以发现编译器也运用了DCLP以及会插入类似的代码，即<code>__Init_thread_header();</code>和<code>__Init_thread_footer();</code>以保证初始化过程的线程安全。若是C++ 11中，在类的静态函数内声明<code>static T* instance = new T;</code>除了delete存在问题外，此实现也是可以的；如果将 <span class="lang:c++ decode:true crayon-inline ">static T* instance;</span> 的声明移到函数外（类内），则手动DCLP还是有必要的。
 最后，值得一提的是，C++他爹在《C++ Core Guidelines》中提到要 avoid singletons o .o。
 <h3>参考资料：</h3>
 <a href="http://www.aristeia.com/Papers/DDJ_Jul_Aug_2004_revised.pdf" target="_blank">C++ and the Perils of Double-Checked Locking</a>
 
 <a href="https://github.com/isocpp/CppCoreGuidelines" target="_blank">C++ Core Guidelines</a>
 
 <a href="http://preshing.com/20130930/double-checked-locking-is-fixed-in-cpp11/" target="_blank">Double-Checked Locking is Fixed In C++11</a>
 
 <a href="https://en.wikipedia.org/wiki/Double-checked_locking" target="_blank">Double-checked locking</a>