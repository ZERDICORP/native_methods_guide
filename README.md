# native_methods_guide :octopus:
#### A small guide to creating your own native methods in Java.

## What are native methods? :eyes:
**Native methods** are:
* Methods that have **no implementation** (are essentially a **function prototype**);
* Methods that describe functions written in **another programming language**;
* Methods that, when called, **run code from another programming language**.

Yes, In Java, it is possible to call functions from **other programming languages** (for example, those written in **C**).  

This mechanism is called **JNI**, and in this quick tutorial, I'll show you how to call your custom **C** function directly from Java code.

## Quick Step-By-Step Guide :older_man:

#### 1. Let's create two java classes
> As you can see from the class below, it has the static (static is not necessary) native method we need, called `compute` and accepting 2 integers as parameters.  
> You may also notice how we load a `libSum.so` file (we'll be sure to create one later) containing the implementation of our native method.
```java
// Sum.java

public class Sum {
  static {
    System.loadLibrary("Sum"); // will load "/usr/lib/libSum.so"
  }

  public static native int compute(int x, int y);
}
```
> Below, we simply call our native method, passing in some numbers as arguments.
```java
// Main.java

public class Main {
  public static void main(String[] args) {
    System.out.println(Sum.compute(5, 5));
  }
}

```

#### 2. Let's convert our Sum class to a C header file
> Just run the command below in the folder with the Sum class.
```
$ javac ./Sum.java -h .
```
> The result of the execution will be two files: `Sum.class` and `Sum.h`.  
> We need a second one. Its content should be as follows:
```c
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class Sum */

#ifndef _Included_Sum
#define _Included_Sum
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     Sum
 * Method:    compute
 * Signature: (II)I
 */
JNIEXPORT jint JNICALL Java_Sum_compute
  (JNIEnv *, jclass, jint, jint);

#ifdef __cplusplus
}
#endif
#endif
```
> Wow! Is that goblin language?  
> Yes, it is, so it's better not to touch anything (by the way, this is indicated by a comment at the beginning of the file).  
> We only need two lines starting with `JNIEXPORT jint...` to create our function implementation.

#### 3. Now let's write the implementation of the function in C
> We simply copy the function declaration from the header file, then add names to the parameters and write the function logic.  
> Also don't forget to include the header file.
```c
#include "Sum.h"

JNIEXPORT jint JNICALL Java_Sum_compute
  (JNIEnv *, jclass, jint x, jint y)
{ 
  return x + y;
}
```

#### 4. Creating a `libSum.so` file
> The command below will create a `.so` file and place it in `/usr/lib` (which is where Java will look for it).  
> P.S. Do not forget to specify the paths to your version of openjdk (in my case, this is version 17).
```
$ gcc -I/usr/lib/jvm/java-17-openjdk/include/ -I/usr/lib/jvm/java-17-openjdk/include/linux/ -o /usr/lib/libSum.so -shared Sum.c
```

#### 5. Finally, let's run our java program
```
$ javac Main.java && java Main
10
```

## Conclusion :coffee:
Using **native methods** turned out to be very simple and straightforward.  

Of course, in the example above, the task is very simple (add two numbers), and for this it is not particularly advisable to create a native method.  

***When should this approach be used?***

Well, here's a summary of the **facts**:
1. **DO NOT** use native methods as a potential speed-up for your Java program (yes, it seems like a paradox, but native methods are nearly 6 times slower than those written in Java itself);
2. This approach should be used in cases where the functionality of Java is **not enough** (this can happen if you want to manipulate some things at the **OS level**).
3. When trying to **access the Java environment** from native methods, there are **several pitfalls** associated with the work of the JVM and GC. You can read more about this [here](https://developer.ibm.com/articles/j-jni/).
