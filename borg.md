#Large-scale cluster management at Google with Borg

## Concepts
Job - 由一个或多个Task组成<br/>
Task -  a set of linux processes 类似k8s的pod<br/>
Cell - a set of machines <br/>
Cluster - 集群，一般由一个主Cell和多个附属Cell（测试、功能型）组成<br/>
Alloc - a reserved set of resources on a machine <br/>

## User perspective
1. Workload分类<br/>
按任务类型<br/>
①long-running services：日常业务，面向用户，对响应时间敏感，never go down<br/>
②batch jobs: 批处理任务，持续几秒~几分钟，对响应时间不敏感<br/>
按任务优先程度<br/>
①prod：优先度高，多数long-running是prod级别<br/>
②non-prod：优先度低，多数batch jobs是non-prod级别<br/>

2. Priority and Quota<br/>
①Priority: 按不重叠的优先级段划分，production段内Tasks间不能互相抢占，防止preemption cascade发生（高优任务抢占低优任务，低优任务抢占更低优，导致级联抢占）<br/>
②Quota: 某Task在某一优先级下，某一时刻能请求到的最大资源数量；over-sell低优Quota，低优Quota可能被批准，但是只有资源充足时才获得资源。最高优Quota等于实际资源数量。用这种策略应对使用者的overbuy。<br/>

3. Naming<br/>
基于Chubby的BNS服务，用于服务治理（发现、注册、健康检查等）。<br/>

4. Monitoring<br/>
每个Task都包含一个built-in HTTP Server用来报告任务健康信息和监控指标。同时提供UI界面和日志收集服务等。

## Borg architecture
1. Borgmaster
