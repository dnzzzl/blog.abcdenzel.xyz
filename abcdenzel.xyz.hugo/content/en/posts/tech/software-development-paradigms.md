+++
date = '2025-11-16T13:00:00-04:00'
draft = true
title = 'Software Development Paradigms Explained (Without the Consulting-Speak)'
tags = ['agile', 'scrum', 'kanban', 'devops', 'tdd', 'software-engineering']
categories = ['tech']
+++

Someone asked me to explain the difference between TDD, BDD, and CDD, and then between Agile, Scrum, and Kanban, and then how CI/CD fits into all of it.

Turns out these are actually two different categories of concepts that people mix together constantly. Here's what they actually mean, without the enterprise consulting nonsense.

## Two Different Sets

**Set 1: Testing/Development Practices**

- TDD (Test-Driven Development)
- BDD (Behavior-Driven Development)
- CDD (Contract-Driven Development)

**Set 2: Project Management Frameworks**

- Agile (the philosophy)
- Scrum (the framework)
- Kanban (the system)
- Lean Manufacturing (the origin)
- Just-in-Time Manufacturing (the strategy)

These are related but distinct. The first set is about *how you write code*. The second set is about *how you organize work*.

Let's break them down.

## Set 1: Testing and Development Practices

### Test-Driven Development (TDD)

Write tests before you write code.

The cycle:

1. Write a failing test
2. Write just enough code to make it pass
3. Refactor while keeping tests green
4. Repeat

The point isn't really about testing—it's about forcing yourself to think through what the code should do before you write it. The tests are just the artifact of that thinking.

Does it work? Sometimes. When you know exactly what you're building, TDD is great.
I have honestly never done it, when you're exploring and don't know what the code should do yet, TDD can feel like writing in handcuffs.

### Behavior-Driven Development (BDD)

TDD's more communicative cousin. Same basic idea (write tests first), but the tests are written in a format that non-technical people can read.

Instead of `assert(user.isValid())`, you write:

```gherkin
Given a user with a valid email
When they attempt to log in
Then they should be authenticated
```

The benefit is that product managers, QA, and developers can all agree on what "correct behavior" means before anyone writes code. The downside is maintaining yet another layer of abstraction.

BDD is most useful when miscommunication is your biggest problem. If everyone already agrees on requirements, it's just extra overhead.

### Contract-Driven Development (CDD)

Define the API contract first, then implement it.

If you're building a microservice, you write the OpenAPI spec (or equivalent) before writing any code. The contract specifies: "This endpoint takes X, returns Y, fails with Z."

Then you implement the service to match the contract, and you test against that contract.

This is useful when multiple teams need to integrate before anyone's finished building. Frontend team can develop against the contract while backend team implements it.

The downside is that contracts are hard to change once people depend on them, so you better get it right the first time.

### What They Have in Common

All three are about *defining expectations before implementation*. They differ in what those expectations look like:

- TDD: Code-level tests
- BDD: Behavior specifications
- CDD: API contracts

## Set 2: Project Management and Process

### Agile (The Philosophy)

Not a specific process—it's a philosophy about how software development should work.

The original idea (from the [Agile Manifesto](https://agilemanifesto.org/)):

- Individuals and interactions over processes and tools
- Working software over comprehensive documentation
- Customer collaboration over contract negotiation
- Responding to change over following a plan

That's it. Everything else is interpretation.

Agile says: Build in small increments, get feedback, adapt. Don't spend six months building the wrong thing.

It doesn't tell you *how* to do that. That's where frameworks like Scrum and Kanban come in.

### Scrum (The Framework)

Scrum is a specific implementation of Agile principles.

**Core components:**

- **Sprints**: Fixed time-boxes (usually 2 weeks) where you build a set of features
- **Roles**: Product Owner (decides what to build), Scrum Master (removes obstacles), Development Team (builds it)
- **Ceremonies**:
  - Sprint Planning (decide what to build this sprint)
  - Daily Standup (quick sync on progress)
  - Sprint Review (demo what you built)
  - Sprint Retrospective (figure out how to improve)
- **Artifacts**: Product Backlog (all the things), Sprint Backlog (things for this sprint), Increment (working software)

Scrum is prescriptive. It says "do these things, in this way, at these intervals."

**When it works:** Teams that need structure, predictable delivery cycles, and stakeholder involvement.

**When it doesn't:** Teams that hate meetings, have constantly changing priorities, or need continuous deployment.

### Kanban (The System)

Kanban is about visualizing work and limiting work-in-progress.

**Core principles:**

- Visualize the workflow (usually as a board with columns: To Do, In Progress, Done)
- Limit WIP (Work in Progress)—only X items can be "In Progress" at once
- Manage flow—optimize for smooth, continuous delivery
- Make process policies explicit
- Improve collaboratively

Kanban is less prescriptive than Scrum. No sprints, no required meetings, no fixed roles. Just: "Here's the work, here's where it is, don't start new things until you finish current things."

**When it works:** Teams doing continuous deployment, support/maintenance work, or anything with unpredictable arrival of tasks.

**When it doesn't:** When stakeholders want predictable delivery dates, or when the team needs more structure.

### Lean Manufacturing & Just-in-Time

These aren't software concepts originally—they're from manufacturing (Toyota, specifically).

**Lean**: Eliminate waste. Anything that doesn't directly create customer value is waste.

**Just-in-Time (JIT)**: Produce exactly what's needed, exactly when it's needed. No inventory sitting around.

Software borrowed these ideas:

- Don't build features no one will use (Lean)
- Don't deploy ahead of when users need it (JIT)
- Minimize work-in-progress (Lean)
- Deliver continuously instead of in batches (JIT)

Kanban is basically JIT applied to software.

## Where CI/CD Fits In

**Continuous Integration (CI)**: Developers integrate code into a shared repository frequently (multiple times a day). Each integration triggers automated builds and tests.

**Continuous Deployment (CD)**: Code that passes tests automatically deploys to production.

CI/CD is a *technical practice* that enables Agile methodologies.

You can't do Scrum well without CI—you need working software at the end of each sprint, which means integration can't be a nightmare.

You definitely can't do Kanban well without CD—continuous flow requires continuous deployment.

CI/CD automates the feedback loop that Agile depends on. It's not a methodology itself; it's the infrastructure that makes modern methodologies possible.

## DevOps: The Cultural Wrapper

DevOps isn't a role or a tool—it's a cultural shift.

Traditional model:

- Developers write code
- Operations team deploys and maintains it
- These teams have conflicting incentives (Devs want to ship fast, Ops wants stability)

DevOps model:

- Same team handles development and operations
- Shared incentives (ship fast *and* keep it stable)
- Heavy automation (because you can't do both without it)

DevOps relies on CI/CD pipelines, monitoring, infrastructure as code, and collaborative culture.

It sits above all the other frameworks. You can do DevOps with Scrum, or with Kanban, or with whatever process you want. DevOps is about breaking down the wall between development and operations.

## How They Relate

```
Agile (Philosophy)
├── Scrum (Framework for iterative delivery)
│   └── Requires CI for working software each sprint
├── Kanban (Framework for continuous delivery)
│   └── Requires CD for continuous flow
└── Lean (Minimize waste, maximize value)
    └── Inspires Kanban and DevOps practices

DevOps (Culture)
└── Enabled by CI/CD (Technical practices)
    ├── Continuous Integration
    └── Continuous Deployment

Testing Practices (Code-level)
├── TDD (Tests first)
├── BDD (Behavior specs first)
└── CDD (Contracts first)
```

They're complementary. You might use Scrum for project management, TDD for development, and CI/CD to automate testing and deployment, all under a DevOps culture.

Or you might use Kanban with BDD and continuous deployment.

Or any other combination that fits your context.

## What Actually Matters

Here's the thing: none of these are magic bullets.

Scrum won't save a dysfunctional team. Kanban won't work if you have 50 items in progress. TDD won't help if you're testing the wrong things. DevOps won't fix bad code.

They're all tools. Use the ones that solve your actual problems:

- **Too much work in progress?** Try Kanban with WIP limits.
- **Stakeholders want predictable delivery?** Try Scrum with sprints.
- **Integration is painful?** Implement CI.
- **Deployment is scary?** Add automated tests and CD.
- **Dev and Ops hate each other?** Try DevOps culture (and therapy).
- **Unsure what to build?** Use BDD to align on requirements.

Don't adopt a framework because it's trendy. Adopt it because it solves a problem you actually have.

## Further Reading

If you want the source material instead of my opinions:

- [Agile Manifesto](https://agilemanifesto.org/) (short, worth reading)
- [Scrum Guide](https://scrumguides.org/) (official Scrum definition)
- [Kanban Guide](https://kanbanguides.org/) (official Kanban principles)
- *The Phoenix Project* by Gene Kim (DevOps as a novel—surprisingly good)
- *Continuous Delivery* by Jez Humble (the CI/CD bible)

---

Now you can explain the difference at your next standup meeting. You're welcome.
