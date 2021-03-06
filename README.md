# no-server-november
Lets have some fun... https://serverless.com/blog/no-server-november-challenge/

# Strategy

* All challenges will be grouped into the same folder, under the same serverless.yml, but will be
  documented separately below.

# Lorem Ipsum

For this challenge, we implement a simple lorem ipsum generator with custom text borrowed straight
from [stranger ipsum].  A single function, using Server-Side Rendering, will provide a
well-formatted HTML page, with custom text generated on every request.  We use [wizipsum] to create
the text, and [he] to encode it for HTML.  You can see the generator in action
[here](https://no-server-november.ulfhedinn.net/lorem_ipsum).

[stranger ipsum]: https://github.com/robertcoopercode/stranger-ipsum/blob/master/src/generator.js
[wizipsum]: https://github.com/wizbii/wizipsum
[he]: https://github.com/mathiasbynens/he

# Dad Bot

Lets twitter up a bot, shall we?  We create a new bot, whose name is [@NDadbot], which is using the
Twitter API to broadcast delightfully terrible jokes from the [dadbot api].  Fortunately for us,
there already exists a [dadbot client] in nodejs, as does the [twitter api], so this primarily
involved creating a new twitter account, promoting that account to a developer by applying, and
provisioning tokens.  We're using [credstash] from my private account to manage tokens, because we
are responsible adults.

[@NDadbot]: https://twitter.com/NDadbot
[credstash]: https://github.com/fugue/credstash
[dadbot api]: https://icanhazdadjoke.com/api
[dadbot client]: https://github.com/jmptr/node-icanhazdadjoke-client
[twitter api]: https://github.com/desmondmorris/node-twitter

# GitHub Check

Alright, so we are tasked with creating a github check.  Mercifully, we already *have* a github repo
to work from — so let us create a check on that (this?).  We've set up a github webhook to point to
`https://no-server-november.ulfhedinn.net/github_check`, and you'll see checks appear when this
piece is complete.  We'll be using the official octokit client to talk to github, and we've created
a [new github application](https://github.com/settings/applications/932381) to authenticate against
this.  This was alleviated using [github-app], which made the whole github app song-and-dance more
palletable.  We use a simple regex to match JIRA tickets, specifically `/[A-Z]{2,}-[0-9]+:/g`, and
you can see the check in action [here](https://github.com/jontg/no-server-november/pull/1).

[official octokit]: https://github.com/octokit/rest.js
[github-app]: https://github.com/probot/github-app

# Cat Meme

A brutally simple website using the giphy API.  We're using the [giphy-api] tool to ease the burden
of querying giphy, and of course credstash for the API token.

[giphy-api]: https://github.com/austinkelleher/giphy-api

# One Direction Facts

okay so if we want to talk Boy Bands, why couldn't it be N'Sync?  Oh well, so we're gonna scrape
[some facts off the internet], use vim to make this less painful:
* `:v/[0-9]\+)/d` (deletes all lines not starting like `3)`)
* `:%s/"/\\"/g` (turn every `"` into `\"`, so this does not mess up JSON later)
* `:%s/^[0-9]\+) \(.*\)$/  "\1",/g` (turn every line into a JSON-like string)
* Add `[` to the beginning of the file, remove the final trailing `,` and add `]` to the end of the
  file.
The final file should look similar to this [cat facts] JSON payload.  Then we follow along with the
[example serverless app], register the app as a simple skill, and voila.  Notably, the Alexa Skill
setup took longer than expected (the setup instructions changed!).

[some facts off the internet]: https://planetradio.co.uk/hits-radio/entertainment/music/101-one-direction-facts/
[cat facts]: https://gist.github.com/tonkku107/c079131c11a8f761a136a4ed305a0d9d
[example serverless app]: https://github.com/serverless/examples/blob/master/aws-node-alexa-skill/

# Image Classifier Bot

Follow the same steps as above; this time the bot is named [@ServerImage].  This time however, we
need to respond to direct messages — that means we're going to need to set up some webhooks using
the [Activity API].  I set up an Account Activity API Sandbox, and added my new Image Classifier app
to a dev environment there.  Also, don't forget to change the permissions of the app to include
read, write *and* direct messages.  Do this before creating access tokens...

In order to register a webhook, we first need to implement a GET end-point to prove to Twitter that
this endpoint is owned by this application; we borrow heavily from [another codebase] for their CRC
implementation for that.  Next time, most of the registration crap should probably be handled
through [twurl]

In order to kick off the process, we manually set up the subscription with a `PATCH` call to the bot
(so we can lazily avoid figuring out how to `curl`).  Notably, this took a *hell* of a long time to
get right.

For the love of... ok, we're only now getting to the actual meat of this problem.  It's garbage
issues like [this crap] that makes the Twitter API such a hassle to deal with.  Also, the
direct_messages event API just doesn't work with the [node-twitter] package (and a bunch of the
other end-points), so we end up using the regular `request` package for like half of these requests.

Useful scripts here:
```bash
serverless invoke local -f image_classifier -d '{"httpMethod": "PATCH", "body": "{\"appId\": \"<APP_ID>\"}"}'
serverless invoke local -f image_classifier -d '{"httpMethod": "DELETE", "body": "{\"appId\": \"<APP_ID>\", \"id\": \"<ID>\"}"}'
```

And in the nodejs console...
```nodejs
request.get({ oauth: { token, token_secret, consumer_key, consumer_secret }, url: "https://api.twitter.com/1.1/account_activity/all/dev/webhooks.json" }, console.log);
```

[@ServerImage]: https://twitter.com/ServerImage
[Activity API]: https://developer.twitter.com/en/docs/accounts-and-users/subscribe-account-activity/guides/getting-started-with-webhooks
[another codebase]: https://itnext.io/serverless-twitter-bot-with-google-cloud-35d370676f7
[twurl]: https://github.com/twitter/twurl
[this crap]: https://twittercommunity.com/t/errors-message-could-not-authenticate-you-code-32/1223/13

The web-hook itself...
```bash
Payload { id: '1064008442954706944',
  url: 'https://no-server-november.ulfhedinn.net/imagebot',
  valid: true,
  created_timestamp: '2018-11-18 04:12:36 +0000' }
```

# Twilio Integration - Wake Me Up!

Wake me up every morning at 8am PDT (4pm UTC).  Super simple, deliciously straightforward, and now I
have a new phone number I can text from!

Interestingly enough, with this addition we see a significant increase in memory pressure, so we are
now deploying with more memory:
```bash
node --max-old-space-size=12000 ./node_modules/serverless/bin/serverless deploy
```
