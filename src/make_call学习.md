## simple_pjsua.c流程

<img src="../img/make_call流程.png" alt="img" style="zoom:50%;" />

## pjsua_call_make_call

<img src="../img/pjsua_call_make_call.png" alt="img" style="zoom:50%;" />

```c
status = pjsua_call_make_call(acc_id, &uri, 0, NULL, NULL, NULL);
```

### 1、参数详解

**acc_id ** 获得

```
pjsua_acc_add(&cfg, PJ_TRUE, &acc_id);
```

**uri** 用户传入

```
pj_str_t uri = pj_str(argv[1]);
```

