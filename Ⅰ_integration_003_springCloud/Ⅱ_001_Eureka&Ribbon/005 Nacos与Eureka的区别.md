 **1.** Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式

 **2.** 临时实例心跳不正常会被剔除，非临时实例则不会被剔除

 **3.**  Nacos支持服务列表变更的消息推送模式，服务列表更新更及时

 **4.** Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式