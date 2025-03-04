## 方案概览

互联网应用加速解决方案面向各行各业的互联网应用，提供一站式加速网络访问、提高网络稳定性的服务。例如，您在法兰克福有对应的互联网业务，用户可以通过访问域名www.example.com来获取网站服务。中国内地的用户访问该网站的速度需要提升，则可使用本方案实现加速并安全地访问域名。

### 方案架构

您的域名，例如www.example.com通过阿里云解析DNS服务解析到全球加速实例的CNAME，然后经过全球加速的监听功能流转到WAF，最后再通过WAF配置的监听和转发功能流转至NLB，并实现后端服务的负载均衡。

方案提供的默认设置完成部署后在阿里云上搭建的网站运行环境如架构图所示。实际部署时您可以根据资源规划修改部分设置，但最终形成的运行环境与架构图相似。![image.png](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/3229064961/p719599.png)

本方案的技术架构包括以下基础设施和云服务：

* 1个专有网络VPC：为网络型负载均衡NLB、云服务器ECS、云数据库PolarDB MySQL版等云资源形成云上私有网络。
* 2台交换机：将2台云服务器ECS连接在同一网络上，实现服务器之间的通信，并提供基本的网络分段和隔离功能。
* 1个公网网络型负载均衡NLB：对外提供访问并将用户请求分配到不同云服务器ECS上的博客网站服务。
* 4台云服务器ECS：用于部署博客网站服务。
* 1个Web应用防火墙 3.0（按量付费）：为网站服务提供安全服务。
* 1个全球加速标准型实例：为中国内地用户加速访问欧洲网站。
* 云解析DNS：将用户的请求分别解析到全球加速GA。

## 方案部署

### 部署准备

开始部署前，请按以下指引完成账号申请、账号充值、RAM用户创建和授权。

#### 准备账号

1. 如果您还没有阿里云账号，请访问[阿里云账号注册页面](https://account.aliyun.com/register/qr_register.htm)，根据页面提示完成注册。阿里云账号是您使用云资源的付费实体，因此是部署方案的必要前提。
2. [为阿里云账号充值](https://help.aliyun.com/document_detail/324650.html)。

   * 本方案的云资源支持按量付费，且默认设置均采用按量付费引导操作。如果确定任何一个云资源采用按量付费方式部署，账户余额都必须大于等于100元。
   * 完成本方案的部署及体验，预计产生费用不超过2元。

     假设您选择可用的最低规格资源，且资源运行时间不超过1小时。实际情况中可能部分实例无法购买需要根据实际情况调整资源规格，同时因您操作过程中实际使用的流量差异，会导致费用有所变化，请以控制台显示的实际报价以及最终账单为准。

     下表仅供参考，具体费用以实际情况为准。


     | **序号**          | **产品**          | **费用来源** | **规格**          | **地域**             | **预估费用参考** | **相关文档**                                                                                                     |
     | ----------------- | ----------------- | ------------ | ----------------- | -------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------- |
     | 1                 | 云服务器ECS       | 实例费       | ecs.t5-lc2m1.nano | 德国（法兰克福）     | 0.05元/时        | [按量付费](https://help.aliyun.com/zh/ecs/pay-as-you-go-1)                                                       |
     | 系统盘费          | 0.02元/时         |              |                   |                      |                  |                                                                                                                  |
     | 2                 | 全球加速GA        | 实例费       | -                 | 中国香港（加速地域） | 0.137元/小时     | [按量付费全球加速实例计费](https://help.aliyun.com/zh/ga/product-overview/billing-of-pay-as-you-go-ga-instances) |
     | CU费              | 0.386元/个        |              |                   |                      |                  |                                                                                                                  |
     | 3                 | 网络型负载均衡NLB | 实例费       | -                 | 德国（法兰克福）     | 0.147元/小时     | [NLB计费规则](https://help.aliyun.com/zh/slb/network-load-balancer/product-overview/nlb-billable-items)          |
     | 性能容量单位LCU费 | 0.037元/个/小时   |              |                   |                      |                  |                                                                                                                  |
     | 公网网络费        | 0.036元/个        |              |                   |                      |                  |                                                                                                                  |
     | 4                 | Web应用防火墙     | SeCU         | -                 | 非中国内地区域       | 0.05元/个        | [计费说明](https://help.aliyun.com/zh/waf/web-application-firewall-3-0/billing-description-v3)                   |
3. 阿里云账号拥有操作所有资源的最高权限，为了安全起见，建议您使用RAM用户。RAM用户需要获得相关权限才能完成方案部署，自定义权限策略请参考以下示例：


   | **云服务**        | **需要的权限**                     | **描述**                    |
   | ----------------- | ---------------------------------- | --------------------------- |
   | 云服务器ECS       | AliyunECSFullAccess                | 管理云服务器ECS的权限       |
   | 全球加速GA        | AliyunGlobalAccelerationFullAccess | 管理全球加速GA的权限        |
   | 专有网络VPC       | AliyunVPCFullAccess                | 管理专有网络VPC的权限       |
   | 网络型负载均衡NLB | AliyunNLBFullAccess                | 管理网络型负载均衡NLB的权限 |
   | Web应用防火墙     | AliyunYundunWAFv3FullAccess        | 管理WAF的权限               |
   | 云解析DNS         | AliyunDNSFullAccess                | 管理云解析DNS的权限         |
   | 资源编排ROS       | AliyunROSFullAccess                | 管理资源编排ROS的权限       |

   RAM用户的使用请参见[创建RAM用户](https://help.aliyun.com/zh/ram/user-guide/create-a-ram-user)、[创建自定义权限策略](https://help.aliyun.com/zh/ram/user-guide/create-a-custom-policy)、[为RAM用户授权](https://help.aliyun.com/zh/ram/user-guide/grant-permissions-to-the-ram-user)。

### 规划网络和资源

#### 网络规划

请参考表格中的说明和方案默认示例值为每个规划项做详细规划并在实际部署时将默认示例值修改为您的实际规划。


| **规划项**        | **数量** | **说明**                                                                                                                                                    |
| ----------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 地域              | 1        | 您的云服务部署的地域。选择地域的基本原则请参见[地域和可用区](https://help.aliyun.com/document_detail/40654.html)。                                          |
| 专有网络VPC       | 1        | 在部署过程中新建一个VPC作为本方案的专有网络。                                                                                                               |
| 交换机            | 2        | 本方案需要至少2台交换机，用来连接不同的云资源实例。                                                                                                         |
| 网络型负载均衡NLB | 1        | 本方案需要1台网络型负载均衡NLB实例，用于对网站的多台云服务器进行流量分发。NLB可以通过流量分发扩展应用系统的服务能力，通过消除单点故障提升应用系统的可用性。 |
| 全球加速          | 1        | 创建1台按流量计费型的标准实例。                                                                                                                             |
| 安全组            | 1        | 用于限制专有网络VPC下云服务器ECS的网络流入和流出规则。                                                                                                      |

### 规划云资源

请参考表格中的说明和方案默认示例值为每个规划项做详细规划并在实际部署时将默认示例值修改为您的实际规划。


| **规划项**    | **数量** | **说明**                                                      |
| ------------- | -------- | ------------------------------------------------------------- |
| ECS           | 4        | 本方案需要4台ECS实例，用于同时部署网站服务。                  |
| Web应用防火墙 | 1        | 本方案需要1套Web防火墙，用于对NLB提供服务的网站进行安全保护。 |

### 一键部署

规划好资源后，请按照以下步骤部署方案中的所有资源。

### 步骤一：一键部署资源

一键配置基于阿里云资源编排服务ROS（Resource Orchestration Service）实现，旨在帮助开发者通过IaC（Infrastructure as Code）的方式体验资源的自动化配置。本方案中的一键部署脚本可完成除WAF以外的资源部署。

1. 单击[一键部署](https://ros.console.aliyun.com/cn-hangzhou/stacks/create?templateUrl=https://ros-public-templates.oss-cn-hangzhou.aliyuncs.com/service_template/technical-solution/ga-nlb-global-accelerate.yml&pageTitle=%E4%BA%92%E8%81%94%E7%BD%91%E5%BA%94%E7%94%A8%E5%85%A8%E7%90%83%E5%8A%A0%E9%80%9F&disableRollback=false&isSimplified=true&productNavBar=disabled)前往ROS控制台，系统自动打开使用新资源创建资源栈的面板，并跳转至**配置模板参数**页签。

   **说明**

   ROS控制台默认处于您上一次访问控制台时的地域，请根据您创建的资源所在地域修改地域后再执行下一步。本方案设置为**德国（法兰克福）**。
2. **资源栈名称**可保持默认值。
3. 在**可用区配置**区域，分别选择**可用区A**和**可用区B**。
4. 在**ECS实例配置**区域，选择**共享型实例**，推荐使用最低规格，并设置**实例密码**。
5. 在**加速域名配置**区域，设置**加速地域ID**为**中国香港**，设置**加速域名**为您所需加速的域名。
6. 单击**创建**。

### 步骤二：添加网站配置

1. 访问[Web应用防火墙3.0（按量付费）购买页](https://common-buy.aliyun.com/?commodityCode=waf_v2_public_cn)，在**非中国内地****区域**开通服务。
2. 登录[Web应用防火墙3.0控制台](https://yundunnext.console.aliyun.com/?p=wafnew#//cn-hangzhou)。在顶部菜单栏，选择WAF实例的资源组和地域（**非中国内地**）。
3. 在左侧导航栏，单击**接入管理**。
4. 在**CNAME接入**页签，单击**接入**。
5. 在**配置监听**向导页，完成如下配置后，单击**下一步**。

   | **配置项**                  | **配置说明**                                                                                                                                                                                                                      |
   |--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   | **域名**                   | 填写要防护的域名，本文示例**www.example.com**。                                                                                                                                                                                             |
   | **协议类型**                 | 本文选择**HTTP**并配置端口为**80**。                                                                                                                                                                                                     |
   | **WAF前是否有七层代理（高防/CDN等）** | 本方案选择**是**。                                                                                                                                                                                                                   |
   | **资源组**                  | 从资源组下拉列表中选择该域名所属资源组。如果不选择，则默认加入**默认资源组**。 **说明**  您可以使用资源管理服务创建资源组，根据业务部门、项目等维度对云资源进行分组管理。更多信息，请参见[创建资源组](https://help.aliyun.com/zh/resource-management/resource-group/user-guide/create-a-resource-group#task-xpl-kjm-4fb)。 |

6. 在**配置转发**向导页，完成如下配置后，单击**提交**。

   | **配置项**    | **说明**                                                                                      |
   |------------|---------------------------------------------------------------------------------------------|
   | **负载均衡算法** | 本示例选择**轮询**。                                                                                |
   | **服务器地址**  | 本示例选择域名，并填写NLB的CNAME。**说明**  服务器地址为域名时，只支持IPv4地址，暂不支持IPv6地址，即WAF只会将客户端请求转发到源站域名解析出来的IPv4地址。 |

7. 在电脑中打开命令行窗口。
8. 执行以下命令，查看未加速前数据包延迟情况。

   **说明**

   请将`<域名>`替换为您的域名。

   ```
   curl -o /dev/null -s -w "time_connect: %{time_connect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" "http://<GA域名/WAF域名>:80" -H "host:<域名>"
   ```


### 步骤三：配置全球加速监听WAF CNAME

1. 登录[全球加速管理控制台](https://ga.console.aliyun.com/list)。
2. 在**实例列表**页面，找到目标全球加速实例，在**操作**列单击**配置监听**。
3. 在**监听**页签下，单击**添加监听**，完成以下配置后，单击**提交**。
4. 在**配置监听和协议**配置向导页面，根据以下信息配置监听，然后单击**下一步**。

   | **配置**       | **说明**        |
   |--------------|---------------|
   | ****监听名称**** | 输入监听的名称。      |
   | **协议**       | 本文选择**HTTP**。 |
   | **端口**       | 本文输入**80**。   |
   | **客户端亲和性**   | **关闭**。       |

   b. 配置终端节点。

   | **配置**          | **说明**                          |
   |-----------------|---------------------------------|
   | **节点组名称**       | 输入终端节点组的名称。                     |
   | **地域**          | 选择终端节点组所属的阿里云地域，本文选择**中国（香港）**。 |
   | ****后端服务部署在**** | **非阿里云**。                       |
   | **后端服务类型**      | 选择**自定义域名**。                    |
   | **后端服务**        | 填入WAF的CNAME。                    |
   | **后端服务协议**      | 勾选**HTTP**。                     |

### 步骤四：将DNS解析到全球加速

您必须更新规则对应域名的DNS解析全球加速CNAME记录，将中国内地用户的业务流量切换至全球加速，使规则生效。

1. 登录[阿里云云解析DNS控制台](https://dns.console.aliyun.com/)。
2. 在**域名解析**页面，找到目标域名，单击**操作**列下的**解析设置**。
3. 在**解析设置**页面，单击**添加记录**。
4. 在**添加记录**对话框，配置记录，然后单击**确定**。

   1. **记录类型**：选择记录类型。

      本方案需要将Web域名指向另一个域名，所以选择**CNAME**。
   2. **主机记录**：输入加速域名的前缀。

      本方案输入**www**。
   3. **解析请求来源**：选择**默认**。
   4. **记录值**：设置为全球加速的CNAME地址。
   5. **TTL**：域名解析的生效时间。

      本方案选择**10分钟**。

### 完成及清理

#### 方案验证

**通过对比加速前后连接时间，验证全球加速效果**

1. 在中国内地的电脑中打开命令行窗口。
2. 执行以下命令，查看加速后数据包延迟情况。

   ```
   curl -o /dev/null -s -w "time_connect: %{time_connect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" "http://<GA域名/WAF域名>:80" -H "host:<DNS域名>"
   ```

   **说明**

   * `time_connect`：连接时间，从开始到建立TCP连接完成所用的时间。
   * `time_starttransfer`：开始传输时间。在客户端发出请求后，到后端服务器响应第一个字节所用的时间。
   * `time_total`：连接总时间。客户端发出请求后，到后端服务器响应会话所用的时间。
3. 查看返回的参数，进行对比如下：![image.png](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/8601354961/p719140.png)

   **说明**

   对比加速前后的time\_connect和time\_starttransfer、time\_total三项数值变化可明显看到加速后访问速度得到了明显的提升。

#### 清理资源

1. 登录[资源编排管理控制台](https://ros.console.aliyun.com/)，左侧导航栏菜单选择**资源栈**。
2. 在页面的顶部选择部署的资源栈所在地域，找到部署的资源栈。单击其右侧**操作**列的**删除**并确认，可一键删除一键配置所创建的资源。
3. 释放Web应用防火墙。

   登录[Web应用防火墙控制台](https://yundunnext.console.aliyun.com/?spm=a2c63.p38356.0.0.65c3185ezPgKGn&p=wafnew)，在**顶部菜单栏**，选择WAF实例的资源组和地域，在**总览**页面右上角的实例基本信息卡片区域，单击**关闭WAF**。
