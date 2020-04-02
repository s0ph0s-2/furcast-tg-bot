# FurCast Telegram Bot

* Gates entry to the main FurCast group, to reduce bot activity
* Information and utility functions like `/next [show] [tz]`

Runs as a free Google Cloud Function, as long as the bot isn't an admin in a
busy group, which would cause the webhook to be called for every message or
action and quickly exceed the quota.

## Commands
```
next - See next scheduled show, e.g. "/next fnt" or "/next fc Europe/London"
topic - Request chat topic, e.g. "/topic Not My Cup Of Legs"
```

## How to use

* Generate a random alphanumeric token to be used as an API key for the bot, eg:

```bash
APIKEY=$(tr -cd '[:alnum:]'</dev/urandom|fold -w32|head -n1)
```
* Create a private group (or make your group private), save the invite link for
  `$JOIN_LINK` below
* Create a bot, eg `@furcastbot`, to run this, and save the bot token for
  `$TELEGRAM_TOKEN` below
* Create a channel with the @ you want, eg. `@furcast`, and write a message
  instructing users to talk to the bot for entry to the group.
* If using poll bot, assuming linux account name is `bots`:
  * Copy .env.example to .env and edit
  * `pipenv install`, since apparently `run` doesn't imply it
  * `pipenv run ./main.py` to verify functionality
  * `sudo loginctl enable-linger bots`
  * `ln -s ../../../furcast-tg-bot/furcast-tg-bot.service
    ~/.config/systemd/user/furcast-tg-bot.service`
  * `systemctl --user daemon-reload`
  * `systemctl --user enable --now furcast-tg-bot`

* Otherwise, for webhooks:
  * Set up a
    [new GCP project](https://console.cloud.google.com/projectcreate?previousPage=%2Ffunctions%2Flist)
  * [Enable Cloud Functions](https://console.cloud.google.com/flows/enableapi?apiid=cloudfunctions)
    for the project
  * EITHER create a new function and manually configure it and upload the source, or continue:
  * Set up the Google Cloud SDK's gcloud tool with a configuration named 'xbn':

```bash
gcloud config configurations create xbn
gcloud auth login
gcloud config set project xana-broadcasting
```

* Deploy to GCF with the Google Cloud SDK (Repeat after code/config updates)

```bash
gcloud beta functions deploy furcast-tg-bot --runtime python37 --trigger-http \
    --entry-point webhook --memory 128M --timeout 3s --configuration xbn \
    --set-env-vars "JOIN_LINK=$JOIN_LINK,TELEGRAM_TOKEN=$TELEGRAM_TOKEN,APIKEY=$APIKEY"
```

* From the output, get `httpsTrigger.url`, and set the webhook in the telegram bot:

```bash
curl "https://api.telegram.org/bot$TELEGRAM_TOKEN/setWebhook?url=$TRIGGER_URL&apikey=$APIKEY"
```

### Helpful stuff:
```bash
# See configured webhooks for bot
curl "https://api.telegram.org/bot$TELEGRAM_TOKEN/getWebhookInfo"
# See currently running version
curl "$TRIGGER_URL?apikey=$APIKEY&version"
# Re-deploy with the same settings,
gcloud beta functions deploy furcast-tg-bot --configuration xbn
```
