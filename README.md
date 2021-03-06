# Coroutine  
This is a simple coroutine based on ucontext.  
  
# Example
```
#include <stdio.h>
#include <coroutine.h>

void coroutine(Scheduler_t* scheduler,CoHandle_t handle)
{
    for(int i = 0; i < 10; i++)
    {
        // print i
        printf("coroutine : %d\n",i);
        CoYield(scheduler,handle,1);//yield
    }
}

int main()
{
    Scheduler_t* scheduler = CoCreateScheduler(10);
    CoHandle_t handle = CoCreate(scheduler,coroutine);
    //resume coroutine until CoResume return 0
    while(CoResume(scheduler,handle,1));
	
    return 0;
}
```
This code will print 
```
coroutine : 0
coroutine : 1
coroutine : 2
coroutine : 3
coroutine : 4
coroutine : 5
coroutine : 6
coroutine : 7
coroutine : 8
coroutine : 9
```

# APIs
```
/* function 	: CoCreateScheduler
 *
 * description 	: Create a scheduler
 *
 * parameters	: listLength - max quantity of coroutinr in this scheduler
 *
 * return       : a pointer of Scheduler_t , NULL means failed
 */
Scheduler_t* CoCreateScheduler(size_t listLength);





/* function 	: CoDeleteScheduler
 *
 * description 	: Delete a scheduler
 *
 * parameters	: scheduler - scheduler to delete
 *
 * return       : void
 */
void CoDeleteScheduler(Scheduler_t* scheduler);




/* function 	: CoResume
 *
 * description 	: resume a coroutine
 *
 * parameters	: scheduler 	- scheduler of coroutine
 *                handle        - handle of coroutine
 *                yield_rvalue	- will be the return value of previous CoYield invocation
 *                              - if yield_rvalue is 0 ,  CoYield will return 1
 *
 * return       : 3rd parameter of next CoYield invocation , 0 means failed
 */
int CoResume(Scheduler_t* scheduler,CoHandle_t handle,int yield_rvalue);




/* function 	: CoYield
 *
 * description 	: yield a coroutine
 *
 * parameters	: scheduler     - scheduler of coroutine
 *                handle        - handle of coroutine
 *                resume_rvalue	- will be the return value of previous CoResume invocation
 *                              - if resume_rvalue is 0 , CoResume  will return 1
 *
 * return       : 3rd parameter of next CoResume invocation
 */
int CoYield(Scheduler_t* scheduler,CoHandle_t handle,int resume_rvalue);




/* function 	: CoCreate
 *
 * description 	: Create a coroutine
 *
 * parameters	: scheduler - scheduler of coroutine
 *                func      - function of coroutine
 *
 * return       : a handle of coroutine , -1 means failed
 */
CoHandle_t CoCreate(Scheduler_t* scheduler,CoFunc func);
```

# How to use
* Create a scheduler by CoCreateScheduler , it's parameter is the max quantity of coroutine , it will return a scheduler(NULL means failed). For example:
```
Scheduler_t* scheduler = CoCreateScheduler(10);
```

* Define your coroutine functions , the return value must be void , and parameters must be Scheduler_t* and CoHandle_t. as this:
```
void coroutine(Scheduler_t* scheduler,CoHandle_t handle)
{
    for(int i = 0; i < 10; i++)
    {
        printf("coroutine\n",);
        CoYield(scheduler,handle,1);
    }
}
```

* Create coroutines by CoCreate, it's parameters are Scheduler_t* and a coroutine function , it will return the handle of coroutine(-1 means failed). For Example:
```
CoHandle_t handle = CoCreate(scheduler,coroutine);
```

* Use CoResume and CoYield to resume and yield coroutines, their parameters are Scheduler_t* scheduler , CoHandle_t handle and a integer.  
  - the 3rd parameter of CoResume will be the return value of previous CoYield.  
  - the 3rd parameter of CoYield will be the return value of previous CoResume.  
  - the 3rd parameter should be bigger than 0 , but if 3rd parameter is 0 , previous CoResume or CoYield will return 1.
  - if CoResume 0 that means coroutine has terminated.