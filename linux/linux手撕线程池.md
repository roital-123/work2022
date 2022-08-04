[TOC]

# 线程池实现思想

- 任务队列：

- 线程池结构：

  ```c
  struct threadpool_t {
  	pthread_mutex_t lock;				/*当前结构体的锁*/
      pthread_mutex_t thread_counter;		/*记录忙线程个数的锁--busy_thr_num*/
      pthread_cond_t queue_not_full;		/*当任务队列满，添加任务线程，阻塞等待次条件变量*/
      pthread_cond_t queue_not_empty;		/*当任务队列不空，通知等待处理任务的线程*/
      
      pthread_t *threads;					/*线程池中存储线程id的数组*/
      pthread_t adjust_thread;			/*管理者线程*/
      thread_task_t *task_queue;			/*任务队列，记录任务的回调函数*/
      
      int min_thr_num;					/*线程池中最小线程数*/
      int max_thr_num;					/*线程池中最大线程数*/
      int live_thr_num;					/*线程池中当前存活线程数*/
      int busy_thr_num;					/*线程池中忙线程数*/
      int wait_exit_thr_num;				/*线程池中等待销毁线程数*/
      
      int queue_front;					/*任务队列队头下标*/
      int queue_rear;						/*任务队列队尾下标*/
      int queue_size;						/*当前任务队列大小*/
      int queue_max_size;					/*任务队列可容纳任务上限*/
      
      int shutdowm;
  };

- 管理者线程：

  > 线程扩容：
  >
  > 线程缩减：

- 锁以及条件变量

  > `lock`	锁，获取线程池结构体信息需要加锁；
  >
  > `pthread_cond_not_empty`	从任务队列中取任务时，当队列为空时阻塞
  >
  > `pthread_cond_not_full`	向任务队列中添加任务时当队列满时阻塞

- 实现

```c
// threadpool.c
#define true 1
#definr false 0

typedef struct {
	void* (*function)(void*);	/*回调函数*/
    void* arg;					/*回调函数参数*/
}threadpool_task_t;				/*各子线程任务结构体*/

struct threadpool_t {
	pthread_mutex_t lock;				/*当前结构体的锁*/
    pthread_mutex_t thread_counter;		/*记录忙线程个数的锁--busy_thr_num*/
    pthread_cond_t queue_not_full;		/*当任务队列满，添加任务线程，阻塞等待次条件变量*/
    pthread_cond_t queue_not_empty;		/*当任务队列不空，通知等待处理任务的线程*/
    
    pthread_t *threads;					/*线程池中存储线程id的数组*/
    pthread_t adjust_thread;			/*管理者线程*/
    thread_task_t *task_queue;			/*任务队列，记录任务的回调函数*/
    
    int min_thr_num;					/*线程池中最小线程数*/
    int max_thr_num;					/*线程池中最大线程数*/
    int live_thr_num;					/*线程池中当前存活线程数*/
    int busy_thr_num;					/*线程池中忙线程数*/
    int wait_exit_thr_num;				/*线程池中等待销毁线程数*/
    
    int queue_front;					/*任务队列队头下标*/
    int queue_rear;						/*任务队列队尾下标*/
    int queue_size;						/*当前任务队列实际任务数大小*/
    int queue_max_size;					/*任务队列可容纳任务上限*/
    
    int shutdowm;						/*描述线程池使用标志*/
};

void *threadpool_thread(void *threadpool);	/*线程池子线程的执行函数，参数为线程池结构体本身*/
void *adjust_thread(void *threadpool);		/*管理线程的执行函数，参数为线程池结构体本身*/
int is_thread_alive(pthread_t tid);
int threadpool_free(pthreadpool_t *pool);

threadpool_t* threadpool_create(int min_thr_num, int max_thr_num, int queue_max_size) {
    threadpool_t *pool = NULL:
    do {
        if ((pool = (threadpool_t*)malloc(sizeof(threadpool_t))) == NULL) {
            printf("malloc threadpool failed...");
            break;
        }
        
        pool->min_thr_num = min_thr_num;
        pool->max_thr_num = max_thr_num;
        pool->busy_thr_num = 0;
        pool->live_thr_num = 0;
        pool->wait_exit_thr_num = 0;
        pool->queue_size = 0;
        pool->queue_max_size = queue_max_size;
        pool->queue_front = 0;
        pool->queue_rear = 0;
        pool->shutdown = false;
        
        /*给线程数组开辟空间*/
        pool->threads = (thread_t *)malloc(sizeof(thread_t)*max_thr_num);
        if (pool->threads == NULL) {
            printf("malloc threads failed...");
            break;
        }
        memset(pool->threads, 0, sizeof(pool->threads)*max_thr_num);
        
        /*给任务队列开辟空间*/
        pool->task_queue = (threadpool_task_t*)malloc(sizeof(threadpool_task_t)*queue_max_size);
        if (pool->task_queue == NULL) {
            printf("malloc task_queue failed...");
            break;
        }
        
        /*初始化互斥量、信号量，动态初始化*/
        if (pthread_mutex_init(&(pool->lock), NULL) != 0
           || pthread_mutex_init(&(pool->thread_counter), NULL) != 0
           || pthread_cond_init(&(pool->queue_not_full), NULL) != 0
           || pthread_cond_init(&(pool->queue_not_emoty), NULL) != 0) 
        {
            printf("init lock or cond failed...");
            break;
        }
        
        /*创建线程*/
        for(int i=0; i<min_thr_num; i++) {
            pthread_create(&(pool->threads[i]), NULL, threadpool_thread, (void*)pool);
            printf("start thread 0x%x...\n", (unsigned int)pool->threads[i]);
        }
        pthread_create(&(pool->adjust_thread), NULL, adjust_thread, (void*)pool);
        return pool;
    }while(0);
    
    /*失败则释放空间*/
    threadpool_free(pool);
    return NULL;
}

/*向任务队列中添加任务*/
int threadpool_add(threadpool_t* pool, void*(*functoin)(void* arg), void* arg) {
	pthread_mutex_lock(&(pool->lock));
    
    while ((pool->queue_size == pool->queue_max_size) && (!pool->shutdown)) {
        pthread_cond_wait(&(pool->queue_not_full), &(pool->lock));
    }
    if (pool->shutdown) {
        pthread_cond_broadcast(&(pool->queue_not_empty));
        pthread_mutex_unlock(&(pool->lock));
        return 0;
    }
 
    /* 清空工作线程，调用回调函数的参数arg */
    if (pool->task_queue[pool->queue_rear].arg != NULL) {
        pool->task_queue[pool->queue_rear].arg = NULL;
    }
    
    /*添加任务到任务队列里*/
    pool->task_queue[pool->queue_rear].function = function;
    pool->task_queue[pool->queue_rear].arg = arg;
    pool->queue_rear = (pool->queue_rear + 1) % pool->queue_max_size;
    pool->queue_size++;

    /*添加完任务后，队列不为空，唤醒线程池中 等待处理任务的线程*/
    pthread_cond_signal(&(pool->queue_not_empty));
    pthread_mutex_unlock(&(pool->lock));
    
    return 0;
}

/*子线程回调函数*/
void* threadpool_thread(void* threadpool) {
    threadpool_t* pool = (threadpool_t*)threadpool;
    threadpool_task_t task;
    
    while(true) {
        pthread_mutex_lock(&(pool->lock));
        
        while((pool->queue_size == 0) && (!pool->shutdown)) {
            printf("thread 0x%x is waiting\n", (unsigned int)pthread_self());
            /*阻塞等待条件变量*/
            pthread_cond_wait(&(pool->queue_not_empty), &(pool->lock));
            
            /*清除指定数目空闲线程数*/
            if (pool->wait_exit_thr_num > 0) {
                pool->wait_exit_thr_num--;
                if (pool->live_thr_num > pool->min_thr_num) {
                    printf("thread 0x%x is exiting...", (unsigned int)pthread_self());
                    pool->live_thr_num--;
                    pthread_mutex_unlock(&(pool->lock));
                    
                    pthread_exit(NULL);
                }
            }
        }
        
        if (pool->shutdown) {
            pthread_mutex_unlock(&(pool->lock));
            printf("thread 0x%x is exiting...\n", (unsigned int)pthread_self());
            pthread_thread_detach(pthread_self());
            pthread_exit(NULL);
        }
        
        task.function = pool->task_queue[pool->queue_front].function;
        task.arg = pool->task_queue[pool->queue_front].arg;
        
        pool->queue_front = (pool->queue_front + 1) % pool->queue_max_size;
        pool->queue_size--;
        
        /*通知可以向任务队列添加任务*/
        pthread_cond_broadcast(&(pool->queue_not_full));
        pthread_mutex_unlock(&(pool->lock));
        
        printf("thread 0x%x is start working...\n", (unsigned int)pthread_self());
        pthread_mutex_lock(&(pool->thread_counter));
        pool->busy_thr_num++;
        pthread_mutex_unlock(&(pool->thread_counter));
        
        /*执行任务*/
        (*(task.function))(task.arg);
        
        printf("thread 0x%x is end working...\n", (unsigned int)pthread_self());
        pthread_mutex_lock(&(pool->thread_counter));
        pool->busy_thr_num--;
        pthread_mutex_unlock(&(pool->thread_counter));
    }
}

/*管理线程回调函数*/
void* adjust_thread(threadpool_t* threadpool) {
    threadpool_t *pool = (threadpool_t*)threadpool;
    while(!pool->shutdown) {
        sleep(DEFAULT_TIME); 						/*定时管理*/
        
        pthread_mutex_lock(&(pool->lock));
        int queue_size = pool->queue_size;			/*任务队列内任务数*/
        int live_thr_num = pool->live_thr_num;		/*线程池中存活线程数*/
        pthread_mutex_unlock(&(pool->lock));
        
        pthread_mutex_lock(&(pool->thread_counter));
        int busy_thr_num = pool->busy_thr_num;		/*忙线程数*/
        pthread_mutex_unlock(&(pool->thread_counter));
        
        /*扩容*/
        if (queue_size >= MIN_WAIT_TASK_NUM && live_thr_num <= pool->max_thr_num) {
            pthread_mutex_lock(&(pool->lock));
            int add = 0;
            for(int i=0; i < pool->max_thr_num && add < DEFAULT_THREAD_VARY && 
               pool->live_thr_num < pool_max_thr_num; i++) 
            {
                if (pool->threads[i] != 0 || !is_thread_alive(pool->threads[i])) {
                    pthread_create(&(pool->thread[i]), NULL, threadpool_thread, (void*)pool);
                    add++;
                    pool->live_thr_num++;
                }
            }
            pthread_mutex_unlock(&(pool->lock));
        }
        /*销毁*/
        if ((busy_thr_num * 2) < live_thr_num && live_thr_num > pool->min_thr_num) {
           	pthread_mutex_lock(&(pool->lock));
            pool->wait_exit_num = DEFAULT_THREAD_VARY; 	/*销毁步长*/
            pthread_mutex_unlock(&(pool->lock));
            
            for(int i=0; i < DEFAULT_THREAD_VARY; i++) {
                pthread_cond_signal(&(pool->queue_not_empty));
            }
        }
    }
}

int threadpool_destroy(threadpool_t* pool) {
    if (pool == NULL) {
        return -1;
    }
    
    pool->shutdown = true;
    pthread_join(pool->adjust_thread, NULL);
    
    for(int i=0; i<pool->live_thr_num; i++) {
        pthread_cond_signal(&(pool->queue_not_empty));
    }
    for(int i=0; i<pool->live_thr_num; i++) {
        pthread_join(pool->thread[i], NULL);
    }
    
    threadpool_free(pool);
}

int threadpool_free(threadpool_t* pool) {
    if (pool == NULL) {
        return -1;
    }
    if (pool->task_queue) {
        free(pool->task_queue);
    }
    if (pool->threads) {
        free(pool->threads);
        pthread_mutex_lock(&(pool->lock));
        pthread_mutex_destory(&(pool->lock));
        pthread_mutex_lock(&(pool->counter));
        pthread_mutex_destroy(&(pool->counter));
        pthread_mutex_destroy(&(pool->queue_not_full));
        pthread_mutex_destroy(&(pool->queue_not_empty));
    }
    free(pool);
    pool = NULL;
    
    return 0;
}

int is_thread_alive(pthread_t tid) {
	int kill_rc = pthread_kill(tid, 0);     //发0号信号，测试线程是否存活
    if (kill_rc == ESRCH) {
        return false;
    }
    return true;
}

void *process(void *arg){
    printf("thread 0x%x working on task %d\n ",
           (unsigned int)pthread_self(), *(int*)arg);
    sleep(1);
    printf("task %d is end\n", *(int*)arg); 
    return NULL;
}

int main(void) {
    /*开始：创建线程池*/
    threadpool_t *thpool = threadpool_create(3, 100, 100); 
    printf("pool inited...");
    
    /*工作：添加任务*/
    int num[20];
    for(int i=0; i<20; i++) {
        num[i]=i;
        printf("add task %d\n", i);
        threadpool_add(thpool, process, (void*)&num[i]);
    }
    
    sleep(10);						/*子线程完成任务*/
    threadpool_destroy(thpool);		/*销毁线程池*/
    
    return 0;
}
```