# Channel Message AutoResponder
<details>
  <summary>Table of Contents</summary>

1. [Overview](#overview)
2. [Integrate ChatGPT within Slack](#integrate-chatgpt-within-slack)
3. [Integrate Python within Slack](#integrate-python-within-slack)
4. [Example](#example)

</details>

## Overview
This project is designed to automate response to Slack channel messages and manage client communication effectively. It utilizes the power of OpenAI's ChatGPT model and integrates with Slack to intelligently respond to messages. Moreover, it employs Python for efficient extraction of business emails and automates the process of sending communication emails to clients.
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
Step 4: Writing the Python script
The following is a script which listens for mentions of your bot in a channel, then sends the text of the message to ChatGPT to generate a response, and finally sends the generated message back to the channel.

(Please replace "YOUR_TOKEN" with your actual tokens for the Slack bot token, the Slack app token, and the OpenAI API key.).To run the code, open the chatgpt_slack.py file.:
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

Step 5: Test out(see example)

## Integrate Python within Slack

Step 1: Create a Slack App & Install It

Add the following bot token scopes: channels:history, channels:read, groups:history, im:history, mpim:history, channels:join.

Install the app in your workspace

Step 2: Writing the Python script


(Please replace "bot_token" with your actual tokens for the Slack bot token and "channel_id" with your actual channel id for the Slack channel).To run the code, open the python_slack.py file.
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

### Ask the bot to generate a business email for communication with your client.

Slack Channel: #sales 

Post: 

I’m XX XX from abc Corp. we are interested in your product. We like to get basic info. Then we like to schedule a demo. Kindly reach out to us at oooooo@oooooo.com 

Thanks, XX


https://github.com/zzkeihou/CHATGPT-Email-Automation/assets/116041838/be22d40a-953b-4529-bf41-8dc94c93b5bd

Before you can utilize the bot, make sure to add it to the desired channel via the integration tab. Once the bot is successfully integrated, you can call it to automatically generate business emails.



### Automatically extract business emails and send client communications using Python

To run the code, open the automatically_email.py file.:
``` python
import os
import smtplib
import ssl
from email.message import EmailMessage

# Setup email account
email_sender = 'sender@gmail.com'
email_password = os.getenv('EMAIL_PASSWORD') # It's safer to load sensitive info as environment variable
email_receiver = 'receiver@gmail.com'

# Email subject/body
subject = 'Basic Information'
body = message['text']

# Create EmailMessage object
em = EmailMessage()
em['From'] = email_sender
em['To'] = email_receiver 
em['Subject'] = subject
em.set_content(body)

# Create a secure SSL context
context = ssl.create_default_context()

# Send email
with smtplib.SMTP_SSL('smtp.gmail.com', 465, context=context) as smtp:
    smtp.login(email_sender, email_password)
    smtp.sendmail(email_sender, email_receiver, em.as_string()) 
```
This script can be used to automate the process of sending emails. For instance, it can be used to send automated emails, reminders, or reports. 

![WechatIMG47514](https://github.com/zzkeihou/CHATGPT-Email-Automation/assets/116041838/fa150195-2b65-4422-8628-a8c659d091ea)



