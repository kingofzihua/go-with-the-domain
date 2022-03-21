# Welcome on board! 欢迎加入 

## Why business software 为什么选择商业软件

If you think “business software” is just enterprise payroll applications, think again. If you work on any web
application or SaaS product, you write business software.

如果你认为"商业软件"只是企业的薪资应用程序，那就再想想。如果你在任何网络应用或 SaaS 产品上工作，你就在编写商业软件

You’ve seen poor code before, and we did as well. What’s worse, we’ve experienced how poorly designed codebases hold the
product’s rapid growth and the company’s success.

你以前见过糟糕的代码，我们也是如此。更糟糕的是，我们经历过设计不良的代码库如何牵制产品的快速增长和公司的成功。

We looked into different patterns and techniques on how to improve this. We applied them in practice within our teams
and seen some work very well.

我们研究了如何改进的不同模式和技术。我们在团队中将它们应用到实践中，并看到有些方法非常好用。

## Why Go 为什么选择 Go

Go’s initial success happened mostly within low-level applications, like CLI tools and infrastructure software, like
Kubernetes. However, [Go 2020 Survey](https://blog.golang.org/survey2020-results) shows that majority of Go developers
write API/RPC services. 68% percent of respondents use Go for “web programming”.

Go 最初的成功主要发生在基础应用中，像 CLI 工具和基础设施软件，如 Kubernetes。然而，[Go 2020调查](https://blog.golang.org/survey2020-results) 显示，大多数Go开发者编写
API/RP C服务。68%的受访者将 Go 用于 "网络编程"。

Some claim that Go is missing advanced features to become a productive tool. On the other hand, we feel simplicity is
Go’s strength. It’s viable for writing business applications not despite the lack of features but because of it.

有些人声称，Go 缺少先进的功能，无法成为一个高效的工具。另一方面，我们认为简单性是 Go 的优势。 尽管缺乏功能，但它对于编写业务应用程序是可行的。

You don’t want your business’s most crucial code to be smart and using the latest hacks. You need it to be maintainable,
extremely clear, and easy to change. We’ve found that using Go along advanced architectural patterns can give you that.

您不希望您的企业最关键的代码变得痛苦并容易受到的黑客攻击。您需要它是可维护的、非常清晰且易于更改的。我们发现，使用 Go 和高级架构模式可以给你这个。

It’s naive to think you will avoid technical debt just thanks to using Go. Any software project that’s complex enough
can become a [Big Ball of Mud](https://en.wikipedia.org/wiki/Big_ball_of_mud) if you don’t think about its design
beforehand. Spoiler alert: microservices don’t reduce complexity.

认为使用 Go 就能避免技术债务的想法是天真的。任何足够复杂的软件项目，如果你不事先考虑它的设计，就会变成一个[大泥球](https://en.wikipedia.org/wiki/Big_ball_of_mud)
。剧透一下：微服务并不能减少复杂性。

## How is this book different 这本书有什么不同

In 2020, we started a new series on our blog focused on building business software.

2020年，我们在博客上开始了一个新的系列，专注于构建商业软件。

We didn’t want to write articles where the author makes bold claims, but the described techniques are hard to apply.
Examples are great for learning, so we decided to create a real, open-source, and deployable web application that would
help us show the patterns.

我们不想写作者提出大胆主张的文章，但所描述的技术很难应用。示例非常适合学习，因此我们决定创建一个真实的、开源的、可部署的 Web 应用程序来帮助我们展示这些模式。

In the initial version of the application, we’ve hidden on purpose some anti-patterns. They seem innocent, are hard to
spot, and can hurt your development speed later.

在应用程序的初始版本中，我们故意隐藏了一些反模式。它们看起来很无辜，很难被发现，并且会影响您以后的开发速度。

In the following posts from the series, we refactored the application several times, showing the anti-patterns and how
to avoid them. We also introduced some essential techniques like DDD, CQRS, and Clean Architecture. The entire commit
history is available for each article.

在本系列的后续文章中，我们多次重构了应用程序，展示了反模式以及如何避免它们。我们还介绍了一些基本技术，例如 DDD、CQRS 和 Clean Architecture。每篇文章都有完整的提交历史记录。

Posts from that blog series are Go with the Domain’s foundation - edited and prepared for a better reading experience.

该博客系列中的帖子是 Go with the Domain 的基础 - 为更好的阅读体验进行了编辑和准备。

## Who is this book for 这本书是为谁准备的

We assume you have basic knowledge of Go and already worked on some projects. Ideally, you’re looking for patterns that
will help you design applications to not become legacy software in a few months.

我们假设您具有 Go 的基本知识并且已经参与过一些项目。理想情况下，你正在寻找能够帮助你设计应用程序的模式，使之在几个月内不会成为遗留软件。

Most of the ideas in this book shine in applications with complex business scenarios. Some make sense in simpler cases,
and some will look like terrible over-engineering if used in small projects. Try to be pragmatic and choose the best
tool for the job. We share some hints on this topic as well.

本书中的大多数想法在具有复杂业务场景的应用中大放异彩。有些在比较简单的情况下是有意义的，有些如果用在小项目中就会看起来像可怕的过度工程。试着务实一点，选择最适合的工具。我们也会分享一些关于这个主题的提示。

## Who we are 我们是谁

We’re [Miłosz](https://twitter.com/m1_10sz) and [Robert](https://twitter.com/roblaszczak), co-founders
of [Three Dots Labs](https://threedotslabs.com/). We based this book on posts from
our [tech blog](https://threedots.tech/) 。

我们是 [Miłosz](https://twitter.com/m1_10sz) 和 [Robert](https://twitter.com/roblaszczak)
，[三点实验室](https://threedotslabs.com/) 的共同创办人。这本书是根据我们的 [技术博客](https://threedots.tech/) 的文章编写的。