# CS Fundamentals — Interview Preparation

Structured notes for software engineering interviews covering **Computer Networks, Operating Systems, DBMS, and Object-Oriented Programming**.

The objective is to explain concepts, reason through examples, compare alternatives, and handle follow-up questions—not just memorize definitions. These notes target foundational undergraduate and entry-level software engineering interviews, with selected deeper follow-ups.

## Subject map

| Subject | Reading order | Learning outcome |
|---|---|---|
| [Computer Networks](01_computer_network/README.md) | Layers → addressing → transport → web → practice | Explain how a browser communicates with a server |
| [Operating Systems](02_operating_system/README.md) | Processes → synchronization → memory → storage → practice | Explain how the OS shares CPU, memory, and devices |
| [DBMS](03_dbms/README.md) | Modeling → SQL → transactions → indexing → practice | Model, query, protect, and retrieve data efficiently |
| [OOP](04_oops/README.md) | Foundations → polymorphism → design → patterns → practice | Design objects with clear responsibilities and replaceable behavior |

## How to study

1. Read a numbered chapter and explain its central idea aloud without looking.
2. Work through the example by hand; change one input and predict the result.
3. Answer the interview questions before reading the answers.
4. Complete each subject's `05_interview_practice.md` file.
5. Use the [revision plan](REVISION_PLAN.md) to revisit weak areas.

Build answers as **definition → mechanism → example → trade-off**. Start with a 30–60 second explanation, then expand when asked.

## Conventions

- **Core** topics deserve priority; **follow-up** sections add depth.
- SQL examples use an employee/department schema; dialect-specific behavior is identified.
- OOP examples use Java unless stated otherwise. Code snippets illustrate concepts rather than complete applications.
- OS and network calculations state their simplifying assumptions.
- Implementation-dependent behavior is called out instead of presented as universal.

The subjects can be studied independently. A useful first-pass order is **OOP → DBMS → OS → CN**. Then connect them: a web request reaches a process, runs application objects, reads database pages, and returns bytes over the network.
