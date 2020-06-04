
<p align=center>
  <img alt="k8s" src="./resources/k8s.svg" />
</p>

#

[Kubernetes](https://kubernetes.io/) is an open-source system for automating deployment, scaling, and management of containerized applications.

Some of the key features of k8s are:

1. Service discovery & load balancing
2. Automated rollouts/rollbacks
3. Storage orchestration
4. Self-healing in case of problems
5. Horizontal scaling
6. Secretes management
7. Configuration management

This is just some of the benefits system like k8s offers. From developer point of view, this are some of the most crucial ones, imo. From development point of view, there is some additional benefits that come with using `containers` and `k8s`. Yes, we could use the `docker` and `docker-compose` for most of the things that we are gonna mention. On the other hand, combining power of `docker` and benefits mentioned above from k8s, we are getting a really powerful tool in the arsenal.

Lets reflect on the benefits that we have using tools like `docker` and `containers` during development:

<p align=center>
  <img alt="benefits of containers" src="./resources/benefits_of_containers.svg" />
</p>

With this out of the way, lets see some really useful benefits of the k8s for developers:

<p align=center>
  <img alt="benefits of k8s" src="./resources/benefits_of_k8s.svg" />
</p>

Some of the reasons you would like to run k8s during development lifecycle:

1. Emulating environment setup (probably just partially) locally
2. E2E testing
3. Load testing
4. Troubleshooting

One of the personal examples on when it is useful to run the k8s locally was implementing a `CronJob` for a feature I was working on. When run with `docker-compose` the container was behaving without any issues, but when finally deployed to the test environment (that runs on k8s) it would almost it would end up in a `CrashLoopBackOff` on the start. After some time trying to figure out (logging was also not showing any useful information about failure), I started the k8s locally and deployed my `CronJob`. After just minutes into looking into the problem, it was obvious what was the problem. The `secrete` injector was having a problem with one of the secretes being used for the `CronJob`.

Another example is when you start playing around with optimizing your cluster and defining resources (CPU/RAM) for your `pods`. It can be useful to run the setup locally and put under a load test to see what are some baseline numbers you can configure for proper scaling (and how many replicas is a good start for your scenario).

## [Desired state and k8s](./docs/desired_state.md)

## [Running k8s locally](./docs/running_k8s_locally.md)

## [Pods](./docs/pods.md)

## [Deployments](./docs/deployments.md)

## [Services](/docs/services.md)

## [Storage](./docs/storage.md)

## [ConfigMaps](./docs/configmaps.md)

## [Secrets](./docs/secrets.md)

## [Namespaces](./docs/namespaces.md)

## [Tips & Tricks](./docs/tips_and_tricks.md)

## [Tools](./docs/tools.md)
