# 通信协议TLM(Transaction Level Modeling)  

建立两个组件之间的通道，提供阻塞和非阻塞等获取数据的方法。  

## 操作类型  

put操作：发起人到目标的信息流动  
get操作：发起人向目标寻求数据  
transport操作：类似于request- response操作  

动作的发起者接口称之为port，动作的目标组件接口为export。其次还有imp接口，imp接口在通信时用于被动承担者。  
三个接口的控制流优先级顺序为：port、export以及imp。  

## 接口类型  

均有blocking_put_port、nonblocking_put_port以及put_port接口。  

## imp类型  

整个通信的落脚点均为imp。对于多个组件之间的关联，则采取fifo的形式。  
