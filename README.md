# Butterfield

A Slack bot framework using Slack's [Real Time Messaging API](https://api.slack.com/rtm), Python 3 and [asyncio](https://docs.python.org/3/library/asyncio.html).

**Butterfield is unstable and being actively developed. Breaking changes will occur.**

## Echo bot example

This simple bot will listen for *message* events and echo the message to the same channel. 

	import asyncio
	import butterfield
	
	@asyncio.coroutine
	def echo(bot, msg: 'message'):
		yield from bot.post(
			msg['channel'],
			msg['text']
		)
	
	b = butterfield.Bot('slack-bot-key')
	b.listen(echo)

	butterfield.start()

## Running butterfield

This package provides the *butterfield* command line utility. This command takes one argument, a path to a configuration file, and runs the bot as defined.

	$ butterfield mybot-config.json

### Bot configuration files

A butterfield config file contains a JSON array containing objects defining the bots that will be created.

	[
		{
			"key": "i-made-this-key-up",
			"plugins": [
				"butterfield.handlers.devel.log",
				"butterfield.handlers.devel.emoji"
			]
		},
		{
			"key": "also-a-made-up-key",
			"plugins": [
				"butterfield.handlers.devel.log",
			]
		}
	]

The *key* property contains the [Slack bot user](https://api.slack.com/bot-users) key. The *plugins* property is a list of strings that are module paths to event handler plugins.

## Writing your own bot

Bots are created by instantiating an instance with a [Slack bot user](https://api.slack.com/bot-users) key.

	from butterfield import Bot
	
	mybot = Bot('this-is-not-a-real-key')
	
### Receiving Slack events

Message handlers are asyncio [coroutines](https://docs.python.org/3/library/asyncio-task.html#coroutine). When executed, they receive the bot instance that invoked the handler and a copy of the event message from Slack.

	import asyncio

	@asyncio.coroutine
	def console_printer(bot, msg: "message"):
		if 'text' in msg:
			print(msg['text'])

A [parameter annotation](https://www.python.org/dev/peps/pep-3107/) is used on the message parameter to specify the message types the coroutine will receive. This can be a single string or a list of strings to specificy multiple message types. To trigger the handler for all event types, use `"*"` or `butterfield.ALL`. The full list of event types can be found in the [RTM API docs](https://api.slack.com/rtm).

Handlers are added to the bot using the *listen* method. The handler parameter can be either a direct reference to the coroutine or the module path as a string.

	mybot.listen(console_printer)
	mybot.listen("butterfield.handlers.devel.log")

Creating a bot adds it to a master registry of bots in butterfield. Now just start the bots and they'll run... FOREVER.

	butterfield.start()

### Posting to Slack

Handlers can post messages back to Slack using the *post* method on the bot that was passed to it. The first parameter is the id of the channel that will receive the post. The second parameter is the message that will be posted.

	bot.post(channel_id, 'Hi, channel!')

## TODO

* Add HTTP API so messages can be pushed through Butterfield to Slack.
* Listen for meta events and modify local Slack environment appropriately (list of channels, users, etc.).
* EVERYTHING ELSE!