# Dewald Laubscher

**Senior / Lead Software Engineer — Full-Stack · Platform, Distributed Systems & Applied AI**  
Atlanta, GA · Open to Remote · laubscher.dewald@gmail.com  
linkedin.com/in/dewaldlaubscher · github.com/dewwwald

## Summary

Engineer with 12+ years across the full stack — from award-agency design craft to petabyte-scale distributed systems and applied AI. I've shipped products people touch and the platforms other engineers build on: large-scale event-driven pipelines and deep PostgreSQL on the backend; React and Angular with a full JavaScript-to-TypeScript migration on the frontend; and an applied-AI feature shipped to production that compiles natural language directly into a query AST. I was a founding engineer at a startup acquired by Mailchimp, so I move fast in ambiguity and take rough problems from prototype to production. I set standards other teams adopt without being told to, I mentor, and I care most about the details that change the outcome for the people downstream.

## Experience

### Intuit Mailchimp

**Senior Software Engineer | Aug 2022 - Jul 2026**  
Atlanta, GA

- Authored an RFC that resolved chronic customer-facing timeouts on serving large audience counts (up to ~49M records at p99): replaced real-time computation with an asynchronously precomputed value under eventual consistency — adopted cross-team, taken to first implementation by another team, then iterated until timeouts fell from ~1,000 affected accounts to a handful.
- Built and shipped to production an applied-AI feature that compiled plain-language customer intent directly into Mailchimp's segmentation AST — collapsing a complex multi-step workflow into a conversation, serving real users.
- Helped modernize the platform from colo monolith toward petabyte-scale, event-driven microservices — driving cross-team adoption of distributed-systems patterns (event-driven design, eventual consistency, precomputed read models) across Kafka, Redshift, Debezium, and Strangler Fig migrations.
- Built, maintained and managed the implementation cloud compute using Infrastructure as code (version controlled using GIT) through Terraform in Cloud Formation, Jenkins and Argo.
- Went deep on PostgreSQL at scale — read replicas, connection pooling, MVCC, query optimization — and designed REST APIs and read-model/caching layers for reliable serving.
- Cut algorithmic cost where it compounds: redesigned large-scale migration backfills with a hashing-based lookup, reducing processed-ID checks from O(n) to O(1) with idempotent, resumable, failure-tolerant execution.
- Led a cross-functional initiative that built BigQuery tooling over internal BI and commit-history data to identify ~$100K/month in dead infrastructure — coordinating access across BI, platform, and engineering orgs.
- Built AI orchestration and multi-agent tooling aimed at developer productivity, and use AI-assisted development daily to write, debug, and optimize code.
- Set engineering standards adopted beyond my own team — I built a dependency-injection framework ideation used by multiple teams — and mentored engineers on ownership, maintainability, and engineering judgment.
- Owned SLO definition and observability (Splunk p75/p99, PagerDuty, Wavefront, Grafana), on-call reliability, and end-to-end Playwright test suites wired into CI/CD with provisioned cloud infrastructure.
- Built Inbox, a customer-facing surface in React unifying email, SMS, and contact activity into a single per-contact timeline — reconciling heterogeneous sources into one coherent, stateful experience.

**Software Engineer (Contract) | Jun 2020 - Aug 2022**  
Remote (South Africa)

- Led technical implementation for a product engineering function, overseeing a team of engineers through Mailchimp's growth and acquisition by Intuit.
- Designed and led implementation of a website builder for surveys that supercharged the UX with multimedia and deeply customizable sections (React/Redux client-state architecture).
- Drawing on deep HTML and HTTP knowledge, led implementation of email cookies that let users begin answering a survey inside an email and continue to the web version with their responses already populated.
- Owned ingress infrastructure and peak-season capacity planning — sustaining ~10,000 requests/day with spikes into the millions of records, and moving high-usage customers onto dedicated database servers to hold availability under load.
- Partnered with the data-steward organization to define and implement the logging standards the team adopted — grounded in data minimization: store and log only what is strictly necessary, discard the rest.
- Implemented GDPR masking for logging along with other application security CSRF, Honeypot, Input validation.
- Built platform-side input validation that closed a supply-chain risk in our form builder — malicious code hidden inside reusable agency design templates that could exfiltrate data.

### BigTeam

**Founding Engineering Lead | Oct 2019 - Jun 2020**  
Johannesburg, South Africa

- Led a team of 4 engineers through the company's acquisition by Mailchimp — owning architecture decisions, code review, delivery planning, and production readiness.
- Built and owned CI/CD pipelines and deployment infrastructure from scratch, establishing the engineering foundation the team scaled on from platform engineering, Session Auth, OAuth and frameworks.
- Translated ambiguous product and business needs into executable engineering plans, and established practices around ownership, autonomy, and maintainable systems.

**Founding Engineer | Aug 2017 - Oct 2019**  
Johannesburg, South Africa

- Authored a shared library (published to an internal Node registry, installed on both frontend and backend) that ran identical event-driven logic across client and server — processing JSON event blobs as mutators applied to accumulating state: a custom, in-process implementation of the event-sourcing pattern.
- Migrated the entire product codebase from JavaScript to TypeScript single-handedly — a language-level upgrade to the foundation every engineer built on, delivered without disrupting active product development.
- Built the app-wide reactive client data layer in Angular using RxJS observables, service layers, and decorators (and with React/Redux at Mailchimp) — client-state architecture owned end to end.
- Designed and built the application's authentication and session infrastructure from scratch — OAuth flows, server-side session management, and JWT issuance/validation — securing both the client and backend surfaces of the product.
- Built four core product surfaces full-stack — a results dashboard, the survey builder, audience management, and the survey-taking experience — owning reusable components, a high degree of interaction and motion design, and the backend behind each.
- Built a custom annotation store that overlaid comments at pinned coordinates over both images and video, timestamping video comments so they surfaced in sync during playback — a spatial-and-temporal layer well beyond standard form tooling.
- Designed and implemented a custom NLP thematic-analysis pipeline using classical techniques — stemming, lemmatization, sentiment scoring, and rule-based classification — to surface recurring themes across customer feedback, years before LLMs made this turnkey.

### Prior Experience

**Front-End / Web Developer | Sep 2014 - Aug 2017**  
Johannesburg, South Africa

- Built around 80 pixel-perfect, fully responsive marketing and product websites in my first years as a developer at one of the region's top web-design agencies — where visual polish, interactivity, and craft were the product.
- Owned projects end-to-end across backend, frontend, deployment, and production support, establishing the design-and-build foundations that carried into every role since.
- Built out extendible CMS software in PHP to rapidly meet customer needs with increasing reusability and design flexibility.
- Built dynamic forms with security-first principles like CSRF, Honeypots, and input sanitization — avoiding SQL injection.

## Technical Skills

**Languages:** Go, TypeScript, JavaScript, Python, Java, Kotlin, Zig/C  
**Distributed Systems & Data:** Kafka, Debezium, Redshift, event-driven design, event sourcing, eventual consistency, precomputed read models, stream processing, petabyte-scale pipelines, backfills  
**Backend & APIs:** PostgreSQL at scale (read replicas, MVCC, query optimization), REST API design, HTTP, caching / read-model design, message queues (Kafka, SQS, RabbitMQ)  
**Frontend:** React, Angular, TypeScript (full JS→TS migration), client-side data & state (RxJS/reactive and Redux/flux), reusable components, pixel-perfect responsive design, interaction & motion  
**Applied AI:** LLM orchestration, multi-agent / agentic tooling, prompt orchestration, natural language-to-AST / code generation, AI-assisted development (daily), classical NLP; currently deepening RAG, embeddings, vector databases & evaluation  
**Platform & Infra:** AWS, Kubernetes, Docker, CI/CD, GitHub Actions, Strangler Fig migrations, colo-to-cloud  
**Observability:** Splunk (p75/p99 SLOs), PagerDuty, Wavefront, Grafana, Bugsnag; on-call & incident response  
**Data Privacy & Governance:** logging standards, PII handling, data minimization, privacy-conscious logging, data-steward collaboration  
**Architecture & Leadership:** Domain-Driven Design, dependency injection, cross-team standards without authority, mentoring, 0→1 delivery, architectural tradeoffs, product sense

## Education

**B.Sc. Computer Science & Information Technology** — North-West University (NWU), Potchefstroom, South Africa · 2011–2013
