## epoll学习

1. [struct](https://so.csdn.net/so/search?q=struct&spm=1001.2101.3001.7020) epoll_event

  结构体epoll_event被用于注册所感兴趣的事件和回传所发生待处理的事件，定义如下：

  typedef [union](https://so.csdn.net/so/search?q=union&spm=1001.2101.3001.7020) epoll_data {
    void *ptr;
     int fd;
     __uint32_t u32;
     __uint64_t u64;
   } epoll_data_t;//保存触发事件的某个文件描述符相关的数据

   struct epoll_event {
     __uint32_t events;   /* [epoll](https://so.csdn.net/so/search?q=epoll&spm=1001.2101.3001.7020) event */
     epoll_data_t data;   /* User data variable */
   };

  其中events表示感兴趣的事件和被触发的事件，可能的取值为：
  EPOLLIN：表示对应的文件描述符可以读；
  EPOLLOUT：表示对应的文件描述符可以写；
  EPOLLPRI：表示对应的文件描述符有紧急的数可读；

  EPOLLERR：表示对应的文件描述符发生错误；
  EPOLLHUP：表示对应的文件描述符被挂断；
  EPOLLET：  ET的epoll工作模式；



 

 所涉及到的函数有：

1、epoll_create函数
  函数声明：int epoll_create(int size)
  
  功能：该函数生成一个epoll专用的文件描述符，其中的参数是指定生成描述符的最大范围；


2、epoll_ctl函数
  函数声明：int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)   在epfd上注册一个fd-events, 监听event->events的事件类型
  
  功能：用于控制某个文件描述符上的事件，可以注册事件，修改事件，删除事件。
  
  @epfd：由epoll_create生成的epoll专用的文件描述符；
  
   @op：要进行的操作，EPOLL_CTL_ADD注册、EPOLL_CTL_MOD修改、EPOLL_CTL_DEL删除；
  
   @fd：关联的文件描述符；
  
  @event：指向epoll_event的指针；
  
  成功：0；失败：-1


3、epoll_wait函数
  函数声明:int epoll_wait(int epfd,struct epoll_event * events,int maxevents,int timeout)

  功能：该函数用于轮询I/O事件的发生；

  @epfd：由epoll_create生成的epoll专用的文件描述符；

  @epoll_event：用于回传代处理事件的数组；

  @maxevents：每次能处理的事件数；

  @timeout：等待I/O事件发生的超时值；

  成功：返回发生的事件数；失败：-1



`epoll_ctl` 是用于控制 epoll 实例的系统调用之一，它用于向 epoll 实例注册或删除事件，并指定相应的文件描述符和事件类型。

`epoll_ctl` 的原型如下：

```
cCopy code
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

其中参数的含义如下：

- `epfd`：epoll 实例的文件描述符，通过 `epoll_create` 创建得到。

- ```
  op
  ```

  ：操作类型，可以是以下几种值之一：

  - `EPOLL_CTL_ADD`：添加一个新的文件描述符到 epoll 实例中。
  - `EPOLL_CTL_MOD`：修改一个已注册的文件描述符的事件类型。
  - `EPOLL_CTL_DEL`：从 epoll 实例中删除一个文件描述符。

- `fd`：要添加、修改或删除的文件描述符。

- `event`：一个指向 `struct epoll_event` 结构体的指针，用于描述要注册的事件类型和相关的数据。

`struct epoll_event` 结构体定义如下：

```
cCopy code


解释struct epoll_event {
    uint32_t events;  // 事件类型，可以是 EPOLLIN、EPOLLOUT、EPOLLERR 等
    epoll_data_t data; // 事件关联的数据，通常是一个联合体
};
```

`epoll_ctl` 的主要作用是管理 epoll 实例中的文件描述符和事件，可以添加、修改或删除特定的文件描述符以及相应的事件类型。通过这个系统调用，可以实现高效的 I/O 多路复用机制，使得应用程序能够监视多个文件描述符上的事件，并在事件发生时做出相应的处理。



```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/epoll.h>

#define MAX_EVENTS 10

int main() {
    int epoll_fd, nfds, i;
    struct epoll_event event, events[MAX_EVENTS];

    // 创建 epoll 实例
    epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }

    // 打开一个文件描述符（这里以标准输入描述符为例）
    int stdin_fd = STDIN_FILENO;

    // 配置要监听的事件
    event.events = EPOLLIN; // 监听读事件
    event.data.fd = stdin_fd; // 将标准输入描述符与事件关联

    // 将事件添加到 epoll 实例中
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, stdin_fd, &event) == -1) {
        perror("epoll_ctl: EPOLL_CTL_ADD");
        close(epoll_fd);
        exit(EXIT_FAILURE);
    }

    // 等待事件发生
    while (1) {
        nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            perror("epoll_wait");
            close(epoll_fd);
            exit(EXIT_FAILURE);
        }

        // 处理所有发生的事件
        for (i = 0; i < nfds; ++i) {
            if (events[i].events & EPOLLIN) {
                printf("Event on file descriptor %d: EPOLLIN\n", events[i].data.fd);

                // 从标准输入读取数据并进行处理
                char buffer[1024];
                ssize_t bytes_read = read(events[i].data.fd, buffer, sizeof(buffer));
                if (bytes_read == -1) {
                    perror("read");
                    close(epoll_fd);
                    exit(EXIT_FAILURE);
                }
                // 处理读取的数据
                // ...
            }
        }
    }

    // 关闭 epoll 实例
    close(epoll_fd);

    return 0;
}

```

