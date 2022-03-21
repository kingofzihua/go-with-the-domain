# Welcome on board!

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

Some claim that Go is missing advanced features to become a productive tool. On the other hand, we feel simplicity is
Go’s strength. It’s viable for writing business applications not despite the lack of features but because of it.

You don’t want your business’s most crucial code to be smart and using the latest hacks. You need it to be maintainable,
extremely clear, and easy to change. We’ve found that using Go along advanced architectural patterns can give you that.

It’s naive to think you will avoid technical debt just thanks to using Go. Any software project that’s complex enough
can become [a Big Ball of Mud](https://en.wikipedia.org/wiki/Big_ball_of_mud) if you don’t think about its design
beforehand. Spoiler alert: microservices don’t reduce complexity.

## How is this book different

In 2020, we started a new series on our blog focused on building business software.

We didn’t want to write articles where the author makes bold claims, but the described techniques are hard to apply.
Examples are great for learning, so we decided to create a real, open-source, and deployable web application that would
help us show the patterns.

In the initial version of the application, we’ve hidden on purpose some anti-patterns. They seem innocent, are hard to
spot, and can hurt your development speed later.

In the following posts from the series, we refactored the application several times, showing the anti-patterns and how
to avoid them. We also introduced some essential techniques like DDD, CQRS, and Clean Architecture. The entire commit
history is available for each article.

Posts from that blog series are Go with the Domain’s foundation - edited and prepared for a better reading experience.

## Who is this book for

We assume you have basic knowledge of Go and already worked on some projects. Ideally, you’re looking for patterns that
will help you design applications to not become legacy software in a few months.

Most of the ideas in this book shine in applications with complex business scenarios. Some make sense in simpler cases,
and some will look like terrible over-engineering if used in small projects. Try to be pragmatic and choose the best
tool for the job. We share some hints on this topic as well.

## Who we are

We’re [Miłosz](https://twitter.com/m1_10sz) and [Robert](https://twitter.com/roblaszczak), co-founders
of [Three Dots Labs](https://threedotslabs.com/). We based this book on posts from
our [tech blog](https://threedots.tech/)。