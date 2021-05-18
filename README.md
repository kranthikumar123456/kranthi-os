# Starve free Readers - Writers Problem



The reader-writer problems deal with synchronizing multiple processes trying to read or write upon a shared data. The first and second problem provide a solution where either the reader or writer processes can possibly starve. The third readers-writers problem deals with an implementation where neither the reader or the writer process will ever starve. Following is the explanation on how it would be done with cpp-like pseudocode :

### The Semaphore

First, let's discuss the implementation of the semaphore. For a starve-free implementation, we need a semaphore that has a First-In-First-Out manner of handling the waiting processes. i.e. - the process that call wait first are the ones are the process that are given the access to semaphore first. 

The code for the semaphore :

```cpp
// The code for a FIFO semaphore.
struct Semaphore{
  int value = 1;
  FIFO_Queue* Q = new FIFO_Queue();
}
    
void wait(Semaphore *S,int* process_id){
  S->value--;
  if(S->value < 0){
  S->Q->push(process_id);
  block(); //this function will block the process by using system call and will transfer it to the waiting queue
           //the process will remain in the waiting queue till it is waken up by the wakeup() system calls
           //this is a type of non busy waiting
  }
}
    
void signal(Semaphore *S){
  S->value++;
  if(S->value <= 0){
  int* PID = S->Q->pop();
  wakeup(PID); //this function will wakeup the process with the given pid using system calls
  }
}


//The code for the queue which will allow us to make a FIFO semaphore.
struct FIFO_Queue{
    ProcessBlock* front, rear;
    int* pop(){
        if(front == NULL){
            return -1;            // Error : underflow.
        }
        else{
            int* val = front->value;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return val;
        }
    }
    void* push(int* val){
        ProcessBlock* blk = new ProcessBlock();
        blk->value = val;
        if(rear == NULL){
            front = rear = n;
            
        }
        else{
            rear->next = blk;
            rear = blk;
        }
    }
    
}

// A Process Block.
struct ProcessBlock{
    ProcessBlock* next;
    int* process_block;
}
```


Thus, a FIFO semaphore can be implemented. A queue is used to manage the waiting processes. The process gets blocked after pushing itself onto the queue and is woken up in FIFO order when some other process releases the semaphore. I have also shown the implementation of the Queue required for the same. 

Next, we discuss the readers and writers process and then discuss how it works in the end.


###  Global Variables

Following are the global variables and their initialization.

```cpp
//Shared data members
Semaphore in  = new Semaphore();
Semaphore out  = new Semaphore();
Semaphore writer = new Semaphore();
writer ->value = 0; // this semaphore is initialized to zero
int var_start = 0; // a count of how many readers have started reading.
int var_completed = 0;// a count of how many readers have completed reading.
//var_start and var_completed are kept in different variable
//instead of merging them into one is because they would be changed by different semaphores.
bool  wait = false; // this indicated whether a writing is waiting.
```



### Reader Process Code

Following is the code for the reader process :

```cpp
//Reader Process
wait(in,process_id); //wait on the "in" semaphore.
var_start++;//increment var_start since this process will now start reading.
 signal(in);

//Read  data.  "critical section".

 wait(out,process_id); //wait on the out semaphore.
 var_completed++;//increment var_completed since we have completed reading.
if(wait && var_started == var_completed)
{
    signal(writer);
}
 signal(out);
```



### Writer Process Code

Following is the code for the writer process : 

```cpp
//Writer Process
 wait(in,process_id);
 wait(out,process_id);
if(var_start == var_completed)
{
    signal(out);
}
else
{
    wait = true;
      signal(out);
       wait(writer,process_id);
    wait = false;
}

//Write the data. This is the "critical section"

   signal(in);
```



### Explanation on how it works

The starve-free solution works on this method : Any number of readers can simultaneously read the data.  A writer will make its presence known once it has started waiting by setting `wait` to true. 

Once a writer has come, no new process that comes after it can start reading. This is ensured by the writer acquiring the `in` semaphore and not leaving it until the end of its process. This ensures that any new process that comes after this (be it reader or writer) will be queued up on `in`.  Now, from the looks of it, it might seem like this is a method that is biased towards writer and may lead to starvation of readers. However, this is where the FIFO nature of semaphore comes into picture. 

Suppose processes come as `RRRWRWRRR`. Now, by our method the first three readers will first read. Then, when the next process which is a writer takes the `in` semaphore, it won't give it up until it's done. Now, suppose by the time this writer is done writing, the next writer has already arrived. However, it will be queued up on the `in` semaphore. Thus, it won't get to start writing before the process before it, the reader process has completed. Thus, the reader process will get done and then the writer process will again block any more processes from starting and so on.

To explain the writer process further, in case there's no other reader, it doesn't wait at all. Otherwise, it sets `wait` to true and waits on the `writer` semaphore. Also, to make the `writer` semaphore synchronizable across both the process where one process only executes `wait()` and other only executes `signal()`, we initialize it to `0`.

To explain the reader process further, it firsts increments the `var_start` then reads the data and then increments the `var_completed`. After this, it checks both these variables are equal. If they are and a writer is waiting (found from `wait`), it signals the `writer` semaphore and finishes.

Thus, it will be pretty clear how we manage to make a starve-free solution for the readers-writers problem with a FIFO semaphore. We can say that all processes will be handled in a FIFO manner. Readers will allow other readers in the queue to start reading with them but writers will block all other processes waiting in the queue from executing before it finishes. In this way, we can implement a reader-writer solution where no process will have to indefinitely wait leading to starvation.


The main idea here is that a Writer indicates to Readers its necessity to access the working area. At the same time no new Readers can start working. Every Reader
leaving the working area checks if there is a Writer waiting and the last leaving Reader signals Writer that it is safe to proceed now. Upon completing access to the
working area Writer signals waiting Readers that it finished allowing them to access the working area again.
### References

- [arXiv:1309.4507](https://arxiv.org/abs/1309.4507)
- Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts

