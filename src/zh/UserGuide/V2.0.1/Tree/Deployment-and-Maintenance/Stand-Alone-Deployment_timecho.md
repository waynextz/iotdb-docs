<!--

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

-->
# 单机版部署

本章将介绍如何启动IoTDB单机实例，IoTDB单机实例包括 1 个ConfigNode 和1个DataNode（即通常所说的1C1D）。

## 注意事项

1. 安装前请确认系统已参照[系统配置](./Environment-Requirements.md)准备完成。

2. 部署时推荐优先使用`hostname`进行IP配置，可避免后期修改主机ip导致数据库无法启动的问题。设置hostname需要在目标服务器上配置/etc/hosts，如本机ip是192.168.1.3，hostname是iotdb-1，则可以使用以下命令设置服务器的 hostname，并使用hostname配置IoTDB的`cn_internal_address`、dn_internal_address、dn_rpc_address。

   ```shell
   echo "192.168.1.3  iotdb-1" >> /etc/hosts 
   ```

3. 部分参数首次启动后不能修改，请参考下方的【参数配置】章节进行设置

4. 无论是在linux还是windows中，请确保IoTDB的安装路径中不含空格和中文，避免软件运行异常。

5. 请注意，安装部署（包括激活和使用软件）IoTDB时需要保持使用同一个用户进行操作，您可以：
- 使用 root 用户（推荐）：使用 root 用户可以避免权限等问题。
- 使用固定的非 root 用户：
  - 使用同一用户操作：确保在启动、激活、停止等操作均保持使用同一用户，不要切换用户。
  - 避免使用 sudo：尽量避免使用 sudo 命令，因为它会以 root 用户权限执行命令，可能会引起权限混淆或安全问题。 

6. 推荐部署监控面板，可以对重要运行指标进行监控，随时掌握数据库运行状态，监控面板可以联系商务获取，部署监控面板步骤可以参考：[监控面板部署](./Monitoring-panel-deployment.md)。

## 安装步骤

### 解压安装包并进入安装目录

```shell
unzip  iotdb-enterprise-{version}-bin.zip
cd  iotdb-enterprise-{version}-bin
```

### 参数配置

#### 环境脚本配置

- ./conf/confignode-env.sh（./conf/confignode-env.bat）配置

| **配置项**  |                **说明**                | **默认值** |                    **推荐值**                    |     备注     |
| :---------: | :------------------------------------: | :--------: | :----------------------------------------------: | :----------: |
| MEMORY_SIZE | IoTDB ConfigNode节点可以使用的内存总量 |     空     | 可按需填写，填写后系统会根据填写的数值来分配内存 | 重启服务生效 |

- ./conf/datanode-env.sh（./conf/datanode-env.bat）配置

| **配置项**  |               **说明**               | **默认值** |                    **推荐值**                    |     备注     |
| :---------: | :----------------------------------: | :--------: | :----------------------------------------------: | :----------: |
| MEMORY_SIZE | IoTDB DataNode节点可以使用的内存总量 |     空     | 可按需填写，填写后系统会根据填写的数值来分配内存 | 重启服务生效 |

#### 系统通用配置

打开通用配置文件（./conf/iotdb-system.properties 文件），设置以下参数：

|        **配置项**         |             **说明**             |   **默认值**   |                    **推荐值**                    |           备注            |
| :-----------------------: | :------------------------------: | :------------: | :----------------------------------------------: | :-----------------------: |
|       cluster_name        |             集群名称             | defaultCluster | 可根据需要设置集群名称，如无特殊需要保持默认即可 |    首次启动后不可修改     |
| schema_replication_factor | 元数据副本数，单机版此处设置为 1 |       1        |                        1                         | 默认1，首次启动后不可修改 |
|  data_replication_factor  |  数据副本数，单机版此处设置为 1  |       1        |                        1                         | 默认1，首次启动后不可修改 |

#### ConfigNode配置

打开ConfigNode配置文件（./conf/iotdb-system.properties文件），设置以下参数：

|     **配置项**      |                           **说明**                           |    **默认**     |                      推荐值                      |      **备注**      |
| :-----------------: | :----------------------------------------------------------: | :-------------: | :----------------------------------------------: | :----------------: |
| cn_internal_address |              ConfigNode在集群内部通讯使用的地址              |    127.0.0.1    | 所在服务器的IPV4地址或hostname，推荐使用hostname | 首次启动后不能修改 |
|  cn_internal_port   |              ConfigNode在集群内部通讯使用的端口              |      10710      |                      10710                       | 首次启动后不能修改 |
|  cn_consensus_port  |            ConfigNode副本组共识协议通信使用的端口            |      10720      |                      10720                       | 首次启动后不能修改 |
| cn_seed_config_node | 节点注册加入集群时连接的ConfigNode 的地址，cn_internal_address:cn_internal_port | 127.0.0.1:10710 |       cn_internal_address:cn_internal_port       | 首次启动后不能修改 |

#### DataNode 配置

打开DataNode配置文件 ./conf/iotdb-system.properties，设置以下参数：

| **配置项**                      | **说明**                                                     | **默认**        | 推荐值                                           | **备注**           |
| :------------------------------ | :----------------------------------------------------------- | :-------------- | :----------------------------------------------- | :----------------- |
| dn_rpc_address                  | 客户端 RPC 服务的地址                                        | 0.0.0.0         | 所在服务器的IPV4地址或hostname，推荐使用hostname | 重启服务生效       |
| dn_rpc_port                     | 客户端 RPC 服务的端口                                        | 6667            | 6667                                             | 重启服务生效       |
| dn_internal_address             | DataNode在集群内部通讯使用的地址                             | 127.0.0.1       | 所在服务器的IPV4地址或hostname，推荐使用hostname | 首次启动后不能修改 |
| dn_internal_port                | DataNode在集群内部通信使用的端口                             | 10730           | 10730                                            | 首次启动后不能修改 |
| dn_mpp_data_exchange_port       | DataNode用于接收数据流使用的端口                             | 10740           | 10740                                            | 首次启动后不能修改 |
| dn_data_region_consensus_port   | DataNode用于数据副本共识协议通信使用的端口                   | 10750           | 10750                                            | 首次启动后不能修改 |
| dn_schema_region_consensus_port | DataNode用于元数据副本共识协议通信使用的端口                 | 10760           | 10760                                            | 首次启动后不能修改 |
| dn_seed_config_node             | 节点注册加入集群时连接的ConfigNode地址,即cn_internal_address:cn_internal_port | 127.0.0.1:10710 | cn_internal_address:cn_internal_port             | 首次启动后不能修改 |

> ❗️注意：VSCode Remote等编辑器无自动保存配置功能，请确保修改的文件被持久化保存，否则配置项无法生效

### 启动 ConfigNode 节点

进入iotdb的sbin目录下，启动confignode

```shell
./start-confignode.sh    -d      #“-d”参数将在后台进行启动 
```
如果启动失败，请参考[常见问题](#常见问题)。

### 激活数据库

#### 方式一：激活文件拷贝激活

- 启动confignode节点后，进入activation文件夹, 将 system_info文件复制给天谋工作人员
- 收到工作人员返回的 license文件
- 将license文件放入对应节点的activation文件夹下；

#### 方式二：激活脚本激活

- 获取激活所需机器码，进入安装目录的sbin目录，执行激活脚本:

```shell
 cd sbin
./start-activate.sh
```

- 显示如下信息，请将机器码（即该串字符）复制给天谋工作人员：

```shell
Please copy the system_info's content and send it to Timecho:
01-KU5LDFFN-PNBEHDRH
Please enter license:
```

- 将工作人员返回的激活码输入上一步的命令行提示处 `Please enter license:`，如下提示：

```shell
Please enter license:
Jw+MmF+AtexsfgNGOFgTm83BgXbq0zT1+fOfPvQsLlj6ZsooHFU6HycUSEGC78eT1g67KPvkcLCUIsz2QpbyVmPLr9x1+kVjBubZPYlVpsGYLqLFc8kgpb5vIrPLd3hGLbJ5Ks8fV1WOVrDDVQq89YF2atQa2EaB9EAeTWd0bRMZ+s9ffjc/1Zmh9NSP/T3VCfJcJQyi7YpXWy5nMtcW0gSV+S6fS5r7a96PjbtE0zXNjnEhqgRzdU+mfO8gVuUNaIy9l375cp1GLpeCh6m6pF+APW1CiXLTSijK9Qh3nsL5bAOXNeob5l+HO5fEMgzrW8OJPh26Vl6ljKUpCvpTiw==
License has been stored to sbin/../activation/license
Import completed. Please start cluster and excute 'show cluster' to verify activation status
```

### 启动DataNode 节点

进入iotdb的sbin目录下，启动datanode：

```shell
cd sbin
./start-datanode.sh   -d   #-d参数将在后台进行启动 
```

### 验证部署

可直接执行 ./sbin 目录下的 Cli 启动脚本：

```shell
./start-cli.sh  -h  ip(本机ip或域名)  -p  端口号(6667)
```

成功启动后，出现如下界面显示IOTDB安装成功。

![](https://alioss.timecho.com/docs/img/%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)

出现安装成功界面后，继续看下是否激活成功，使用`show cluster`命令

当看到最右侧显示ACTIVATED表示激活成功

![](https://alioss.timecho.com/docs/img/show%20cluster.png)

> 出现`ACTIVATED(W)`为被动激活，表示此ConfigNode没有license文件（或没有签发时间戳最新的license文件）。此时建议检查license文件是否已放入license文件夹，没有请放入license文件，若已存在license文件，可能是此节点license文件与其他节点信息不一致导致，请联系天谋工作人员重新申请.


## 常见问题

1. 部署过程中多次提示激活失败
    - 使用 `ls -al` 命令：使用 `ls -al` 命令检查安装包根目录的所有者信息是否为当前用户。
    - 检查激活目录：检查 `./activation` 目录下的所有文件，所有者信息是否为当前用户。  

2. Confignode节点启动失败

    步骤 1: 请查看启动日志，检查是否修改了某些首次启动后不可改的参数。

    步骤 2: 请查看启动日志，检查是否出现其他异常。日志中若存在异常现象，请联系天谋技术支持人员咨询解决方案。

    步骤 3: 如果是首次部署或者数据可删除，也可按下述步骤清理环境，重新部署后，再次启动。

    步骤 4: 清理环境：

    a. 结束所有 ConfigNode 和 DataNode 进程。
   
    ```Bash
        # 1. 停止 ConfigNode 和 DataNode 服务
        sbin/stop-standalone.sh

        # 2. 检查是否还有进程残留
        jps
        # 或者
        ps -ef|gerp iotdb

        # 3. 如果有进程残留，则手动kill
        kill -9 <pid>
        # 如果确定机器上仅有1个iotdb，可以使用下面命令清理残留进程
        ps -ef|grep iotdb|grep -v grep|tr -s '  ' ' ' |cut -d ' ' -f2|xargs kill -9
    ```
     b.  删除 data 和 logs 目录。 

    说明：删除 data 目录是必要的，删除 logs 目录是为了纯净日志，非必需。
    ```Bash
        cd /data/iotdb
        rm -rf data logs
    ```