# Slackrock
Converse with your favorite [LLMs](https://en.wikipedia.org/wiki/Large_language_model) without ever leaving Slack! **Slackrock** is a conversational AI assistant Slack bot powered by [Amazon Bedrock](https://aws.amazon.com/bedrock/) & your favorite cutting-edge frontier models. The project is focused on cost efficiency & simplicity.

<video src="docs/video/slackrock.mp4" controls>
Your browser does not support the video tag.
</video>

## ✨ Features

💬 Uses Amazon Bedrock [Converse API](https://aws.amazon.com/about-aws/whats-new/2024/05/amazon-bedrock-new-converse-api/) for multi-turn conversations, allowing models to retain context of the interaction over a long period of time. Come back to a thread days later and pick up where you left off!

🎛️ **Customization**: Quickly evaluate and experiment with [supported models](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html#conversation-inference-supported-models-features) including Amazon Titan, Anthropic Claude, Mistral AI, Cohere Command, and Meta Llama. Using [Slack Bolt for Python](https://slack.dev/bolt-python/tutorial/getting-started) makes it easy to extend the Lambda code to create your own event handlers or Slack Slash Commands.

🧮 **Cost Efficiency**: Only pay for what you use with Amazon Bedrock's pay-as-you-go pricing.

- Costs automatically decrease during decreased periods of usage ie. holidays and weekends.
- Serverless compute requires minimal operational overhead and uses efficient [Graviton2](https://aws.amazon.com/ec2/graviton/) processors.
- Works with free Slack workspace, pro/enterprise plans not required.

😊 **Simplicity**: Get your own bot up and running quickly with just a few `sam` commands.

# 🏛️ Architecture
![Alt Text](docs/images/slackrock_architecture.png)


# 🏗️ Getting started
## 📋 Prerequisites
- AWS account with an IAM User or Role with necessary permissions to create and manage all resources required for the solution. See: [How do I create and activate a new AWS account?](https://repost.aws/knowledge-center/create-and-activate-aws-account)
- A Slack workspace where you have permission to create and install apps. A pro Slack plan is **not** required.
- [aws-sam-cli](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
- [aws-cli](https://github.com/aws/aws-cli)
- Python 3.12


## 🚀 Deploy Slackrock
Let's get started!

1. [Request access](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html) to desired Amazon Bedrock models. The default model used in this solution is `anthropic.claude-3-sonnet-20240229-v1:0`. Feel free to request as many or as few as you like, the process is automated and models become available immediately.

1. Create a Slack app (https://api.slack.com/apps) from manifest using the `slack-app-manifest.yaml` file provided in this project.

    <video src="docs/video/create_slack_app.mp4" controls>
    Your browser does not support the video tag.
    </video>

1. Once the app is created, install the app in your Slack workspace.

    ![GitHub Logo](/docs/images/install_slack_app.png)

1. Gather Slack secrets
    1. Navigate to 'Basic Information' and note the `Signing Secret`
    1. Navigate to 'OAuth & Permissions' and note the `Bot User OAuth Token`

1. Store the secrets in AWS Secrets Manager using the `aws` cli:
    ```bash
    aws secretsmanager create-secret \
    --name "slackrock" \
    --secret-string '{
        "SLACK_BOT_TOKEN": "xoxb-XXXXXXX-XXXXXXXXXXXXX-XXXXXXXXXXXXXXXXX",
        "SLACK_SIGNING_SECRET": "XXXXXXXXXXXXXXXXXXXXXXX"
    }'
    ```

1. Next, build & deploy the app to your AWS account. Ensure AWS access credentials are loaded into your current environment. (Pro-tip: https://www.granted.dev/)

    1. Run `sam build`

        Note: any changes you make to template.yaml or the lambda code will require re-running `sam build`.

    1. Run `sam deploy --guided`

        ❗ Answer `y` to this prompt, see FAQ for further details: `Slackrock Function Url has no authentication. Is this okay? [y/N]`

1. After the deployment is completed, note the CloudFormation stack output `SlackrockUrl` and head back to your Slack app developer page.

    ![sam_template_output_url.png](/docs/images/sam_template_output_url.png)

1. Navigate to `Event Subscriptions` and Enable Events

1. Enter your `SlackAppUrl` into Request URL field and confirm the endpoint is verified.

    ![slack_enable_events.png](/docs/images/slack_enable_events.png)

1. Add the following bot event subscriptions and click the green save changes button at the bottom.

- [app_mention](https://api.slack.com/events/app_mention)
- [message.channels](https://api.slack.com/events/message.channels)
- [app_home_opened](https://api.slack.com/events/app_home_opened) (optional)

    ![slack_bot_events.png](/docs/images/slack_bot_events.png)


## 🎉 Say Hey!
It's time to test your bot! Review Cloudwatch Logs if you get any errors or the bot is unresponsive.

1. Invite the app to a channel
1. Mention `@Slackrock` in a message, maybe ask about the history of Bundnerkase cheese 🧀, or anything else you'd like to know!

## Up next
- Support direct message interactions
- Multi-modal support, process image & document attachments, etc.
- User interfaces to easily switch the model without leaving Slack
    - Slack app home page drop-down selection.
    - A slash-command to changes the currently active model
    - Potentially make this preference customizable per-user in slack workspace. Leverage dynamodb (`AWS::Serverless::SimpleTable`)
- Support sam integration with terraform
- Pull from secrets manager directly via python boto3? I'm worried this approach will increase execution time & add additional latency, but it is more secure. Will test

# FAQ

## How do I make real-time changes to my code?
To iterate on the code locally, use `sam sync`. The command below starts a process that watches your local application for changes and automatically syncs them to the AWS Cloud.

```
sam sync --watch --stack-name lanecor-slackapp
```

## How do I interact with different Bedrock models?
Update the `BEDROCK_MODEL_ID` envirohnment variable in `template.yaml` to one of the [supported models](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html#conversation-inference-supported-models-features) and run `sam deploy`.

Alternatively, you can also update the Lambda function environment variables directly via the AWS console or CLI.

## How do I find the Model IDs?
The complete list of currently available foundation model IDs is [available here](https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids.html). However, keep in mind that not all available Bedrock models are supported by the Converse API.

## Which models are supported?
The Bedrock [Converse API](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html) supports Amazon Bedrock models and model features [listed here](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference.html#conversation-inference-supported-models-features).

## Can I use Anthropic Claude 3 Opus?
Currently this model is only available in US West (Oregon, us-west-2). Change the `region` parameter in `samconfig.toml` and try it out!

## Can I customize the personality or overall tone of the responses?
Absolutely! This is a very useful and often fun feature to experiment with. Simply adjust the [system prompt](https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html#bedrock-runtime_Converse-request-system) to the converse api call to gain more control over how Bedrock models answer your questions. See: [Getting started with the Amazon Bedrock Converse API](https://community.aws/content/2hHgVE7Lz6Jj1vFv39zSzzlCilG/getting-started-with-the-amazon-bedrock-converse-api#setting-a-system-prompt). 

The default Slackrock system promopt simply instructs the models to use Slack compatible Markdown formatting. Anthropic has a great page to [learn more about system prompts](https://docs.anthropic.com/en/docs/system-prompts).

Here is an example that will cause your app to respond to all queries

```python
response = client.converse(
    modelId=modelId,
    messages=messages,
    system=[
        {"text": "Respond like a pirate."}
    ],
)
```

Example response:

![pirate_system_prompt.png.png](/docs/images/pirate_system_prompt.png)


Another approach is to set the stage and ask the model to respond in a certain way in your first message, see the demo video for an example of this.

## Can I run the solution locally?
Yes! Use [ngrok](https://ngrok.com/) and modify the sam template to use API gateway. Currently the `sam local start-api` command [does not support](https://github.com/aws/aws-sam-cli/issues/4299) Lambda Function URLs.

1. Install ngrok and create a free static domain.
1. Run `ngrok http --domain=free-custom-domain.ngrok-free.app 3000`. This will be your Slack App event URL.
1. Add the `Events` property to `AWS::Serverless::Function`:
    ```
    Resources:
    BedrockSlackApp:
        Type: AWS::Serverless::Function
        Properties:
        Events:
            ApiEvent:
            Type: Api
            Properties:
                Path: /slack/events
                Method: post
    ```
1. Run `sam local start-api --env-vars env.json --warm-containers eager`

## Why use AWS Lambda provisioned concurrency?
By default, provisioned concurrency is not enabled in the SAM template to keep costs as low as possible. But you may want to enable it depending on your performance requirements. 

Using Lambda provisioned concurrency for a Slack app endpoint function is beneficial for three main reasons in this case:

1. Provisioned concurrency reduces cold start latencies by keeping a specified number of function instances warm and ready to respond, eliminating delays caused by initializing the execution environment.
1. Slack requires a response within 3 seconds or will resend messages and cause unecessary duplicate Bedrock model invocations.
1. Fast response times are essential for providing the best user experience in realtime chat applications like Slack apps. Provisioned concurrency minimizes latency and delivers a smoother user experience.

By allocating provisioned concurrency, you ensure that your Slack app endpoint function is always ready to handle incoming requests promptly, meeting Slack's strict latency requirements and providing a reliable and responsive user experience.

## Why is there no authentication on the Lambda Function URL?
AWS Lambda function URLs are publicly accessible by default, which may seem concerning from a security perspective. However, Slack apps require publicly accessible endpoints to receive events and commands from Slack's servers.

The `slack_bolt` Python library used in this project uses the `signing_secret` and `token` to secure your Lambda function URL endpoint by verifying the authenticity and integrity of the incoming requests from Slack.

Here's how the `slack_bolt` library utilizes these secrets:

1. `SLACK_SIGNING_SECRET`:
    - When you create a Slack app, Slack generates a unique `SLACK_SIGNING_SECRET` for your app. This secret is used to verify the signature of the requests sent by Slack to your app.
    - When Slack sends a request to your Lambda function URL, it includes a special `X-Slack-Signature` header in the request. This header contains a hash signature that is computed using your app's `SLACK_SIGNING_SECRET` and the request payload.
    - The `slack_bolt` library automatically verifies the `X-Slack-Signature` header by recomputing the signature using the `SLACK_SIGNING_SECRET` and comparing it with the signature sent by Slack. If the signatures match, it confirms that the request is genuinely coming from Slack and hasn't been tampered with.
    - If the signature verification fails, the `slack_bolt` library will reject the request, preventing unauthorized access to your Lambda function.

2. `SLACK_BOT_TOKEN`:
    - The `SLACK_BOT_TOKEN` is an access token that is used to authenticate your app when making API calls to Slack.
    - When you initialize the `App` instance in your Lambda function with the `token` parameter set to your `SLACK_BOT_TOKEN`, the `slack_bolt` library uses this token to authenticate requests made to the Slack API on behalf of your app.
    - The `SLACK_BOT_TOKEN` ensures that your app has the necessary permissions to perform actions and retrieve information from Slack.
