## Introduction to Strategic DDD

Robert Laszczak

A couple of years ago, I worked in a SaaS company that suffered from probably all possible issues with software
development. Code was so complex that adding simples changes could take months. All tasks and the scope of the project
were defined by the project manager alone. Developers didn’t understand what problem they were solving. Without an
understanding the customer’s expectations, many implemented functionalities were useless. The development team was also
not able to propose better solutions.

Even though we had microservices, introducing one change often required changes in most of the services. The
architecture was so tightly coupled that we were not able to deploy these “microservices” independently. The business
didn’t understand why adding “one button” may take two months. In the end, stakeholders didn’t trust the development
team anymore. We all were very frustrated. But the situation was not hopeless.

I was lucky enough to be a bit familiar with Domain-Driven Design. I was far from being an expert in that field at that
time. But my knowledge was solid enough to help the company minimize and even eliminate a big part of mentioned
problems.

Some time has passed, and these problems are not gone in other companies. Even if the solution for these problems exists
and is not arcane knowledge. People seem to not be aware of that. Maybe it’s because old techniques like GRASP (1997),
SOLID (2000) or DDD (Domain-Driven Design) (2003) are often forgotten or considered obsolete? It reminds me of the
situation that happened in the historical Dark Ages, when the ancient knowledge was forgotten. Similarly, we can use the
old ideas. They’re still valid and can solve present-day issues, but they’re often ignored. It’s like we live in
Software Dark Ages.

Another similarity is focusing on the wrong things. In the historical Dark Ages, religion put down science. In Software
Dark Ages, infrastructure is putting down important software design techniques. I’m not claiming that religion is not
important. Spiritual life is super important, but not if you are suffering from hunger and illness. It’s the same case
for the infrastructure. Having awesome Kubernetes cluster and most fancy microservices infrastructure will not help you
if your software design sucks.