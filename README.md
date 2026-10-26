# 售货机员工端后端 · dkd-app

面向售货机运营与维修人员的 Spring Boot 后端学习项目，提供员工验证码登录、工单查询、接单、取消、完成及任务明细等接口。

这是 **Java 后端服务**，不是 Android/iOS App 前端源码。

## 功能

| 模块 | 当前代码内容 |
| --- | --- |
| 员工登录 | 验证码读取与校验、登录令牌、员工和角色信息 |
| 工单查询 | 分页、设备编号、用户、工单号、状态、类型和日期筛选 |
| 工单操作 | 接受、取消和完成工单，按当前状态与登录人校验 |
| 任务统计 | 员工任务排行相关接口 |
| 明细 | 按工单 ID 查询明细 |
| 设备联动 | 根据工单类型调整设备状态，补货时更新货道库存 |

工单状态在当前实现中包含：待办、进行、取消、完成。

## 技术栈与目录

- Java 11、Spring Boot 2.7.3。
- MyBatis / MyBatis-Plus、MySQL、Druid、PageHelper。
- Redis、JWT、Hutool、Lombok。
- Knife4j 与阿里云短信相关依赖。

```text
src/main/java/com/dkd/app/
├─ Application.java
├─ controller/
├─ service/impl/
├─ mapper/
├─ domain/
└─ interceptor/
src/main/resources/
├─ application.yml
└─ application-dev.yml
```

## 准备环境

1. 安装 **JDK 11** 和 Maven。
2. 准备 MySQL、Redis，以及与实体和 Mapper 匹配的员工、角色、工单、设备和货道等表。
3. 本仓库没有 SQL 初始化脚本，需要另行准备对应版本的数据库结构与数据。
4. 检查 [application.yml](src/main/resources/application.yml) 和 [application-dev.yml](src/main/resources/application-dev.yml)：
   - `dkd.datasource`：数据库连接。
   - `dkd.redis`：Redis 连接与认证。
   - `dkd.sms`：预留短信集成参数。

当前默认启用 `dev`。真实密码、短信密钥等应通过本地或外部配置管理，不要提交到 GitHub。

## 启动

```shell
git clone https://github.com/Trappedqueens/dkd-app.git
cd dkd-app
mvn spring-boot:run
```

该仓库为私有仓库，需要有权限的 GitHub 账号。

默认端口：**9007**。也可构建后运行：

```shell
mvn clean package -DskipTests
java -jar target/dkd-app.jar
```

IDE 启动类为 `com.dkd.app.Application`。

## 接口概览

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/user-service/user/code/{mobile}` | 验证码相关入口 |
| POST | `/user-service/user/login` | 员工登录 |
| GET | `/user-service/user/{id}` | 员工基本信息 |
| GET | `/task-service/task/search` | 工单查询 |
| GET | `/task-service/task/rank/{userId}` | 任务统计 |
| GET | `/task-service/task/accept/{taskId}` | 接受工单 |
| POST | `/task-service/task/cancel/{taskId}` | 取消工单 |
| GET | `/task-service/task/complete/{taskId}` | 完成工单 |
| GET | `/task-service/taskDetails/{taskId}` | 工单明细 |

完整参数见 [controller](src/main/java/com/dkd/app/controller)。当前拦截器从 `Authorization` 请求头读取令牌，令牌格式应按登录返回值及拦截器实现对接。

注意：当前部分改变业务状态的接口沿用 GET 方法，调试时不要把“接受/完成工单”当作只读查询。

## 验证码实现与使用边界

- 当前 `EmpServiceImpl.sendSms` 中，随机验证码生成和真实短信发送调用被注释，使用的是演示验证码逻辑；不能把它当作已接通真实短信服务。
- 登录还需要数据库中存在匹配的员工信息，以及 Redis 中的验证码数据。
- 接单、取消、完成涉及持久化修改与库存/设备联动，应在独立测试数据库中验证。
- 当前仓库没有员工端前端、完整数据库和可直接复用的生产环境配置。

本次仅补充 README，未修改业务代码，未发送短信，未执行构建或业务接口测试。
