In a typical quad core machine you have 4 cores

Refer diagram below:
![QuadCoreMemory](https://github.com/nnsantosh/java-memory-model/blob/master/quadCore_machine_memory.jpg)

Each core has registers which are nothing but memory area which is very small but within the core and hence fastest to retrieve any value from here or store value.
L1 cache is memory area set aside for each of the core.
L2 cache is memory area that could be shared between two cores.
L3 cache is memory area that could be shared across all cores.
RAM is memory area that could be shared across all cores.

Farther the memory from the core it will induce more latency but will have more size and will be cheaper.

Refer diagram below:
![Volatile](https://github.com/nnsantosh/java-memory-model/blob/master/volatile.jpg)
Consider we have a dual core machine and we have below class:

public class VolatileVisibility{
   int x = 0;

  public void writerThread(){
    x = 1;
  }

  public void readerThread(){
    int r2 = x;
  }
}

Let us assume we create instance of this class. And we use this instance in two separate threads. One will be the writerThread and other will be the readerThread.
The writerThread only calls the method writerThread and the readerThread only calls the method readerThread.

Let us assume both these threads run in parallel. So writerThread runs on core1 and readerThread runs on core2.
Initially the value of x is in shared cache and it is 0.
writerThread sets the value of x to 1 and this is in local cache of core1.
Now when readerThread reads the value of x it will read from shared cache and the value of x is still 0 and it will set value of r2 to 0 in local cache of core2.

So to avoid this problem whenever value of x changes it needs to be flushed back to shared cache so that any other thread accessing the value of x always gets the updated value.

To do this "volatile" keyword is used in java for declaring variable x.

public class VolatileVisibility{
   volatile int x = 0;

  public void writerThread(){
    x = 1;
  }

  public void readerThread(){
    int r2 = x;
  }
}

JMM is a specification which guarantees visibility of fields (aka as happens before) amidst reordering of instructions.
JMM rules have to be implemented by all the JVMs.

JMM has a rule that any fields that are written to, before the value of x is written those values should be updated and visible to any other thread after it has read the value of x.

Example:
public class VolatileVisibility{
  int a = 0, b = 0, c = 0;
   volatile int x = 0;

  public void writerThread(){
    a = 1;
    b = 1;
    c = 1;

    x = 1; //write of x
  }

  public void readerThread(){
    int r2 = x; //read of x

    int d1 = a;
    int d2 = b;
    int d3 = c;


  }
}

This concept of happen before is not applicable to volatile but also :
1. Synchronized
2. Locks
3. Concurrent collections
4. Thread operations (join, start)

final fields(special behavior)

public class SynchronizedFieldVisibility{
  int a = 0, b = 0, c = 0;
   int x = 0;

  public void writerThread(){
    a = 1;
    b = 1;
    c = 1;

    synchronized(this){
      x = 1;
    }
  }

  public void readerThread(){
    synchronized(this){
      int r2 = x;
    }

    int d1 = a;
    int d2 = b;
    int d3 = c;

  }
}

The above thing has same effect as volatile keyword because of the use of synchronized block. But the synchronized keyword should be applied on the same object.

The best practice would be to put all the variables in synchronized block.
