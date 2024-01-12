### 以下是自定义实现的一个线程池，如果需要更完善、更稳定的线程池实现，请使用libuv或其他第三方库

# 自定义简易线程池，注意编译时需要加链接库参数 -l pthread
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

typedef struct _PoolTask{
    int tasknum;                   // 模拟任务编号
    void *arg;                     // 回调函数参数
    void *(*task_func)(void *arg); // 任务的回调函数
}PoolTask;

typedef struct _ThreadPool{
    int max_job_num;               // 最大任务数
    int job_num;                   // 实际任务个数
    PoolTask *tasks;               // 任务队列数组
    int job_push;                  // 入队位置
    int job_pop;                   // 出队位置

    int thr_num;                   // 线程池内线程个数
    pthread_t *threads;            // 线程池内线程数组
    int shutdown;                  // 是否关闭线程池
    pthread_mutex_t pool_lock;     // 线程池的锁
    pthread_cond_t empty_task;     // 任务队列为空
    pthread_cond_t not_empty_task; // 任务队列不为空
}ThreadPool;


ThreadPool* create_threadpool(int thrnum, int maxtasknum);            // 创建线程池，参数是线程个数和任务队列数量

void destroy_threadpool(ThreadPool *pool);                            // 销毁线程池

void addtask(ThreadPool *pool, void *(*run)(void *), void *arg);      // 添加任务到线程池

void* taskRun(void *arg);                                             // 任务回调函数


ThreadPool* create_threadpool(int thrnum, int maxtasknum){

    printf("开始创建线程池，线程数量是%d, 任务队列长度是%d\n", thrnum, maxtasknum);

    ThreadPool *thrPool = (ThreadPool*)malloc(sizeof(ThreadPool));

    thrPool->thr_num=thrnum;
    thrPool->max_job_num=maxtasknum;

    thrPool->job_num=0;  // 初始任务数量是0
    thrPool->job_push=0; // 初始入队位置是0
    thrPool->job_pop=0;  // 初始出队位置是0
    thrPool->shutdown=0; // 线程池状态0表示使用中，1表示销毁

    thrPool->tasks = (PoolTask*)malloc(sizeof(PoolTask) * maxtasknum);

    // 初始化锁和条件变量
    pthread_mutex_init(&thrPool->pool_lock, NULL);
    pthread_cond_init(&thrPool->empty_task, NULL);
    pthread_cond_init(&thrPool->not_empty_task, NULL);

    thrPool->threads = (pthread_t*)malloc(sizeof(pthread_t) * thrnum);

    pthread_attr_t attr;
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    int i=0;

    for(i=0; i<thrnum; i++){
        pthread_create(&thrPool->threads[i], &attr, taskRun, thrPool);
    }

    return thrPool;
}


void addtask(ThreadPool *pool, void *(*run)(void *), void *arg){
    // 上互斥锁
    pthread_mutex_lock(&pool->pool_lock);

    if(pool->max_job_num <= pool->job_num){
        // 若任务数量超过最大任务数就等待，直到有空闲的任务数并且被其他线程唤醒
        pthread_cond_wait(&pool->empty_task, &pool->pool_lock);
    }

    int taskpos = (pool->job_push++)%pool->max_job_num;

    pool->tasks[taskpos].tasknum=taskpos;
    pool->tasks[taskpos].arg=arg;
    pool->tasks[taskpos].task_func=run;

    pool->job_num++;

    pthread_mutex_unlock(&pool->pool_lock);

    pthread_cond_signal(&pool->not_empty_task);
}


void* taskRun(void *arg){

    ThreadPool *pool=(ThreadPool*) arg;
    PoolTask *task= (PoolTask*)malloc(sizeof(PoolTask));

    while(1){

        pthread_mutex_lock(&pool->pool_lock);

        while(pool->job_num<=0 && !pool->shutdown){
            pthread_cond_wait(&pool->not_empty_task, &pool->pool_lock);
        }

        // 判断线程池是否销毁
        if(pool->shutdown){
            pthread_mutex_unlock(&pool->pool_lock);
            free(task);
            printf("线程%ld被销毁\n", pthread_self());
            pthread_exit(NULL);
        }

        // 任务可能在上述等待条件变量的过程中被其他线程处理了，判断任务是否还存在需要处理
        if(pool->job_num){
            int taskpos = (pool->job_pop++)%pool->max_job_num;

            memcpy(task, &pool->tasks[taskpos], sizeof(PoolTask));
            pool->job_num--;
            pthread_cond_signal(&pool->empty_task);
        }

        pthread_mutex_unlock(&pool->pool_lock);
        task->task_func(task->arg);
    }
}

void destroy_threadpool(ThreadPool *pool){
    pool->shutdown=1;  // 设置线程池关闭
    pthread_cond_broadcast(&pool->not_empty_task); // 唤醒所有的子线程执行清理工作

    //int i=0;
    //for(i=0;i<pool->thr_num;i++){
    //    pthread_join(pool->threads[i], NULL);
    //}

    // 回收锁和条件变量
    pthread_mutex_destroy(&pool->pool_lock);
    pthread_cond_destroy(&pool->empty_task);
    pthread_cond_destroy(&pool->not_empty_task);

    // 回收线程池空间
    free(pool->tasks);
    free(pool->threads);
    free(pool);
}

// 用户真正的任务函数
void* usertask(void * args){
    printf("哈哈%d\n", *(int*)args);
    free(args);
    return;
}

void main(){
    ThreadPool* thrPool = create_threadpool(5, 20);
    int i=0;
    for(i=0; i<100; i++){
        void* j = malloc(sizeof(int));
        memcpy(j, &i, sizeof(int));
        addtask(thrPool, usertask, j);
    }
    // 防止主线程退出导致子线程没有执行完任务就退出
    sleep(5);

    // 销毁线程池，回收各种资源
    destroy_threadpool(thrPool);
}
~~~
---