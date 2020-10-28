# Pancake Studio

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/stevengill/pancake-studio/tree/main)

A server that receives json payloads, flattens the keys (brings all keys to top level), and sends this new payload to a webhook that triggers a workflow in Slack.

Slack allows workflows to be triggered via webhooks in workflow builder. These workflows only support the extraction of variables at the top level of the payload structure. To handle nested payloads (for example payloads GitHub sends via webhooks), they need to be flattened before being sent to slack's workflow builder webhook url. 

## Usage

1) In Slack, go to tools -> workflow builder and create a new workflow. Choose `webhook` for the type of workflow you are creating. 
2) Add a second step (such as send a message to a person or channel)
3) Publish the workflow (this will allow the workflow to generate a webhook url). The webhook url should look something like `https://hooks.slack.com/workflows/***`
4) Now, we need to manually edit the host portion of the url to match our server. If you are using https://pancake.studio, you would change the url to `https://pancake.studio/workflows/***`. You can also go to https://pancake.studio and enter your slack webhook url into the input field to have us do the conversion for you.
5) Use the new url (`https://pancake.studio/workflows/***`) as the payload url when setting up webhooks with third parties (example: if you create a webhook for a repo on Github, use this url as the payload url)
6) Update the first step in your new workflow to add variables which can be used in subsequent steps in the workflow. Variable names must match the key name it wants to pull data from

Pancake studio will receive the payload from github, flatten it, change the hostname back to `https://hooks.slack.com/` and then send the flattened payload to the original hooks url.

Pancake studio is stateless. It does not store the content of the payloads or the webhook urls.

## Flat Payload Structure

Under the hood, we are using the [`flat` npm package](https://www.npmjs.com/package/flat).

```
var flatten = require('flat')
 
const flatJson = flatten({
    key1: {
        keyA: 'valueI'
    },
    key2: {
        keyB: 'valueII'
    },
    key3: { a: { b: { c: 2 } } }
})

console.log(flatJson)
// {
//   'key1.keyA': 'valueI',
//   'key2.keyB': 'valueII',
//   'key3.a.b.c': 2
// }
```

## Learnings

- Michael and Steve learned about Workflow Builder Webhooks.
- Michael learned about LiveShare in Visual Code.
- Workflow Builder `Received a webhook request that was missing a required field` error when a configured variable is missing from the incoming payload. Optional fields would allow the workflow to be more versatile.
- Workflow Builder `invalid_field` error if incoming value is not a string.
- Workflow Builder / Slackbot only unfurls a URL once, but simple workflows may want to leverage unfurls to create beautiful messages with ease.
- Workflow Builder does not support the use of variables in hyperlinks.
- Workflow Builder does not support the use of block kit elements.

## Roadmap

- Help user by previewing flattened keys and copying them to the clipboard for use in Workflow Builder. Build UI to paste source JSON payload and preview it from the landing page.
