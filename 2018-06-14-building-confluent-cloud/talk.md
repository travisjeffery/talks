# Building Confluent Cloud

Travis Jeffery

- [twitter.com/travisjeffery](twitter.com/travisjeffery)
- [github.com/travisjeffery](github.com/travisjeffery)
- [medium.com/@travisjeffery](medium.com/@travisjeffery)

---

# Talk roadmap

How to build services on multiple regions, multiple clouds in a manageable way.

- What is Confluent Cloud
- How we built it
- Lessons learned
- Where we're heading
- Ask Me Anything

---

# Travis Jeffery

- Cloud Eng at Confluent
- Wrote my own Kafka in Go called Jocko
- Other projects: Timecop, Mocha, Clang-Format on Xcode
- Worked at Basecamp, Segment

My face on the internet ->

![right](~/Downloads/Important Things/ben-shahn-face-no-sig.jpeg)


---

# What is Confluent Cloud?

- Today: Kafka as a Service
- Available on AWS and GCP in many regions

![right](/Users/travis/Desktop/Screen Shot 2018-06-14 at 11.55.31 AM.png)

^ Kafka as a Service. So you can sign up, create a cluster, get a endpoint to point your Kafka producer and consumers to and we take care of the rest.

---

# What is Confluent Cloud?

Tomorrow:

- More services: Schema Registry, Connect, Streams, Confluent Control Center
- More features: Topic management, dashboard metrics, SSO auth, etc.
- More regions, more clouds

---

# Building it

The goal: Try to build a PaaS on multiple regions, multiple clouds in a manageable and cost efficient way.

---

# Starting point

- CLI in Python
- Orchestrated by Kubernetes
- Generated YAML with Jinja
- Shelled out to kubectl

---

# Problems with this setup

- No API to build on
- No UI - no user access, staff managed
- No infrastructure management
- No version consensus
- Hard to test
- Static configuration
- Lots of work to onboard to dev or use it

^ No customers to first few enterprise customers. I joined just after this was setup. And we started working on a new setup.

---

# Fresh start

- Libraries/APIs first
- Type-safe calls via Go and gRPC
- Kafka for async, secure messaging to other regions
- Terraform managed infrastructure/configuration
- Still orchestrated by Kubernetes

^ Build around libraries and APIs and having those as the source of truth rather than a CLI. Have type-safe code that checks calls and is easy to refactor.

---

# Request flow

![inline](~/Documents/cloud-3.png)

^ So what happens here is... Regional services include a sync service which creates objects in K8s and an operator which turns those k8s objects into Kafka clusters.

---

# Libraries first, then APIs, then CLIs

- Helps you focus on flexible, robust API
- Can put aside requirements of end program
- End program is a small layer tying together config and libs
- Accessed via HTTP/RPC and CLI

^ Starting with a library first helps you focus on writing a flexible and robust API, you don't have to completely ignore the requirements of the end program, but it can be beneficial to ignore it while working on the library. The resulting program we're building should ideally be nothing more than a small layer that ties together configuration and underlying libraries implemented to fulfill its needs.

---

# Go and gRPC

- Same lang as our infra, tighter integration and clients: Kubernetes, Terraform, Docker
- Type-safe calls
- Easy to run different API versions
- Defined/managed in protocol buffers
- Service clients for free

^ gRPC RPC framework released by Google. RPC means remote procedure call, so you make calls on something that works like local object while it's handling the networking transparently.

---

# Kafka

- Cross region, cross cloud, simple, and secure networking
- Central cluster in mothership
- SASL/PLAIN authentication
- All services just need to know its endpoint on the internet

^ Endpoint is available on the internet which means there's no work for services in other regions to hit it. Security is built into Kafka.

---

# Routing messages per region

![inline](~/Documents/router.png)

^ The regional consumer knows its cloud and region and checks if the router matches to know whether it should operate on the message or ignore it. Good setup because there's no overhead to adding regions. Problem is that you send messages to regions that don't need them which is why this isn't good for high ingress and egress. The service I work on which is a Kafka as a Service, uses this style to schedule clusters.

---

# Terraform

- Provisions infrastructure
- Ties configuration
- Secrets stored in/looked up from KMS

![right 75%](/Users/travis/Desktop/Screen Shot 2018-06-14 at 12.54.44 PM.png)

All it takes to add a new region ->

^ Infrastructure as code. So you define your infrastructure in Hashicorp's configuration language. For example: using AWS' provider - create an auto scaling group with 3 instances in this security group, using Datadog's provider - create a Kafka dashboard with these graphs and metrics, using Kubernetes' provider - create a deployment for our scheduler service.

---

# Billing

- Using Stripe for payments
- Subscription API was too limited - no postpay billing
- Wrote our own billing service
- Event table stores each change user made on their clusters with associated price per second
- Job runs next month, sums total from events, bills

---

# Bill item example

- Cluster created March 15th at a $0.10 price per second
- Cluster updated March 16th to a $0.20 price per second
- Cluster deleted March 17th

=> `[0.10 * (24*60*60)] + [0.20 * (24*60*60]`.

---

# What's next

- Schema Registry, Connect, Streams, Confluent Control Center
  - Metrics: Traffic metrics for users via Kafka and showno in our UI
- Tooling: Internal services for support, on-call, etc.
- Testing: Integration and UI

---

# Ask Me Anything

---

# Where to go

- Blog posts and open source from what we're building
- [github.com/travisjeffery](github.com/travisjeffery)
- [medium.com/@travisjeffery](medium.com/@travisjeffery)
  - medium.com/@travisjeffery
  - github.com/travisjeffery
- Get in touch if you're interested in working on this stuff
- Check out Confluent Cloud
