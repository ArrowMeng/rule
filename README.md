config/rule.xml是规则引擎的配置文件，如果使用spring,请import进spring上下文配置文件里；

规则引擎采用了jboss drools的设计思路：

规则＝规则执行条件＋规则执行内容＋规则分组 + 规则优先级

规则的执行内容和执行条件建议以java脚本的形式存在数据库中，比如我使用了mysql,那么建表语句如下：

create table `rule` (

`id` int(10) unsigned not null auto_increment comment '规则主键',

`rule_name` varchar(50) not null default '' comment '规则名称',

`priority` int(4) not null default 0 comment '规则优先级，越大优先级越高',

`group_type` tinyint not null default 0 comment '分组类型 1：占位',

`group_name` varchar(50) not null default '' comment '分组名称',

`match_condition` varchar(100) not null default 'true' comment  '规则进入条件',

`execute_content` varchar(1000) not null default '' comment '规则执行内容',

`del_flag` tinyint(4) not null default 0 comment '是否删除 0 未删除 1已删除',

`op_time` timestamp not null default current_timestamp on update current_timestamp comment '操作时间',

primary key (`id`),

index (`group_type`)

)ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='wanglei@2014-11-25 规则配置表';

规则会被缓存在服务部署的机器内存里，所以需要提供一个后门用于刷新缓存,API如下：

RuleRunner.java

/**
* 
*Description:刷新规则内容<br> 
*  
*@author wanglei<br>
*@taskId <br> <br>
*/

public synchronized void refreshRules() {

   ruleList = ruleMapper.getRuleList();
   
   rulesCache = new HashMap<Integer,List<Rule>>();
   
}

后续会考虑使用memcache这种分布式的缓存，目前有没有使用，因为觉得有点杀鸡用牛刀。

最近发现阿里的diamond真是特别好用，如果公司有接入diamond的话，可以考虑直接用diamond存储规则，不过这样一来RuleManager就得再写一个了，也许这里应该最初就设计成一个工厂或者模板的模式。

规则的解析基于taocode的qlexpress,我这里做了简单的封装，以适应于我的项目，关于qlexpress，参考:

http://code.taobao.org/p/QLExpress/wiki/index/
