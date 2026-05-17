# Job-Hunt-Autopilot 🚀🤖

A full-fledged, AI-powered job application automation system orchestrated entirely by **Kestra**. 

No backend required. Just pure orchestration. This project scrapes job boards every morning, uses **Groq AI** to score listings against your profile, automatically writes tailored cover letters for the best matches, logs everything into PostgreSQL, and sends you a ranked digest email. It even handles automated 7-day follow-ups!

---

## 🌟 The Automation Magic (Features)

- **🕸️ Automated Job Scraping**: Pulls job listings from multiple sources automatically on a schedule.
- **🧠 AI Scoring & Matchmaking**: Leverages the blazing-fast **Groq AI** API to score each job listing against your specific skills and experience.
- **📝 Auto-Generated Cover Letters**: Groq AI automatically drafts a highly tailored, role-specific cover letter for the best job matches.
- **🗄️ PostgreSQL Tracking**: Securely logs every job, score, and application status into a structured database.
- **📧 Ranked Daily Digest**: Sends a beautifully formatted HTML email every morning with your top job matches and AI-generated insights.
- **⏰ Automated Follow-ups**: A cron job runs every Monday, checks PostgreSQL for applications older than 7 days, and sends a follow-up reminder email template.
- **🔄 Database Synchronization**: Automatically updates the `followup_sent` status to prevent duplicate follow-ups.

---

## 📂 Workflows Included

1. **`job-hunter-main.yml`** (The Core Engine):
   - Triggered on a schedule (e.g., every morning).
   - Scrapes job data and filters listings.
   - Passes listings to a Python script that calls the **Groq API** (Llama3) to score them against your profile.
   - Generates actionable job search tips.
   - Outputs a beautifully crafted HTML email digest containing the top jobs and custom cover letters.

2. **`job-hunter-tracker.yml`** (The Webhook Logger): 
   - Triggered by a Webhook when you actually apply.
   - Inserts the record into the `jh_applications` PostgreSQL table.
   - Sends a confirmation email that the application was tracked.

3. **`job-hunter-followup.yml`** (The Closer):
   - Triggered by a CRON Schedule (Every Monday at 10:00 AM IST).
   - Queries PostgreSQL for applications older than 7 days where a follow-up hasn't been sent.
   - Marks those records as `followup_sent = TRUE` and sends you an email containing an actionable follow-up template.

---

## 🚀 Quick Setup (Zero to Hero in 2 Minutes)

1. **Start the Environment**
   Run the included Docker Compose file to instantly spin up Kestra and a PostgreSQL database perfectly configured for these workflows:
   ```bash
   docker-compose up -d
   ```
   Kestra will be available at `http://localhost:8080`.

2. **Configure Your Secrets in Kestra**
   Go to your Kestra UI (`http://localhost:8080`), navigate to **Namespaces**, and create the following secrets under the `job.hunter` namespace:
   - `GROQ_API_KEY`: Your API key from Groq.
   - `GMAIL_APP_PASSWORD`: Your Gmail app password for SMTP.

3. **Configure Your Profile**
   In the Kestra UI, update the inputs in `job-hunter-main.yml` with your details (Name, Skills, Target Roles, Experience).

4. **Import Workflows**
   Go to **Flows** and import `job-hunter-main.yml`, `job-hunter-tracker.yml`, and `job-hunter-followup.yml`.

5. **Enable Triggers**
   Turn on the schedules and you are officially on Autopilot! ✈️

---

## 📸 Screenshots & Architecture

Here are visuals of the workflows, architecture, and execution outputs:

| Flow Graph & Topology | Execution Gantt & Email |
| :---: | :---: |
| <img src="./Outputs/flow-graph-1778335718144.jpeg" alt="Flow Graph" width="400"/> <br> <img src="./Outputs/Topology.png" alt="Topology" width="400"/> | <img src="./Outputs/GANTT.png" alt="Gantt Chart" width="400"/> <br> <img src="./Outputs/Email.png" alt="Email Output" width="400"/> |

*Additional visuals: [Kestra Architecture](./Outputs/Architect_kestra.png), [Kestra Certificate](./Outputs/Certificate-Kestra.png)*

---

## 📜 License

This project is licensed under the MIT License.
