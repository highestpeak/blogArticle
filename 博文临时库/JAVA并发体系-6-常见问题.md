耗时操作的线程接口回调，异步操作的众多场景和写法

向线程中传递对象时的final问题，如果该状态是不可变的，那么随便多个线程访问都是没有问题的。但是final仅仅是不能修改该变量的引用，但是引用里边的数据是可以改的。

SimpleDateFormat线程不安全

https://juejin.im/post/5b4f44d4f265da0f93139dee#heading-5

https://stackoverflow.com/questions/12805698/why-cant-an-abstract-method-be-synchronized