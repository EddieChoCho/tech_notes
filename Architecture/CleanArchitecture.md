# Clean Architecture: A Craftsman's Guide to Software Structure and Design (Robert C. Martin Series)

## I. Introduction

### Ch.1 What is Design and Architecture?
* The low-level details and the high-level structure are all part of the same whole. 
* They form a continuous fabric that defines the shape of the system. 
* You can’t have one without the other; indeed, no clear dividing line separates them. 
* There is simply a continuum of decisions from the highest to the lowest levels.

#### THE GOAL?
* The goal of software architecture is to minimize the human resources required to build and maintain the required system.
* The measure of design quality is simply the measure of the effort required to meet the needs of the customer. 
	* If that effort is low, and stays low throughout the lifetime of the system, the design is good.
	* If that effort grows with each new release, the design is bad. It’s as simple as that.

### Ch.2 A Tale of Two Values
* Every software system provides two different values to the stakeholders: behavior and structure.
* Software developers are responsible for ensuring that both those values remain high.

#### Behavior
* We do this by helping the stakeholders develop a functional specification, or requirements document. 
* Then we write the code that causes the stakeholder’s machines to satisfy those requirements.

#### Architecture
* When the stakeholders change their minds about a feature, that change should be simple and easy to make. 
* The difficulty in making such a change should be proportional only to the scope of the change, and not to the shape of the change.

* The more the architecture prefers one shape over another, the more likely new features will be harder and harder to fit into that structure.
* Therefore architectures should be as shape agnostic are practical.

#### EISENHOWER’S MATRIX
#### FIGHT FOR THE ARCHITECTURE

## II. Starting with the Bricks: Programming Paradigms

### Ch.3 Paradigm Overview
#### Structured programming
* Structured programming imposes discipline on direct transfer of control.

#### Object-orient programming
* Object-oriented programming imposes discipline on indirect transfer of control.

#### Functional programming
* Functional programming imposes discipline upon assignment.

#### Food for Thought
* Each of the paradigms removes capabilities from the programmer. None of them adds new capabilities. 
* Each imposes some kind of extra discipline that is negative in its intent. The paradigms tell us what not to do, more than they tell us what to do.

* Three big concerns of architecture: function, separation of components, and data management.

### Ch.4 Structured Programming

### Ch.5 Object-Oriented Programming

### Ch.6 Functional Programming

## III. Design Principles

### Ch.7 SRP: The Single Responsibility Principle

### Ch.8 OCP: The Open-Closed Principle

### Ch.9 LSP: The Liskov Substitution Principle

### Ch.10 ISP: The Interface Segregation Principle 

## IV. Component Principles

### Ch.12 Components

### Ch.13 Component Cohesion

### Ch.14 Component Coupling

## V. Architecture

### Ch.15 What is Architecture?

### Ch.16 Independence

### Ch.17 Boundaries: Drawing Lines

### Ch.18 Boundary Anatomy

### Ch.19 Policy and Level

### Ch.20 Buiness Rules

### Ch.21 Screaming Architecture

### Ch.22 The Clean Architecture

### Ch.23 Presenters and Humble Objects

### Ch.24 Partial Boundaries

### Ch.25 Layers and Boundaries

### Ch.26 The Main Component

### Ch.27 Services: Great adn Samll

### Ch.28 The Test Boundary

### Ch.29 Clean Embedded Architecture

## VI. Details

### Ch.30 The Database is a Detail

### Ch.31 The Web is a Detial

### Ch.32 Frameworks are Details

### Ch.33 Case Study: Video Sales

### Ch.34 The Missing Chapter

## VII. Appendix










