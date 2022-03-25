## Domain-Driven Design Lite

Robert Laszczak

When I started working in Go, the community was not looking positively on techniques like DDD (Domain-Driven Design) and
Clean Architecture. I heard multiple times: “Don’t do Java in Golang!”, “I’ve seen that in Java, please don’t!”.

These times, I already had almost 10 years of experience in PHP and Python. I’ve seen too many bad things already there.
I remember all these “Eight-thousanders” (methods with +8k lines of code ) and applications that nobody wanted to
maintain. I was checking old git history of these ugly monsters, and they all looked harmless in the beginning. But with
time, small, innocent problems started to become more significant and more serious. I’ve also seen how DDD and Clean
Architecture solved these issues.

Maybe Golang is different? Maybe writing microservices in Golang will fix this issue?

### It was supposed to be so beautiful

Now, after exchanging experience with multiple people and having the ability to see a lot of codebases, my point of view
is a bit cleaner than 3 years ago. Unfortunately, I’m now far from thinking that just using Golang and microservices
will save us from all of these issues that I’ve encountered earlier. I started to actually have flashbacks from the old,
bad times.

It’s less visible because of the relatively younger codebase. It’s less visible because of the Golang design. But I’m
sure that with time, we will have more and more legacy Golang applications that nobody wants to maintain.

Fortunately, 3 years ago, despite to chilly reception I didn’t give up. I decided to try to use DDD and related
techniques that worked for me previously in Go. With Milosz we were leading teams for 3 years that were all successfully
using DDD, Clean Architecture, and all related, not-popular-enough techniques in Golang. They gave us the ability to
develop our applications and products with constant velocity, regardless of the age of the code.

It was obvious from the beginning, that moving patterns 1:1 from other technologies will not work. What is essential, we
did not give up idiomatic Go code and microservices architecture - they fit together perfectly!
Today I would like to share with you first, most straightforward technique – DDD lite.

### State of DDD in Golang

Before sitting to write this chapter, I checked a couple of articles about DDD in Go in Google. I will be brutal here:
they are all missing the most critical points making DDD working. **If I imagine that I would read these articles
without any DDD knowledge, I would not be encouraged to use them in my team. This superficial approach may also be the
reason why DDD is still not socialized in the Go community**.

In this book, we try to show all essential techniques and do it in the most pragmatic way. Before describing any
patterns, we start with a question: what does it give us? It’s an excellent way to challenge our current thinking.

I’m sure that we can change the Go community reception of these techniques. We believe that they are the best way to
implement complex business projects. **I believe that we will help to establish the position of Go as a great language
for building not only infrastructure but also business software**.

### You need to go slow, to go fast

It may be tempting to implement the project you work on in the simplest way. It’s even more tempting when you feel
pressure from “the top”. Are we using microservices, though? If it is needed, will we just rewrite the service? I heard
that story multiple times, and it rarely ended with a happy end. **It’s true that you will save some time with taking
shortcuts. But only in the short term**.

Let’s consider the example of tests of any kind. You can skip writing tests at the beginning of the project. You will
obviously save some time and management will be happy. **Calculation seems to be simple – the project was delivered
faster**.

But this shortcut is not worthwhile in the longer term. When the project grows, your team will start to be afraid of
making any changes. In the end, the sum of time that you will spend will be higher than implementing tests from the
beginning. **You will be slow down in the long term because of sacrificing quality for quick performance boost at the
beginning**. On the other hand - if a project is not critical and needs to be created fast, you can skip tests. It
should be a pragmatic decision, not just “we know better, and we do not create bugs”.

The case is the same for DDD. When you want to use DDD, you will need a bit more time in the beginning, but long-term
saving is enormous. However, not every project is complex enough to use advanced techniques like DDD.

**There is no quality vs. speed tradeoff. If you want to go fast in the long term, you need to keep high quality**.

**That’s great, but do you have any evidence it works?**

If you asked me this question two years ago, I would say: “Well, I feel that it works better!”. But just trusting my
words may be not enough.

There are many tutorials showing some dumb ideas and claiming that they work without any evidence – let’s don’t trust
them blindly!

![](./chapter06/ch0601.png)

<center>Figure 6.1: ‘Is High Quality Software Worth the Cost?’ from martinfowler.com</center>

Just to remind: if someone has a couple thousand Twitter
followers, [that alone is not a reason to trust them](https://en.wikipedia.org/wiki/Authority_bias)!

Fortunately, 2 years
ago [Accelerate: The Science of Lean Software and DevOps: Building and Scaling High Perform- ing Technology Organizations](https://www.amazon.com/Accelerate-Software-Performing-Technology-Organizations/dp/1942788339)
was released. In brief, this book is a description of what factors affect development teams’ performance. But the reason
this book became famous is that it’s not just a set of not validated ideas – **it’s based on scientific research.**

**I was mostly interested with the part showing what allows teams to be top-performing teams.** This book shows some
obvious facts, like introducing DevOps, CI/CD, and loosely coupled architecture, which are all an essential factor in
high-performing teams.

> If things like DevOps and CI/CD are not obvious to you, you can start with these books: [The Phoenix Project](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592) and [The DevOps Handbook](https://www.amazon.com/DevOps-Handbook-World-Class-Reliability-Organizations/dp/1942788002)

What Accelerate tells us about top-performing teams?

We found that high performance is possible with all kinds of systems, provided that systems and the teams that build and
maintain them—are loosely coupled.

This key architectural property enables teams to easily test and deploy individual components or services even as the
organization and the number of systems it operates grow—that is, it allows organizations to increase their productivity
as they scale.

So let’s use microservices, and we are done? I would not be writing this chapter if it was enough.

- Make large-scale changes to the design of their system without depending on other teams to make changes in their
  systems or creating significant work for other teams
- Complete their work without communicating and coordinating with people outside their team
- Deploy and release their product or service on demand, regardless of other services it depends upon
- Do most of their testing on demand, without requiring an integrated test environment Perform deployments during normal
  business hours with negligible downtime

Unfortunately, in real life, many so-called service-oriented architectures don’t permit testing and deploy- ing services
independently of each other, and thus will not enable teams to achieve higher performance.

[...] employing the latest whizzy microservices architecture deployed on containers is no guarantee of higher
performance if you ignore these characteristics. [...] To achieve these characteristics, design systems are loosely
coupled — that is, can be changed and validated independently of each other.

Using just microservices architecture and splitting services to small pieces is not enough. If it’s done in a wrong way,
it adds extra complexity and slows teams down. DDD can help us here.

I’m mentioning DDD term multiple times. What DDD actually is?

## What is DDD (Domain-Driven Design)

Let’s start with Wikipedia definition:

- Domain-driven design (DDD) is the concept that the structure and language of your code (class names, class methods,
  class variables) should match the business domain. For example, if your software processes loan applications, it might
  have classes such as LoanApplication and Customer, and methods such as AcceptOffer and Withdraw.

  ![](./chapter06/ch0602.png)

  <center>Figure 6.2: No, God please no!</center>

Well, it’s not the perfect one. It’s still missing some most important points.

It’s also worth to mention, that DDD was introduced in 2003. That’s pretty long time ago. Some distillation may be
helpful to put DDD in the 2020 and Go context.

> If you are interested in some historical context on when DDD was created, you should check [Tackling Complexity in the Heart of Software](https://youtu.be/dnUFEg68ESM?t=1109) by the DDD creator - Eric Evans
>

My simple DDD definition is: Ensure that you solve valid problem in the optimal way. After that implement the solution
in a way that your business will understand without any extra translation from technical language needed.

How to achieve that?

### Coding is a war, to win you need a strategy!

I like to say that “5 days of coding can save 15 minutes of planning”.

Before starting to write any code, you should ensure that you are solving a valid problem. It may sound obvious, but in
practice from my experience, it’s not as easy as it sounds. This is often the case that the solution created by
engineers is not actually solving the problem that the business requested. A set of patterns that helps us in that field
is named Strategic DDD patterns.

From my experience, DDD Strategic Patterns are often skipped. The reason is simple: we are all developers, and we like
to write code rather than talk to the “business people”. Unfortunately, this approach when we are closed in a basement
without talking to any business people has a lot of downsides. Lack of trust from the business, lack of knowledge of how
the system works (from both business and engineering side), solving wrong problems – these are just some of the most
common issues.

The good news is that in most cases it’s caused by a lack of proper techniques like Event Storming. They can give both
sides advantages. What is also surprising is that talking to the business may be one of the most enjoyable parts of the
work!

Apart from that, we will start with patterns that apply to the code. They can give us some advantages of DDD. They will
also be useful for you faster.

Without Strategic patterns, I’d say that you will just have 30% of advantages that DDD can give you. We will go back to
Strategic patterns in the next chapters.

### DDD Lite in Go

After a pretty long introduction, it’s finally time to touch some code! In this chapter, we will cover some basics of
**Tactical Domain-Driven Design patterns in Go**. Please keep in mind that this is just the beginning. There will be a
couple more chapters needed to cover the entire topic.

One of the most crucial parts of Tactical DDD is trying to reflect the domain logic directly in the code.

But it’s still some non-specific definition – and it’s not needed at this point. I don’t also want to start by
describing what are Value Objects, Entities, Aggregates. Let’s better start with practical examples.

### Refactoring of trainer service

The first (micro)service that we will refactor is trainer. We will leave other services untouched now – we will go back
to them later.

This service is responsible for keeping the trainer schedule and ensuring that we can have only one training scheduled
in one hour. It also keeps the information about available hours (trainer’s schedule).

The initial implementation was not the best. Even if it is not a lot of logic, some parts of the code started to be
messy. I have also some feeling based on experience, that with time it will get worse.

```go
package main

type GrpcServer struct {
	db db
}

func (g GrpcServer) UpdateHour(ctx context.Context, req *trainer.UpdateHourRequest) (*trainer.EmptyResponse, error) {
	trainingTime, err := grpcTimestampToTime(req.Time)
	if err != nil {
		return nil, status.Error(codes.InvalidArgument, "unable to parse time")
	}

	date, err := g.db.DateModel(ctx, trainingTime)
	if err != nil {
		return nil, status.Error(codes.Internal, fmt.Sprintf("unable to get data model: %s", err))
	}

	hour, found := date.FindHourInDate(trainingTime)
	if !found {
		return nil, status.Error(codes.NotFound, fmt.Sprintf("%s hour not found in schedule", trainingTime))
	}

	if req.HasTrainingScheduled && !hour.Available {
		return nil, status.Error(codes.FailedPrecondition, "hour is not available for training")
	}

	if req.Available && req.HasTrainingScheduled {
		return nil, status.Error(codes.FailedPrecondition, "cannot set hour as available when it have training scheduled")
	}
	if !req.Available && !req.HasTrainingScheduled {
		return nil, status.Error(codes.FailedPrecondition, "cannot set hour as unavailable when it have no training scheduled")
	}
	hour.Available = req.Available

	if hour.HasTrainingScheduled && hour.HasTrainingScheduled == req.HasTrainingScheduled {
		return nil, status.Error(codes.FailedPrecondition, fmt.Sprintf("hour HasTrainingScheduled is already %t", hour.HasTrainingScheduled))
	}

	hour.HasTrainingScheduled = req.HasTrainingScheduled
	if err := g.db.SaveModel(ctx, date); err != nil {
		return nil, status.Error(codes.Internal, fmt.Sprintf("failed to save date: %s", err))
	}

	return &trainer.EmptyResponse{}, nil
}
```

Source: [grpc.go on GitHub](https://bit.ly/3qR2nWY)

Even if it’s not the worst code ever, it reminds me what I’ve seen when I checked the git history of the code I worked
on. I can imagine that after some time some new features will arrive and it will be much worse. It’s also hard to mock
dependencies here, so there are also no unit tests.

### The First Rule - reflect your business logic literally

While implementing your domain, you should stop thinking about structs like dummy data structures or “ORM like” entities
with a list of setters and getters. You should instead think about them like **types with behavior**.

When you are talk with your business stakeholders, they say “I’m scheduling training on 13:00”, rather than “I’m setting
the attribute state to ‘training scheduled’ for hour 13:00.”.

They also don’t say: “you can’t set attribute status to ‘training_scheduled’”. It is rather: “You can’t schedule
training if the hour is not available”. How to put it directly in the code?

```go
package hour

func (h *Hour) ScheduleTraining() error {
	if !h.IsAvailable() {
		return ErrHourNotAvailable
	}
	h.availability = TrainingScheduled
	return nil
}
```

Source: [availability.go on GitHub](https://bit.ly/3aBnS8A)

One question that can help us with implementation is: “Will business understand my code without any extra translation of
technical terms?”. You can see in that snippet, that **even not technical person will be able to understand when you can
schedule training**.

This approach’s cost is not high and helps to tackle complexity to make rules much easier to understand. Even if the
change is not big, we got rid of this wall of ifs that would become much more complicated in the future.

We are also now able to easily add unit tests. What is good – we don’t need to mock anything here. The tests are also a
documentation that helps us understand how Hour behaves.

```go
package hour_test

func TestHour_ScheduleTraining(t *testing.T) {
	h, err := hour.NewAvailableHour(validTrainingHour())
	require.NoError(t, err)
	require.NoError(t, h.ScheduleTraining())
	assert.True(t, h.HasTrainingScheduled())
	assert.False(t, h.IsAvailable())
}

func TestHour_ScheduleTraining_with_not_available(t *testing.T) {
	h := newNotAvailableHour(t)
	assert.Equal(t, hour.ErrHourNotAvailable, h.ScheduleTraining())
}
```

Source: [availability_test.go on GitHub](https://bit.ly/3bofN6D)

Now, if anybody will ask the question “When I can schedule training”, you can quickly answer that. In a bigger systems,
the answer to this kind of question is even less obvious – multiple times I spent hours trying to find all places where
some objects were used in an unexpected way. The next rule will help us with that even more.

### Testing Helpers

It’s useful to have some helpers in tests for creating our domain entities. For example: `newExampleTrainingWithTime`,
`newCanceledTraining` etc. It also makes our domain tests much more readable.

Custom asserts, like `assertTrainingsEquals` can also save a lot of
duplication. [github.com/google/go-cmp](https://github.com/google/go-cmp) library is extremely useful for comparing
complex structs. It allows us to compare our domain types with private field,
skip [some field validation](https://godoc.org/github.com/google/go-cmp/cmp/cmpopts#IgnoreFields) or
implement [custom validation functions](https://godoc.org/github.com/google/go-cmp/cmp/cmpopts#IgnoreFields).

```go
package test2

func assertTrainingsEquals(t *testing.T, tr1, tr2 *training.Training) {
	cmpOpts := []cmp.Option{
		cmpRoundTimeOpt,
		cmp.AllowUnexported(
			training.UserType{},
			time.Time{},
			training.Training{},
		),
	}
	assert.True(
		t,
		cmp.Equal(tr1, tr2, cmpOpts...),
		cmp.Diff(tr1, tr2, cmpOpts...),
	)
}

```

It’s also a good idea to provide a Must version of constructors used often, so for example `MustNewUser`. In contrast
with normal constructors, they will panic if parameters are not valid (for tests it’s not a problem).

```go
package t

func NewUser(userUUID string, userType UserType) (User, error) {
	if userUUID == "" {
		return User{}, errors.New("missing user UUID")
	}
	if userType.IsZero() {
		return User{}, errors.New("missing user type")
	}
	return User{userUUID: userUUID, userType: userType}, nil
}

func MustNewUser(userUUID string, userType UserType) User {
	u, err := NewUser(userUUID, userType)
	if err != nil {
		panic(err)
	}
	return u
}
```

The Second Rule: always keep a valid state in the memory

- I recognize that my code will be used in ways I cannot anticipate, in ways it was not designed, and for longer than it
  was ever intended.

  The Rugged Manifesto (https://ruggedsoftware.org/)

The world would be better if everyone would take this quote into account. I’m also not without fault here. From my
observation, when you are sure that the object that you use is always valid, it helps to avoid a lot of ifs and bugs.
You will also feel much more confident knowing that you are not able to do anything stupid with the current code.

I have many flashbacks that I was afraid to make some change because I was not sure of the side effects of it.
**Developing new features is much slower without confidence that you are correctly using the code!**

Our goal is to do validation in only one place (good DRY) and ensure that nobody can change the internal state of the
Hour. The only public API of the object should be methods describing behaviors. No dumb getters and setters!. We need to
also put our types to separate package and make all attributes private.

```go

package hour

type Hour struct {
	hour         time.Time
	availability Availability
}

// ...
func NewAvailableHour(hour time.Time) (*Hour, error) {
	if err := validateTime(hour); err != nil {
		return nil, err
	}
	return &Hour{
		hour:         hour,
		availability: Available,
	}, nil
}
```

Source: [hour.go on GitHub](https://bit.ly/3se9o4I)

We should also ensure that we are not breaking any rules inside of our type.

Bad example:

```
h := hour.NewAvailableHour("13:00")
if h.HasTrainingScheduled() {
  h.SetState(hour.Available)
} else {
  return errors.New("unable to cancel training")
}
```

Good example:

```
func (h *Hour) CancelTraining() error {
  if !h.HasTrainingScheduled() {
    return ErrNoTrainingScheduled
  }
  h.availability = Available
    return nil
}
  
  // ...
  h := hour.NewAvailableHour("13:00")
  if err := h.CancelTraining(); err != nil {
    return err
  }
```

### The Third Rule - domain needs to be database agnostic

There are multiple schools here – some are telling that it’s fine to have domain impacted by the database client. From
our experience, keeping the domain strictly without any database influence works best.

The most important reasons are:

- domain types are not shaped by used database solution – they should be only shaped by business rules
- we can store data in the database in a more optimal way
- because of the Go design and lack of “magic” like annotations, ORM’s or any database solutions are affecting in even
  more significant way

### Domain-First approach

> If the project is complex enough, we can spend even 2-4 weeks to work on the domain layer, with just in-memory database implementation. In that case, we can explore the idea deeper and defer the decision to choose the database later. All our implementation is just based on unit tests.
> We tried that approach a couple of times, and it always worked nicely. It is also a good idea to have some timebox here, to not spend too much time.
> Please keep in mind that this approach requires a good relationship and a lot of trust from the business! **If your relationship with business is far from being good, Strategic DDD patterns will improve that. Been there, done that!**

To not make this chapter long, let’s just introduce the Repository interface and assume that it works.

I will cover this topic more in-depth in the next chapter.

```go

package hour

type Repository interface {
	GetOrCreateHour(ctx context.Context, time time.Time) (*Hour, error)
	UpdateHour(
		ctx context.Context,
		hourTime time.Time,
		updateFn func(h *Hour) (*Hour, error),
	) error
}

```

Source: [repository.go on GitHub](https://bit.ly/2NP1NuA)

> You may ask why UpdateHour has updateFn func(h *Hour) (*Hour, error) – we will use that for han- dling transactions nicely. You can learn more in The Repository Pattern (Chapter 7).

### Using domain objects

I did a small refactor of our gRPC endpoints, to provide an API that is more “behavior-oriented” rather
than [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete). It reflects better the new characteristic of
the domain. From my experience, it’s also much easier to maintain multiple, small methods than one, “god” method
allowing us to update everything.

```shell
--- a/api/protobuf/trainer.proto
+++ b/api/protobuf/trainer.proto
@@ -6,7 +6,9 @@ import "google/protobuf/timestamp.proto";
 
 service TrainerService {
   rpc IsHourAvailable(IsHourAvailableRequest) returns (IsHourAvailableResponse) {}
-  rpc UpdateHour(UpdateHourRequest) returns (EmptyResponse) {}
+  rpc ScheduleTraining(UpdateHourRequest) returns (EmptyResponse) {}
+  rpc CancelTraining(UpdateHourRequest) returns (EmptyResponse) {}
+  rpc MakeHourAvailable(UpdateHourRequest) returns (EmptyResponse) {}
 }
 
 message IsHourAvailableRequest {
@@ -19,9 +21,6 @@ message IsHourAvailableResponse {
 
 message UpdateHourRequest {
   google.protobuf.Timestamp time = 1;
-
-  bool has_training_scheduled = 2;
-  bool available = 3;
 }
 
 message EmptyResponse {}

```

Source: [0249977c58a310d343ca2237c201b9ba016b148e on GitHub](https://bit.ly/3pFL4H0)

The implementation is now much simpler and easier to understand. We also have no logic here - just some orchestration.
Our gRPC handler now has 18 lines and no domain logic!

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

#### No more Eight-thousanders

> As I remember from the old-times, many Eight-thousanders were actually controllers with a lot of domain logic in HTTP controllers.
>
> With hiding complexity inside of our domain types and keeping rules that I described, we can prevent uncontrolled growth in this place.
>

### That’s all for today

I don’t want to make this chapter too long – let’s go step by step!

If you can’t wait, the entire working diff for the refactor is available on [GitHub](https://bit.ly/3umRYES). In the
next chapter, I’ll cover one part from the diff that is not explained here: repositories.

Even if it’s still the beginning, some simplifications in our code are visible.

The current implementation of the model is also not perfect – that’s good! You will never implement the perfect model
from the beginning. **It’s better to be prepared to change this model easily, instead of wasting time to make it
perfect**. After I added tests of the model and separated it from the rest of the application, I can change it without
any fear.

### Can I already put that I know DDD to my CV?

Not yet.

I needed 3 years after I heard about DDD to the time when I connected all the dots (It was before I heard about Go).
After that, I’ve seen why all techniques that we will describe in the next chapters are so important. But before
connecting the dots, it will require some patience and trust that it will work. It is worth that! You will not need 3
years like me, but we currently planned about 10 chapters on both strategic and tactical patterns. It’s a lot of new
features and parts to refactor in Wild Workouts left!

I know that nowadays a lot of people promise that you can become expert in some area after 10 minutes of an article or a
video. The world would be beautiful if it would be possible, but in reality, it is not so simple.

Fortunately, a big part of the knowledge that we share is universal and can be applied to multiple technologies, not
only Go. You can treat these learnings as an investment in your career and mental health in the long term. There is
nothing better than solving the right problems without fighting with unmaintainable code!