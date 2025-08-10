# WhatsApp-Google Drive Assistant Workflow

Control your Google Drive via WhatsApp commands using Twilio Sandbox, Google OAuth, AI summarization, and n8n automation. This workflow allows you to list files, delete files, move files between folders, or summarize file contents in Google Drive by sending commands via WhatsApp. All credentials are stored in a `.env` file for secure configuration.

---

## Overview

This n8n workflow integrates:
- **Twilio** for WhatsApp messaging to receive commands and send responses.
- **Google Drive API** to manage files and folders (list, delete, move).
- **Google Sheets API** to log actions (e.g., timestamps, commands, results).
- **Hugging Face API** for summarizing text from PDF, DOCX, or TXT files.
- **n8n** as the automation platform to orchestrate the workflow.

The workflow supports four commands: **LIST**, **DELETE**, **MOVE**, and **SUMMARY**. Each command triggers specific actions, and results are logged in a Google Sheet and sent back via WhatsApp.

---

## Setup Instructions

### 1. Prerequisites

- An n8n instance (cloud or self-hosted). For self-hosting, install n8n via Docker or npm. See [n8n installation guide](https://docs.n8n.io/getting-started/installation/).
- A Twilio account with WhatsApp Sandbox enabled.
- A Google Cloud project with Google Drive and Google Sheets APIs enabled.
- A Hugging Face account with an API token for text summarization.
- Basic knowledge of n8n workflows, JSON, and environment variables.

### 2. Environment Variables

1. **Create a `.env` File**:
   - In your n8n project directory, create a `.env` file with the following content:
     ```env
     TWILIO_ACCOUNT_SID=your_twilio_account_sid
     TWILIO_AUTH_TOKEN=your_twilio_auth_token
     TWILIO_WHATSAPP_SANDBOX_NUMBER=your_twilio_whatsapp_sandbox_number
     GOOGLE_OAUTH_CLIENT_ID=your_google_oauth_client_id
     GOOGLE_OAUTH_CLIENT_SECRET=your_google_oauth_client_secret
     HUGGING_FACE_API_TOKEN=your_hugging_face_api_token
     N8N_BASIC_AUTH_ACTIVE=true
     N8N_BASIC_AUTH_USER=admin
     N8N_BASIC_AUTH_PASSWORD=your_secure_password
     ```
   - Replace placeholders with your actual credentials:
     - **Twilio**: Find `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, and `TWILIO_WHATSAPP_SANDBOX_NUMBER` in the [Twilio Console](https://www.twilio.com/console) under **Account > General Settings** and **Messaging > Try it Out > WhatsApp Sandbox**.
     - **Google**: Obtain `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET` from [Google Cloud Console](https://console.cloud.google.com/apis/credentials).
     - **Hugging Face**: Get `HUGGING_FACE_API_TOKEN` from [Hugging Face Settings > Access Tokens](https://huggingface.co/settings/tokens).
     - **n8n**: Replace `your_secure_password` with a strong password for n8n access.
2. **Ensure `.env` File Security**:
   - Store the `.env` file in a secure location and restrict access (e.g., set file permissions to `600` on Linux/macOS).
   - Do not commit the `.env` file to public repositories.

### 3. Twilio Sandbox for WhatsApp

1. **Sign Up/Login**: Go to [Twilio Console](https://www.twilio.com/console) and create or log into your account.
2. **Enable WhatsApp Sandbox**:
   - Navigate to **Messaging > Try it Out > WhatsApp Sandbox**.
   - Follow instructions to join the sandbox by sending the provided join code to the sandbox WhatsApp number (e.g., `+14155238886`).
3. **Set Webhook**:
   - In the Twilio WhatsApp Sandbox settings, set the **"When a message comes in"** webhook URL to your n8n webhook endpoint:
     ```
     https://your-n8n-instance.com/webhook/5cbf8fa3-452c-43ed-be05-ef5f6347e281
     ```
   - Ensure the HTTP method is set to `POST`.

### 4. Google OAuth Credentials

1. **Create a Google Cloud Project**:
   - Go to [Google Cloud Console](https://console.cloud.google.com/apis/credentials).
   - Create a new project or select an existing one.
2. **Enable APIs**:
   - Navigate to **APIs & Services > Library**.
   - Enable **Google Drive API** and **Google Sheets API**.
3. **Create OAuth 2.0 Credentials**:
   - Go to **APIs & Services > Credentials**.
   - Click **Create Credentials** and select **OAuth 2.0 Client IDs**.
   - Choose **Web Application** as the application type.
   - Set the **Authorized redirect URI** to:
     ```
     https://your-n8n-instance.com/rest/oauth2-credential/callback
     ```
   - Copy the **Client ID** and **Client Secret** to the `.env` file as `GOOGLE_OAUTH_CLIENT_ID` and `GOOGLE_OAUTH_CLIENT_SECRET`.
4. **Configure n8n Credentials**:
   - In n8n, go to **Credentials > Add Credential**.
   - Create a **Google Drive OAuth2 API** credential and a **Google Sheets OAuth2 API** credential.
   - Instead of manually entering `Client ID` and `Client Secret`, configure n8n to use environment variables:
     - Set `Client ID` to `{{process.env.GOOGLE_OAUTH_CLIENT_ID}}`.
     - Set `Client Secret` to `{{process.env.GOOGLE_OAUTH_CLIENT_SECRET}}`.
     - Complete the OAuth authentication flow by logging into your Google account when prompted.

### 5. Hugging Face API Token

1. **Get API Token**:
   - Go to [Hugging Face](https://huggingface.co/) and log in.
   - Navigate to **Settings > Access Tokens**.
   - Create a new token with **Read** access (or higher if required).
   - Copy the token and add it to the `.env` file as `HUGGING_FACE_API_TOKEN`.


### 6. Google Sheets Setup

1. **Create a Spreadsheet**:
   - In Google Drive, create a new Google Sheet named "WhatsApp action Spreadsheet" (or any name).
   - Note the **Spreadsheet ID** from the URL (e.g., `1ykYPqZ2U8ii5s2H6ND9eusysBIYg_H28j76ajVcQExE` in `https://docs.google.com/spreadsheets/d/1ykYPqZ2U8ii5s2H6ND9eusysBIYg_H28j76ajVcQExE/edit`).
   - Ensure the sheet has a tab named `Sheet1` (or note the `gid` if different).
2. **Configure Workflow**:
   - In the workflow, locate the `Append row in sheet` nodes (e.g., `Append row in sheet`, `Append row in sheet1`, etc.).
   - Update the `documentId` field with your Spreadsheet ID.
   - Update the `sheetName` field with the `gid` of your sheet tab (e.g., `gid=0` for `Sheet1`).

### 7. Running n8n

1. **Install n8n** (if self-hosting):
   - Use Docker to run n8n, ensuring the `.env` file is loaded. On Linux/macOS:
     ```bash
     docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n --env-file .env n8nio/n8n
     ```
   - On Windows (PowerShell or CMD):
     ```bash
     docker run -it --rm --name n8n -p 5678:5678 -v C:\Users\YourUsername\.n8n:C:\home\node\.n8n --env-file .env n8nio/n8n
     ```
     Replace `YourUsername` with your actual Windows username.
2. **Access n8n**:
   - Open your browser and go to `http://localhost:5678`.
   - Log in using the credentials from the `.env` file (e.g., username `admin`, password from `N8N_BASIC_AUTH_PASSWORD`).
3. **Import Workflow**:
   - In n8n, go to **Workflows > Import Workflow**.
   - Copy and paste the provided JSON workflow configuration into n8n.
   - Save and activate the workflow.

### 8. Configure Workflow Credentials

1. **Twilio Credentials**:
   - In n8n, go to **Credentials > Add Credential** and select **Twilio API**.
   - Use environment variables:
     - Set `Account SID` to `{{process.env.TWILIO_ACCOUNT_SID}}`.
     - Set `Auth Token` to `{{process.env.TWILIO_AUTH_TOKEN}}`.
   - Name the credential (e.g., `Twilio account`) and assign it to all `Send an SMS/MMS/WhatsApp message` nodes.
   - Update the `from` field in these nodes to `{{process.env.TWILIO_WHATSAPP_SANDBOX_NUMBER}}` (e.g., `+14155238886`).
   - Update the `from` field in these nodes to `{{process.env.TWILIO_WHATSAPP_USER_NUMBER}}` (e.g., `+14155238886`). put your main twilio user numbers
2. **Google Drive Credentials**:
   - Create a **Google Drive OAuth2 API** credential.
   - Use environment variables for `Client ID` and `Client Secret` as described above.
   - Assign this credential to all `Google Drive` and `HTTP Request` nodes (e.g., `Search files and folders`, `HTTP Request`, `Delete a file`, etc.).
3. **Google Sheets Credentials**:
   - Create a **Google Sheets OAuth2 API** credential.
   - Use the same environment variables (`GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET`).
   - Assign this credential to all `Append row in sheet` nodes.
4. **Verify Webhook**:
   - Ensure the `Webhook` node’s URL matches the one set in the Twilio WhatsApp Sandbox settings.

### 9. Test the Workflow

1. **Send a Test Command**:
   - From WhatsApp, send a message to the Twilio Sandbox number (e.g., `+14155238886`) with a command like:
     ```
     LIST /ProjectX
     ```
     ```
     DELETE /ProjectX/report.pdf
     ```
     ```
     MOVE /ProjectX/report.pdf /Archive
     ```
     ```
     SUMMARY /ProjectX
     ```
2. **Check Responses**:
   - The workflow will process the command and send a response via WhatsApp (e.g., a list of files, confirmation of deletion, or a file summary).
   - Verify that actions are logged in the Google Sheet under columns: `Time Stamp`, `Action`, `performed by`, `Result`, and `status`.
3. **Debugging**:
   - If errors occur, check the n8n workflow execution log.
   - Ensure the `.env` file is correctly loaded and all environment variables are accessible.
   - Verify all credentials and API access.

---

## WhatsApp Command Syntax

| Command                        | Description                      | Example                            |
|-------------------------------|---------------------------------|----------------------------------|
| `LIST /FolderName`             | Lists all files in the specified folder. | `LIST /ProjectX`                  |
| `DELETE /FolderName/File.pdf` | Deletes the specified file from Google Drive. | `DELETE /ProjectX/report.pdf`    |
| `MOVE /File.pdf /Archive`      | Moves a file to another folder in Google Drive. | `MOVE /ProjectX/report.pdf /Archive` |
| `SUMMARY /FolderName`          | Generates an AI summary of PDF, DOCX, or TXT files in the folder. | `SUMMARY /ProjectX`               |

*Notes*:
- Commands are case-insensitive (e.g., `list`, `LIST`, or `LiSt` all work).
- Paths must match your Google Drive folder/file structure exactly.
- For `DELETE`, only one file can be deleted per request to avoid errors (enforced by the workflow).
- For `SUMMARY`, only the first file (PDF, DOCX, or TXT) in the folder is summarized.

---

## Workflow Details

The workflow (`My workflow 4`) is structured as follows:

1. **Webhook Trigger**:
   - Receives WhatsApp messages via Twilio.
2. **Command Parsing** (`Code` node):
   - Extracts and processes the command (`LIST`, `DELETE`, `MOVE`, `SUMMARY`) and arguments.
3. **Switch Node**:
   - Routes the workflow based on the command to one of four branches.
4. **Branches**:
   - **LIST**: Queries Google Drive for files in the specified folder and sends a list via WhatsApp.
   - **DELETE**: Checks if the request is valid (single file), deletes the file, and logs the action.
   - **MOVE**: Identifies source file, source folder, and destination folder, moves the file, and logs the action.
   - **SUMMARY**: Retrieves a file (PDF, DOCX, or TXT), extracts text, summarizes it using Hugging Face’s BART model (with API token from `.env`), and sends the summary via WhatsApp.
5. **Logging**:
   - All actions are logged in a Google Sheet with details like timestamp, action type, user, result, and status.
6. **Error Handling**:
   - Invalid commands or multiple delete requests trigger error messages sent via WhatsApp.

---

## Troubleshooting

- **Webhook Not Triggering**: Ensure the Twilio webhook URL matches the n8n `Webhook` node’s URL and is publicly accessible (use a tool like ngrok for local testing).
- **Google API Errors**: Verify that Google Drive and Sheets APIs are enabled and OAuth credentials are correctly configured in the `.env` file.
- **Hugging Face API Errors**: Ensure the `HUGGING_FACE_API_TOKEN` in the `.env` file is valid and the model (`facebook/bart-large-cnn`) is accessible.
- **No Response in WhatsApp**: Check that `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, and `TWILIO_WHATSAPP_SANDBOX_NUMBER` are correctly set in the `.env` file and used in the workflow.
- **File Not Found**: Ensure the folder/file paths in commands match the exact Google Drive structure.
- **Log Issues**: Verify the Google Sheet ID and tab `gid` in `Append row in sheet` nodes.
- **Environment Variable Issues**: Ensure the `.env` file is in the correct directory and loaded by Docker (`--env-file .env`). Check that environment variables are accessible in n8n.

---

## Security Notes

- **Credentials**: Store all sensitive data (Twilio, Google, Hugging Face, n8n) in the `.env` file and restrict access (e.g., file permissions `600` on Linux/macOS).
- **Access Control**: Use a strong password for `N8N_BASIC_AUTH_PASSWORD` to secure your n8n instance.
- **Google Drive Permissions**: Ensure the Google account used has access to the folders/files specified in commands.
- **Rate Limits**: Be aware of Twilio, Google, and Hugging Face API rate limits to avoid service disruptions.

---

## Extending the Workflow

- **Add More Commands**: Extend the `Switch` node to support additional commands (e.g., `UPLOAD`, `RENAME`).
- **Custom Summarization**: Modify the Hugging Face API parameters in `Code3`, `Code4`, and `Code5` nodes to adjust summary length or use a different model.
- **Multi-File Summarization**: Update the `SUMMARY` branch to process multiple files in a folder.
- **Error Notifications**: Add a node to send error alerts to an admin via WhatsApp or email.

---

## License

This workflow is provided as-is for personal or organizational use. Ensure compliance with Twilio, Google, and Hugging Face terms of service when deploying.

For further assistance, refer to:
- [n8n Documentation](https://docs.n8n.io/)
- [Twilio WhatsApp API](https://www.twilio.com/docs/whatsapp)
- [Google Drive API](https://developers.google.com/drive)
- [Google Sheets API](https://developers.google.com/sheets)
- [Hugging Face API](https://huggingface.co/docs/api-inference)

Happy automating! 🚀