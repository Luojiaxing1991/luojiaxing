+ 简介
+ 软件调用栈
+ acpi_evaluate_object
+ acpi_ns_evaluate
+ acpi_ex_enter_interpreter

# 简介
内核驱动通过ACPI接口从BIOS中获取硬件资源信息，以及调用硬件强相关的操作方法。本文档主要描述内核驱动如何通过ACPI接口调用BIOS方法的具体实现。

# 软件调用栈
``` C
acpi_evaluate_object - +
                       | - acpi_ns_evaluate - +
                                              | - acpi_ex_enter_interpreter  //抢锁
                                              | - acpi_ps_execute_method     //执行方法
                                              | - acpi_ex_exit_interpreter   //释放
```

# acpi_evaluate_object

该函数定义在 文件./drivers/acpi/acpica/nsxfeval.c中，函数头描述如下：
``` C
/*******************************************************************************
 *
 * FUNCTION:    acpi_evaluate_object
 *
 * PARAMETERS:  handle              - Object handle (optional)
 *              pathname            - Object pathname (optional)
 *              external_params     - List of parameters to pass to method,
 *                                    terminated by NULL. May be NULL
 *                                    if no parameters are being passed.
 *              return_buffer       - Where to put method's return value (if
 *                                    any). If NULL, no value is returned.
 *
 * RETURN:      Status
 *
 * DESCRIPTION: Find and evaluate the given object, passing the given
 *              parameters if necessary. One of "Handle" or "Pathname" must
 *              be valid (non-null)
 *
 ******************************************************************************/
```

函数描述翻译过来是查找并评估一个给定的对象

## struct acpi_evaluate_info *info

函数一开始就分配了一个 struct acpi_evaluate_info 的结构体，暂时还不知道该结构体的作用，但大概率会贯穿本函数的执行。

## 寻找prefix_node和relative_pathname

就像我们去看ACPI表一样，每一个ACPI对象(有可能这个对象并不是一个设备，有可能是设备资源里面的一个小节，如_RST)会有自己的一个节点(node)，以及他在这个节点里面的偏移。
prefix_node和relative_pathname就是提供上述这两项信息，帮助程序找到对应的ACPI表项内容。并保存在 info->prefix_node 和 info->relative_pathname。

## 保存 external_params
这个就无需多讲了，例如 HNS在复位解复位的时候会通过 external_params 来告知port和复位type

## 参数检查
主要检查是否多传，或者少传了一些参数

## 【重点】 acpi_ns_evaluate 开始评估对象
这个我们单独一节来描绘，此处跳过

## 收集返回值
对于部分object，协议规定是没有返回值的，例如_RST，那么这段代码不执行；如果协议规定object带返回值，那按照规则进行返回值的收集。

# acpi_ns_evaluate

该函数定义在 ./drivers/acpi/acpica/nseval.c中，函数头描述如下：
``` C
/*******************************************************************************
 *
 * FUNCTION:    acpi_ns_evaluate
 *
 * PARAMETERS:  info            - Evaluation info block, contains these fields
 *                                and more:
 *                  prefix_node     - Prefix or Method/Object Node to execute
 *                  relative_path   - Name of method to execute, If NULL, the
 *                                    Node is the object to execute
 *                  parameters      - List of parameters to pass to the method,
 *                                    terminated by NULL. Params itself may be
 *                                    NULL if no parameters are being passed.
 *                  parameter_type  - Type of Parameter list
 *                  return_object   - Where to put method's return value (if
 *                                    any). If NULL, no value is returned.
 *                  flags           - ACPI_IGNORE_RETURN_VALUE to delete return
 *
 * RETURN:      Status
 *
 * DESCRIPTION: Execute a control method or return the current value of an
 *              ACPI namespace object.
 *
 * MUTEX:       Locks interpreter
 *
 ******************************************************************************/
```
有上述描述可知，该函数可以重入，但是会锁 解释器
从这个函数的函数头可知，一个info对应着一个ACPI namespace object(ACPI命名空间对象)。

## 找到 该info对应的 ACPI命名空间对象
保存在 info->node 中

## 获取parameter以及检查parameter个数

## 执行Method或者获取object value
函数内有描述，目前这个接口函数能够支持的是 Method，非Method以及其他没法评估的object
``` C
	/*
	 * Three major evaluation cases:
	 *
	 * 1) Object types that cannot be evaluated by definition
	 * 2) The object is a control method -- execute it
	 * 3) The object is not a method -- just return it's current value
	 */
```

# acpi_ex_enter_interpreter

该函数位于 ./drivers/acpi/acpica/exutils.c ，函数头描述如下：
``` C
/*******************************************************************************
 *
 * FUNCTION:    acpi_ex_enter_interpreter
 *
 * PARAMETERS:  None
 *
 * RETURN:      None
 *
 * DESCRIPTION: Enter the interpreter execution region. Failure to enter
 *              the interpreter region is a fatal system error. Used in
 *              conjunction with exit_interpreter.
 *
 ******************************************************************************/
```
该函数用于进入解释器执行域，用于锁定

## acpi_ut_acquire_mutex
``` C
acpi_ut_acquire_mutex - +
                        | - acpi_os_acquire_mutex - +
                                                    | - acpi_os_wait_semaphore - +
                                                                                 | - down_timeout
```

## acpi mutex
acpi的mutex存储在 acpi_gbl_mutex_info 这个数组中，通过 acpi_os_create_mutex ->  acpi_os_create_semaphore 创建了一个二进制信号量
