title: Built on Ruby: Cloud Foundry, the Open Source PaaS Story

!SLIDE

## Agenda Today

<% left do %>

### 1- Cloud Foundry at Launch

Goals and initial architecture.

### 2- Cloud Foundry Today

Lessons learned.

### 3- Cloud Foundry Future

Help us build it.
<% end %>

<% right do %>
![main](/img/LSRC/china.png)
<% end %>

!SLIDE

<% left do %>
<%= include "../shared/us/monica.md" %>
<% end %>

<% right do %>
![main](/img/head2.jpg)
<% end %>


!SLIDE vcenterH2

## This is the story of a PaaS

!SLIDE vcenterH2

## But, first

!SLIDE vcenter

## What is a PaaS ?

### PaaS stands for `Platform As a Service`

!SLIDE

## What does a PaaS do ?

### A PaaS is a system that allows you to:


### 1 - Easily `deploy` web `Applications`


### 2 - `Bind` the applications to `Services`


### 3 - `Scale` the `Applications`


!SLIDE vcenterH2

## Part One

!SLIDE vcenterH2

## April 2011

!SLIDE vcenterH2

## Cloud Foundry Launches

!SLIDE

## First Open Platform as a Service(PaaS)

![main](/img/Panel.png)

### Built by VMware

!SLIDE vcenter

## Cloud Foundry's Goals

### Raise the unit of currency to be the application and its associated services, not the infrastructure

!SLIDE vcenterH2

## How ?

!SLIDE vcenterH2

## Simple Toolchain

!SLIDE vcenter

## VMC

### VMware Cloud CLI

!SLIDE

<%= include "../shared/vmc.md" %>

!SLIDE

<%= include "../shared/core_vmc.md" %>


!SLIDE

<%= include "../shared/update_apps_vmc.md" %>

!SLIDE

<%= include "../shared/vmc_services.md" %>

!SLIDE vcenter

## Cloud Foundry's Goals

### Favor Choice and Openness

!SLIDE

## OPEN as in:

<% left do %>
### Multi Languages

### Multi Frameworks

### Multi Services

### Multi Clouds

    vmc target api.yourcloud.com

<% end %>

<% right do %>
![main](/img/LSRC/start-triangle.png)
<% end %>

!SLIDE vcenterH2

## Most importantly...

!SLIDE vcenter

## Extensibility

### Easy to add your choice language, framework or service

!SLIDE

## Open Source

- Cloud Foundry was Open Sourced under Apache License
- On Github [https://github.com/cloudfoundry/vcap](https://github.com/cloudfoundry/vcap)

![main](/img/open_source_1.png)

!SLIDE vcenterH2

## More than 90% of the Cloud Foundry code is written in `Ruby`

!SLIDE vcenterH2

## Let's Explore

!SLIDE

## How was Cloud Foundry Built?

<% left do %>
### Kernel (CloudFoundry OSS)

Core PaaS System

### Kernel and Orchestrator Shells

Layered on top of IaaS

### Orchestrator

IaaS creation, management and orchestration
￼
<% end %>

<% right do %>
￼￼
![main](/img/LSRC/overview-high-level.png)

<% end %>

!SLIDE

<%left do%>

## Basic Premises

### Fail `Fast`

### Self-Healing

### Horizontally Scalable Components

### Distributed State

### No Single Point of Failure

### Should be as simple as possible

<%end%>

<%right do%>

## Basic Patterns

### Event-Driven

### Asynchronous

### Non-blocking

### Independent, Idempotent

### Message Passing

### Eventually Consistent

<%end%>

!SLIDE

<% left do %>

## Basic Design

### All components loosely coupled

### Few “Classes”, many “Instances”

### Messaging as foundation

### Addressing and Component Discovery

- Command and Control JSON payloads
- HTTP or File/Blob for data transport
<% end %>


<%right do%>
## Objectives

### All dynamically discoverable

### Launch and scale in any order

### Can come and go as needed

### Monitor via `HTTP` and `JSON`

### Location independent

<%end%>

!SLIDE

<% left do %>

## Kernel Components

### - Router

### - Cloud Controller

### - DEA

### - Health Manager

### - Service Provisioning Agent

### - Messaging System

<% end %>

<% right do %>

## Logical View
￼￼￼
![main](/img/LSRC/logical-view.png)

<% end %>

!SLIDE

## Architecture

![main](/img/LSRC/arch.png)

!SLIDE

## CloudController “The King”

![main](/img/LSRC/cc.png)

### Handles all state transitions

### Deals with users, apps, and services

### Packages and Stages applications

### Binds Services to Applications

### Presents external REST API

!SLIDE

## DEA “Droplet Execution Agent”

### Responsible for running all applications Monitors all applications

CPU, Mem, IO, Threads, Disk, FDs, etc

### All apps look same to DEA

`start` and `stop`

### Express ability and desire to run an application

### runtimes, options, cluster avoidance, memory/cpu

### Alerts on any change in state of applications

### Single or Multi-Tenant

- Hypervisor, Unix File and User, Linux Containers*
- Provides secure/constrained OS runtime

!SLIDE

## HealthManager “Court Jester”

<% left do %>

### Monitors the state of the world

### Initial value with realtime delta updates to “intended” vs “real”

### Determines drift

### Complains to the CloudControllers when something is not correct

### No power to change state itself

<% end %>

<% right do %>
![main](/img/LSRC/healthmanager.png)
<% end %>

!SLIDE

## Messaging “The Nervous System”

<% left do %>

### Addressing and Discovery
  - No static IPs or DNS lookups req’d
  - Just Layer 4

### Command and Control

### Central communication system

### Dial tone, fire and forget

### Protects *itself* at all costs

### Idempotent semantics

<% end %>
<% right do %>
![main](/img/LSRC/messaging.png)
<% end %>

!SLIDE

## Router “Traffic Cop”

<% left do %>

### Handles all HTTP traffic

### Maintains distributed routing state

### Routes URLs to applications

### Distributes load among instances

### Realtime distributed updates to routing tables from DEAs

<% end %>
<% right do %>
![main](/img/LSRC/router.png)
<% end %>

!SLIDE vcenterH2

## How does it all Work?


!SLIDE

## Pushing an App

![main](/img/LSRC/pushing-app.png)

- Client pushes meta-data to CC
- Client optionally pushes resource signatures (diff analysis, sys wide)
- Client pushes app resources to CC CC puts app together
- CC stages app asynchronously
- CC binds and stages services Droplet ready

!SLIDE

## Running an App

    vmc start <app_name>

- CC asks DEAs for “help”
- First DEA back wins! Simple
- CC sends start request to selected DEA
- DEA pushes the “green” button
- DEA waits and monitors pid and ephemeral port for app to bind
- When app is healthy, sends “register” message
- Register message is seen by HM and Routers
- Routers bind URL to host:port

### DEAs answer?

- DEAs first determine YES or NO
  - correct runtime, options, memory, etc
- Then calculate a Delay Taint
  - SHA hash of application
  - memory
  - cpu
- Taint allows balancing and selection

!SLIDE

## Scale up & down?

    vmc instances <app_name> +5

    vmc instances <app_name> -2

    vmc stats <app_name>

- Same `exact steps` as running the app the first time
- SHA1 taint helps avoid clustering
  - memory/cpu taint helps distribute as evenly as possible
- Nothing pre-computed
- Nothing assumed

!SLIDE

## Crashes?

### If your app stops and we did not tell it to, that is a crash

### Crashed apps are immediately detected by DEA and messaged

### Routers disconnect route instantly HM will signal CC something is wrong

### CC will issue run sequence again

!SLIDE

<% left do %>

## Access to my App?

### All routers understand where all instances of your application are running

### Will randomly pick backend, not semantically aware.

### Will remove routes that are stale or unhealthy

### Session stickiness and replication available, but best to avoid if possible

<% end %>

<%right do %>

## Serving Requests

![main](/img/LSRC/http-request.png)

<% end %>

!SLIDE

## What about Services?

![main](/img/LSRC/service_provisioning.png)

!SLIDE

## Services

### Service Advertisement

### Service Provisioning

### Gateway fronts multi-backends Service

### Nodes scale independent App and service talk directly

### API to register into system

### Closure for additional value

!SLIDE

## Provisioning

PIC

!SLIDE

## Access

PIC

!SLIDE

<%= include "../shared/cfdotcom.md" %>

!SLIDE vcenter

## April 2012

!SLIDE

![main](/img/LSRC/adoption.png)

!SLIDE

<%= include "../shared/runtimes.md" %>

!SLIDE

<%= include "../shared/runtimes2.md" %>

!SLIDE vcenterH2


### Visible Progress

!SLIDE

## Services

### Caldecott

– Tunnel into your services
- Explore with standard client tools

### New Services on CloudFoundry.com

- PostgreSQL
- RabbitMQ

### New Services on OSS Mater Branch

- Elastic Search
- CouchDB
- Memcached

!SLIDE

## Frameworks, Runtimes, and Tools

- Node.js, Java and Ruby Auto-Reconfiguration
- Scala, node.JS 0.6.*, Erlang, JRuby, PHP, Python, .NET, Spring 3.1, Grails 2.0, Play
- Multi-Node Chef based deployment tools
- Maven Plugin, Eclipse Integration
- VMC manifests, Java Debugging, Rails Console


!SLIDE vcenterH2

## Lessons Learned Along the Way

!SLIDE vcenterH2

## Orchestration

!SLIDE

## CF 2010

- 15 distinct components
- manual updates
- hard to deploy end-to-end
- growing number of components and developers

!SLIDE

## Motivation

- Frequent Deployments
- Minimal Disruption
- Repeatable and Predictable
- Easy to use
- Single Tool
- Dev, Staging, and Production
- Infrastructure Agnostic

!SLIDE vcenter

## BOSH

### An open source tool chain for release engineering, deployment and lifecycle management of large scale distributed services.


!SLIDE

## Logical concepts

### Packages

- just the bits (source and binary objects)

### Jobs

- configuration templates
- control/init scripts
- supervision and monitoring

### Releases

- fully self-contained
- shared
- git repository
- environment and infrastructure agnostic

!SLIDE vcenterH2

## What does BOSH do ?

!SLIDE

## Functionality

### Initial Deployment

### Scaling Up and Down

### Component Updates

### OS Updates

!SLIDE

## Today

### CloudFoundry.com

- 40 distinct components
- many hundreds of VMs
- thousands of deployments

### BOSH

- running in production for 18+ months - used daily by 50+ developers
- CPIs: vSphere, AWS, OpenStack
- self-hosted
- open sourced on 4/11/2012

!SLIDE

## Collaboration

http://www.slideshare.net/marklucovsky/cloud-foundry-anniversary-technical-slides/7

http://blog.cloudfoundry.org/2012/06/20/heads-up-on-some-new-cloud-controller-features/

!SLIDE vcenterH2

## Secure extensibility

!SLIDE vcenterH2

## UAA enters the room

!SLIDE vcenterH2

## Thanks to you &

### Mark Lucovsky

### Derek Collison

### Vadim Spivak



!SLIDE vcenterH2

## [github.com/cloudfoundry](http://github.com/cloudfoundry)

￼