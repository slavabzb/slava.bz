---
layout: post
title:  "HighLoad++ 2018"
date:   2018-08-30 12:41:38 +0300
categories: programming trends
---
# HighLoad++ 2018

* [What do we know about microservices?](#what-do-we-know-about-microservices)
* [Difficulties of containerized applications and microservices](#difficulties-of-containerized-applications-and-microservices)
* [A way to realize Data Discovery in microservice architecture](a-way-to-realize-data-discovery-in-microservice-architecture)
* [Delayed data as a way to provide consistency](#delayed-data-as-a-way-to-provide-consistency)
* [How search is organized in hh.ru](#how-search-is-organized-in-hh.ru)
* [How to deal with techdebt](#how-to-deal-with-techdebt)
* [Infrastructure development in Badoo](#infrastructure-development-in-badoo)

## What do we know about microservices?

*by Vadim Madison*, [Avito](https://www.avito.ru)

In microservice architecture we have:
- lots of services (> 100);
- lots of repositories;
- lots of teams that work separately;
- disjoint contexts (nobody knows the truth);
- data fragmentation.

There should be a proper way to create, organize and maintain microservices to keep the whole system up to date.

### Typical microservice pipeline

#### CLI-push

The first step is to create a service from scratch.

- Create an app from the template. Provide templates for different languages.
- Deploy DEV-environment on local machine by single CLI-command.
- Connect to databases (no configuration is required).
- Live-building: any changes in code made in IDE trigger rebuild machinary.
- Generate autotest stubs.
- Basic validation: documentation presence, dependencies, metrics and alerts (service owner decide what to monitor).

#### CI

Integrate the service into your environment.

- Prepare a repository.
- Generate pipeline in [TeamCity](https://www.jetbrains.com/teamcity/).
- Grant required access.
- Search the owner of the service (at least 2).
- Register service in [Atlas](https://atlas.hashicorp.com/).
- Migration checks: if the migration is probably dangerous, we will register a trigger in Atlas and put the migration into quarantine area.

#### Bake

Prepare to deploy.

- Create a [docker](https://www.docker.com/) image.
- Generate [Helm charts](https://github.com/helm/charts).
- Create admin tickets to complete service-specific setup.
- Run the tests and send coverage report to Atlas.
- Check CPU and RAM requirements.

#### Deploy

Check the conventions.

- Service's endpoints.
- Swagger schemas.
- Format of logs.
- Presence of required headers (X-Source-ID) in HTTP requests.
- Presence X-Source-ID when communicate through the bus.

#### Test

Run unit tests.

- Test in closed environment, no connections to external resources ([HoverFly](https://hoverfly.io/)).
- Load testing.
- Compare resource limits declared by service with real resource consumption. If the service consumes:
    - too low - check whether it's killed by OOM killer;
    - too many - the service cannot be utilized.

#### Canary

Test on real users.

- How many users - start canary testing from low percent of real users (< 0.1%).
- How long to test:
    - 5 mins (minimum);
    - 2 hours (usual);
    - customize time for sophisticated services (monoliths).
- What to check:
    - service-specific metrics;
    - [Sentry](https://sentry.io) errors;
    - response statuses;
    - response time (accurate and average);
    - latency;
    - exceptions (handled and unhandled);
    - ... (lots of product metrics).

#### Squeeze testing

Test by load.

- Load 1 brick until it's reached a point of failure.
- Add +1 brick, load 2 bricks again and see delta.
- Add +1 brick, load 3 bricks and see delta.
- ...
- See the regression.
- Use collected data to update or change service's load testing configuration and send it to Atlas.

### Documentation

Documentation has to be reviewed like any other codes!

- Service description - a few lines of what the service does.
- A link to the architecture diagram.
- Runbook - how to run the app, what preparation steps are required.
- FAQ - list well-known issues.
- API endpoints description.
- Labels - links to a product, a team or a component (billing, filing form, etc.).
- Owner(s) - fill manually or automatically.

### General

- Auto scaling - not only by CPU or RAM, but by app-specific metrics.
- Provide security updates - base images, libs and deps.
- Warn about outdated services.
- Consider to remove or aggregate services which used to be high loaded, but now have low traffic.
- Provide a microservice dashboard with common metrics.

## Difficulties of containerized applications and microservices

*by Ivan Kruglov*, [Booking.com](https://www.booking.com)

For the last two years Booking.com has exploited three generations of private clouds including:

- [Apache Mesos](http://mesos.apache.org/) + [Marathon](https://mesosphere.github.io/marathon/) (half a year),
- [OpenShift](https://www.openshift.com/) (a year),
- [Kubernetes](https://kubernetes.io/) (current).

### Marathon

Marathon took off as a hackathon, but the problem is that it's still in that stage.

> Bright Thought 1. Don't underestimate the ability of code to survive.

### OpenShift

OpenShift is shipped with Kubernetes. Booking.com used it along with [Ansible](https://www.ansible.com/) and [Open vSwitch](https://www.openvswitch.org/) because:

- it's super cool;
- network level isolation;
- dynamic firewall holes.

The problem is - such a system is very hard to maintain. It's obvious, but **there is no cloud, it's just someone else's computer**.

> Bright Thought 2. Cool vs. Boring.

Wanted:

- Simple and convenient interface like in [Heroku](https://www.heroku.com/): one button to solve all the problems.

Got:

- Great first experience.
- Confusion: what's going on?
- Hard debug of the problem apps.
- Unhappy experienced users.

> Bright Thought 3. Formulate user expectations from the system and system expections from the users.

Furthermore, it was hard to integrate that system to existed environment.

- DLB (distributed load balancing);
- Service Discovery / Service Mesh;
- Graphite;
- Storage ([MySQL](https://www.mysql.com/), [Cassandra](http://cassandra.apache.org/), [Redis](https://redis.io/), [Kafka](https://kafka.apache.org/), ...);
- SSL certificates / identity key management;
- ...

> Bright Thought 4. Kubernetes is good, but it's integration with current environment is better! *
> 
> \* If you don't start from scratch.

The result of exploitation that system was:

- lots of help tickets;
- users are waiting for magic without understanding the infrastructure;
- loss of trust.

The plan to escape:

- Minimum cool features.
- Infrastructure monitoring.
- Clear expectations from:
    - team,
    - users,
    - integration providers.
- Investment in knowledge:
    - trainings,
    - courses,
    - docs,
    - books.

### Kubernetes

Finally we've switched to Kubernetes, rejected some of the cool features (threw Ansible and Open vSwitch away) and realized proactive monitoring (we inform you, that we have a problem). After realization of the plan we've got:

- stable system;
- ~ 200 unique services + stable growth;
- understanding of what's going on;
- fine-tuned interaction;
- self service;
- education system.

## A way to realize Data Discovery in microservice architecture

*by Nikolay Golov*, [Avito](https://www.avito.ru)

In microservice architecture we use Database-per-service pattern instead of shared database.

But in real systems we have a combination of patterns: Micro + Macro + Monolith + Shared Bases + DB-per-Service + ... .

The first step to split monolith app is to do [domain modelling](https://en.wikipedia.org/wiki/Domain_model) (get an image of how data is organized).

> **Data Discovery** is a way to understand complexity of a continuously evolving system from data point of view and form actionable trends.

[Digital Twin](https://en.wikipedia.org/wiki/Digital_twin) is an adequate way to get an image of the following difficulties.

- What services store what important data?
- How important data transfer between services:
    - who creates it?
    - who copies it?
    - who removes it?
- Who can access what data?

The answer to these questions is a graph called **Persistent Fabric**. The levels are:

- UI points,
- API endpoints,
- services,
- storages,
- teams,
- persons,
- hosts,
- entities.

Each level should be actualized by using different sources.

For example, to grant access to a database, we can query Persistent Fabric like that:

- -[get owner]-> Storage -[r/w access]-> Service -[owned by]-> Team -[leaded by]-> Person.

This graph can help to:
- find a subgraph for isolated testing,
- prioritize monolith splitting,
- loose coupling.

Different levels of the fabric should be filled by different people. For example, DBA fills Storage level, owner fills Service level and so on. Thanks to a single buffer, we can ask Persistent Fabric to compute:

- historical graph (changes),
- current graph (now),
- graph graph (real time API),
- bus data streams graph,
- UI points graph (user paths).

## Delayed data as a way to provide consistency

*by Andrey Litunenko*, [2GIS](http://info.2gis.com/)

In some sophisticated systems there may be the following problems:

- lots of imported data of different types,
- subsystems publish data with different speed,
- there are dependencies between the subsystems,
- some items of data may be uploaded without its subitems,
- backward compatibility is required on any schema changes.

### Approach 1.0

- Import system written on PHP.
- In-house message delivery system.
- Collecting data by cron.
- Stubs for consistency.

For instance, if entity attribute is received before the entity, we'll create a stub for the entity. Later we'll replace the stub with actual model.

Negative results:

- Duplicate key error on simultaneous inserting a stub and an entity (serial run of the commands may be a solution).
- Slowing down on importing 8 mln records.
- Shutting down on importing all the records.

### Approach 2.0

- Delayed data concept - put inconsistent data in separate table and wait until all the deps are received.
- Notifications about outdated delayed data help to:
    - find internal problems,
    - find problems with external sources.
- Apache Kafka:
    - Topics with all the change logs (store for 1 year).
    - Compacted topics with actual state (store forever).
    - Deduplication.
    - Re-read the partitions only for specific consumer.
- [Golang](https://golang.org/).

### Summary

- Draw data dependency.
- Use delayed data to separate consistent and inconsistent data.
- Provide backward compatibility on schema changes.
- Think how to upload all data without a chance to shut down the whole system.
- Run commands asynchronously.

## How search is organized in hh.ru

*by Alexey Bichuk*, [hh.ru](https://hh.ru/)

Original article on [habr.com](https://habr.com/company/hh/blog/413261/).

What search is?

- Find something **relevant**.
- Support **different queries**.
- **Filter** by different parameters.
- **Ranking**.
- **Hide** garbage from search result.
- Do it **fast**.

What are the existing solutions?

- [Lucene](http://lucene.apache.org/) (we use it),
- [Elasticsearch](https://www.elastic.co/),
- [Sphinx](http://sphinxsearch.com/),
- [Solr](http://lucene.apache.org/solr/).

Why Lucene?

- Historical reasons:
    - there was no Elasticsearch on the market,
    - Solr was unstable,
    - Sphinx is not for Java.
- Customizing.
- Good community.

Lucene out-of-box is able to:

- preprocess documents,
- construct inverted index,
- execute different queries,
- segment index,
- use basic ranking formulas,
- use suggests, facets, counters, etc.

General questions.

- How to search by similar words?
    - On indexing put synonym hash link close to original phrase.
    - Extend the query with synonyms.
- How keep all the things in memory?
    - Use sharding by:
        - types (vacancies, resumes, companies, ...),
        - date (active vacancies, archive vacancies, ...),
        - by id (inside the type).
- Load and fault tolerance.
    - 95% queries must be done in less than 200ms.
- What is downtime?
    - 20 HTTP 500 errors per second from frontend.
    - 5% queries from frontend respond in more than 4s.
- How to reduce the load?
    - Put the most popular filters into the cache in form of byte mask (true if document match the filter).
- Ranking:
    - TF-IDF,
    - BM25.
- Personalized ranking:
    - deal with empty queries,
    - tag resumes and vacancies.
- How to deal with timeouts?
    - Retry policy.
    - Speculative timeouts.
- ML in search:
    - user saw vacancy in search,
    - clicked on vacancy,
    - added in favourites,
    - got response,
    - got response with invitation.
- How to withstand the ML load on prod?
    - Use model cascade:
        - 620k vacancies,
        - heuristic filters - 100k,
        - linear filtering model - 30k,
        - XGBoost filtering model - 10k,
        - XGBoost ranking model - ~500 features.
- How to measure quality of search?
    - Session success.
    - Average and absolute number of conversions.
    - Clicks on search results page (and number of page).
- How to organize experiments with search?
    - Rebuild index (if necessary).
    - Use A/B testing - for 2 weeks on 5% users.

## How to deal with techdebt

*by Anton Potapov*, [Ingram Micro Cloud](http://www.ingrammicrocloud.ru/)

What is technical debt?

- Сrutches, hacks, poor technologies, etc.
- Build, check, static analysis, deployment.
- Unit, integration, UI e2e (end-to-end) tests.
- Documentation.

What to do with it?

- Nothing.
- Don't rewrite.
- Rewrite.

Why to invest in techdebt?

- More benefits:
    - more improvements,
    - delivering is quicker,
    - quality is higher, performance or predictability,
    - ability to free up people to work on product features.
- Less costs on:
    - development,
    - support,
    - usage.

> **You have to have clear plan with concrete numbers and dates.**

## Infrastructure development in Badoo

*by Anton Povarov*, [Badoo](https://badoo.com/)

In the beginning of any startup:

- everything is growing,
- business wants more,
- long infrastructure issues appear (sharding, logs, metrics, visualization, open second data center, ...).

It's better to delegate these issues to separate team called **Platform**. Benefits are:

- consistency,
- responsibility,
- spend their 10 000 hours.

Platform:

- Big features machinery.
- C / C++ / Go.

Infrastructure:

- Scripts collection.
- Queues.
- Photos.
- Data migrations between DCs.

Tools:

- Metrics: store and visualize.
- Build.
- Deploy.
- Test.

![](http://www.collaborativeclimate.com/blog/wp-content/uploads/2017/08/Ditch-Annoyance.jpg)

How to choose important?

- Horizontal communications.
- Focus on the result.
- Continuous engineering feedback.

Project creation funnel.

- Sources: collect issues, ideas, projects.
- Involvement: find a customer.
- Business ideas: what issue to solve?
- Ready to dev: engineer project, resources.
- Priorities: what to do now?

Platform issues are specific:

- long term projects,
- the mistake price is really high,
- you cannot just "write codes",
- launch is often difficult.

Who does it?

- Developer:
    - does kick off,
    - forms tech plan,
    - leads the project,
    - is responsible for the result!

It's important to keep feedback cycle:

- clarity,
- management of expectations.

Stages:

- What exactly will have been done?
- When this feature will have been on prod?

Retrospective:

- explain, why deadline shift has been occurred again.

PR:

- Tell about what you have done:
    - projects are long term,
    - changes are always "invisible" for the users by design.
- Collect any feedback.
- Tell, what you are going to do.

When to tell this? Always. What will you get? Confidence that problems are solved. And, as a result, new problems!

### Summary

Platform issues lifecycle:

1. Listen.
2. Involve.
3. Suggest a solution.
4. Have it done.
5. Tell about it.
6. Go to step 1.
