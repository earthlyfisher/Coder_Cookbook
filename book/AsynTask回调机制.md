#异步任务回调
##利用`FutureTask`实现回调
```java
Callable<Integer> callable = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                return new Random().nextInt(100);
            }
        };

        FutureTask<Integer> future = new FutureTask<Integer>(callable);
        new Thread(future).start();
        try {
            Thread.sleep(5000);// 可能做一些事情
            System.out.println(future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
```
##利用线程池`submit`实现机制
###利用Future集合方式实现的Future是按照添加的顺序排列的
```java
ExecutorService threadPool = Executors.newFixedThreadPool(3);
        List<Future<String>> list = new ArrayList<Future<String>>();
        for (int i = 1; i < 5; i++) {
            final int taskID = i;
            list.add(threadPool.submit(new Callable<String>() {
                public String call() throws Exception {
                    TimeUnit.SECONDS.sleep((5 - taskID));
                    return Thread.currentThread().getName() + ":" + taskID;
                }
            }));
        }
        // 可能做一些事情
        for (int i = 0; i < 4; i++) {
            try {
                System.out.println(list.get(i).get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
```
###提交到CompletionService中的Future是按照完成的顺序排列的
```java
ExecutorService threadPool = Executors.newFixedThreadPool(3);
        CompletionService<String> cs = new ExecutorCompletionService<String>(threadPool);
        for (int i = 1; i < 5; i++) {
            final int taskID = i;
            cs.submit(new Callable<String>() {
                public String call() throws Exception {
                    TimeUnit.SECONDS.sleep((5 - taskID));
                    return Thread.currentThread().getName() + ":" + taskID;
                }
            });
        }
        // 可能做一些事情
        for (int i = 1; i < 5; i++) {
            try {
                System.out.println(cs.take().get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
```
