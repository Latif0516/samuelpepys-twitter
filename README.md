# Samuel Pepys Twitter

Python script for posting tweets at specific times. Used for the [@samuelpepys](http://twitter.com/samuelpepys) Twitter account.

Uses Redis to store the time the script last ran.


## Tweet files

There are files of tweets in dated yearly directories and monthly files, eg, `tweets/1660/01.txt`. Most recent tweets at the top of each file. The years should be `YEARS_AHEAD` years ago. eg, if `YEARS_AHEAD` is set to `353`, then tweets in the `1660` directory will be used in 2013.

You could set `YEARS_AHEAD` to `0` and then tweets will be sent on the day they're dated for. Possibly a more useful and common requirement!

Tweets should be in time order, with most recent (ie, probably in the future) first. Each tweet should be on a single line, preceded by its date and time, eg:

    1660-01-02 11:20 Great talk that many places have declared for a free Parliament; it is believed they will be forced to fill up the House with old members. 

Any lines that aren't of that format (ie, with that datetime format at the start, followed by a tweet) will be ignored. So feel free to comment out any tweets to be ignored by prepending them with a different character, and leave blank lines to make reading easier.

The script doesn't check for length of tweet, so any tweets longer than 140 characters will be submitted and, I expect, rejected. Having a line like this:

    YYYY-MM-DD HH:MM 12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890

at the start of each file, and setting your window to be exactly as wide as the line, makes it easy to get tweet length correct.


## What gets tweeted

The script looks through all the tweets and grabs any whose time (adjusted with `YEARS_AHEAD`) fulfills all three conditions:

1. It is earlier than *now* (ie, not in the future).
2. It is since the script last ran.
3. It is also since `MAX_TIME_WINDOW` minutes ago.

The last condition is to catch the following scenario: Something goes wrong with the server or script and it isn't run successfully for, say, 12 hours. The next time it's run it would instantly tweet all tweets set for the last 12 hours. Assuming we don't want this, set `MAX_TIME_WINDOW` to how many minutes back we'd want to check.

If you *would* want to tweet all the past 12 hours worth of tweets, set `MAX_TIME_WINDOW` to a very large number.

Any tweets that match those conditions will be tweeted a couple of seconds apart, in the order their datetimes are in.


## Configuration 

Configuration can either be set in a config file or in environment settings. If `config.cfg` is present, that is used, otherwise environment settings. Copy `config_example.cfg` to `config.cfg` to use that.

If using environment settings, they are listed below. The Twitter OAuth settings are required, the rest are optional. (Although if `REDIS_URL` or its config file equivalents are left out, the script tries to use a local, un-password-protected, database.)

    # OAuth settings from your Twitter app at https://dev.twitter.com/apps/
    TWITTER_CONSUMER_KEY=YOURCONSUMERKEY
    TWITTER_CONSUMER_SECRET=YOURCONSUMERSECRET
    TWITTER_ACCESS_TOKEN=YOURACCESSTOKEN
    TWITTER_ACCESS_TOKEN_SECRET=YOURACCESSTOKENSECRET

    # Output extra debug text while running? 1 or 0 (Default).
    VERBOSE=1

    # How many years ahead of the dated tweets are we? (Default: 0)
    YEARS_AHEAD=353

    # Regardless of when the script last ran, never send tweets that are older than this many minutes. (Default: 20)
    MAX_TIME_WINDOW=20

    # Which timezone are the times of the tweets in? (Default: 'Europe/London')
    TIMEZONE='Europe/London'

	# Example value:
	REDIS_URL = redis://rediscloud:sjPfErI4xocRopQW@pub-redis-18850.us-east-1-2.1.ec2.garantiadata.com:18850

See [Wikipedia's list](http://en.wikipedia.org/wiki/List_of_tz_database_time_zones) of TZ timezone strings for the `TIMEZONE` setting.


## Local setup

Use [pip](http://www.pip-installer.org/) to install required packages by doing:

    $ pip install -r requirements.txt

Set up config values as above, either via a config file (probably best) or environment settings.

Then just run the script:

    $ python tweeter.py

That will send a tweet if there is one with an appropriate date and time.


## Heroku setup

Set up a new Heroku app.

Set Heroku environment variables for all the environment variables, eg:

    $ heroku config:set MAX_TIME_WINDOW=20

Add a Redis database, eg:

	$ h addons:add rediscloud

Copy the Redis add-on's URL to the `REDIS_URL` environment variable:

	$ h config:get REDIS_CLOUD_URL
	[ copy that value ]
	$ h config:set REDIS_URL=redis://rediscloud:...

Push all the code  and tweets to your Heroku app:

    $ git push heroku master

There you go. I think that's it... The `Procfile` specifies a `clock` process
that runs `clock.py`. This sets up a scheduler to run the code in `tweeter.py`
every minute.


## Running on Heroku with Scheduler

Previously we didn't use the clock process but ran this using the Scheduler. The downside is that it can only run up to once every 10 minutes. 

To do it that way, remove the Procfile and push the code to Heroku.

Add the free [Heroku Scheduler](https://addons.heroku.com/scheduler) to your app:

    $ heroku addons:add scheduler:standard

Have it run `python tweeter.py` every 10 minutes.

