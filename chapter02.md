# Building a serverless application with Google Cloud Run and Firebase 使用 Google Cloud Run 和 Firebase 构建无服务器应用程序

Robert Laszczak 罗伯特·拉斯扎克

Welcome to the first chapter covering how to build business-oriented applications in Go! In this book, we want to show
you how to build applications that are easy to develop, maintain, and fun to work with in the long term.

欢迎来到第一章，介绍如何在 Go 中构建面向业务的应用程序！在本书中，我们想向您展示如何构建易于开发、维护和长期使用的应用程序。

The idea of this book is to not focus too much on infrastructure and implementation details. But we need to have some
base on which we can build later. In this chapter, we start by covering some basic tools from Google Cloud that can help
us to do that.

本书的想法是不要过多地关注基础设施和实施细节。但我们需要有一些基础，以便我们以后可以建立。在这一章中，我们首先介绍 Google Cloud 的一些基本工具，它们可以帮助我们做到这一点。

Why serverless? 为什么选择 serverless？

**Running a Kubernetes cluster requires a lot of support from “DevOps teams”.** Let’s skip the fact that DevOps is not a
job title for now.

**运行一个Kubernetes集群需要 "DevOps团队 "的大量支持**。让我们暂时跳过DevOps不是一个工作头衔这一事实。

![Figure 2.1](./chapter02/ch0201.png)

Figure 2.1: DevOps is a culture and mindset. It is not a job title! Slide from The gordian knot - Alberto Brandolini

Figure 2.1: DevOps是一种文化和思维方式。它不是一个工作头衔! 幻灯片来自于《戈尔迪安结》 - 阿尔贝托·布兰多里尼

Small applications that can be easily run on one virtual machine are now being deployed on super complex Kubernetes
clusters. All these clusters require a lot of maintenance.

可以很容易地在一台虚拟机上运行的小型应用程序，现在被部署在超级复杂的Kubernetes集群上。所有这些集群都需要大量的维护。

On the other hand, moving applications to containers has given us much flexibility in building and deploying them. It
allowed us to do rapid deployments of hundreds of microservices with a lot of autonomy. But the cost for that is high.

另一方面，将应用程序转移到容器中，使我们在构建和部署它们时有了很大的灵活性。它使我们能够对数百个微服务进行快速部署，并具有很大的自主性。但这样做的成本很高。

Wouldn’t it be great if any fully managed solution existed?

如果存在一个完全托管的解决方案，那不是很好吗？

Maybe your company is already using a managed Kubernetes cluster. If so, you probably already know that even your
managed cluster still requires a ton of “DevOps” support.

也许您的公司已经在使用托管 Kubernetes 集群。如果是这样，您可能已经知道，即使您的托管集群仍然需要大量的“DevOps”支持。

Maybe serverless? Well, splitting a big application to multiple, independent Lambdas (Cloud Functions) is a great way to
an unmaintainable cataclysm.

也许无服务器？好吧，将一个大型应用程序拆分为多个独立的 Lambda（云函数）是解决不可维护灾难的好方法。

![Figure 2.2](./chapter02/ch0202.png)

Figure 2.2: You should probably rethink your architecture if it can be used for summoning demons. But wait, is it the
only way to build serverless applications? No!

Figure 2.2: 如果它可以用于召唤恶魔，您可能应该重新考虑您的架构。但是等等，这是构建无服务器应用程序的唯一方法吗？不！ // 如果它未来可能带来灾难，您可能应该重新考虑您的架构

### Google Cloud Run

The idea of Google Cloud Run is pretty simple - you **just need to provide a Docker container, and Google Cloud runs
it.**

Google Cloud Run的理念非常简单--**你只需要提供一个Docker容器，Google Cloud就可以运行它**。

Inside this container, you can run an application written in any language that can expose a port with your HTTP or gRPC
API.

在这个容器内，你可以运行一个用任何语言编写的应用程序，它可以用你的 HTTP 或 gRPC API 暴露一个端口。

You are not limited to synchronous processing – you can process Pub/Sub messages inside of this container.

你并不局限于同步处理--你可以在这个容器中处理Pub/Sub消息。

**And that’s all that you need from the infrastructure side. Google Cloud does all the magic**. Based on the traﬀic, the
container will automatically scale up and down. Sounds like a perfect solution ?

**这就是你在基础设施方面所需要的一切。谷歌云做所有的魔法**。 基于traﬀic，容器将自动扩大和缩小。听起来像是一个完美的解决方案？

In practice, it is not so simple. There are many articles showing how to use Google Cloud Run, but they usually show
**small bricks that may be used for building an application**.

在实践中，事情并不是那么简单。有很多文章展示了如何使用 Google Cloud Run，但它们通常显示 **可用于构建应用程序的小部分**。

**It’s hard to join all these bricks from multiple places to create a fully working project** (been there, done that).

**很难将所有这些来自多个地方的砖块连接起来，形成一个完整的工作项目**(亲身经历过)。

In most cases, these articles leave out the problem of vendor lock-in. The deployment method should be just an
implementation detail. I’ve already covered this topic in
the [Why using Microservices or Monolith can be just a detail?](https://threedots.tech/post/microservices-or-monolith-its-detail/)
article in 2018.

在大多数情况下，这些文章忽略了厂商锁定的问题。部署方法应该只是一个实施细节。我已经在2018年的 [《为什么使用微服务或单片机可以只是一个细节？》](https://threedots.tech/post/microservices-or-monolith-its-detail/)一文中谈到了这个话题。

But what is most important – using all the newest and most shiny technologies doesn’t mean that your application will
not become a hated legacy in the next 3 months.

但最重要的是——使用所有最新、最耀眼的技术并不意味着您的应用程序在未来 3 个月内不会成为令人讨厌的遗产。

**Serverless solves only infrastructure challenges. It doesn’t stop you from building an application that is hard to
maintain**. I even have the impression that it’s the opposite – all these fancy applications sooner or later are the
hardest to maintain.

无服务器仅解决基础架构挑战。它无法阻止您构建难以维护的应用程序。我甚至觉得情况恰恰相反——所有这些花哨的应用程序迟早会变成最难维护的。

**For this book, we created a fully functional, real-life application. You can deploy this application with one command
to Google Cloud using Terraform. You can run the local copy with one docker-compose command.**

在本书中，我们创建了一个功能齐全的真实应用程序。你可以使用 Terraform，用一条命令将这个应用程序部署到谷歌云。你可以用一条 docker-compose 命令来运行本地副本。

There is also one thing that we are doing differently than others. **We included some subtle issues, that from our
observations are common in Go projects.** In the long term, these small issues become critical and stop us from adding
new features.

还有一件事我们正在做的事情与其他事情不同。我们包括了一些微妙的问题，根据我们的观察，这些问题在 Go 项目中很常见。从长远来看，这些小问题变得至关重要，并阻止我们添加新功能。

Have we lost our minds? Not yet. **This approach will help you to understand what issues you can solve and what
techniques can help.** It is also a kind of challenge for practices that we use. If something is not a problem, why
should we use any technique to solve it?

我们失去了理智吗？还没有。这种方法将帮助你了解你可以解决哪些问题，哪些技术可以帮助你。这也是对我们使用的做法的一种挑战。如果某件事不是问题，我们为什么要用任何技术来解决它？

### Plan 方案

In the next couple of chapters, we will cover all topics related to running the application on Google Cloud. In this
part, we didn’t add any issues or bad practices. **The first chapters may be a bit basic if you already have some
experience in Go. We want to ensure that if you are just starting with Go, you will be able to follow more complex
topics that come next.**

在接下来的几章中，我们将介绍与在 Google Cloud 上运行应用程序相关的所有主题。在这一部分中，我们没有添加任何问题或不良做法。如果你已经有一些 Go 经验，前几章可能有点基础。我们希望确保如果您刚开始使用
Go，您将能够关注接下来出现的更复杂的主题。

Next, we will refactor parts of the application handling business logic. **This part will be much more complex.**

接下来，我们将重构应用程序中处理业务逻辑的部分。这一部分会复杂得多。

### Running the project locally 在本地运行项目

The ability to run a project locally is critical for eﬀicient development. It is very annoying when you are not able to
check changes that you did in a simple and fast way.

在本地运行一个项目的能力对于高效的开发来说至关重要。如果你不能以简单而快速的方式检查你所做的修改，那是非常令人讨厌的。

It’s much harder to achieve it for projects built from hundreds of microservices. Fortunately, our project has only 5
services. In Wild Workouts we created Docker Compose with live code reloading for both frontend and backend. For the
frontend, we are using a container with vue-cli-service serve tool. For the backend, the situation is a bit more
complex. In all containers, we are running the reflex tool. reflex listens for any code changes that trigger the
recompilation of the service. If you are interested in details, you can find them in
our [Go Docker dev environment with Go Modules and live code reloading](https://threedots.tech/post/go-docker-dev-environment-with-go-modules-and-live-code-reloading)
blog post.

对于由数百个微服务构建的项目来说，实现它要困难得多。幸运的是，我们的项目只有 5 个服务。在 Wild Workouts 中，我们为前端和后端创建了带有热加载的 Docker Compose。对于前端，我们使用带有
vue-cli-service 服务工具的容器。对于后端，情况要复杂一些。在所有容器中，我们都在运行 reflex 工具。 reflex 侦听任何触发重新编译服务的代码更改。如果您对细节感兴趣，您可以在我们的
[带有Go Modules和 热加载 的Go Docker开发环境](https://threedots.tech/post/go-docker-dev-environment-with-go-modules-and-live-code-reloading)
博客文章找到它们。

#### Requirements 要求

The only requirements needed to run the project are [Docker](https://www.docker.com/)
and [Docker Compose](https://docs.docker.com/compose/).

运行该项目所需的唯一要求是 [Docker](https://www.docker.com/) 和 [Docker Compose](https://docs.docker.com/compose/)。

#### Running

```shell
git clone https://github.com/ThreeDotsLabs/wild-workouts-go-ddd-example.git && cd wild-workouts-go-ddd-example
```

And run Docker Compose:
运行 Docker Compose:

```shell
docker-compose up
```

After downloading all JS and Go dependencies, you should see a message with the frontend address:

下载所有 JS 和 Go 依赖项后，您应该会看到一条带有前端地址的消息：

```shell
web_1             | $ vue-cli-service serve
web_1             |  INFO  Starting development server...
web_1             |  DONE  Compiled successfully in 6315ms11:18:26 AM
web_1             |
web_1             |
web_1             |   App running at:
web_1             |   - Local:   http://localhost:8080/
web_1             |
web_1             |   It seems you are running Vue CLI inside a container.
web_1             |   Access the dev server via http://localhost:<your container's external mapped port>/
web_1             |
web_1             |   Note that the development build is not optimized.
web_1             |   To create a production build, run yarn build.
```

Congratulations! Your local version of the Wild Workouts application is available
at [http://localhost:8080/](http://localhost:8080/).

恭喜你！Wild Workouts 应用程序的运行在本地[http://localhost:8080/](http://localhost:8080/)。

There is also a public version available
at [https://threedotslabs-wildworkouts.web.app](https://threedotslabs-wildworkouts.web.app/).

这里也有一个公开的版本，可在[https://threedotslabs-wildworkouts.web.app](https://threedotslabs-wildworkouts.web.app/)。

### What Wild Workouts can do? Wild Workouts 能做什么？

How often did you see tutorials without any real-life functionality? How often didn’t patterns from these tutorials work
in real projects? **Probably too often. Real life is not as simple as in tutorials**.

您多久看到一次没有任何实际功能的教程？这些教程中的模式在实际项目中不起作用的频率如何？**可能太频繁了。现实生活并不像教程中那么简单**。

To avoid this problem, we created Wild Workouts application as a fully functional project. It’s much harder to do
shortcuts and skip extra complexity when an application needs to be complete. It makes all chapters longer, but there
are no shortcuts here. **If you don’t spend enough time in the beginning, you will lose much more time later during the
implementation. Or even worse – you’ll be fixing problems in a rush with the application already running on
production.**

为了避免这个问题，我们将 Wild Workouts
应用程序创建为一个功能齐全的项目。当一个应用程序需要完整时，做捷径和跳过额外的复杂性要难得多。它使所有章节都变长了，但这里没有捷径。如果您在开始时没有花费足够的时间，那么您将在稍后的实施过程中浪费更多时间。或者更糟糕的是 ——
你会在已经在生产环境中运行的应用程序中匆忙解决问题。

#### tl;dr

Wild Workouts is an application for personal gym trainers and attendees.

Wild Workouts 是一款针对私人健身教练和参与者的应用程序。

![Figure 2.3: Login page](./chapter02/ch0203.png)

Figure 2.3: 登陆页

Trainers can set a schedule when they are available for the training.

培训师可以设定一个他们可以参加培训的时间表。

Attendees can schedule training for provided dates.

参加者可以在提供的日期安排培训

Other functionalities are:

其他功能还有

- management of “credits” (how many trainings the attendee can schedule)
  “学分”管理（参加者可以安排多少次培训）
- cancellation
  取消
    * if a training is canceled less than 24 hours before the time it begins, the attendee will not receive their
      credits back
      如果在培训开始前24小时内取消培训，参训者将不会收他们的学分。

![Figure 2.4: Schedule](./chapter02/ch0204.png)
Figure 2.4: Schedule

Figure 2.4: 日程

![Figure 2.5: Schedule training](./chapter02/ch0205.png)
Figure 2.5: Schedule training

Figure 2.5: 安排培训时间

- training reschedule
  培训重新安排时间
    * if someone wants to reschedule a training less than 24 hours before the time it begins, it needs to be approved by
      the second participant (trainer or attendee)
      如果有人想在培训开始前24小时内重新安排培训时间，需要得到第二位参与者（培训师或参会者）的批准。
- calendar view
  浏览日程表
Sounds simple. What can go wrong?

### Frontend 前端

If you are not interested in the frontend part, you can go straight to the Backend section.

I’m rather a backend engineer and to be honest I’m not the greatest JavaScript and frontend specialist. **But we can’t
have an application working end-to-end without a frontend!**

In this book, we focus on the backend part. I will give a high-level overview of what technologies I used on the
frontend. It will be nothing new to you if you have any basic frontend knowledge. For more details, I recommend to check
the source code in the web/ [directory](https://bit.ly/3uw3xtB).

### OpenAPI (Swagger) client

Nobody likes to keep API contracts up to date manually. It’s annoying and counterproductive to keep multiple bor- ing
JSONs up-to-date. OpenAPI solves this problem with JavaScript HTTP client and Go HTTP server generated from the provided
[specification](https://bit.ly/37D3hPE). We will dive into details in the Backend part.

![](./chapter02/ch0206.png)
Figure 2.6: OpenAPI

### Bootstrap

You probably already know [Bootstrap](https://getbootstrap.com/), the greatest friend of every backend engineer, like
me. Fighting with HTML and CSS is the part of frontend development that I dislike the most. Bootstrap provided me almost
all building blocks needed for creating HTML of the application.

![](./chapter02/ch0207.png)

Figure 2.7: Bootstrap

### Vue.js

After checking a couple of the most popular frontend frameworks, I decided to use [Vue.js](https://vuejs.org/v2/guide/).
I really enjoyed the simplicity of this solution.
![](./chapter02/ch0208.png)
Figure 2.8: Vue.js

I was starting my journey as a full-stack developer in the pre-jQuery times. The frontend tooling made huge progress...
but I’ll stay with the backend for now.

## Backend

The backend of Wild Workouts is built from 3 services.

- *trainer* – provides public [HTTP](https://bit.ly/3aGUC0b) and internal [gRPC endpoints](https://bit.ly/3aGUC0b) for
  managing trainer schedule
- *trainings* – provides public [HTTP](https://bit.ly/3upucIc) for managing attendee trainings
- *users* – provides public [HTTP endpoints](https://bit.ly/3uh4rtA) and
  internal [gRPC endpoints](https://bit.ly/3uh4rtA) , manages credits and user data

If a service exposes 2 types of APIs, each of them is exposed in a separate process.

### Public HTTP API

Most operations performed by applications are triggered by the public HTTP API. I’ve heard many times the question from
newcomers to Go what framework they should use to create an HTTP service. **I always advise against using any kind of
HTTP framework in Go. A simple router, like [chi](https://github.com/go-chi/chi) is more than enough.** chi provides us
only the lightweight glue to define what URLs and methods are supported by our API. Under the hood, it uses the Go
standard library http package, so all related tools like middlewares are 100% compatible.

It may be a bit weird to not use a framework if you are coming from any language where Spring, Symfony, Django, or
Express may be the obvious choice. It was weird for me too. Using any kind of framework in Go adds unnecessary
complexity and will couple your project with this framework. [KISS](https://en.wikipedia.org/wiki/KISS_principle).

All the services are running the HTTP server in the same way. It sounds like a good idea to not copy it 3 times.

```go
package server

func RunHTTPServer(createHandler func(router chi.Router) http.Handler) {
	apiRouter := chi.NewRouter()
	setMiddlewares(apiRouter)
	rootRouter := chi.NewRouter()
	// we are mounting all APIs under /api path
	rootRouter.Mount("/api", createHandler(apiRouter))
	logrus.Info("Starting HTTP server")
	http.ListenAndServe(":"+os.Getenv("PORT"), rootRouter)
}
```

Source: http.go on [GitHub](https://bit.ly/3bonCsL)

chi provides us with a set of useful built-in HTTP middlewares, but we are not limited only to them. All middlewares
compatible with Go standard library will work.

> Long story short – middlewares allow us to do anything before and after a request is executed (with access to the http.Request). Using HTTP middlewares gives us a lot of flexibility in building our custom HTTP server. We are building our server from multiple decoupled components that you can customize for your purposes.

```go
package server

func setMiddlewares(router *chi.Mux) {
	router.Use(middleware.RequestID)
	router.Use(middleware.RealIP)
	router.Use(logs.NewStructuredLogger(logrus.StandardLogger()))
	router.Use(middleware.Recoverer)

	addCorsMiddleware(router)
	addAuthMiddleware(router)

	router.Use(
		middleware.SetHeader("X-Content-Type-Options", "nosniff"),
		middleware.SetHeader("X-Frame-Options", "deny"),
	)
	router.Use(middleware.NoCache)
}
```

Source: http.go on [GitHub](https://bit.ly/3aFCFzn)

We have our framework almost ready now. :) It’s time to use that. We can call server.RunHTTPServer in the trainings
service.

```go
package main

// ...
func main() {
	// ...
	server.RunHTTPServer(func(router chi.Router) http.Handler {
		return HandlerFromMux(HttpServer{firebaseDB, trainerClient, usersClient}, router)
	})
}
```

Source: main.go on [GitHub](https://bit.ly/3k63dNf)

createHandler needs to return http.Handler. In our case it is HandlerFromMux generated
by [oapi-codegen](https://github.com/deepmap/oapi-codegen).

It provides us all the paths and query parameters from the [OpenAPI specs](https://bit.ly/37D3hPE).

```go
package main

// HandlerFromMux creates http.Handler with routing matching OpenAPI spec based on the provided mux.
func HandlerFromMux(si ServerInterface, r chi.Router) http.Handler {
	r.Group(func(r chi.Router) {
		r.Use(GetTrainingsCtx)
		r.Get("/trainings", si.GetTrainings)
	})
	r.Group(func(r chi.Router) {
		r.Use(CreateTrainingCtx)
		r.Post("/trainings", si.CreateTraining)
	})
	// ...
	return r
}
```

Source: openapi_api.gen.go on [GitHub](https://bit.ly/3bqtTUN)

```yaml
# ...
paths:
  /trainings:
    get:
      operationId: getTrainings
      responses:
        '200':
          description: todo
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Trainings'
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
# ...
```

Source: trainings.yml on [GitHub](https://bit.ly/3pFKRDI)

If you want to make any changes to the OpenAPI spec, you need to regenerate Go server and JavaScript clients after. You
need to run:

```shell
make openapi
```

Part of the generated code is ServerInterface. It contains all methods that need to be supported by the API.
Implementation of server functionality is done by implementing that interface.

```go
package main

type ServerInterface interface {
	// (GET /trainings)
	GetTrainings(w http.ResponseWriter, r *http.Request)
	// (POST /trainings)
	CreateTraining(w http.ResponseWriter, r *http.Request)
	// ...
}
```

Source: openapi_api.gen.go on [GitHub](https://bit.ly/3pInFol)

This is an example of how trainings.HttpServer is implemented:

```go
package main

import (
	"github.com/go-chi/render"
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/auth"
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/genproto/trainer"
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/genproto/users"
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/server/httperr"
	"net/http"
)

type HttpServer struct {
	db            db
	trainerClient trainer.TrainerServiceClient
	usersClient   users.UsersServiceClient
}

func (h HttpServer) GetTrainings(w http.ResponseWriter, r *http.Request) {
	user, err := auth.UserFromCtx(r.Context())
	if err != nil {
		httperr.Unauthorised("no-user-found", err, w, r)
		return
	}
	trainings, err := h.db.GetTrainings(r.Context(), user)
	if err != nil {
		httperr.InternalError("cannot-get-trainings", err, w, r)
		return
	}
	trainingsResp := Trainings{trainings}
	render.Respond(w, r, trainingsResp)
}
```

Source: http.go on [GitHub](https://bit.ly/3ukpVWH)

But HTTP paths are not the only thing generated from OpenAPI spec. More importantly, it provides us also the models for
responses and requests. Models are, in most cases, much more complex than API paths and methods. Generating them can
save time, issues, and frustration during any API contract changes.

```yaml
# ...

schemas:
  Training:
    type: object
    required: [ uuid, user, userUuid, notes, time, canBeCancelled, moveRequiresAccept ]
    properties:
      uuid:
        type: string
        format: uuid
      user:
        type: string
        example: Mariusz Pudzianowski
      userUuid:
        type: string
        format: uuid
      notes:
        type: string
        example: "let's do leg day!"
      time:
        type: string
        format: date-time
      canBeCancelled:
        type: boolean
      moveRequiresAccept:
        type: boolean
      proposedTime:
        type: string
        format: date-time
      moveProposedBy:
        type: string

  Trainings:
    type: object
    required: [ trainings ]
    properties:
      trainings:
        type: array
        items:
          $ref: '#/components/schemas/Training'
# ...
```

Source: trainings.yml on [GitHub](https://bit.ly/2P0KI1n)

```go
package main

import "time"

// Training defines model for Training.
type Training struct {
	CanBeCancelled     bool       `json:"canBeCancelled"`
	MoveProposedBy     *string    `json:"moveProposedBy,omitempty"`
	MoveRequiresAccept bool       `json:"moveRequiresAccept"`
	Notes              string     `json:"notes"`
	ProposedTime       *time.Time `json:"proposedTime,omitempty"`
	Time               time.Time  `json:"time"`
	User               string     `json:"user"`
	UserUuid           string     `json:"userUuid"`
	Uuid               string     `json:"uuid"`
}

// Trainings defines model for Trainings.
type Trainings struct {
	Trainings []Training `json:"trainings"`
}
```

Source: openapi_types.gen.go on [GitHub](https://bit.ly/3k7NYDj)

### Cloud Firestore database

All right, we have the HTTP API. But even the best API without any data and without the ability to save anything is
useless.

If we want to build the application in the most modern, scalable, and truly serverless way, Firestore is a natural
choice. We will have that out of the box. What is the cost of that?

![](./chapter02/ch0209.png)
Figure 2.9: Firestore

If we are talking about financial cost for
the [Europe multi-region](https://firebase.google.com/docs/firestore/pricing#europe) option we need to pay:

- $0.06 per 100,000 documents reads
- $0.18 per 100,000 documents writes
- $0.02 per 100,000 documents deletes
- $0.18/GiB of stored data/month

Sounds pretty cheap?

For comparison, let’s take the cheapest [Cloud SQL MySQL](https://cloud.google.com/sql/pricing#2nd-gen-pricing)
db-f1-micro instance with shared Virtual CPU and 3 GB of storage as a reference – it costs $15.33/month. The cheapest
instance with high availability and with 1 non-shared Virtual CPU costs $128.21/month

What is even better, in the [free plan](https://firebase.google.com/pricing), you can store up to 1 GiB of data with 20k
document writes per day.

Firestore is a NoSQL database, so we should not expect to build relational models in the SQL manner. Instead of that, we
have a system of [hierarchical collections](https://firebase.google.com/docs/firestore/data-model). In our case, the
data model is pretty simple, so we have only one level of collections.

In contrast to many NoSQL databases, Firestore offers ACID transactions on any operation. It also works when updating
multiple documents.

#### Firestore limitations

The important limitation may be the limit of 1 update / second / one document. It still means that you can update a lot
of independent documents in parallel. It is an important factor that you should consider while designing your database.
In some cases, you should consider batching operations, different documents design, or using a different database. If
data is changing often, maybe a key-value database would be a good choice?

From my experience, the limitation of 1 update per second per document is not a serious problem. In most cases, when I
was using Firestore, we were updating a lot of independent documents. This is also true

![](./chapter02/ch0210.png)

Figure 2.10: Firestore Console

when using Firestore for event sourcing - you will only use append operations. In Wild Workouts, we should also not have
a problem with this limit.
> I have also observed that Firestore needs some time to warm up. In different words – if you want to insert 10 mln documents within one minute after you just set up a new project, it may not work. I guess this is related to some internal magic that handles scaleability. Fortunately, in the real world, it is not common to have traﬀic spikes from 0 to 10 mln writes / minute.

#### Running Firestore locally

Unfortunately Firestore emulator is not perfect.

I found some situations where the emulator was not 100% compatible with the real version. I also had some situations
when I was doing an update and read of the same document in the transaction, and it caused a deadlock. From my point of
view, this functionality is enough for local development.

The alternative may be to have a separate Google Cloud project for local development. My preference here is to have a
local environment that is truly local and doesn’t depend on any external services. It is also easier to set up and can
be used later in Continuous Integration.

**Since the end of May, the Firestore emulator provides a UI**. It is added to the Docker Compose and is available
at http://localhost:4000/. When I’m writing this, sub-collections are not displayed properly32 in the UI in the
emulator. Don’t worry, for Wild Workouts it’s not a problem.

#### Using Firestore

Apart from Firestore implementation, the code works in the same way locally and on the production. When we are using
emulator locally, we need to run our application with env FIRESTORE_EMULATOR_HOST set to emulator hostname (in our case
firestore:8787). It is set in the .env file

On production, all magic is done by Google Cloud under the hood, and no extra configuration is needed.

```go
package main

func main() {
	// ...

	firebaseClient, err := firestore.NewClient(ctx, os.Getenv("GCP_PROJECT"))
	if err != nil {
		panic(err)
	}

	// ...
}
```

Source: main.go on [GitHub](https://bit.ly/3bnhTDC)

Here is the example of how I used Firestore client for querying trainer schedule. You can see how I
used [queries functionality]() to get only dates from the queried dates interval.

```go
package main

import (
	// ...
	"cloud.google.com/go/firestore"
	// ...
)

// ...
type db struct {
	firestoreClient *firestore.Client
}

func (d db) TrainerHoursCollection() *firestore.CollectionRef {
	return d.firestoreClient.Collection("trainer-hours")
}

// ...
func (d db) QueryDates(params *GetTrainerAvailableHoursParams, ctx context.Context) ([]Date, error) {
	iter := d.
		TrainerHoursCollection().
		Where("Date.Time", ">=", params.DateFrom).
		Where("Date.Time", "<=", params.DateTo).
		Documents(ctx)
	var dates []Date

	for {
		doc, err := iter.Next()
		if err == iterator.Done {
			break
		}
		if err != nil {
			return nil, err
		}

		date := Date{}
		if err := doc.DataTo(&date); err != nil {
			return nil, err
		}
		date = setDefaultAvailability(date)
		dates = append(dates, date)
	}

	return dates, nil
}
```

Source: firestore.go on [GitHub](https://bit.ly/3btJPG9)

There is also no extra data mapping needed. Firestore library can marshal any struct with public fields or
`map[string]interface`. As long as there is nothing weird inside. You can find the entire specification of how
conversion is done in
cloud.google.com/go/firestore [GoDoc](https://godoc.org/cloud.google.com/go/firestore#DocumentSnapshot.DataTo).

```go
package main

type Date struct {
	Date         openapi_types.Date `json:"date"`
	HasFreeHours bool               `json:"hasFreeHours"`
	Hours        []Hour             `json:"hours"`
}
```

Source: openapi_types.gen.go on [GitHub](https://bit.ly/3pBnSd3)

```
date := Date{}
if err := doc.DataTo(&date); err != nil {
    return nil, err
}
```

Source: firestore.go on [GitHub](https://bit.ly/3k7rPoM)

### Production deployment tl;dr

You can deploy your own version of Wild Workouts with one command:

```shell
cd terraform/
make

Fill all required parameters:
    project [current: wild-workouts project]:       # <----- put your Wild Workouts Google Cloud project name here (it will be created) 
    user [current: email@gmail.com]:                # <----- put your Google (Gmail, G-suite etc.) e-mail here
    billing_account [current: My billing account]:  # <----- your billing account name, can be found here https://console.cloud.google.com/billing
    region [current: europe-west1]: 
    firebase_location [current: europe-west]: 

# it may take a couple of minutes...

The setup is almost done!

Now you need to enable Email/Password provider in the Firebase console.
To do this, visit https://console.firebase.google.com/u/0/project/[your-project]/authentication/providers

You can also downgrade the subscription plan to Spark (it's set to Blaze by default).
The Spark plan is completely free and has all features needed for running this project.

Congratulations! Your project should be available at: https://[your-project].web.app

If it's not, check if the build finished successfully: https://console.cloud.google.com/cloud-build/builds?project=[your-project]

If you need help, feel free to contact us at https://threedots.tech

```

We will describe the deployment in detail in the next chapters.

### What’s next?

In the next chapters, we will cover internal gRPC communication between services and HTTP Firebase authentication.

The code with gRPC communication and authentication is already on our GitHub. Feel free to read, run, and experiment
with it.

#### Deployment and infrastructure

The infrastructure part is described in Chapter 14. It covers in details:

- Terraform
- Cloud Run
- CI/CD
- Firebase hosting

#### What is wrong with this application?!

After finishing all chapters describing the current application, we will start the part related with refactoring and
adding new features to Wild Workouts.

**We don’t want to use many fancy techniques just to make our CVs look better. Our goal is to solve issues present in
the application using** Domain-Driven Design, Clean Architecture, CQRS, Event Storming, and Event Modeling.

We will do it by refactoring. It should be then visible **what issues are solved and how it makes the imple- mentation
more clean**. Maybe during this we will even see that some of these techniques are not useful in Go, who knows.