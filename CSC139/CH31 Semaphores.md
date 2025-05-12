A semaphore is an object with an integer value that we can manipulate with two routines; in the POSIX standard, these routines are `sem_wait()` and `sem_post()`. Because the initial value of the semaphore determines its behavior, before calling any other routine to interact with the semaphore, we must first initialize it to some value, as the code in Figure 31.1 does.
```C
1 #include <semaphore.h>
2 sem_t s;
3 sem_init(&s, 0, 1);
```
**Figure 31.1: Initializing A Semaphore**

```C
1 int sem_wait(sem_t *s) {
2 decrement the value of semaphore s by one
3 wait if value of semaphore s is negative
4 }
5
6 int sem_post(sem_t *s) {
7 increment the value of semaphore s by one
8 if there are one or more threads waiting, wake one
9 }
```
**Figure 31.2: Semaphore: Definitions Of Wait And Post**

```C
1 sem_t m;
2 sem_init(&m, 0, X); // init to X; what should X be?
3
4 sem_wait(&m);
5 // critical section here
6 sem_post(&m);
```
**Figure 31.3: A Binary Semaphore (That Is, A Lock)**

## **Semaphores for Ordering**
```C
1 sem_t s;
2
3 void *child(void *arg) {
4 printf("child\n");
5 sem_post(&s); // signal here: child is done
6 return NULL;
7 }
8
9 int main(int argc, char *argv[]) {
10 sem_init(&s, 0, X); // what should X be?
11 printf("parent: begin\n");
12 pthread_t c;
13 Pthread_create(&c, NULL, child, NULL);
14 sem_wait(&s); // wait here for child
15 printf("parent: end\n");
16 return 0;
17 }
```
**Figure 31.6: A Parent Waiting For Its Child**

The answer, of course, is that the value of the semaphore should be set to is 0. There are two cases to consider. First, let us assume that the parent creates the child but the child has not run yet (i.e., it is sitting in a ready queue but not running). In this case (Figure 31.7, page 6), the parent will call `sem_wait()` before the child has called `sem_post()`; we’d like the parent to wait for the child to run. The only way this will happen is if the value of the semaphore is not greater than 0; hence, 0 is the initial value. The parent runs, decrements the semaphore (to -1), then waits (sleeping). When the child finally runs, it will call `sem_post()`, increment the value of the semaphore to 0, and wake the parent, which will then return from `sem_wait()` and finish the program.

## **Bounded Buffer**
```C
1 int buffer[MAX];
2 int fill = 0;
3 int use = 0;
4
5 void put(int value) {
6 buffer[fill] = value; // Line F1
7 fill = (fill + 1) % MAX; // Line F2
8 }
9
10 int get() {
11 int tmp = buffer[use]; // Line G1
12 use = (use + 1) % MAX; // Line G2
13 return tmp;
14 }
```
**Figure 31.9: The Put And Get Routines**

```C
1 sem_t empty;
2 sem_t full;
3
4 void *producer(void *arg) {
5 int i;
6 for (i = 0; i < loops; i++) {
7 sem_wait(&empty); // Line P1
8 put(i); // Line P2
9 sem_post(&full); // Line P3
10 }
11 }
12
13 void *consumer(void *arg) {
14 int tmp = 0;
15 while (tmp != -1) {
16 sem_wait(&full); // Line C1
17 tmp = get(); // Line C2
18 sem_post(&empty); // Line C3
19 printf("%d\n", tmp);
20 }
21 }
22
23 int main(int argc, char *argv[]) {
24 // ...
25 sem_init(&empty, 0, MAX); // MAX are empty
26 sem_init(&full, 0, 0); // 0 are full
27 // ...
28 }
```
**Figure 31.10: Adding The Full And Empty Conditions**

```C
1 void *producer(void *arg) {
2 int i;
3 for (i = 0; i < loops; i++) {
4 sem_wait(&mutex); // Line P0 (NEW LINE)
5 sem_wait(&empty); // Line P1
6 put(i); // Line P2
7 sem_post(&full); // Line P3
8 sem_post(&mutex); // Line P4 (NEW LINE)
9 }
10 }
11
12 void *consumer(void *arg) {
13 int i;
14 for (i = 0; i < loops; i++) {
15 sem_wait(&mutex); // Line C0 (NEW LINE)
16 sem_wait(&full); // Line C1
17 int tmp = get(); // Line C2
18 sem_post(&empty); // Line C3
19 sem_post(&mutex); // Line C4 (NEW LINE)
20 printf("%d\n", tmp);
21 }
22 }
```
**Figure 31.11: Adding Mutual Exclusion (Incorrectly)**

Imagine two threads, one producer and one consumer. The consumer gets to run first. It acquires the mutex (Line C0), and then calls `sem_wait()` on the full semaphore (Line C1); because there is no data yet, this call causes the consumer to block and thus yield the CPU; importantly, though, the consumer still holds the lock. 

A producer then runs. It has data to produce and if it were able to run, it would be able to wake the consumer thread and all would be good. Unfortunately, the first thing it does is call `sem_wait()` on the binary mutex semaphore (Line P0). The lock is already held. Hence, the producer is now stuck waiting too. 

There is a simple cycle here. The consumer *holds* the mutex and is *waiting* for the someone to signal full. The producer could *signal* full but is *waiting* for the mutex. Thus, the producer and consumer are each stuck waiting for each other: a classic deadlock.

## **Reader-Writer Locks**
```C
1 typedef struct _rwlock_t {
2 sem_t lock; // binary semaphore (basic lock)
3 sem_t writelock; // allow ONE writer/MANY readers
4 int readers; // #readers in critical section
5 } rwlock_t;
6
7 void rwlock_init(rwlock_t *rw) {
8 rw->readers = 0;
9 sem_init(&rw->lock, 0, 1);
10 sem_init(&rw->writelock, 0, 1);
11 }
12
13 void rwlock_acquire_readlock(rwlock_t *rw) {
14 sem_wait(&rw->lock);
15 rw->readers++;
16 if (rw->readers == 1) // first reader gets writelock
17 sem_wait(&rw->writelock);
18 sem_post(&rw->lock);
19 }
20
21 void rwlock_release_readlock(rwlock_t *rw) {
22 sem_wait(&rw->lock);
23 rw->readers--;
24 if (rw->readers == 0) // last reader lets it go
25 sem_post(&rw->writelock);
26 sem_post(&rw->lock);
27 }
28
29 void rwlock_acquire_writelock(rwlock_t *rw) {
30 sem_wait(&rw->writelock);
31 }
32
33 void rwlock_release_writelock(rwlock_t *rw) {
34 sem_post(&rw->writelock);
35 }
```
**Figure 31.13: A Simple Reader-Writer Lock**

## **The Dining Philosophers**
![[Pasted image 20250512005747.png]]
```C
while (1) {
think();
get_forks(p);
eat();
put_forks(p);
}
```

```C
// helper functions
int left(int p) { return p; }
int right(int p) { return (p + 1) % 5; }
```

```C
1 void get_forks(int p) {
2 sem_wait(&forks[left(p)]);
3 sem_wait(&forks[right(p)]);
4 }
5
6 void put_forks(int p) {
7 sem_post(&forks[left(p)]);
8 sem_post(&forks[right(p)]);
9 }
```

```C
1 void get_forks(int p) {
2 if (p == 4) {
3 sem_wait(&forks[right(p)]);
4 sem_wait(&forks[left(p)]);
5 } else {
6 sem_wait(&forks[left(p)]);
7 sem_wait(&forks[right(p)]);
8 }
9 }
```
**Breaking the cycle**
