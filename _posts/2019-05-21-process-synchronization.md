---
layout: post
title: Process Synchronization
tags: [os]
mathjax: true
---


## Race Condition & Critical Section Problem

Consider the following producer and consumer programs:


{% highlight c++ %}
//producer process
while(true){
    while(counter==BUFFER_SIZE){
        // buffer is full, do nothing.
    }
    buffer[in] = next.produced;
    in = (in+1)%BUFFER_SIZE;
    counter++;
}
{% endhighlight %}



{% highlight c++ %}
//consumer process
while(true){
    while(counter==0){
        // buffer is empty, nothing to consume. do nothing.
    }
    next_consumed = buffer[out];
    out = (out+1)% BUFFER_SIZE;
    counter--;
}
{% endhighlight %}


Suppose the consumer and producer processes are run concurrently, at it just so happens that producer process enters the ```counter++``` row at about the same time as the consumer process entering the ```counter--``` row, a problem might occur.

To execute the decrementation or incrementation of a variable, the cpu fetches the data from the memory location of the variable ```counter``` and store it in a register. It then increments (or decrements) the register by one and then save the data back into the memory location of ```counter``` variable.

What's wrong here?

Let's imagine that when it happens there are 4 items in the buffer and ```BUFFER_SIZE``` is equal to 10. Logically speaking, The producer would like to increment the ```counter``` by one so that counter should then be 5, and the consumer would decrement the counter after consumption so that the counter ends up being 4.

However, since the programs are run concurrently, when both programs execute the incrementation/decrementation rows, the sequence of the underlying operations might look like

T0: producer register1 = counter // register1 is 4 and counter is 4
T1: producer register1 = register + 1 // register1 is 5, counter is still 4.
T2: consumer register2 = counter // register2 is 4 and counter is still 4
T3: producer counter = register1 // counter is now 5
T4: consumer register2 = register2 - 1 // register2 is now 3
T5: consumer counter = register2 // counter is now 3

Did you see it? In the end the counter is incorrectly set to 3 instead of what it should be (4).


When multiple processes access and manipulate the same data concurrently and the correct outcome depends on particular sequence of access/manipulations, it becomes a potential bug called [race condition](https://en.wikipedia.org/wiki/Race_condition).

The critical section problem is introduced to solve this kind of issues and it aims to design a protocol for each process to request permission before it enters its critical section, within which process would access and manipulate shared data.

A qualified solution must satisfy

1. mutual exclusion: when one process is in its critical section, no other processs should be in its critical section

2. progress: the selection of the process that enters its critical section cannot be postponed indefinitely, and therefore progress is made in each process.

3. bounded waiting: there is a bound that limits the number of times process A enters its critical section after a process B requests to enter its critical section.


## Peterson's Solution

Two processes share ```int turn;``` and ```boolean flag[2];```.

{% highlight c++ %}
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
{% endhighlight %}

### Mutual Exclusion (Mutex)

For process i to be in its critical section ```flag[j]``` must be false or ```turn==j``` must be false. Since turn cannot be j and i at the same time, mutual exclusion is achieved.

### Progress & Bounded Waiting

Process i can be prevented from entering its critical section if ```flag[j]``` stays true and ```turn==j``` stays true. 

Imagine that now process i is stuck in while loop waiting for process j, there are several cases to be examined:

1. process j is running the 1st line of code setting ```flag[j]=false``` and ```turn=i```, so process i can then enter its critical section.

2. process j is executing the while loop too. in this case, either ```turn==i``` or ```turn===j``` , if turn is i, then i enters its critical section; if turn is j, then j enters its critical section, finishes and set ```flag[j]=false```, which in turn allows process i to enter its critical section.

Thus, process i will enter its critical section (progress) after at most one time of process j's entering its critical section (bounded waiting)



