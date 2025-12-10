---
sidebar_position: 1
---

零、0625汇报遗留问题
============

1、ci权限控制先出方案，还是要对ci数据有一定限制，特别是写权限 (后移，非紧急)  
2、模型枚举值 需支持单选/多选 ，且改定义时需考虑存量数据,比如单选改多选 或新增枚举值 需控制不影响/删除存量数据；（添加支持）  
3、需支持ci批量更新属性值功能，需考虑是否有约束限制或影响自动关联；（添加支持）  
4、支持ci数据批量导出、导入（添加支持，导入也需要跟批量修改一样判断能否支持）  
5、支持关系配置后的按规则自动全量初始化  （添加支持）

![](http://localhost/server/index.php?s=/api/attachment/visitFile&sign=2fd95f8af8e974dab97b734be3168ea5)
  

一、性能优化-MySQL的查询增加缓存 （谭）
=======================

要求：缓存可一次构建或被动构建，当MySQL数据变化时按需更新缓存

范围：所有的findById

重点考虑：Redis做缓存还是JVM做缓存，决定了CMDB是单点部署还是集群部署，最好缓存层能与实现松耦合，先JVM，后续有必要扩展成集群时换种方式实现。

  

二、权限管理 （谭）
==========

按权限大小分：admin和normal俩角色，区别是只有admin才能增删改元数据，normal只能查看元数据和增删改ci、relation。

按使用人分为：个人用户和系统用户。个人用户接入LDAP，只进行鉴权即可，CMDB内部维护一个白名单，白名单内的用户才允许进入CMDB。 系统用户Token+IP白名单，允许来自某个IP地址的请求，校验token即可。

系统用户只有normal角色，个人用户按需分配。

要求：上述两点管理员UI可维护。

  

三、通知机制 （谭）
==========

要求：当CMDB需支持ci数据和ci关系数据变化时通过MQ、Http两种方式实时通知。

<table class="wysiwyg-macro" style="background-image: url('http://wiki.imgo.tv/plugins/servlet/confluence/placeholder/macro-heading?definition=e2NvZGV9&locale=zh_CN&version=2'); background-repeat: no-repeat;" data-macro-name="code" data-macro-id="b02f32dd-e57f-4175-9a5a-d057ef9d5efb" data-macro-schema-version="1" data-macro-body-type="PLAIN_TEXT"><tbody><tr><td class="wysiwyg-macro-body"><pre>private void sendEntityNofity(String ciTypeName, EntityNofity nofity){
        CompletableFuture<Boolean> isOK = nofifyProvider.sendEntityNofity(ciTypeName, nofity);
        isOK.whenComplete((result, ex) -> {
            metricProvider.addEntityNotifyMetric(ciTypeName, result);
        });
    }

    private void sendRelationNodify(RelationNofity nodify){
        CompletableFuture<Boolean> isOK = nofifyProvider.sendRelationNofity(nodify);
        isOK.whenComplete((result, ex) -> {
            metricProvider.addRelationNotifyMetric(nodify.getRelTypeName(), result);
        });
    }</pre></td></tr></tbody></table>

  

![](http://wiki.imgo.tv/download/attachments/106838488/image2025-5-27_18-40-12.png?version=1&modificationDate=1748343092000&api=v2 "运维安全部-项目空间 > 202506开发任务 > image2025-5-27_18-40-12.png")

任务：

1、支持多个外部系统订阅同一个ci\_type或ci\_type\_rel

2、MQ对接（message已开发完成），包括生产和消费

3、对外http通知，需要快速超时，需要metric指标化，这样当订阅某种ci或relation的外部系统访问有问题时可以通过Prometheus报警，留好审计日志，避免出现双方因是否通知过发生扯皮

4、UI上要能有入口，让用户知道某种ci或relation过去某段时间（例如1H）变化了多少次、发送到MQ多少次、发送给每个http多少次，成功率多少。

5、CMDB当ci、relation变化后往MQ中生产的这一下如果失败，需要存储下来，页面可查看到失败记录，针对投递MQ失败的记录支持查看和编辑消息并重新投递到MQ中。

Http的WebHook可参考**Jira**的WebHook产品设计，走POST请求。

![](http://wiki.imgo.tv/download/attachments/106838488/image2025-6-10_16-40-34.png?version=1&modificationDate=1749544835000&api=v2 "运维安全部-项目空间 > 202506开发任务 > image2025-6-10_16-40-34.png")![](http://wiki.imgo.tv/download/attachments/106838488/image2025-6-10_16-41-32.png?version=1&modificationDate=1749544893000&api=v2 "运维安全部-项目空间 > 202506开发任务 > image2025-6-10_16-41-32.png")

  

四、Collector设计与开发 （叶）
====================

详见：[Collector - 运维安全部-项目空间 - 大芒果仓库](https://wiki.imgo.tv/display/SRE/Collector)

任务：

1、确定collector与CMDB的报文协议，需涵盖全量校准、增量同步两部分

2、CMDB端处理逻辑

3、Collector插件基础包，包括健康检查、自动同步、人工同步、全量同步的能力。

  

五：生产环境搭建与容灾演练
=============

按我们当前设计，理论上只要MySQL和neo4j数据不丢，都可以恢复回来

需要做好数据备份与恢复，并通过演练真正完成一次重建。

  

六、Java性能优化 （叶）
==============

entityService中内存开销比较大，集合的处理需要改成流式。

  

七：ci数据支持tag能力 \--待定
===================

用neo4j这种tag的效果，但不通过lable，给每种资源类型内置一个tag属性即可。
