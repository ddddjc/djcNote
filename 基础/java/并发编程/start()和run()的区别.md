start：

用start来启动线程，真正实现了多线程的运行，这时无需等待run方法体代码执行完毕而终结执行下面的代码。通过调用Thread类的start()方法来启动一个线程，这时线程处于就绪（可运行状态），并没内有运行，一旦得到cpu切片，就开始执行run方法，这里的方法run称为线程题，它包含了要执行的这个线程的内容，Run方法运行结束，线程随之停止

run：

run方法只是类的普通方法而已，如果直接调用run方法，程序依然只有主线程这一线程，其程序执行的路径还只有一条，还是要顺序执行，等待run方法执行完才开一执行后面的代码。

总结：调用start方法方可启动线程，而run方法只是thread的一个普通方法调用，还是在主线程里执行。