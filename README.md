# Building an AI-Powered Resume-to-Job Matching Platform with Microservices

## Introduction

Finding the right job is often more difficult than simply searching for job openings. A candidate may have relevant skills that are described differently in a job description, while recruiters may have to manually compare resumes against hundreds of requirements.

To explore this problem, I built **TalentSync AI**, an AI-powered resume-to-job matching platform that analyzes a candidate's resume against a job description and generates a structured assessment of their compatibility.

The platform combines **React, Spring Boot, PostgreSQL, Google Gemini, Docker, Kubernetes, Helm, and GitHub Actions** into a microservice-based architecture.

The goal was not just to integrate an LLM into an application, but to design a system that could be deployed, scaled, and maintained as independent services.

## The Problem

Traditional resume screening generally depends on keyword matching.

For example, if a job description requires:

* Java
* Spring Boot
* PostgreSQL
* Docker
* Kubernetes

a simple keyword-based system may only check whether these exact words appear in a resume.

However, candidates may describe the same experience differently.

For example:

> "Developed containerized applications and deployed them using Kubernetes."

This demonstrates Kubernetes and Docker-related experience even if the resume does not contain exactly the same wording as the job description.

This is where semantic understanding from an LLM can provide additional value.

## System Architecture

I designed TalentSync AI using separate services rather than building everything into one application.

The architecture consists of:

**React Frontend → Upload Service → Matching Service → PostgreSQL**

The application uses two independently deployable Spring Boot microservices:

1. **Upload Service**
2. **Matching Service**

Each service has its own PostgreSQL database.

The services and frontend are containerized with Docker and deployed through Kubernetes using Helm.

## Upload Service

The first step is uploading a candidate's resume.

The Upload Service is responsible for:

* Receiving the uploaded resume
* Processing the file
* Extracting relevant text
* Persisting the required information
* Providing the data needed by the matching workflow

Keeping this responsibility separate from the matching service makes the system easier to maintain and allows the upload functionality to evolve independently.

## Matching Service

The Matching Service is responsible for the intelligence behind the application.

It receives the candidate's resume information and job description and sends the relevant context to the **Google Gemini API**.

Instead of returning an unstructured paragraph, the application requests structured information such as:

* Overall fit score
* Matched skills
* Missing skills
* Skill gaps
* Job-resume compatibility analysis

This makes the AI output easier for the frontend to consume and present to users.

## Why PostgreSQL?

PostgreSQL was selected as the primary persistence layer because the application requires reliable relational storage while also benefiting from flexible JSON-based data.

One useful optimization was storing structured AI results using PostgreSQL's JSONB capability.

For repeated resume-job combinations, the application can reuse previously generated results rather than making another LLM request.

This has two important benefits:

**Lower latency**

Users do not need to wait for the same AI analysis repeatedly.

**Reduced AI API usage**

Repeated requests can be served from the database instead of generating the same result again.

## Containerization with Docker

Each application component is packaged as a Docker container.

The platform contains containers for:

* React frontend
* Upload Service
* Matching Service

Multi-stage Docker builds help separate the build environment from the runtime environment and keep the resulting images more efficient.

Containerization also makes the development and deployment environments more reproducible.

Instead of depending on a developer's local machine configuration, each service can run using the same container definition.

## Kubernetes Deployment

After containerizing the services, I used Kubernetes to orchestrate them.

The deployment configuration is managed using **Helm**, allowing the application components to be described and deployed consistently.

This provides several advantages:

* Independent service deployment
* Reproducible environments
* Centralized configuration
* Easier scaling
* Clear separation between application components

For local Kubernetes development, I used a kind cluster to reproduce the deployment workflow without requiring a full cloud Kubernetes environment.

## CI/CD

The project also includes a GitHub Actions-based CI pipeline.

The purpose of the pipeline is to automate repetitive steps such as building and validating application components.

This creates a more reliable development workflow:

**Code → GitHub → CI Pipeline → Build → Container → Deployment**

Automation is especially important in a microservice architecture because multiple services need to remain consistent as the application evolves.

## Designing AI Features Responsibly

Integrating an LLM into an application is not simply about sending a prompt and displaying the response.

A production-oriented AI application needs to consider:

* Input validation
* Prompt structure
* Output format
* Error handling
* API failures
* Response consistency
* Caching
* Cost
* Latency

For TalentSync AI, structured output was particularly important because the frontend needs predictable fields such as the fit score, matched skills, missing skills, and gap analysis.

This makes the AI component behave more like a service within the application rather than an uncontrolled text generator.

## Challenges

One of the main challenges was deciding how to divide responsibilities between services.

If too much functionality is placed inside a single service, the system starts moving toward a monolithic architecture.

On the other hand, creating too many small services can introduce unnecessary complexity.

The goal was therefore to create service boundaries around meaningful responsibilities.

Another challenge was managing AI-generated data.

LLM calls can introduce additional latency and cost, so caching previously generated results became an important optimization.

## What I Learned

This project taught me that building an AI-powered application requires more than understanding an AI API.

The application needs to combine several engineering disciplines:

**Frontend development** for creating a usable interface.

**Backend engineering** for APIs and business logic.

**Database design** for reliable persistence and caching.

**AI integration** for semantic analysis.

**Containerization** for reproducible environments.

**Kubernetes** for deployment and orchestration.

**CI/CD** for automation.

The most important lesson was that AI should be treated as one component of a larger software system rather than the entire system itself.

## What's Next?

There are several improvements I would consider for the next version of TalentSync AI.

These include:

* Supporting additional LLM providers
* Adding authentication and user accounts
* Introducing asynchronous processing for large documents
* Adding observability and distributed tracing
* Improving resume parsing for complex layouts
* Adding evaluation datasets to measure matching quality
* Introducing semantic/vector search for more advanced candidate-job matching
* Deploying the platform to a managed Kubernetes environment

## Conclusion

TalentSync AI started as an experiment in combining AI with software engineering, but it became an opportunity to explore how modern applications can integrate LLMs with microservices, databases, containers, and cloud-native deployment practices.

The project reinforced an important principle for me:

> **Good AI products are not just about choosing a powerful model. They are about designing reliable software around that model.**

By combining AI capabilities with solid engineering practices, it becomes possible to build applications that are not only intelligent, but also maintainable, deployable, and scalable.
