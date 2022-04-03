## High-Quality Database Integration Tests

Robert Laszczak

Did you ever hear about a project where changes were tested on customers that you don’t like or countries that are not
profitable? Or even worse – did you work on such project?

It’s not enough to say that it’s just not fair and not professional. It’s also hard to develop anything new because you
are afraid to make any change in your codebase.

In [2019 HackerRank Developer Skills Report](https://research.hackerrank.com/developer-skills/2019#jobsearch3)
Professional growth & learning was marked as the most critical factor during looking for a new job. Do you think you can
learn anything and grow when you test your application in that way?

It’s all leading to frustration and burnout.

To develop your application easily and with confidence, you need to have a set of tests on multiple levels. __In this
chapter, I will cover in practical examples how to implement high-quality database integration tests. I will also cover
basic Go testing techniques, like test tables, assert functions, parallel execution, and black-box testing.__

What it actually means that test quality is high?

### 4 principles of high-quality tests

I prepared 4 rules that we need to pass to say that our integration tests quality is high.

#### 1. Fast

Good tests need to be fast. There is no compromise here.

Everybody hates long-running tests. Let’s think about your teammates’ time and mental health when they are waiting for
test results. Both in CI and locally. It’s terrifying.

When you wait for a long time, you will likely start to do anything else in the meantime. After the CI passes (
hopefully), you will need to switch back to this task. Context switching is one of the biggest productivity killers.
It’s very exhausting for our brains. We are not robots.

I know that there are still some companies where tests can be executing for 24 hours. We don’t want to follow this
approach. You should be able to run your tests locally in less than 1 minute, ideally in less than 10s. I know that
sometimes it may require some time investment. It’s an investment with an excellent ROI (Return Of Investment)! In that
case, you can really quickly check your changes. Also, deployment times, etc. are much shorter.

It’s always worth trying to find quick wins that can reduce tests execution the most from my
experience. [Pareto principle (80/20 rule)](https://en.wikipedia.org/wiki/Pareto_principle) works here perfectly!

#### 2. Testing enough scenarios on all levels

I hope that you already know that 100% test coverage is not the best idea (as long as it is not a simple/critical
library).

It’s always a good idea to ask yourself the question “how easily can it break?”. It’s even more worth to ask this
question if you feel that the test that you are implementing starts to look exactly as a function that you test. At the
end we are not writing tests because tests are nice, but they should save our ass!

From my experience, __coverage like 70-80% is a pretty good result in Go__.

It’s also not the best idea to cover everything with component or end-to-end tests. First – you will not be able to do
that because of the inability to simulate some error scenarios, like rollbacks on the repository. Second – it will break
the first rule. These tests will be slow.

![](./chapter08/ch0801.png)

![](./chapter08/ch0802.png)

Tests on several layers should also overlap, so we will know that integration is done correctly.

You may think that solution for that is simple: the test pyramid! And that’s true… sometimes. Especially in applications
that handle a lot of operations based on writes.

![](./chapter08/ch0803.png)

<center>Figure 8.1: Testing Pyramid</center>

But what, for example, about applications that aggregate data from multiple other services and expose the data via API?
It has no complex logic of saving the data. Probably most of the code is related to the database operations. In this
case, we should use __reversed test pyramid__ (it actually looks more like a christmas tree). When big part of our
application is connected to some infrastructure (for example: database) it’s just hard to cover a lot of functionality
with unit tests.

#### 3. Tests need to be robust and deterministic

Do you know that feeling when you are doing some urgent fix, tests are passing locally, you push changes to the
repository and ... after 20 minutes they fail in the CI? It’s incredibly frustrating. It also discourages us from adding
new tests. It’s also decreasing our trust in them.

You should fix that issue as fast as you can. In that
case, [Broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory) is really valid.

![](./chapter08/ch0804.png)

<center>Figure 8.2: Christmas Tree</center>

#### 4. You should be able to execute most of the tests locally

Tests that you run locally should give you enough confidence that the feature that you developed or refactored is still
working. __E2E tests should just double-check if everything is integrated correctly__.

You will have also much more confidence when contracts between services are robust because of
using [gRPC](./chapter03.md), protobuf, or OpenAPI.

This is a good reason to cover as much as we can at lower levels (starting with the lowest): unit, integration, and
component tests. Only then E2E.

### Implementation

We have some common theoretical ground. But nobody pays us for being the master of theory of programming. Let’s go to
some practical examples that you can implement in your project.

Let’s start with the repository pattern that I described in the previous chapter.

The way how we can interact with our database is defined by the hour.Repository interface. It assumes that our
repository implementation is stupid. All complex logic is handled by domain part of our application. __It should just
save the data without any validations, etc. One of the significant advantages of that approach is the simplification of
the repository and tests implementation__.

In the previous chapter I prepared three different database implementations: MySQL, Firebase, and in-memory. We will
test all of them. All of them are fully compatible, so we can have just one test suite.

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

Source: [repository.go on GitHub](https://bit.ly/3aBdRbt)

Because of multiple repository implementations, in our tests we iterate through a list of them.

> It’s actually a pretty similar pattern to how we implemented [tests in Watermill](https://github.com/ThreeDotsLabs/watermill/blob/master/pubsub/tests/test_pubsub.go). All Pub/Sub implementations are passing the same test suite.
>

All tests that we will write will be black-box tests. In other words – we will only cover public functions with tests.
To ensure that, all our test packages have the _test suﬀix. That forces us to use only the public interface of the
package.
__It will pay back in the future with much more stable tests, that are not affected by any internal changes.__
If you cannot write good black-box tests, you should consider if your public APIs are well designed.

All our repository tests are executed in parallel. Thanks to that, they take less than 200ms. After adding multiple test
cases, this time should not increase significantly.

```go
package main_test

// ...
func TestRepository(t *testing.T) {
	rand.Seed(time.Now().UTC().UnixNano())

	repositories := createRepositories(t)

	for i := range repositories {
		// When you are looping over slice and later using iterated value in goroutine (here because of t.Parallel()),
		// you need to always create variable scoped in loop body!
		// More info here: https://github.com/golang/go/wiki/CommonMistakes#using-goroutines-on-loop-iterator-variables
		r := repositories[i]

		t.Run(r.Name, func(t *testing.T) {
			// It's always a good idea to build all non-unit tests to be able to work in parallel.
			// Thanks to that, your tests will be always fast and you will not be afraid to add more tests because of slowdown.
			t.Parallel()

			t.Run("testUpdateHour", func(t *testing.T) {
				t.Parallel()
				testUpdateHour(t, r.Repository)
			})
			t.Run("testUpdateHour_parallel", func(t *testing.T) {
				t.Parallel()
				testUpdateHour_parallel(t, r.Repository)
			})
			t.Run("testHourRepository_update_existing", func(t *testing.T) {
				t.Parallel()
				testHourRepository_update_existing(t, r.Repository)
			})
			t.Run("testUpdateHour_rollback", func(t *testing.T) {
				t.Parallel()
				testUpdateHour_rollback(t, r.Repository)
			})
		})
	}
}
```

Source: [hour_repository_test.go on GitHub](https://bit.ly/3sj7H69)

When we have multiple tests, where we pass the same input and check the same output, it is a good idea to use a
technique known as test table. The idea is simple: you should define a slice of inputs and expected outputs of the test
and iterate over it to execute tests.

```go
package main_test

func testUpdateHour(t *testing.T, repository hour.Repository) {
	t.Helper()
	ctx := context.Background()

	testCases := []struct {
		Name       string
		CreateHour func(*testing.T) *hour.Hour
	}{
		{
			Name: "available_hour",
			CreateHour: func(t *testing.T) *hour.Hour {
				return newValidAvailableHour(t)
			},
		},
		{
			Name: "not_available_hour",
			CreateHour: func(t *testing.T) *hour.Hour {
				h := newValidAvailableHour(t)
				require.NoError(t, h.MakeNotAvailable())

				return h
			},
		},
		{
			Name: "hour_with_training",
			CreateHour: func(t *testing.T) *hour.Hour {
				h := newValidAvailableHour(t)
				require.NoError(t, h.ScheduleTraining())

				return h
			},
		},
	}

	for _, tc := range testCases {
		t.Run(tc.Name, func(t *testing.T) {
			newHour := tc.CreateHour(t)

			err := repository.UpdateHour(ctx, newHour.Time(), func(_ *hour.Hour) (*hour.Hour, error) {
				// UpdateHour provides us existing/new *hour.Hour,
				// but we are ignoring this hour and persisting result of `CreateHour`
				// we can assert this hour later in assertHourInRepository
				return newHour, nil
			})
			require.NoError(t, err)

			assertHourInRepository(ctx, t, repository, newHour)
		})
	}
}
```

Source: [hour_repository_test.go on GitHub](https://bit.ly/3siawUZ)

You can see that we used a very popular [github.com/stretchr/testify](https://github.com/stretchr/testify) library. It’s
significantly reducing boilerplate in tests by providing multiple helpers
for [asserts](https://godoc.org/github.com/stretchr/testify/assert).

#### require.NoError()

> When assert.NoError assert fails, tests execution is not interrupted.
>
> It’s worth to mention that asserts from require package are stopping execution of the test when it fails. Because of that, it’s often a good idea to use require for checking errors. In many cases, if some operation fails, it doesn’t make sense to check anything later.
>
> When we assert multiple values, assert is a better choice, because you will receive more context.

If we have more specific data to assert, it’s always a good idea to add some helpers. It removes a lot of duplication,
and improves tests readability a lot!

```go
package main_test

func assertHourInRepository(ctx context.Context, t *testing.T, repo hour.Repository, hour *hour.Hour) {
	require.NotNil(t, hour)
	hourFromRepo, err := repo.GetOrCreateHour(ctx, hour.Time())
	require.NoError(t, err)
	assert.Equal(t, hour, hourFromRepo)
}
```

Source: [hour_repository_test.go on GitHub](https://bit.ly/37uw4Wk)

### Testing transactions

Mistakes taught me that I should not trust myself when implementing complex code. We can sometimes not understand the
documentation or just introduce some stupid mistake. You can gain the confidence in two ways:

1. TDD - let’s start with a test that will check if the transaction is working properly.
2. Let’s start with the implementation and add tests later.

```go
package main_test

func testUpdateHour_rollback(t *testing.T, repository hour.Repository) {
	t.Helper()
	ctx := context.Background()
	hourTime := newValidHourTime()
	err := repository.UpdateHour(ctx, hourTime, func(h *hour.Hour) (*hour.Hour, error) {
		require.NoError(t, h.MakeAvailable())
		return h, nil
	})
	err = repository.UpdateHour(ctx, hourTime, func(h *hour.Hour) (*hour.Hour, error) {
		assert.True(t, h.IsAvailable())
		require.NoError(t, h.MakeNotAvailable())
		return h, errors.New("something went wrong")
	})
	require.Error(t, err)
	persistedHour, err := repository.GetOrCreateHour(ctx, hourTime)
	require.NoError(t, err)
	assert.True(t, persistedHour.IsAvailable(), "availability change was persisted, not rolled back")
}

```

Source: [hour_repository_test.go on GitHub](https://bit.ly/2ZBXKUM)

When I’m not using TDD, I try to be paranoid if test implementation is valid.

To be more confident, I use a technique that I call tests sabotage.

The method is pretty simple - let’s break the implementation that we are testing and let’s see if anything failed.

```shell
 func (m MySQLHourRepository) finishTransaction(err error, tx *sqlx.Tx) error {
-       if err != nil {
-               if rollbackErr := tx.Rollback(); rollbackErr != nil {
-                       return multierr.Combine(err, rollbackErr)
-               }
-
-               return err
-       } else {
-               if commitErr := tx.Commit(); commitErr != nil {
-                       return errors.Wrap(err, "failed to commit tx")
-               }
-
-               return nil
+       if commitErr := tx.Commit(); commitErr != nil {
+               return errors.Wrap(err, "failed to commit tx")
        }
+
+       return nil
 }
```

If your tests are passing after a change like that, I have bad news...

### Testing database race conditions

Our applications are not working in the void. It can always be the case that two multiple clients may try to do the same
operation, and only one can win!

In our case, the typical scenario is when two clients try to schedule a training at the same time.
__We can have only one training scheduled in one hour.__

This constraint is achieved by optimistic locking (described in __The Repository Pattern__ (Chapter 7)) and domain
constraints (described in __Domain-Driven Design Lite__ (Chapter 6)).

Let’s verify if it is possible to schedule one hour more than once. The idea is simple:
__let’s create 20 goroutines, that we will release in one moment and try to schedule training.__
We expect that exactly one worker should succeed.

```go
package main_test

func testUpdateHour_parallel(t *testing.T, repository hour.Repository) {
	if _, ok := repository.(*main.FirestoreHourRepository); ok {
		// todo - enable after fix of https://github.com/googleapis/google-cloud-go/issues/2604
		t.Skip("because of emulator bug, it's not working in Firebase")
	}

	t.Helper()
	ctx := context.Background()

	hourTime := newValidHourTime()

	// we are adding available hour
	err := repository.UpdateHour(ctx, hourTime, func(h *hour.Hour) (*hour.Hour, error) {
		if err := h.MakeAvailable(); err != nil {
			return nil, err
		}
		return h, nil
	})
	require.NoError(t, err)

	workersCount := 20
	workersDone := sync.WaitGroup{}
	workersDone.Add(workersCount)

	// closing startWorkers will unblock all workers at once,
	// thanks to that it will be more likely to have race condition
	startWorkers := make(chan struct{})
	// if training was successfully scheduled, number of the worker is sent to this channel
	trainingsScheduled := make(chan int, workersCount)

	// we are trying to do race condition, in practice only one worker should be able to finish transaction
	for worker := 0; worker < workersCount; worker++ {
		workerNum := worker

		go func() {
			defer workersDone.Done()
			<-startWorkers

			schedulingTraining := false

			err := repository.UpdateHour(ctx, hourTime, func(h *hour.Hour) (*hour.Hour, error) {
				// training is already scheduled, nothing to do there
				if h.HasTrainingScheduled() {
					return h, nil
				}
				// training is not scheduled yet, so let's try to do that
				if err := h.ScheduleTraining(); err != nil {
					return nil, err
				}

				schedulingTraining = true

				return h, nil
			})

			if schedulingTraining && err == nil {
				// training is only scheduled if UpdateHour didn't return an error
				trainingsScheduled <- workerNum
			}
		}()
	}

	close(startWorkers)

	// we are waiting, when all workers did the job
	workersDone.Wait()
	close(trainingsScheduled)

	var workersScheduledTraining []int

	for workerNum := range trainingsScheduled {
		workersScheduledTraining = append(workersScheduledTraining, workerNum)
	}

	assert.Len(t, workersScheduledTraining, 1, "only one worker should schedule training")
}
```

Source: [hour_repository_test.go on GitHub](https://bit.ly/3pKAVJu)

__It is also a good example that some use cases are easier to test in the integration test, not in acceptance or E2E
level.__
Tests like that as E2E will be really heavy, and you will need to have more workers to be sure that they execute
transactions simultaneously.

### Making tests fast

__If your tests can’t be executed in parallel, they will be slow.__ Even on the best machine.

s putting t.Parallel() enough? Well, we need to ensure that our tests are independent. In our case,
__if two tests would try to edit the same hour, they can fail randomly.__
This is a highly undesirable situation.

To achieve that, I created the `newValidHourTime()` function that provides a random hour that is unique in the current
test run. In most applications, generating a unique UUID for your entities may be enough.

In some situations it may be less obvious, but still not impossible. I encourage you to spend some time to find the
solution. Please treat it as the investment in your and your teammates’ mental health .

```go
package main_test

// usedHours is storing hours used during the test,
// to ensure that within one test run we are not using the same hour
// (it should be not a problem between test runs)
var usedHours = sync.Map{}

func newValidHourTime() time.Time {
	for {
		minTime := time.Now().AddDate(0, 0, 1)

		minTimestamp := minTime.Unix()
		maxTimestamp := minTime.AddDate(0, 0, testHourFactory.Config().MaxWeeksInTheFutureToSet*7).Unix()

		t := time.Unix(rand.Int63n(maxTimestamp-minTimestamp)+minTimestamp, 0).Truncate(time.Hour).Local()

		_, alreadyUsed := usedHours.LoadOrStore(t.Unix(), true)
		if !alreadyUsed {
			return t
		}
	}
}
```

Source: [hour_repository_test.go on GitHub](https://bit.ly/2NN5YGV)

What is also good about making our tests independent, is no need for data cleanup. In my experience, doing data cleanup
is always messy because:

- when it doesn’t work correctly, it creates hard-to-debug issues in tests,
- it makes tests slower,
- it adds overhead to the development (you need to remember to update the cleanup function)
- it may make running tests in parallel harder.

It may also happen that we are not able to run tests in parallel. Two common examples are:

- pagination – if you iterate over pages, other tests can put something in-between and move “items” in the pages.
- global counters – like with pagination, other tests may affect the counter in an unexpected way.

In that case, it’s worth to keep these tests as short as we can.

Please, don’t use sleep in tests!

The last tip that makes tests flaky and slow is putting the sleep function in them. Please, don’t! It’s much better to
synchronize your tests with channels or `sync.WaitGroup{}`. They are faster and more stable in that way.

If you really need to wait for something, it’s better to use assert.Eventually instead of a sleep.

- Eventually asserts that given condition will be met in waitFor time, periodically checking target function each tick.
    ```
    assert.Eventually(
        t,
        func() bool { return true }, // condition 
        time.Second, // waitFor 
        10*time.Millisecond, // tick
    )
    ```
  godoc.org/github.com/stretchr/testify/assert (https://godoc.org/github.com/stretchr/testify/assert#Eventually)

### Running

Now, when our tests are implemented, it’s time to run them!

Before that, we need to start our container with Firebase and MySQL with docker-compose up.

I prepared make test command that runs tests in a consistent way (for example, -race flag). It can also be used in the
CI.

```shell

$ make test
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/auth [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/client   [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/genproto/trainer [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/genproto/users   [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/logs [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/server   [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/common/server/httperr   [no test files]
ok     github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/trainer 0.172s
ok     github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/trainer/domain/hour 0.031s
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/trainings   [no test files]
?      github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/internal/users   [no test files]
```

#### Running one test and passing custom params

If you would like to pass some extra params, to have a verbose output (-v) or execute exact test (-run), you can pass it
after make test --.

```shell
$ make test -- -v -run ^TestRepository/memory/testUpdateHour$
--- PASS: TestRepository (0.00s)
  --- PASS: TestRepository/memory (0.00s)
    --- PASS: TestRepository/memory/testUpdateHour (0.00s)
      --- PASS: TestRepository/memory/testUpdateHour/available_hour (0.00s) 
      --- PASS: TestRepository/memory/testUpdateHour/not_available_hour (0.00s) 
      --- PASS: TestRepository/memory/testUpdateHour/hour_with_training (0.00s)
PASS
```

If you are interested in how it is implemented, I’d recommend you check my [Makefile magic](https://bit.ly/2ZCt0mO).

### Debugging

Sometimes our tests fail in an unclear way. In that case, it’s useful to be able to easily check what data we have in
our database.

For SQL databases my first choice for that are mycli for [MySQL](https://www.mycli.net/install) and pgcli
for [PostgreSQL](https://www.pgcli.com/). I’ve added make mycli command to Makefile, so you don’t need to pass
credentials all the time.

```shell
$ make mycli

mysql user@localhost:db> SELECT * from `hours`;
+---------------------+--------------------+
| hour                | availability       |
|---------------------+--------------------|
| 2020-08-31 15:00:00 | available          |
| 2020-09-13 19:00:00 | training_scheduled |
| 2022-07-19 19:00:00 | training_scheduled |
| 2023-03-19 14:00:00 | available          |
| 2023-08-05 03:00:00 | training_scheduled |
| 2024-01-17 07:00:00 | not_available      |
| 2024-02-07 15:00:00 | available          |
| 2024-05-07 18:00:00 | training_scheduled |
| 2024-05-22 09:00:00 | available          |
| 2025-03-04 15:00:00 | available          |
| 2025-04-15 08:00:00 | training_scheduled |
| 2026-05-22 09:00:00 | training_scheduled |
| 2028-01-24 18:00:00 | not_available      |
| 2028-07-09 00:00:00 | not_available      |
| 2029-09-23 15:00:00 | training_scheduled |
+---------------------+--------------------+
15 rows in set
```

For Firestore, the emulator is exposing the UI at [localhost:4000/firestore](http://localhost:4000/firestore/)

![](./chapter08/ch0805.png)
<center>Figure 8.3: Firestore Console</center>

### First step for having well-tested application

The biggest gap that we currently have is a lack of tests on the component and E2E level. Also, a big part of the
application is not tested at all. We will fix that in the next chapters. We will also cover some topics that we skipped
this time.

But before that, we have one topic that we need to cover earlier – Clean/Hexagonal architecture! This approach will help
us organize our application a bit and make future refactoring and features easier to implement. Just to remind,
__the entire source code of Wild Workouts
is [available on GitHub](https://github.com/ThreeDotsLabs/wild-workouts-go-ddd-example/). You can run it locally and
deploy to Google Cloud with one command.__