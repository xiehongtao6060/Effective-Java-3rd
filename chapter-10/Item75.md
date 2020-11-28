# 在详细消息中包含失败捕获信息

当一个程序因为一个未捕获的异常而失败时，这个系统会自动打印出异常的堆栈信息。堆栈信息包含了异常的字符串表示，即调用异常的`toString`方法得到的结果。堆栈信息通常有异常的类名和它的详细信息组成。这通常是程序员或站点可靠性工程师在调查软件故障时所掌握的唯一信息。如果故障不容易重现，则可能很难或不可能获得更多的信息。因此，异常的`toString`方法应该尽可能地返回更多的关于失败原因的信息，这是非常重要的。换句话说，异常的详细信息应该捕获故障，以便进行后续分析。
