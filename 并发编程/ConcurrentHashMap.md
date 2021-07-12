## ConcurrentHashMap

* put
* get

### Hash表

* hash函数，也叫散列函数，主要作用是把一个任意长度的数通过散列算法，变成一个固定长度的数据。

  * MD5、SHA--- 32长度编码。

    



我们还可能会遇到处于运行期且非阻塞的状态的线程，这种情况下，直接调用Thread.interrupt()中断线程是不会得到任响应的，如下代码，将无法中断非阻塞状态下的线程：



https://blog.csdn.net/javazejian/article/details/72828483?locationNum=5&fps=1