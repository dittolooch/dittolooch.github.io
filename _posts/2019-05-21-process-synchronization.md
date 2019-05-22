---
layout: post
title: Process Synchronization
tags: [os]
mathjax: true
---

>Wikipedia: Thread synchronization is defined as a mechanism which ensures that two or more concurrent processes or threads do not simultaneously execute some particular program segment known as critical section.

## Race Condition & Critical Section Problem

Consider the following producer and consumer programs:


```
//producer process
while(true){
    while(counter==BUFFER_SIZE){
        // buffer is full, do nothing.
    }
    buffer[in] = next.produced;
    in = (in+1)%BUFFER_SIZE;
    counter++;
}
```



```
//consumer process
while(true){
    while(counter==0){
        // buffer is empty, nothing to consume. do nothing.
    }
    next_consumed = buffer[out];
    out = (out+1)% BUFFER_SIZE;
    counter--;
}
```


Suppose the consumer and producer processes are run concurrently, at it just so happens that producer process enters the ```counter++``` row at about the same time as the consumer process entering the ```counter--``` row, a problem might occur.

To execute the decrementation or incrementation of a variable, the cpu fetches the data from the memory location of the variable ```counter``` and store it in a register. It then increments (or decrements) the register by one and then save the data back into the memory location of ```counter``` variable.

<strong>What's wrong here?
</strong>

Let's imagine that when it happens there are 4 items in the buffer and ```BUFFER_SIZE``` is equal to 10. Logically speaking, The producer would like to increment the ```counter``` by one so that counter should then be 5, and the consumer would decrement the counter after consumption so that the counter ends up being 4.

However, since the programs are run concurrently, when both programs execute the incrementation/decrementation rows, the sequence of the underlying operations might look like

T0: producer register1 = counter // register1 is 4 and counter is 4<br>
T1: producer register1 = register + 1 // register1 is 5, counter is still 4.<br>
T2: consumer register2 = counter // register2 is 4 and counter is still 4<br>
T3: producer counter = register1 // counter is now 5<br>
T4: consumer register2 = register2 - 1 // register2 is now 3<br>
T5: consumer counter = register2 // counter is now 3<br>

Did you see it? In the end the counter is incorrectly set to 3 instead of what it should be (4).


When multiple processes access and manipulate the same data concurrently and the correct outcome depends on particular sequence of access/manipulations, it becomes a potential bug called [race condition](https://en.wikipedia.org/wiki/Race_condition).

The critical section problem is introduced to solve this kind of issues and it aims to design a protocol for each process to request permission before it enters its critical section, within which process would access and manipulate shared data.

A qualified solution must satisfy

1. mutual exclusion: when one process is in its critical section, no other processs should be in its critical section

2. progress: the selection of the process that enters its critical section cannot be postponed indefinitely, and therefore progress is made in each process.

3. bounded waiting: there is a bound that limits the number of times process A enters its critical section after a process B requests to enter its critical section.


## Peterson's Solution

Two processes share ```int turn;``` and ```boolean flag[2];```.

```
do{
    flag[i] = true;
    turn = j;
    while(flag[j] && turn==j){
        // do nothing, process j is in its critical section
    }
    //critical section, do something
    flag[i] = false;
    //remainder section, do something
}while(true)
```

<strong>Mutual Exclusion (Mutex)</strong>

For process i to be in its critical section ```flag[j]``` must be false or ```turn==j``` must be false. Since turn cannot be j and i at the same time, mutual exclusion is achieved.

<strong>Progress & Bounded Waiting</strong>

Process i can be prevented from entering its critical section if ```flag[j]``` stays true and ```turn==j``` stays true.

Imagine that now process i is stuck in while loop waiting for process j, there are several cases to be examined:

1. process j is running the 1st line of code setting ```flag[j]=false``` and ```turn=i```, so process i can then enter its critical section.

2. process j is executing the while loop too. in this case, either ```turn==i``` or ```turn===j``` , if turn is i, then i enters its critical section; if turn is j, then j enters its critical section, finishes and set ```flag[j]=false```, which in turn allows process i to enter its critical section.

Thus, process i will enter its critical section (progress) after at most one time of process j's entering its critical section (bounded waiting)

## Hardware Solution

Hardware instructions that allow programmers to access/manipulate data <strong>atomically</strong> can help solve the critical section problem. By <i>atomically</i>, it means that the instructions are run as one uninterruptible unit.

An example of such hardware instruction:

```
boolean test_and_set(boolean* target){
  boolean rv = *target;
  *target = true;
  return rv;
}
```

Given that ```test_and_set``` can be run atomically, it can be used to solve critical section problem like this:

```

boolean available = false;
do {
  while(!test_and_set(available)){
    //lock not available, waiting
  }
  // critical section
  available = true;
} while(true);
```

Another example is ```compare_and_swap```:

```
int compare_and_swap(int *value, int expected, int new_value){
  int temp = *value;
  if (*value == expected){
    *value = new_value;
  }
  return temp;
}
```

```
do {
  while(compare_and_swap(&lock, 0, 1) != 0){
    //do nothing
  }
  //critical section
  lock = 0;
  //remainder section
} while(true)
```

In this example, if 2 processes are run concurrently and both run the while statement. Since the function ```compare_and_swap``` is run atomically, one process would go first and the second one follows after the first one exits ```compare_and_swap```. At the beginning the lock is initiated as 0, so the first process that runs ```compare_and_swap``` is able to change the value of the lock (```(*value==expected)``` is true) and return its initial value of 0, thereby allowing process 1 to exit the while loop and enter its critical section. Process 2 now enters ```compare_and_swap``` and since ```*value != expected ``` it is trapped in the while loop until process 1 finishes its critical section and set the lock to 0.



## Software Solution - Mutex Lock

With ```acquire()``` and ```release()``` functions, a process has to acquire the lock before entering its critical section. If the local is not available, the process busy waits until the lock becomes available.

```
acquire(){
  while(!available){
    //do nothing, wait
  }
  available = false;
}

release(){
  available = true;
}
```

Both methods have to be able to be run atomically for it to work, therefore some hardware instructions that are run atomically can be of help to implement ```acquire``` and ```release```.

Assuming we have a struct ```lock``` that has a variable ```int available```, we can implement the mutex lock as shown below:

```
typedef struct {
  int available;
} lock;

void acquire(lock* mutex){
  while (compare_and_swap(mutex->available, 0, 1)!=0){
    //lock is not available, waiting
  }
}

void release(lock *mutex){
  test_and_set(mutex->available);
}
```

The implementation has a major drawback: spinlock. Spinlock means that when a process is in its critical section, other waiting processes are in a continuous while loop (therefore spinning busily).

The busy waiting wastes CPU cycle. However busy busy wait does not require context switch, which requires a significant amount of time. Thus if the process spins only for a short period of time, it's actually more advantageous to busy wait.

Spinlocks are especially common for multi-processor systems because a thread can spin on one processor while another thread perform s its critical section on another processor.


## Software Solution - Symaphore

Semaphores help sync processes by incrementing and decrementing. There are two functions that work with semaphores: ```signal``` and ```wait```

```
wait (S){
  while (S<=0){
    //no resource, waiting
  }
  S--;
}

signal(S){
 S++;
}
```

For counting semaphores, their values can be in the range of all integers. Let's say for example there are 10 resources that are shared by <strong>n</strong> process where $n \geq 10$. The semaphore S is therefore initiated to 10 and each process calls wait(S) before executing its critical section which manipulates one resource, and calls signal when it has finished using the resource.

The implementation above still suffers from spinlocks (the while loop); to improve it, we can revise the implementation as :

```
typedef struct {
  int value;
  struct process* queue;
}semaphore;

wait (S){
  S--;
  if (S<0){
    add this process to S->queue.
    block();
  }
}

signal(S){
  S++;
  if (S->value<=0){
    dequeue one process from S->queue
    wakeup the process
  }
}
```

## Deadlocks and Starvation

With the previous implementation of semaphores, we are introduced with two new concepts: deadlocks and starvation.

For example, given two processes P1 and P2, they might be executing:

```
//process P1
wait(S1);
wait(S2);

signal(S1);
signal(S2);
```

```
//process P2
wait(S2);
wait(S1);

signal(S2);
signal(S1);
```

In this example, let's assume S1 and S2 are binary semaphores.

If P1 and P2 are run concurrently and both successfully pass the first line of their codes. P1 will then be blocked in ```wait(S2)``` because P2 has obtained the lock of S2. P2 will then be blocked in ```wait(S1)``` because P1 has obtained the lock of S1.

Both are now trapped and can never get out because P1 relies on P2 calling ```signal(S2)``` and P2 relies on P1 calling ```signal(S1)``` to be waken up.

Situations, where every process is waiting for an event (e.g. releasing the resource access) that can only be caused by another process, are called <strong>deadlocks</strong>.

<strong>Starvation</strong>, on the other hand, happens when a process waits indefinitely in a semaphore, which might occur when the waiting list of the semaphore is implemented in a stack-like manner (last-in-first-out).

5.6.4
