# metri

A private food diary that lives in your conversation with your AI assistant.

Tell your AI assistant what you ate, in your own words or with a photo, and
metri keeps track of it. No forms, no barcode scanning, no looking things up.

## What you can do

- **Log meals naturally.** Type or dictate "two eggs and toast with butter", or
  share a photo of your plate, a package, or a nutrition label. Several foods
  can go in one message.
- **See your day at a glance.** Check calories, protein, carbs, and fat against
  your goals whenever you like.
- **Look back.** Review any day, week, or longer stretch, and fix or remove
  entries you got wrong.
- **Save what you eat often.** Keep personal foods and cooked batches so
  logging them next time takes a few words.
- **Set your goals.** Choose a daily calorie target, with optional protein,
  carbohydrate, and fat targets.

## Getting started

Install metri in your assistant and ask it to log something. The first time,
you'll sign in to metri. After that, just talk.

- **Claude:** open [metri in Claude's directory](https://claude.ai/directory/metri)
  and connect it.
- **ChatGPT:** open
  [metri in ChatGPT](https://chatgpt.com/plugins/plugin_asdk_app_6ae24c42c1608191b46a29ddbcfcae2c)
  and connect it.
- **Claude Code:**
  `claude mcp add --transport http metri https://mcp.metri-diary.com/mcp`
- **Gemini CLI:**
  `gemini extensions install https://github.com/Filich3737/metri-plugin`
- **Any other MCP client:** add the remote server
  `https://mcp.metri-diary.com/mcp` (Streamable HTTP, OAuth sign-in).

## Usage

Talk to your assistant the way you'd tell a friend what you ate:

- "I had two eggs and a slice of toast for breakfast"
- "How many calories and how much protein do I have left today?"
- "Another slice of the apple pie I baked on Sunday"
- "What did I eat on average this week?"
- A photo of your plate or a nutrition label, with "log this, I ate half"

The assistant works out the portions, looks up anything it doesn't know and
saves calories, protein, carbs and fat to your diary. A card in the chat shows
today's progress, and a full day breakdown lists every meal.

## Screenshots

![Today's calories and macros against the daily goal](https://metri-diary.com/directory/1-card.png)

![Full day breakdown of every meal](https://metri-diary.com/directory/2-full-day.png)

![A logged meal with its nutrients](https://metri-diary.com/directory/3-meal.png)

## Your data

metri connects your AI assistant to metri's own server
(`mcp.metri-diary.com`), where your diary is stored privately in your account.
Payments are handled by Stripe. See our
[Privacy policy](https://metri-diary.com/privacy) and
[Terms](https://metri-diary.com/terms) for details.

Questions? Write to support@metri-diary.com.
