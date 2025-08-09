# WhatsApp Google Drive Assistant

A ready-to-import n8n workflow for WhatsApp-based Google Drive file management and AI summaries.

## Features

- WhatsApp command interface (via Twilio Sandbox)
- Google Drive integration (OAuth2, user-scoped)
- AI summaries (OpenAI GPT-4o, Claude, or Groq)
- Audit logging and safety checks (confirmation for deletes)
- Dockerized deployment

## Setup

1. **Clone this repo**
2. **Copy `.env.sample` to `.env`** and fill in your credentials.
3. **Start n8n via Docker:**
   ```sh
   docker-compose up -d
   ```
4. **Import `workflow.json` into n8n**
5. **Configure Twilio Sandbox for WhatsApp** and set webhook URL to your n8n instance.
6. **Set up Google Drive OAuth2 credentials in n8n**
7. **Set up Groq/OpenAI/Claude credentials in n8n**

## Command Syntax

- `LIST /folder` — List files in a folder
- `DELETE /file.ext CONFIRM` — Delete a file (requires CONFIRM)
- `MOVE /source /destination` — Move or rename files
- `SUMMARY /folder` — AI summary of documents
- `HELP` — Show help message

## How it Works

This solution uses [n8n](https://n8n.io/) to automate WhatsApp-based file management and AI summaries for Google Drive. Here’s an overview of the workflow:

1. **Inbound WhatsApp Message:**  
   - Twilio forwards incoming WhatsApp messages to your n8n webhook (proxied via Railway/ngrok).
2. **Message Validation & Parsing:**  
   - The workflow checks if the message is valid and parses the command (e.g., LIST, DELETE, MOVE, SUMMARY, HELP).
   - If the command is invalid or incomplete, a helpful fallback or error message is sent.
3. **Command Routing:**  
   - Depending on the command, the workflow routes the request:
     - **LIST:** Lists files in the specified Google Drive folder.
     - **DELETE:** Deletes a file (only after receiving a `CONFIRM` keyword for safety).
     - **MOVE:** Moves or renames a file.
     - **SUMMARY:** Fetches file info and sends it to an AI (Groq/OpenAI/Claude) for a concise summary.
     - **HELP:** Sends a help message with usage instructions.
4. **Google Drive Integration:**  
   - All file operations use OAuth2 and only access the authenticated user’s Google Drive.
5. **AI Summarization:**  
   - For the SUMMARY command, file details are sent to an AI model to generate a WhatsApp-friendly summary.
6. **Response Formatting:**  
   - All responses are formatted with emojis and concise text for WhatsApp.
7. **Audit Logging:**  
   - Every operation is logged for traceability and safety.
8. **WhatsApp Response:**  
   - The formatted response is sent back to the user via Twilio WhatsApp.

This workflow is modular, secure, and extensible. All secrets and configuration are managed via environment variables.

## Security

- Only your Google Drive files are accessible.
- Delete requires explicit confirmation.

## Deployment

- See `docker-compose.yml` for local deployment.
- All secrets/configuration via `.env`.

## Logging

- Audit logs are output to the n8n console (can be extended to Google Sheets).

## Exposing n8n to the Internet (Proxy/Webhook)

To receive WhatsApp messages from Twilio, your n8n instance must be accessible from the internet. We recommend using [Railway](https://railway.app/) as a proxy:

1. **Deploy Railway Proxy:**
   - Fork or create a new Railway project.
   - Point it to your local n8n Docker instance (port 5678).
   - Railway will provide a public URL, e.g., `https://your-railway-app.up.railway.app`.

2. **Configure n8n:**
   - Set `WEBHOOK_TUNNEL_URL` in your `.env` to the Railway public URL.
   - n8n will use this as the webhook endpoint.

3. **Set Twilio Webhook:**
   - In your Twilio Sandbox, set the incoming webhook to:
     ```
     https://your-railway-app.up.railway.app/webhook/whatsapp-webhook
     ```

4. **Online Services in Docker:**
   - Docker provides outbound internet access by default.
   - No extra configuration is needed for n8n to call Google Drive, Groq/OpenAI, or Twilio APIs.

## Example Railway Proxy Setup

- See [Railway docs](https://docs.railway.app/deploy/deployments) for details.
- You can also use [ngrok](https://ngrok.com/) or [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) as alternatives.

## License

MIT

## Demo video:
https://www.youtube.com/watch?v=XY8j_kh2nUM
