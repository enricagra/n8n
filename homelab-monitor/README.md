# Home Monitor (n8n Workflow)

This repository contains an n8n workflow (workflow_clean.json) called "Home Monitor" that gathers monitoring data from Zabbix, MikroTik, and other sources, processes logs, generates a health heartbeat file in a GitHub repo, and sends a summary email (optionally enriched by an LLM).

## Workflow overview

Name: Home Monitor
Purpose: Collect home lab / lab infrastructure telemetry, process and summarize it, update a heartbeat file in a GitHub repository, and send a summary email.

Main nodes and responsibilities:
- Schedule Trigger
  - Runs the workflow on a schedule (configured to trigger daily at 07:00, every day).
- Zabbix Request
  - Calls the Zabbix JSON-RPC API to retrieve hosts and telemetry (various `history.get` calls for metrics and logs).
- Suricata fast logs
  - Zabbix-backed request to retrieve Suricata logs (followed by data cleaning).
- Fix Suricata Logs (Code)
  - JavaScript node that normalizes the Suricata log results into structured items.
- SCA Summary
  - Another Zabbix request used for SCA data, followed by Fix SCA Table (Code) to format that data.
- Fix SCA Table (Code)
  - Cleans and formats the SCA report data.
- MikroTik Request
  - Calls a MikroTik REST API endpoint for router resource info (basic HTTP Basic Auth credential).
- GitHub Notification
  - Writes/edits a `heartbeat.txt` file in the configured GitHub repository (default repository set to `homelab-status`).
- Merge
  - Combines multiple branches of data before sending it to the LLM chain.
- Basic LLM Chain + Ollama Model
  - Uses a LangChain node and an Ollama model (`qwen2.5:3b`) to finalize RAW HTML for the report and write analysis; configured with a prompt that includes role and critical rules. This node is optional and requires the corresponding LangChain and Ollama setup.
- Send an Email
  - Sends the final HTML report via SMTP. Subject and HTML template are included inside the workflow but must be configured with real addresses and SMTP credentials.

## Files
- workflow_clean.json — The workflow exported from n8n (use this file to import into your n8n instance).

## Prerequisites
- n8n instance (self-hosted or cloud) compatible with the nodes used in this workflow.
- n8n nodes and community nodes:
  - n8n-nodes-base.httpRequest
  - n8n-nodes-base.github
  - n8n-nodes-base.emailSend
  - n8n-nodes-base.code
  - n8n-nodes-base.scheduleTrigger
  - n8n-nodes-base.merge
  - LangChain nodes: @n8n/n8n-nodes-langchain.chainLlm and @n8n/n8n-nodes-langchain.lmOllama (if using the LLM steps)
- External services and credentials configured in n8n:
  - Zabbix API credentials (zabbixApi credential inside nodes)
  - MikroTik HTTP Basic Auth credentials
  - GitHub personal access token with repo contents write access (to update heartbeat.txt)
  - SMTP credentials for sending email
  - Ollama API or local Ollama instance credentials if using Ollama model
- Timezone: The workflow is configured with timezone Asia/Manila. Adjust in workflow settings if needed.

## Importing into n8n
1. In your n8n instance, go to Workflows → Import from file and upload `workflow_clean.json`.
2. Review and update all credential references and node parameter placeholders (IPs, emails, webhook IDs, IDs, and secrets).
3. Enable the workflow (set `active` to true) after testing.

## Configuration notes & security
- The exported workflow contains placeholders for:
  - IP addresses (e.g., Zabbix API URL, MikroTik API URL)
  - Emails (`fromEmail`, `toEmail`)
  - Credential IDs (GitHub, SMTP, Zabbix, Ollama, etc.)
  - Webhook IDs, node IDs and other identifiers
- Do NOT commit real secrets (API tokens, passwords, private keys) to this repository. Use n8n's credential store.
- Carefully review the LLM prompt inside the `Basic LLM Chain` node. It contains rules and instructions meant for the model — do not expose private data to third-party LLM services unless allowed by policy.

## Customization
- Schedule: Edit the Schedule Trigger node to change the run frequency and time.
- GitHub repo/file: The GitHub node currently writes to `homelab-status/heartbeat.txt`. Change `repository` and `filePath` if you want to write somewhere else.
- Email templates: Edit the subject and HTML template in the `Send an Email` node.
- Model: Swap or reconfigure the Ollama model node if you prefer a different model or provider.

## Troubleshooting
- If a node fails due to credential issues, confirm the credential connection and permissions.
- For API failures (timeouts, 4xx/5xx), check network access from your n8n host to the target services.
- When debugging JavaScript code nodes, use console logs or temporary email outputs to inspect intermediate payloads.

## License & contact
This README is provided as-is. Adapt it to your needs. For questions about the workflow configuration, contact the repository owner.
