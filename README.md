# Starve-free-reader-writer-problem

The readers-writers problem relates to an object such as a file that is shared between multiple processes.
Some of these processes are readers i.e. they only want to read the data from the object and some of the
processes are writers i.e. they want to write into the object.It is used to manage the synchronisation
so that there are no problems with the object data.For example:
Consider a situation where 2 or more writers (or atleast one reader and atleast one writer) access an 
object at the same time.This will create problems like malformation of data (when two or more writers write
simultaneously) or incomplete data (when a reader and a writer work at the same time on an object).

To solve this situation,
a writer should get exclusive access to an object i.e. when a writer is accessing the object,
no reader or writer may access it. However, multiple readers can access the object at the same time.

This can be implemented using semaphores.

### The Semaphore

For a starve free implementation,we need a semaphore that has queue based handling for waiting process
(or say FIFO handling).A semaphore is a variable or abstract data type used to control and manage access
to a common resource by multiple processes and avoid critical section problems (like deadlock or starvation)
in a system such as a multitasking operating system

```cpp
// Implementation of a FIFO semaphore in c++
struct Semaphore
{
    int val = 1;
    Queue* Q = new Queue();
    
    void wait(int pid)
    {
        val--;
        if(val < 0)
        {
            Q->push(process_id);
            blockotherprocess(); //To block the processes
            
        }
    }
    
    void signal()
    {
        val++;
        if(val <= 0)
        {
            int pid = Q->pop();
            wakeupprocess(pid); //To wakeup a process with id=pid
        }
    }
}

//Implement queue
struct Queue
{
    Node* front=NULL, rear=NULL;
    
   	void push(int val){
    
        Node* n = new Node();
        n->val = val;
        if(rear == NULL)
        {
            front = rear = n;
            
        }
        else
        {
            rear->next = n;
            rear = n;
        }
    }
    
    int pop()
    {
        if(front != NULL)
        {
            int val = front->val;
            front = front->next;
            if(front == NULL)
            {
                rear = NULL;
            }
            return val;            
        }
        else
        {
            return -1; // Queue is empty
        }
    }
}

// define Node for queue
Struct Node
{
    Node* next;
    int val;
}
```
Semaphores are initialised to 1.Writer semaphore is initialised to 0.
Next, Let's look over the implementation of reader and writer

```cpp
Semaphore *in = new Semaphore();
Semaphore *out = new Semaphore();
Semaphore *wrt = new Semaphore();
wrt->val = 0;//To synchronise across the both processes where one executes wait() and other executes signal()
int ctrin = 0; //How many reader processes enter or started
int ctrout = 0; //How many reader processes completed
waiting = false; //is a writer process waiting

//Reader process

in->wait(pid); //pid is id for the reader process
ctrin++;  //increase the counter for the reader process
in->signal() //signal in semaphore


//[Critical section] Reader is reading the data


out->wait();
ctrout++; // a reader process just completed

if (waiting==true && ctrin==ctrout)
   wrt->signal();
out->signal();


//Writer process

in->wait();
out->wait();
if (ctrin==ctrout)
   out->signal();   
else
  waiting=true;
out->signal();
wrt->wait();
waiting=false;

//[Critical section] a writer is writing data 

in->signal(); 

```

### Explanation

Thus,we see that in avoids starvation by allowing multiple readers to read data simultaneously 
but blocks all the processes when a writer is writing its data untill it finishes.
Although other processes are blocked when a writer process is executed,but FIFO nature of semaphore
is here to avoid the starvation.Thus no process waits for infinite time avoiding starvation.

Let's put it more clearly by taking an example of a process train.
Suppose our queue has a train `RRRWWRWRW` of process.
First, all 3 first reader processes are allowed to read the data
of object simultaneously.A writer process on 4th number sets its waiting to true
untill all reader processes are finished.As soon as ctrin==ctrout(All 4 readers have finished)
The writer process enters the critical section and further processes in queue is blocked untill 
the writer finishes.When the writer finishes,it releases the semaphore to allow the other waiting 
processes to get executed.
This procedure keeps going on untill our queue is empty.

### References
- [wikipedia] (https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)
- [tutorialspoint] (https://www.tutorialspoint.com/readers-writers-problem)
