# PJSIP log

**日志用法**

使用之前需要初始化日志，但这一步是内部函数pj_init自己调用的，应用程序无需显示调用。

```cpp
/**
 * Internal function to be called by pj_init()
 */

pj_status_t pj_log_init(void);
*   PJ_LOG(3, ("main.c", "Starting hello..."));

*   PJ_LOG(3, ("main.c", "Hello world from process %d", pj_getpid()));
```

3表示日志级别，级别越高值越小，main.c表示日志发送者，即写日志的模块，类似于tag，后面就是格式化字符串。

**日志API**

```cpp
#define PJ_LOG(level,arg)	do { \

				    if (level <= pj_log_get_level()) { \

						pj_log_wrapper_##level(arg); \

				    } \
				    
				} while (0)
```

日志一般使用上面的宏，该宏会判断输入等级是否高于设置的等级（等级越高值越小，所以是<=），是则调用对应的等级函数。

pj_log_wrapper_1到pj_log_wrapper_6最终都是调用最底层的函数pj_log。

```cpp
/**
 * Write to log.
 *
 * @param sender    Source of the message.
 * @param level	    Verbosity level.
 * @param format    Format.
 * @param marker    Marker.
 */

PJ_DECL(void) pj_log(const char *sender, int level, const char *format, va_list marker);
```

pj_log根据样式的配置组织字符串，最后调用log_writer写入不同的对象。

**日志配置**

日志的配置非常灵活，有哪些配置呢

```cpp
/**
 * Log decoration flag, to be specified with #pj_log_set_decor().
 */

enum pj_log_decoration
{

    PJ_LOG_HAS_DAY_NAME   =    1, /**< Include day name [default: no] 	      */

    PJ_LOG_HAS_YEAR       =    2, /**< Include year digit [no]		      */

    PJ_LOG_HAS_MONTH	  =    4, /**< Include month [no]		      */

    PJ_LOG_HAS_DAY_OF_MON =    8, /**< Include day of month [no]	      */

    PJ_LOG_HAS_TIME	  =   16, /**< Include time [yes]		      */

    PJ_LOG_HAS_MICRO_SEC  =   32, /**< Include microseconds [yes]             */

    PJ_LOG_HAS_SENDER	  =   64, /**< Include sender in the log [yes] 	      */

    PJ_LOG_HAS_NEWLINE	  =  128, /**< Terminate each call with newline [yes] */

    PJ_LOG_HAS_CR	  =  256, /**< Include carriage return [no] 	      */

    PJ_LOG_HAS_SPACE	  =  512, /**< Include two spaces before log [yes]    */

    PJ_LOG_HAS_COLOR	  = 1024, /**< Colorize logs [yes on win32]	      */

    PJ_LOG_HAS_LEVEL_TEXT = 2048, /**< Include level text string [no]	      */

    PJ_LOG_HAS_THREAD_ID  = 4096, /**< Include thread identification [no]     */

    PJ_LOG_HAS_THREAD_SWC = 8192, /**< Add mark when thread has switched [yes]*/

    PJ_LOG_HAS_INDENT     =16384  /**< Indentation. Say yes! [yes]            */

};
```

从上面看，比较重要的有时间格式，发送者tag，和线程名字，可以通过pj_log_set_decor接口设置，比如。

```cpp
int param_log_decor = PJ_LOG_HAS_NEWLINE | PJ_LOG_HAS_TIME | PJ_LOG_HAS_MICRO_SEC;

pj_log_set_decor(param_log_decor);
```

设置日志等级

```cpp
PJ_DECL(void) pj_log_set_level(int level);
```

设置颜色

```cpp
PJ_DECL(void) pj_log_set_color(int level, pj_color_t color);
```

设置缩进排版

```cpp
/**
 * Add indentation to log message. Indentation will add PJ_LOG_INDENT_CHAR
 * before the message, and is useful to show the depth of function calls.
 *
 * @param indent    The indentation to add or substract. Positive value
 * 		    adds current indent, negative value subtracts current
 * 		    indent.
 */

PJ_DECL(void) pj_log_add_indent(int indent);

/**
 * Push indentation to the right by default value (PJ_LOG_INDENT).
 */

PJ_DECL(void) pj_log_push_indent(void);

/**
 * Pop indentation (to the left) by default value (PJ_LOG_INDENT).
 */

PJ_DECL(void) pj_log_pop_indent(void);
```

**输出对象**

pj_log最终调用log_writer写到输出对象，log_writer是一个函数回调指针

```cpp
/**
 * Signature for function to be registered to the logging subsystem to
 * write the actual log message to some output device.
 *
 * @param level	    Log level.
 * @param data	    Log message, which will be NULL terminated.
 * @param len	    Message length.
 */

typedef void pj_log_func(int level, const char *data, int len);

static pj_log_func *log_writer = &pj_log_write;
```

pj_log_write则在编译时指定编译为printk、printf等。对应log_write_printk.c、log_write_stdout.c，可以看出，如果我们想把日志保存的文件，自己实现一个pj_log_write函数，在函数里写到文件即可。