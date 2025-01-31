#  Node.js微服务实践(一)

## 什么是微服务

微服务是一种架构风格，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。
微服务的概念源于2014年3月Martin Fowler所写的一篇文章“Microservices”(http://martinfowler.com/articles/microservices.html)。

尽管“微服务”这种架构风格没有精确的定义，但其具有一些共同的特性，如围绕业务能力组织服务、自动化部署、智能端点、对语言及数据的“去集中化”控制等等。微服务架构的思考是从与整体应用对比而产生的。

通常，在一家公司随着业务需求的增长为逐步发展（自然增长）的过程中，前期往往是以单块架构的方式来组织系统的。因为对于软件的初期构建来说，单块架构的方式是最容易且最高效的。但是若干年（甚至是几个月）后，受限于前期既有单块软件系统内部耦合性，再向该系统加新功能变得越来越艰难。

### 单块软件

企业应用，因为服务于众多业务需求，因此会有些特定的软件应用提供许多功能，而一般惯例是把这些功能都堆在单个单片应用中。例如，ERP，CRM和其他各种软件系统被规划构建为具有数百个功能的整体。这种带坑应用一经部署，在往后的故障排除、扩展和升级场景中就是一场接一场的恶梦了。

面向服务架构（SOA）旨在通过引入“服务”的概念来克服以上的限制，“服务”是从应用程序提供的类似功能中提取出来的聚合和分组。因此，使用SOA，软件应用程序就会被设计为“粗粒度”服务的组合。然而，SOA中服务的范围非常广泛，这又引出了复杂而庞大的服务与一大堆操作（功能）以及复杂无比的消息格式和标准（如WS *标准）。

![image](https://qnstatic.sunil.wang/image-1538966090031.png)

多数情况下，SOA中的服务彼此独立，只不过它们与所有其他服务一起部署在相同的运行时间里（只需考虑将部署到同一个Tomcat实例中的多个Web应用）。和单片软件应用类似，这些服务通过积累各种功而具有随时间推移的习惯。图1是一个非常好的一个单片架构的例子，显示了包括多个服务的零售软件应用，所有这些服务都能部署到同一应用程序。

说了那么多，大家是不是有点不明所以？我总结了一些重点，以下就是基于单快软件架构的应用程序的一些特性归纳：

- 单片应用程序被设计、开发和部署为单个单元。
- 单片应用相对复杂，导致维护，升级和新特性困难重重。
- 难以用单片架构来实践敏捷开发和交付方法。
- 想更新某部分，很可能要重新部署整个应用。
- 规模需求冲突的情况下，若想做单点扩展，要做好多倍资源甚至推倒重来的准备（例如，想给A服务更多CPU，B服务可能要重做，也可能要补两三倍memory）
- 可靠性：一个不稳定的服务可以拖慢整个应用性能。
- 难创新：采用新技术和框架非常困难，因为所有的功能必须建立在均匀的技术/框架上。

## 面向微服务的架构

面向微服务的架构拥有一些特质。任意规模的大中型公司如果想让自身IT系统保持弹性，并希望随时都可以对系统规模进行伸缩的话，必定会对这些特质趋之若鹜。

### 为什么面向微服务的架构更好

微服务架构（MSA）的基础是将单片应用作为一套小型和独立服务来开发，这些服务都在自己的空间独立开发、部署和运行。在微服务体系结构的大多数定义中，它普遍被解释为将巨大的可用服务隔离成一套独立服务的过程。

但是面向微服务的架构并是不工程的圣杯。弹性、可组合性以及灵活性是面向微服务架构设计的关键原则。如果不遵守你将失去一个完美的解决方案，并最终在单块应用分拆多台机器上时遇到大量的问题。

### 不足之处
事无绝对，微服务也会有几下问题：

- 网络延迟：微服务具有分布式的特性，从而无可避免地会存在网络延迟
- 运维负担：更多的服务器也意味着需要更多的维护工作
- 最终一致性：在一个对事务性要求较高的系统中，考虑到实现的局限性，各个节点在某一时间内可能会出现数据不一致

### 关键设计原则

一般来说，我们可以确定一下设计原则：

- 单一责任原则（SRP）：为微服务提供有限和重点突出的业务范围，有助于我们满足开发和提供服务的敏捷性。
- 在微服务的设计阶段，我们应该找到各服务的边界，并将其与业务能力（也称为域驱动设计中的有界环境 ）保持一致。
- 微服务设计要确保敏捷/独立开发和部署服务的丝滑稳定。
- 我们的重点应该放在微服务的范围上，而不是使服务更"小"。服务的大小应该是指所需的范围大小，以促进给定的业务能力。
- 与SOA中的服务不同，给定的微服务应该具有很少的操作/功能和简单的消息格式。
- 随着时间的推移，首先开始具有相对广泛的服务边界，重构到较小的服务界限（基于业务需求），这是一个很好的做法。


## 关键的好处

现在我们看几项关键的好处。

### 弹性

维基百科将弹性定义为`系统处理变化的能力`。我对弹性的理解是在问题被解决后系统从异常状态或者压力期中优雅的恢复，同时又不会影响系统性能的能力。

这虽然看上去很简单，但是在构建面向微服务软件的时候，问题源会由于系统的分布式特性而被放大，有时很难防止所有的异常情况。

弹性是从错误中优雅回复的能力。但是同样也为系统带来了新的复杂度：如果一个微服务出现了问题，我们能否防止系统的常规故障？理想情况下，我们应该以这样一种方式来构建服务：仅对服务响应进行降级而非让系统出现常规故障，即使这样做也并不容易。

### 可伸缩性

如今各大公司的一个通病是系统存在可伸缩性的问题。通常，这些问题并不涉及应用的每一层次或所有子系统。往往只有个别子系统或服务会明显鳗鱼其余部分，一旦没有处理好容量问题就会导致整个应用发生故障。

下图描述了如何对微服务进行扩展（扩展成两个邮件服务）的，同时又不牵扯系统的其余部分：

![image](https://qnstatic.sunil.wang/image-1538965170184.png)
![image](https://qnstatic.sunil.wang/image-1538965247709.png)

### 技术的多样性

软件世界每几个月就出更新系统。新语言进入业界成为某类系统事实标准的节奏片刻位停。

- Golang：因其结合了强大的性能与优雅简洁的语法而成为当前一种趋势，任何只要拥有一门编程语言经验的人都可以在几天学会它。
- Java：自从Sping Boot 发布以后，他成为在编写敏捷服务方面相当有吸引力的技术栈。
- DJango：是一款强大且可用于编写微服务的Python的框架，与Ruby on Raile 非常相似。
- Node.js：利用了著名的JavaScript 的优势，创建了一个新的服务端技术栈，从而改变了工程师们编写新软件的方式。

那么，将这些语言的技术栈结合起来会有什么问题吗？平心而论，这是一个优势：`我们可以选择合适的工具来做相对应的工作`。只要待集成的技术是标准化的，面向微服务的架构便可以帮你实现这一点。

下图展示了微服务是如何隐藏数据的存取逻辑的，两个服务在存取数据方面共用同一个通信点，从而能很好地互相解耦：

![image](https://qnstatic.sunil.wang/image-1538965275990.png)

Node.js 并不是一门适合执行并行任务的语言。针对那些处于压力之下的微服务来说，我们可以选择一门更适合的语言来进行开发，比如 Erlang，从而可以以一种更加优雅的方式来管理并发。

### 可替换性

可替换性是指替换系统中某个组件而不影响系统行为的一种能力。
当我们在讨论软件的时候，可替换性往往是与低耦合密不可分的。在编写微服务的时候不能讲内部逻辑暴露给调用服务，服务实现对客户端来说是透明的，客户端只需要了解的只有接口。

### 独立性

所有服务应该都是独立的，他们通过接口进行交互。除了协定确认接口这一环节之外，不同的工程师团队可以在无需交流的情况下完成对服务的开发。

### 易于部署

微服务应当易于部署，原因如下：

- 少量的业务逻辑，导致更易于部署。
- 微服务是自治的工作单元，所以升级一个服务对于复杂的系统来说是一个局部可控的问题。无需重新部署整个系统。
- 微服务架构中的基础设施和配置都应该尽可能的自动化（如 Docker 来部署微服务）。

## SOA 与微服务的比较

面向微服务架构（SOA）与微服务架构非常想象的。那么他们之间的区别到底是什么呢？

微服务是细粒度的 SOA 组件。换句话说，某个单个SOA组件可以拆成多个微服务，而这些微服务通过分工协作，可以提供与原SOA组件相同级别的功能，如下图：

![image](https://qnstatic.sunil.wang/image-1538965313168.png)

微服务与 SOA 之间的另一个不同之处是服务互联和编写服务时所使用的技术。而另一方面，微服务推崇执行的标准（例如 HTTP）却是人民广泛了解并共同使用的。我们可以通过选择合适的语言或者工作来构建某个组件（微服务），进而获得 技术多样性 提到的关键好处。

除了技术栈和服务规模之外，在SOA于微服务之间还有一个更大的区别：领域模型。在一个机遇微服务的软件中，每个微服务应该在本地存储自身管理数据，并将领域模型分别隔离到单个服务中。而在面向SOA的软件中，数据往往存储在单个大型的数据库中，服务之间会共享领域模型。

## 为什么选择 Node.js

Node.js 是一门新型技术栈，对于很多人而言，它仅仅只是一个趋势，离作为解决问题的实际工具还欠点火候。

让我们来看重点看看 Node.js 。 Node.js 是用来构建面向微服务的架构的绝佳选择，原因如下：

- 学习门槛低（如果想要精通还是有一定门槛的）
- 易于扩展
- 对测试友好
- 易于部署
- 可以通过 npm 进行依赖管理
- 有着大量与主流标准协议相集成的库

### API 聚合

API 聚合是一项用于将不同功能（插件、方法等）组合成一个接口的高级技术。例子：

```javascript
const express = require('express');
const app = express();

app.get('/sayhello', function (req, res) {
  res.send('Hello World!');
});
app.get('/saygoodbye', function(req, res) {
  res.send('Bye bye!');
}

const server = app.listen(3000, function () {

  let host = server.address().address;
  let port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);

});

```

前面的例子使用了 Express, 这是一个在Node.js技术栈中广为流行的 Web 框架。该框架同样也是围绕API聚合来构建的。

让我们来看下 第四行和第七行。在这些代码里，开发者注册了两个方法。当某人以 GET 请求的方式分别请求 URL：/sayhello 和 /saygoodbye 时，这两个对应的方法将被执行。在该例子中，该接口就是一个在3000端口上监听的app。

## 小结

在本文中你了解一些面向微服务的架构的关键好处，比如可以为相应的服务选择合适的语言（语言多样性）；以及一些可能会加重我们负担的误区，比如如何有面向微服务的架构分布式特性而带来的运维方面开销。

最后，我们讨论了 Node.js 是用来构建微服务的强大工具，以及如何通过利用像 API 聚合这样的技术来从 JavaScript 获益，从而构建出高质量的软件组件。
