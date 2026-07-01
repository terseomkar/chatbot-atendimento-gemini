# chatbot-atendimento-gemini

A Discord customer-service chatbot for the fictional delivery company **Hare Express**, powered by Google's **Gemini** model. The bot answers customer questions using an attached FAQ document and can look up live parcel-tracking information through a function-calling tool.

> This is a fork/copy of the original project by [alexandreaquiles](https://github.com/alexandreaquiles/chatbot-atendimento-gemini). The FAQ and system prompt are written in Portuguese (Brazil).

## How it works

1. **Discord front end** — the bot connects to Discord via `discord.py` and listens for messages in any channel it can see (`atendimento_bot.py`).
2. **Gemini with grounding** — each incoming message is forwarded to the `gemini-1.5-flash` model. A system prompt instructs the bot to act as a polite, formal Hare Express agent and to answer **only** from the attached FAQ (`faq-hare-express.pdf`), declining anything unrelated to the company's services. Temperature is set to `0` for deterministic answers.
3. **Automatic function calling / tracking tool** — the model is given the `busca_info_rastreamento` tool (`correios_rastreamento.py`). When a customer asks about a shipment, Gemini automatically calls this function with the tracking code, which queries the [Link & Track](https://linketrack.com) API and returns the delivery events. Automatic function calling wires the tool result back into the conversation.
4. **Reply** — the model's response is posted back to the same Discord channel.

## Project structure

| File | Purpose |
|---|---|
| `atendimento_bot.py` | Main entry point: Discord client, Gemini setup, message handling |
| `correios_rastreamento.py` | Parcel-tracking tool (Gemini function-calling target) with retry-enabled HTTP client |
| `faq-hare-express.pdf` | Knowledge base uploaded to Gemini and used to ground answers |
| `faq-hare-express.md` | English translation of the FAQ (reference) |
| `requirements.txt` | Python dependencies |
| `.gitignore` | Excludes the virtualenv, `.env`, and `__pycache__` |

## Requirements

- Python 3.9+
- A Google Gemini (Generative AI) API key
- A Discord bot token (with the **Message Content** privileged intent enabled)

## Setup

```bash
# 1. Create and activate a virtual environment
python -m venv bot-env
source bot-env/bin/activate        # Windows: bot-env\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure secrets in a .env file (see below)

# 4. Run the bot
python atendimento_bot.py
```

## Environment variables

Create a `.env` file in the project root (it is gitignored):

```env
GEMINI_API_KEY=your_google_gemini_api_key
DISCORD_CLIENT_TOKEN=your_discord_bot_token
```

## Notes

- The Discord bot needs the **Message Content Intent** turned on in the Discord Developer Portal, since the code enables `intents.message_content`.
- The tracking tool uses a public demo endpoint of the Link & Track API; for production use you should supply your own credentials.
- Answers are constrained to the FAQ content, so the bot intentionally refuses off-topic questions.
