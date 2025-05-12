## **Workload Assumptions**
We will make the following assumptions about the processes, sometimes called **jobs**, that are running in the system:
1. Each job runs for the same amount of time.
2. All jobs arrive at the same time.
3. Once started, each job runs to completion
4. All jobs only use the CPU (i.e., they perform no I/O)
5. The run-time of each job is known.

## **Scheduling Metrics**
### $T_{turnaround} = T_{completion} - T_{arrival}$

Because we have assumed that all jobs arrive at the same time, for now $T_{arrival} = 0$ and hence $T_{turnaround} = T_{completion}$. This fact will change as we relax the aforementioned assumptions.

## **First In, First Out (FIFO/FCFS)**

![[Pasted image 20250510213542.png]]
Imagine three jobs arrive in the system, A, B, and C, at roughly the same time ($T_{arrival}=0$). **Average turnaround time** would be $\frac{10 + 20 + 30}{3} = 20$.

If A runs for 100 seconds, FIFO does not work too well.
![[Pasted image 20250510214004.png]]
Average turnaround time would be $\frac{100+110+120}{3}=110$. This problem is generally referred to as the **convoy effect**, where a number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer.

## **Shortest Job First (SJF)**
![[Pasted image 20250510214205.png]]
Average turnaround time improves from the last example to $\frac{10+20+120}{3}=50$, more than a factor of two improvement. 

Although this scheduling algorithm struggles when we have short late arrivals.
![[Pasted image 20250510214411.png]]
Average turnaround time is $\frac{100+(110-10)+(120-10)}{3}=103.33$ seconds.

## **Shortest Time-to-Completion First (STCF)**
![[Pasted image 20250510214554.png]]
If B and C are late arrivals with a shorter completion time, it can **preempt** job A and decide to run another job, perhaps continuing A later.

With STCF we get a average turnaround time of $\frac{(120-0)+(20-10)+(30-10)}{3}=50$ seconds.

## **Response Time**

### $T_{response} = T_{firstrun} - T_{arrival}$

We define response time as the time from when the job arrives in a system to the first time it is scheduled.

![[Pasted image 20250510215104.png]]

## **Round Robin (RR)**
Round robin runs a job for a **time slice** (sometimes called a **scheduling quantum**) and then switches to the next job in the run queue. RR is sometimes called **time-slicing**.

The length of the time slice is critical for RR. The shorter it is, the better performance of RR under the response-time metric. However, making the time slice too short is problematic: suddenly the cost of context switching will dominate overall performance. Thus, deciding on the length of the time slice presents a trade-off to a system designer, making it long enough to **amortize** the cost of switching without making it so long that the system is no longer responsive.

Average response time of RR is 1 second; for SJF, it is 5 seconds.

Short time slices trade response time at the cost of turnaround time.

## **I/O Blocking**

![[Pasted image 20250510215627.png]]

