# WhatsApp-Google Drive Assistant Workflow

Control your Google Drive via WhatsApp commands using Twilio Sandbox, Google OAuth, AI summarization, and n8n automation.

---

## Setup Instructions

### 1. Twilio Sandbox for WhatsApp

- Go to [Twilio Console](https://www.twilio.com/console) and log in or sign up.
- Navigate to **Messaging > Try it Out > WhatsApp Sandbox**.
- Send the provided join code to the sandbox WhatsApp number.
- Set the sandbox webhook URL for incoming messages to:  

https://your-server.com/webhook/whatsapp-webhook

- Record your **Account SID** and **Auth Token** for environment variables.

### 2. Google OAuth Credentials

- Visit [Google Cloud Console](https://console.cloud.google.com/apis/credentials).
- Enable **Google Drive API** and **Google Sheets API**.
- Create OAuth 2.0 Client ID credentials (Web Application).
- Set the **Authorized redirect URI** to:  

https://your-server.com/rest/oauth2-credential/callback

- Save the **Client ID** and **Client Secret** for environment variables.

### 3. WhatsApp Command Syntax

| Command                        | Description                      | Example                            |
|-------------------------------|---------------------------------|----------------------------------|
| `LIST /FolderName`             | List all files in the folder    | `LIST /ProjectX`                  |
| `DELETE /FolderName/File.pdf` | Delete a specific file           | `DELETE /ProjectX/report.pdf`    |
| `MOVE /File.pdf /Archive`      | Move file to another folder      | `MOVE /ProjectX/report.pdf /Archive` |
| `SUMMARY /FolderName`          | Generate AI summary of files    | `SUMMARY /ProjectX`               |

*Commands are case-insensitive and paths must match your Google Drive structure.*

---

## Environment Variables

Create a `.env` file in your project folder with the following content, replacing placeholders:

```env
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_WHATSAPP_SANDBOX_NUMBER=your_twilio_whatsapp_sandbox_number

GOOGLE_OAUTH_CLIENT_ID=your_google_oauth_client_id
GOOGLE_OAUTH_CLIENT_SECRET=your_google_oauth_client_secret

OPENAI_API_KEY=your_openai_api_key

N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=changeme

as i have made in  in my repo fill up the credentials