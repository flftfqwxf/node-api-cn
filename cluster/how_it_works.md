
<!--type=misc-->

The worker processes are spawned using the [`child_process.fork()`][] method,
so that they can communicate with the parent via IPC and pass server
handles back and forth.

`worker`子进程是使用child_process.fork（）方法生成的，以便它们可以通过IPC与父进程通信，并且来回传递服务器句柄。

The cluster module supports two methods of distributing incoming
connections.

集群模块支持两种方式来分配传入连接。

The first one (and the default one on all platforms except Windows),
is the round-robin approach, where the master process listens on a
port, accepts new connections and distributes them across the workers
in a round-robin fashion, with some built-in smarts to avoid
overloading a worker process.

第一种方式（除了Windows之外的所有平台上都是默认的）：循环模式，主进程在端口上监听，接受新的连接，并以循环方式分配给`worker`子进程，其中一些内置方法智能的避免重载`worker`进程。

The second approach is where the master process creates the listen
socket and sends it to interested workers. The workers then accept
incoming connections directly.

第二种方法是主进程创建`socket` 监听，并将其发送给感兴趣的`worker`。 `worker`直接接受传入的连接。

The second approach should, in theory, give the best performance.
In practice however, distribution tends to be very unbalanced due
to operating system scheduler vagaries. Loads have been observed
where over 70% of all connections ended up in just two processes,
out of a total of eight.

在理论上，第二种方法应该是最好的性能。 然而，实际上，由于操作系统调度程序的变化，分配往往非常不平衡。 已经观察到负载，其中超过70％的连接最终在两个过程中，总共八个。

Because `server.listen()` hands off most of the work to the master
process, there are three cases where the behavior between a normal
Node.js process and a cluster worker differs:

因为server.listen（）将大部分工作交给主进程，所以有三种情况，即正常的Node.js进程和集群员之间的行为有所不同：

1. `server.listen({fd: 7})` Because the message is passed to the master,
   file descriptor 7 **in the parent** will be listened on, and the
   handle passed to the worker, rather than listening to the worker's
   idea of what the number 7 file descriptor references.
2. `server.listen(handle)` Listening on handles explicitly will cause
   the worker to use the supplied handle, rather than talk to the master
   process.  If the worker already has the handle, then it's presumed
   that you know what you are doing.
3. `server.listen(0)` Normally, this will cause servers to listen on a
   random port.  However, in a cluster, each worker will receive the
   same "random" port each time they do `listen(0)`.  In essence, the
   port is random the first time, but predictable thereafter.  If you
   want to listen on a unique port, generate a port number based on the
   cluster worker ID.

There is no routing logic in Node.js, or in your program, and no shared
state between the workers.  Therefore, it is important to design your
program such that it does not rely too heavily on in-memory data objects
for things like sessions and login.

Node.js或程序中没有路由逻辑，`worker` 之间没有共享状态。因此，重要的是设计您的程序，使其不会太依赖内存数据对象进行会话和登录。

Because workers are all separate processes, they can be killed or
re-spawned depending on your program's needs, without affecting other
workers.  As long as there are some workers still alive, the server will
continue to accept connections.  If no workers are alive, existing connections
will be dropped and new connections will be refused.  Node.js does not
automatically manage the number of workers for you, however.  It is your
responsibility to manage the worker pool for your application's needs.

因为`worker`都是独立的过程，所以根据你的程序的需要，他们可能被杀死或重新产生，而不影响其他工作人员。 只要有一些`worker`还处于连接中，服务器就会继续接受连接。 如果没有`worker`连接中，现有的连接将被丢弃，新的连接将被拒绝。 然而，Node.js不会自动管理您的`worker`。 您有责任根据您的应用需求管理工作池。



