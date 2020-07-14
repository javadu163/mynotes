# 这里记录了我自己迷惑的一些java知识

记录一下双校验锁实现单例模式
代码如下：

```
public class DoubleCheck {
private static volatile DoubleCheck dc;
private DoubleCheck(){}
private static DoubleCheck getInstance(){
if(dc==null){
    dc=new DoubleCheck();
    synchronized (DoubleCheck.class){
        if(dc==null){
            dc=new DoubleCheck();
                }
        }
      }
return dc;
}
```

这里我们要注意类对象要加static和volatile关键字。并对对象是否为空进行两次判断

