# Job-Hunt-Autopilot

An autonomous, serverless job application tracking and orchestration pipeline built entirely on **Kestra**. 

This system operates as a zero-backend ETL and automation pipeline. It aggregates job listings from multiple APIs, evaluates candidate-role alignment using Groq AI (Llama-3), generates tailored cover letters dynamically, and manages stateful follow-up logic via PostgreSQL.

## System Architecture

The project leverages pure workflow orchestration to design a resilient system that executes complex, multi-stage data pipelines without requiring a dedicated backend service.

### Core Capabilities

- **Automated Ingestion**: Scheduled data extraction from external job boards via REST endpoints.
- **LLM Integration**: Direct API integration with Groq to execute inference (Llama-3-8b-8192) for candidate scoring and cover letter generation.
- **State Management**: PostgreSQL serves as the source of truth, persisting application states, evaluation scores, and chronological audit logs.
- **Asynchronous Communications**: Automated digest compilation and HTML email dispatching.
- **Idempotent Follow-ups**: Stateful cron-triggered workflows that query PostgreSQL to detect stale applications (>7 days) and execute automated follow-up sequences.

## Workflow Topologies

### 1. `job-hunter-main.yml` (The Aggregation & Inference Engine)
Acts as the primary scheduled orchestrator.
- **Data Extraction**: Pulls and sanitizes job data.
- **Processing**: Iterates through datasets and executes Python scripts to communicate with the Groq API.
- **Inference**: Scores job descriptions against candidate profiles and generates custom cover letters.
- **Delivery**: Compiles results into an HTML digest and dispatches via SMTP.

### 2. `job-hunter-tracker.yml` (The Webhook Listener)
Acts as the entry point for manual or extension-driven applications.
- **Trigger**: Listens on a dedicated Kestra webhook endpoint.
- **Persistence**: Executes `INSERT` operations into the `jh_applications` PostgreSQL table.
- **Notification**: Triggers a confirmation email via the Kestra mail plugin.

### 3. `job-hunter-followup.yml` (The State Reconciler)
Ensures applications do not go cold.
- **Trigger**: Cron schedule (`0 10 * * 1`).
- **Query**: Selects records where `status = 'applied'`, `applied_at < NOW() - INTERVAL '7 days'`, and `followup_sent = FALSE`.
- **Execution**: Updates state to prevent duplicate execution and dispatches follow-up emails.

## Deployment & Configuration

### Environment Setup

A `docker-compose.yml` is provided to instantly provision a localized Kestra orchestration server and the required PostgreSQL instance.

```bash
docker-compose up -d
```
Kestra will initialize and become available at `http://localhost:8080`.

### Secret Management

To maintain security standards, credentials must not be hardcoded. Configure the following secrets within the Kestra UI under the `job.hunter` namespace:

- `GROQ_API_KEY`: Your Groq API authentication token.
- `GMAIL_APP_PASSWORD`: SMTP application password for dispatching emails.

### Workflow Initialization

1. Navigate to the Kestra UI -> **Flows**.
2. Import `job-hunter-main.yml`, `job-hunter-tracker.yml`, and `job-hunter-followup.yml`.
3. Update the candidate variables (Name, Skills, Target Roles, Experience) in the `inputs` section of `job-hunter-main.yml`.
4. Enable the triggers to activate the pipeline.

## Execution Outputs

| Execution Topology | Pipeline Observability |
| :---: | :---: |
| <img src="./Outputs/flow-graph-1778335718144.jpeg" alt="Flow Graph" width="400"/> <br> <img src="./Outputs/Topology.png" alt="Topology" width="400"/> | <img src="./Outputs/GANTT.png" alt="Gantt Chart" width="400"/> <br> <img src="./Outputs/Email.png" alt="Email Output" width="400"/> |

*Reference Documentation: [Kestra Architecture](./Outputs/Architect_kestra.png), [Certification](./Outputs/Certificate-Kestra.png)*

## License

Distributed under the MIT License.
