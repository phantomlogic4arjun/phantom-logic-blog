---
layout: post
title: "Getting Started with Agentic Systems"
date: 2026-11-23
categories:
  - Engineering
  - Agentic Systems
  - Development
tags:
  - Phantom Logic
  - Agentic Systems
  - Autonomous Agents
  - Python
  - PostgreSQL
  - Ollama
  - OpenRouter
  - OpenCode
  - Multi-Agent Systems
---

## Humans Guide. Agents Execute.

This is an experiment in building agentic systems and exploring how multiple autonomous agents can work together under human guidance.

I'm a beginner in this space, learning as I build, experimenting with different approaches, making mistakes, fixing things, and gradually putting the pieces together.

This is where I document that journey: what I'm building, why I'm making certain decisions, what works, and what doesn't.

## What am I trying to build?

Most of us are familiar with using assistants to answer questions, generate content, or help with individual tasks.

But what happens when we take things a step further?

What if, instead of relying on one assistant to do everything, we create a system of specialized agents, each with its own responsibilities?

For example:

- A Product Manager agent understands requests and defines requirements.
- A Developer agent works on implementation.
- A Reviewer examines the implementation.
- A QA agent tests the result.
- An Orchestrator coordinates the different agents and manages the overall workflow.

Each agent has a specific role rather than trying to do everything.

The goal is to have these agents collaborate, exchange information, hand work over to one another, and collectively complete tasks.

But there is an important part of this idea: **human involvement never disappears.**

Humans provide direction, make important decisions, grant approvals where required, and retain oversight of the work. The agents handle execution within the boundaries established for them.

I'm interested in exploring how far this approach can go while keeping the system understandable, accountable, and manageable.

## How Phantom Logic is taking shape

I'm approaching this as a collection of connected modules rather than trying to build everything at once.

Here are the main components I've been working on.

### 1. Agent Architecture and Orchestration

The first challenge is defining the different roles within the system.

Each agent needs a clear responsibility, a defined scope, and instructions explaining how it should perform its work.

Some of the roles I'm exploring include:

- **Orchestrator:** Coordinates requests, delegates work, and manages interactions between agents. Consider this the Chief Doer
- **Product Manager:** Converts ideas into requirements and work items.
- **Developer:** Handles implementation tasks.
- **Reviewer:** Examines proposed changes and checks their quality. Mostly oversees the work of the Developer.
- **QA:** Tests completed work and identifies issues.

These aren't simply different names for assistants. The intention is to give each role a distinct purpose and establish rules for how they interact.

The Orchestrator is particularly important because it acts as the coordination point. It needs to understand the request, determine which agent should handle it, receive the results, and decide what should happen next.

I'm also working on defining the boundaries between agents so that responsibilities don't overlap unnecessarily.

### 2. Work Item Management

One of the first things I realized was that agents need a common way to understand and manage the work they're expected to perform.

A collection of agents without a structured way to define tasks, track progress, and establish responsibilities would quickly become difficult to manage.

I approached this in two parts.

**GitHub Issues and Projects**

I'm using GitHub's work management capabilities as part of the setup.

This has involved more than simply creating issues. I've been configuring the structure of the work management system, taking some inspiration from the work item types and hierarchy in Azure DevOps, which I'm more accustomed to.

- Issue types to distinguish different kinds of work.
- Custom fields to capture additional information.
- Project views to organize and visualize work.
- Workflows to manage how work progresses.
- Relationships and hierarchy between related work items.

The intention is to establish a consistent way of defining and organizing work before agents begin processing it.

**Phantom Logic Work Item Management**

Alongside GitHub's capabilities, I've been developing our own Work Item Management module.

This is an application-side implementation with its own models, services, business rules, relationships, state transitions, and database persistence.

It provides a defined interface through which agents can create, retrieve, and manage work items.

I didn't want the agents to depend entirely on an external work management interface. Phantom Logic needs its own understanding of work items and the rules governing how they are handled.

These are complementary parts of the design: GitHub provides work organization and tracking capabilities, while Phantom Logic implements the application-side functionality required for agent workflows.

### 3. Audit and Governance

This is one of the areas I consider particularly important.

Giving agents the ability to perform actions is only one part of the challenge. We also need to know what they did, why they did it, and what happened as a result.

I've therefore been developing a central audit mechanism.

Important actions are reported to the Orchestrator, which is responsible for submitting them through the existing Audit Writer.

The audit records capture information such as:

- Who requested the action?
- Which agent performed it?
- What action was requested?
- Why was it performed?
- Which resources were affected?
- Was approval required?
- What was the outcome?
- How was the result verified?

One architectural decision I've made is that individual agents and Work Item Management should not write directly to the central audit log.

The Orchestrator remains the single point through which audit records are submitted.

This is intended to make accountability consistent across the system.

I also want human approval to remain part of the process wherever an action requires it. An agent should not treat its ability to perform an action as permission to do so.

### 4. Database and Data Management

As the system grows, it needs a reliable way to store and retrieve information.

I'm using **PostgreSQL** as the relational database, with **Python and SQLAlchemy** handling database interaction.

**Alembic** is used for database schema migrations. This allows changes to the database structure to be tracked and managed rather than making changes manually without a clear history.

The database layer provides the foundation for storing work items and their relationships.

I'm also working toward keeping the different application layers separate so that business logic, database operations, and agent-facing interfaces don't become unnecessarily dependent on one another.

### 5. Model Infrastructure: Ollama and OpenRouter

An important part of this experiment is understanding which models to use, where to run them, and how different agents can benefit from different models.

I don't want Phantom Logic to depend entirely on cloud-hosted models and paid API services. My longer-term intention is to make the system local-first wherever practical, while retaining the flexibility to use cloud models when a task requires them.

**Ollama and local models**

I'm using **Ollama** to experiment with locally hosted models.

Ollama makes it possible to download and run models directly on my machine and expose them through a local API. This allows applications and agent environments to communicate with locally running models without having to send every request to an external provider.

My current local model is **Qwen3:8B**. I must admit that it struggles quite often, and tends to hit its context limits, especially when dealing with more involved tasks.

I want to explore using local models for routine tasks such as documentation, planning, information processing, and other activities that don't necessarily require a more capable cloud model.

Some of the advantages I'm interested in are:

- Running models locally on my own hardware.
- Greater control over where data is processed.
- Reduced dependence on external model providers.
- Experimenting with different models without paying for every API request.
- Having a local model infrastructure that can eventually support multiple agents.

Of course, local models have their own limitations. Available memory, processing power, response speed, and model capability all influence what can realistically run on a particular machine.

**OpenRouter and cloud models**

For cloud model access, I'm using **OpenRouter**.

One of the useful things about OpenRouter is that it provides access to models from different providers through a common API. This makes it easier to experiment with different models without having to integrate each provider separately.

The cloud model I've successfully used for orchestration is **GPT-OSS 20B**, accessed through OpenRouter.

I've been using it to experiment with the Orchestrator and its ability to delegate work to specialist agents.

This has been an important part of testing the multi-agent setup because orchestration involves more than generating a response. The model needs to understand the task, decide which agent should handle it, and coordinate the results.

**The intended direction**

The longer-term idea is to have a hybrid model infrastructure:

- Local models through Ollama for tasks that can be handled locally.
- Cloud models through OpenRouter when additional capability is needed.
- The flexibility to select different models for different agents and responsibilities.

I haven't finalized the model allocation strategy yet. I'm still experimenting with model selection, performance, and how different agents should use different models.

I see this as an important architectural decision because the choice of model can affect the cost, speed, reliability, and overall capabilities of the system.

### 6. OpenCode Integration

Another important part of the project is connecting the agent environment to the Python application.

I'm using **OpenCode** to run and coordinate the agents.

OpenCode provides the environment in which agents can receive instructions, use available tools, interact with the codebase, and delegate work.

To connect it with Phantom Logic's backend, I've been developing a plugin using **TypeScript and Bun**.

The plugin acts as a bridge between the agent environment and the Python application.

One issue I encountered was that the plugin was attempting to use the system Python interpreter instead of the Python interpreter inside the project's virtual environment. As a result, it couldn't locate the Phantom Logic modules.

After correcting the interpreter path and restarting OpenCode, the integration worked.

It was a relatively small issue, but a useful reminder that even when the architecture looks right, the actual integration details matter.

### 7. Repository and Engineering Practices

I've been spending time organizing the project itself. Python isn't my primary language, so I'm still getting familiar with its conventions and best practices. At the same time, I'm learning how agentic systems are structured and how their different components fit together. It's been a learning experience on both the technology and architectural sides.

As the number of modules, agents, instructions, and supporting components increases, keeping everything structured becomes important.

The repository contains the application code, agent instructions, company-level rules, database migrations, tests, and supporting documentation. I am still not sure if this is the way to go.

The intention is to make Phantom Logic easier to understand, maintain, and extend as it evolves.

## The technology stack

These are the main technologies I'm currently using and exploring.

| Technology | Purpose |
|---|---|
| Python | Core application and backend logic |
| PostgreSQL | Relational database |
| SQLAlchemy | Database interaction and ORM |
| Alembic | Database schema migrations |
| TypeScript | OpenCode plugin development |
| Bun | Plugin runtime and execution |
| OpenCode | Agent execution and coordination |
| Ollama | Running and managing local models |
| Qwen3:8B | Current local model |
| OpenRouter | Access to cloud-hosted models |
| GPT-OSS 20B | Cloud model used for orchestration |
| Git and GitHub | Source control, project hosting, Issues and Projects |
| JSON | Structured communication between components |
| Markdown | Agent instructions, rules, and documentation |
| Automated testing tools | Testing application components and functionality |

The stack may evolve as the project grows. I'm more interested in getting the architecture and responsibilities right than committing to a particular technology for everything.

## A small but important milestone

One of the first things I wanted to verify was whether an agent could create a work item and whether that action would be recorded in the central audit log.

It sounds straightforward, but it involves several components working together.

The Product Manager agent needed to:

1. Receive the request.
2. Use the Work Item Management interface.
3. Create a Feature.
4. Return the result and report the action to the Orchestrator.
5. Allow the Orchestrator to submit the audit record.
6. Verify that the record was actually persisted.

I encountered an integration issue along the way, which turned out to be related to the Python interpreter used by the OpenCode plugin.

After fixing it, I successfully created a test Feature through the Product Manager agent.

The operation returned a Work Item ID, identified the Product Manager as the actor, and reported a successful creation event.

I then inspected the central audit log and confirmed that the corresponding audit record had been written.

This was a useful milestone because it demonstrated that the agent environment, plugin, Python backend, Work Item Management, Orchestrator, and Audit Writer could work together for this operation.

It is only one part of the larger system. The complete development workflow, including implementation, independent review, and QA, is still ahead.

But having one integration path working end to end is encouraging.

## What's next?

There is a long way to go. I don't want to rush into building more agents before the existing foundations are reliable.

Here is how I see the next stages.

### Phase 1: Complete Work Item Management

The immediate focus is to verify that agents can retrieve, update, assign, and transition work items through the approved interfaces.

Every significant operation should be reported and audited appropriately.

I also want to make sure that the Work Item definitions, application-side implementation, and agent instructions remain consistent.

### Phase 2: Establish the Complete Development Workflow

I want to move beyond creating features and establish a complete workflow:

- Product Manager defines the requirements.
- Developer receives and implements the work.
- Reviewer examines the implementation.
- QA validates the result.
- Orchestrator coordinates the process and records important outcomes.

The workflow should also handle failures, rework, and situations where human intervention is necessary.

### Phase 3: Introduce Independent Review and Quality Assurance

I don't want the same agent that produces work to be the only one deciding whether that work is correct.

Independent review and testing should be part of the process.

The goal is to make quality checks a normal part of development rather than an afterthought.

### Phase 4: Build Product Management Infrastructure

Eventually, I want Phantom Logic to manage multiple products and projects.

That will require a way to maintain product information, requirements, development plans, versions, and release history.

Each product should have its own development repository while Phantom Logic provides the coordination and management infrastructure.

### Phase 5: Expand Local Model Usage

I want to experiment with running more of the system locally.

This will involve testing different models, understanding their strengths and limitations, and deciding which responsibilities can be handled reliably by local models.

The longer-term goal is to reduce unnecessary dependence on cloud services without compromising the quality of the work.

Cloud models will still have a place when their capabilities justify using them.

### Phase 6: Workflow Automation

Once the individual components are reliable, I want to explore more complex workflows.

Agents should be able to identify the next appropriate action, coordinate with other agents, and continue working within their defined responsibilities.

Human approval and intervention will remain important boundaries.

### Phase 7: Release and Operations

The longer-term goal is to explore how agents can assist with the entire product lifecycle, from initial ideas through development, testing, release, and ongoing maintenance.

This is still a future direction, not something I've implemented yet.

## Why I'm documenting all this

I'm not writing this as someone who has already figured everything out.

I'm learning about agentic systems by actually trying to build one.

Some things will work. Some won't. There will be architectural decisions that need to be revisited, integrations that break, and ideas that turn out to be more complicated than expected.

I think documenting those experiences is valuable.

I also want to be able to look back at where the project started, understand why certain decisions were made, and see how the architecture evolves over time.

If you're also exploring autonomous agents, multi-agent systems, or similar ideas, I hope some of these experiences will be useful to you.

For now, Phantom Logic is a work in progress.

There is a lot left to learn and build.

Whatever these systems become in the future, I believe human judgment, direction, and responsibility must remain at the centre. The goal isn't to replace the human element, but to build systems where people and agents work together.
**Humans Guide. Agents Execute.**
