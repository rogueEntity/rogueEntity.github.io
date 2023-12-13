---
title: "Java Memory Structure"
excerpt: "memory management in java"

date: 2022-12-11
last_modified_at: 2022-12-11

categories:
  - study

tags:
  - Java
  - theoretical

comments: true

toc: true
toc_sticky: true
---

## Intro

![java_giphy.gif](https://media.giphy.com/media/Zei8AMzoPUTQGSPK02/giphy.gif){: .align-center}

↪ Last time, we talked about memory. This time, we're going to take a quick look at how to manage memory in Java. While writing this post, I found some things that I was mistaken about, and it was a meaningful time for me personally.

---

## Memory Structure in Java
### JVM

![jvm_01.jpg](/assets/images/posts/2022-12-11-java-memory-structure/jvm_01.jpg){: .align-center}

↪ The .java file we created is compiled into a .class file by javac (Java compiler). The Class file created in this way is loaded on the Runtime Data Area by the Class Loader. And it is done by the Execution Engine by converting it into byte code.

This time, we're going to look especially at <span style="color:red">**Runtime Data Area**</span>, or memory area. Runitme Data Area is the target of the GC (Garbage Collector), where every thread is assigned, as shown, and all <span style="color:red">**Shared Area**/span>.

The PC Register and Native Method Stack, which are excluded from the details in this post, will be briefly written down and moved on.

- <span style="color:red">**PC register**/span>: The address of the command running among the methods running in the thread. Command means each line of the byte code in which the class has been converted.
- <span style="color:red">**Native Method Stack**<span>: Area used to invoke a method written in a native programming language, such as C/C++, rather than Java.

### Shared Area

![shared_area_01.jpg](/assets/images/posts/2022-12-11-java-memory-structure/shared_area_01.jpg){: .align-center}

↪ It is a diagram of the shared memory area of the entire JVM structure. It can be largely viewed as <span style="color:red">**Heap**</span> area and <span style="color:red">**Metaspace**</span> area, but there was no Metaspace area before Java 8. The existing area is the Permanent area, where Meta information of Class, Meta information of Method, static variable, and constant information were stored.

Beginning with Java 8, this Permanent Area (PermGen) was replaced by Metaspace, and the static variable and constant information were changed to be stored in the Heap area. This change has been made to address structural issues that often arise with Out of Memory (OOM) problems, as developers store static variables and constants in PermGen, where fixed sizes are assigned. Therefore, the dynamic new Metaspace will be treated as a Native Memory Area managed at the OS level, not as a Heap area managed by the JVM.

### Heap & GC

![gc_01.jpg](/assets/images/posts/2022-12-11-java-memory-structure/gc_01.jpg){: .align-center}

↪ Among the shared memory areas, the area we will see to understand what is assigned to the memory as the code operates is the <span style="color:red">**Heap**</span> area. This Heap area is also divided in relation to the behavior of the GC because the instances dynamically assigned to the Heap area are stored, and in Java, the <span style="color:red">**GC**</span> (Garbage Collector) is responsible for returning dynamically assigned memory.

1. When an object is first created, it is stored in the <span style="color:red">Eden</span> area.
2. <span style="color:red">minor GC</span> occurs.
  1. The surviving object in the Eden area moves to the <span style="color:red">survivor</span> area. (S0,S1)
  2. Objects that survive in the survivor zone alternate between S0 and S1 until the age bit, the record of surviving minor GC, exceeds the setting (-XX:MaxTenuringThreshold) called <spanstyle="color:red">MaxTenuringThreshold</span>.
  3. If the MaxTenuringThreshold value is exceeded by the age bit, move to the <span style="color:red">Tenured</span> area.
3. If Major GC occurs, GC will occur in Tenured area.

GC roughly happens in the same process as above, and when GC happens, an event called <span style="color:red">**STW(STOP-THE-WORLD)**</span> occurs. As the name suggests, all other threads, except for the thread that runs the GC, stop working, and it is advantageous to run the GC at the right frequency because it takes longer for Major GC than Minor GC.

---

## Heap & Stack
### String

![string_giphy.gif](https://media.giphy.com/media/iFU36VwXUd2O43gdcr/giphy.gif){: .align-center}

↪ Now, before we take a moment to look at how allocations are made in Heap and Stack with the example code, we're going to talk briefly about Java's <span style="color:red">**String**</span> because we're going to have a string type in the example.

There are two ways to create a String object in Java.

1. String literal (quote for replacement)
2. How to use the 'new' operator

Because String is also a dynamically assigned object, of course, the new operator will be able to create an instance by assigning memory to the Heap area. So what's the difference between using String literal?

Inside the Heap area is <span style="color:red">**Constant Pool**</span>. When you create a String object using double quotation marks, it references the string stored here. And when you create another String object with the same string as the string generated here, it references the string as well.

In other words, if you created a String instance through the 'new' operator rather than the String literal, even a String object with the same string value would have a different reference value, but if you created multiple String instances with the same string value through the String literal, they would all reference the same string. (You can still use the <pan style="color:red">**intern**</pan> method to view the Constant Pool.)

Additionally, the strings stored inside the Constant Pool cannot be modified. Therefore, when an operation such as 'String string = 'front' + 'back';' is performed, three strings are created inside the Constant Pool: 'front', 'back', and 'front and back'. Naturally, if the words 'front' and 'back' are not used afterwards, they will be the target of the GC. For this reason, if string modifications occur frequently, <span style="color:red">**StringBuilder**</span> should be used to handle strings so that there is no performance problem.

Although the \+ operation uses StringBuilder internally since Java 5 version, it creates a new instance each time to call append() and return the String.

### Example Code & Visualization

![wtf_giphy.gif](https://media.giphy.com/media/xL7PDV9frcudO/giphy.gif){: .align-center}

↪ I'm also a bit confused, so we're going to look at how memory allocation works when the code runs one line with a more intuitive and simple example code.

### Code

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<Comics> myComicsList = new ArrayList<>();

        myComicsList.add(new Comics("Walking Dead", 1));
        myComicsList.add(new Comics("Walking Dead", 2));
        myComicsList.add(new Comics(new String("Walking Dead"), 3));

        printMyComics(myComicsList);
    }

    public static void printMyComics(ArrayList<Comics> _comicsList) {
        _comicsList.forEach(_rec -> {
            _rec.setTitle(_rec.getTitle() + " " + _rec.getEpisode());
            System.out.println(_rec.getTitle());
        });
    }
}

class Comics {
    private String title;
    private int episode;

    // constructor
    public Comics(String _title, int _episode) {
        this.setTitle(_title);
        this.setEpisode(_episode);
    }

    // getter & setter
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public int getEpisode() { return episode; }
    public void setEpisode(int episode) { this.episode = episode; }
}
```

↪ Let's look at the memory when the above code is executed.

### Visualization

![stack_and_heap_1.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_1.jpg){: .align-center}

↪ The process begins by storing the parameter variable of the main function, which is the entry point, in the stack. If I divide the pictures step by step, I will group them together several steps because I think the writing will be too long.

![stack_and_heap_2.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_2.jpg){: .align-center}

**1\.** The local variable, myComicsList, creates an ArrayList instance of Comics type and stores the address value.

**2\.** A new instance of Comics is created, and the string "Walking Dead" is not in <span style="color:red">**Constant Pool**</span>, so the Titles, a property of Comics, looks at it.  
The first element in the ArrayList looks at the newly created Comics instance.

**3\.** Because a new instance of Comics is created and the string "Walking Dead" is in <span style="color:red">**Constant Pool**</span>, the property of Comics created this time looks at it.  
The first element in the ArrayList looks at the newly created Comics instance.

![stack_and_heap_3.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_3.jpg){: .align-center}

**4\.** The `new` keyword creates a <span style="color:red">**String instance**</span>. A new instance of Comics is created, and the title property looks at this String instance.

![stack_and_heap_4.jpg](/assets/images/posts/2022-12-11-java-memory-structure/stack_and_heap_4.jpg){: .align-center}

**5\.** Calling printMyComics method pushes parameter_comicsList to Stack and looks at the instance myComicsList was looking at.

**6\.** Inside the forEach method of ArrayList, parameter _rec of the anonymous function defined as the arrow function looks at the first element of ArrayList that _comicsList is looking at.  
Because the string "Walking Dead 1" is not in <span style="color:red">**Constant Pool**</span>, the newly assigned property of Comics, title, looks at it.

**7\.** Inside the forEach method of ArrayList, the parameter _rec of the anonymous function defined as the arrow function looks at the second element of ArrayList that _comicsList is looking at.  
Because the string "Walking Dead 2" does not exist in <span style="color:red">**Constant Pool**</span>, the newly assigned property of Comics, title, looks at it.

**8\.** Inside the forEach method of ArrayList, the parameter _rec of the anonymous function defined as the arrow function looks at the third element of ArrayList that _comicsList is looking at.  
<span style="color:black;background-color:#ff666">There is no change in **Constant Pool** because none of the operands used in the addition operation of String are **literal**.</span>  
Saves the value of the String instance that title, property of Comics, was looking at as "Walking Dead 3".

---
