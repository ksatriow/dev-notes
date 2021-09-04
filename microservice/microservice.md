Microservices - Cheat sheet
=======

Based on [Microservices presentation](https://www.youtube.com/watch?v=wgdBVIX9ifA) by Martin Fowler recorded at GOTO Berlin 2014.

# Common caracteristics

1. Components (= independently upgradable, replaçable) communicating through services (instead of libs for monoliths)
2. Organized around business capabilities
3. Smart endpoints and dumb pipes
4. Decentralized data management
5. Requires infrastructure automation. CI/CD, Monitoring tools
6. Should be designed for failure (What happens if a specific microservice fails)
7. Evolutive design

# MicroService vs SOA

* SOA, different definitions for different people. For some of them, this means ESB. But ESB are not compatible with third Microservices common caracteristic
* MicroServices ⊂ SOA. Microservices architecture is a subset of SOA

# Main advantages

* Different technologies possible per components
* Platform / Technologies best suited for a specific Microservice
* Component internal evolution minimize impacts compared to a functionnality or module evolution within a Monolith
* Multiple Microservices deployed on different machines compared to a Monolith cluster easing horizontal scaling. (If Microservice A is more loaded than B, A can be scaled horizontally but not B)
* Easier Feature teams organization (Catalog, order management, ...) compared to teams organized by technologies (DBA, .Net Devs, ...)
* No unique monolithic database (with hard model refactoring, deadlocks between unrelated functionalities)

# Monolith vs MicroService

## Monolith

* Simplicity
* Consistence
* Inter-module refactoring

## Microservice

* Partial deployment
* Avaibility (If A Microservice fails, users still can use a fonctionnality managed by B, with a design for failure)
* Preserve modularity
* Platform / Technology best suited

# Prerequisites

* Rapid provisioning (rapid environment set-up)
* Basic monitoring
* Rapid applications deployment
* Devops culture
