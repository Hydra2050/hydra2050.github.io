---
layout: post
title: "从dispatch_get_current_queue()说起"
date: 2015-02-10 10:12:36 +0800
comments: true
categories: 
---

`dispatch_get_current_queue`从iOS6.0开始被废弃掉，在《Effective Object-C 2.0》中，也单独有一个章节讲解。在队列同步操作的时候，有时需要检查当前的执行队列。在试图使用dispatch_get_current_queue判断当前队列的时候，仍然可能出错。

## 使用dispatch_get_current_queue()存在的问题

先看下面一段代码：

		dispatch_queue_t queueA = dispatch_queue_create("c.b.a", NULL);
        dispatch_queue_t queueB = dispatch_queue_create("c.b.b", NULL);
        dispatch_sync(queueA, ^{
           dispatch_sync(queueB, ^{
               dispatch_block_t block = ^{
                  // ...
               };
               dispatch_sync(queueA, ^{
                   block();
               });
           });
        });
    

运行会发现死锁的情况，尝试使用dispatch_get_current_queue()来解决：

	dispatch_sync(queueA, ^{
	           dispatch_sync(queueB, ^{
	               dispatch_block_t block = ^{
	                  // ...
	               };
	               if(dispatch_get_current_queue() == queueA){
	                   block();
	               }else{
	                   dispatch_sync(queueA, ^{
	                       block();
	                   });
	               }
	           });
	        });
	        
运行后发现，同样会死锁。dispatch_get_current_queue判断，当前队列为queueB。

如果把每个队列看做一个管道，将管道B放在管道A中，dispatch_get_current_queue只能看到自己这一层，没法打破管道。

## dispatch_get_specific()试一试

尝试使用dispatch_get_specific()关联键值对的方式判断当前队列：

		dispatch_queue_t queueA = dispatch_queue_create("c.b.a", NULL);
        dispatch_queue_t queueB = dispatch_queue_create("c.b.b", NULL);
        static int kQueueSpecificA;
        static int kQueueSpecificB;
        dispatch_queue_set_specific(queueA, &kQueueSpecificA, (void *)CFSTR("queueA"), (dispatch_function_t)CFRelease);
        dispatch_queue_set_specific(queueB, &kQueueSpecificB, (void *)CFSTR("queueB"), (dispatch_function_t)CFRelease);
        dispatch_sync(queueA, ^{
           dispatch_sync(queueB, ^{
               dispatch_block_t block = ^{
                  // ...
               };
               void *retrievedValueA = dispatch_get_specific(&kQueueSpecificA);
               void *retrievedValueB = dispatch_get_specific(&kQueueSpecificB);
               if(retrievedValueA){
               		NSLog(@"queueA");
               }
               if(retrievedValueB){
               		NSLog(@"queueB");
               }
               if(retrievedValueA){
                   block();
               }else{
                   dispatch_sync(queueA, ^{
                            block();
                        });
               }
           });
        });
        
运行之后，发现仍然有问题。dispatch_get_specific()仍然无法破壁，窥视外面的管道。

## 破壁还需dispatch_set_target_queue()

dispatch_set_target_queue()将第一个参数指定在第二个参数的队列内执行，同时也使参数A的队列优先级与参数B的队列相同。

	dispatch_queue_t queueA = dispatch_queue_create("c.b.a", NULL);
        dispatch_queue_t queueB = dispatch_queue_create("c.b.b", NULL);
        static int kQueueSpecificA;
        static int kQueueSpecificB;
        dispatch_queue_set_specific(queueA, &kQueueSpecificA, (void *)CFSTR("queueA"), (dispatch_function_t)CFRelease);
        dispatch_queue_set_specific(queueB, &kQueueSpecificB, (void *)CFSTR("queueB"), (dispatch_function_t)CFRelease);
        dispatch_set_target_queue(queueB, queueA);
            dispatch_sync(queueB, ^{
                dispatch_block_t block = ^{
                    // ...
                };
                void *retrievedValueA = dispatch_get_specific(&kQueueSpecificA);
                void *retrievedValueB = dispatch_get_specific(&kQueueSpecificB);
                if(retrievedValueA){
                    NSLog(@"queueA");
                }
                if(retrievedValueB){
                    NSLog(@"queueB");
                }
                if(retrievedValueA){
                    block();
                }else{
                    dispatch_sync(queueA, ^{
                        block();
                    });
                }
            });
           
  这样执行的结果就没有问题了~  
  

## dispatch_set_target_queue()注意的问题

执行上面的代码会发现，`retrievedValueA`和`retrievedValueB`都不为空。是的，这两个管道已经互通状态了！所以在使用的时候，判断的时候也需要注意。

