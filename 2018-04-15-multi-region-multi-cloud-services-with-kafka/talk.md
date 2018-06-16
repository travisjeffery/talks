# Building multi-region & multi-cloud services with Kafka

Travis Jeffery

- github.com/travisjeffery
- twitter.com/travisjeffery
- medium.com/@travisjeffery

---

# What we're talking about today

- Building services on multiple regions, multiple clouds in a manageable way
- Example: running services in AWS with others in GCP or us-west-2 & us-east-1
- 3 system architectures according to message ingress and egress

^ By ingress and egress I mean with respect to the kafka brokers, how many messages you're producing to kafka, how many consumers you have.

---

# Why?

---

# Why multiple regions?

- **Latency**: Customers want your service to be fast.
- **Compliance**: You want a global customer base but different countries have different laws saying where/how data is stored.
- **Availability**: Increase your resilience above what a single region's availability zones.

^ Compliance: Lots of companies including us are working on GDPR compliance for European countries. Availability: like if a natural disaster hit a region, availability zones won't help.

---

# Why multiple clouds?

- Features and services available differ on each cloud.
- Save money with betters deals on different vendors.
- Reduce dependency on your provider.

^ Features: I'm a big fan of Big Query. I've worked at a couple companies who started on AWS and later started to use Big Query.
Reduce dependency in case you wanted to switch or stop using a provider or start using a new one.

---

# How?

---

# A typical, naive system

---

## Request-response to each region & cloud

![inline](reqresp.png)

^ This is very high level and hides how messy such a system is.

---

## Request-response to each region & cloud

![inline](reqresp2.png)

^ If we zoom in, we see can start to see the issue. This is only with 3 services and two clouds, as you grow your complexity grows exponentially.

---

## Request-response to each region & cloud

Bad:

- Security: Secure all endpoints
- Routing: VPN tunnels, firewalls
- Service discovery: Must know where & how to msg

Good:

- Latency: Direct connections.

---

# A new, smarter system

---


# Using Kafka as central message hub

![inline](kafka.png)

^ Kafka solves the bad of request-response. Each service just needs to know where Kafka is; Kafka has various security protocols built-in SSL, SASL, Kerberos; and services just need to know about themselves, they don't need to know where other serivces are or the APIs of those other services. Messages are persisted. Has built in access control. Latency: not too much overhead, but can't beat direct.

---

## Message header routing: low ingress, low egress

![inline](mothership.png)

^ You'd run a consumer in each region.

---

## Message header routing: low ingress, low egress

![inline](router.png)

^ The regional consumer knows its cloud and region and checks if the router matches to know whether it should operate on the message or ignore it. Good setup because there's no overhead to adding regions. Problem is that you send messages to regions that don't need them which is why this isn't good for high ingress and egress. The service I work on which is a Kafka as a Service, uses this style to schedule clusters.

---

## Topic per cloud and region: high ingress, low egress

![inline](topic.png)

^ With this setup, each region gets their own topic. So messages are routed only where they need to be. And you're not wasting bandwidth unless you have lots of egress, that is lots of different consumers in each region. This setup is good if you have like one or two regional consumers that write the messages to a local database. There's is overhead added, when you add a region you need to create a topic.

---

##  Topic per cloud and region into local kafka clusters: high ingress, high egress ##

![inline](replicator.png)

^ With this setup, each region gets their own topic and their own local Kafka cluster. So if you have a lot of data to run through Kafka, and want to do a lot of processing that's specific to a region. This is the way to go. The issue is a lot of operational overhead. Having to run a Kafka cluster per region.

---


## Extending for when you need a request-response ##

- Problem: End-to-end latency, for example a UI with a live search field
- Solution: Give the UI a regional API to request directly

---

## Extending for when you need a request-response ##

![inline](direct.png)

^ For example we used this for our Confluent Control Center where people manage their kafka clusters. We just expose them an endpoint directly to the cluster. Otherwise using something like a live updating search field would suck.

---

# Questions? #

Absolutely anything: Kafka, gRPC, Terraform...

Can ask me after too.

---

## Where to go next ##

- Don't want to manage Kafka? Confluent Cloud: https://www.confluent.io/confluent-cloud/
- github.com/travisjeffery
- twitter.com/travisjeffery
- medium.com/@travisjeffery
