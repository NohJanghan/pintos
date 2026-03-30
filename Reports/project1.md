# Project 1: Alarm Clock

20235059 Janghan Noh [janghan.noh@gm.gist.ac.kr](mailto:janghan.noh@gm.gist.ac.kr)

## Introduction

The objective of this project is to improve the timer mechanism in Pintos kernel. The current implementation of `timer_sleep` in `devices/timer.c` uses a 'busy waiting' mechanism, where a sleeping thread checks whether they should wake up or not at every timer tick. Thus, it should be improved by replacing the 'busy waiting' design with a new block-based design. Code is available at [https://github.com/NohJanghan/pintos](https://github.com/NohJanghan/pintos.git).

## Problem Definition

The given `timer_sleep` function uses 'busy waiting'. When a thread calls `timer_sleep`, it goes to a while statement where it yields the CPU to other threads until the target tick is reached. Since it moves to the back of the ready queue after yielding, it will check if it should wake up repeatedly. Thus, it consumes CPU cycles even when the target tick is not reached yet.

![Fig 1. Busy-waiting result on 'alarm-multiple'](<project1 - before.png>)

As shown in Fig. 1 , the 'alarm-multiple' test reports that many kernel ticks were used. If other threads have been running, the tasks would have been done late. This problem is worse when many threads are sleeping because each of them checks whether they should wake up at every timer tick. 

## Policy and Algorithm Design

The problem can be solved by avoiding using 'busy waiting' approach. Instead of busy waiting, it can be handled using block-based approach. In this approach, the thread which calls `timer_sleep` blocks itself and goes into a sleep queue. The sleep queue is a priority queue ordered by the target wake-up tick, which contains threads in sleep state. Then, as timer tick changes, the system checks the sleep queue. If the current timer tick hits or passes by the target ticks which the threads requested, the threads should be woken up. For this, each thread should maintain a variable storing the tick when it should wake up. The woken thread doesn't hold the CPU immediately, but it goes into the back of the ready queue and wait for the scheduling.

The priority queue described above can be implemented in various ways, such as a sorted list or a heap. In this project, a sorted list was chosen for the implementation. The threads in the list keep sorted by the target ticks when they will wake up. This implementation assures that the target tick checking procedure and removing element complete in time complexity of $O(1)$. However, the time complexity of insertion is $O(n)$.

An alternative of the sorted list in implementing a priority queue is using a heap. The implementation with heap would be efficient because the time complexity of access to the first element is $O(1)$ and the time complexities of insertion and deletion are $O(\log n)$. Nonetheless, time complexity of $O(n)$ is affordable in a small scale like Pintos. Also, the sorted list provides advantages in terms of simplicity because there already exists the implementation of sorted list in `list.c`.

The entire design can be divided to two algorithms. When a thread enters a sleep state, it puts itself in the sleep list with a variable denoting the target tick and blocks itself. To handle an edge case where a thread wants to sleep for zero or negative ticks, a conditional is positioned before the logic. The conditional checks the input and returns immediately if the sleep tick is zero or negative.

The other procedure is handled in the timer interrupt. At every timer tick, the timer interrupt checks the first element of the sleep list and if it should be woken up by looking up its target tick. If the thread needs to be woken up, it is removed from the list and goes to the ready list, an implementation of the ready queue. This procedure is repeated until there's no need to wake up the first element of the list.

## Implementation

A new static variable `sleep_list` was declared in `timer.c` to manage the sleep queue. The list is a variable of type `struct list` which is implementation of a doubly linked list. The list is initialized in `timer_init`. When a `thread` goes into `sleep_list`, the function `list_insert_ordered` is called. The function takes a function pointer of type `list_less_func` for sorting. A static function `tick_less` was implemented in `timer.c` to compare two threads' target ticks. Also, a new member variable `tick_to_wakeup` is declared in `thread`, which manages the target tick to wake up.

The function `timer_sleep` is in charge of managing the procedure where a thread goes to sleep. In the function, a thread puts its `elem` in `sleep_list` while setting its `tick_to_wakeup` variable to the sum of current tick and ticks to sleep. After that it blocks itself using the function `thread_block`. The conditional for the edge case checks the input and returns immediately if the required sleep tick is zero or negative.

`sleep_list` is shared by `timer_sleep` and `timer_interrupt`, and `thread_block` must be called with interrupts turned off. Therefore, inserting the current thread into `sleep_list` and blocking it should be performed as a single critical section with interrupts disabled to prevent race conditions. `timer_sleep` disables interrupts before inserting the thread into the list and restores the previous interrupt level after `thread_block` returns.

The other procedure is handled in `timer_interrupt`. At every timer tick, `timer_interrupt` checks the first element of the sleep list and if it should be woken up by looking up its `tick_to_wakeup` variable. If the thread needs to be woken up, it pops out of the sleep list and goes to `ready_list` by calling `thread_unblock`. As noted in the previous section, this procedure is repeated until there's no need to wake up the first element. Note that the popping out must happen before the insertion to `ready_list`, because both `ready_list` and `sleep_list` share a single `list_elem` variable `elem`  in `thread`.

### Modified Files

- `src/devices/timer.c`
  - Added static `sleep_list` and initialized in `timer_init`
  - Added `tick_less` comparator for ordered insertion
  - Modified `timer_sleep`: block-based sleep instead of busy waiting
  - Modified `timer_interrupt`: wake expired threads from `sleep_list`
- `src/threads/thread.h`
  - Added `tick_to_wakeup` field to `struct thread`

## Conclusion

Before the improvement, Pintos spent most time on kernel mode since it should keep checking all the sleeping threads until they wake up. Through this improvement, it stays mostly in the idle thread even when multiple threads call `timer_sleep`, as shown in Fig. 2. It is a result of replacing 'busy waiting' mechanism with the block-based mechanism.


![Fig 2. Improved result on 'alarm-multiple'](<project1 - after.png>)