---
title: 鸟宿池中树，僧敲月下门：Good Taste in Code
date: 2018-02-10
---

　　作为一个程序员，按时完成业务需求是分内的责任，而作为工程师，我们也应当对代码有美的追求。
　　
　　Linus在2016年的TED中，举了一个小例子来说明代码品味的问题，C Style的伪代码如下：

```c
remove_list_entry(entry)
{

    prev = NULL;
    walk = head;
    
    // Walk the list
    
    while(walk != entry){
        prev = walk;
        walk = walk->next;
    }
    
    // Remove the entry by updating the
    // head or previous entry
    
    if(!prev)
        head = entry->next;
    else
        prev->next = entry->next;

}
```

```c
remove_list_entry(entry)
{
    // The "indirect" pointer points to the
    // *address* of the thing we'll update
    
    indirect = &head;
    
    // Walk the list, looking for the thing that
    // points to the entry we want to remove
    
    while((*indirect) != entry)
        indirect = &(*indirect)->next;
        
    // .. and just remove it
    *indirect = entry->next;
}
```
　　两段伪码都是为了实现删除链表中的指定节点。实现同样功能的两段代码，Linus认为第二段更具美感。
　　
> Linus:This,the first not very good taste approach,is basically how it's taught to be done when you start out coding.And you don't have to understand the code.The most interesting part to me is the last if statement.
> ......


> Linus:I don't want you understand why it doesn't have the if statement,but I want you to understand that sometimes you can see a problem in a different way and rewrite it so that a special case goes away and becomes the normal case.
> ......


> Linus:To me,the sign of people I really want to work with is that they have good taste.
> ......


> Linus:I sent you this stupid example that is not relevant because it's too small.Good taste is much bigger than this.Good taste is about really seeing the big patterns and kind of instinctively knowing what's the right way to do things.
> ......

　　追求代码之美不是为了炫技，我认为所谓“代码之美”，其实就是指代码的稳健、易维护、可复用。在大多数情况下，特别是在紧张的工期中，很难一次性写出高效优雅的代码。代码之美是建立在多次重构之上的，在重构中进步，在进步中重构，逐渐就会锤炼出具有美感的代码。


[TED:the Man Behind Linux（Linux背后的男人）](https://www.bilibili.com/video/av5140622/)

