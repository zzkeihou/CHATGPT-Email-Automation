# Channel Message AutoResponder
<details>
  <summary>Table of Contents</summary>

1. [Overview](#overview)
2. [Integrate ChatGPT within Slack](#integrate-chatgpt-within-slack)
3. [Integrate Python within Slack](#integrate-python-within-slack)
4. [Example](#example)

</details>

## Overview
## Integrate ChatGPT within Slack
Step 1: Create an app with Slack and obtain tokens

- Log in to your Slack workspace.
- Visit the [Slack API website.](https://api.slack.com/)
- Click on "Create an app" and select "From scratch".
- Give your app a name and select your Slack workspace.
- In Basic information > Add features and functionality > Permissions, add the following Bot Token Scopes: app_mentions:read, channels:history, channels:read, chat:write.
- In Settings, click on "Socket Mode", enable it, and give the token a name. Copy the Slack Bot App Token (starts with xapp).
- In Basic information > Add features and functionality > Event Subscriptions, enable this and select "app_mention" under "Subscribe to bot events". Save changes.
- Go to the "OAuth & Permissions" section and install your app to your workspace. Copy the Slack Bot Token (starts with xoxb).

Step 2: Obtain the OpenAI API key

Visit the [OpenAI website.](https://platform.openai.com/account/api-keys) to get API key

Step 3: Install necessary dependencies

The following Python libraries are required for this example: slack_sdk (for Slack integration)and openai (for ChatGPT integration). You can install these dependencies by running the following command in your terminal:
```python
pip install openai
pip install slack-bolt
pip install slack-sdk
```
Step 4: Writing the Code
The following is a script which listens for mentions of your bot in a channel, then sends the text of the message to ChatGPT to generate a response, and finally sends the generated message back to the channel:
```python
SLACK_BOT_TOKEN = "YOUR_TOKEN"
SLACK_APP_TOKEN = "YOUR_TOKEN"
OPENAI_API_KEY  = "YOUR_TOKEN"

import os
import openai
from slack_bolt.adapter.socket_mode import SocketModeHandler
from slack import WebClient
from slack_bolt import App

# Event API & Web API
app = App(token=SLACK_BOT_TOKEN) 
client = WebClient(SLACK_BOT_TOKEN)

# This gets activated when the bot is tagged in a channel    
@app.event("app_mention")
def handle_message_events(body, logger):
    # Log message
    print(str(body["event"]["text"]).split(">")[1])
    
    # Create prompt for ChatGPT
    prompt = str(body["event"]["text"]).split(">")[1]
    
    # Let thre user know that we are busy with the request 
    response = client.chat_postMessage(channel=body["event"]["channel"], 
                                       thread_ts=body["event"]["event_ts"],
                                       text=f"Hello from your bot! :robot_face: \nThanks for your request, I'm on it!")
    
    # Check ChatGPT
    openai.api_key = OPENAI_API_KEY
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=1024,
        n=1,
        stop=None,
        temperature=0.5).choices[0].text
    
    
    # Reply to thread 
    response = client.chat_postMessage(channel=body["event"]["channel"], 
                                       thread_ts=body["event"]["event_ts"],
                                       text=f"Here you go: \n{response}")

if __name__ == "__main__":
    SocketModeHandler(app, SLACK_APP_TOKEN).start()
```

## Integrate Python within Slack

Step 1: Create a Slack App & Install It

Add the following bot token scopes: channels:history, channels:read, groups:history, im:history, mpim:history, channels:join.

Install the app in your workspace

Step 2: Writing the Code
```python
import slack_sdk

# Your bot user token, which you saved when you installed the app
bot_token = 'xoxb-XXXX-XXXX'

# Initialize a Web API client
slack_client = slack_sdk.WebClient(token=bot_token)

# Specify the ID of the channel you want to grab text from
channel_id = 'CXXXXXXXX'

# Call the conversations.history method using the WebClient
result = slack_client.conversations_history(channel=channel_id)

# Get only messages from the response (excluding other types like user joins, etc.)
messages = [message for message in result.data['messages'] if message.get('type') == 'message' and 'subtype' not in message]

# Print each message
for message in messages:
    print(message['text'])

```
This script connects to the Slack API, grabs the history of a specified channel, and prints out each message.

Once you have your Slack App installed in your workspace and the Python script ready, you can run your script to start collecting messages from the specified channel.


## Example
Slack Channel: #sales 

Post: 

I’m XX XX from abc Corp. we are interested in your product. We like to get basic info. Then we like to schedule a demo. Kindly reach out to us at oooooo@oooooo.com 

Thanks, XX
