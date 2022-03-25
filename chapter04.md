## Authentication using Firebase / 使用 Firebase 进行身份验证

Robert Laszczak / 罗伯特·拉斯扎克

We intentionally built the current version of the application to make it hard to maintain and develop in the future. In
a subtle way.

我们特意构建了当前版本的应用程序，以使其在未来难以维护和开发。以一种微妙的方式。

In the next chapter we will start the refactoring process. **We will show you how subtle issues and shortcuts can become
an issue in the long term. Even adding a new feature now may not be as easy, as developing greenfield features (even if
the application is still pretty new)**.

在下一章中，我们将开始重构过程。**我们将向你展示微妙的问题和捷径如何在长期内成为一个问题。即使现在添加新功能也可能不像开发新功能那样容易（即使应用程序仍然很新）**

But before that, we have one important topic to cover – authentication. This is the part of the application that will
not need refactoring.

但在此之前，我们有一个重要的话题要讨论——身份验证。这是应用程序中不需要重构的部分。

### You shouldn’t reinvent the wheel / 你不应该重新造轮子

(I hope that) you are not creating a programming language, and a framework for every project you work on. Even if you do
that, **besides losing a lot of time, it is not harmful. That case is not the same for authentication. Why**?

(我希望）你不是在为你所做的每个项目创建一种编程语言，和一个框架。即使你这样做，除了浪费很多时间，它也没有害处。这种情况与身份验证不同。为什么

Let’s imagine that you are working in a company implementing a cryptocurrency market. At the beginning of the project,
you decided to build your own authentication mechanism. The first reason for that is your boss, who doesn’t trust
external authentication providers. The second reason is that you believe that it should be simple.

让我们想象一下，你在一家实施加密货币市场的公司工作。在项目开始的时候，你决定建立自己的认证机制。第一个原因是你的老板，他不信任外部认证供应商。第二个原因是，你认为它应该是简单的。

“You did it many times” – this is what strange voices in your head repeat every time.

“你已经做了很多次” —— 你脑海中每次都有这种奇怪的声音。

Have you seen the [Mayday / Air Crash Investigation](https://en.wikipedia.org/wiki/List_of_Mayday_episodes) documentary
TV series? In the Polish edition, every episode starts with more or less: “Catastrophes are not a matter of coincidence,
but series of unfortunate events”. In programming, this sentence is surprisingly true.

你看过 [Mayday / Air Crash Investigation](https://en.wikipedia.org/wiki/List_of_Mayday_episodes)
纪录片电视剧吗？在波兰版中，每一集的开头或多或少都有。"灾难不是巧合，而是一系列不幸的事件"。在编程中，这句话出乎意料地真实。

In our hypothetical project, in the beginning, your company decided to introduce a service account that was able to move
funds between every account. Even from a wallet that, in theory, should be offline. Of course, temporarily. That doesn’t
sound like a good idea, but it simplified customer support from the very beginning.

假设我们有一个项目，一开始，你的公司决定引入一个服务账户，能够在每个账户之间移动资金。甚至从理论上来说，应该是离线的钱包。当然，是暂时的。这听起来不像是一个好主意，但它从一开始就简化了客户支持。

With time everyone forgot about this feature. Until one day, when hackers found a bug in the authentication mechanism,
that allowed them to hijack any account in our market. Including the service account.

随着时间的推移，每个人都忘记了这个功能。直到有一天，黑客在认证机制中发现了一个错误，使他们能够劫持我们市场上的任何账户。包括服务账户。

Our boss and coworkers were less happy than hackers because they lost all the money. The problem would be not so big if
we would lose “only” all company’s money. In that case, we also lost all our customers’ money.

我们的老板和同事比黑客更不高兴，因为他们失去了所有的钱。如果我们 "只 "失去公司所有的钱，问题就不会那么大。在这种情况下，我们也失去了所有客户的钱。

![](./chapter04/ch0401.png)

<center>Figure 4.1: Aannnd it’s gone</center>

<center>Figure 4.1: 然后它就消失了</center>


This example may sound a bit extreme and unlikely to happen. Is it a rare
scenario? [OWASP Top Ten report](https://owasp.org/www-project-top-ten/) lists Broken Authentication at **second
place**!

- **Broken Authentication**. Application functions related to authentication and session manage- ment are often
  implemented incorrectly, allowing attackers to compromise passwords, keys, or ses- sion tokens, or to exploit other
  implementation flaws to assume other users’ identities temporarily or permanently

这个例子听起来有点极端，不太可能发生。这是一种罕见的情况吗？ [OWASP 十大报告](https://owasp.org/www-project-top-ten/)  将 Broken Authentication 列为第二名！

- **Broken Authentication**. 与身份验证和会话管理相关的应用程序功能经常被错误地实现，允许攻击者破坏密码、密钥或会话令牌，或利用其他实现缺陷来临时或永久地假设其他用户的身份

You may say: “My company is just selling toasters! Why should I care?”. I’m pretty sure that you still care about your
company’s image. **Discreditation after an incident caused by hackers hijacking your customers’ accounts is always
embarrassing**.

你可能会说：“我的公司只卖烤面包机！我为什么要在乎？”。我很确定您仍然关心您公司的形象。黑客劫持您客户的帐户造成的事件后的名誉扫地总是令人尴尬的。

Do you still feel that you can implement a perfect authentication? Even giants with hundreds of researchers and
developers working just on authentication are not able to do that. In March 2020, a
researcher [found a Facebook vulnerability](https://latesthackingnews.com/2020/03/03/10-year-old-facebook-oauth-framework-flaw-discovered/)
that could allow hijacking anyone’s Facebook account. Just a few months later, a bug in Apple Sign In mechanism allowed
[full account takeover of user accounts on that third party application](https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple/)
.

你还觉得你能实现一个完美的认证吗？即使是拥有数百名研究人员和开发人员只从事认证工作的巨头也无法做到这一点。2020年3月，一位研究人员发现了一个 [Facebook 的漏洞](https://latesthackingnews.com/2020/03/03/10-year-old-facebook-oauth-framework-flaw-discovered/)
，可以允许劫持任何人的Facebook账户。就在几个月后，苹果登录机制的一个漏洞允许 [完全接管该第三方应用程序上的用户账户](https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple/)
。

If you are still not convinced, it’s worth to consider saving your time. I’m sure **customers of your company will be
much happier about giving them a long-awaited feature. Not fancy, custom authentication**.

如果你仍然不相信，值得考虑节省你的时间。我相信你公司的客户会对给他们一个期待已久的功能而感到更高兴。不是花哨的，自定义的认证。

### Using Firebase authentication / 使用 Firebase 身份验证

There are, of course, many solutions that can be used for implementing authentication. **In this chapter we want to go
with the fastest and easiest way for our context**. We are also not considering any vendor lock-in problems – we will
address that in the future.

当然，有许多解决方案可用于实现身份验证。**在这一章中，我们想用最快和最简单的方式来处理我们的情况。**我们也没有考虑任何供应商锁定问题 -- 我们将在未来解决这个问题。

> If you are looking for a simple solution, not dependent on any provider, you should check [JWT](https://jwt.io/) and [dgrijalva/jwt-go library](https://github.com/dgrijalva/jwt-go).
>
> 如果你正在寻找一个简单的解决方案，不依赖于任何供应商，你应该查看 [JWT](https://jwt.io/) 和 [dgrijalva/jwt-go library](https://github.com/dgrijalva/jwt-go) 库。


The Wild Workouts example application that we created for this ebook is now hosted on Firebase hosting. It’s a natural
choice to use [Firebase Authentication](https://firebase.google.com/docs/auth). There is one significant advantage of
this solution – it works almost totally out of the box both from the backend and frontend side.

我们为这本电子书创建的Wild Workouts示例应用程序现在已经托管在Firebase主机上。使用 [Firebase认证](https://firebase.google.com/docs/auth)
是一个自然的选择。这个解决方案有一个显著的优势--它在后台和前台方面几乎完全是开箱即用。

> Deployment of the project is described in detail in [Setting up infrastructure with Terraform](./chapter14.md). [In Building a serverless application with Google Cloud Run and Firebase](./chapter02.md) you can find deployment tl;dr at the end.
>
> 项目的部署在[《用Terraform建立基础设施》](./chapter14.md) 中做了详细描述。在[《用Google Cloud Run和Firebase构建无服务器应用程序》](./chapter02.md)中，你可以在结尾处找到部署要点。
>
> During the setup, don’t forget about enabling Email/Password Sign-in provider in [Authentication / Sign-in method tab in the Firebase Console!](https://console.firebase.google.com/project/_/authentication/providers)
>
> 在设置过程中，不要忘记在 [Firebase 控制台的认证/登录方式选项卡](https://console.firebase.google.com/project/_/authentication/providers) 中启用电子邮件/密码登录提供者的功能。

### Frontend 前端

The first thing that we need to do
is [the initialization of Firebase SDK](https://firebase.google.com/docs/web/setup#add-sdks-initialize).

我们需要做的第一件事是 [Firebase SDK初始化](https://firebase.google.com/docs/web/setup#add-sdks-initialize) 。

Next, in the form on the main page, we call the `loginUser` function. This function calls `Auth.login`,
`Auth.waitForAuthReady` and `Auth.getJwtToken`. The result is set to OpenAPI generated clients by `setApiClientsAuth`.

接下来，在主页面的表单中，我们调用 `loginUser` 函数。这个函数调用 `Auth.login`、`Auth.waitForAuthReady` 和 `Auth.getJwtToken`。
其结果通过 `setApiClientsAuth` 设置给 OpenAPI 生成的客户端。

```js
// ...
export function loginUser(login, password) {
    return Auth.login(login, password)
        .then(function () {
            return Auth.waitForAuthReady()
        })
        .then(function () {
            return Auth.getJwtToken(false)
        })
        .then(token => {
            setApiClientsAuth(token)
        });
    // ...
}
```

Source: [user.js on GitHub](https://bit.ly/3k5ivSq)

`Auth` is a class with two implementations: Firebase and mock. We will go through mock implementation later – let’s
focus on Firebase now.

`Auth` 是一个有两种实现的类。Firebase 和 mock 。我们将在后面讨论mock的实现 -- 现在让我们专注于Firebase。

To log-in we need to call `firebase.auth().signInWithEmailAndPassword`. If everything is
fine, `auth().currentUser.getIdToken` returns our JWT token.

为了登录，我们需要调用 `firebase.auth().signInWithEmailAndPassword `。如果一切正常，`auth().currentUser.getIdToken` 会返回我们的 JWT 令牌。

```js
class FirebaseAuth {
    login(login, password) {
        return firebase.auth().signInWithEmailAndPassword(login, password)
    }

    waitForAuthReady() {
        return new Promise((resolve) => {
            firebase
                .auth()
                .onAuthStateChanged(function () {
                    resolve()
                });
        })
    }

    getJwtToken(required) {
        return new Promise((resolve, reject) => {
            if (!firebase.auth().currentUser) {
                if (required) {
                    reject('no user found')
                } else {
                    resolve(null)
                }
                return
            }
            firebase.auth().currentUser.getIdToken(false)
                .then(function (idToken) {
                    resolve(idToken)
                })
                .catch(function (error) {
                    reject(error)
                });
        })
    }

    // ...
}
```

Source: [auth.js on GitHub](https://bit.ly/3aFCDYh)

> Firebase also provides out of the box support for logging in with most popular OAuth providers like Facebook, Gmail, or GitHub.
>
> Firebase还提供了开箱即用的支持，可以使用大多数流行的 OAuth 提供商，如 Facebook 、Gmail 或 GitHub 进行登录。

Next, we need to set `authentications['bearerAuth'].accessToken` of [OpenAPI generated](./chapter02.md) clients
attribute to the JWT token received from `Auth.getJwtToken(false)`. Now we need to set this token to OpenAPI clients and
voilà! All our requests are now authenticated.

接下来，我们需要将 [OpenAPI生成](./chapter02.md) 的客户端 属性的 `authentications['bearerAuth'].accessToken`
设置为从 `Auth.getJwtToken(false)` 收到的JWT令牌。现在我们需要将这个令牌设置为 OpenAPI 客户端，然后就可以了! 我们所有的请求现在都被认证了。

```js
export function setApiClientsAuth(idToken) {
    usersClient.authentications['bearerAuth'].accessToken = idToken
    trainerClient.authentications['bearerAuth'].accessToken = idToken
    trainingsClient.authentications['bearerAuth'].accessToken = idToken
}
```

Source: [auth.js on GitHub](https://bit.ly/3drjzyG)

> If you are creating your own OpenAPI spec, it will not work without proper [authentication definition](https://swagger.io/docs/specification/authentication/bearer-authentication/). In Wild Workouts’ [OpenAPI spec](https://bit.ly/3qXbiqh) it’s already done.
>
> 如果你正在创建你自己的 OpenAPI 规范，没有适当的 [认证定义](https://swagger.io/docs/specification/authentication/bearer-authentication/) ，它将无法工作。在Wild Workouts的 [OpenAPI spec](https://bit.ly/3qXbiqh) 中，它已经完成了。

If you would like to know more, I would recommend you to
check [Firebase Auth API reference](https://firebase.google.com/docs/reference/js/firebase.auth).

如果您想了解更多信息，我建议您查看 [Firebase Auth API reference](https://firebase.google.com/docs/reference/js/firebase.auth) 参考。

### Backend / 后端

The last part is actually using this authentication in our HTTP server. I created a simple HTTP middleware that will do
that task for us.

最后一部分是在我们的HTTP服务器中实际使用这种认证。我创建了一个简单的HTTP中间件，它将为我们完成这一任务。

This middleware does three things:

这个中间件做了三件事：

1. Get token from HTTP header。 / 从 HTTP 标头获取令牌。
2. Verify token with Firebase auth client。 / 使用 Firebase 身份验证客户端验证令牌。
3. Save user data in credentials。 / 将用户数据保存在凭证中。

```go
package auth

import (
	"context"
	"firebase.google.com/go/auth"
	"github.com/pkg/errors"
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/server/httperr"
	"net/http"
	"strings"
)

type FirebaseHttpMiddleware struct {
	AuthClient *auth.Client
}

func (a FirebaseHttpMiddleware) Middleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ctx := r.Context()
		bearerToken := a.tokenFromHeader(r)
		if bearerToken == "" {
			httperr.Unauthorised("empty-bearer-token", nil, w, r)
			return
		}
		token, err := a.AuthClient.VerifyIDToken(ctx, bearerToken)
		if err != nil {
			httperr.Unauthorised("unable-to-verify-jwt", err, w, r)
			return
		}
		// it's always a good idea to use custom type as context value (in this case ctxKey)
		// because nobody from the outside of the package will be able to override/read this value
		ctx = context.WithValue(ctx, userContextKey, User{
			UUID:        token.UID,
			Email:       token.Claims["email"].(string),
			Role:        token.Claims["role"].(string),
			DisplayName: token.Claims["name"].(string),
		})
		r = r.WithContext(ctx)
		next.ServeHTTP(w, r)
	})
}

type ctxKey int

const (
	userContextKey ctxKey = iota
)

// ...
func UserFromCtx(ctx context.Context) (User, error) {
	u, ok := ctx.Value(userContextKey).(User)
	if ok {
		return u, nil
	}
	return User{}, NoUserInContextError
}
```

Source: [http.go on GitHub](https://bit.ly/2NoUda0)

User data can now be accessed in every HTTP request by using `auth.UserFromCtx` function.

现在可以通过使用 `auth.UserFromCtx` 函数在每个HTTP请求中访问用户数据。

```go
package main

func (h HttpServer) GetTrainings(w http.ResponseWriter, r *http.Request) {
	user, err := auth.UserFromCtx(r.Context())
	if err != nil {
		httperr.Unauthorised("no-user-found", err, w, r)
		return

	}
	// ...
}
```

Source: [http.go on GitHub](https://bit.ly/2OOyhFy)

We can also limit access to some resources, based on the user role.

我们还可以根据用户角色限制对某些资源的访问。

```go
package main

func (h HttpServer) MakeHourAvailable(w http.ResponseWriter, r *http.Request) {
	user, err := auth.UserFromCtx(r.Context())
	if err != nil {
		httperr.Unauthorised("no-user-found", err, w, r)
		return
	}
	if user.Role != "trainer" {
		httperr.Unauthorised("invalid-role", nil, w, r)
		return
	}
	//..
}

```

Source: [http.go on GitHub](https://bit.ly/3k6hPMq)

### Adding users / 添加用户

In our case, we add users during the start of the `users service`. You can also add them from the FirebaseUI.
Unfortunately, via UI you cannot set all required data, like claims – you need to do it via API.

在我们的例子中，我们在启动 `users service` 的时候添加用户。你也可以从 FirebaseUI 中添加他们。不幸的是，通过用户界面你不能设置所有需要的数据，比如索赔 - 你需要通过API来做。

```go
package main

func createFirebaseUsers() ([]string, error) {

	//...
	config := &firebase.Config{ProjectID: os.Getenv("GCP_PROJECT")}
	firebaseApp, err := firebase.NewApp(context.Background(), config, opts...)
	if err != nil {
		return nil, err
	}

	authClient, err := firebaseApp.Auth(context.Background())
	if err != nil {
		return nil, err
	}

	// ...

	for _, user := range usersToCreate {
		userToCreate := (&auth.UserToCreate{}).
			Email(user.Email).
			Password("123456").
			DisplayName(user.DisplayName)
		createdUser, err := authClient.CreateUser(context.Background(), userToCreate)
		// ...
		err = authClient.SetCustomUserClaims(context.Background(), createdUser.UID, map[string]interface{}{
			"role": user.Role,
		})
		//...
	}
	//..
}
```

Source: [fixtures.go on GitHub](https://bit.ly/3siayw5)

![](./chapter04/ch0402.png)
<center>Figure 4.2: Firestore Console</center>

<center>Figure 4.2: Firestore 控制台</center>

Mock Authentication for local dev

本地开发环境模拟身份验证

There is high demand and a [discission going around](https://github.com/firebase/firebase-tools/issues/1677) support for
Firebase Authentication emulator. Unfortunately, it doesn’t exist yet. The situation is pretty similar to Firestore here
– I want to be able to run my application locally without any external dependencies. As long as there is no emulator,
there is no other way than to implement a simple mock implementation.

对 Firebase 身份验证模拟器的支持有很高的需求和 [争议](https://github.com/firebase/firebase-tools/issues/1677) 。不幸的是，它还不存在。这种情况与此处的 Firestore
非常相似——我希望能够在本地运行我的应用程序，而无需任何外部依赖。只要没有模拟器，就只能实现一个简单的 mock 实现。

There is nothing really complicated in both backend and frontend implementation. [Frontend’s](https://bit.ly/3pHyEi8)
`getJwtToken` is implemented by just generating JWT token with mock `secret`. [Backend](https://bit.ly/3shJcGf) instead
of calling Firebase to verify the token is checking if JWT was generated with mock `secret`.

后端和前端实现都没有什么复杂的。[前端](https://bit.ly/3pHyEi8)的 `getJwtToken` 是通过使用模拟密钥生成 JWT 令牌来实现的。[后端](https://bit.ly/3shJcGf) 不是调用
Firebase 来验证令牌，而是检查 JWT 是否是使用模拟密钥生成的。

It gives us some confidence if our flow is implemented more or less correctly. But is there any option to test it with
production Firebase locally?

这给了我们一些信心，如果我们的流程实现得或多或少是正确的。但是有什么办法可以在本地用生产的Firebase进行测试吗？

### Firebase Authentication for local dev / 用于本地开发的 Firebase 认证

Mock authentication does not give us 100% confidence that the flow is working correctly. We should be able to test
Firebase Authentication locally. To do that, you need to do some extra steps.

模拟身份验证不能让我们 100% 确信流程正常工作。我们应该能够在本地测试 Firebase 身份验证。为此，您需要执行一些额外的步骤。

**It is not straightforward. If you are not changing anything in the authentication, you can
probably [skip this part](./chapter05.md).**

这并不简单。如果您不更改身份验证中的任何内容，您可以跳过这部分。

First of all, you need to generate a service account file into the repository.

首先，你需要生成一个服务账户文件到存储库。

> Remember to not share it with anyone! Don’t worry, it’s already in `.gitignore`.
>
> 切记不要与任何人分享！别担心，它已经在 `.gitignore` 中了。

```shell
gcloud auth login

gcloud iam service-accounts keys create service-account-file.json --project [YOUR PROJECT ID] --iam-account [YOUR PROJECT ID]@appspot.gserviceaccount.com
```

Next, you need to uncomment **all** lines linking service account in `docker-compose.yml`

接下来，你需要取消对 `docker-compose.yml` 中连接服务账户的所有行的注释

```shell
     volumes:
       - ./pkg:/pkg
-#      - ./service-account-file.json:$SERVICE_ACCOUNT_FILE
+      - ./service-account-file.json:$SERVICE_ACCOUNT_FILE
     working_dir: /pkg/trainer
     ports:
       - "127.0.0.1:3000:$PORT"

       context: ./dev/docker/app
     volumes:
       - ./pkg:/pkg
-#      - ./service-account-file.json:$SERVICE_ACCOUNT_FILE
+      - ./service-account-file.json:$SERVICE_ACCOUNT_FILE
     working_dir: /pkg/trainer

@ ... do it for all services!
```

Source: [docker-compose.yml on GitHub](https://bit.ly/37zdoVl)

After that, you should set `GCP_PROJECT` to your project id, uncomment `SERIVCE_ACCOUNT_FILE` and set `MOCK_AUTH`
to `false` in `.env`

之后，你应该将 `GCP_PROJECT` 设置为你的项目ID，取消对 `SERIVCE_ACCOUNT_FILE` 注释，并在 `.env` 中设置 `MOCK_AUTH` 为 `false`。

```shell
-GCP_PROJECT=threedotslabs-cloudnative
+GCP_PROJECT=YOUR-PROJECT-ID
 
 PORT=3000
 
-FIRESTORE_PROJECT_ID=threedotslabs-cloudnative
+FIRESTORE_PROJECT_ID=YOUR-PROJECT-ID
 FIRESTORE_EMULATOR_HOST=firestore:8787
 # env used by karhoo/firestore-emulator container
-GCP_PROJECT_ID=threedotslabs-cloudnative
+GCP_PROJECT_ID=YOUR-PROJECT-ID
 
 TRAINER_GRPC_ADDR=trainer-grpc:3000
 USERS_GRPC_ADDR=users-grpc:3000

@ ...
 
 CORS_ALLOWED_ORIGINS=http://localhost:8080
 
-#SERVICE_ACCOUNT_FILE=/service-account-file.json
-MOCK_AUTH=true
+SERVICE_ACCOUNT_FILE=/service-account-file.json
+MOCK_AUTH=false
 LOCAL_ENV=true
```

Source: [.env on GitHub](https://bit.ly/37PNRrp)

```shell
-const MOCK_AUTH = process.env.NODE_ENV === 'development'
+const MOCK_AUTH = false”
```

Source: [auth.js on GitHub](https://bit.ly/3k9K0uc)

Now you need to go to [Project Settings](https://console.firebase.google.com/project/_/settings/general) (you can find
it after clicking the gear icon right to Project overview) and update your Firebase settings
in `web/public/__/firebase/init.json`. 现在你需要进入[项目设置](https://console.firebase.google.com/project/_/settings/general)（
你可以在点击项目概览右边的齿轮图标后找到它）并更新你的 Firebase 设置在 `web/public/__/firebase/init.json`。

![](./chapter04/ch0403.png)

<center>Figure 4.3: Firestore Console</center>

<center>Figure 4.3: Firestore 控制台</center>

```shell
- "apiKey": "",
- "appId": "",
- "authDomain": "",
- "databaseURL": "",
- "messagingSenderId": "",
- "projectId": "",
- "storageBucket": ""
+ "apiKey": "AIzaSyBW6TNG5Xdq1F7MleYml8CovOej5SbxXhw",
+ "authDomain": "wild-workouts-6.firebaseapp.com",
+ "databaseURL": "https://wild-workouts-6.firebaseio.com",
+ "projectId": "wild-workouts-6",
+ "storageBucket": "wild-workouts-6.appspot.com",
+ "messagingSenderId": "519519305315",
+ "appId": "1:519519305315cde76e4847f0d95cc84256"
}

```

Source: [init.json on GitHub](https://bit.ly/2NLBghq)

**The last thing is stopping and starting again docker-compose** to reload envs and links.

**最后停止并重新启动 `docker-compose` **以重新加载环境和链接。

### And that’s all / 就这样

As you can see, the setup was really straightforward. We were able to save a lot of our time. **The situation could be
more complicated if we wouldn’t like to host our application on Firebase**. Our idea from the beginning was to prepare a
simple base for future chapters.

正如你所看到的，设置真的很简单明了。我们能够节省大量的时间。**如果我们不愿意将我们的应用程序托管在Firebase上，情况可能会更复杂。** 我们的想法从一开始就是为未来的章节准备一个简单的基础。

We didn’t exhaust the topic of authentication in this chapter. But at least for now, we don’t plan to explore
authentication deeper.

在本章中，我们并没有列出所有认证的话题。但至少在目前，我们不打算对认证进行更深入的探讨。

In this book we would like to show you how to build applications that are easy to develop, maintain, and fun to work
with in the long term. Maybe you’ve seen examples of Domain-Driven Design or Clean Architecture in Go. Most of them are
not done in a really pragmatic way that works in the language context.

在这本书中，我们想告诉你如何构建易于开发、维护和长期工作的应用程序。也许你已经看到了 Go 中 `Domain-Driven Design` 或 `Clean Architecture`
的例子。他们中的大多数都没有以真正实用的方式在语言环境中工作。

**In following chapters, we will show patterns that we successfully use in teams that we lead for a couple years**. We
will also show when applying them makes sense, and when it’s over-engineering or CV-Driven Development. I’m sure that it
will force you to rethink your point of view for these techniques.

在下面的章节中，我们将展示我们在领导了几年的团队中成功使用的模式。我们还将展示什么时候应用它们是有意义的，什么时候是过度设计或 CV驱动 的开发。我相信，这将迫使你重新思考你对这些技术的观点。