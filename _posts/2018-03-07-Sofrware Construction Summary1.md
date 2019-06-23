---
layout: post
title: "Software Construction Summary 1"
subtitle: ""
date: 2019-03-14
author: Aether
category: coding
tags: SOFTWARE-CONSTRUCTION
finished: true
---

## Multi-dimensional software views

- By phases: **build-time** and **run-time** views

- By dynamics: **moment** and **period** views

- By levels: **code** and **component** views
	- logically / physically
	- in-memory states / physical environment

### Build-time
- idea—>requirement—>design—>code—>installable / executable package
- moment
	- code
		- Build-time, moment, and code-level view
		- Lexical-oriented source code :e.g.,Lexical-based semi-structured source code
		- Syntax-oriented program structure: e.g., Abstract Syntax Tree (AST)
		- **Semantics-oriented program structure: e.g., Class Diagram(UML)**
	- component
		- Build-time, period, and code-level view
		- file, package, library
- period
	- code
		- code churn
	- component
		- SCI

### Run-time
- High-level concepts
	- Executable programs
	- Libraries
	- Configuration and data files
	- Distributed program
- moment
	- code
		- Snapshot diagram
		- Memory dump
	- component
		- Deployment diagram in URL
		- Dynamic linking
		- Framework/Net
- period
	- code
		- **Sequence diagram in UML**
		- Stack Trace
	- component
		- Event log

### Types of Transformations
- NULL —> Code
	- – Programming / Coding
	- – Review, static analysis/checking
- Code —> Component
	- – Design (Chapter 3 ADT/OOP
	- – Build: compile, static link, package, install, etc
- Build-time —> Run-time
	- – Install / deploy
	- – Debug, unit/integration testing
- Moment —> Period
	- – Refactoring
	- – Version control
	- – Loading, dynamic linking, interpreting,execution

## Quality Objectives

### External quality factors
- **Correctness**
	- Conditional
- **Robustness**
- **Extendibility**
- **Reusability**
- Compatibility
- **Efficiency**
- Portability
- Ease of use
- Functionality
- Timeliness
- Other qualities
	- Verifiability
	- Integrity
	- Repairability
	- Economy

### External quality factors
- LoC
- Complexity
- Easy to understand

## Software Lifecycle and Configuration Management

### SCM
- SCI
	- Software Configuration Item (e.g., file)
- baseline
	- release
- CMDB
	- .git

### Version Control System (VCS)
- Local VCS
- Centralized VCS
- Distributed VCS
- objectgraph
	- Commits: nodes in Object Graph


## Software Development Lifecycle(SDLC)
- Waterfall (Linear)
- Incremental (Linear)
- Prototyping (Iterative)
- V-Model (for verification and validation)
- Sprial (Iterative)
- Agile (Incremental+Iterative)
