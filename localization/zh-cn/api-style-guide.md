# API 设计指南

# 介绍
PayPal 平台为开发者提供若干服务，这些服务将 PayPal 的业务能力进行了可重用的封装。开发者可通过 API 访问这些服务。得益于不同服务间 API 统一的设计模式和原则，开发者能够快速地利用这些 API 构建出复杂的业务过程。

PayPal APIs 尽可能地遵循了 [RESTful](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)  风格。为了这一目标，我们在 RESTful API 的设计中制定了若干标准、规范和约定。这些标准帮助我们设计并维护了数以百计的 API ，并使其在若干年的时间内都能应对大量不同的使用场景。

在此我们将这些设计规范共享出来，以推广更好的 API 设计实践。我们从社区中汲取了大量的内容，鉴于此，我们相信回报社区也是非常重要的。该文档力求通用，使得大家可以方便地将规范整合到自己的项目中。如果有任何的更新，建议或者其他内容期望贡献，欢迎提 PR 或者开 issue.


### 文档语义、格式与命名

本文档中的关键字 "必须（MUST）"、"不得（MUST NOT）"、"需要（REQUIRED）"、"应当（SHALL）"、"不应（SHALL NOT）"、"应该（SHOULD）"、"不该（SHOULD NOT）", "建议的（RECOMMENDED）", "可以（MAY）", and "可选的（OPTIANAL）" 的含义在标准 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 中被解释和描述。

"REST" 和 "RESTful" 不可记做其他写法，以大写字母表示缩写。 "JSON"、"XML" 和其他缩写词也是如此。

机器可读的文本，如 URL，HTTP 动词和源代码等，使用等宽字体表示。

URI 中包含变量的部分遵循 [URI Template RFC 6570](https://tools.ietf.org/html/rfc6570) 中的规范。例如，一条包含变量 `account_id` 的 URI 应当记做 https://foo.com/accounts/{account_id}/。

### 贡献者

[Sanjay Dalal](https://www.linkedin.com/in/sanjaydalal) (former member: PayPal API Platform), [Jason Harmon](https://es.linkedin.com/in/jasonhnaustin) (former member: PayPal API Platform), [Erik Hogan](https://www.linkedin.com/in/erik-hogan-81431) (PayPal API Platform), [Jayadeba Jena](https://www.linkedin.com/in/jayadeba-jena-1a6a0020) (PayPal API Platform), [Nikhil Kolekar](https://www.linkedin.com/in/nikhil-kolekar-28627a2/) (PayPal API Platform), [Gagan Maheshwari](https://www.linkedin.com/in/gaganmaheshwari) (former member: PayPal API Platform), [Michael McKenna](http://linkedin.com/in/mgmckenna) (PayPal Globalization), [George Petkov](https://www.linkedin.com/in/gbpetkov) (former member: PayPal API Platform) and Andrew Todd (PayPal Credit).

# 目录
<!-- TOC depthTo:3 -->

- [API 设计指南](#api-设计指南)
- [介绍](#介绍)
        - [文档语义、格式与命名](#文档语义格式与命名)
        - [贡献者](#贡献者)
- [目录](#目录)
- [规范释义](#规范释义)
    - [所用词汇](#所用词汇)
- [服务设计原则](#服务设计原则)
    - [松耦合（Loose Coupling）](#松耦合loose-coupling)
    - [封装（Encapsulation）](#封装encapsulation)
    - [稳定（Stability）](#稳定stability)
    - [可复用（Reusable）](#可复用reusable)
    - [基于接口约定的（Contract-based）](#基于接口约定的contract-based)
    - [一致性（Consistency）](#一致性consistency)
    - [易用性（Ease Of Use）](#易用性ease-of-use)
    - [可外部化的（Externalizable）](#可外部化的externalizable)
- [HTTP 方法, Header 与状态](#http-方法-header-与状态)
    - [数据资源与 HTTP 方法](#数据资源与-http-方法)
        - [业务能力与资源建模](#业务能力与资源建模)
        - [HTTP 方法](#http-方法)
    - [处理](#处理)
        - [数据模型](#数据模型)
        - [序列化](#序列化)
        - [输入与输出的严格性](#输入与输出的严格性)
    - [HTTP Headers](#http-headers)
        - [假设](#假设)
        - [HTTP 标准 Headers](#http-标准-headers)
        - [HTTP 自定义 Header](#http-自定义-header)
- [超媒体](#超媒体)
    - [HATEOAS](#hateoas)
        - [Example](#example)
        - [allOf](#allof)
        - [anyOf/oneOf](#anyofoneof)

<!-- /TOC -->

# 规范释义

## 所用词汇

#### 资源（Resource）
REST 中的信息的关键抽象即资源。根据 [Fielding's dissertation section 5.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2) 的表述，任何可被命名的信息都可以是资源：一个文档或图片、一个临时的服务（如“洛杉矶今天的天气”）、一个其他资源的集合或一个真实的对象（如一个人）等。资源是一组实体的概念映射，而不是某个特定时间点上对应的特定实体。更准确地说，资源 `R` 是一个随时间变化的成员函数 `MR(t)`，对于同一个资源在同一个时间 `t` ，得到的映射的实体或值的集合是不变的。集合中的值可以是资源的表述和/或资源标识符。

#### 资源标识符（Resource Identifier）
REST 使用资源标识符来区分资源实例。标识符由命名机构（如提供 API 的组织）来分配，并确保在时间维度上维持其语义的有效性（即确保成员函数不发生变化），这样方能令资源标识符成为资源的引用 -  [Fielding's dissertation section 5.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2) 

#### 表述（Representation）
REST 组件使用表述来对当前或期望的资源状态进行呈现，且该表述会用于在不同组件间传递。表述由一组字节，以及对这组字节进行描述的元数据构成 -  [Fielding's dissertation section 5.2](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2) 

#### 领域（Domain）
根据维基百科 ，[领域模型](https://en.wikipedia.org/wiki/Domain_model) 是系统的抽象，用于表述特定方面的知识、变化或活动。领域模型的概念包括业务涉及的数据，以及业务和数据关联的规则。举例来说， PayPal 的领域模型包括支付，风控，履约，身份以及消费支持等。

#### 能力（Capability）
能力是从业务取向和消费端视角下，呈现的业务逻辑。API 按照能力来组织，可使 API 成为稳定的，系统业务视角下，可被消费端消费的体验式API。能力的例子包括履约，信用，分增，零售和风控等。

能力驱动接口，而领域则更粗粒度，更加贴近代码和组织架构。从服务的角度来说，能力和领域是相互独立的。

#### 命名空间（Namespace）
能力驱动着服务建模的过程，而命名空间则影响 API 组合（protfolio）的具体实现。命名空间是业务能力模型（Business Capability Model）的一部分，我们可以以`compliance`，`device`，`transfers`，`credit`，`limits`等作为命名空间。

命名空间应当反映同一个领域中组织起来的业务能力。领域的定义应当从消费端的视角体现平台能力的组织。在这里，反映公司层级、组织结构或者代码架构是不必要的。当我们需要反映 API 的目标或者客户导向的平台组织形式时，相应的领域需要被明确定义。随着时间的推移，潜在的服务实施和组织结构可能需要按照领域所划定的边界进行迁移。

#### 服务（Service）
无论具体的成员函数如何定义，也无论响应请求的软件类型是什么，服务负责提供通用的 API 对资源中值的集合进行操作。服务本身则是软件本身的组成部分，可以执行任意数量的函数。因此，思考存在的不同类型的服务是有益的。

逻辑上，我们可以把服务及其暴露的 API 分为两类：

1. 功能性 API（Capability APIs）：暴露了服务所实现的一般性可重用的业务能力。
2. 体验性 API（Experience-specific）：基于功能性 API 构建，可提供基于特定渠道，或基于特定上下文优化的一般性业务能力。上下文信息可包括时间、地点、设备、渠道、认证、用户、角色、权限等级等。

##### 功能性服务与 API
功能性 API 是可重用业务能力的公用接口。*公用* 在这里意味着这些接口仅被 体验式 API 服务，外部消费端和其他领域的内部消费端消费。

##### 体验性服务与 API
体验性服务在核心业务能力的基础上，进行转换与适配，使其符合特定的场景、渠道或是设备的需要。其输入/输出功能仅限服务调用。

#### 客户端 / API 客户端 / API 消费端
发起 API 请求，并消费 API 响应的实体。

# 服务设计原则

本节涵盖 API 服务设计的指导原则。服务涉及将特定能力的相关方法作为 API 暴露给内部或外部开发者、中间件、合作方或是分支机构。

以下是服务的核心设计原则。

## 松耦合（Loose Coupling）

**服务和消费端之间必须是松耦合的**

耦合指的是两件事物之间的联结或关系。我们可以参照（某个服务的）独立性来衡量耦合的程度。该原则提倡，在服务的设计中，不论是服务间、服务与其实现、或是服务与消费端之间，都应当注意减少相互依赖。

松耦合原则可以提升设计的独立性，也使得服务的逻辑和实现保持独立。但该原则并不改变消费端必须依赖服务能力这一基本关系。

该原则也意味着：

* 服务不应当暴露实现细节
* 服务可以在不影响现有消费端的情况下进行演化
* 特定领域的服务可以独立于其他领域进行演化

## 封装（Encapsulation）

**领域服务只能通过其他服务获取不属于自己的数据和方法**

服务公开的功能包括它拥有和实现的功能和数据，以及它所依赖的功能和数据。该原则主张任何服务依赖和不拥有的功能或数据只能通过其他服务来访问。

该原则也意味着：

* 服务应当对功能和数据的所属权拥有清晰的边界
* 服务不能直接暴露不属于自己的数据

## 稳定（Stability）
**服务接口必须是稳定的**

服务接口的设计和演变必须保证对现有的消费端后向兼容。如果服务接口需要以不兼容的方式演变，应该清楚地传达出来。

该原则也意味着：

* 现有的服务应当在文档描述的时间段中都为客户端提供支持
* 功能的增加应当不影响现有的消费端
* 服务接口的弃用和迁移应当被清晰地表达，以建立消费端的预期

## 可复用（Reusable）
**服务必须以可复用的方式被开发，无论上下文或消费端的异同**

API 平台的主要目标是通过使用和组合服务，使应用程序能够被多快好省地开发。只有在服务接口是针对不同使用场景和不同消费端灵活地开发时，这才是有可能的。这一原则主张服务的开发方式应使其能够被不同消费端和环境使用，消费端或环境可能会随着时间的推移而发生变化。

该原则也意味着：

* 服务接口应当不仅仅为当下进行设计，应当能够支持针对不同消费端和场景的扩展
* 服务接口可能需要随着时间的推移对更多的消费端和场景进行支持

## 基于接口约定的（Contract-based）
**功能和数据必须通过标准的服务接口约定暴露**

服务通过服务接口约定暴露其意图和能力。服务接口的约定包括了功能性、非功能性（如可用性，响应时间）和业务性（如每单成本、条约和限制）等方面。*标准化*意味着服务接口约定必须遵守接口设计的标准。

这一原则主张所有的功能和数据只应当通过标准化的服务接口约定暴露。因此，消费端只能通过服务接口约定来解读和访问功能和数据。

该原则也意味着：

* 功能和数据不允许在服务接口约定外被解读或获取
* （数据存储中的）每一部分数据都仅属于单一的服务

## 一致性（Consistency）
**服务必须遵循一系列通用的规则、交互形式、词汇表和共享的类型**

使用同一套规则进行服务定义，以便以一致的方式暴露这些服务。这一原则通过减少消费端对新服务的学习曲线，增加了API平台的易用性。

该原则也意味着：

* 服务要遵守的所定义的标准
* 服务间应该使用来自通用词典中的一致性词汇
* 一致的交互方式、服务粒度和类型是操作兼容和组合便利的关键

## 易用性（Ease Of Use）
**服务对于消费端和应用程序来说应当是容易使用和组合的**

一个难以使用的服务会削减微服务架构的带来好处，并且鼓励消费端去寻找其他达成相同目的手段。可组合性（Composability）意味着服务可以很容易地组合起来，因为服务的接口约定和访问协议是一致的，而且每个服务接口约定都不需要有不同的理解。

该原则也意味着：

* 服务的接口约定是容易被发现和理解的
* 服务的接口约定和协议在各方面都是一致的，如身份和权限验证机制、错误语义、基本类型的使用、以及分页等。
* 服务拥有明确的所有权，因此消费端开发者可以与服务所有者就服务水平协议，要求，问题达成一致
* 消费端开发者可以轻松地集成，测试和部署使用此服务的消费端
* 消费端开发者可以容易地监视服务的非功能方面

## 可外部化的（Externalizable）
**服务应当被设计得易于将其提供的功能外部化**

使用开发服务的消费端，可能来自另一个领域或团队，另一个业务部门或另一个公司。在所有这些情况下，暴露的功能应当都是相同的，改变的只有访问机制或服务实施的策略，如认证，授权和限流。由于暴露的功能是相同的，服务应该仅被设计一次，然后根据业务需求通过适当的策略外部化。

该原则也意味着：

* 服务接口必须从领域模型派生出来，并且能够支持其预期的使用场景
* 服务的接口约定和访问协议必须得到足够的支持，以满足消费端的需求
* 服务的外部化不一定要求实现或服务接口约定的变更


# HTTP 方法, Header 与状态

## 数据资源与 HTTP 方法

### 业务能力与资源建模

一个组织的业务能力通常以 API 的形式暴露。不同的 API **不得（MUST NOT）** 提供相同的功能，但资源（如用户账户或信息卡）在不同的场景下是可重用的。

### HTTP 方法

大多数服务都可用标准数据资源模型提供服务，其主要操作可以用缩写CRUD（创建，读取，更新和删除）来表示。这些操作能够很好地映射到标准的 HTTP 动词上。

|HTTP 方法|描述|
|---|---|
| `GET`| _获取_ 一个资源 |
| `POST`| _创建_ 一个资源， 或为一个资源 _执行_ 复杂操作 |
| `PUT`| _更新_ 一个资源 |
| `DELETE`| _删除_ 一个资源 |
| `PATCH`| _部分更新_ 一个资源 |

实际的操作 **必须（MUST）** 与上表中 HTTP 方法的语义一致。

* **`GET`** 方法 **不得（MUST NOT）** 有副作用， **不得（MUST NOT）** 改变资源的状态。
* **`POST`** 方法 **应该（SHOULD）** 被用于在集合中创建新的资源。
    * **如：** 增加一张信用卡 `POST https://api.foo.com/v1/vault/credit-cards`
    * 语义的幂等性: 如果这是相同调用的后续执行（包含[`Foo-Request-Id`](#http-自定义-header)Header），并且资源已经被创建，那么请求 **应该（SHOULD）** 是幂等的。
* **`POST`** 方法 **应该（SHOULD）** 被用于创建新的次级资源并建立其与主体资源的关系。
    * **如：** 为付款 ID 为 12345 的记录发起退款: `POST https://api.foo.com/v1/payments/payments/12345/refund`
* **`POST`** 方法 **可以（MAY）** 被用于以其命名的复杂操作。这也被称作[_控制器模式_](patterns.md#controller-resource)，通常认为这是 RESTful 模型的例外情况。在资源代表了一个业务过程，而 `POST` 操作是其中的步骤或行为时，这种模式更为适用。更多信息可参考[RESTful Web Services Cookbook][29]，[section 2.6](http://techbus.safaribooksonline.com/9780596809140/chapter-identifying-resources)。
* **`PUT`** 方法 **应该（SHOULD）** 被用于更新资源的属性或是建立资源与现有次级资源的关系。在建立关系的情况下，应当更新主体资源到次级资源的引用。

## 处理

我们约定在指导原则中，
请求主体和响应主体都 **必须（MUST）** 使用[JavaScript Object Notation（JSON）][30]发送。 JSON 是由无序键值对组成的一种轻量级数据交换格式。 JSON 可以表示四种基本类型（字符串，数字，布尔值和空值）以及两种结构化类型（对象和数组）。处理 API 方法调用时， **应该（SHOULD）** 遵循以下原则：

### 数据模型

表示层的数据模型必须符合 [RFC 7159](https://tools.ietf.org/html/rfc7159) 中 JSON 数据传输格式的描述。

### 序列化

* 资源节点 **必须（MUST）** 支持 `application/json` 内容类型。
* 如果请求发送了 `Accept` 头，但 `application/json` 并不是可接受的返回类型，则必须返回 `406 Not Acceptable` 错误。

### 输入与输出的严格性

API **必须（MUST）** 对返回的内容进行严格约束，同时也 **应该（SHOULD）** 对输入进行严格约束。


我们开发的是编程接口，所以我们需要尽量避免猜测我们会收到什么内容。通常，集成对于开发者来说是一次性任务，因此在我们提供了良好文档的情况下，我们应当严格检查开发者的输入。另外，我们也应当考虑到 [Postel定律](https://en.wikipedia.org/wiki/Robustness_principle) 与不严格地输入解析所带来的诸多危险。

## HTTP Headers

HTTP header 的目的是为请求体提供原信息，或是以统一、标准化且独立的方式提供发送者的信息。HTTP header 的名称是大小写 **不** 敏感的。

* HTTP header **应该（SHOULD）** 仅被用于提供切面处理所需的信息。
* API 实现 **不该（SHOULD NOT）** 引入或依赖于 HTTP header。
* HTTP header **不得（MUST NOT）** 包含 API 或领域相关的特殊信息。
* 在可能的情况下，**必须（MUST）** 使用 HTTP 标准 header 而非自定义 header。

### 假设

**服务消费者与服务提供者：**

* **不该（SHOULD NOT）** 认为自己可以获取某个特定的 HTTP header。HTTP 调用的中间环节可能会抛弃某些 HTTP header，因此我们的业务逻辑 **不该（SHOULD NOT）** 依赖于 HTTP header。
* **不该（SHOULD NOT）** 假设在 HTTP 信息传输的过程中，header 的值不会被更改。

**基础设施组件** (如 Web 服务框架、客户端请求库、企业服务总线（ESB）、负载均衡（LB）等涉及 HTTP 消息传输的部分):

* **可以（MAY）** 基于特定 header 携带信息的有效性与正确性，直接返回错误信息，而不需要将信息继续转发至下一层。如客户端认证机制带来的鉴权错误即属于这种情况。
* **可以（MAY）** 增加、删除或变更某个 HTTP header 的值。

### HTTP 标准 Headers

标准 Header 在 [HTTP/1.1 规范 (RFC 7231)](http://tools.ietf.org/html/rfc7231#page-33) 中被定义。他们的目的、语法、值和语义都已被清晰地定义并可被多种基础设施组件所理解。

| HTTP Header 名称 | 描述 |
|-------------|------------|
| `Accept` | 该请求头制定了 API 客户端所能处理的响应媒体类型。系统假设 HTTP 请求 **应该（SHOULD）** 发送这个 header。处理请求的服务 **不该（SHOULD NOT）** 假设自己一定能够获取到这个 header。我们假设遵循该 API 规范的 API 都会支持 `application/json` |
| `Accept-Charset` | 该请求头指定了 API 客户端能够处理的相信字符集<ul><li>`Accept-Charset` **应该（SHOULD）** 包含  `utf-8`</li></ul> |
| `Content-Language` | 该响应/请求头用于指定内容的语言，默认语言为 `en-US`（译注：PayPal 的情况）。API 客户端 **应该（SHOULD）** 使用 `Content-Language` header 指定内容的语言。API **必须（MUST）** 在响应中包含该 header。 <br/><br/>例如：<pre>Content-Language: en-US</pre> |
| `Content-Type` | 该响应/请求头用于指定请求或返回的媒体类型。<br/><ul><li>API 客户端请求在请求体存在的情况下 **必须（MUST）** 包含该请求头, 如 `POST`, `PUT`, 或 `PATCH` 请求。</li><li>只要返回有响应体（不是 `204` 响应），API 开发者就 **必须（MUST）** 在 HTTP 响应中使用该 header。</li><li>如果内容基于文本类型，如[JSON][30], 则 `Content-Type` **必须（MUST）包含字符集参数，且必须为 UTF-8。</li><li> 目前来说，唯一支持的媒体类型即 `application/json`</li></ul>例如:<pre>(在 HTTP 请求中)    Accept: application/json<br/>                   Accept-Charset: utf-8<br/>(在 HTTP 响应中)   Content-Type: application/json; charset=utf-8</pre> |
| `Link` | 根据 [Web Linking RFC 5988](https://tools.ietf.org/html/rfc5988), 链接（link）指的是两个资源之间以国际资源定位符（Internationalised Resource Identifiers，IRIs）所表示的连接关系。 `Link` 提供了在 HTTP header 中序列化一个或多个链接的手段。 <br><br> API **应该（SHOULD）** 基于如下假设被构建：API 本身及其客户端的业务逻辑不会依赖 header 中的信息。 Headers 只可被用于提供切面处理所需的信息，如安全、可追踪性或监控相关等方面。<br><br>因此， `Link` header 不得在状态码为 `201` 或 `300` 的响应中使用。可以考虑在返回体中使用 [HATEOAS 链接](#超媒体) 作为替代。 |
| `Location` | 该响应头被用于重定向到一个新的地址以完成请求或对新的资源进行定位。 <br><br> API **应该（SHOULD）** 基于如下假设被构建：API 本身及其客户端的业务逻辑不会依赖 header 中的信息。 Headers 只可被用于提供切面处理所需的信息，如安全、可追踪性或监控相关等方面。<br><br>因此， `Link` header 不得在状态码为 `201` 或 `300` 的响应中使用。可以考虑在返回体中使用 [HATEOAS 链接](#超媒体) 作为替代。 | 
| `Prefer` | [`Prefer`](https://tools.ietf.org/html/rfc7240) 请求头被用于告知服务端，某种特定的行为是被客户端所`偏好(prefferred)`的，但不是客户端得以完成请求的必要条件。这是一个 **必须（MUST）** 被代理转发的`端到端 (end to end)` 字段，除非其在 `Connection` 中被显式地标记为 `节点到节点（hop by hop）`。若 API 文档声称支持 `Prefer`，则以下值应当是可用的。<br><br>`respond-async`: API 客户端期望 API 服务端异步地处理其请求。<pre>Prefer: respond-async </pre> 服务端返回 `202 (Accepted)` 并且异步地处理请求。在处理完成后，服务端可以使用 webhook 通知客户端，也可由客户端在稍后发起 `GET` 请求获取处理结果。更多信息可参考 [Asynchronous Operations](patterns.md#asynchronous-operations)。<br/><br/>`read-consistent`: API 客户端期望 API 服务端从持久化的存储中返回一致性的数据。对于不支持对其进行选择 API 来说，这应当是默认的行为，无需要求客户端进行设置<pre>Prefer: read-consistent</pre>`read-eventual-consistent`: API 期望 API 服务端返回的是最终一致性的数据，其来源可以是缓存或是推定的最终一致数据源。如果在这两类数据源都没有数据，则 API 服务可能会返回持久化存储中的数据<pre>Prefer: read-eventual-consistent</pre>`read-cache`: API 客户端期望 API 服务端返回缓存数据。如果缓存未命中，则可能从最终一致数据源或持久化存储中返回数据<pre>Prefer: read-cache</pre>`return=representation`: API 客户端期望API 服务端包含请求成功后相应资源实体的表现层状态。在创建资源（`POST`）或更改资源（`PUT`或`PATCH`）后，可以将 `Perfer` 设为该值以省去后续使用 `GET` 请求获取变更后的资源需要，以优化服务端与客户端的通信。<pre>Prefer: return=representation</pre>`return=minimal`: API 客户端期望服务端对于成功的请求返回最小响应即可。怎样定义合理的 “最小”仅取决于服务端<pre>Prefer: return=minimal</pre>|


### HTTP 自定义 Header

以下是自定义 header 的指南，这并不是 HTTP 规范的一部分。

| HTTP Header 名称 | 描述 |
|:-------------|------------|
| `Foo-Request-Id`    | API 消费端 **可以（MAY）** 选择以唯一 ID 发送该请求头，以便于追踪请求。  <br/><br/>该请求头也可被用于内部日志的记录。在同步的响应或是 webhook 中，**建议（RECOMMENDED）**将请求头中的该字段作为响应头返回。 |



<h3 id="http-header-propagation">HTTP Header Propagation</h3>

When services receive request headers, they MUST pass on relevant custom headers in addition to the HTTP standard headers in requests/messages dispatched to downstream applications. 


<h2 id="http-status-codes">HTTP Status Codes</h2>

RESTful services use HTTP status codes to specify the outcomes of HTTP method execution. HTTP protocol specifies the outcome of a request execution using an integer and a message. The number is known as the _status code_ and the message as the _reason phrase_. The reason phrase is a human readable message used to clarify the outcome of the response. HTTP protocol categorizes status codes in ranges.

<h3 id="status-code-ranges">Status Code Ranges</h3>

When responding to API requests, the following status code ranges MUST be used.

|Range|Meaning|
|---|---|
|`2xx`|Successful execution. It is possible for a method execution to succeed in several ways. This status code specifies which way it succeeded.|
|`4xx`|Usually these are problems with the request, the data in the request, invalid authentication or authorization, etc. In most cases the client can modify their request and resubmit.|
| `5xx`| Server error: The server was not able to execute the method due to site outage or software defect. 5xx range status codes SHOULD NOT be utilized for validation or logical error handling. |

<h3 id="status-reporting">Status Reporting</h3>

Success and failure apply to the whole operation not just to the SOA framework portion or to the business logic portion of code exectuion. 

Following are the guidelines for status codes and reason phrases.

* Success MUST be reported with a status code in the `2xx` range.
* HTTP status codes in the `2xx` range MUST be returned only if the complete code execution path is successful. This includes any container/SOA framework code as well as the business logic code execution of the method.
* Failures MUST be reported in the `4xx` or `5xx` range. This is true for both system errors and application errors.
* There MUST be a consistent, JSON-formatted error response in the body as defined by the [`error.json`][22] schema. This schema is used to qualify the kind of error. Please refer to [Error Handling](#error-handling) guidelines for more details.
* A server returning a status code in the `4xx` or `5xx` range MUST return the `error.json` response body.
* A server returning a status code in the `2xx` range MUST NOT return response following `error.json`, or any kind of error code, as part of the response body.
* For client errors in the `4xx` code range, the reason phrase SHOULD provide enough information for the client to be able to determine what caused the error and how to fix it.
* For server errors in the `5xx` code range, the reason phrase and an error response following `error.json` SHOULD limit the amount of information to avoid exposing internal service implementation details to clients. This is true for both external facing and internal APIs. Service developers should use logging and tracking utilities to provide additional information.

<h3 id="allowed-status-codes">Allowed Status Codes List</h3>

All REST APIs MUST use only the following status codes. Status codes in **`BOLD`** SHOULD be used by API developers. The rest are primarily intended for web-services framework developers reporting framework-level errors related to security, content negotiation, etc.

* APIs MUST NOT return a status code that is not defined in this table.
* APIs MAY return only some of status codes defined in this table.

| Status Code | Description |
|-------------|-------------|
| **`200 OK`** | Generic successful execution. |
| **`201 Created`** | Used as a response to `POST` method execution to indicate successful creation of a resource. If the resource was already created (by a previous execution of the same method, for example), then the server should return status code `200 OK`. |
| **`202 Accepted`** | Used for asynchronous method execution to specify the server has accepted the request and will execute it at a later time. For more details, please refer [Asynchronous Operations](patterns.md#asynchronous-operations). |
| **`204 No Content`** | The server has successfully executed the method, but there is no entity body to return.|
| **`400 Bad Request`** | The request could not be understood by the server. Use this status code to specify:<br/> <ul><li>The data as part of the payload cannot be converted to the underlying data type.</li><li>The data is not in the expected data format.</li><li>Required field is not available.</li><li>Simple data validation type of error.</li></ul>|
| `401 Unauthorized` | The request requires authentication and none was provided. Note the difference between this and `403 Forbidden`. |
| **`403 Forbidden`** | The client is not authorized to access the resource, although it may have valid credentials. API could use this code in case business level authorization fails. For example, accound holder does not have enough funds. |
| **`404 Not Found`** | The server has not found anything matching the request URI. This either means that the URI is incorrect or the resource is not available. For example, it may be that no data exists in the database at that key. |
| `405 Method Not Allowed` | The server has not implemented the requested HTTP method. This is typically default behavior for API frameworks.
| `406 Not Acceptable` | The server MUST return this status code when it cannot return the payload of the response using the media type requested by the client. For example, if the client sends an `Accept: application/xml` header, and the API can only generate `application/json`, the server MUST return `406`. |
| `415 Unsupported Media Type` | The server MUST return this status code when the media type of the request's payload cannot be processed. For example, if the client sends a `Content-Type: application/xml` header, but the API can only accept `application/json`, the server MUST return `415`. |
| **`422 Unprocessable Entity`** | The requested action cannot be performed and may require interaction with APIs or processes outside of the current request. This is distinct from a 500 response in that there are no systemic problems limiting the API from performing the request. |
| `429 Too Many Requests` | The server must return this status code if the rate limit for the user, the application, or the token has exceeded a predefined value. Defined in Additional HTTP Status Codes [RFC 6585](https://tools.ietf.org/html/rfc6585). |
| **`500 Internal Server Error`** | This is either a system or application error, and generally indicates that although the client appeared to provide a correct request, something unexpected has gone wrong on the server. A `500` response indicates a server-side software defect or site outage. `500` SHOULD NOT be utilized for client validation or logic error handling. |
| `503 Service Unavailable` | The server is unable to handle the request for a service due to temporary maintenance. |

<h3 id="mapping">HTTP Method to Status Code Mapping</h3>

For each HTTP method, API developers SHOULD use only status codes marked as "X"  in this table. If an API needs to return any of the status codes marked with an **`X`**, then the use case SHOULD be reviewed as part of API design review process and maturity level assessment. Most of these status codes are used to support very rare use cases.

| Status Code | 200 Success | 201 Created |202 Accepted | 204 No Content | 400 Bad Request |  404 Not Found |422 Unprocessable Entity | 500 Internal Server Error |
|-------------|:------------|:------------|:------------|:---------------|:----------------|:---------------|:------------------------|:--------------------------|
| `GET`       | X         |               |               |                  | X             | X              | **`X`**           | X                       |
| `POST`      | X         | X         | **`X`**           |                 |  X             | **`X`**              | **`X`**               | X                       |
| `PUT`       | X             |               | **`X`**           | X           | X             | X              | **`X`**           | X                       |
| `PATCH`     | X             |               |               | X           | X            | X              | **`X`**           | X                       |
| `DELETE`    | X             |               |               | X           | X             | X              | **`X`**               | X                       |



* `GET`: The purpose of the `GET` method is to retrieve a resource. On success, a status code `200` and a response with the content of the resource is expected. In cases where resource collections are empty (0 items in `/v1/namespace/resources`), `200` is the appropriate status (resource will contain an empty `items` array). If a resource item is 'soft deleted' in the underlying data, `200` is not appropriate (`404` is correct) unless the 'DELETED' status is intended to be exposed.

* `POST`: The primary purpose of `POST` is to create a resource. If the resource did not exist and was created as part of the execution, then a status code `201` SHOULD be returned.
    * It is expected that on a successful execution, a reference to the resource created (in the form of a link or resource identifier) is returned in the response body.
    * Idempotency semantics: If this is a subsequent execution of the same invocation (including the [`Foo-Request-Id`](#http-custom-headers) header) and the resource was already created, then a status code of `200` SHOULD be returned. For more details on idempotency in APIs, refer to [idempotency](patterns.md#idempotency).
	* If a sub-resource is utilized ('controller' or data resource), and the primary resource identifier is non-existent, `404` is an appropriate response.

* `POST` can also be used while utilizing the [controller pattern](patterns.md##controller-resource), `200` is the appropriate status code.

* `PUT`: This method SHOULD return status code `204` as there is no need to return any content in most cases as the request is to update a resource and it was successfully updated. The information from the request should not be echoed back. 
    * In rare cases, server generated values may need to be provided in the response, to optimize client flow (if the client necessarily has to perform a `GET` after `PUT`). In these cases, `200` and a response body are appropriate. 

* `PATCH`: This method should follow the same status/response semantics as `PUT`, `204` status and no response body.
	* `200` + response body should be avoided at all costs, as `PATCH` performs partial updates, meaning multiple calls per resource is normal. As such, responding with the entire resource can result in large bandwidth usage, especially for bandwidth-sensitive mobile clients.

* `DELETE`: This method SHOULD return status code `204` as there is no need to return any content in most cases as the request is to delete a resource and it was successfully deleted.

    * As the `DELETE` method MUST be idempotent as well, it SHOULD still return `204`, even if the resource was already deleted. Usually the API consumer does not care if the resource was deleted as part of this operation, or before. This is also the reason why `204` instead of `404` should be returned.



# 超媒体

## HATEOAS

Hypermedia, an extension of the term [hypertext](https://en.wikipedia.org/wiki/Hypertext), is a nonlinear medium of information which includes graphics, audio, video, plain text and hyperlinks according to [wikipedia](https://en.wikipedia.org/wiki/Hypermedia). Hypermedia As The Engine Of Application State (`HATEOAS`) is a constraint of the REST application architecture described by Roy Fielding in his [dissertation][0]. 

In the context of RESTful APIs, a client could interact with a service entirely through hypermedia provided dynamically by the service. A hypermedia-driven service provides representation of resource(s) to its clients to navigate the API dynamically by including hypermedia links in the responses. This is different than other form of SOA, where servers and clients interact based on WSDL-based specification defined somewhere on the web or exchanged off-band.


<h2 id="hateoas-api">Hypermedia Compliant API</h2>

A hypermedia compliant API exposes a finite state machine of a service. Here, requests such as `DELETE`, `PATCH`, `POST` and `PUT` typically initiate a transition in state while responses indicate the change in the state. Lets take an example of an API that exposes a set of operations to manage a user account lifecycle and implements the HATEOAS interface constraint.

A client starts interaction with a service through a fixed URI `/users`. This fixed URI supports both `GET` and `POST` operations. The client decides to do a `POST` operation to create a user in the system.

**Request**:

```

POST https://api.foo.com/v1/customer/users

{
    "given_name": "James",
    "surname" : "Greenwood",
    ...

}

```


**Response**:

The API creates a new user from the input and returns the following links to the client in the response.

  * A link to retrieve the complete representation of the user (aka `self` link) (`GET`).
  * A link to update the user (`PUT`).
  * A link to partially update the user (`PATCH`).
  * A link to delete the user (`DELETE`).


```

{
HTTP/1.1 201 CREATED
Content-Type: application/json
...

"links": [
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "self",
        
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "delete",
        "method": "DELETE"
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "replace",
        "method": "PUT"
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "edit",
        "method": "PATCH"
    }
]

}
```

A client can store these links in its database for later use. 

A client may then want to display a set of users and their details before the admin decides to delete one of the users. So the client does a `GET` to the same fixed URI `/users`.



**Request**:

```
GET https://api.foo.com/v1/customer/users

```

The API returns all the users in the system with respective `self` links.

**Response**:

```

{
    "total_items": "166",
    "total_pages": "83",
    "users": [
    {
        "given_name": "James",
        "surname": "Greenwood",
        ...
        "links": [
            {
                "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
                "rel": "self"
            }
        ]
    },
    {
        "given_name": "David",
        "surname": "Brown",
        ...
        "links": [
            {
                "href": "https://api.foo.com/v1/customer/users/ALT-MDFSKFGIFJ86DSF",
                "rel": "self"
            }
   },
   ...
}

```
The client MAY follow the `self` link of the user and figure out all the possible operations that it can perform on the user resource. 

**Request**:

```
GET https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI
```


**Response**:

```
HTTP/1.1 200 OK
Content-Type: application/json
{
        "given_name": "James",
        "surname": "Greenwood",
        ...
        "links": [
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "self",
        
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "delete",
            "method": "DELETE"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "replace",
            "method": "PUT"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "edit",
            "method": "PATCH"
        }


}
```


To delete the user, the client retrieves the URI of the link relation type `delete` from its data store and performs a delete operation on the URI.

**Request**:

```
DELETE https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI
```

In summary:

* There is a well defined entry point for an API which a client navigates to in order to access all other resources.
* The client does not need to build the logic of composing URIs to execute different requests or code any kind of business rule by looking into the response details (more in detail is described in the later sections) that may be associated with the URIs and state changes.
* The client acknowledges the fact that the process of creating URIs belongs to the server.
* Client treats URIs as opaque identifiers.
* APIs using hypermedia in representations could be extended seamlessly. As new methods are introduced, responses could be extended with relevant HATEOAS links. In this way, clients could take advantage of the functionality in incremental fashion. For example, if the API starts supporting a new `PATCH` operation then clients could use it to do partial updates.

The mere presence of links does not decouple a client from having to learn the data required to make requests for a transition and all associated link semantics, particularly for `POST`/`PUT`/`PATCH` operations. An API MUST provide documentation to clearly describe all the links, link relation types and request response formats for each of the URIs.

Subsequent sections provide more details about the structure of a link and what different relationship types mean.


<h2 id="link-description-object">Link Description Object</h2>

Links MUST be described using the *[Link Description Object (LDO)] [4]* schema. An LDO describes a single link relation in the links array. Following is brief description for properties of Link Description Object.

* `href`:

    * A value for the `href` property MUST be provided.  

    * The value of the `href` property MUST be a *[URI template] [6]* used to determine the target URI of the related resource. It SHOULD be resolved as a URI template per [RFC 6570](https://tools.ietf.org/html/rfc6570).  

    * Use ONLY absolute URIs as a value for `href` property. Clients usually bookmark the absolute URI of a link relation type from the representation to make API requests later. Developers MUST use the *[URI Component Naming Conventions](#uri-component-names)* to construct absolute URIs. The value from the incoming `Host` header (e.g. api.foo.com) MUST be used as the `host` field of the absolute URI. 

* `rel`: 
    * `rel` stands for relation as defined in Link Relation Type 

    * The value of the `rel` property indicates the name of the relation to the target resource.   

    * A value for the `rel` property MUST be provided.   

* ` method`: 
    * The `method` property identifies the HTTP verb that MUST be used to make a request to the target of the link. The `method` property assumes a default value of `GET` if it is ommitted.

* `title`: 
    * The `title` property provides a title for the link and is a helpful documentation tool to facilitate understanding by the end clients. This property is NOT REQUIRED.


<h4 id="not-using-http-headers-for-ldo">Not Using HTTP Headers For LDO</h4>

Note that these API guidelines do not recommend using the HTTP `Location` header to provide a link. Also, they do not recommend using the `Link` header as described in [JAX-RS](https://java.net/projects/jax-rs-spec/pages/Hypermedia). The scope of HTTP header is limited to point-to-point interaction between a client and a service. Since responses might be passed around to other layers and components on the client side which may not directly interact with the service, any information that is stored in a header may not be available. Therefore, we recommend returning Link Description Object(s) in HTTP response body. 

<h2 id="links-array">Links Array</h2>

The `links` array property of schemas is used to associate a Link Description Objects with a *[JSON hyper-schema draft-04] [3]* instance.   

* This property MUST be an array.
* Items in the array must be of type Link Description Object.

<h4 id="specifying-links-array">Specifying the Links array</h4>

Here's an example of how you would describe links in the schema.

* A links array similar to the one defined in the sample JSON schema below MUST be provided as part of the API resource schema definition. Please note that the links array needs to be declared within the `properties` keyword of an object. This is required for code generators to add setter/getter methods for the links array in the generated object.
* All possible links that an API returns as part of the response MUST be declared in the response schema using a URI template. The links array of URI templates MUST be declared outside the `properties` keyword.

```
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-04/hyper-schema#",
    "description": "A sample resource representing a customer name.",
    "properties": {
        "id": {
	    "type": "string",
            "description": "Unique ID to identify a customer."
        },
        "first_name": {
            "type": "string",
            "description": "Customer's first name."
        },
        "last_name": {
            "type": "string",
            "description": "Customer's last name."
        },
        "links": {
            "type": "array",
            "items": {
                "$ref": "http://json-schema.org/draft-04/hyper-schema#definitions/linkDescription"
            }
        }
    },
    "links": [
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "self"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "delete",
            "method": "DELETE"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "replace",
            "method": "PUT"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "edit",
            "method": "PATCH"
        }
    ]

}
```


Below is an example response that is compliant with the above schema.

```
{
    "id": "ALT-JFWXHGUV7VI",
    "first_name": "John",
    "last_name": "Doe",
    "links": [
        {
            "href": "https://api.foo.com/v1/cusommer/users/ALT-JFWXHGUV7VI",
            "rel": "self"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "edit",
            "method": "PATCH"
        }
    ]
}
```


<h2 id="link-relation-type">Link Relation Type</h2>

A `Link Relation Type` serves as an identifier for a link. An API MUST assign a meaningful link relation type that unambiguously describes the semantics of the link. Clients use the relevant Link Relation Type in order to identify the link to use from a representation.

When the semantics of a Link Relation Type defined in *[IANA's list of standardized link relations] [5]* matches with the one you want to define, then it MUST be used. The table below describes some of the commonly used link relation types. It also lists some additional lin relation types defined by these guidelines.

|Link Relation Type | Description|
|---------|------------|
|`self`  | Conveys an identifier for the link's context. Usually a link pointing to the resource itself.|
|`create` | Refers to a link that can be used to create a new resource.|
|`edit` | Refers to editing (or partially updating) the representation identified by the link. Use this to represent a `PATCH` operation link.|
|`delete` | Refers to deleting a resource identified by the link. Use this `Extended link relation type` to represent a `DELETE` operation link.|
|`replace` | Refers to completely update (or replace) the representation identified by the link. Use this `Extended link relation type` to represent a `PUT` operation link.|
|`first` | Refers to the first page of the result list.|
|`last` | Refers to the last page of the result list provided `total_required` is specified as a query parameter.|
|`next` | Refers to the next page of the result list.|
|`prev` | Refers to the previous page of the result list.|
|`collection` | Refers to a collections resource (e.g /v1/users).|
|`latest-version` | Points to a resource containing the latest (e.g., current) version.|
|`search` | Refers to a resource that can be used to search through the link's context and related resources.|
|`up` | Refers to a parent resource in a hierarchy of resources.|

For all [`controller`](patterns.md#controller-resource) style complex operations, the controller `action` name must be used as the link relation type (e.g. `activate`,`cancel`,`refund`).



<h2 id="use-cases">Use Cases</h2>

See [HATEOAS Use Cases](patterns.md#hateoas-use-cases) to find where HATEOAS could be used. 


<h1 id="naming-conventions">Naming Conventions</h1>

Naming conventions for URIs, query parameters, resources, fields and enums are described in this section. Let us emphasize here that these guidelines are less about following the conventions exactly as described here but they are more about defining some naming conventions and sticking to them in a consistent manner while designing APIs. For example, we have followed [snake_case](https://en.wikipedia.org/wiki/Snake_case) for field and file names, however, you could use other forms such as [CamelCase](https://en.wikipedia.org/wiki/Camel_case) or something else that you have devised yourself. It is important to adhere to a defined convention. 

<h2 id="uri-component-names">URI Component Names</h2>


URIs follow [RFC 3986][8] specification. This specification simplifies REST API service development and consumption. The guidelines in this section govern your URI structure and semantics following the RFC 3986 constraints.

<h3 id="uri-components">URI Components</h3>

According to RFC 3986, the generic URI syntax consists of a hierarchical sequence of components referred to as the scheme, authority, path, query, and fragment as shown in example below.
   

```
         https://example.com:8042/over/there?name=ferret#nose
         \___/   \_______________/\_________/\_________/\__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
```

<h3 id="uri-naming-convention">URI Naming Conventions</h3>

Following is a brief description of the URI specific naming convention guidelines for APIs. This specification uses parentheses "( )" to group, an asterisk " * " to specify zero or more occurrences, and brackets "[ ]" for optional fields.

```
[scheme"://"][host[':'port]]"/v" major-version '/'namespace '/'resource ('/'resource)* '?' query

```

* URIs MUST start with a letter and use only lower-case letters.
* Literals/expressions in URI paths SHOULD be separated using a hyphen ( - ). 
* Literals/expressions in query strings SHOULD be separated using underscore ( _ ).
* URI paths and query strings MUST percent encode data into [UTF-8 octets](https://en.wikipedia.org/wiki/Percent-encoding).
* Plural nouns SHOULD be used in the URI where appropriate to identify collections of data resources.
  * `/invoices`
  * `/statements`
* An individual resource in a collection of resources MAY exist directly beneath the collection URI.
  * `/invoices/{invoice_id}`
* Sub-resource collections MAY exist directly beneath an individual resource. This should convey a relationship to another collection of resources (invoice-items, in this example).
  * `/invoices/{invoice_id}/items`
* Sub-resource individual resources MAY exist, but should be avoided in favor of top-level resources.
  * `/invoices/{invoice_id}/items/{item_id}`
  * Better: `/invoice-items/{invoice_item_id}`
* Resource identifiers SHOULD follow [recommendations](#resource-identifiers) described in subsequent section.

**Examples**

* `https://api.foo.com/v1/vault/credit-cards`
* `https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA`
* `https://api.foo.com/v1/payments/billing-agreements/I-V8SSE9WLJGY6/re-activate`


**Formal Definition:**

|Term|Defiition|
|---|---|
|URI|[end-point] '**/**' resource-path ['**?**'query]|
|end-point|[scheme "**://**"][ host ['**:**' port]]|
|scheme|"**http**" or "**https**"|
|resource-path|"**/v**" version  '**/**' namespace-name '**/**' resource ('**/**' resource)|
|resource|resource-name ['**/**' resource-id]|
|resource-name|Alpha (Alpha \| Digit \| '-')* |
|resource-id|value| 
|query|name '**=**' value ('**&**' name = value)* |
|name|Alpha (Alpha \| Digit \| '_')* |
|value|URI Percent encoded value|


Legend

```
'   Surround a special character with single quotes
"	Surround strings with double quotes
()	Use parentheses for grouping
[]	Use brackets to specify optional expressions
*	An expression can be repeated zero or more times
```

<h3 id="resource-names">Resource Names</h3>

When modeling a service as a set of resources, developers MUST follow these principles:

* Nouns MUST be used, not verbs.
* Resource names MUST be singular for singletons; collections' names MUST be plural.
    * A description of the automatic payments configuration on a user's account
        * `GET /autopay` returns the full representation
    * A collection of hypothetical charges:
        * `GET /charges` returns a list of charges that have been made
        * `POST /charges` creates a new charge resource, `/charges/1234`
        * `GET /charges/1234` returns a full representation of a single charge
        
* Resource names MUST be lower-case and use only alphanumeric characters and hyphens.
    * The hyphen character, ( - ), MUST be used as a word separator in URI path literals. **Note** that this is the only place where hyphens are used as a word separator. In nearly all other situations, the underscore character, ( _ ), MUST be used.


<h3 id="query-parameter-names">Query Parameter Names</h3>

* Literals/expressions in query strings SHOULD be separated using underscore ( _ ).
* Query parameters values MUST be percent-encoded.
* Query parameters MUST start with a letter and SHOULD be all in lower case. Only alpha characters, digits and the underscore ( _ ) character SHALL be used.
* Query parameters SHOULD be optional. 
* Some query parameter names are reserved, as indicated in [Resource Collections](#resource-collections).

For more specific info on the query parameter usage, see [URI Standards](#uri-standards-query).

<h2 id="field-names">Field Names</h2>

The data model for the representation MUST conform to [JSON][30]. The values may themselves be objects, strings, numbers, booleans, or arrays of objects.

* Key names MUST be lower-case words, separated by an underscore character, ( _ ).
    * `foo`
    * `bar_baz`
* Prefix such as `is_` or `has_` SHOULD NOT be used for keys of type boolean. 
* Fields that represent arrays SHOULD be named using plural nouns (e.g. authenticators-contains one or more authenticators, products-contains one or more products).  

<h2 id="enum-names">Enum Names</h2>

Entries (values) of an enum SHOULD be composed of only upper-case alphanumeric characters and the underscore character, ( _ ).

* `FIELD_10` 
* `NOT_EQUAL`

If there is an industry standard that requires us to do otherwise, enums MAY contain other characters.

<h2 id="link-relation-names">Link Relation Names</h2>

A link relation type represented by `rel` must be in lower-case.

* Example

```
"links": [
    {
        "href": "href": "https://uri.foo.com/v1/customer/partner-referrals/ALT-JFWXHGUV7VI/activate",
        "rel": "activate",
        "method": "POST"

    }
   ]
```

<h2 id="file-names">File Names </h2>

JSON schema for various types used by API SHOULD each be contained in separate files, referenced using the `$ref` syntax (e.g. `"$ref":"object.json"`). JSON Schema files SHOULD use underscore naming syntax, e.g. `transaction_history.json`. 


<h1 id="uri">URI</h1>

<h2 id="resource-path">Resource Path</h2>

An API's [resource path](#uri-components) consists of URI's path, query and fragment components. It would include API's major version followed by namespace, resource name and optionally one or more sub-resources. For example, consider the following URI. 

`https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA`

Following table lists various pieces of the above URI's resource path.

|Path Piece|Description|Definition|
|---|---|---|
|`v1`|Specifies major version 1 of the API|The API major version is used to distinguish between two backward-incompatible versions of the same API. The API major version is an integer value which MUST be included as part of the URI. |
|`vault`|The namespace|Namespace identifiers are used to provide a context and scope for resources. They are determined by logical boundaries in the business capability model implemented by the API platform.|
|`credit-cards`|The resource name|If the resource name represents a collection of resources, then the `GET` method on the resource should retrieve the list of resources. Query parameters should be used to specify the search criteria.|
|`CARD-7LT50814996943336KESEVWA`|The resource ID|To retrieve a particular resource out of the collection, a resource ID MUST be specified as part of the URI. Sub-resources are discussed below.|


<h4 id="subresource-path">Sub-Resources</h4>

Sub-resources represent a relationship from one resource to another. The sub-resource name provides a meaning for the relationship. If cardinality is 1:1, then no additional information is required. Otherwise, the sub-resource SHOULD provide a sub-resource ID for unique identification. If cardinality is 1:many, then all the sub-resources will be returned. No more than two levels of sub-resources SHOULD be supported.

|Example|Description|
|---|---|
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents`|This call should return all the documents associated with dispute ABCD1234.|
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents/102030`|This call should return only the details for a particular document associated with this dispute. Keep in mind that this is only an illustrative example  to show how to use sub-resource IDs. In practice, two step invocations SHOULD be avoided. If second identifier is unique, top-level resource (e.g. `/v1/customer-support/documents/102030`) is preferred.|
|`GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/transactions`|The following example should return all the transactions associated with this dispute and their details, so there SHOULD NOT be a need to specify a particular transaction ID. If specific transaction ID retrieval is needed, `/v1/customer-support/transactions/ABCD1234` is preferred (assuming IDs are unique).|

<h3 id="resource-identifiers">Resource Identifiers</h3>

[Resource identifiers](#resource-identifier) identify a resource or a sub-resource. These **MUST** conform to the following guidelines.

* The lifecycle of a resource identifier MUST be owned by the resource's domain model, where they can be guaranteed to uniquely identify a single resource. 
* APIs MUST NOT use the database sequence number as the resource identifier.
* A [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier), Hashed Id ([HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code) based) is preferred as a resource identifier.
* For security and data integrity reasons all sub-resource IDs MUST be scoped within the parent resource only.<br />
**Example:** `/users/1234/linked-accounts/ABCD`<br /> Even if account "ABCD" exists, it MUST NOT be returned unless it is linked to user: 1234.
* Enumeration values can be used as sub-resource IDs. String representation of the enumeration value SHOULD be used.
* There MUST NOT be two resource identifiers one after the other.<br/>
**Example:** `https://api.foo.com/v1/payments/payments/12345/102030`
* Resource IDs SHOULD try to use either Resource Identifier Characters or ASCII characters. There **SHOULD NOT** be any ID using UTF-8 characters.
* Resource IDs and query parameter values MUST perform URI percent-encoding for any character other than URI unreserved characters. Query parameter values using UTF-8 characters MUST be encoded.

<h2 id="query-parameters">Query Parameters</h2>


Query parameters are name/value pairs specified after the resource path, as prescribed in RFC 3986.
[Naming Conventions](#naming-conventions) should also be followed when applying the following section.

#### Filter a resource collection

* Query parameters SHOULD be used only for the purpose of restricting the resource collection or as search or filtering criteria.
* The resource identifier in a collection SHOULD NOT be used to filter collection results, resource identifier should be in the URI.
* Parameters for pagination SHOULD follow [pagination](#pagination) guidelines.
* Default sort order SHOULD be considered as undefined and non-deterministic. If a explicit sort order is desired, the query parameter `sort` SHOULD be used with the following general syntax: `{field_name}|{asc|desc},{field_name}|{asc|desc}`. For instance: `/accounts?sort=date_of_birth|asc,zip_code|desc`

#### Query parameters on a single resource
In typical cases where one resource is utilized (e.g. `/v1/payments/billing-plans/P-94458432VR012762KRWBZEUA`), query parameters SHOULD NOT be used.

#### Cache-friendly APIs
In rare cases where a resource needs to be highly cacheable (usually data with minimal change), query parameters MAY be utilized as opposed to `POST` + request body. As `POST` would make the response uncacheable, `GET` is preferred in these situations. This is the only scenario in which query parameters MAY be required.

#### Query parameters with POST
When `POST` is utilized for an operation, query parameters are usually NOT RECOMMENDED in favor of request body fields. In cases where `POST` provides paged results (typically in complex search APIs where `GET` is not appropriate), query parameters MAY be used in order to provide hypermedia links to the next page of results.

#### Passing multiple values for the same query parameter

When using query parameters for search functionality, it is often necessary to pass multiple values. For instance, it might be the case that a resource could have many states, such as `OPEN`, `CLOSED`, and `INVALID`. What if an API client wants to find all items that are either `CLOSED` or `INVALID`?

* It is RECOMMENDED that APIs implement this functionality by repeating the query parameter. This is inherently supported by HTTP standards and already built in to most client libraries.
    * The query above would be implemented as `?status=CLOSED&status=INVALID`.
    * The parameter MUST be marked as repeatable in API specifications using `"repeated": true` in the parameter's definition section.
    * The parameter's name SHOULD be singular.

* URIs have practical length limits that are quite low - most conservatively, about [2,000 characters](https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers#417184). Therefore, there are situations where API designers MAY choose to use a single query parameter that accepts comma-separated values in order to accommodate more values in the query-string. Keep in mind that server and client libraries don't consistently provide this functionality, which means that implementers will need to write additional string parsing code. Due to the additional complexity and divergence from HTTP standards, this solution is NOT RECOMMENDED unless justified.
    * The query above would be implemented as `?statuses=CLOSED,INVALID`.
    * The parameter MUST NOT be marked as repeatable in API specifications.
    * The parameter MUST be marked as `"type": "string"` in API specifications in order to accommodate comma-separated values. Any other `type` value MUST NOT be used. The parameter description should indicate that comma separated values are accepted.
    * The query-parameter name SHOULD be plural, to provide a hint that this pattern is being employed.
    * The comma character (Unicode `U+002C`) SHOULD be used as the separator between values.
    * The API documentation MUST define how to escape the separator character, if necessary.


<h1 id="json-schema">JSON Schema</h1>

We would assume that [JSON Schema](http://json-schema.org/) is used to describe request/response models. Determine the version of JSON Schema to use for your APIs. At the time of writing this, [draft-04][9] is the latest version. JSON Schema [draft-03][2] has been deprecated, as support in tools is mostly focused on draft-04. The draft-04 is backwards incompatible with draft-03. 

<h2 id="api-contract-description">API Contract Description</h2>

There are various options available to define the API's contract interface (API specification or API description). Examples are: [OpenAPI (fka Swagger)][11], [Google Discovery Document](https://developers.google.com/discovery/v1/reference/apis#method_discovery_apis_getRest), [RAML](http://raml.org/), [API BluePrint](#https://apiblueprint.org/) and so on. 

[OpenAPI][11] is a vendor neutral API description format. The OpenAPI [Schema Object][12] (or OpenAPI JSON) is based on the [draft-04][9] and uses a predefined subset of the draft-04 schema. In addition, there are extensions provided by the  specification to allow for more complete documentation. 

We have used OpenAPI wherever we need to describe the API specification throughout this document.


<h2 id="schema">$schema</h2>

**A note about using $schema with OpenAPI**

As of writing this (Q12017), OpenAPI tools DO NOT recognize `$schema` value and (incorrectly) assume the value of $schema to be `http://swagger.io/v2/schema.json#` only. The following description applies to JSON schema (http://json-schema.org) used in API specification specified using specifications other than OpenAPI.


Use [$schema](http://json-schema.org/latest/json-schema-core.html#anchor22) to indicate the version of JSON schema used by each JSON type you define as shown below.

```
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "name": "Order",
    "description": "An order transaction.",
    "properties": {

    }
}

```
In case your JSON type uses `links`, `media` and other such keywords or schemas such as for `linkDescription` that are defined in [http://json-schema.org/draft-04/hyper-schema](http://json-schema.org/draft-04/hyper-schema), you should provide the value for `$schema` accordingly as shown below.

```
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-04/hyper-schema#",
    "name": "Linked-Order",
    "description": "An order transaction using HATEOAS links.",
    "properties": {

    }
}
```

If you are unsure about the specific schema version to refer to, it would be safe to refer `http://json-schema.org/draft-04/hyper-schema#` schema since it would cover all aspects of any JSON schema.




<h2 id="readOnly">readOnly</h2>

When resources contain immutable fields, `PUT`/`PATCH` operations can still be utilized to update that resource. To indicate immutable fields in the resource, the `readOnly` field can be specified on the immutable fields.

### Example

```
"properties": {
		"id": {
			"type": "string",
			"description": "Identifier of the resource.",
			"readOnly": true
		},
```


<h2 id="migration-from-draft03">Migration From draft-03</h2>

Key differences between JSON schema [draft-03][10] and draft-04 have been captured by JSON Schema contributors in a [changelog](https://github.com/json-schema/json-schema/wiki/ChangeLog). Additionally, this [StackOverflow thread](http://stackoverflow.com/questions/17205260/json-schema-draft4-vs-json-schema-draft3) provides some additional quality migration guidance.

The following items are the most common migration issues API specifications will need to address:

<h4 id="required">required</h4>

draft-03 defines required fields in the composition of a field:

```
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-03/schema#",
    "name": "Order",
    "description": "An order transaction.",
    "properties": {
        "id": {
            "type": "string",
            "description": "Identifier of the order transaction."
        },
        "amount": {
            "required": true,
            "description": "Amount being collected.",
            "$ref": "v1/schema/json/draft-04/amount.json"
        }
    }
}
```

In draft-04, an array as a peer to properties is used to designate the required fields:

```
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "name": "Order",
    "description": "An order transaction.",
    "required": [
        "amount"
    ],
    "properties": {
        "id": {
            "type": "string",
            "description": "Identifier of the order transaction."
        },
        "amount": {
            "description": "Amount being collected.",
            "$ref": "v1/schema/json/draft-04/amount.json"
        }
    }
}
```

<h4 id="readOnly">readOnly</h4>

The `readonly` field was removed from draft-03, and replaced by [`readOnly`](http://json-schema.org/latest/json-schema-hypermedia.html#anchor15) (note the upper case `O`).

#### Format

The [format](http://tools.ietf.org/html/draft-zyp-json-schema-03#section-5.23) attribute is used when defining fields which fall into a certain predefined pattern.

These following formats are deprecated in draft-04:

* `date`
* `time`

Therefore, only `"format": "date-time"` MUST be used for any variation of date, time, or date and time. Any references to formats of `date` or `time` should be updated to `date-time`. 

Field descriptions SHOULD indicate specifics of whether date or time is accepted. `date-time` specifies that ISO-8601/RFC3339 dates are utilized, which includes date-only and time-only.

<h2 id="advanced-syntax-draft04">Advanced Syntax draft-04</h2>

Be aware that `anyOf`/`allOf`/`oneOf` syntax can cause issues with tooling, as code generators, documentation tools and validation of these keywords is often not implemented. 


### allOf

The `allOf` keyword MUST only be used for the purposes listed here.

#### Extend object

The `allOf` keyword in JSON Schema SHOULD be used for extending objects. In draft-03, this was implemented with the `extends` keyword, which has been deprecated in draft-04. 

##### Example
A common need is to extend a common type with additional fields. In this example, we will extend the [address](v1/schema/json/draft-04/address_portable.json) with a `type` field.

```
"shipping_address": { "$ref": "v1/schema/json/draft-04/address_portable.json" }
```

Using the `allOf` keyword, we can combine both the common type address schema and an extra schema snippet for the address type:

```
"shipping_address": {
  "allOf": [
    // Here, we include our "core" address schema...
    { "$ref": "v1/schema/json/draft-04/address_portable.json" },

    // ...and then extend it with stuff specific to a shipping
    // address
    { "properties": {
        "type": { "enum": [ "residential", "business" ] }
      },
      "required": ["type"]
    }
  ]
}
```

### anyOf/oneOf

The `anyOf` and `oneOf` keywords SHOULD NOT be used to design APIs. A variety of problems occur from these keywords:

* Codegen tools do not have accurate way to generate models/objects from these definitions.
* Developer portals would have significant difficulty in clearly explaining to API consumers the meaning of these relationships.
* Consumers using statically typed languages (e.g. C#, Java) have a more complex experience trying to conditionally deserialize objects which change based on some element.
  * Custom deserialization code is required to represent objects based on the response, standard libraries do not accommodate this out of the box.
  * Flat structures which combine all possible fields in an object are automatically deserialized properly.
  
##### anyOf/oneOf Problems: Example

```
{
    "activity_type": {
        "description": "The entity type of the item. One of 'PAYMENT', 'MONEY-REQUEST', 'RECURRING-PAYMENT-PROFILE', 'ORDER', 'PAYOUT', 'SUBSCRIPTION', 'INVOICE'",
        "type": "string",
        "enum": [
            "PAYMENT",
            "MONEY-REQUEST"
        ]
    },
    "extensions": {
        "type": "object",
        "description": "Extension to core activity fields",
        "oneOf": [
            {
                "$ref": "extended_properties.json#/definitions/payment_properties"
            },
            {
                "$ref": "extended_properties.json#/definitions/money_request_properties"
            }
        ]
    }
}
```

In order for an API consumer to deserialize this response (where POJO/POCO objects are used), standard mechanisms would not work. Because the `extensions` field can change on any given response, the consumer is forced to create a composite object to represent both `payment_properties.json` and `money_request_properties.json`. 

A better approach:

```
{
    "activity_type": {
        "description": "The entity type of the item. One of 'PAYMENT', 'MONEY-REQUEST', 'RECURRING-PAYMENT-PROFILE', 'ORDER', 'PAYOUT', 'SUBSCRIPTION', 'INVOICE'",
        "type": "string",
        "enum": [
            "PAYMENT",
            "MONEY-REQUEST"
        ]
    },
    "payment": {
        "type": "object",
        "description": "Payment-specific activity.",
        "$ref": "payment.json"
    },
    "money_request": {
        "type": "object",
        "description": "Money request-specific activity.",
        "$ref": "money_request.json"    
    }
}
```

In this scenario, both `payment` and `money_request` are in the definition. However, in practice, only one field would be serialized based on the `activity_type`. 

For API consumers, this is a very predictable response, and allows for easy deserialization through standard libraries, without writing custom deserializers.

<h1 id="json-types">JSON Types</h1>

This section provides guidelines related to usage of [JSON primitive types](#json-primitive) as well as [commonly useful JSON types](#common-types) for address, name, currency, money, country, phone, among other things.

<h2 id="json-primitive">JSON Primitive Types</h2>

JSON Schema [draft-04][9] SHOULD be used to define all fields in APIs. As such, the following notes about the JSON Schema primitive types SHOULD be respected. Following are the guidelines governing use of JSON primitive type representations.

<h3 id="string">String</h3>

At a minimum, `strings` SHOULD always explicitly define a `minLength` and `maxLength`.

There are several reasons for doing so. 

1. Without a maximum length, it is impossible to reliably define a database column to store a given string. 
2. Without a maximum and minimum, it is also impossible to predict whether a change in length will break backwards-compatibility with existing clients.
3. Finally, without a minimum length, it is often possible for clients to send an empty string when they should not be allowed to do so.

APIs MAY avoid defining `minLength` and `maxLength` only if the string value is from another upstream system that has refused to provide any information on these values. This decision must be documented in the schema.

API authors SHOULD consider practical limitations when defining `maxLength`. For example, when using the VARCHAR2 type, modern versions of Oracle can safely store a Unicode string of no more than 1,000 characters. (The size limit is [4,000](https://docs.oracle.com/cd/B28359_01/server.111/b28320/limits001.htm) bytes and each Unicode character may take up to four bytes for storage).

`string` SHOULD utilize the `pattern` property as appropriate, especially when defining enumerated values or numbers. However, it is RECOMMENDED not to overly constrain fields without a valid technical reason.


<h3 id="enum">Enumeration</h3>

The JSON Schema `enum` keyword is difficult to use safely. It is not possible to add new values to an `enum` in a schema that describes a service response without breaking backwards compatibility. In that scenario, clients will often reject responses with values that are not in the older copy of the schema that they posess. This is usually not the desired behavior. Clients should usually handle unknown values more gracefully, but since you can't control nor verify their behavior, it is not safe to add new enum values.

For the reasons stated above, the schema author MUST comply with the following guidelines while using an `enum` with the JSON type `string`.

* The keyword `enum` SHOULD be used only when the set of values are fixed and would never change in future.
* If you anticipate adding new values to the `enum` array in future, avoid using the keyword `enum`. You SHOULD instead use a `string` type and document all acceptable values for the `string`. When using a `string` type to express enumerated values, you SHOULD enforce [naming conventions](#enum-names) through a `pattern` field. 
* If there is no technical reason to do otherwise -- for instance, a pre-existing database column of smaller size -- `maxLength` should be set to 255. `minLength` should be set to 1 to prevent clients sending the empty string.
* All possible values of an `enum` field SHOULD be precisely defined in the documentation. If there is not enough space in the `description` field to do so, you SHOULD use the API’s user guide to define them. 

Given below is the JSON snippet for enforcing naming conventions and length constraints.

```
{
    "type": "string",
    "minLength": 1,
    "maxLength": 255,
    "pattern": "^[0-9A-Z_]+$",
    "description": "A description of the field. The possible values are OPTION_ONE and OPTION_TWO."
}

```

<h3 id="number">Number</h3>

There are a number of difficulties associated with `number` type in JSON.

JSON itself defines a `number` very simply: it is an unbounded, fixed-point value. This is illustrated well by the railroad diagram for `number` at [JSON][30]. There is only one `number` type in JSON; there is no separate `integer` type.

JSON Schema diverges from JSON and defines two number types: `number` and `integer`. This is purely a convenience for schema validation; the JSON number type is used to implement both. Just as in JSON, both types are unbounded unless the schema author provides explicit `minimum` and `maximum` values.

Many programming languages do not safely handle unbounded values in JSON. JavaScript is an excellent example. A JSON deserializer is provided as part of the [ECMAScript](http://www.ecma-international.org/publications/standards/Ecma-262.htm) specification. However, it requires that all JSON numbers are deserialized into the only `number` type supported by JavaScript -- 64-bit floating point. This means that attempting to deserialize any JSON number larger than about 2^53 in a JavaScript client will result in an exception.

To ensure the greatest degree of cross-client compatibility possible, schema authors SHOULD:

* Never use the JSON Schema `number` type. Some languages may interpret it as a fixed-point value, and some as floating-point. Always use `string` to represent a decimal value. 

* Only define `integer` types for values that can be represented in a 32-bit signed integer, that is to say, values between (`(2^31) - 1) and -(2^31`). This ensures compatibility across a wide range of programming languages and circumstances. For example, array indices in JavaScript are signed 32-bit integers.

* When using an `integer` type, always provide an explicit minimum and a maximum. This not only allows backwards-incompatible changes to be detected, it also guarantees that all values can fit in a 32-bit signed integer. If there are no technical reasons to do otherwise, the `maximum` and `minimum` should be defined as `2147483647 (((2^31) - 1))` and `-2147483648 (-(2^31))` or `0` respectively. Common sense should be used to determine whether to allow negative values. Business logic that could change in the future generally SHOULD NOT be used to determine boundaries; business rules can easily change and break backwards compatibility.

If there is any possibility that the value could not be represented by a signed 32-bit integer, now or in the future, not use the JSON Schema `integer` type. Use a `string` instead.

**Examples**

This `integer` type might be used to define an array index or page count, or perhaps the number of months an account has been open.

```
{
    "type": "integer",
    "minimum": 0,
    "maximum": 2147483647
}
```

When using a `string` type to represent a number, authors MUST provide boundaries on size using `minLength` and `maxLength`, and constrain the definition of the string to only represent numbers using `pattern`.

For example, this definition only allows positive integers and zero, with a maximum value of 999999:

```
{
    "type": "string",
    "pattern": "^[0-9]+$",
    "minLength": 1,
    "maxLength": 6
}
```

The following definition allows the representation of fixed-point decimal values both positive or negative, with a maximum length of 32 and no requirements on scale:

```
{
    "type": "string",
    "pattern": "^(-?[0-9]+|-?([0-9]+)?[.][0-9]+)$"
    "maxLength": 32,
    "minLength": 1,
}
```

<h3 id="array">Array</h3>

JSON defines `array` as unbounded.

Although practical limits are often much lower due to memory constraints, many programming languages do place maximum theoretical limits on the size of arrays. For example, JavaScript is limited to the length of a 32-bit unsigned integer by the ECMA-262 specification. Java is [limited to about `Integer.MAX_VALUE - 8`](https://stackoverflow.com/questions/3038392/do-java-arrays-have-a-maximum-size#3039805), which is less than half of JavaScript.

To ensure maximum compatibility across languages and encourage paginated APIs, `maxItems` SHOULD always be defined by schema authors. `maxItems` SHOULD NOT have a value greater than can be represented by a 16-bit signed integer, in other words, `32767` or `(2^15) - 1)`.

Note that developers MAY choose to set a smaller value; the value `32767` is a default choice to be used when no better choice is available. However, developers SHOULD design their API for growth. For example, although a paginated API may only support a maximum of 100 results per page today, performance improvements may allow deveopers to improve that to 1,000 results next year. Therefore, `maxItems` SHOULD NOT be used to communicate maximum page size.

`minItems` SHOULD also be defined. In most situations, its value will be either `0` or `1`.

<h3 id="null">Null</h3>

APIs MUST NOT produce or consume `null` values.

`null` is a primitive type in JSON. When validating a JSON document against JSON Schema, a property's value can be `null` only when it is explicitly allowed by the schema, using the `type` keyword (e.g. `{"type": "null"}`). Since in an API `type` will always need to be defined to some other value such as `object` or `string`, and these standards prohibit using schema composition keywords such as `anyOf` or `oneOf` that allow multiple types, if an API produces or consumes null values, it is unlikely that, according to the API's own schemas, this is actually valid data.

In addition, in JSON, a property that doesn't exist or is missing in the object is considered to be undefined; this is conceptually separate from a property that is defined with a value of `null`, but many programming languages have difficulty drawing this distinction.

For example, a property `my_property` defined as `{"type": "null"}` is represented as 

```
{
    "my_property": null
}
```

While a property that is `undefined` would not be present in the object:

```
{ }
```

In most strongly typed languages, such as Java, there is no concept of an `undefined` type, which means that for all undefined fields in a JSON object, a deserializer would return the value of such types as `null` when you try to retrieve them. Similarly, some Java-based JSON serializers serialize fields to JSON `null` by default, even though it is not possible for the serializer to determine whether the author of the Java object intended for that property to be defined with a value of `null`, or simply undefined. In Jackson, for example, this behavior is moderated through use of the [`JsonInclude`](https://fasterxml.github.io/jackson-annotations/javadoc/2.8/com/fasterxml/jackson/annotation/JsonInclude.html) annotation. On the other hand, the org.json library defines an object called [NULL](https://stleary.github.io/JSON-java/org/json/JSONObject.html#NULL) to distinguish between `null` and `undefined`.

Eschewing JSON null completely helps avoid many of these subtle cross-language compatibility traps.

<h3 id="additional-properties">Additional Properties</h3>

Setting of [`additionalProperties`](http://json-schema.org/latest/json-schema-validation.html#anchor134) to `false` in schema objects breaks backward compatibility in those clients that use an API's JSON schemas (defined by its contract) to validate the API requests and responses. For the same reason, the schema author MUST not explicitly set the `additionalProperties` to `false`.

The API implementation SHOULD instead enforce the conformance of requests and responses to an API contract by hard validating the requests and responses against the defined API contract at run-time.

<h2 id="common-types">Common Types</h2>

Resource representations in API MUST reuse the [common data type][13] definitions where possible. Following sections provide some details about some of these common types. Please refer to the [`README`][14]and the [schema][13] for more details.

<h3 id="address">Address</h3>


We recommend using [`address_portable.json`][19] for all requirements related to address. The `address_portable.json` is

* backward compatible with [hcard](http://microformats.org/wiki/hcard) address microformats,
* forward compatible with Google open-source address validation metadata ([i18n-api][10]) and W3 HTML5.1 [autofill][11] fields,
* allows mapping to and from many address normalization services (ANS) such as [AddressDoctor][16].

Please refer to [README for Address][12] for more details about the address type, guidance on how to map it to i18n-api's address and W3 HTML5.1's autofill fields. 


<h3 id="money">Money</h3>

Money is a standard type to represent amounts. The Common Type [`money.json`][15] provides common definition of money.

**Data-type integrity rules:**

* Both `currency_code` and `value` MUST exist for this type to be valid.
* Some currencies such as "JPY" do not have sub-currency, which means the decimal portion of the value should be ".0".
* An amount MUST NOT be negative. For example a $5 bill is never negative. Negative or positive is an internal notion in association with a particular account/transaction and in respect of the type of the transaction performed.

<h3 id="apr">Percentage, Interest rate, or APR</h3>

Percentages and interest rates are very common when dealing with money. One of the most common examples is annual percentage rate, or APR. These interest rates SHOULD always be represented according to the following rules:

* The  Common Type [`percentage.json`](v1/schema/json/draft-04/percentage.json) MUST be used. This ensures that the rate is represented as a fixed-point decimal.
    * All validation rules defined in the type MUST be followed.
* The value MUST be represented as a percentage.
    * **Example:** if the interest rate is 19.99%, the value returned by the API MUST be `19.99`.
* The field's JSON schema description field SHOULD inform clients how the representation works.
    * **Example:** "The interest rate is represented as a percentage. For example, an interest rate of 19.99% would be serialized as 19.99."
* It is the responsibility of the client to transform this value into a format suitable for display to the end-user. For example, some countries use the comma ( , ) as a decimal separator instead of the period ( . ). Services MUST NOT vary the format of values passed to or from a service based on end-user display concerns.

<h3 id="internationalization">Internationalization</h3>

The following common types MUST be used with regard to global country, currency, language and locale.

* [`country_code`](v1/schema/json/draft-04/country_code.json)
  * All APIs and services MUST use the [ISO 3166-1 alpha-2](http://www.iso.org/iso/country_codes.htm) two letter country code standard.
* [`currency_code`](v1/schema/json/draft-04/currency_code.json)
  * Currency type MUST use the three letter currency code as defined in [ISO 4217](http://www.currency-iso.org/). For quick reference on currency codes, see [http://en.wikipedia.org/wiki/ISO_4217](http://en.wikipedia.org/wiki/ISO_4217).
* [`language.json`](v1/schema/json/draft-04/language.json)
 	* Language type uses [BCP-47](https://tools.ietf.org/html/bcp47) language tag.
* [`locale.json`](v1/schema/json/draft-04/locale.json)
 	* Locale type defines the concept of locale, which is composed of `country_code` and `language`. Optionally, IANA timezone can be included to further define the locale.
* [`province.json`](v1/schema/json/draft-04/province.json) 
    * Province type provides detailed definition of province or state, based on [ISO-3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) country subdivisions, with room for variant local, international, and abbreviated representations of province names. Useful for logistics, statistics, and building state pull-downs for on-boarding.


<h3 id="date-time">Date, Time and Timezone</h3>

When dealing with date and time, all APIs MUST conform to the following guidelines.

* The date and time string MUST conform to the `date-time` universal format defined in section `5.6` of [RFC3339][21]. In use cases where you would require only a subset of the fields (e.g `full-date` or `full-time`) from the RFC3339 `date-time` format, you SHOULD use the [Date Time Common Types](#date-common-types) to express these.

* All APIs MUST only emit [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) time (aka [Zulu time](https://en.wikipedia.org/wiki/List_of_military_time_zones) or [GMT](https://en.wikipedia.org/wiki/Greenwich_Mean_Time)) in the responses.

* When processing requests, an API SHOULD accept `date-time` or time fields that contain an offset from UTC. For example, `2016-09-28T18:30:41.000+05:00` SHOULD be accepted as equivalent to `2016-09-28T13:30:41.000Z`. This helps ensure compatibility with third parties who may not be capable of normalizing values to UTC before sending requests. In such cases the offset SHOULD only be used to calculate the equivalent UTC time before it is persisted in the system (because of known platform/language/DB interoperability issues). A UTC offset MUST NOT be used to derive anything else. 

* If the business logic requires expressing the timezone of an event, it is RECOMMENDED that you capture the timezone explicitly by using a separate request/response field. You SHOULD NOT use offset to derive the timezone information. The offset alone is insufficient to accurately transform the stored UTC time back to a local time later. The reason is that a UTC offset might be same for many geographical regions and based on the time of the year there may be additional factors such as daylight savings. For example, an offset UTC-05:00 represents Eastern Standard Time during winter, Central Dayight Time during summer, and year-round offset for Panama, Columbia, and Peru. 
 
* The timezone string MUST be per [IANA timezone database](https://www.iana.org/time-zones) (aka **Olson** database or **tzdata** or **zoneinfo** database), for example *America/Los_Angeles* for Pacific Time, or *Europe/Berlin* for Central European Time.

* When expressing [floating](https://www.w3.org/International/wiki/FloatingTime) time values that are not tied to specific time zones such as user's date of birth, expiry date, publication date etc. in requests or responses, an API SHOULD NOT associate it with a timezone. The reason is that a UTC offset changes the meaning of a floating time value. For examples, all countries with timezones west of prime meridian would consider a floating time value to be the previous day.

<h4 id="date-common-types">Date Time Common Types</h4>

The following common types MUST be used to express various date-time formats:

* [`date_time.json`](v1/schema/json/draft-04/date_time.json) SHOULD be used to express an RFC3339 `date-time`.
* [`date_no_time.json`](v1/schema/json/draft-04/date_no_time.json) SHOULD be used to express `full-date` from RFC 3339.
* [`time_nodate.json`](v1/schema/json/draft-04/time_nodate.json) SHOULD be used to express `full-time` from RFC3339.
* [`date_year_month.json`](v1/schema/json/draft-04/date_year_month.json) SHOULD be used to express a floating date that contains only the **month** and **year**. For example, card expiry date (`2016-09`).
* [`time_zone.json`](v1/schema/json/draft-04/time_zone.json) SHOULD be used for expressing timezone of a RFC3339 `date-time` or a `full-time` field.


<h1 id="error-handling">Error Handling</h1>

As per HTTP specifications, the outcome of a request execution could be specifiedusing an integer and a message. The number is known as the _status code_ and the message as the _reason phrase_. The reason phrase is a human readable message used to clarify the outcome of the response. The HTTP status codes in the `4xx` range indicate client-side errors (validation or logic errors), while those in the `5xx` range indicate server-side errors (usually defect or outage). However, these status codes and human readable reason phrase are not sufficient to convey enough information about an error in a machine-readable manner. To resolve an error, non-human consumers of RESTful APIs need additional help.

Therefore, APIs MUST return a JSON error representation that conforms to the [`error.json`][22] schema defined in the [Common Types][13] repository. It is recommended that the namespace that an API belongs to has an error catalog associated with it. Please refer to [Error Catalog](#error-catalog) for more details.    


<h2 id="error-schema">Error Schema</h2>


An error response following `error.json` as schema MUST include the following fields:

* `name`: A human-readable, unique name for the error. It is recommended that this value would be retrieved from the error catalog [`error_spec.json#name`][24] before sending the error response.
* `details`: An array that contains individual instance(s) of the error with specifics such as the following. This field is required for client side errors (`4xx`).
	* `field`: [JSON Pointer][23] to the field in error if in body, else name of the path parameter or query parameter.
	* `value`: Value of the field in error.
	* `issue`: Reason for error. It is recommended that this value would be retrieved from the error catalog [`error_spec_issue.json#issue`][25] before sending the error response.
	* `location`: The location of the field in the error, either `query`, `path`, or `body`. If this field is not present, the default value is `body`.
* `debug_id`: A unique error identifier generated on the server-side and logged for correlation purposes.
* `message`: A human-readable message, describing the error. This message MUST be a description of the problem NOT a suggestion about how to fix it. It is recommended that this value would be retrieved from the error catalog [`error_spec.json#message`][24] before sending the error response.
* `links`: [HATEOAS](hypermedia.md) links specific to an error scenario. Use these links to provide more information about the error scenario and how to resolve it. You could insert links from [`error_spec.json#suggested_application_actions`][24] and/or [`error_spec.json#suggested_user_actions`][24] here as well as other HATEOAS links relevant to the API. 


The following fields are optional:

* `information_link`: (`deprecated`) A URI for expanded developer information related to this error; this SHOULD be a link to the publicly available documentation for the type of error. Use `links` instead.

##### Use of JSON Pointer

If you have used some other means to identify the `field` in an already released API, you could continue using your existing approach. However, if you plan to migrate to the approach suggested, you would want to bump up the major version of your API and provide migration assistance to your clients as this could be a potential breaking change for them. 

The JSON Pointer for the `field` SHOULD be a [JSON string value](https://tools.ietf.org/html/rfc6901#section-5).


<h3 id="input-errors">Input Validation Errors</h3>

In validating requests, there are a variety of concerns that should be addressed in the following order:

|Request Validation Issue | HTTP status code|
|------------- | -------------|
|Not well-formed JSON. | `400 Bad Request`|
|Contains validation errors that the client can change. | `400 Bad Request`|
|Cannot be executed due to factors outside of the request body. The request was well-formed but was unable to be followed due to semantic errors. | `422 Unprocessable Entity`|

<h2 id="error-samples">Error Samples</h2>

This section provides some samples to describe usage of [`error.json`][22] in various scenarios. 

<h4 id="sampleresponse-invalid">Validation Error Response - Single Field</h4>

The following sample shows a validation error of type `VALIDATION_ERROR` in one field. Because this is a client error, a `400 Bad Request` HTTP status code should be returned.

```

{  
   "name":"VALIDATION_ERROR",
   "details":[  
      {  
         "field":"#/credit_card/expire_month",
         "issue":"Required field is missing",
         "location":"body"
      }
   ],
   "debug_id":"123456789",
   "message":"Invalid data provided",
   "information_link":"http://developer.foo.com/apidoc/blah#VALIDATION_ERROR"
}
```

<h4 id="sampleresponse-multi">Validation Error Response - Multiple Fields</h4>

The following sample shows a validation error of the same  type, `VALIDATION_ERROR`, in two fields. Note that `details` is an array listing all the instances in the error. Because both these are a client errors, a `400 Bad Request` HTTP status code should be returned.

``` 

{  
   "name": "VALIDATION_ERROR",
   "details": [  
      {  
         "field": "/credit_card/expire_month",
         "issue": "Required field is missing",
         "location": "body"
      },
      {  
         "field": "/credit_card/currency",
         "value": "XYZ",
         "issue": "Currency code is invalid",
         "location": "body"
      }
   ],
   "debug_id": "123456789",
   "message": "Invalid data provided",
   "information_link": "http://developer.foo.com/apidoc/blah#VALIDATION_ERROR"
}

```

<h4 id="sampleresponse-bulk">Validation Error Response - Bulk </h4>

For heterogenous types of client-side errors shown below, `OBJECT_NOT_FOUND_ERROR` and `MULTIPLE_CORE_BUNDLES`, an array named `errors` is returned. Each error instance is represented as an item in this array. Because these are client validation errors, a `400 Bad Request` HTTP status code should be returned.

``` 
 
   "errors": [  
      {  
         "name": "OBJECT_NOT_FOUND_ERROR",
         "debug_id": "38cdd677a83a4",
         "message": "Bundle is not found.",
         "information_link": "<link to public doc describing OBJECT_NOT_FOUND_ERROR error>",
         "details": [  
            {  
               "field": "/bundles/0/bundle_id",
               "value": "33333",
               "issue": "BUNDLE_NOT_FOUND",
               "location": "body"
            }
         ]
      },
      {  
         "name": "MULTIPLE_CORE_BUNDLES",
         "debug_id": "52cde38284sd3",
         "message": "Multiple CORE bundles.",
         "information_link": "<link to public doc describing MULTIPLE_CORE_BUNDLES error>",
         "details": [  
            {  
               "field": "/bundles/5/bundle_id",
               "value": "88888",
               "issue": "MULTIPLE_CORE_BUNDLES",
               "location": "body"
            }
         ]
      }
   ]
```

<h4 id="sampleresponse-422">Semantic Validation Error Response </h4>

In cases where client input is well-formed and valid but the request action may require interaction with APIs or processes outside of this URI, an HTTP status code `422 Unprocessable Entity` should be returned.

```
{  
   "name": "BALANCE_ERROR",
   "debug_id": "123456789",
   "message": "The account balance is too low. Add balance to your account to proceed.",
   "information_link": "http://developer.foo.com/apidoc/blah#BALANCE_ERROR"
}
```
    
<h2 id="error-declaration">Error Declaration In API Specification</h2>

It is important that documentation generation tools and client/server-side binding generation tools recognize [`error.json`][22]. Following section shows how you could refer `error.json` in an API specification confirming to OpenAPI. 


``` 
"responses": {
          "200": {
            "description": "Address successfully found and returned.",
            "schema": {
              "$ref": "address.json"
            }
          },
          "403": {
            "description": "Unauthorized request.  This error will occur if the SecurityContext header is not provided or does not include a party_id."
          },
          "404": {
            "description": "The requested address does not exist.",
            "schema": {
              "$ref": "v1/schema/json/draft-04/error.json"
            }
          },
          "default": {
            "description": "Unexpected error response.",
            "schema": {
              "$ref": "v1/schema/json/draft-04/error.json"
            }
          }
        }
   
```



<h2 id="userguide-errors">Samples with Error Scenarios in Documentation</h2>

The User Guide of an API is a document that is exposed to API consumers. In addition to linking to samples showing successful execution for invocation on various methods exposed by the API, the API developer should also provide links to samples showing error scenarios. It is equally, or perhaps more, important to show the API consumers how an API would propagate errors in a machine-readable form in order to build applications that take necessary actions to handle errors gracefully and in a meaningful manner.

In conclusion, we reiterate the message we started with that non-human consumers of RESTful APIs need more help to take necessary actions to resolve an error in a machine-readable manner. Therefore, a representation of errors following the [schema](#error-schema) described here MUST be returned by APIs for any HTTP status code that falls into the ranges of 4xx and 5xx.
 

<h2 id="error-catalog">Error Catalog</h2>

Error handling [guidelines](#error-handling) described earlier show how to provide error related details responses at runtime. This section explains how to catalog the errors so they can be easily consumed in service runtime and for generating documentation. 

An Error Catalog is a single JSON file that contains a collection of error specifications (or error metadata) for a namespace. Each error specification includes error name, error message, issue details and related links among other things. The error catalog supports multiple locales or languages. For a specific error catalog, there should be exactly one default version, known as the top-level catalog, which could be in English for example. There should be corresponding locale-specific catalogs, one for each additional supported locale, as needed.

<h3 id="reasons-to-catalog-errors">Reasons To Catalog Errors</h3>

Following are some reasons to catalog the errors for an API:

1. To _externalize_ hard-coded error message strings from the API implementation: Developers are good at writing code but not necessarily good at writing error messages. Often error messages written by a developer make a lot of assumptions about the context and audience. It is hard to change such error messages if these are embedded in implementation code. For example, to change a message from "Add Card refused due to compliance guidelines" to `Could not add card due to failure to comply with guideline %s`, a service developer has to make the change and also redeploy the service emitting that error.    
2. To _localize_ the error strings: If error related strings such as `message`, `issue`, `actions`, among other things in `error.json` are externalized, it is easy for the documentation and an internationalization team to modify and localize these without any help from service developers and without requiring redeployment of the services.
3. To keep service's implementation and the API documentation in _sync_ with regards to errors: API consumers should be able to refer with confidencethe API's documentation for errors generated by the services at runtime. This helps to reduce the cost and the time spent supporting an API and increases `adoptability` of the API.


<h3 id="error-catalog-schema">Error Catalog Schema</h3>

There are four main JSON schema files for the Error Catalog.

1. [`error_catalog.json`][26] defines top-level catalog container. 
	* `namespace`: API namespace
	* `language`: language used to catalog the errors. Default is US English. This value MUST be a [BCP-47](https://tools.ietf.org/html/bcp47) language tag as in `en` or `en-US`.
	* `errors`: one or more error catalog items.
2. [`error_catalog_item.json`][27] defines an item in error catalog. This schema in its initial version only includes error specification `error_spec`. In future versions, it would provide a space to establish relationship between an error specification and method specifications that would use the error specification to respond with error.
3. [`error_spec.json`][24] is where the core of error specification is defined. The specification includes the following properties.
	* `name`: A human-readable, unique name for the error. This value MUST be the value set in [`error.json#name`][22] before sending the error response.
	* `message`: A human-readable message, describing the error. This message MUST be a description of the problem NOT a suggestion about how to fix it. This value MUST be the value set in [`error.json#message`][22] before sending the error response. This value could be localized. 
	* `log_level`: Log level associated with this error. This MUST NOT be streamed out in error responses or exposed in any external documentation.
	* `legacy_code`: Legacy error code. Use if and only if the existing and published error metadata uses the code and it must continue to be supported. Utilize `additionalProperties` of [`error.json`][22] to send this code in the error response.   
	* `http_status_codes`: Applicable HTTP status codes for this error.
	* `suggested_application_actions`: Suggest practical actions that the developer of application consuming the API could take in order to resolve the error condition. These MUST be in English.
	* `suggested_user_actions`: Suggest practical actions that a user of the application consuming the API could take in order to resolve the error condition. These MUST be in the language used `error_catalog.json#language`.
	* `links`: Error context specific [HATEOAS](#hypermedia) links. Corresponds to [`error.json#links`][22].
	* `issues`: Issues associated with this error as defined in `error_spec_issue.json`. Each issue corresponds to an item in [`error_details.json`][28]. 
* [`error_spec_issue.json`][25] defines details related to the error. For example, there could be multiple validation errors triggering `400 BAD REQUEST`. Each invalid field MUST be listed in the [`error_details.json`][28] while sending the error response. 
	* `id`: Catalog-unique identifier of the issue. This is required in order to search for the `error_spec_issue` from cached error catalog.
	* `issue`: Reason for error. This value MUST be the value set in [`error_details.json#issue`][28]. The issue string could have variables. Please use parametrized string following the Java String Formatter syntax. This value chould be localized.  

<h3 id="string-format">String Format</h3>

For API services that are implemented in Java, various strings found in the error catalog MUST be formatted using Java's `printf-style` inspired format. It's recommended to use Java's [format specification](https://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html#summary) for values of `message` and `issue` fields in the error catalog where applicable.
 
Service developers are strongly encouraged to use tools, such as [Java String Formatter](https://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html) or similar, to interpret the formatted strings as found in the error catalog. 

For example, an `error_spec` having value `Could not add card due to failure to comply with guideline %s` for `message` must be interpreted using a formatter as shown below. 

```
com.foo.platform.error.ErrorSpec errorSpec = <find errorSpec from catalog>

com.foo.platform.error.Error error = new Error();
error.setName(errorSpec.getName());
error.setDebugId("debugId-777");
error.setLogLevel(errorSpec.getLogLevel());
String errorMessageString = String.format(errorSpec.getMessage(), "GUIDELINE: XYZ");
error.setMessage(errorMessageString);

List<Detail> details = new ArrayList<>();
Detail detail = new Detail();
String issueString = String.format(errorSpec.getIssues().get(0).getIssue(), ... <variables in issue string>);
detail.setIssue(issueString);
details.add(detail);
error.setDetails(details);

Response response = Response.status(Response.Status.BAD_REQUEST).entity(error).encoding(MediaType.APPLICATION_JSON).build();
throw new WebApplicationException(response);
```

The above example code is for illustration purposes only. 


<h3 id="samples">Samples</h3>

This section provides some sample error catalogs.

#### Sample catalog : namespace : payments

```
{
	"namespace": "payments",
	"language": "en-US",
	"errors": [{
		"error_spec": {
			"name": "VALIDATION_ERROR",
			"message": "Invalid request - see details",
			"log_level": "ERROR",
			"http_status_codes": [
				400
			],
			"issues": [{
				"id": "InvalidCreditCardType",
				"issue": "Value is invalid (must be visa, mastercard, amex, or discover)"
			}],
			"suggested_application_actions": [
				"Provide an acceptable card type and resend the request."
			]
		}
	}, {
		"error_spec": {
			"name": "PAYEE_ACCOUNT_LOCKED_OR_CLOSED",
			"message": "Payee account is locked or closed",
			"log_level": "ERROR",
			"http_status_codes": [
				422
			],
			"legacy_code": "PAYER_ACCOUNT_LOCKED_OR_CLOSED",
			"issues": [{
				"id": "PayerAccountLocked",
				"issue": "The account receiving this payment is locked or closed and cannot receive payments."
			}],
			"suggested_user_actions": [
				"Contact Customer Service at contact@foo.com"
			]
		}
	}]
}
```

#### Sample catalog : namespace : wallet

```
{
	"namespace": "wallet",
	"language": "en-US",
	"errors": [{
		"error_spec": {
			"name": "INVALID_ISSUER_DETAILS",
			"message": "Invalid issuer details",
			"log_level": "ERROR",
			"http_status_codes": [
				400
			],
			"issues": [{
				"id": "ISSUER_DATA_NOT_FOUND",
				"issue": "Issuer data not found"
			}],
			"suggested_application_actions": [
				"Provide issuer related data and resend the request."
			]
		}
	}, {
		"error_spec": {
			"name": "INSTRUMENT_BLOCKED",
			"message": "Instrument is currently blocked.",
			"log_level": "ERROR",
			"http_status_codes": [
				422
			],
			"issues": [{
				"id": "BankAccountBlocked",
				"issue": "Bank account is blocked due max random deposit retries. "
			}],
			"suggested_user_actions": [
				"Contact Customer Service at contact@foo.com."
			]
		}
	}]
}
```

#### Sample catalog : namespace : payment-networks

```
{
	"namespace": "payment-networks",
	"language": "en-US",
	"errors": [{
		"error_spec": {
			"name": "VENDOR_TIMEOUT",
			"message": "Transaction timed out while waiting for response from downstream service provided by a 3rd party vendor.",
			"log_level": "ERROR",
			"http_status_codes": [
				504
			],
			"suggested_application_actions": [
				"Retry again later."
			]
		}
	}, {
		"error_spec": {
			"name": "INTERNAL_TIMEOUT",
			"message": "Internal error due to timeout. Request took too long to process. The status of the transaction is unknown.",
			"log_level": "ERROR",
			"http_status_codes": [
				500
			],
			"suggested_application_actions": [
				"Contact Customer Service at contact@foo.com and provide Correlation-Id and debug_id for diagnosis along with other details."
			]
		}
	}]
}
```


<h1 id="api-versioning">API Versioning</h1>

This section describes how to version APIs. It describes API's lifecycle states, enumerates versioning policy, describes backwards compatiblity related guidelines and describes an End-Of-Life policy. 

<h2 id="api-lifecycle">API Lifecycle</h2>

Following is an example of states of an API's lifecycle.

|State | Description|
|---------|------------|
|PLANNED |API has been scheduled for development. API release plans have been established.|
|BETA|API is operational and is available to selected new subscribers in production for the purposes of testing, validating, and rolling out a new API.|
|LIVE | API is operational and is available to new subscribers in production. API version is fully supported.|
|DEPRECATED | API is operational and available at runtime to existing subscribers. API version is fully supported, including bug fixes addressed in backwards compatible way. API version is not available to new subscribers.|
|RETIRED | API is unpublished from production and no longer available at runtime to any subscribers. The footprint of all deployed applications entering this state must be completely removed from production and stage environments.|

<h2 id="api-versioning-policy">API Versioning Policy</h2>

API’s are versioned products and MUST adhere to the following versioning principles.

1. API specifications MUST follow the versioning scheme where where the `v` introduces the version, the major is an ordinal starting with `1` for the first LIVE release, and minor is an ordinal starting with `0` for the first minor release of any major release.
2. Every time there is an incremental change to an API, whether or not backward compatible, the API specification MUST be versioned. This allows the change to be labeled, documented, reviewed, discussed, published and communicated.
3. API endpoints MUST only reflect the major version.
4. API specification versions reflect interface changes and MAY be separate from service implementation versioning schemes.
5. A minor API version MUST maintain backward compatibility with all previous minor versions, within the same major version.
6. A major API version MAY maintain backward compatibility with a previous major version.
	
For a given functionality set, there MUST be only one API version in the `LIVE` state at any given time across all major and minor versions. This ensures that subscribers always understand which versioned API product they should be using. For example, v1.2 `RETIRED`, v1.3 `DEPRECATED`, or v2.0 `LIVE`.


<h2 id="backwards-compatibility">Backwards Compatibility</h2>

APIs SHOULD be designed in a forward and extensible way to maintain compatibility and avoid duplication of resources, functionality and excessive versioning.

APIs MUST adhere to the following principles to be considered backwards compatible:

1. All changes MUST be additive.
2. All changes MUST be optional.
3. Semantics MUST NOT change.
4. Query-parameters and request body parameters MUST be unordered.
5. Additional functionality for an existing resource MUST be implemented either:
    1. As an optional extension, or
    2. As an operation on a new child resource, or
    3. By altering a request body, while still accepting all the previous, existing request variations, if an existing operation (e.g., resource creation) cannot be reasonably extended.


<h3 id="backwards-incompatible-changes">Non-exhaustive List of Backwards Incompatible Changes</h3>

<b>URIs</b>

1. A resource URI MAY support additional query parameters but they CANNOT be mandatory.
2. There MUST NOT be any change in the behavior of the API for request URIs without the newly added query parameters.
3. A new parameter with a required constraint SHALL NOT be added to a request.
4. The semantics of an existing parameter, entire representation, or resource SHOULD NOT be changed.
5. A service MUST recognize a previously valid value for a parameter and SHOULD NOT throw an error when used.
6. There MUST NOT be any change in the HTTP status codes returned by the URIs.
7. There MUST NOT be any change in the HTTP verbs (e.g. `GET`, `POST`, `PUT` or `PATCH`) supported earlier by the URI. The URI MAY however support a new HTTP verb.
8. There MUST NOT be any change in the name and type of the request or response headers of an URI. Additional headers MAY be added, provided they’re optional.

<b>APIs only support media type `application/json`. The following applies for JSON representation stability.</b>

1. An existing property in a JSON object of an API response MUST continue to be returned with same name and JSON type (number, integer, string, array, object).
2. If the value of a response field is an array, then the type of the contents in the array MUST NOT change.
3. If the value of the response field is an object, then the compatibility policy MUST apply to the JSON object as a whole.
4. New properties MAY be added to a representation any time, but it SHOULD NOT alter the meaning of an existing property.
5. New properties that are added MUST NOT be mandatory.
6. Mandatory fields are guaranteed to be present in the response.
7. For primitive types, unless there is a constraint described in the API documentation (e.g. length of the string, possible values for an ENUM type), clients MUST not assume that the values are constrained in any way.
8. If the property of an object is a URI, then it MUST have the same stability mentioned as URIs.
9. For an API returning HATEOAS links as part of the representation, the values of rel and href MUST remain the same.
10. For ENUM types, there MUST NOT be any change in already supported enumerated values or meaning of these values.


<h2 id="eol-policy">End of Life Policy</h2>

The End-of-Life (EOL) policy regulates how API versions move from the `LIVE` to the `RETIRED` state. It is designed to ensure a consistent and reasonable transition period for API customers who need to migrate from the old to the new API version while enabling a healthy process to retire technical debt.

<b>Minor API Version EOL</b>

Per versioning policy, minor API versions MUST be backwards compatible with preceding minor versions within the same major version. Thus, minor API versions are `RETIRED` immediately after a newer minor version of the same major version becomes `LIVE`. This change should have no impact on existing subscribers so there is no need to transition through a `DEPRECATED` state to facilitate client migration.

<b>Major API Version EOL</b>

Per versioning policy, major API versions MAY be backwards compatible with preceding major versions. As such, the following rules apply when retiring a major API version.

1. A major API version MUST NOT be in the `DEPRECATED` state until a replacement service is `LIVE` that provides a clear customer migration path for all functionality that will be retained moving forward. This SHOULD include documentation and, as appropriate, migration tools and sample code that provide customers what they need to make a clean migration.
2. The deprecated API version MUST be in the `DEPRECATED` state for a minimum period of time to give client customers adequate notice to migrate. Deprecation of API versions with external clients SHOULD be considered on a case-by-case basis and may require additional deprecation time and/or constraints to minimize impact to the business.
6. If a versioned API in `LIVE` or `DEPRECATED` state has no clients, it MAY move to the `RETIRED` state immediately.

<h3 id="eol-policy-replacement">End of Life Policy – Replacement Major API Version Introduction</h3>

Since a new major API version that results in deprecation of a pre-existing API version is a significant business investment decision, API owners MUST justify the new major version before beginning significant design and development work.  API owners SHOULD explore all possible alternatives to introducing a new major API version with the objective of minimizing the impact on customers before deciding to introduce a new version. Justification SHOULD include the following:

Business Case

1. Customer value delivered by new version that is not possible with existing version.
2. Cost-benefit analysis of deprecated version versus the new version.
3. Explanation of alternatives to introducing an new major version and why those were not chosen.
4. If a backwards incompatible change is required to address a critical security issue, items 1 and 2 (above) are not required.

API Design

1. A domain model of all resources in the new API version and how they compare to the domain model of the previous major API version.
2. Description of APIs operations and use cases they implement.
3. Definition of service level objectives for performance and availability that are equal or better to the major API version to be deprecated.

Migration Strategy

1. Number of existing customers impacted; internal, external, and partners.
2. Communication and support plan to inform existing customers of new version, value, and migration path.




<h1 id="deprecation">Deprecation</h1>

This document describes a solution to deprecate parts of an API as the API evolves. It is an extension to the API Versioning Policy.


<h3 id="deprecation-terms-used">Terms Used</h3>

The term *`API Element`* is used throughout this document to refer to the *things* that could be deprecated in an API. Examples of an `API Element` are: an endpoint, a query parameter, a path parameter, a property within a JSON Object schema, JSON Object schema of a type or a custom HTTP header among other things. 

The term *`old API`* is used to indicate existing minor or major version of your API or an existing different API that your API supersedes. 

The term *`new API`* is used to indicate a new minor or major version of your API or a new different API that supersedes the `old API`. 

*`API definition`* is in the form of specification of an interface of a service following the [OpenAPI][11]. API's definition could be found in `swagger.json`.

<h2 id="deprecation-background">Background</h2>

When defining your API, you must make a lot of material decisions that have long lasting implications. The objective is to make a long-lived, durable, and reusable API. You are trying to get it "right". Practically speaking, however, you are not going to succeed every time. New requirements come in. Your understanding of the problem changes. What probably looked like a good decision at the time, may now limit your ability to elegantly extend your API to satisfy your new understanding. Lightweight decisions made in the past now seem somewhat heavy as the implications come into focus. Maintaining backward compatibility is a persistent challenge.

One option is to create a new major version of your API. This allows you to leave past decisions behind and start fresh. Unfortunately, it also means that all of your clients now need to migrate to new endpoints for any of the new work to deliver customer value. This is hard. Many clients will not move without good incentives. There is a lot of overhead in managing the customer migration. You also need to support two sets of interfaces for quite some time. The other consideration is that your API product may have multiple endpoints, but the breaking changes that you want to make only affect one. Making your customers migrate their applications for all the API endpoints just so you can “fix” one small part is a pretty heavyweight and expensive change. While pure and simple from a philosophical and engineering point of view, it is often unjustifiable from an ROI standpoint. The goal of this guideline is to find a middle ground that provides a more practical path forward when minor changes are needed, but which is still consistent, in spirit, with the [API Versioning Policy](#api-versioning-policy).

<h2 id="deprecation-requirements">Requirements</h2>

Here are the requirements for deprecation.

1. An API developer should be able to deprecate an `API Element` in a minor version of an API. 
2. An API specification MUST highlight one or more deprecated elements of the API so the API consumers are aware.
3. An API server MUST inform client app(s) regarding deprecated elements present in request and/or response at runtime so that tools can recognize this, log warnings and highlight the usage of deprecated elements as needed.
4. Deprecated `API Elements` must remain supported for the life of the major version or until customers are no longer using them (the means to determine this are left to the discretion of the API owner since it's their customers who will ultimately be impacted).

<h2 id="deprecation-solution">Solution</h2>

The following describes how to address the requirements listed above. The solution involves addressing documentation related requirement using an annotation and using a custom header to address the runtime related requirement. 

1. [Documentation](#deprecation-documentation)
2. [Runtime](#deprecation-runtime)

<h3 id="deprecation-documentation">Documentation</h3>

An optional annotation named `x-deprecated` is used to mark an `API Element` as deprecated in the definition of the API.

<h4 id="deprecation-annotation">Annotation: x-deprecated</h4>

`x-deprecated` can be used to deprecate any kind of `API Element`. The annotation should be used inline precisely where the `API Element` is defined. It is expected that the API tools generating documentation by introspecting the API definition would recognize this annotation and highlight the corresponding `API Element` as deprecated. It is also assumed that this annotation can be completely ignored by tools including those generating implementation bindings (POJO). In other words, it is not in scope of this solution that any implementation language specific construct (such as Java annotation `@deprecated`) would be generated for the `x-deprecated` annotation.

It is expected that the API documentation would highlight deprecated `API Elements` annotated by `x-deprecated` in the API specification distinctly and at the appropriate granularity. 

<h4 id="deprecation-schema-annotation">Schema for x-deprecated Annotation</h4>

We have provided specific JSON object types to use for deprecation of specific `API Elements`. This section lists schema for these types. The intent in providing schema for specific application of `x-annotation` is to make it easy for API developers to annotate the `API definition` and for API tool(s) to highlight each deprecated `API Element` with appropriate details.

##### Common Schema Elements

Following are common schema types that are used across new JSON object types to be used for deprecation.

```

		"x-deprecatedValue": {
			"type": "string",
			"description": "Value of the element that is deprecated. Use to deprecate a particular value in parameter or schema property as applicable."
		},
		"x-deprecatedSee": {
			"type": "string",
			"description": "URI (indirect or absolute) or name of to new parameter, resource, method, api_element, as applicable."
		},
		"x-apiVersion": {
			"pattern": "^[1-9][0-9]*[.][0-9]+$",
			"minLength": 3,
			"maxLength": 8,
			"description": "This string should contain the release or version number at which this schema element became deprecated. Version should be in the format '{major}.{minor}' (no leading 'v')."
		}

```

<h5 id="deprecation-schema-resource">Deprecated Resource</h5>

The following schema MUST be used to deprecate resource objects in `API definition`. Examples of resource object in `swagger.json` are: `operation` and `paths`. 


```
		"x-deprecatedResource": {
			"type": "object",
			"title": "Schema for a deprecated resource.",
			"description": "Schema for deprecating a resource API element. A resource API element could be an operation or paths.",
			"properties": {
				"see": {
					"$ref": "#/definitions/x-deprecatedSee"
				},
				"since_version": {
					"$ref": "#/definitions/x-apiVersion"
				}
			}
		}
```

The following section provides several examples showing usage of `deprecatedResource` for `x-deprecated` annotation at resource level.

<h6 id="deprecation-example-resource">Example: Deprecated Resource</h6>

The following example shows deprecated resource named `commercial-entities` in `swagger.json`.


```
    "paths": {
       
        "/commercial-entities": {
             "x-deprecated": {
            		"see": "financial-entities",
            		"since_version": "1.4"
            	},
        ...
        }

```
		


<h6 id="deprecation-example-method">Example: Deprecated Method</h6>

The following example shows a deprecated method `PUT /commercial-entities/{merchant_id}/agreements` and encourages to use new method `PATCH /commercial-entities/{merchant_id}/agreements`.

```
    "paths": {
        "/commercial-entities/{merchant_id}/agreements": {
            "put": {
                "summary": "Updates the Commercial Entity Agreements Details for a Merchant.",
                "operationId": "commercial-entity.agreement.update",
                "x-deprecated": {
            			"see": "patch",
            			"since_version": "1.4"
            	},
            	"parameters": [
                    {
                        "name": "merchant_id",
                        "in": "path",
                        "description": "The encrypted Merchant's identifier.",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "Agreements",
                        "in": "body",
                        "description": "An array of AgreementDetails",
                        "required": true,
                        "schema": {
                            "$ref": "./model/agreement_details.json"
                        }
                    }
                ],
            ...
         }
```

<h5 id="deprecation-schema-parameter">Deprecated Parameter</h5>

`swagger.json` provides a way to define one or more parameter for a method. The type of parameters are: path, query and header. Typically, query and header parameters can be deprecated. The following schema MUST be used while using `x-deprecated` annotation for a parameter.

```
		"x-deprecatedParameter": {
			"type": "object",
			"title": "Schema for a deprecated parameter.",
			"description": "Schema for deprecating an API element inline. The API element could be a custom HTTP header or a query param.",
			"properties": {
				"value": {
					"$ref": "#/definitions/x-deprecatedValue"
				},
				"see": {
					"$ref": "#/definitions/x-deprecatedSee"
				},
				"since_version": {
					"$ref": "#/definitions/x-apiVersion"
				}
			}
		}
```

The following section provides several examples showing usage of `deprecatedParameter` for `x-deprecated` annotation at parameter level.

<h6 id="deprecation-example-query-parameter">Example: Deprecated Query Parameter</h6>

The following example shows usage of the `x-deprecated` annotation in `swagger.json` to indicate deprecation of a query parameter `record_date`.

```
        "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the merchant_id.",
                "description": "Gets the Commercial Entity as denoted by the merchant_id.",
                "operationId": "commercial-entity.get",
                "parameters": [
                    {
                        "name": "record_date",
                        "in": "query",
                        "description": "The date to use for the query; defaulted to yesterday.",
                        "required": false,
                        "type": "string",
                        "format": "date",
                        "x-deprecated": {
                            "since_version": "1.5",
                            "see": "transaction_date"
                        }
                    },
                    {
                        "name": "transaction_date",
                        "in": "query",
                        "description": "The date to use for the query; defaulted to yesterday.",
                        "required": false,
                        "type": "string",
                        "format": "date"
                    },
                    ...
```



<h6 id="deprecation-example-header">Example: Deprecated Header</h6>

The following example shows a deprecated custom HTTP header called `CLIENT_INFO`.


###### OpenAPI

```
        "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the merchant_id.",
                "description": "Gets the Commercial Entity as denoted by the merchant_id.",
                "operationId": "commercial-entity.get",
                "parameters": [
                    ...
                    {
                        "name": "CLIENT_INFO",
                        "in": "header",
                        "description": "Optional header for all the API's to pass on api caller tracking information. This header helps capture any input from the caller service and pass it along to analytics for tracking .",
                        "in" : "header",
                        "x-deprecated": {
                             "since_version": "1.5"
                        }

                    }
                    ...
```


<h6 id="deprecation-example-query-parameter-value">Example: Deprecated Query Parameter Value</h6>


The following example shows usage of the `x-deprecated` annotation in `swagger.json` to indicate deprecation of a specific value (`y`) for a query parameter named `fields`.


```
        "/commercial-entities": {
            "get": {
                "summary": "Gets Commercial Entities.",
                "description": "Gets the Commercial Entities.",
                "operationId": "commercial-entity.get",
                "parameters": [
                    {
                        "name": "fields",
                        "in": "query",
                        "description": "Fields to return in response, default is x, possible values are x, y, z.",
                        "required": false,
                        "type": "string",
                        "x-deprecated": {
                            "since_version": "1.5",
                            "value": "y"
                        }
                    },
                    {
                        "name": "transaction_date",
                        "in": "query",
                        "description": "The date to use for the query; defaulted to yesterday.",
                        "required": false,
                        "type": "string",
                        "format": "date"
                    },
                    ...
```

<h5 id="deprecation-schema-object">Deprecated JSON Object Schema</h5>

In order to deprecate a schema of JSON Object itself or one or more properties within the JSON Object schema, we recommend using a schema called `deprecatedSchema` for the `x-deprecated` annotation. 


```
		"x-deprecatedSchema": {
			"type": "array",
			"description": "Schema for a collection of deprecated items in a schema.",
			"items": {
				"$ref": "#/definitions/x-deprecatedSchemaProperty"
			}
		}
```

```
		"x-deprecatedSchemaProperty": {
			"type": "object",
			"title": "Schema for a deprecated schema property or schema itself.",
			"description": "Schema for deprecating an API element within JSON Object schema. The API element could be an individual property of a schema of a JSON type or an entire schema representing JSON object.",
			"required": ["api_element"],
			"properties": {
				"api_element": {
					"type": "string",
					"description": "JSON pointer to API element that is deprecated. If the API element is JSON Object schema of a type itself, JSON pointer MUST point to the root of that schema. If the API element is a property of schema, the JSON pointer MUST point to that property."
				},
				"value": {
					"$ref": "#/definitions/x-deprecatedValue"
				},
				"see": {
					"$ref": "#/definitions/x-deprecatedSee"
				},
				"since_version": {
					"$ref": "#/definitions/x-apiVersion"
				}
			}
		}
		
```

In order to avoid extending JSON draft-04 schema with `keywords` needed to either deprecate the schema itself by using `x-annotation` in metadata section of the schema or deprecate individual properties inline, we have chosen a less disruptive route of using `x-annotation` next to references for JSON Object in `API definition`. Therefore, this annotation SHOULD be used in `API definition` where schema is "referred". In case if you are OK with inlining, you should go ahead and annotate individual deprecated properties in schema or schema itself. 

As of March 2017, [OpenAPI 3.0.0-rc0](https://github.com/OAI/OpenAPI-Specification/blob/OpenAPI.next/versions/3.0.md) has introduced `deprecated` flag to apply at operation, parameter and schema field levels. `x-annotation` could be used alongside the `deprecated` flag to provide additional useful information for the deprecated `API Element`.


<h6 id="deprecation-example-schema-property">Example: Deprecated Property In Response</h6>

The following example shows usage of the `x-deprecated` annotation in `API definitions` to indicate deprecation of a property named `address` in response.



```
        "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the merchant_id.",
                "description": "Gets the Commercial Entity as denoted by the merchant_id.",
                "operationId": "commercial-entity.get",
                "responses": {
                    "200": {
                        "description": "The Commercial Entity.",
                        "schema": {
                            "$ref": "./model/interaction/commercial-entities/merchant_id/get_response.json",
                            "x-deprecated": [
            					   {
            						  "api_element": "./model/interaction/commercial-entities/merchant_id/get_response.json#/address",
            						  "see": "./model/interaction/commercial-entities/merchant_id/get_response.json#/global_address",
            						  "since_version": "1.4"
            					   }
            				    ] 
                        }
                    },
                    "default": {
                        "description": "Unexpected error",
                        "schema": {
                            "$ref": "v1/schema/json/draft-04/error.json"
                        }
                    }
                }
```

<h6 id="deprecation-example-enum">Example: Deprecated Enum In Response</h6>

The following example shows usage of the `x-deprecated` annotation in `API definitions` to indicate deprecation of an enum `FAILED` used by a property named `state`.


```
       "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the specified merchant identifier.",
                "description": "Gets the Commercial Entity as denoted by the specified merchant identifier.",
                "operationId": "commercial-entity.get",
                "responses": {
                    "200": {
                        "description": "The Commercial Entity.",
                        "schema": {
                            "$ref": "./model/interaction/commercial-entities/merchant_id/get_response.json",
                            "x-deprecated": [
            					   {
            						  "api_element": "./model/interaction/commercial-entities/merchant_id/get_response.json#/state",
            						  "value": "FAILED",
            						  "since_version": "1.4"
            					   }
            				    ] 
                        }
                    },
                    "default": {
                        "description": "Unexpected error",
                        "schema": {
                            "$ref": "v1/schema/json/draft-04/error.json"
                        }
                    }
                }
			

```

<h3 id="deprecation-runtime">Runtime</h3>

The API server MUST inform client app(s) of the deprecated `API element`(s) present in request and/or response. 

<h4 id="deprecation-header">Header: Foo-Deprecated</h4>

It is recommended to use a custom HTTP header named `Foo-Deprecated` to convey deprecation related information. The service MUST respond with the `Foo-Deprecated` header in the following cases:

1. The caller has used one or more deprecated element in request. 
2. There is one or more deprecated element in the response.

In order to avoid bloating the responses with static information related to deprecation that does not change from response to response on the same end point, we recommend that API developers provide just an empty JSON object as a value as shown below. In the future, we plan to replace this with something that still does not bloat the responses but still provides enough information so that tools could scan responses for deprecation and take appropriate actions such as notifying App developers/administrators.

```
"Foo-Deprecated": "{}"
```

**Note**: Applications consuming this header MUST not take any action based on the value of the header at this time. Instead, we recommend that these applications SHOULD take action based only on the existence of the header in the response. 

References

1. [JEP Enhanced Deprecation](http://openjdk.java.net/jeps/277)
2. [How and When to Deprecate APIs](http://docs.oracle.com/javase/7/docs/technotes/guides/javadoc/deprecation/deprecation.html)
3. [Semantic Versioning 2.0](http://semver.org/)


<h1 id="patterns-and-use-cases">Patterns And Use Cases</h1>

Please refer to [Patterns And Use Cases](patterns.md).


[0]: https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm "RESTful Architectural Style"
[1]: https://tools.ietf.org/html/rfc5988#section-4 "Link Relation Type"
[2]: http://tools.ietf.org/html/draft-zyp-json-schema-04 "JSON schema draft-04"
[3]: http://json-schema.org/draft-04/hyper-schema# "JSON hyper-schema draft-04"
[4]: http://json-schema.org/latest/json-schema-hypermedia.html#anchor17 "Link Description Objects (LDO)"
[5]: http://www.iana.org/assignments/link-relations/link-relations.xhtml "IANA's list of standardized link relations"
[6]: https://tools.ietf.org/html/rfc6570 "URI template"
[7]: uri-component-names "URI Definition"
[8]: https://tools.ietf.org/html/rfc3986 "RFC 3968"
[9]: http://tools.ietf.org/html/draft-zyp-json-schema-04 "draft-04"
[10]: http://tools.ietf.org/html/draft-zyp-json-schema-03 "draft-03"
[11]: http://swagger.io/specification "OpenAPI"
[12]: http://swagger.io/specification/#schemaObject "OpenAPI Schema"
[13]: v1/schema/json/draft-04 "Common Types"
[14]: v1/schema/json/draft-04/README.md "README"
[15]: v1/schema/json/draft-04/money.json "money_json"
[16]: https://github.com/googlei18n/libaddressinput/wiki/AddressValidationMetadata "i18n-api"
[17]: https://www.w3.org/TR/html51/sec-forms.html#autofill-field "HTML 5.1 autofill"
[18]: v1/schema/json/README_address.md "README Address"
[19]: v1/schema/json/draft-04/address_portable.json "address_portable.json"
[20]: https://www.informatica.com/content/dam/informatica-com/global/amer/us/collateral/other/addressdoctor-cloud-2_user-guide.pdf "Address Doctor"
[21]: https://www.ietf.org/rfc/rfc3339.txt "RFC3339" 
[22]: v1/schema/json/draft-04/error.json "error.json"
[23]: http://tools.ietf.org/html/rfc6901 "JavaScript Object Notation (JSON) Pointer"
[24]: v1/schema/json/draft-04/error_spec.json "error_spec.json"
[25]: v1/schema/json/draft-04/error_spec_issue.json "error_spec_issue.json"
[26]: v1/schema/json/draft-04/error_catalog.json "error_catalog.json"
[27]: v1/schema/json/draft-04/error_catalog_item.json "error_catalog_item.json"
[28]: v1/schema/json/draft-04/error_details.json "error_details.json"
[29]: http://techbus.safaribooksonline.com/book/web-development/web-services/9780596809140 "RESTful Web Services Cookbook"
[30]: http://json.org/ "JSON"




