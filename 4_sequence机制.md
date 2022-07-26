# sequence机制  

sequence  
sequencer  
driver  

## sequence  

在不同的测试用例中，将不同的sequence设置成sequencer的main_phase的default_sequence。从而使得driver在不改动代码的时候，可以获得不同的激励。验证人员只需要根据测试点设计相应的sequence即可，极大的减少了环境所带来的影响。  

启动方式：  
1. 使用start函数直接启动  
2. 使用default_sequence启动(会自动调用start任务)。  
3. TODO  

当sequence启动后，自动执行body任务。  

## 同步  

通过virtual机制实现同步。  

在最顶层的virtual sequence中raise objection，从而起统一调度的作用。  


