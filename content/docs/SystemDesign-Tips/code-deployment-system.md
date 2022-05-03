---
title: "Design A Code-Deployment System"
bookCollapseSection: true
weight: 10
---

### 1. Gathering System Requirements

As with any systems design interview question, the first thing that we want to do is to gather system requirements; we need to figure out what system we're building exactly.

From the answers we were given to our clarifying questions (see Prompt Box), we're building a system that involves repeatedly (in the order of thousands of times per day) building and deploying code to hundreds of thousands of machines spread out across **5-10 regions around** the world.

Building code will involve grabbing snapshots of source code using commit SHA identifiers; beyond that, we can assume that the actual implementation details of the building action are taken care of. In other words, we don't need to worry about how we would build JavaScript code or C++ code; we just need to design the system that enables the repeated building of code.

Building code will take up to **15 minutes**, it'll result in a binary file of **up to 10GB**, and we want to have the entire deployment process (building and deploying code to our target machines) take at most **30 minutes.**

Each build will need a clear end-state (SUCCESS or FAILURE), and though we care about availability (2 to 3 nines), we don't need to optimize too much on this dimension.

### 2. Coming Up With A Plan

### 3. Build System -- General Overview

### 4. Build System -- Job Queue
### 5. Build System -- SQL Job Queue
### 6. Build System -- Concurrency
### 7. Build System -- Lost Jobs
### 8. Build System -- Scale Estimation
### 9. Build System -- Storage
### 10. Deployment System -- General Overview
### 11. Deployment System -- Replication-Status Service
### 12. Deployment System -- Blob Distribution
### 13. Deployment System -- Trigger
### 14. System Diagram
Final Systems Architecture

![img|320x271](https://prasenjitmanna.com/tech-book/diagrams/code-deployment-system-diagram.svg)
