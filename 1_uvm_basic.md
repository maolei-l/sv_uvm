# uvm的基础知识  

主要包括5个部分：  
1. component和object之分  
2. 树形结构  
3. filed automation机制  
4. 打印信息机制  
5. config_db机制  

## component和object机制  

component继承于object，但是拥有着object不具备的特性：  
1. 在new函数中，指定parent参数，从而形成一种树形的组织结构  
2. 有phase自动执行的特点  


object：  
uvm_sequence_item：从该类派生出transaction，而不是从uvm_transaction中派生。其中uvm_sequence也派生自uvm_transaction。  
uvm_sequence：所有的sequence派生自uvm_sequence。sequence是sequence_item的组合。sequence直接与sequencer打交道。  
config：主要功能是规范验证平台的行为方式。注意与config_db的区别。  
除此之外，还包括寄存器模型，uvm_phase。  

component：所有的组件。  

object的宏：  
uvm_object_utils：将该类注册到factory中。  
uvm_object_utils_begin uvm_object_utils_end：用于field_automation机制  

uvm_component的宏：  
uvm_component_utils:将改类注册到factory中。  

component：为了在构建一个完整的树形结构，我们需要在实例化一个组件时，需要指定他的parent。同时在每一个component的内部，维护一个数组m_children。当一个组件中的某个组件实例化时，将该组件的指针加入到上一级component中的m_children中。  

整个uvm树根有且只有唯一个，即uvm_top（uvm_root）。不采用uvm_test_top，是因为有时候，在new一个组件时，其parent参数是写的null，此时将会把该组件挂到uvm_top中。从而实现唯一一个根。  

## field automation机制  

TODO 为什么数组类型的filed automation机制不需要采用三个参数？两者应该没有区别啊。  

数组类型有：动态数组、静态数组、队列以及联合数组。  

该机制提供了以下函数：  
1. copy函数：只能复制，不能实例化左值。  
2. compare函数：比较两个实例是否相同。  
3. pack_bytes：打包为byte流。  
4. unpack_bytes：将byte流逐一恢复到某个类的实例。  
5. pack：打包为bit流。  
6. unpack：将bit流逐一恢复到某个类的实例中。  
7. pack_ints  
8. unpack_ints  
9. print函数  
10. clone函数：注意与copy函数的区分。  

使用Flag标志位实现不同的功能。但是在一些特殊情形时，也可以结合if语句。  
具体流程为：不清楚某一种类型的数据是否存在，此时用一个整数来判断是否有，也可以通过这个整数在object_begin中结合if语句来注册相应的数据类型。  

## 信息控制  

可使用命令行控制：\<sim command>  +UVM_VERBOSIT=UVM_HIGH  

UVM默认了四种信息严重性：UVM_INFO、UVM_WARNING、UVM_ERROR以及UVM_FATAL。  

命令行控制为：\<sim command> +uvm_set_severity=\<comp>,\<id>,\current svertiy>,\<new severity>  

set_report_max_quit_count(count)：出现count个错误时，退出仿真  
还可以设置相应的技术目标，即那些错误需要被计算。  

设置那些错误应该被计数，可以计数的最大值、断点功能，以及那些信息可以被打印出来。  

这一节很简单，就是简单排列组合。  

## config_db机制  

该机制用于传递控制信息或者数据信息。  

uvm使用发信人的优先级最高，其次则看时间。  

应该避免非直线的设置，也就是在父节点的兄弟节点设置参数，因为不清楚父节点和兄弟节点的执行顺序。  
可以使用非直线的获取，存在两个component，其中的参数是一致的，因此使用get可以避免设置两次，从而减少错误的发生。  

异常情形：config中第二个参数是一个字符串，其实际意义应该是树形结构中的路径。但是如果字符串书写错误，系统并不会报错。此时可以通过check_config_usage来判断，他可以显示截止到此类调用时，有哪些参数是被设置过但是没有被获取过。  

print_config函数以及UVM_CONFIG_DB_TRACE函数一样可以调试  