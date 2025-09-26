
### Technical Architecture

The system is designed as a modular, event-driven workflow orchestrated by a self-hosted n8n instance. This ensures security, privacy, and scalability.

**Core Components:**

1.  **Version Control System (VCS) - GitHub:**
    *   **Role:** Acts as the trigger for the workflow.
    *   **Mechanism:** A webhook is configured in the target repository to listen for `pull_request` events (e.g., `opened`, `synchronize`). When a PR is created or updated, the webhook sends a payload to the n8n endpoint.

2.  **Orchestration Engine - n8n (Self-Hosted):**
    *   **Role:** The central nervous system of the operation. It receives the trigger and executes the entire agentic workflow.
    *   **Workflow Nodes:**
        *   **Webhook Node:** The entry point. It receives the event data from GitHub.
        *   **GitHub Node:** Uses the GitHub API to fetch detailed information about the PR, including the changed files and code diffs.
        *   **Function Node (Code Preparation):** A custom JavaScript/Python node that processes the code diffs, prepares a clean context, and formulates a specific prompt for the AI model.
        *   **HTTP Request Node (AI Interaction):** Sends the prepared code and prompt to the AI Model's API endpoint. This is where the core analysis request happens.
        *   **Switch Node (Severity Routing):** A logic node that parses the AI's response. It directs the workflow down different paths based on the severity of the issues found (e.g., `CRITICAL`, `HIGH`, `LOW`).
        *   **Action Nodes:** Based on the routing, specific actions are taken:
            *   **GitHub Node:** To post comments back to the PR.
            *   **GitHub Node:** To create a new issue.
            *   **Teams/Slack Node:** To send high-priority alerts to designated channels.
        *   **Database/Data Store Node (Optional):** Logs the results of the review for metrics and dashboarding.

3.  **AI Model Core (Pluggable & Secure):**
    *   **Role:** The "brain" of the operation that performs the code analysis.
    *   **Implementation:** This is designed to be flexible. It can be an API endpoint for a large language model (like a fine-tuned Llama 3, a private GPT-4 instance, or a proprietary TCS model).
    *   **Hosting:** To ensure security and privacy, the model is hosted on-premise or in a private cloud, ensuring no source code ever leaves the secure environment.

**Workflow Execution Flow:**

1.  **Trigger:** A developer creates a Pull Request on GitHub.
2.  **Notify:** GitHub sends a webhook to the n8n instance.
3.  **Fetch:** The n8n workflow fetches the code changes from the PR.
4.  **Analyze:** The code is sent to the secure, self-hosted AI model for analysis.
5.  **Respond:** The AI model returns a structured JSON object detailing issues, suggestions, and severity levels.
6.  **Act:** The n8n workflow parses the response and takes action:
    *   Posts comments on the PR for all feedback.
    *   If critical issues are found, sends a notification to a Teams channel.
    *   If configured, creates a GitHub issue summarizing the findings.
7.  **Log:** The outcome of the review is logged to a database for quality tracking.
