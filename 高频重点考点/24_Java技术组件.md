# Java 技术组件

## 考试中一般怎么考

1. 问 J2EE 架构图中各组件的位置和职责。
2. 问组件之间的交互关系，如 Servlet 如何调用 EJB、JNDI 如何查找资源。
3. 问分层架构：表现层、业务层、集成层各由什么组件承担。
4. 问技术选型：EJB vs JavaBean、RMI vs JMS 的区别和适用场景。

## 针对考法梳理知识点

### J2EE 分层架构

J2EE 采用多层架构，每层由不同组件承担：

```
客户端层（Client Tier）
  浏览器、Applet、Java 应用客户端
      ↓
Web 层（Web Tier）
  JSP、Servlet、JavaBean
      ↓
业务层（Business Tier）
  EJB、JavaBean（业务逻辑）
      ↓
集成层/资源层（Integration Tier）
  JDBC、JMS、JCA、JTA → 数据库、消息系统、外部系统
```

容器为各层提供运行环境和基础服务：

- Web Container：管理 JSP 和 Servlet 的生命周期和请求分发。
- EJB Container：管理 EJB 的生命周期、事务、安全和并发。
- Application Server：包含 Web Container 和 EJB Container，提供完整的 J2EE 运行环境（如 WebLogic、WebSphere、JBoss）。

### JSP

JSP（JavaServer Pages）在 HTML 中嵌入 Java 代码，用于生成动态页面。

职责：表现层，负责页面展示。

特点：

- 运行时被编译为 Servlet 执行。
- 适合页面渲染，不适合写复杂业务逻辑。
- 案例中如果出现"JSP 中混大量 Java 代码"，属于设计问题，应改为 MVC 分层。

### Servlet

Servlet 是运行在 Web Container 中的 Java 类，负责接收 HTTP 请求并返回响应。

职责：Web 层控制器，处理请求、调用业务逻辑、转发或重定向到页面。

核心方法：

- `init()`：初始化。
- `service()`：处理请求（doGet/doPost）。
- `destroy()`：销毁。

典型流程：

```
客户端请求 → Web Container → Servlet 处理 → 调用业务层 → 返回响应/JSP
```

Servlet 是 Java Web 的基础，Spring MVC 的 DispatcherServlet 本质上就是一个 Servlet。

### JavaBean

JavaBean 是符合特定规范的 Java 类：私有属性、公有 getter/setter、可序列化、无参构造。

在 J2EE 中的两种角色：

| 角色 | 位置 | 作用 |
| --- | --- | --- |
| 实体 Bean | 各层 | 封装数据，在各层之间传递数据（DTO/VO） |
| 会话 Bean | 业务层 | 封装业务逻辑，早期版本中承担部分 EJB 的职责 |

案例中如果问"JavaBean 和 EJB 的区别"：JavaBean 是普通 Java 类，EJB 是运行在 EJB Container 中的、由容器管理生命周期和企业级服务的组件。

### EJB

EJB（Enterprise JavaBean）是 J2EE 业务层的核心组件，运行在 EJB Container 中，容器自动提供事务管理、安全控制、并发控制、生命周期管理等企业级服务。

EJB 三种类型：

| 类型 | 作用 | 生命周期 |
| --- | --- | --- |
| Session Bean | 封装业务逻辑，最常用 | 无状态（Stateless）：每次调用不保留状态；有状态（Stateful）：为每个客户端保留会话状态 |
| Message-Driven Bean | 异步处理 JMS 消息 | 由消息驱动，类似消息消费者 |
| Entity Bean | 映射数据库表（已被 JPA 取代） | 已过时 |

答题要点：

- EJB 由容器管理，开发者只关注业务逻辑。
- 容器自动提供事务（JTA）、安全、远程调用（RMI）、并发控制等服务。
- 无状态 Session Bean 可被池化复用，性能更好。

### JDBC

JDBC（Java Database Connectivity）是 Java 访问数据库的标准 API。

职责：集成层，负责与数据库交互。

核心组件：

- DriverManager：管理数据库驱动。
- Connection：数据库连接。
- Statement / PreparedStatement：执行 SQL。
- ResultSet：结果集。

JDBC 是最底层的数据库访问方式，直接写 SQL，手动管理连接。实际项目中通常用连接池（如 Druid、HikariCP）管理连接，用 ORM 框架简化操作。

### DAO

DAO（Data Access Object）是数据访问层的设计模式，将数据访问逻辑与业务逻辑分离。

职责：封装对数据库的所有操作（增删改查），业务层通过 DAO 接口访问数据，不直接操作 JDBC。

设计要点：

- 业务层依赖 DAO 接口，不依赖具体实现。
- DAO 实现类使用 JDBC 或 ORM 框架访问数据库。
- 换数据库或换访问方式时，只需替换 DAO 实现，不影响业务层。

案例中如果问"为什么要用 DAO"：实现数据访问和业务逻辑的解耦，便于替换数据源和单元测试。

### JNDI

JNDI（Java Naming and Directory Interface）是 Java 的命名和目录服务 API，用于查找和访问各种资源。

职责：提供统一的资源查找机制。

常见用途：

- 查找数据源（DataSource）。
- 查找 EJB 组件。
- 查找 JMS 连接工厂和目的地。
- 查找环境变量和配置信息。

工作流程：

```
应用代码 → JNDI 查找名称 → 命名服务返回资源对象
```

例如：数据源在 Application Server 中配置，应用通过 JNDI 名称获取连接池，不需要在代码中硬编码数据库地址。

### RMI

RMI（Remote Method Invocation）是 Java 的远程方法调用机制，允许一个 Java 虚拟机上的对象调用另一个虚拟机上的对象方法。

职责：实现 Java 系统间的远程通信。

核心概念：

- 远程接口：定义可被远程调用的方法。
- 桩（Stub）：客户端代理，负责打包参数和网络通信。
- 骨架（Skeleton）：服务端代理，负责解包参数和调用实际对象。

RMI 是 Java 原生的 RPC 机制，EJB 的远程调用底层就基于 RMI。

案例中如果问"EJB 如何实现远程调用"：通过 RMI 协议，客户端通过 JNDI 查找 EJB 的远程引用（Stub），调用方法时由 Stub 将请求序列化后发送到服务端。

### JMS

JMS（Java Message Service）是 Java 的消息服务标准 API，用于在应用之间异步传递消息。

职责：集成层，实现异步通信和系统解耦。

两种消息模型：

| 模型 | 特点 | 适用场景 |
| --- | --- | --- |
| 点对点（Queue） | 一个消息只被一个消费者消费 | 任务分发、订单处理 |
| 发布订阅（Topic） | 一个消息被所有订阅者消费 | 事件广播、通知 |

JMS 与 EJB 的关系：Message-Driven Bean（MDB）是 JMS 消息的消费者，容器自动接收消息并调用 MDB 处理。

案例中如果问"如何实现异步处理"：通过 JMS 发送消息到队列，由 Message-Driven Bean 异步消费处理。

### JTA

JTA（Java Transaction API）是 Java 的分布式事务管理 API。

职责：管理跨多个资源（数据库、消息队列等）的事务。

核心概念：

- UserTransaction：应用通过它显式控制事务边界（begin、commit、rollback）。
- TransactionManager：容器内部使用，管理事务上下文。
- XA 资源：支持两阶段提交的资源（如 XA 数据源、XA JMS 连接）。

JTA 典型场景：

```
开始事务 → 操作数据库 A → 操作数据库 B → 发送 JMS 消息 → 提交/回滚
```

所有操作要么全部成功，要么全部回滚。

EJB Container 默认通过 JTA 管理事务，开发者可通过注解声明事务行为（如 `@TransactionAttribute(REQUIRED)`），无需手动编写事务代码。

### JCA

JCA（Java Connector Architecture）是 Java 连接器架构，用于将 J2EE 应用与企业信息系统（EIS）集成。

职责：集成层，连接外部系统（如 ERP、CRM、大型机、遗留系统）。

核心组件：

- 资源适配器（Resource Adapter）：打包为 .rar 文件，部署到 Application Server 中。
- 连接管理：管理与 EIS 的连接池。
- 事务管理：支持本地事务和 XA 事务。
- 安全映射：将 J2EE 安全身份映射到 EIS 安全身份。

JCA 解决的问题：不同外部系统的接口各异，JCA 提供统一的集成标准，Application Server 通过资源适配器与各种 EIS 对接。

### Web Container

Web Container 是 JSP 和 Servlet 的运行环境。

职责：

- 管理 JSP 和 Servlet 的生命周期。
- 接收 HTTP 请求，分发到对应的 Servlet 或 JSP。
- 管理 Session（会话管理）。
- 管理线程池处理并发请求。
- 提供安全控制（认证、授权）。

常见实现：Tomcat、Jetty。

### EJB Container

EJB Container 是 EJB 的运行环境。

职责：

- 管理 EJB 的生命周期（创建、池化、销毁）。
- 提供声明式事务管理（JTA）。
- 提供安全控制和权限检查。
- 管理并发访问和实例池化。
- 支持远程调用（RMI）。
- 管理消息驱动 Bean 的消息接收。

EJB Container 通常内嵌在 Application Server 中。

### Application Server

Application Server 提供完整的 J2EE 运行环境，包含 Web Container 和 EJB Container，以及各种企业级服务。

核心服务：

- Web Container + EJB Container。
- JTA 事务管理。
- JNDI 命名服务。
- JMS 消息服务。
- JDBC 数据源管理。
- JCA 连接器支持。
- 安全服务。
- 管理和监控。

常见实现：

| 产品 | 特点 |
| --- | --- |
| WebLogic | Oracle 出品，企业级 |
| WebSphere | IBM 出品，大型企业 |
| JBoss/WildFly | 开源，Red Hat |
| Tomcat | 只有 Web Container，不是完整的 Application Server |

注意：Tomcat 只是 Web Container，不包含 EJB Container。如果题目需要 EJB 支持，应选 WebLogic、WebSphere 或 JBoss。

### 组件交互全景图

```
浏览器
  ↓ HTTP
Web Container（Tomcat/WebLogic）
  ├── JSP（页面渲染）
  └── Servlet（请求控制）
        ↓ 调用
      EJB Container（Application Server 内置）
        ├── Session Bean（业务逻辑）
        │     ↓ 通过 JNDI 查找
        │   JTA（分布式事务）
        │     ↓
        │   JDBC / JCA（访问数据库 / 外部系统）
        └── Message-Driven Bean（异步消费 JMS 消息）
              ↑
            JMS（消息队列）
```

关键调用链：

1. 浏览器 → Servlet：HTTP 请求。
2. Servlet → Session Bean：业务调用（通过 JNDI 查找 EJB）。
3. Session Bean → JDBC：数据库操作。
4. Session Bean → JMS：发送异步消息。
5. JMS → Message-Driven Bean：异步消费消息。
6. JTA：跨 JDBC + JMS 的分布式事务。
7. JCA：连接外部遗留系统。

### J2EE 与现代框架的对应关系

案例分析中可能会问"传统 J2EE 和 Spring 的关系"：

| J2EE 组件 | 现代对应 | 说明 |
| --- | --- | --- |
| Servlet | Spring MVC DispatcherServlet | Spring MVC 本质上是 Servlet |
| JSP | Thymeleaf / 前端框架 | 模板引擎或前后端分离 |
| EJB Session Bean | Spring Bean / Spring Service | Spring 通过 IoC 容器管理 Bean，取代 EJB 的重量级容器 |
| JNDI | Spring IoC / Nacos | 依赖注入取代 JNDI 查找 |
| JDBC | MyBatis / JPA / Hibernate | ORM 框架简化数据库操作 |
| JMS | Spring AMQP / Spring Kafka | Spring 封装了消息队列操作 |
| JTA | Spring 事务管理 / Seata | Spring 声明式事务取代 JTA，分布式事务用 Seata |
| JCA | Spring Integration | Spring 集成框架 |
| Application Server | Spring Boot + 内嵌 Tomcat | 无需重量级应用服务器 |

案例中如果问"为什么从 J2EE 迁移到 Spring"：Spring 通过轻量级 IoC 容器和 AOP 机制，用更简单的方式实现了 EJB 的核心能力（事务、安全、远程调用），降低了对重量级 Application Server 的依赖，开发和部署更灵活。
