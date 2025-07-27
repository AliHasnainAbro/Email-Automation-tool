Cold Email Automation with Make.com
This repository contains the blueprint for a Cold Email Automation tool built using Make.com (formerly Integromat). This automation streamlines your cold outreach by fetching prospect data from Google Sheets, generating personalized emails using an AI model (OpenRouter), sending them via Google Email, and updating the status in your Google Sheet. It also includes a follow-up mechanism.

Table of Contents
Features

How it Works

Setup Guide

Prerequisites

Google Sheet Structure

Make.com Setup

Flow Breakdown

Error Handling

Customization

Features
Automated Prospect Fetching: Automatically retrieves new prospect data from a specified Google Sheet.

AI-Powered Personalization: Uses an AI model (Mistral: Mixtral 8x7B Instruct via OpenRouter) to generate highly personalized cold emails based on prospect data.

Automated Email Sending: Sends generated emails using Google Email.

Status Tracking: Updates the Google Sheet with the email "Status" (e.g., "Sent").

Follow-Up Mechanism: Automatically sends a follow-up email if the initial email status is "sent".

Error Handling: Includes basic error handling to ignore issues during email sending.

Configurable Delay: Introduces a configurable delay between sending emails to avoid rate limits.

How it Works
The Make.com scenario is triggered by new rows in a Google Sheet. For each new row (prospect):

Watch Rows (Google Sheets): The scenario starts by watching for new rows in your designated Google Sheet.

Router: A router then directs the flow based on the "Status" column in your Google Sheet:

First Email Path: If the "Status" is not "sent", it proceeds to generate and send the initial cold email.

Follow-Up Path: If the "Status" is "sent", it proceeds to send a follow-up email.

Create a Chat Completion (OpenRouter - for First Email): An AI model generates the personalized cold email content. It uses the about, companyName, and jobRole fields from your Google Sheet to craft a compelling message.

Send an Email (Google Email): The generated email (or the pre-defined follow-up email) is sent to the prospect's email address.

Update a Row (Google Sheets - for First Email): After the initial email is sent, the "Status" column in the Google Sheet for that prospect is updated to "Sent".

Sleep (Utility - for First Email): A configurable delay is introduced to prevent overwhelming email servers or hitting API rate limits.

Setup Guide
Prerequisites
A Make.com account.

A Google account with access to Google Sheets and Google Mail.

An OpenRouter account with an API key (for AI email generation).

Google Sheet Structure
Your Google Sheet should have the following columns (case-sensitive, as used in the blueprint):

Column Name

Description

Example Value

fullName

Full name of the prospect.

John Doe

firstName

First name of the prospect (used for personalization in subject lines).

John

about

A brief description about the prospect or their company (for AI context).

Scaling sales processes

jobRole

The prospect's job role.

Head of Sales

companyName

The prospect's company name.

Acme Corp

email

The prospect's email address.

john.doe@example.com

Status

Crucial: This column tracks the email status (e.g., "Sent").

Sent

Important: The blueprint expects these column headers to be in the first row (A1:Z1). The Status column is specifically G (index 6 in Make.com's 0-based indexing).

Make.com Setup
Import the Blueprint:

Log in to your Make.com account.

Navigate to "Scenarios".

Click "Create a new scenario" or "Import blueprint".

Upload the Cold Email Automation.blueprint.json file.

Configure Connections:

Google Sheets Connection:

Click on the "Watch Rows" module (the first module on the left).

You will see a connection field. Click "Add" or select your existing Google connection. Ensure it has access to the Google Sheet you intend to use.

Select the correct Spreadsheet ID (the Google Sheet where your prospect data is) and Sheet Name (e.g., Sheet1).

OpenRouter Connection:

Click on the "Create a Chat Completion" module.

Add or select your OpenRouter connection. You will need your OpenRouter API key.

Google Email Connection:

Click on both "Send an Email" modules.

Add or select your Google Restricted connection. This connection will be used to send emails from your Gmail account.

Review and Adjust Mappings:

Google Sheets - Watch Rows: Verify that the Limit (e.g., 10) is appropriate for how many new rows you want to process per run.

OpenRouter - Create a Chat Completion:

Review the System and User messages in the Messages array. These define the AI's persona and the prompt for generating emails.

Ensure the mappings {{31.2}} (about), {{31.4}} (companyName), {{31.3}} (jobRole) are correctly pulling data from your Google Sheet.

The model is set to mistralai/mixtral-8x7b-instruct. You can change this to another compatible model if desired.

Google Email - Send an Email (First Email):

To: {{31.5}} (maps to the email column).

Subject: Surprise for {{31.1}}! (uses the firstName column for personalization).

HTML: {{5.choices[].message.content}}<br>\n<p>Regards,<p>\nAli Abro<br>\nAI Automation Engineer (inserts the AI-generated email content and your signature).

Google Email - Send an Email (Follow-Up):

To: {{31.5}} (maps to the email column).

Subject: Surprise for {{31.1}}! (same as the first email, you might want to adjust this for follow-ups).

HTML: Just wanted to quickly follow up on my last email regarding how AI could streamline your client support tasks. Is this something that's still on your radar?<br><br>\n\nThanks (pre-defined follow-up message).

Google Sheets - Update Row:

Spreadsheet ID: Ensure this points to the same Google Sheet as the "Watch Rows" module.

Row number: {{31.ROW_NUMBER}} (automatically gets the row number from the watched row).

Values: {"6": "Sent"} (updates column G with "Sent").

Sleep (Utility):

Duration: Set to 30 seconds. Adjust this value as needed to control the delay between emails.

Activate the Scenario:

Once all connections are set up and mappings are reviewed, save the scenario and turn it "ON".

Set up the scheduling for the "Watch Rows" module (e.g., run every 15 minutes, hourly, etc.) to check for new prospects.

Flow Breakdown
The scenario consists of the following modules:

Google Sheets - Watch Rows (ID: 31)

Purpose: Triggers the scenario when new rows are added to the specified Google Sheet.

Configuration: Watches Sheet1 in the spreadsheet /1e90quE0Sz9nPv9A2oj3q3lTBTZEt9Kb1LFUPdJ_YD90 (named "New Prospects 17 july 2025"). It processes up to 10 rows per run and assumes the first row contains headers.

Output: Provides data for each row, including fullName, firstName, about, jobRole, companyName, email, and Status.

Router (ID: 11)

Purpose: Directs the flow based on conditions, allowing for different actions for initial emails and follow-ups.

Route 1: Follow-up Email

Filter: follow up

Condition: Status (column G) text:contain sent. This path is taken if the initial email has already been sent.

Google Email - Send an Email (ID: 13)

Purpose: Sends a pre-defined follow-up email.

Configuration: Sends to the prospect's email (F) with a subject line personalized with firstName (B).

Error Handling: Includes an onerror route to builtin:Ignore (ID: 18), which will simply ignore any errors during the follow-up email sending.

Route 2: First Email & Status Update

Filter: 1st email

Condition: Status (column G) text:notequal sent. This path is taken if the email has not yet been sent.

OpenRouter - Create a Chat Completion (ID: 5)

Purpose: Generates the personalized cold email content using AI.

Configuration: Uses the mistralai/mixtral-8x7b-instruct model. The system prompt defines the AI as an expert cold email copywriter, and the user prompt provides prospect data (about, companyName, jobRole) for personalization.

Google Email - Send an Email (ID: 12)

Purpose: Sends the AI-generated initial cold email.

Configuration: Sends to the prospect's email (F) with a subject line personalized with firstName (B). The HTML content is the output from the OpenRouter module, followed by a signature.

Error Handling: Includes an onerror route to builtin:Ignore (ID: 28) to catch and ignore any errors during the initial email sending.

Google Sheets - Update Row (ID: 4)

Purpose: Updates the Status column in the Google Sheet to "Sent" after the initial email is successfully sent.

Configuration: Targets the same spreadsheet and sheet, updating the row corresponding to the processed prospect.

Utility - Sleep (ID: 27)

Purpose: Pauses the scenario for a specified duration (30 seconds) to ensure a delay between sending emails. This helps in adhering to email sending limits and avoiding being flagged as spam.

Error Handling
The blueprint includes basic error handling using the builtin:Ignore module on the "Send an Email" modules. This means if an email fails to send for any reason, the scenario will continue without stopping, but the error will be logged in Make.com's history. For more robust error handling, you might consider:

Logging Errors: Sending error notifications to a Slack channel, email, or a dedicated error log sheet.

Retries: Implementing retry mechanisms for failed email sends.

Specific Error Handling: Differentiating between various error types (e.g., invalid email address vs. temporary server issue) and taking appropriate actions.

Customization
Email Content:

Modify the System and User prompts in the "Create a Chat Completion" (OpenRouter) module to refine the AI's email generation style, length, and content.

Adjust the follow-up email content in the "Send an Email" (ID: 13) module.

Google Sheet Columns: If you change your Google Sheet column names or add new ones, you will need to update the mappings in the Make.com modules accordingly (e.g., {{31.0}}, {{31.1}}, etc., will change).

AI Model: Experiment with different AI models available on OpenRouter by changing the model parameter in the "Create a Chat Completion" module.

Delay Duration: Adjust the duration in the "Utility - Sleep" module (ID: 27) to control the delay between emails.

Scheduling: Configure the scheduling of the "Watch Rows" module to match your desired frequency for checking new prospects.

Subject Lines: Customize the subject lines for both initial and follow-up emails.

Signature: Update the email signature in the "Send an Email" modules.
