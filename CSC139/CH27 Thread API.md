**Thread Creation**
```C
#include <pthread.h>
int
pthread_create(pthread_t *thread,
const pthread_attr_t *attr,
void *(*start_routine)(void*),
void *arg);
```

**Thread Completion**
```C
int pthread_join(pthread_t thread, void **value_ptr);
```

**Locks**
```C
// basic routines
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);

// initialization
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
// or
int rc = pthread_mutex_init(&lock, NULL); assert(rc == 0); // always check success!


pthread mutex destroy() // when done
```

**Condition Variables**
```C
// basic routines
int pthread_cond_wait(pthread_cond_t *cond,
                      pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);

// initialization
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// typical usage
Pthread_mutex_lock(&lock);
while (ready == 0)
	Pthread_cond_wait(&cond, &lock);
Pthread_mutex_unlock(&lock);

// waking thread
Pthread_mutex_lock(&lock);
ready = 1;
Pthread_cond_signal(&cond);
Pthread_mutex_unlock(&lock);
```



