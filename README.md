# Hash Hash Hash

This lab implements hash tables that are safe to use concurrently. Hash-table-v1 utilizes one mutex with the goal of correctness, while hash-table-v2 utilizes more than one mutexes in order to prioritize performance.
## Building

To build this program, type "make" in the correct repository.

## Running

./hash-table-tester -t 8 -s 50000
Generation: 69,534 usec
Hash table base: 1,880,940 usec
  - 0 missing
Hash table v1: 1,736,363 usec
  - 0 missing
Hash table v2: 672,493 usec
  - 0 missing

## First Implementation

My first implementation strategy was to simply lock all sections of the code which involved reading or writing data (essentially the whole insert function) because, for this version, performance is irrelevant. This strategy is correct because all areas where the code could be problematic (looking through the hash table and adding to it) are completely blocked off as, effectively, only one thread at a time can add an entry to the hash table. There can be no race conditions: whenever one thread reads the entries, there are no other entries in the process of being entered that could mess it up (only one thread is allowed to access one list at a time). Moreover, whenever a thread inserts an item, there cannot be another other threads working to insert an element in the same place. I also initialized and destroyed the lock outside of the function.

### Performance

./hash-table-tester -t 2 -s 200000
Generation: 74,488 usec
Hash table base: 1,634,858 usec
Hash table v1: 2,012,596 usec
With a low number of threads (2) this implmentation actually seems to slow down the hash table. This could be due to the poor use of mutex and the extra overhead from the threads.

./hash-table-tester -t 4 -s 100000
Generation: 73,685 usec
Hash table base: 1,852,474 usec
  - 0 missing
Hash table v1: 2,552,889 usec
  - 0 missing
Now with a higher number of threads (yet maintaining a constant amount of work), the increase in time is even more drastic. Evidently, the poor implementation becomes detrimental because of the additional overhead created by the threads


## Second Implementation

For my second implementation, I decided to use a mutex for each list in the hash table. This meant that I could lock any threads using that list while other threads were using that particular list. In order to do so, I locked the corresponding list that whichever new entry would use with an array of locks. This works because, when entering a new entry, there are only potential race conditions between this entry and other entries that are meant for the same list. For all possibilities, this prevents race conditions: if there are two threads that are entering into different lists, both are allowed to run - there are no race conditions because they interact with different lists (looking at different elements). If there are two threads that are working in the same list, then both cannot be working the list at the same time (this creates race conditions with misreading while another thread is inserting - an duplicate key, etc.). My array of locks prevents two threads from working on the same list at once, preventing this condition. However, there are still some non-deterministic races that can occur with this implementation. For instance, if two threads want to edit the same exact node and change the value, the thread that gets the lock last will have the final value - this is nondeterministic.
### Performance

./hash-table-tester -t 2 -s 200000
Generation: 75,855 usec
Hash table base: 1,678,258 usec
  - 0 missing
Hash table v1: 2,103,055 usec
  - 0 missing
Hash table v2: 993,239 usec
  - 0 missing
Here with 2 threads, the second implementation seems to speed up the process by around 2x. This shows that there is relatively minimal overhead as the program seems to be working close the ideal (as it should work double as fast with 2 threads). However, due to the overhead, it is not in actuality 2x.

./hash-table-tester -t 4 -s 100000
Generation: 98,657 usec
Hash table base: 1,928,443 usec
  - 0 missing
Hash table v1: 2,338,790 usec
  - 0 missing
Hash table v2: 773,001 usec
  - 0 missing
Here, with 4 threads (my maximum), there is definitely more of a decrease than with the 2 threads. Still, the ratio has seemed to lessen as 4 threads is farther away from being 4x than 2 threads was to 2x. This is potentially due to the increase of overhead that it takes to create double the threads. 

Compared to the first implementation, this implementation is clearly faster and more effective, as it consistently quickens the operation, while the first implementation does not. This is because the threads spend way less time being blocked in the second implementation - they are only blocked if another thread is working on the same list. This way, the second implemenation can work in parallel while the first one is a pseudo parallel.


## Cleaning up

To clean up the binary files, type "make clean" in the correct repository.
