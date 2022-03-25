## When to stay away from DRY / 何时远离 DRY(有且仅有一次)

Miłosz Smółka / 米沃什·斯莫乌卡

In this chapter, we begin refactoring of Wild Workouts. Previous chapters will give you more context, but reading them
isn’t necessary to understand this one.

在本章中，我们开始重构 Wild Workouts。前面的章节会给你更多的背景信息，但是阅读它们并不是理解这一章的必要条件。

### Background story / 背景介绍

Meet Susan, a software engineer, bored at her current job working with legacy enterprise software. Susan started looking
for a new gig and found Wild Workouts startup that uses serverless Go microservices. This seemed like something fresh
and modern, so after a smooth hiring process, she started her first day at the company.

Susan 是一名软件工程师，她对目前的工作感到厌倦，因为她的工作是与传统的企业软件打交道。Susan开始寻找新的工作，并发现了使用无服务器 Go 微服务的 Wild Workouts
创业公司。这似乎是一些新鲜和现代的东西，所以在顺利的招聘过程中，她开始了在公司的第一天。

There were just a few engineers in the team, so Susan’s onboarding was blazing fast. Right on the first day, she was
assigned her first task that was supposed to make her familiar with the application.

团队中只有几个工程师，所以 Susan 的入职速度非常快。就在第一天，她被分配了第一个任务，目的是让她熟悉这个应用程序。

We need to store each user’s last IP address. It will enable new security features in the future, like extra
confirmation when logging in from a new location. For now, we just want to keep it in the database.

我们需要存储每个用户的最后一个 IP 地址。它将在未来启用新的安全功能，比如从新的地点登录时的额外确认。现在，我们只想把它放在数据库里。

Susan looked through the application for some time. She tried to understand what’s happening in each service and where
to add a new field to store the IP address. Finally, she discovered a [User](https://bit.ly/2NUFZha) structure that she
could extend.

Susan 看了一段时间的应用程序。她试图了解每个服务中发生了什么，以及在哪里添加一个新字段来存储IP地址。最后，她发现了一个她可以扩展的 [User](https://bit.ly/2NUFZha) structure。

```shell
 // User defines model for User.
 type User struct {
        Balance     int     `json:"balance"`
        DisplayName string  `json:"displayName"`
        Role        string  `json:"role"`
+       LastIp      string  `json:"lastIp"`
 }
```

It didn’t take Susan long to post her first pull request. Shortly, Dave, a senior engineer, added a comment during code
review.

没过多久，Susan就发布了她的第一个拉动请求。不久，高级工程师 Dave 在代码审查期间添加了一条评论。

<center>I don’t think we’re supposed to expose this field via the REST API.</center>  
<center>我认为我们不应该通过REST API暴露这个字段。</center>  

Susan was surprised, as she was sure she updated the database model. Confused, she asked Dave if this is the correct
place to add a new field.

Susan 很惊讶，因为她确信她更新了数据库模型。她很困惑，问 Dave 这是否是添加新字段的正确位置。

Dave explained that Firestore, the application’s database, stores documents marshaled from Go structures. So the User
struct is compatible with both frontend responses and storage.

Dave 解释说，Firestore 是应用程序的数据库，它存储了从 Go 结构中调取的文件。因此，用户结构与前端响应和存储都是兼容的。

“Thanks to this approach, you don’t need to duplicate the code. It’s enough to change
the [YAML definition](https://bit.ly/2NKI4Mp) once and regenerate the file”, he said enthusiastically.

"多亏了这种方法，你不需要重复编写代码。改变一次 [YAML 定义](https://bit.ly/2NKI4Mp) 并重新生成文件就足够了"，他热情地说道。

Grateful for the tip, Susan added one more change to hide the new field from the API response.

感谢你的提示，Susan 又加了一个改动，把新字段从 API 响应中隐藏起来。

```shell
diff --git a/internal/users/http.go b/internal/users/http.go
index 9022e5d..cd8fbdc 100644
--- a/internal/users/http.go
+++ b/internal/users/http.go
@@ -27,5 +30,8 @@ func (h HttpServer) GetCurrentUser(w http.ResponseWriter, r *http.Request) {
        user.Role = authUser.Role
        user.DisplayName = authUser.DisplayName

+       // Don't expose the user's last IP externally
+       user.LastIp = nil
+
        render.Respond(w, r, user)
 }
```

One line was enough to fix this issue. That was quite a productive first day for Susan.

一行就足以解决这个问题。对 Susan 来说，那是非常富有成效的第一天

### Second thoughts / 第二个想法

Even though Susan’s solution was approved and merged, something bothered her on her commute home. **Is it the right
approach to keep the same struct for both API response and database model?** Don’t we risk accidentally exposing user’s
private details if we keep extending it as the application grows? What if we’d like to change only the API response
without changing the database fields?

尽管 Susan 的解决方案获得了批准和合并，但在她回家的路上还是有一些事情困扰着她。**对API响应和数据库模型保持相同的结构，这种做法是否正确？**
如果我们随着应用程序的增长不断扩展它，难道我们不会有意外暴露用户隐私的风险吗？如果我们只想改变API响应而不改变数据库字段，怎么办？

![](./chapter05/ch0501.png)

<center>Figure 5.1: Original solution</center>
<center>Figure 5.1: 原有的解决方案</center>

What about splitting the two structures? That’s what Susan would do at her old job, but maybe these were enterprise
patterns that should not be used in a Go microservice. Also, the team seemed very rigorous about the
**Don’t Repeat Yourself** principle.

把这两个结构分开怎么样？这是 Susan 在她以前的工作中会做的，但也许这些是企业模式，不应该用在 Go 微服务中。另外，该团队似乎对 "不要重复自己 "的原则非常严格。

### Refactoring / 重构

The next day, Susan explained her doubts to Dave and asked for his opinion. At first, he didn’t understand the concern
and mentioned that maybe she needs to get used to the “Go way” of doing things.

第二天，Susan 向 Dave 解释了她的疑虑并征求他的意见。起初，他不理解她的担忧，并提到也许她需要习惯于 "Go 的方式 "来做事。

Susan pointed to another piece of code in Wild Workouts that used a similar ad-hoc solution. She shared that, from her
experience, such code can quickly get out of control.

Susan 指出了Wild Workouts中的另一段代码，它使用了类似的临时解决方案。她分享说，根据她的经验，这样的代码会很快失去控制。

```
    user, err := h.db.GetUser(r.Context(), authUser.UUID)
    if err != nil {
        httperr.InternalError("cannot-get-user", err, w, r)
        return
    }
    user.Role = authUser.Role
    user.DisplayName = authUser.DisplayName

    render.Respond(w, r, user)
```

It seems the HTTP handler modifies the user in-place.

似乎 HTTP 处理程序就地修改了用户。

Eventually, they agreed to discuss this again over a new PR. Shortly, Susan prepared a refactoring proposal.

最终，他们同意在一个新的PR上再次讨论这个问题。不久，Susan 准备了一份重构建议。

```shell
diff --git a/internal/users/firestore.go b/internal/users/firestore.go
index 7f3fca0..670bfaa 100644
--- a/internal/users/firestore.go
+++ b/internal/users/firestore.go
@@ -9,6 +9,13 @@ import (
        "google.golang.org/grpc/status"
 )

+type UserModel struct {
+       Balance     int
+       DisplayName string
+       Role        string
+       LastIP      string
+}
+
diff --git a/internal/users/http.go b/internal/users/http.go
index 9022e5d..372b5ca 100644
--- a/internal/users/http.go
+++ b/internal/users/http.go
@@ -1,6 +1,7 @@
-       user.Role = authUser.Role
-       user.DisplayName = authUser.DisplayName

-       render.Respond(w, r, user)
+       userResponse := User{
+               DisplayName: authUser.DisplayName,
+               Balance:     user.Balance,
+               Role:        authUser.Role,
+       }
+
+       render.Respond(w, r, userResponse)
 }

```

Source: [14d9e7badcf5a91811059d377cfa847ec7b4592f on GitHub]()

This time, Susan didn’t touch the OpenAPI definition. After all, she wasn’t supposed to introduce any changes to the
REST API. Instead, she manually created another structure, just like User, but exclusive to the database model. Then,
she extended it with a new field.

这一次，Susan 没有碰 OpenAPI 的定义。毕竟，她不应该对 REST API 进行任何修改。相反，她手动创建了另一个结构，就像 User 一样，但对数据库模型是专用的。然后，她用一个新的字段来扩展它。

The new solution is a bit longer in code lines, but it removed code coupling between the REST API and the database
layer (all without introducing another microservice). The next time someone wants to add a field, they can do it by
updating the proper structure.

新的解决方案在代码行上有点长，但它消除了 REST API 和 database 层 之间的代码耦合（都没有引入另一个微服务）。下次有人想增加一个字段时，他们可以通过更新适当的结构来完成。

![](./chapter05/ch0502.png)

<center>Figure 5.2: Refactored solution</center>

<center>Figure 5.2: 重构方案</center>

### The Clash of Principles / 原则的冲突

Dave’s biggest concern was that the second solution breaks
the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle and introduces boilerplate. On the other
hand, Susan was afraid that the original approach violates the Single Responsibility Principle (the “S”
in [SOLID](https://en.wikipedia.org/wiki/SOLID)). Who’s right?

戴夫最担心的是，第二个解决方案打破了 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
原则，并引入了模板。另一方面，苏珊担心原来的方法违反了单一责任原则（ [SOLID](https://en.wikipedia.org/wiki/SOLID) 中的 "S"）。谁是对的？

It’s tough to come up with strict rules. Sometimes code duplication seems like a boilerplate, but it’s one of the best
tools to fight code coupling. It’s helpful to ask yourself if the code using the common structure is likely to change
together. If not, it’s safe to assume duplication is the right choice.

想出严格的规则是很难的。有时，代码重复似乎是一个模板，但它是对抗代码耦合的最好工具之一。问问自己使用共同结构的代码是否有可能一起改变，这很有帮助。如果不是，就可以安全地认为重复是正确的选择。

Usually, DRY is better applied to behaviors, not data. For example, extracting common code to a separate function
doesn’t have the downsides we discussed so far.

通常情况下，DRY 更适用于行为，而不是数据。例如，将常见的代码提取到一个单独的函数中，并没有我们目前讨论的缺点。

### What’s the big deal? / 有什么大不了的？

Is such a minor change even “architecture”?

如此微小的变化甚至是“架构”吗？

Susan introduced a little change that had consequences she didn’t know about. It was obvious to other engineers, but not
for a newcomer. I guess you also know **the feeling of being afraid to introduce changes in an unknown system because
you can’t know what it could trigger**。

Susan 引入了一个她不知道的后果的小变化。这对其他工程师来说是显而易见的，但对一个新来的人来说却不是。我猜你也知道那种不敢在一个未知的系统中引入变化的感觉，因为你无法知道它可能引发什么。

If you make many wrong decisions, even small, they tend to compound. Eventually, developers start to complain that it’s
hard to work with the application. The turning point is when someone mentions a “rewrite”, and suddenly, you know you
have a big problem.

如果你做了许多错误的决定，哪怕是很小的决定，它们往往会变得复杂。最终，开发人员开始抱怨说很难用这个应用程序工作。转折点是当有人提到 "重写 "时，突然，你知道你有一个大问题。

- The alternative to good design is always bad design. There is no such thing as no design.

  Adam Judge ()

- 好的设计的替代品总是坏的设计。不存在没有设计这回事。

  亚当-贾奇

It’s worth discussing architecture decisions before you stumble on issues. “No architecture” will just leave you with
bad architecture.

在你偶然发现问题之前，值得讨论架构决策。"没有架构 "只会给你留下糟糕的架构。

### Can microservices save you? / 微服务能拯救你吗？

With all benefits that microservices gave us, a dangerous idea also appeared, preached by some “how-to build
microservices” guides. **It says that microservices will simplify your application**. Because building large software
projects is hard, some promise that you won’t need to worry about it if you split your application into tiny chunks.

在微服务给我们带来的所有好处的同时，也出现了一个危险的想法，由一些“如何构建微服务”指南鼓吹。**它说微服务将简化您的应用程序。** 因为构建大型软件项目很困难，有些人承诺如果将应用程序拆分成小块，您就不必担心它。

This idea sounds good on paper, but it misses the point of splitting software. How do you know where to put the
boundaries? Will you just separate a service based on each database entity? REST endpoints? Features? **How do you
ensure low coupling between services?**

这个想法上听起来不错，但它忽略了拆分软件的意义。你如何知道在哪里设置边界？你会根据每个数据库实体分离一个服务吗？ REST 端点？功能？如何保证服务之间的低耦合？

### The Distributed Monolith / 分布式单体

If you start with poorly separated services, you’re likely to end up with the same monolith you tried to avoid, with
added network overhead and complex tooling to manage this mess (also known as a distributed monolith). You will just
replace highly coupled modules with highly coupled services. And because now everyone runs a Kubernetes cluster, you may
even think you’re following the industry standards.

如果你从分离不好的服务开始，你很可能最终得到你试图避免的同样的单体，并增加网络开销和复杂的工具来管理这种混乱（也被称为分布式单体）。你将只是用高度耦合的服务来取代高度耦合的模块。而且因为现在每个人都在运行 Kubernetes
集群，你甚至可能认为你在遵循行业标准。

Even if you can rewrite a single service in one afternoon, can you as quickly change how services communicate with each
other? What if they are owned by multiple teams, based on wrong boundaries? Consider how much simpler it is to refactor
a single application.

即使你能在一个下午重写一个服务，你能同样迅速地改变服务之间的交流方式吗？如果它们是由多个团队拥有的，基于错误的边界呢？考虑一下，重构一个单一的应用程序是多么简单的事情。

Everything above doesn’t void other benefits of using microservices, like independent deployments (crucial for
Continuous Delivery) and easier horizontal scaling. Like with all patterns, make sure you use the right tool for the
job, at the right time.

以上所有内容都不会影响使用微服务的其他好处，例如独立部署（对于持续交付至关重要）和更容易的水平扩展。与所有模式一样，请确保在正确的时间使用正确的工具来完成工作。

> We introduced similar issues on purpose in Wild Workouts. We will look into this in future chapters and discuss other splitting techniques. You can find some of the ideas in Robert’s post: [Why using Microservices or Monolith can be just a detail?](https://threedots.tech/post/microservices-or-monolith-its-detail/)
>
> 我们在 Wild Workouts 中故意引入了类似的问题。我们将在以后的章节中对此进行研究，并讨论其他拆分技术。您可以在 Robert 的帖子中找到一些想法：[为什么使用微服务或 Monolith 可能只是一个细节？](https://threedots.tech/post/microservices-or-monolith-its-detail/)

### Does this all apply to Go? / 这一切都适用于 Go 吗？

Open-source Go projects are usually low-level applications and infrastructure tools. This was especially true in the
early days, but it’s still hard to find good examples of Go applications that handle **domain logic**.

开源 Go 项目通常是低级应用程序和基础设施工具。这在早期尤其如此，但现在仍然很难找到处理**领域逻辑**的Go应用的好例子。

By “domain logic”, I don’t mean financial applications or complex business software. **If you develop any kind of web
application, there’s a solid chance you have some complex domain cases you need to model somehow**.

我所说的 "领域逻辑"，并不是指金融应用或复杂的商业软件。如果您开发任何类型的 Web 应用程序，您很有可能会遇到一些需要以某种方式建模的复杂领域案例。

Following some DDD examples can make you feel like you’re no longer writing Go. I’m well aware that forcing OOP patterns
straight from Java is not fun to work with. However, **there are some language-agnostic ideas that I think are worth
considering**.

跟随一些DDD的例子可以让你感觉到你不再是在写Go。我很清楚，直接从Java中强行引入OOP模式的工作并不有趣。然而，有一些语言无关的想法，我认为值得考虑。

### What’s the alternative? / 有什么选择？

We spent the last few years exploring this topic. We really like the simplicity of Go, but also had success with ideas
from Domain-Driven Design
and [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html).

我们在过去的几年里一直在探索这个话题。我们非常喜欢Go的简单性，但也在领域驱动设计和 [清洁架构](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
的理念上取得了成功。

For some reason, developers didn’t stop talking about technical debt and legacy software with the arrival of microser-
vices. Instead of looking for the silver bullet, we prefer to use business-oriented patterns together with microservices
in a pragmatic way.

出于某种原因，随着微服务的到来，开发人员并没有停止谈论技术债务和遗留软件。我们更愿意以务实的方式将面向业务的模式与微服务一起使用，而不是寻找灵丹妙药。

We’re not done with the refactoring of Wild Workouts yet. In [Clean Architecture (Chapter 9)](./chapter09.md), we will
see how to introduce Clean Architecture to the project.

我们还没有完成对 Wild Workouts 的重构。在 [Clean Architecture（第9章）](./chapter09.md)中，我们将看到如何将 Clean Architecture 引入项目。