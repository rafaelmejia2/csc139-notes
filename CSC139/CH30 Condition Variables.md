To wait for a condition to become true, a thread can make use of what is known as a **condition variable**. A **condition variable** is an explicit queue that threads can put themselves on when some state of execution (i.e., some **condition**) is not as desired (by **waiting** on the condition); some other thread, when it changes said state, can then wake one (or more) of those waiting threads and thus allow them to continue (by **signaling** on the condition).

```C
1 int done = 0;
2 pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
3 pthread_cond_t c = PTHREAD_COND_INITIALIZER;
4
5 void thr_exit() {
6 Pthread_mutex_lock(&m);
7 done = 1;
8 Pthread_cond_signal(&c);
9 Pthread_mutex_unlock(&m);
10 }
11
12 void *child(void *arg) {
13 printf("child\n");
14 thr_exit();
15 return NULL;
16 }
17
18 void thr_join() {
19 Pthread_mutex_lock(&m);
20 while (done == 0)
21 Pthread_cond_wait(&c, &m);
22 Pthread_mutex_unlock(&m);
23 }
24
25 int main(int argc, char *argv[]) {
26 printf("parent: begin\n");
27 pthread_t p;
28 Pthread_create(&p, NULL, child, NULL);
29 thr_join();
30 printf("parent: end\n");
31 return 0;
32 }
```
**Figure 30.3: Parent Waiting For Child: Use A Condition Variable**

```C
pthread_cond_wait(pthread_cond_t *c, pthread_mutex_t *m);
pthread_cond_signal(pthread_cond_t *c);
```

## **The Producer/Consumer (Bounded Buffer) Problem**
The next synchronization problem we will confront in this chapter is known as the **producer/consumer** problem, or sometimes as the **bounded buffer** problem, which was first posed by Dijkstra. Indeed, it was this very producer/consumer problem that led Dijkstra and his co-workers to invent the generalized semaphore (which can be used as either a lock or a condition variable); we will learn more about semaphores later. 

Imagine one or more producer threads and one or more consumer threads. Producers generate data items and place them in a buffer; consumers grab said items from the buffer and consume them in some way.

Signaling a thread only wakes them up; it is thus a hint that the state of the world has changed (in this case, that a value has been placed in the buffer), but there is no guarantee that when the woken thread runs, the state will still be as desired. This interpretation of what a signal means is often referred to as **Mesa semantics**, after the first research that built a condition variable in such a manner; the contrast, referred to as **Hoare semantics**, is harder to build but provides a stronger guarantee that the woken thread will run immediately upon being woken. Virtually every system ever built employs Mesa semantics.

```C
1 int buffer[MAX];
2 int fill_ptr = 0;
3 int use_ptr = 0;
4 int count = 0;
5
6 void put(int value) {
7 buffer[fill_ptr] = value;
8 fill_ptr = (fill_ptr + 1) % MAX;
9 count++;
10 }
11
12 int get() {
13 int tmp = buffer[use_ptr];
14 use_ptr = (use_ptr + 1) % MAX;
15 count--;
16 return tmp;
17 }
```
**Figure 30.13: The Correct Put And Get Routines**

```C
1 cond_t empty, fill;
2 mutex_t mutex;
3
4 void *producer(void *arg) {
5 int i;
6 for (i = 0; i < loops; i++) {
7 Pthread_mutex_lock(&mutex); // p1
8 while (count == MAX) // p2
9 Pthread_cond_wait(&empty, &mutex); // p3
10 put(i); // p4
11 Pthread_cond_signal(&fill); // p5
12 Pthread_mutex_unlock(&mutex); // p6
13 }
14 }
15
16 void *consumer(void *arg) {
17 int i;
18 for (i = 0; i < loops; i++) {
19 Pthread_mutex_lock(&mutex); // c1
20 while (count == 0) // c2
21 Pthread_cond_wait(&fill, &mutex); // c3
22 int tmp = get(); // c4
23 Pthread_cond_signal(&empty); // c5
24 Pthread_mutex_unlock(&mutex); // c6
25 printf("%d\n", tmp);
26 }
27 }
```
**Figure 30.14: The Correct Producer/Consumer Synchronization**

Using while loops around conditional checks also handles the case where **spurious wakeups** occur.


## **Covering Conditions**
```C
1 // how many bytes of the heap are free?
2 int bytesLeft = MAX_HEAP_SIZE;
3
4 // need lock and condition too
5 cond_t c;
6 mutex_t m;
7
8 void *
9 allocate(int size) {
10 Pthread_mutex_lock(&m);
11 while (bytesLeft < size)
12 Pthread_cond_wait(&c, &m);
13 void *ptr = ...; // get mem from heap
14 bytesLeft -= size;
15 Pthread_mutex_unlock(&m);
16 return ptr;
17 }
18
19 void free(void *ptr, int size) {
20 Pthread_mutex_lock(&m);
21 bytesLeft += size;
22 Pthread_cond_signal(&c); // whom to signal??
23 Pthread_mutex_unlock(&m);
24 }
```
**Figure 30.15: Covering Conditions: An Example**

Consider the following scenario. Assume there are zero bytes free; thread $T_a$ calls allocate(100), followed by thread $T_b$ which asks for less memory by calling allocate(10). Both $T_a$ and $T_b$ thus wait on the condition and go to sleep; there aren’t enough free bytes to satisfy either of these requests. 

At that point, assume a third thread, $T_c$ , calls free(50). Unfortunately, when it calls signal to wake a waiting thread, it might not wake the correct waiting thread, $T_b$ , which is waiting for only 10 bytes to be freed; $T_a$ should remain waiting, as not enough memory is yet free. Thus, the code in the figure does not work, as the thread waking other threads does not know which thread (or threads) to wake up. 

The solution suggested by Lampson and Redell is straightforward: replace the `pthread_cond_signal()` call in the code above with a call to `pthread_cond_broadcast()`, which wakes up all waiting threads. By doing so, we guarantee that any threads that should be woken are. The downside, of course, can be a negative performance impact, as we might needlessly wake up many other waiting threads that shouldn’t (yet) be awake. Those threads will simply wake up, re-check the condition, and then go immediately back to sleep. 

Lampson and Redell call such a condition a **covering condition**, as it covers all the cases where a thread needs to wake up (conservatively); the cost, as we’ve discussed, is that too many threads might be woken.