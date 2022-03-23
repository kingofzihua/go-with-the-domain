## gRPC communication on Google Cloud Run

Robert Laszczak

In this chapter we show how you can **build robust, internal communication between your services using gRPC**. We also
cover some extra configuration required to set up authentication and TLS for the Cloud Run.

### Why gRPC?

Let’s imagine a story, that is true for many companies:

Meet Dave. Dave is working in a company that spent about 2 years on building their product from scratch. During this
time, they were pretty successful in finding thousands of customers, who wanted to use their product. They started to
develop this application during the biggest “boom” for microservices. It was an obvious choice for them to use that kind
of architecture. Currently, they have more than 50 microservices using HTTP calls to communicate with each other.

Of course, Dave’s company didn’t do everything perfectly. The biggest pain is that currently all engineers are afraid to
change anything in HTTP contracts. It is easy to make some changes that are not compatible or not returning valid data.
It’s not rare that the entire application is not working because of that. “Didn’t we built microservices to avoid that?”
– this is the question asked by scary voices in Dave’s head every day.

Dave already proposed to use OpenAPI for generating HTTP server responses and clients. But he shortly found that he
still can return invalid data from the API.

It doesn’t matter if it (already) sounds familiar to you or not. The solution for Dave’s company is simple and
straightforward to implement. **You can easily achieve robust contracts between your services by using gRPC**.

![](./chapter03/ch0301.png)

<center> Figure 3.1: gRPC logo</center>

The way of how servers and clients are generated from gRPC is way much stricter than OpenAPI. It’s needless to say that
it’s infinitely better than the OpenAPI’s client and server that are just copying the structures.

> It’s important to remember that gRPC doesn’t solve the **data quality problems**. In other words – you can still send data that is not empty, but doesn’t make sense. It’s important to ensure that data is valid on many levels, like the robust contract, contract testing, and end-to-end testing.

Another important “why” may be performance. You can find many studies that **gRPC may be even 10x faster than REST**.
When your API is handling millions of millions of requests per second, it may be a case for cost optimization. In the
context of applications like Wild Workouts, where traﬀic may be less than 10 requests/sec, **it doesn’t matter**.

To be not biased in favor of using gRPC, I tried to find any reason not to use it for internal communication. I failed
here:

- **the entry point is low**,
- adding a gRPC server doesn’t need any extra infrastructure work – it works on top of HTTP/2,
- it workis with many languages like Java, C/C++, Python, C#, JS,
  and [more](https://grpc.io/about/#officially-supported-languages-and-platforms)
- in theory, you are even able to use gRPC for the [frontend communication](https://grpc.io/blog/state-of-grpc-web/) (I
  didn’t test that),
- it’s “Goish” – **the compiler ensures that you are not returning anything stupid**.

Sounds promising? Let’s verify that with the implementation in Wild Workouts!

### Generated server

Currently, we don’t have a lot of gRPC endpoints in Wild Workouts. We can update trainer hours availability and user
training balance (credits).

Let’s check Trainer gRPC service. To define our gRPC server, we need to create trainer.proto file.

![](./chapter03/ch0302.png)

<center> Figure 3.2: Architecture </center>

```protobuf
syntax = "proto3";

package trainer;

import "google/protobuf/timestamp.proto";

service TrainerService {
  rpc IsHourAvailable(IsHourAvailableRequest) returns (IsHourAvailableResponse) {}
  rpc UpdateHour(UpdateHourRequest) returns (EmptyResponse) {}
}

message IsHourAvailableRequest {
  google.protobuf.Timestamp time = 1;
}

message IsHourAvailableResponse {
  bool is_available = 1;
}

message UpdateHourRequest {
  google.protobuf.Timestamp time = 1;

  bool has_training_scheduled = 2;
  bool available = 3;
}

message EmptyResponse {}
```

Source: trainer.proto on [GitHub](https://bit.ly/3blVhna)

The .proto definition is converted into Go code by using Protocol Buffer Compiler (protoc).

```makefile
.PHONY: proto
proto:
    protoc --go_out=plugins=grpc:internal/common/genproto/trainer -I api/protobuf api/protobuf/trainer.proto
    protoc --go_out=plugins=grpc:internal/common/genproto/users -I api/protobuf api/protobuf/users.proto
```

Source: Makefile on [GitHub](https://bit.ly/3aEAsnE)
> To generate Go code from .proto you need to install [protoc](https://grpc.io/docs/protoc-installation/) and protoc [Go Plugin](https://grpc.io/docs/quickstart/go/). A list of supported types can be found in Protocol Buffers Version 3 [Language Specification](https://developers.google.com/protocol-buffers/docs/reference/proto3-spec#fields). More complex built-in types like Timestamp can be found in Well-Known Types [list](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf)


This is how an example generated model looks like:

```go
package trainer

type UpdateHourRequest struct {
	Time                 *timestamp.Timestamp `protobuf:"bytes,1,opt,name=time,proto3" json:"time,omitempty"`
	HasTrainingScheduled bool                 `protobuf:"varint,2,opt,name=has_training_scheduled,json=hasTrainingScheduled,proto3" json:"has_training_scheduled,omitempty"`
	Available            bool                 `protobuf:"varint,3,opt,name=available,proto3" json:"available,omitempty"`
	XXX_NoUnkeyedLiteral struct{}             `json:"-"`
	// ... more proto garbage ;)
}
```

Source: trainer.pb.go on [GitHub](https://bit.ly/37yjVjc)

And the server:

```go
package trainer

type TrainerServiceServer interface {
	IsHourAvailable(context.Context, *IsHourAvailableRequest) (*IsHourAvailableResponse, error)
	UpdateHour(context.Context, *UpdateHourRequest) (*EmptyResponse, error)
}
```

Source: trainer.pb.go on [GitHub](https://bit.ly/3dwBjIN)

The difference between HTTP and gRPC is that in gRPC we don’t need to take care of what we should return and how to do
that. If I would **compare the level of confidence with HTTP and gRPC, it would be like comparing Python and Go**. This
way is much more strict, and it’s impossible to return or receive any invalid values – **compiler will let us know about
that**.

Protobuf also has built-in ability to handle deprecation of fields and handling backward-compatibility7. That’s pretty
helpful in an environment with many independent teams.

### Protobuf vs gRPC

> Protobuf (Protocol Buffers) is Interface Definition Language used by default for defining the service interface and the structure of the payload. Protobuf is also used for serializing these models to binary format.
>
> You can find more details about gRPC and Protobuf on gRPC [Concepts](https://grpc.io/docs/guides/concepts/) page.

Implementing the server works in almost the same way as in HTTP generated by [OpenAPI](./chapter02.md) – we need to
implement an interface (`TrainerServiceServer in that case).

```go
package main

type GrpcServer struct {
	db db
}

func (g GrpcServer) IsHourAvailable(ctx context.Context, req *trainer.IsHourAvailableRequest) (*trainer.IsHourAvailableResponse, error) {
	timeToCheck, err := grpcTimestampToTime(req.Time)
	if err != nil {
		return nil, status.Error(codes.InvalidArgument, "unable to parse time")
	}

	model, err := g.db.DateModel(ctx, timeToCheck)
	if err != nil {
		return nil, status.Error(codes.Internal, fmt.Sprintf("unable to get data model: %s", err))
	}

	if hour, found := model.FindHourInDate(timeToCheck); found {
		return &trainer.IsHourAvailableResponse{IsAvailable: hour.Available && !hour.HasTrainingScheduled}, nil
	}

	return &trainer.IsHourAvailableResponse{IsAvailable: false}, nil
}

```

Source: grpc.go on [GitHub](https://bit.ly/3pG9SPj)

As you can see, you cannot return anything else than `IsHourAvailableResponse`, and you can always be sure that you will
receive `IsHourAvailableRequest`. In case of an error, you can return one of predefined error codes10. They are more
up-to-date nowadays than HTTP status codes.

Starting the gRPC server is done in the same way as with HTTP server

```go
package main

func main() {

	// ...

	server.RunGRPCServer(func(server *grpc.Server) {
		svc := GrpcServer{firebaseDB}
		trainer.RegisterTrainerServiceServer(server, svc)
	})

	// ...

}
```

Source: main.go on [GitHub](https://bit.ly/2ZCKnDQ)

### Internal gRPC client

After our server is running, it’s time to use it. First of all, we need to create a client
instance. `trainer.NewTrainerServiceClient` is generated from `.proto`.

```go
package trainer

type TrainerServiceClient interface {
	IsHourAvailable(ctx context.Context, in *IsHourAvailableRequest, opts ...grpc.CallOption) (*IsHourAvailableResponse, error)
	UpdateHour(ctx context.Context, in *UpdateHourRequest, opts ...grpc.CallOption) (*EmptyResponse, error)
}

type trainerServiceClient struct {
	cc grpc.ClientConnInterface
}

func NewTrainerServiceClient(cc grpc.ClientConnInterface) TrainerServiceClient {
	return &trainerServiceClient{cc}
}

```

Source: trainer.pb.go on [GitHub](https://bit.ly/3k8V5vo)

To make generated client work, we need to pass a couple of extra options. They will allow handling:

- authentication
- TLS encryption
- “service discovery” (we use hardcoded names of services provided by [Terraform](./chapter14.md) via TRAINER_GRPC_ADDR
  env).

```go
package client

import (
	// ...
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/genproto/trainer"

// ...

func NewTrainerClient() (client trainer.TrainerServiceClient, close func() error, err error) {
	grpcAddr := os.Getenv("TRAINER_GRPC_ADDR")
	if grpcAddr == "" {
		return nil, func() error { return nil }, errors.New("empty env TRAINER_GRPC_ADDR")
	}
	opts, err := grpcDialOpts(grpcAddr)
	if err != nil {
		return nil, func() error { return nil }, err
	}
	conn, err := grpc.Dial(grpcAddr, opts...)
	if err != nil {
		return nil, func() error { return nil }, err
	}
	return trainer.NewTrainerServiceClient(conn), conn.Close, nil
}
```

Source: grpc.go on [GitHub](https://bit.ly/2M94vKJ)

After our client is created we can call any of its methods. In this example, we call `UpdateHour` while creating a
training.

```go
package main

import (
	"github.com/golang/protobuf/ptypes"
	// ...
	"github.com/pkg/errors"
	"gitlab.com/threedotslabs/wild-workouts/pkg/internal/genproto/trainer"
	// ...
)

type HttpServer struct {
	db            db
	trainerClient trainer.TrainerServiceClient
	usersClient   users.UsersServiceClient
}

// ...
func (h HttpServer) CreateTraining(w http.ResponseWriter, r *http.Request) {
	// ...
	timestamp, err := ptypes.TimestampProto(postTraining.Time)
	if err != nil {
		return errors.Wrap(err, "unable to convert time to proto timestamp")
	}
	_, err = h.trainerClient.UpdateHour(ctx, &trainer.UpdateHourRequest{
		Time:                 timestamp,
		HasTrainingScheduled: true,
		Available:            false,
	})
	if err != nil {
		return errors.Wrap(err, "unable to update trainer hour")
	}
	// ...
}
```

Source: http.go on [GitHub](https://bit.ly/3aDglGr)

### Cloud Run authentication & TLS

Authentication of the client is handled by Cloud Run out of the box. The simpler (and recommended by us) way is using
Terraform. We describe it in details in [Setting up infrastructure with Terraform (Chapter 14)](./chapter14.md).

One thing that doesn’t work out of the box is sending authentication with the request. Did I already mention that
standard gRPC transport is HTTP/2? For that reason, we can just use the old, good JWT for that.

To make it work, we need to implement `google.golang.org/grpc/credentials.PerRPCCredentials` interface. The
implementation is based on the official guide
from [Google Cloud Documentation](https://cloud.google.com/run/docs/authenticating/service-to-service#go).

![](./chapter03/ch0303.png)

<center>Figure 3.3: You can also enable Authentication from Cloud Run UI. You need to also grant the roles/run.invoker
role to service’s service account.</center> 

```go
package client

import (
	"context"
	"fmt"
	"strings"

	"cloud.google.com/go/compute/metadata"
	"github.com/pkg/errors"
	"google.golang.org/grpc/credentials"
)

type metadataServerToken struct {
	serviceURL string
}

func newMetadataServerToken(grpcAddr string) credentials.PerRPCCredentials {
	// based on https://cloud.google.com/run/docs/authenticating/service-to-service#go
	// service need to have https prefix without port
	serviceURL := "https://" + strings.Split(grpcAddr, ":")[0]

	return metadataServerToken{serviceURL}
}

// GetRequestMetadata is called on every request, so we are sure that token is always not expired
func (t metadataServerToken) GetRequestMetadata(ctx context.Context, in ...string) (map[string]string, error) {
	// based on https://cloud.google.com/run/docs/authenticating/service-to-service#go
	tokenURL := fmt.Sprintf("/instance/service-accounts/default/identity?audience=%s", t.serviceURL)
	idToken, err := metadata.Get(tokenURL)
	if err != nil {
		return nil, errors.Wrap(err, "cannot query id token for gRPC")
	}

	return map[string]string{
		"authorization": "Bearer " + idToken,
	}, nil
}

func (metadataServerToken) RequireTransportSecurity() bool {
	return true
}
```

Source: auth.go on [GitHub](https://bit.ly/3k62Y4N)

The last thing is passing it to the `[]grpc.DialOption` list passed when creating all gRPC clients.

It’s also a good idea to ensure that the our server’s certificate is valid with `grpc.WithTransportCredentials`.

Authentication and TLS encryption are disabled on the local Docker environment.

```go
package client

func grpcDialOpts(grpcAddr string) ([]grpc.DialOption, error) {
	if noTLS, _ := strconv.ParseBool(os.Getenv("GRPC_NO_TLS")); noTLS {
		return []grpc.DialOption{grpc.WithInsecure()}, nil
	}
	systemRoots, err := x509.SystemCertPool()
	if err != nil {
		return nil, errors.Wrap(err, "cannot load root CA cert")
	}
	creds := credentials.NewTLS(&tls.Config{
		RootCAs: systemRoots,
	})
	return []grpc.DialOption{
		grpc.WithTransportCredentials(creds),
		grpc.WithPerRPCCredentials(newMetadataServerToken(grpcAddr)),
	}, nil
}
```

Source: grpc.go on [GitHub](https://bit.ly/2ZAMMz2)

### Are all the problems of internal communication solved?

**A hammer is great for hammering nails but awful for cutting a tree. The same applies to gRPC or any other technique**.

**gRPC works great for synchronous communication, but not every process is synchronous by nature. Applying synchronous
communication everywhere will end up creating a slow, unstable system**. Currently, Wild Workouts doesn’t have any flow
that should be asynchronous. We will cover this topic deeper in the next chapters by implementing new features. In the
meantime, you can check [Watermill](http://watermill.io/) library, which was also created by us. It helps with building
asynchronous, event-driven applications the easy way.

![](./chapter03/ch0304.png)
<center>Figure 3.4: Watermill</center>

### What’s next?

Having robust contracts doesn’t mean that we are not introducing unnecessary internal communication. In some cases
operations can be done in one service in a simpler and more pragmatic way.

It is not simple to avoid these issues. Fortunately, we know techniques that are successfully helping us with that. We
will share that with you soon.