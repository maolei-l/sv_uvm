# UVM方法学  

验证的目的：模拟芯片在真实情形下，各种可能出现的情形  

根据上述定义，需要有以下组件完成验证的目的：  
1. driver：生成各种激励，将激励传送到DUT  
2. monitor：DUT在接收到driver发送的激励后，会产生相应的“结果”信号。monitor负责监控该信号  
3. reference model：用高级语言实现DUT的功能
4. scoreboard：将会对比refence model的结果和DUT（rtl实现）的结果  

TODO：补全验证的顶层框架图  

## uvm语言特性  

1. 验证平台中所有的组件应该派生自UVM的类  
2. uvm_component类要求每一个new函数指定两个参数：name，parent
3. UVM由phase管理验证平台的运行  
   1. objection机制：在phase中挂起及drop，可控制仿真平台的运行及终止。一般和sequence相伴相生  
   2. build_phase，函数phase  
      1. uvm启动后，自动执行该phase。在new函数之后，main函数之前执行。一般通过config_db来传递一些数据以及实例化成员变量等。  
      2. 先执行树根的build_phase，再执行树叶的build_phase。把整个树的build_phase建立完成后，再执行其他phase。
   3. connect_phase：在build_phase执行后，马上执行。与build不同的是，其执行是从树叶到树根。---->可以这么理解，在相关组件还没有例化时，你该如何连接他们；也只有在相关组件建立后，你才能连接他们。所以当build从上到下执行时，在最底层时，所有的组件已创建，此时执行connect，将各个组件的端口打通。
   4. main_phase，任务phase  
   5. report_phase，函数phase。在main执行后执行。  
4. 存在函数和任务  
   1. 任务：消耗时间  
   2. 函数：不消耗时间
   3. TODO 补全任务和函数
5. UVM的factory机制：自动创建一个类的实例并调用其中的函数（function）和任务（task）
   1. 该机制由uvm_component_utils宏实现
   2. 只要类使用了UVM_component_utils，则实例化时，该类的main_phase会自动调用  
6. interface：**避免使用绝对路径的一种方法**
   1. 在类中不能实例化接口，因此在driver、monitor以及scoreboard等模块中必须使用虚拟接口。  
   2. 在module中实例化接口  
   3. 此时interface实在module中实例，在某一个类中使用。在UVM中使用fonfig_db构建两者的桥梁  
      1. 使用set以及get两个函数，成功构造两者的通信。两者均为静态函数，不需要实例化，其由::引用便可得知。  
      2. uvm_config_db#(virtual my_if)::set(null, "uvm_test_top", "vif", input_if)
      3. uvm_config_db#(virtual my_if)::get(this, "", "vif", vif)  
7. transaction：将协议所需要的数据打包成一笔交易，其是uvm_object或是uvm_object的派生类  
   1. 所有的transaction均派生自uvm_sequence_item基类  
   ```sv
   class trac_name extends uvm_sequence_item;

   endclass
   ```
8. field_automation：print函数，compare函数以及copy函数，在每一个项目中都需要这三个函数，因此没必要每一个项目都需要自己写这些通用函数，而是通过field_automation机制自动化完成。
9. 发 

## uvm语言杂项

1. uvm_info宏  
2. uvm中的任何一个节点都有一个相应的字符串类型的路径索引，get_full_name  
3. 无论run_test的参数是什么，创建的实例名均为uvm_test_top。  
4. post_randomize  
5. 

## top.sv  

```sv
//TODO 补全timescale信息  
`timescale 1ns/1ps //时间精度  
`include "uvm_macros.svh"  

import uvm_pkg::*;
`include "my_driver.sv"
//TODO 还需要引用更多的自定义的sv文件  

//顶层模块的例化  
//各组件的例化  
//激励的编写等
```

## agent  

由于monitor和driver本质上是处理同一种协议，因此将monitor和driver封装成agent。而sequencer发送transaction，driver接受transaction，所以也将sequencer集成到agent中。不同的agent代表不同的协议。

1. 使用is_active变量控制该agent是否有driver  

### driver

1. 派生自UVM_driver类  
    派生类的格式为：
   ```sv  
    class my_driver extends uvm_driver;
        `uvm_component_utils(my_driver)//将my_driver注册到factory中的一张表  
        function new(string name = [自定义], uvm_component parent = [后续章节阐述]);
            super.new(name, parent);
        endfunction
        extern virtual task main_phase(uvm_phase phase);
    endclass
   ```
2. 因为driver永远都在驱动数据，从不产生transaction，有transaction就驱动。因此该组件永不停歇，在main_phase中使用while(1)达成上述目的。

工作流程：
1. get_next_item()任务获得一个新的req，并且驱动它。  
2. 某些其他操作。 
3. 使用item_done()告知sequencer，让sequencer将副本删掉。当没有调用done()函数时，sequencer将会重新发送原来的transaction，以保证发送的准确性。  

### monitor  

1. 派生自uvm_monitor类  
    ```sv
    class monitor_name extends uvm_monitor;
        virtual 接口类型 name；
        `uvm_component_utils(monitor_name)
        function new(string name = [自定义], uvm_component parent = [后续章节阐述])；
            super.new(name, parent);
        endfunction
    endclass
    ```
2. 该组件永不停歇，因此需要在main_phase中使用while(1)实现该目的  

### sequencer



## reference model  

reference model的main_phase中需要复制i_agt中的transaction，并进一定的运算，将结果写入scoreboard。  
因此涉及到一个问题：  
1. 数据是如何通信的？  
    解答：具体解决方法可以分为两步。第一步，在需要通信的组件中创建各自的端口；第二部，在需要通信的上一级组件中，将两个端口连接起来。  
TODO：通信较为复杂，在后面章节中详细阐述  

## scoreboard  

工作流程：  
1. 在初始化过程中，创建两个端口，分别连接DUT和reference model  
2. 创建两个队列，用于存放DUT和ref的数据。一般而言，ref的数据是实时的，因此在比对不通过以及DUT有数据，而ref没数据时，可以报错。  

## sequence机制  

sequence机制：一是，sequence，uvm_object类型；二是，sequencer，uvm_component。

sequencer：component组件。sequencer产生transaction，driver接受transaction。  

sequence：  
1. sequence派生自uvm_sequence  
2. sequence都存在一个body任务，当sequence启动后，自动执行body中的代码。
3. 工作流程：  
   1. 向sequencer发送一个请求，告知sequencer，我（sequence）将要向你发送数据了，你能接受吗？  
   2. sequencer接收到该信息后。第一，检查仲裁队列中是否有某个其他的sequence发送transaction请求；第二，检测driver是否有申请transaction。即判断能否接受transaction，以及判断是否有需求，即有没有必要接受。  
      1. 仲裁队列有请求，driver没需求。sequencer一直处于等待driver的状态，直到driver有需求。此时，sequencer同意sequence的请求，sequence得到sequencer的批准后，产生transaction并交给sequencer，后者将transaction传送给driver  
      2. 仲裁队列没请求，driver有需求。sequencer一直处于等待sequence的状态，知道sequence递交发送请求，sequencer马上同意该请求，然后就是transaction的传送流程了。  
      3. 仲裁队列有请求，driver有需求。transaction常规的流程。  
      4. 仲裁队列没请求，driver没需求。那就相安无事。  

driver和sequencer中通过通道进行联系，driver可以通过get_next_item(req)函数向sequencer发送需求申请。  

uvm_do宏：向sequencer发送transaction，driver取走该数据后，uvm_do并不会立刻返回执行下一次uvm_do宏，而是停留在哪里，知道driver返回item_done()。此时一次完成的uvm_do执行完毕。  

sequence的关联方法：在某一个component中的main_phase中启动该sequence即可。  
1. 声明一个seq  
2. 实例化seq  
3. seq.start(你需要传递给那个sequencer)  

## base_test  

base是整个验证环境的顶层。

1. 设置report_phase收集整个仿真时所发生的错误类型。  
2. 设置整个验证平台的超时退出时间。  
3. 通过config_db设置验证平台中某些参数的值。  

