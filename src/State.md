# State

转载：[sip信令超时时间调整_pjsip的最长呼叫等待时间-CSDN博客](https://blog.csdn.net/croop520/article/details/78666799)

**UAC （呼叫方）状态机转换如下：**

![img](https://img-blog.csdn.net/20171129164001702?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3Jvb3A1MjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

刚开始呼叫时，sip_transaction的状态机处于tsx_on_state_null状态，拨打电话发出INVITE信令后，状态机转为Calling状态，处理函数：tsx_on_state_calling，应用可以收到Calling的状态通知。

之后会收到被叫方发来的100 try信令，状态机转为Processing状态，处理函数：tsx_on_state_proceeding_uac

在收到180 ringing的信令后，还是在tsx_on_state_proceeding_uac中处理，应用会收到Early的状态通知。

如果被叫方接听了该呼叫，会收到被叫方的200 OK信令，状态机会转为Terminated状态，处理函数：tsx_on_state_terminated，应用会收到Connecting的状态通知，之后会通过timeout方式，把状态转为Destroyed。Call的状态也转为Confirmed。

**Bye信令状态转换：**

![img](https://img-blog.csdn.net/20171129165213239?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3Jvb3A1MjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

发出Bye信令时，状态机转为Calling状态（tsx_on_state_calling），在收到200OK信令时，转为Completed状态（tsx_on_state_completed_uac），应用会收到Disconnected的状态通知

，之后通过timeout的方式，状态转为Terminated和Destroyed

**UAS（被叫方）状态机转换如下：**

***\*![img](https://img-blog.csdn.net/20171129170550757?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3Jvb3A1MjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)\****

Bye状态转换：

![img](https://img-blog.csdn.net/20171129170759124?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3Jvb3A1MjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 

被叫方的转换和呼叫方类似。