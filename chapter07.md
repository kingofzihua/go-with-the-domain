## The Repository Pattern / 存储模式

Robert Laszczak / 罗伯特·拉斯扎克

I’ve seen a lot of complicated code in my life. Pretty often, the reason of that complexity was application logic
coupled with database logic. **Keeping logic of your application along with your database logic makes your application
much more complex, hard to test, and maintain.**

在我的生活中，我见过很多复杂的代码。很多时候，这种复杂性的原因是应用逻辑与数据库逻辑的结合。**把你的应用程序的逻辑和你的数据库逻辑放在一起，使你的应用程序更加复杂，难以测试和维护。**

There is already a proven and simple pattern that solves these issues. The pattern that allows you to **separate your
application logic from database logic**. It allows you to **make your code simpler and easier to add new
functionalities**. As a bonus, you can **defer important decision** of choosing database solution and schema. Another
good side effect of this approach is out of the box **immunity for database vendor lock-in**. The pattern that I have in
mind is repository.

已经有一个经过验证的简单模式来解决这些问题。该模式允许你将你的**应用逻辑与数据库逻辑分开**
。它允许您使您的代码更简单，更容易添加新功能。作为奖励，您可以推迟选择数据库解决方案和架构的重要决定。这种方法的另一个好的副作用是对数据库供应商锁定的开箱即用免疫。我想到的模式是存储库。

When I’m going back in my memories to the applications I worked with, I remember that it was tough to understand how
they worked. **I was always afraid to make any change there – you never know what unexpected, bad side effects it could
have.** It’s hard to understand the application logic when it’s mixed with database implementation. It’s also a source
of duplication.

当我回忆起我使用过的应用程序时，我记得很难理解它们是如何工作的。我总是害怕在那里做出任何改变——你永远不知道它会产生什么意想不到的不良副作用。当应用程序的逻辑与数据库的实现混在一起时，变得很难理解。这也是重复的来源。

Some rescue here may be
building [end-to-end tests](https://martinfowler.com/articles/microservice-testing/#testing-end-to-end-introduction).
But it hides the problem instead of really solving it. Having a lot of E2E tests is slow, flaky, and hard to maintain.
Sometimes they even prevent us from creating new functionality, rather than help.

有一些补救方式可能是构建[端到端测试](https://martinfowler.com/articles/microservice-testing/#testing-end-to-end-introduction)
。但它隐藏了问题，而不是真正解决它。进行大量 E2E 测试是缓慢、不稳定且难以维护的。有时它们甚至阻止我们创建新功能，而不是提供帮助。

In this chapter, I will teach you how to apply this pattern in Go in a pragmatic, elegant, and straightforward way. I
will also deeply cover a topic that is often skipped - **clean transactions handling**

在这一章中，我将教你如何以一种务实、优雅和直接的方式在Go中应用这种模式。我还将深入介绍一个经常被忽略的话题 - **clean transactions handling.**.

To prove that I prepared 3 implementations: **Firestore, MySQL, and simple in-memory**.

为了证明这一点，我准备了3个实现方案。**Firestore, MySQL, and simple in-memory**。

Without too long introduction, let’s jump to the practical examples!

没有太长的介绍，直接上实例!

### Repository interface / 仓储接口

The idea of using the repository pattern is:

使用仓储模式的想法是：

Let’s abstract our database implementation by defining interaction with it by the interface. You need to be able to use
this interface for any database implementation – that means that it should be free of any implementation details of any
database.

让我们通过定义接口与数据库的交互来抽象出我们的数据库实现。你需要能够对任何数据库的实现使用这个接口--这意味着它应该不受任何数据库的实现细节的影响。

Let’s start with the refactoring of [trainer](https://bit.ly/2NyKVbr) service. Currently, the service allows us to get
information about hour availability [via HTTP API](https://bit.ly/2OXsgGB) and [via gRPC](https://bit.ly/2OXsgGB). We
can also change the availability of the hour [via HTTP API](https://bit.ly/3unIQzJ) and [gRPC](https://bit.ly/3unIQzJ) .

让我们从 [trainer](https://bit.ly/2NyKVbr) 服务的重构开始。目前，该服务允许我们通过[via HTTP API](https://bit.ly/2OXsgGB)
和 [via gRPC](https://bit.ly/2OXsgGB) 来获取小时可用性的信息。我们还可以通过 [via HTTP API](https://bit.ly/3unIQzJ)
和 [gRPC](https://bit.ly/3unIQzJ) 改变小时的可用性。

In the previous chapter, we refactored Hour to use DDD Lite approach. Thanks to that, we don’t need to think about
keeping rules of when Hour can be updated. Our domain layer ensures that we can’t do anything “stupid”. We also don’t
need to think about any validation. We can just use the type and execute necessary operations.

在上一章中，我们重构了Hour，以使用DDD Lite方法。得益于此，我们不需要考虑保留Hour何时能被更新的规则。我们的领域层确保我们不会做任何 "傻事"。我们也不需要考虑任何验证的问题。我们只需使用该类型并执行必要的操作。

We need to be able to get the current state of Hour from the database and save it. Also, in case when two people would
like to schedule a training simultaneously, only one person should be able to schedule training for one hour.

我们需要能够从数据库中获取 Hour 的当前状态并保存。此外，如果两个人想同时安排培训，则应该只有一个人可以安排一小时的培训。

Let’s reflect our needs in the interface:

让我们在界面中反映我们的需求：

```go
package hour

type Repository interface {
	GetOrCreateHour(ctx context.Context, hourTime time.Time) (*Hour, error)
	UpdateHour(
		ctx context.Context,
		hourTime time.Time,
		updateFn func(h *Hour) (*Hour, error),
	) error
}
```

Source: [repository.go on GitHub](https://bit.ly/3bo6QtT)

We will use `GetOrCreateHour` to get the data, and `UpdateHour` to save the data.

我们将使用 `GetOrCreateHour` 来获取数据，并使用 `UpdateHour` 来保存数据。

We define the interface in the same package as the `Hour` type. Thanks to that, we can avoid duplication if using this
interface in many modules (from my experience, it may often be the case). It’s also a similar pattern to `io.Writer`,
where io package defines the interface, and all implementations are decupled in separate packages. How to implement that
interface?

我们将接口定义在与 `Hour` 类型相同的包中。正因为如此，如果在许多模块中使用此接口，我们可以避免重复（根据我的经验，可能经常出现这种情况）。这也是与 `io.Writer` 类似的模式，其中 io
包定义了接口，所有实现都在单独的包中解耦。如何实现该接口？

### Reading the data / 读取数据

Most database drivers can use the `ctx context.Context` for cancellation, tracing, etc. It’s not specific to any
database (it’s a common Go concept), so you should not be afraid that you spoil the domain.

大多数数据库驱动程序可以使用 `ctx context.Context` 来取消、追踪等。这不是任何数据库所特有的（这是一个通用的Go概念），所以你不应该害怕你破坏了这个领域。

In most cases, we query data by using UUID or ID, rather than time.Time. In our case, it’s okay – the hour is unique by
design. I can imagine a situation that we would like to support multiple trainers – in this case, this assumption will
not be valid. Change to UUID/ID would still be simple. But for now,
[YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)!

在大多数情况下，我们通过使用UUID或ID来查询数据，而不是使用time.Time。在我们的案例中，这没有问题--在设计上，小时是唯一的。我可以想象，我们希望支持多个培训师的情况--在这种情况下，这个假设就不成立了。改为UUID/ID仍然会很简单。
但是现在，[YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)！

For clarity – this is how the interface based on UUID may look like:

为了清楚起见--这是基于 UUID 可能看起来是这样的。

```
GetOrCreateHour(ctx context.Context, hourUUID string) (*Hour, error)
```

> You can find an example of a repository based on UUID in **Combining DDD, CQRS, and Clean Architecture** (Chapter 11).
>
> 你可以在《结合 DDD、CQRS 和清洁架构》（第11章）中找到一个基于UUID的资源库的例子。

How is the interface used in the application?

该接口在应用中是如何使用的？

```go
package main

import (
	// ...
	"github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/trainer/domain/hour"
	// ...
)

type GrpcServer struct {
	hourRepository hour.Repository
}

// ...
func (g GrpcServer) IsHourAvailable(ctx context.Context, request *trainer.IsHourAvailableRequest) (*trainer.IsHourAvailableResponse, error) {
	trainingTime, err := protoTimestampToTime(request.Time)
	if err != nil {
		return nil, status.Error(codes.InvalidArgument, "unable to parse time")
	}
	h, err := g.hourRepository.GetOrCreateHour(ctx, trainingTime)
	if err != nil {
		return nil, status.Error(codes.Internal, err.Error())
	}
	return &trainer.IsHourAvailableResponse{IsAvailable: h.IsAvailable()}, nil
}

```

Source: [grpc.go on GitHub](https://bit.ly/2ZCdmaR)

No rocket science! We get hour.Hour and check if it’s available. Can you guess what database we use? No, and that is the
point!

没有高深的东西! 我们得到 hour.Hour 并检查它是否可用。你能猜到我们使用什么数据库吗？不能，这就是问题的关键!

As I mentioned, we can avoid vendor lock-in and be able to easily swap the database. If you can swap the database,
__it’s a sign that you implemented the repository pattern correctly__ . In practice, the situation when you change the
database is rare. In case when you are using a solution that is not self-hosted (like Firestore), it’s more important to
mitigate the risk and avoid vendor lock-in.

正如我所提到的，我们可以避免被厂商锁定，并且能够轻松地交换数据库。如果你能交换数据库，__就说明你正确地实现了资源库模式__。在实践中，你改变数据库的情况是很少的。如果你使用的是一个非自我托管的解决方案（如 Firestore
），那么降低风险和避免厂商锁定就更加重要了。

The helpful side effect of that is that we can defer the decision of which database implementation we would like to use.
I call this approach Domain First. I described it in depth in **Domain-Driven Design Lite** (Chapter 6). **Deferring the
decision about the database for later can save some time at the beginning of the project. With more informations and
context, we can also make a better decision.**

其有益的作用是，我们可以推迟决定我们想使用的数据库实现。我把这种方法称为领域优先。我在__《领域驱动设计之光》（第6章）__
中对它进行了深入的描述。将关于数据库的决定推迟到以后可以在项目开始时节省一些时间。有了更多的信息和背景，我们也可以做出更好的决定

When we use the Domain-First approach, the first and simplest repository implementation may be in-memory implementation.

当我们使用领域优先的方法时，第一个也是最简单的资源库实现可能是内存实现。

### Example In-memory implementation / 内存实现示例

Our memory uses a simple map under the hood. `getOrCreateHour` has 5 lines (without a comment and one newline)!

我们的内存在底层使用了一个简单的map。 `getOrCreateHour` 有5行（不包括注释和一个换行）！这就是我们的内存实现。

```go
package main

type MemoryHourRepository struct {
	hours       map[time.Time]hour.Hour
	lock        *sync.RWMutex
	hourFactory hour.Factory
}

func NewMemoryHourRepository(hourFactory hour.Factory) *MemoryHourRepository {
	if hourFactory.IsZero() {
		panic("missing hourFactory")
	}
	return &MemoryHourRepository{
		hours:       map[time.Time]hour.Hour{},
		lock:        &sync.RWMutex{},
		hourFactory: hourFactory,
	}
}
func (m MemoryHourRepository) GetOrCreateHour(_ context.Context, hourTime time.Time) (*hour.Hour, error) {
	m.lock.RLock()
	defer m.lock.RUnlock()
	return m.getOrCreateHour(hourTime)
}
func (m MemoryHourRepository) getOrCreateHour(hourTime time.Time) (*hour.Hour, error) {
	currentHour, ok := m.hours[hourTime]
	if !ok {
		return m.hourFactory.NewNotAvailableHour(hourTime)
	}
	// we don't store hours as pointers, but as values
	// thanks to that, we are sure that nobody can modify Hour without using UpdateHour return &currentHour, nil
	// ...
}
```

Source: [hour_memory_repository.go on GitHub](https://bit.ly/2Mc636Q)

Unfortunately, the memory implementation has some downsides. The biggest one is that it doesn’t keep the data of the
service after a restart. It can be enough for the functional pre-alpha version. To make our application
production-ready, we need to have something a bit more persistent.

不幸的是，内存实现有一些缺点。最大的一个是，它在重启后不能保留服务的数据。对于功能性的前 alpha 版本来说，这已经足够了。为了使我们的应用程序可以投入生产，我们需要有一些更持久的东西。

### Example MySQL implementation / MySQL 实现示例

We already know how our model [looks like](https://bit.ly/3btQqQD) and how [it behaves](https://bit.ly/3btQqQD). Based
on that, let’s define our SQL schema.

我们已经知道了我们的模型是 [什么样子](https://bit.ly/3btQqQD) 的，以及它的[行为方式](https://bit.ly/3btQqQD) 。在此基础上，我们来定义我们的 SQL schema。

```sql
CREATE TABLE `hours`
(
    hour         TIMESTAMP NOT NULL,
    availability ENUM ('available', 'not_available', 'training_scheduled') NOT NULL,
    PRIMARY KEY (hour)
);
```

Source: [schema.sql on GitHub](https://bit.ly/3qFUWCb)

When I work with SQL databases, my default choices are:

- [sqlx](https://github.com/jmoiron/sqlx) – for simpler data models, it provides useful functions that help with using
  structs to unmarshal query results. When the schema is more complex because of relations and multiple models, it’s
  time for...
- [SQLBoiler](https://github.com/volatiletech/sqlboiler) - is excellent for more complex models with many fields and
  relations, it’s based on code gener- ation. Thanks to that, it’s very fast, and you don’t need to be afraid that you
  passed invalid `interface{}` instead of another `interface{}`. Generated code is based on the SQL schema, so you can
  avoid a lot of duplication.

当我使用 SQL 数据库时，我的默认选择是：

- [sqlx](https://github.com/jmoiron/sqlx) – 对于较简单的数据模型，它提供了有用的函数，帮助使用结构来解读查询结果。当模式因为关系和多个模型而变得更加复杂时，就需要...
- [SQLBoiler](https://github.com/volatiletech/sqlboiler) -
  非常适合具有许多领域和关系的更复杂的模型，它基于代码生成。多亏了这一点，它非常快，您不必担心您传递了无效的 `interface{}` 而不是另一个 `interface{}` 。生成的代码基于 SQL 模式，因此可以避免大量重复。

We currently have only one table. sqlx will be more than enough . Let’s reflect our DB model, with “transport type”.

我们目前只有一个表。 sqlx将是绰绰有余的。让我们用“传输类型”来反映我们的 DB 模型。

```go
package main

type mysqlHour struct {
	ID           string    `db:"id"`
	Hour         time.Time `db:"hour"`
	Availability string    `db:"availability"`
}
```

Source: [hour_mysql_repository.go on GitHub](https://bit.ly/2NoUdqw)

> You may ask why not to add the db attribute to hour.Hour? From my experience, it’s better to entirely separate domain types from the database. It’s easier to test, we don’t duplicate validation, and it doesn’t introduce a lot of boilerplate.
>
> 你可能会问为什么不给 hour.Hour 添加 db 属性呢？根据我的经验，最好将域类型与数据库完全分开。它更容易测试，我们不会重复验证，也不会引入很多样板。
>
> In case of any change in the schema, we can do it just in our repository implementation, not in the half of the project. Miłosz described a similar case in **When to stay away from DRY** (Chapter 5). I also described that rule deeper in **Domain-Driven Design Lite** (Chapter 6).
>
> 如果模式有任何改变，我们可以只在我们的存储库实现中做，而不是在项目的一半中做。Miłosz在__《何时远离DRY》__（第5章）中描述了一个类似的情况。我也在《领域驱动设计精简版》（第6章）中更深入地描述了这个规则。


How can we use this struct?

我们如何使用这个结构？

```go
package main

// sqlContextGetter is an interface provided both by transaction and standard db connection
type sqlContextGetter interface {
	GetContext(ctx context.Context, dest interface{}, query string, args ...interface{}) error
}

func (m MySQLHourRepository) GetOrCreateHour(ctx context.Context, time time.Time) (*hour.Hour, error) {
	return m.getOrCreateHour(ctx, m.db, time, false)
}
func (m MySQLHourRepository) getOrCreateHour(ctx context.Context,
	db sqlContextGetter,
	hourTime time.Time,
	forUpdate bool,
) (*hour.Hour, error) {
	dbHour := mysqlHour{}
	query := "SELECT * FROM `hours` WHERE `hour` = ?"
	if forUpdate {
		query += " FOR UPDATE"
	}
	err := db.GetContext(ctx, &dbHour, query, hourTime.UTC())
	if errors.Is(err, sql.ErrNoRows) {
		// in reality this date exists, even if it's not persisted
		return m.hourFactory.NewNotAvailableHour(hourTime)
	} else if err != nil {
		return nil, errors.Wrap(err, "unable to get hour from db")
	}
	availability, err := hour.NewAvailabilityFromString(dbHour.Availability)
	if err != nil {
		return nil, err
	}
	domainHour, err := m.hourFactory.UnmarshalHourFromDatabase(dbHour.Hour.Local(), availability)
	if err != nil {
		return nil, err
	}
	return domainHour, nil
}
```

Source: [hour_mysql_repository.go on GitHub](https://bit.ly/3uqtFGf)

With the SQL implementation, it’s simple because we don’t need to keep backward compatibility. In previous chapters, we
used Firestore as our primary database. Let’s prepare the implementation based on that, keeping backward compatibility.

使用 SQL 实现，这很简单，因为我们不需要保持向后兼容性。在前面的章节中，我们使用 Firestore 作为我们的主数据库。让我们在此基础上准备实现，保持向后兼容性。

### Firestore implementation / Firestore 实现

When you want to refactor a legacy application, abstracting the database may be a good starting point.

当您想要重构遗留应用程序时，抽象数据库可能是一个很好的起点。

Sometimes, applications are built in a database-centric way. In our case, it’s an HTTP Response-centric approach – our
database models are based on Swagger generated models. In other words – our data models are based on Swagger data models
that are returned by API. Does it stop us from abstracting the database? Of course not! It will need just some extra
code to handle unmarshaling.

有时，应用程序是以数据库为中心的方式构建的。在我们的例子中，它是一种以 HTTP 响应为中心的方法 —— 我们的数据库模型基于 Swagger 生成的模型。换句话说，我们的数据模型基于 API 返回的 Swagger
数据模型。它会阻止我们抽象数据库吗？当然不是！它只需要一些额外的代码来处理解组。

**With Domain-First approach, our database model would be much better, like in the SQL implemen- tation.** But we are
where we are. Let’s cut this old legacy step by step. I also have the feeling that CQRS will help us with that.

__有了领域优先的方法，我们的数据库模型会好得多，就像在SQL实现中一样。__ 但我们现在所处的位置。让我们一步一步地削减这个旧的遗产。我也感觉到，CQRS将帮助我们实现这一目标。

> In practice, a migration of the data may be simple, as long as no other services are integrated directly via the database.
>
> 在实践中，数据的迁移可能很简单，只要没有其他服务直接通过数据库集成即可。
>
> Unfortunatly, it’s an optimistic assumption when we work with a legacy response/database-centric or CRUD service...
>
> 不幸的是，当我们使用遗留响应/以数据库为中心或 CRUD 服务时，这是一个乐观的假设......
>

```go
package main

func (f FirestoreHourRepository) GetOrCreateHour(ctx context.Context, time time.Time) (*hour.Hour, error) {
	date, err := f.getDateDTO(
		// getDateDTO should be used both for transactional and non transactional query, // the best way for that is to use closure
		func() (doc *firestore.DocumentSnapshot, err error) {
			return f.documentRef(time).Get(ctx)
		},
		time)
	if err != nil {
		return nil, err
	}

	hourFromDb, err := f.domainHourFromDateDTO(date, time)
	if err != nil {
		return nil, err
	}
	return hourFromDb, err
}
```

Source: [hour_firestore_repository.go on GitHub](https://bit.ly/2ZBYsBI)

```go
package main

// for now we are keeping backward comparability, because of that it's a bit messy and overcomplicated
// todo - we will clean it up later with CQRS :-)
func (f FirestoreHourRepository) domainHourFromDateDTO(date Date, hourTime time.Time) (*hour.Hour, error) {
	firebaseHour, found := findHourInDateDTO(date, hourTime)
	if !found {
		// in reality this date exists, even if it's not persisted
		return f.hourFactory.NewNotAvailableHour(hourTime)
	}

	availability, err := mapAvailabilityFromDTO(firebaseHour)
	if err != nil {
		return nil, err
	}

	return f.hourFactory.UnmarshalHourFromDatabase(firebaseHour.Hour.Local(), availability)
}
```

Source: [hour_firestore_repository.go on GitHub](https://bit.ly/3dwlnX3)

Unfortunately, the Firebase interfaces for the transactional and non-transactional queries are not fully compatible. To
avoid duplication, I created `getDateDTO` that can handle this difference by passing `getDocumentFn`.

不幸的是，Firebase 的事务性查询和非事务性查询的接口并不完全兼容。为了避免重复，我创建了 `getDateDTO`，它可以通过传递 `getDocumentFn` 来处理这种差异。

```go
package main

func (f FirestoreHourRepository) getDateDTO(
	getDocumentFn func() (doc *firestore.DocumentSnapshot, err error), dateTime time.Time, ) (Date, error) {
	doc, err := getDocumentFn()
	if status.Code(err) == codes.NotFound {
		// in reality this date exists, even if it's not persisted
		return NewEmptyDateDTO(dateTime), nil
	}
	if err != nil {
		return Date{}, err
	}
	date := Date{}
	if err := doc.DataTo(&date); err != nil {
		return Date{}, errors.Wrap(err, "unable to unmarshal Date from Firestore")
	}
	return date, nil
}
```

Source: [hour_firestore_repository.go on GitHub](https://bit.ly/2M83z9p)

Even if some extra code is needed, it’s not bad. And at least it can be tested easily.

即使需要一些额外的代码，也不错。至少它可以很容易地测试。

### Updating the data / 更新数据

As I mentioned before – it’s critical to be sure that **only one person can schedule a training in one hour**. To handle
that scenario, we need to use **optimistic locking and transactions**. Even if transactions is a pretty common term,
let’s ensure that we are on the same page with Optimistic Locking.

正如我之前提到的--关键是要确保在一个小时内只有一个人可以安排培训。为了处理这种情况，我们需要使用乐观锁和事务。即使事务是一个相当常见的术语，让我们确保我们与乐观锁定在同一起跑线上。

**Optimistic concurrency control** assumes that many transactions can frequently complete without interfering with each
other. While running, transactions use data resources without acquiring locks on those resources. Before committing,
each transaction verifies that no other transaction has modified the data it has read. If the check reveals conflicting
modifications, the committing transaction rolls back and can be restarted.

__乐观并发控制__ 假设许多事务可以频繁完成而不会相互干扰。在运行时，事务使用数据资源而不获取这些资源的锁。在提交之前，每个事务都验证没有其他事务修改它已读取的数据。如果检查发现有冲突的修改，提交事务会回滚并重新启动。

wikipedia.org (https://en.wikipedia.org/wiki/Optimistic_concurrency_control)

Technically transactions handling is not complicated. The biggest challenge that I had was a bit different – how to
manage transactions in a clean way that does not affect the rest of the application too much, is not dependent on the
implementation, and is explicit and fast.

从技术上讲，事务处理并不复杂。我所面临的最大挑战有点不同--如何以一种干净的方式管理事务，不对应用程序的其他部分造成太大影响，不依赖于实现，并且是明确和快速的。

I experimented with many ideas, like passing transaction via context.Context, handing transaction on HTTP/gRPC/message
middlewares level, etc. All approaches that I tried had many major issues – they were a bit magical, not explicit, and
slow in some cases.

我尝试了很多想法，比如通过 context.Context 传递事务，在 HTTP / gRPC / 消息中间件 层面传递事务，等等。我所尝试的所有方法都有很多主要问题--它们有点神奇，不明确，而且在某些情况下很慢。

Currently, my favorite one is an approach based on closure passed to the update function.

目前，我最喜欢的是一种基于传递给更新函数的闭合的方法。

```go
package hour

type Repository interface { // ...
	UpdateHour(
		ctx context.Context,
		hourTime time.Time,
		updateFn func(h *Hour) (*Hour, error),
	) error
}
```

Source: [repository.go on GitHub](https://bit.ly/3bo6QtT)

The basic idea is that we when we run `UpdateHour`, we need to provide `updateFn` that can update the provided hour.

基本思想是我们在运行 `UpdateHour` 时，需要提供 `updateFn` 来更新提供的小时。

So in practice in one transaction we:

- get and provide all parameters for `updateFn` (`h *Hour` in our case) based on provided UUID or any other parameter (
  in our case `hourTime time.Time`)
- execute the closure (`updateFn` in our case)
- save return values (`*Hour` in our case, if needed we can return more)
- execute a rollback in case of an error returned from the closure

所以在实践中，我们在一笔交易中：

- 根据提供的UUID或任何其他参数（在我们的例子中是 `hourTime.Time` ），获取并提供 `updateFn` 的所有参数（`h *Hour`）。
- 执行闭包（在我们的例子中是 `updateFn` ）
- 保存返回值（在我们的例子中是 `*Hour` ，如果需要我们可以返回更多）
- 在闭包返回错误的情况下执行回滚

How is it used in practice?

如何在实践中使用？

```go
package main

func (g GrpcServer) MakeHourAvailable(ctx context.Context, request *trainer.UpdateHourRequest) (*trainer.EmptyResponse, error) {
	trainingTime, err := protoTimestampToTime(request.Time)
	if err != nil {
		return nil, status.Error(codes.InvalidArgument, "unable to parse time")
	}

	if err := g.hourRepository.UpdateHour(ctx, trainingTime, func(h *hour.Hour) (*hour.Hour, error) {
		if err := h.MakeAvailable(); err != nil {
			return nil, err
		}

		return h, nil
	}); err != nil {
		return nil, status.Error(codes.Internal, err.Error())
	}

	return &trainer.EmptyResponse{}, nil
}

```

Source: [grpc.go on GitHub](https://bit.ly/3pD36Kd)

As you can see, we get Hour instance from some (unknown!) database. After that, we make this hour Available. If
everything is fine, we save the hour by returning it. As part of **Domain-Driven Design Lite** (Chapter 6), **all
validations were moved to the domain level, so here we are sure that we aren’t doing anything “stupid”. It also
simplified this code a lot**.

正如你所看到的，我们从某个（未知的！）数据库中获得了Hour实例。之后，我们使这个小时可用。如果一切正常，我们通过返回它来保存这个小时。作为领域驱动设计精简版（第 6
章）的一部分，所有验证都移到了领域级别，所以在这里我们确信我们没有做任何“愚蠢”的事情。它也大大简化了这段代码。

```shell
+func (g GrpcServer) MakeHourAvailable(ctx context.Context, request *trainer.UpdateHourRequest) (*trainer.EmptyResponse, error) {
@ ...
-func (g GrpcServer) UpdateHour(ctx context.Context, req *trainer.UpdateHourRequest) (*trainer.EmptyResponse, error) {
-   trainingTime, err := grpcTimestampToTime(req.Time)
-   if err != nil {
-       return nil, status.Error(codes.InvalidArgument, "unable to parse time")
-   }
-
-   date, err := g.db.DateModel(ctx, trainingTime)
-   if err != nil {
-       return nil, status.Error(codes.Internal, fmt.Sprintf("unable to get data model: %s", err))
-   }
-
-   hour, found := date.FindHourInDate(trainingTime)
-   if !found {
-       return nil, status.Error(codes.NotFound, fmt.Sprintf("%s hour not found in schedule", trainingTime))
-   }
-
-   if req.HasTrainingScheduled && !hour.Available {
-       return nil, status.Error(codes.FailedPrecondition, "hour is not available for training")
-   }
-
-   if req.Available && req.HasTrainingScheduled {
-       return nil, status.Error(codes.FailedPrecondition, "cannot set hour as available when it have training scheduled")
-   }
-   if !req.Available && !req.HasTrainingScheduled {
-       return nil, status.Error(codes.FailedPrecondition, "cannot set hour as unavailable when it have no training scheduled")
-   }
-   hour.Available = req.Available
-
-   if hour.HasTrainingScheduled && hour.HasTrainingScheduled == req.HasTrainingScheduled {
-       return nil, status.Error(codes.FailedPrecondition, fmt.Sprintf("hour HasTrainingScheduled is already %t", hour.HasTrainingScheduled))
-   }
-
-   hour.HasTrainingScheduled = req.HasTrainingScheduled
-   if err := g.db.SaveModel(ctx, date); err != nil {
-       return nil, status.Error(codes.Internal, fmt.Sprintf("failed to save date: %s", err))
-   }
-
-   return &trainer.EmptyResponse{}, nil
-}
```

Source: [0249977c58a310d343ca2237c201b9ba016b148e on GitHub](https://bit.ly/2ZCKjnL)

In our case from `updateFn` we return only (*Hour, error) – **you can return more values if needed**. You can return
event sourcing events, read models, etc.

在我们的例子中，从 `updateFn` 中我们只返回（*Hour, error）--如果需要，你可以返回更多的值。你可以返回事件源事件，读取模型，等等。

We can also, in theory, use the same `hour.*Hour`, that we provide to updateFn. I decided to not do that. Using the
returned value gives us more flexibility (we can replace a different instance of hour.*Hour if we want).

理论上，我们也可以使用我们提供给 `updateFn` 的相同的 `hour.*Hour`。我决定不这么做。使用返回的值给了我们更多的灵活性（如果我们想的话，我们可以替换一个不同的 `hour.*Hour `实例）。

It’s also nothing terrible to have multiple UpdateHour-like functions created with extra data to save. Under the hood,
the implementation should re-use the same code without a lot of duplication.

使用要保存的额外数据创建多个类似 `UpdateHour` 的函数也没什么可怕的。在幕后，实现应该重复使用相同的代码，而不需要大量重复。

```go
package repo

type Repository interface { // ...
	UpdateHour(
		ctx context.Context,
		hourTime time.Time,
		updateFn func(h *Hour) (*Hour, error),
	) error
	UpdateHourWithMagic(
		ctx context.Context,
		hourTime time.Time,
		updateFn func(h *Hour) (*Hour, *Magic, error),
	) error
}
```

How to implement it now?

现在如何实现？

#### In-memory transactions implementation / 内存事务实现

The memory implementation is again the simplest one. We need to get the current value, execute closure, and save the
result.

内存实现又是最简单的一种。我们需要获取当前值，执行闭包，并保存结果。

What is essential, in the map, we store a copy instead of a pointer. Thanks to that, we are sure that without the
“commit” (`m.hours[hourTime] = *updatedHour`) our values are not saved. We will double-check it in tests.

最重要的是，在map中，我们存储了一个副本而不是一个指针。多亏了这一点，我们确信没有“提交”（`m.hours[hourTime] = *updatedHour`）我们的值不会被保存。我们将在测试中仔细检查它。

```go
package main

func (m *MemoryHourRepository) UpdateHour(
	_ context.Context,
	hourTime time.Time,
	updateFn func(h *hour.Hour) (*hour.Hour, error),
) error {
	m.lock.Lock()
	defer m.lock.Unlock()
	currentHour, err := m.getOrCreateHour(hourTime)
	if err != nil {
		return err
	}
	updatedHour, err := updateFn(currentHour)
	if err != nil {
		return err
	}
	m.hours[hourTime] = *updatedHour
	return nil
}
```

Source: [hour_memory_repository.go on GitHub](https://bit.ly/2M7RLnC)

#### Firestore transactions implementation / Firestore 事务实现

Firestore implementation is a bit more complex, but again – it’s related to backward compatibility. Functions
`getDateDTO`, `domainHourFromDateDTO`, `updateHourInDataDTO` could be probably skipped when our data model would be
better. Another reason to not use Database-centric/Response-centric approach!

Firestore的实现要复杂一些，但还是那句话--这与向后兼容有关。当我们的数据模型变得更好时，`getDateDTO`、`domainHourFromDateDTO`、`updateHourInDataDTO等函数可能会被跳过`
。另一个不使用以数据库为中心/以响应为中心的方法的原因!

```go
package main

func (f FirestoreHourRepository) UpdateHour(
	ctx context.Context,
	hourTime time.Time,
	updateFn func(h *hour.Hour) (*hour.Hour, error),
) error {
	err := f.firestoreClient.RunTransaction(ctx, func(ctx context.Context, transaction *firestore.Transaction) error {
		dateDocRef := f.documentRef(hourTime)
		firebaseDate, err := f.getDateDTO(
			// getDateDTO should be used both for transactional and non transactional query, // the best way for that is to use closure
			func() (doc *firestore.DocumentSnapshot, err error) {
				return transaction.Get(dateDocRef)
			},
			hourTime)
		if err != nil {
			return err
		}
		hourFromDB, err := f.domainHourFromDateDTO(firebaseDate, hourTime)
		if err != nil {
			return err
		}
		updatedHour, err := updateFn(hourFromDB)
		if err != nil {
			return errors.Wrap(err, "unable to update hour")
		}
		updateHourInDataDTO(updatedHour, &firebaseDate)
		return transaction.Set(dateDocRef, firebaseDate)
	})
	return errors.Wrap(err, "firestore transaction failed")
}
```

Source: [hour_firestore_repository.go on GitHub](https://bit.ly/3keLqU3)

As you can see, we get `*hour.Hour`, call `updateFn`, and save results inside of RunTransaction.

正如你所看到的，我们得到 `*hour.Hour`，调用 `updateFn`，并将结果保存在 `RunTransaction` 中。

**Despite some extra complexity, this implementation is still clear, easy to understand and test.**

尽管有一些额外的复杂性，这个实现仍然是清晰的，易于理解和测试。

#### MySQL transactions implementation / MySQL 事务实现

Let’s compare it with MySQL implementation, where we designed models in a better way. Even if the way of implementation
is similar, the biggest difference is a way of handling transactions.

让我们把它与MySQL的实现进行比较，在那里我们以更好的方式设计模型。即使实现的方式相似，最大的区别是处理事务的方式。

In the SQL driver, the transaction is represented by *db.Tx. We use this particular object to call all queries and do a
rollback and commit. To ensure that we don’t forget about closing the transaction, we do rollback and commit in
the `defer`.

在SQL驱动中，事务是由*db.Tx表示的。我们使用这个特殊的对象来调用所有的查询，并做回滚和提交。为了确保我们不会忘记关闭事务，我们在`defer`中做回滚和提交。

```go
package main

func (m MySQLHourRepository) UpdateHour(
	ctx context.Context,
	hourTime time.Time,
	updateFn func(h *hour.Hour) (*hour.Hour, error),
) (err error) {
	tx, err := m.db.Beginx()
	if err != nil {
		return errors.Wrap(err, "unable to start transaction")
	}
	// Defer is executed on function just before exit.
	// With defer, we are always sure that we will close our transaction properly. defer func() {
	// In `UpdateHour` we are using named return - `(err error)`.
	// Thanks to that that can check if function exits with error.
	//
	// Even if function exits without error, commit still can return error.
	// In that case we can override nil to err `err = m.finish...`.
	err = m.finishTransaction(err, tx)
}()
existingHour, err := m.getOrCreateHour(ctx, tx, hourTime, true) if err != nil {
return err }
updatedHour, err := updateFn(existingHour) if err != nil {
return err }
if err := m.upsertHour(tx, updatedHour); err != nil { return err
}
return nil }

```

Source: [hour_mysql_repository.go on GitHub](https://bit.ly/3bi6pRG)

In that case, we also get the hour by passing forUpdate == true to getOrCreateHour. This flag is adding FOR UPDATE
statement to our query. The FOR UPDATE statement is critical because without that, we will allow parallel transactions
to change the hour.

在这种情况下，我们还可以通过将 forUpdate == true 传递给 getOrCreateHour 来获取小时。该标志将 FOR UPDATE 语句添加到我们的查询中。 FOR UPDATE
语句很关键，因为没有它，我们将允许并行事务更改小时。

SELECT ... FOR UPDATE

For index records the search encounters, locks the rows and any associated index entries, the same as if you issued an
UPDATE statement for those rows. Other transactions are blocked from updating those rows.

对于搜索遇到的索引记录，锁住这些行和任何相关的索引条目，就像你为这些行发布UPDATE语句一样。其他事务被阻止更新这些记录。

dev.mysql.com (https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)

```shell
func (m MySQLHourRepository) getOrCreateHour(
   ctx context.Context,
   db sqlContextGetter,
   hourTime time.Time,
   forUpdate bool,
) (*hour.Hour, error) {
   dbHour := mysqlHour{}

   query := "SELECT * FROM `hours` WHERE `hour` = ?"
   if forUpdate {
      query += " FOR UPDATE"
   }

    // ...

```

Source: [hour_mysql_repository.go on GitHub](https://bit.ly/3sdIE46)

I never sleep well when I don’t have an automatic test for code like that. Let’s address it later.

当我没有对这样的代码进行自动测试时，我总是睡不好觉。我们以后再解决这个问题吧。

`finishTransaction` is executed, when `UpdateHour` exits. When commit or rollback failed, we can also override the
returned error.

当 `UpdateHour` 退出时，`finishTransaction` 被执行。当提交或回滚失败时，我们也可以覆盖返回的错误。

```go
package main

// finishTransaction rollbacks transaction if error is provided.
// If err is nil transaction is committed.
//
// If the rollback fails, we are using multierr library to add error about rollback failure.
// If the commit fails, commit error is returned.
func (m MySQLHourRepository) finishTransaction(err error, tx *sqlx.Tx) error {
	if err != nil {
		if rollbackErr := tx.Rollback(); rollbackErr != nil {
			return multierr.Combine(err, rollbackErr)
		}
		return err
	} else {
		if commitErr := tx.Commit(); commitErr != nil {
			return errors.Wrap(err, "failed to commit tx")
		}
		return nil
	}
}
```

Source: [hour_mysql_repository.go on GitHub](https://bit.ly/3dwAkbR)

```go
package main

// upsertHour updates hour if hour already exists in the database.
// If your doesn't exists, it's inserted.
func (m MySQLHourRepository) upsertHour(tx *sqlx.Tx, hourToUpdate *hour.Hour) error {
	updatedDbHour := mysqlHour{
		Hour:         hourToUpdate.Time().UTC(),
		Availability: hourToUpdate.Availability().String(),
	}
	_, err := tx.NamedExec(
		`INSERT INTO
         hours (hour, availability)
      VALUES
         (:hour, :availability)
      ON DUPLICATE KEY UPDATE
         availability = :availability`,
		updatedDbHour,
	)
	if err != nil {
		return errors.Wrap(err, "unable to upsert hour")
	}
	return nil
}
```

Source: [hour_mysql_repository.go on GitHub](https://bit.ly/3dwAgZF)

### Summary

Even if the repository approach adds a bit more code, it’s totally worth making that investment.
**In practice, you may spend 5 minutes more to do that, and the investment should pay you back shortly.**

即使存储库的方法增加了一些代码，也完全值得进行这种投资。**在实践中，你可能会多花5分钟来做这件事，而投资应该很快就能收回。**

In this chapter, we miss one essential part – tests. Now adding tests should be much easier, but it still may not be
obvious how to do it properly.

在这一章中，我们错过了一个重要部分--测试。现在添加测试应该更容易了，但如何正确地添加测试可能还是不明显。

I will cover tests in the next chapter. The entire diff of this refactoring, including tests,
is [available on GitHub](https://bit.ly/3qJQ13d).

我将在下一章介绍测试。这次重构的全部内容，包括测试，都可以在 [GitHub上找到](https://bit.ly/3qJQ13d)。

And just to remind – you can also run the application with one [command](./chapter02.md) and find the entire source code
on [GitHub](https://github.com/ThreeDotsLabs/wild-workouts-go-ddd-example) !

提醒一下 - 你也可以用一个命令运行应用程序，并在 [GitHub](https://github.com/ThreeDotsLabs/wild-workouts-go-ddd-example) 上找到整个源代码 !

Another technique that works pretty well is Clean/Hexagonal architecture – Miłosz covers that in **Clean Archi-
tecture** (Chapter 9).

另一种效果很好的技术是 清洁 / 六边形架构 架构 —— Miłosz 在 Clean Architecture（第 9 章）中介绍了这一点。