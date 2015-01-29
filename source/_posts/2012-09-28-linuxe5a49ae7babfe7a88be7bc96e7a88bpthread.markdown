---
author: admin
comments: true
date: 2012-09-28 03:24:16+00:00
layout: post
slug: linux%e5%a4%9a%e7%ba%bf%e7%a8%8b%e7%bc%96%e7%a8%8bpthread
title: linux多线程编程pthread
wordpress_id: 527
categories:
- IT
- linux
tags:
- linux
- 多线程
---

在linux下面使用多线程编程要用到pthread.h头文件，因为linux是不存在线程概念的，所以它其实是用进程模拟线程，产生出线程的效果。即linux中的多线程实际上是多进程。





那么为什么linux下有了进程还要再引入线程呢？原因有很多，首先，线程小巧，不需要单独分配地址空间，也不需要维护各种数据表单，然后线程间共享数据，通信起来非常方便.





不过，正因为linux下线程共享数据，如果线程胡乱修改数据的话，会出现一些灾难性的错误，所以就需要操纵系统中所强调的同步、互斥机制来控制。





除了上述一些区别，进程和线程还有如下共同的一些优点：
1) 提高应用程序响应。这对图形界面的程序尤其有意义，当一个操作耗时很长时，整个系统都会等待这个操作，此时程序不会响应键盘、鼠标、菜单的操作，而使用多线程技术，将耗时长的操作（time consuming）置于一个新的线程，可以避免这种尴尬的情况。
2) 使多CPU系统更加有效。操作系统会保证当线程数不大于CPU数目时，不同的线程运行于不同的CPU上。
3) 改善程序结构。一个既长又复杂的进程可以考虑分为多个线程，成为几个独立或半独立的运行部分，这样的程序会利于理解和修改。





下面就进入正题，开始linux下的多线程编程了。




{% codeblock %}

#include 
#include 
#include 

void *print_message_function( void *ptr );

main()
{
pthread_t thread1, thread2;
char *message1 = "Thread 1";
char *message2 = "Thread 2";
int iret1, iret2;

/* Create independent threads each of which will execute function */

iret1 = pthread_create( &thread1;, NULL, print_message_function, (void*) message1);
iret2 = pthread_create( &thread2;, NULL, print_message_function, (void*) message2);

/* Wait till threads are complete before main continues. Unless we */
/* wait we run the risk of executing an exit which will terminate */
/* the process and all threads before the threads have completed. */

pthread_join( thread1, NULL);
pthread_join( thread2, NULL);

printf("Thread 1 returns: %d\n",iret1);
printf("Thread 2 returns: %d\n",iret2);
exit(0);
}

void *print_message_function( void *ptr )
{
char *message;
message = (char *) ptr;
printf("%s \n", message);
}

{% endcodeblock %}




编译: C++ compiler: g++ -lpthread pthread.cpp
运行: ./a.out
结果:
Thread 1
Thread 2
Thread 1 returns: 0
Thread 2 returns: 0





pthread_create函数如下：
int pthread_create(pthread_t * thread, const pthread_attr_t * attr, void * (*start_routine)(void *), void *arg);
第一参数是线程id，第二参数是线程的一些参数包括栈大小等，一般缺省设置为default，第三个参数是线程运行的函数，可以理解为当前线程的主函数定义，第四个参数就是传给注函数的参数了。返回值是一个整型，是0的话，表示创建线程成功，否则就是失败了。





pthread_join函数如下：
int pthread_join(pthread_t th, void **thread_return);
第一个参数是线程编号，第二个参数是线程返回值，如果有返回值的话。此函数的返回值同上，0表示正常结束。





同步互斥机制：
linux下面提供3个关于同步互斥的库工具，包括mutexes,joins和condition variables.





先说mutex（互斥）：




{% codeblock %}

/* Note scope of variable and mutex are the same */
pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
int counter=0;

/* Function C */
void functionC()
{
pthread_mutex_lock( &mutex1; );
counter++
pthread_mutex_unlock( &mutex1; );
}

{% endcodeblock %}




用之前要先初始化，然后加锁就是pthread_mutex_lock( &mutex1 );解锁就是pthread_mutex_unlock( &mutex1 );
显然，mutex1要设置到全局去。





注意：这里的mutex1实在定义的时候初始化的，如果你的程序要运行多次，那么在每次主程序开启所有线程之前，要对mutex都重新初始化一遍，防止出现运行过程中mutex加锁，然后程序结束却没有解锁的情况出现。当然，如果每次操作都注意加锁解锁必定会在程序的所有流程中成对出现，那就无所谓了。





另外，关于mutex还有一个函数：
pthread_mutex_trylock() – 尝试互斥量是否已经被锁，这个函数可以检测互斥量，防止出现死线程。（互斥量一开始就是锁的，线程死等互斥量解锁）





join就是等待线程结束，用法在一开始已经写过。





最后说一下最有意思的Condition Variables(条件变量)
创建和销毁条件变量:
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;//最常用的
int pthread_cond_destroy(pthread_cond_t *cond);
等待条件信号:
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
//每个条件信号量都是和一个互斥量关联到一起的，因为我要保证唤醒的时候唤醒的是当前我这个线程，有了互斥量就只有我这个线程进入了，所以就保证了唯一性
int pthread_cond_timedwait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restrict abstime);
//和上一个函数的区别就是加了个时间，时间超过了以后就不等了，返回一个非0整数，表示超时。另外由于创建这个struct timespec需要用到一个时间函数，在编译的时候需要加上-lrt参数，不然链接的时候会出错，即“g++ -lpthread -lrt ex.cpp”
唤醒线程:
int pthread_cond_broadcast(pthread_cond_t *cond);//唤醒所有等待这个信号量的线程
int pthread_cond_signal(pthread_cond_t *cond);//唤醒单个等待这个信号量的线程




{% codeblock %}

#include 
#include 
#include 

pthread_mutex_t count_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t condition_var = PTHREAD_COND_INITIALIZER;

void *functionCount1();
void *functionCount2();
int count = 0;
#define COUNT_DONE 10
#define COUNT_HALT1 3
#define COUNT_HALT2 6

main()
{
pthread_t thread1, thread2;

pthread_create( &thread1;, NULL, &functionCount1;, NULL);
pthread_create( &thread2;, NULL, &functionCount2;, NULL);

pthread_join( thread1, NULL);
pthread_join( thread2, NULL);

printf("Final count: %d\n",count);

exit(0);
}

// Write numbers 1-3 and 8-10 as permitted by functionCount2()

void *functionCount1()
{
for(;;)
{
// Lock mutex and then wait for signal to relase mutex
pthread_mutex_lock( &count;_mutex );

// Wait while functionCount2() operates on count
// mutex unlocked if condition varialbe in functionCount2() signaled.
pthread_cond_wait( &condition;_var, &count;_mutex );
count++;
printf("Counter value functionCount1: %d\n",count);

pthread_mutex_unlock( &count;_mutex );

if(count >= COUNT_DONE) return(NULL);
}
}

// Write numbers 4-7

void *functionCount2()
{
for(;;)
{
pthread_mutex_lock( &count;_mutex );

if( count < COUNT_HALT1 || count > COUNT_HALT2 )
{
// Condition of if statement has been met.
// Signal to free waiting thread by freeing the mutex.
// Note: functionCount1() is now permitted to modify "count".
pthread_cond_signal( &condition;_var );
}
else
{
count++;
printf("Counter value functionCount2: %d\n",count);
}

pthread_mutex_unlock( &count;_mutex );

if(count >= COUNT_DONE) return(NULL);
}

}

{% endcodeblock %}




编译: g++ -lpthread pthread.cpp
运行: ./a.out
显示:





Counter value functionCount1: 1
Counter value functionCount1: 2
Counter value functionCount1: 3
Counter value functionCount2: 4
Counter value functionCount2: 5
Counter value functionCount2: 6
Counter value functionCount2: 7
Counter value functionCount1: 8
Counter value functionCount1: 9
Counter value functionCount1: 10
Final count: 10





另外记录一下关于N个线程同步的peterson算法：




{% codeblock %}

#define FALSE 0
#define TRUE 1

int turn;
int interested[2];

void enterRegion(int process)
{
int other = 1 – process;
interested[process] = TRUE;
turn = process;

while (interested[other] == TRUE && turn == process)
;
}

void leaveRegion(int process)
{
interested[process] = FALSE;
}

{% endcodeblock %}



{% codeblock %}

#define FALSE 0
#define N 10 /* First process is indicated with 1, not 0 */

int turn[N];
int stage[N + 1];

void enterRegion(int process)
{
int i, j;

for (i = 1; i <= N – 1; i++) {
stage[process] = i;
turn[i] = process;
for (j = 1; j <= N; j++) {
if (j == process)
continue;
while (stage[j] >= i && turn[i] == process)
;
}
}
}

void leaveRegion(int process)
{
stage[process] = FALSE;
}

{% endcodeblock %}


