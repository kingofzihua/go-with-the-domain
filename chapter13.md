## Repository Secure by Design

Robert Laszczak

Thanks to the tests and code review, you can make your project bug-free. Right? Well... actually, probably not. That
would be too easy. These techniques lower the chance of bugs, but they can’t eliminate them entirely. But does it mean
we need to live with the risk of bugs until the end of our lives?

Over one year ago, I found a pretty [interesting PR](https://github.com/goharbor/harbor/pull/8917/files) in the harbor
project. This was a fix for the issue that
**allowed to create admin user by a regular user. This was obviously a severe security issue.**
Of course, automated tests didn’t found this bug earlier.

This is how the bugfix looks like:

```shell
        ua.RenderError(http.StatusBadRequest, "register error:"+err.Error())
        return
    }
+
+   if !ua.IsAdmin && user.HasAdminRole {
+       msg := "Non-admin cannot create an admin user."
+       log.Errorf(msg)
+       ua.SendForbiddenError(errors.New(msg))
+       return
+   }
+
    userExist, err := dao.UserExists(user, "username")
    if err != nil {
```

One if statement fixed the bug. Adding new tests also should ensure that there will be no regression in the future. Is
it enough?
**Did it secure the application from a similar bug in the future? I’m pretty sure it didn’t.**

The problem becomes bigger in more complex systems with a big team working on them. What if someone is new to the
project and forgets to put this if statement? Even if you don’t hire new people currently, they may be hired in the
future.
**You will probably be surprised how long the code you have written will live. **
We should not trust people to use the code we’ve created in the way it’s intended – they will not.

**In some cases, the solution that will protect us from issues like that is good design. Good design should not allow
using our code in an invalid way. **
Good design should guarantee that you can touch the existing code without any fear. People new to the project will feel
safer introducing changes.

In this chapter, I’ll show how I ensured that only allowed people would be able to see and edit a training. In our case,
a training can only be seen by the training owner (an attendee) and the trainer. I will implement it in a way that
doesn’t allow to use our code in not intended way. By design.

Our current application assumes that a repository is the only way how we can access the data. Because of that, I will
add authorization on the repository level.

**By that, we are sure that it is impossible to access this data by unauthorized users.**

> What is Repository tl;dr
>
> If you had no chance to read our previous chapters, repository is a pattern that helps us to abstract database implementation from our application logic.
>
> If you want to know more about its advantages and learn how to apply it in your project, read [The Repository Pattern](./chapter07.md) (Chapter 7).
>

But wait, is the repository the right place to manage authorization? Well, I can imagine that some people may be
skeptical about that approach. Of course, we can start some philosophical discussion on what can be in the repository
and what shouldn’t. Also, the actual logic of who can see the training will be placed in the domain layer. I don’t see
any significant downsides, and the advantages are apparent. In my opinion, pragmatism should win here.

> Tip
>
> What is also interesting in this book, we focus on business-oriented applications. But even if the Harbor project is a pure system application, most of the presented patterns can be applied as well. After introducing [Clean Architecture](./chapter09.md) to our team, our teammate used this approach in his game to abstract rendering engine.
>
> _(Cheers, Mariusz, if you are reading that!)_

### Show me the code, please!

To achieve our robust design, we need to implement three things:

1. Logic who can see the training ([domain layer](https://bit.ly/3kdFQBf),
2. Functions used to get the training (GetTraining in the [repository](https://bit.ly/3qE1TDP))),
3. Functions used to update the training (UpdateTraining in the [repository](https://bit.ly/2NqUhpB).

#### Domain layer

The first part is the logic responsible for deciding if someone can see the training. Because it is part of the domain
logic (you can talk about who can see the training with your business or product team), it should go to the domain
layer. It’s implemented with CanUserSeeTraining function.

It is also acceptable to keep it on the repository level, but it’s harder to re-use. I don’t see any advantage of this
approach – especially if putting it to the domain doesn’t cost anything.

```go
package training

// ...
type User struct {
	userUUID string
	userType UserType
}

// ...
type ForbiddenToSeeTrainingError struct {
	RequestingUserUUID string
	TrainingOwnerUUID  string
}

func (f ForbiddenToSeeTrainingError) Error() string {
	return fmt.Sprintf(
		"user '%s' can't see user '%s' training",
		f.RequestingUserUUID, f.TrainingOwnerUUID,
	)
}
func CanUserSeeTraining(user User, training Training) error {
	if user.Type() == Trainer {
		return nil
	}
	if user.UUID() == training.UserUUID() {
		return nil
	}
	return ForbiddenToSeeTrainingError{user.UUID(), training.UserUUID()}
}
```

Source: [user.go on GitHub](https://bit.ly/2NFEZgC)

#### Repository

Now when we have the CanUserSeeTraining function, we need to use this function. Easy like that.

```shell
func (r TrainingsFirestoreRepository) GetTraining(
    ctx context.Context,
    trainingUUID string,
+   user training.User,
) (*training.Training, error) {
    firestoreTraining, err := r.trainingsCollection().Doc(trainingUUID).Get(ctx)

    if status.Code(err) == codes.NotFound {
        return nil, training.NotFoundError{trainingUUID}
    }
    if err != nil {
        return nil, errors.Wrap(err, "unable to get actual docs")
    }

    tr, err := r.unmarshalTraining(firestoreTraining)
    if err != nil {
        return nil, err
    }
+
+   if err := training.CanUserSeeTraining(user, *tr); err != nil {
+       return nil, err
+   }
+
    return tr, nil
}
```

Source: [trainings_firestore_repository.go on GitHub](https://bit.ly/3qE1TDP)

Isn’t it too simple? Our goal is to create a simple, not complex, design and code.
**This is an excellent sign that it is deadly simple.**

We are changing UpdateTraining in the same way.

```shell
func (r TrainingsFirestoreRepository) UpdateTraining(
    ctx context.Context,
    trainingUUID string,
+   user training.User,
    updateFn func(ctx context.Context, tr *training.Training) (*training.Training, error),
) error {
    trainingsCollection := r.trainingsCollection()
    return r.firestoreClient.RunTransaction(ctx, func(ctx context.Context, tx *firestore.Transaction) error {
        documentRef := trainingsCollection.Doc(trainingUUID)

        firestoreTraining, err := tx.Get(documentRef)
        if err != nil {
            return errors.Wrap(err, "unable to get actual docs")
        }

        tr, err := r.unmarshalTraining(firestoreTraining)
        if err != nil {
            return err
        }
+
+       if err := training.CanUserSeeTraining(user, *tr); err != nil {
+           return err
+       }
+
        updatedTraining, err := updateFn(ctx, tr)
        if err != nil {
            return err
        }

        return tx.Set(documentRef, r.marshalTraining(updatedTraining))
    })
}
```

Source: [trainings_firestore_repository.go on GitHub](https://bit.ly/2NqUhpB)

And... that’s all! Is there any way that someone can use this in a wrong way? As long as the User is valid – no.

This approach is similar to the method presented in [Domain-Driven Design Lite](./chapter06.md) (Chapter 6). It’s all
about creating code that we can’t use in a wrong way.

This is how usage of `UpdateTraining` now looks like:

```go
package command

func (h ApproveTrainingRescheduleHandler) Handle(ctx context.Context, cmd ApproveTrainingReschedule) (err error) {
	defer func() {
		logs.LogCommandExecution("ApproveTrainingReschedule", cmd, err)
	}()

	return h.repo.UpdateTraining(
		ctx,
		cmd.TrainingUUID,
		cmd.User,
		func(ctx context.Context, tr *training.Training) (*training.Training, error) {
			originalTrainingTime := tr.Time()

			if err := tr.ApproveReschedule(cmd.User.Type()); err != nil {
				return nil, err
			}

			err := h.trainerService.MoveTraining(ctx, tr.Time(), originalTrainingTime)
			if err != nil {
				return nil, err
			}

			return tr, nil
		},
	)
}
```

Source: [approve_training_reschedule.go on GitHub](https://bit.ly/3scHx4W)

Of course, there are still some rules if Training can be rescheduled, but this is handled by the Training domain type.

### Handling collections

Even if this approach works perfectly for operating on a single training, you need to be sure that access to a
collection of trainings is properly secured. There is no magic here:

```go
package adapters

func (r TrainingsFirestoreRepository) FindTrainingsForUser(ctx context.Context, userUUID string) ([]query.Training, error) {
	query := r.trainingsCollection().Query.
		Where("Time", ">=", time.Now().Add(-time.Hour*24)).
		Where("UserUuid", "==", userUUID).
		Where("Canceled", "==", false)
	iter := query.Documents(ctx)

	return r.trainingModelsToQuery(iter)
}
```

Source: [trainings_firestore_repository.go on GitHub](https://bit.ly/3pEIXDF)

Doing it on the application layer with the `CanUserSeeTraining` function will be very expensive and slow. It’s better to
create a bit of logic duplication.

If this logic is more complex in your application, you can try to abstract it in the domain layer to the format that you
can convert to query parameters in your database driver. I did it once, and it worked pretty nicely.

But in Wild Workouts, it will add unnecessary complexity -
let’s [Keep It Simple, Stupid](https://en.wikipedia.org/wiki/KISS_principle).

### Handling internal updates

We often want to have endpoints that allow a developer or your company operations department to do some “backdoor”
changes. The worst thing that you can do in this case is creating any kind of “fake user” and hacks.

It ends with a lot of if statements added to the code from my experience. It also obfuscates the audit log (if you have
any). Instead of a “fake user”, it’s better to create a special role and explicitly define the role’s permissions.

If you need repository methods that don’t require any user (for Pub/Sub message handlers or migrations), it’s better to
create separate repository methods. In that case, naming is essential – we need to be sure that the person who uses that
method knows the security implications.

From my experience, if updates are becoming much different for different actors, it’s worth to even introduce a
separate[ CQRS Commands](./chapter10.md) per actor. In our case it may be `UpdateTrainingByOperations`.

### Passing authentication via context.Context

As far as I know, some people are passing authentication details via context.Context.

I highly recommend not passing anything required by your application to work correctly via context.Context. The reason
is simple – when passing values via context.Context, we lose one of the most significant Go advantages – static typing.
It also hides what exactly the input for your functions is.

If you need to pass values via context for some reason, it may be a symptom of a bad design somewhere in your service.
Maybe the function is doing too much, and it’s hard to pass all arguments there? Perhaps it’s the time to decompose
that?

### And that’s all for today!

As you see, the presented approach is straightforward to implement quickly.

I hope that it will help you with your project and give you more confidence in future development.

Do you see that it can help in your project? Do you think that it may help your colleagues? Don’t forget to share it
with them!