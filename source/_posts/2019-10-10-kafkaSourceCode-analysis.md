---
title: kafkaServerStartable.startup() 主流程
date: 2019-10-10 15:20:02
categories: kafka
tags:
 - kafka
 - 源码
---

## KafkaServer.startup() 方法主流程

* shutdown & startupComplete 状态校验
* **isStartingUp** 标识设置为 true, 原子操作
* **`brokerState.currentState`** 标识为 Starting
* initZkClient: kafka 启动时，保证 kafka 基础业务目录和文件已创建
* getOrGenerateClusterId: `/cluster/id` 创建
* config.dynamicConfig.initialize(zkClient)
* **kafkaScheduler** 初始化 & 启动 : kafkaScheduler.startup()
* **metrics** 初始化: new Metrics(...)
* **_brokerTopicStats** 初始化: new BrokerTopicStats
* **quotaManagers** 初始化: QuotaFactory.instantiate(...)
* notifyClusterListeners(...)
* **logDirFailureChannel** 初始化: new LogDirFailureChannel(...)
* **logManager** 初始化 & 启动: logManager.startup()
* **metadataCache** 初始化: new MetadataCache(...)
* **tokenCache** 初始化: new DelegationTokenCache(...)
* **credentialProvider** 初始化: new CredentialProvider(...)
* **socketServer** 初始化 & 启动: socketServer.startup(..)
* **replicaManager** 初始化 & 启动: replicaManager.startup()
* 创建 **brokerInfo** 并注册到 zk 中: zkClient.registerBrokerInZk(brokerInfo)
* checkpointBrokerId(config.brokerId)
* **tokenManager** 初始化 & 启动: tokenManager.startup()
* **kafkaController** 初始化 & 启动: kafkaController.startup()
* **adminManager** 初始化: new AdminManager(...)
* **groupCoordinator** 初始化 & 启动: groupCoordinator.startup()
* **transactionCoordinator** 初始化 & 启动
* **authorizer** 初始化
* **fetchManager** 初始化
* **apis** 初始化，依赖上面初始化组件
    * **`socketServer.requestChannel`**
    * **replicaManager**
    * **adminManager**
    * **groupCoordinator**
    * **transactionCoordinator**
    * **kafkaController**
    * **zkClient**
    * config.brokerId
    * config: 配置文件
    * **metadataCache**
    * **metrics**
    * **authorizer**
    * **quotaManagers**
    * **fetchManager**
    * **brokerTopicStats**
    * clusterId
    * time: 配置文件
    * **tokenManager**
* **requestHandlerPool** 初始化，依赖上面初始化的组件
    * config.brokerId
    * **`socketServer.requestChannel`**
    * **apis**
    * time: 配置文件
    * config.numIoThreads: 配置文件
* Mx4jLoader.maybeLoad()
* config.dynamicConfig.addReconfigurables(...)
* **dynamicConfigHandlers** 映射初始化，Map[String, ConfigHandler]
    * ConfigType.Topic  ————>   new TopicConfigHandler(logManager, config, quotaManagers)
    * ConfigType.Client ————>   new ClientIdConfigHandler(quotaManagers)
    * ConfigType.User   ————>   new UserConfigHandler(quotaManagers, credentialProvider)
    * ConfigType.Broker ————>   new BrokerConfigHandler(config, quotaManagers)
    代码实现如下:
    ```scala
            dynamicConfigHandlers = Map[String, ConfigHandler](ConfigType.Topic -> new TopicConfigHandler(logManager, config, quotaManagers),
                                                           ConfigType.Client -> new ClientIdConfigHandler(quotaManagers),
                                                           ConfigType.User -> new UserConfigHandler(quotaManagers, credentialProvider),
                                                           ConfigType.Broker -> new BrokerConfigHandler(config, quotaManagers))
    ```
* **dynamicConfigManager** 初始化 & 启动,依赖上面初始化的组件
    * **zkClient**
    * **dynamicConfigHandlers**
* **socketServer** 启动处理器: socketServer.startProcessors()
* **`brokerState.currentState`** 标识为 RunningAsBroker
* **shutdownLatch** 初始化: new CountDownLatch(1)
* **startupComplete** 标识为 true
* **isStartingUp** 标识为 false
