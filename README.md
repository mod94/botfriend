# Botfriend

Botfriend is a Python framework for managing a lot of creative bots
that post to a number of different services.

I think the primary features of Botfriend are these:

* Minimal Python coding -- just write the interesting part of your bot.
* Simple configuration based on YAML.
* Easy scheduling of posts.
* Each bot can post to Twitter and/or Mastodon. (Tumblr support is planned.)
* Built-in access to art supplies through [Olipy](https://github.com/leonardr/olipy).

Botfriend is a Python library that runs on a server. If you're not
comfortable with setting up a cron job, or writing Python code, I
recommend you check out [Cheap Bots, Done
Quick](http://cheapbotsdonequick.com/) instead as a simpler way to
express your creativity.

# The Story

I wrote Botfriend to manage about [thirty different Twitter
bots](https://www.crummy.com/features/) that I created. I found myself
constantly copying and pasting, writing the same code over and
over. Every bot does something different, but they all have certain
basic needs: connecting to various services and APIs, deciding when to
post something, managing backlogs, and so on. There's no reason each
bot needs its own code for this. The only part of a bot that needs
different _code_ is the creative bit.

My other big problem was, I've come to dislike Twitter. It's a great
platform for creative bots, but every time I created a bot, I felt
guilty about increasing the value of an platform I think is making the
world worse. (Also, Twitter has started suspending my bots for no reason.)

I didn't want to give up my botmaking hobby, so I started
investigating the world of [Mastodon](https://joinmastodon.org/)
bots. This created another problem: it's a big pain to rewrite thirty
bots to post to a different service. If I was going to do that much
work, I wanted the end product to be a reusable library that could
save everyone time.

So I went through my thirty bots, rewrote everything, and moved all of
the reusable code into Botfriend. Now my bots are a lot smaller and
easier to manage. All of the tedious code is in one place, and I can
focus on the fun part of bot-writing. My bots can still post to
Twitter if I want them to, but they also post to Mastodon with no
extra code.

If you want to save code on your own bot projects, port your bots from
Twitter to Mastodon, or just expand the reach of your bots, I hope
you'll consider Botfriend.

# Setup

I recommend you run Botfriend in a Python virtual environment. Here's
how to create a virtual environment called `env` and install Botfriend
into it.

```
$ virtualenv env
$ source env/bin/activate
$ pip install botfriend
```

You'll interact with Botfriend exclusively through command-line
scripts. From this point on I'll be giving lots of example command
lines. All of my examples assume you've entered the Botfriend virtual
environment by running this command beforehand:

```
$ source env/bin/activate
```

# A simple example: Number Jokes

By default, Botfriend expects you to put the source code for the bots
in a directory `bots/`, located in the same directory as your virtual
environment. So if your virtual environment is located in
`/home/myusername/botfriend/env`, Botfriend will expect your bots to
live underneath `/home/myusername/botfriend/bots`.

The Botfriend database itself will be stored in the bot directory as
`botfriend.sqlite`.

If you want to store your Botfriend data somewhere other than `bots/`,
every Botfriend script takes a `--config` argument that points to your
bot directory. But most of the time, `bots/` is fine.

Each individual bot will live in a subdirectory of your bot directory,
named after the bot. Let's get started with a simple example, called
`number-jokes`.

```
$ mkdir bots
$ mkdir bots/number-jokes
```

Each bot needs to contain two special files: `__init__.py` for source
code and `bot.yaml` for configuration.

## `__init__.py`: Coming up with the joke

Imagine walking up to to a comedian and saying "Tell me a joke!" A
human comedian probably won't appreciate it, but this is what bots
live for. For a Botfriend bot, `__init__.py` is where the comedian
comes up with their jokes.

To get started, we'll make a simple bot that makes up observational
humor about numbers.

To get started, open up `bots/number-jokes/__init__.py` in a text
editor and write this in there:

```
import random
from botfriend.bot import TextGeneratorBot

class NumberJokes(TextGeneratorBot):

    def generate_text(self):
        """Tell a joke about numbers."""
        num = random.randint(1,10)
        arguments = dict(
            num=num,
            plus_1=num+1,
            plus_3=num+3
        )
        setup = "Why is %(num)d afraid of %(plus_1)d? "
        punchline = "Because %(plus_1)d ate %(plus_3)d!"
        return (setup + punchline) % arguments

Bot = NumberJokes
```

Botfriend provides a lot of utilities to help you write a good
`__init__.py`, but there's only one hard-and-fast rule: by the end of
the file, you have to have a class called `Bot`. The Botfriend scripts
are going to load your `Bot` class, instantiate it, and use it to
do... whatever the bot does.

Some bots do a lot of work to come up with a single "joke". They might
draw pictures, do database queries, make API calls, all sorts of
complicated things. As befits an example, `NumberJokes` here does
almost no work. It just picks a random number and puts it into a string.

## `bot.yaml`: Telling the joke

Comedians think of jokes all the time, but if no one ever hears the
joke, what's the point? The `bot.yaml` file explains how a Botfriend
bot should _tell_ its jokes.

Open up the file `bots/number-jokes/bot.yaml` and write this in there:

```
name: "Number Jokes"
schedule: 60
publish:
    file:
      filename: "number-jokes.txt"
```

Like `__init__.py`, `bot.yaml` can get really complicated, but most of
the time it's pretty simple. This file is saying:

* The name of the bot is "Number Jokes".

* The bot should 'tell a joke' once an hour.

* This bot tells jokes by writting them to the file
  `number-jokes.txt`. (This is inside the bot directory, so it's going
  to be in `bots/number-jokes/number-jokes.txt`.)

Now you're ready to make your bot tell some jokes, using some basic
Botfriend scripts.

# The basic scripts

## `botfriend.post`

The `botfriend.post` script makes each of your bots come up with a
joke and tell it. Run it now:

```
$ botfriend.post
# LOG 2019-01-20 | Number Jokes | file | Published 2019-01-20 | Why is 4 afraid of 5… 
```

Now look at the file you configured in `bot.yaml`. You told Number
Jokes to post its jokes to `bots/number-jokes/number-jokes.txt`. That file
didn't exist before you ran `botfriend.post`, but now it does exist,
and it's got a joke in it:

```
$ cat bots/number-jokes/number-jokes.txt
2019-01-20 10:23:44 | Why is 4 afraid of 5? Because 5 ate 7!
```

Hilarious, right? You'll be running this script a lot, probably as
part of an automated process. On my site run `botfriend.post` every
five minutes. If your bot isn't scheduled to tell a joke,
`botfriend.post` will do nothing. Run it again now -- nothing will
happen.

```
$ botfriend.post
```

Number Jokes just told a joke, and (as you told it in `bot.yaml`) it's
only supposed to tell one joke an hour.

## Running a script on just one bot

By specifying a directory name on the command line, you can make
`botfriend.post` (and most other Botfriend scripts) operate on just
one bot, not all of your bots. Right now, it doesn't make a
difference, because you only have one bot, but here's how to do it:

```
$ botfriend.post number-jokes
```

You can use `--force` to make a bot tell a joke even if its schedule
wouldn't normally allow it.

```
$ botfriend.post number-jokes --force
# LOG 2019-01-20 | Number Jokes | file | Published 2019-01-20 | Why is 9 afraid of 10… 
```

Now `bots/number-jokes/number-jokes.txt` contains two jokes.

```
$ cat bots/number-jokes/number-jokes.txt
2019-01-20 10:23:44 | Why is 4 afraid of 5? Because 5 ate 7!
2019-01-20 10:26:12 | Why is 9 afraid of 10? Because 10 ate 11!
```

## `botfriend.dashboard`

This script is good for getting an overview of your bots. It shows
what they've been up to lately and when they're scheduled to run again.

```
$ botfriend.dashboard number-jokes
# Number Jokes | Most recent post: Why is 9 afraid of 10? Because 10 ate 12!
# Number Jokes | file posted 0m ago (2018-01-20 10:26:12)
# Number Jokes | Next post in 59m
```

## `botfriend.bots`

If you have a lot of bots, it can be annoying to remember all their
names. The `botfriend.bots` script just lists all the bots known to
Botfriend.

```
$ botfriend.bots
# number-jokes
```

## `botfriend.test.stress`

It's difficult to test a bot that does random things. You might have a
bug that makes the bot crash only one time in a thousand. Or your bot
might never crash, but sometimes take a long time to run.

This is why we have the `botfriend.test.stress` script, which asks a
bot to come up with ten thousand jokes in a row. The jokes aren't
published anywhere; the goal is just to give a good test of all the
possible cases that might happen inside your bot.

Since Number Jokes is really simple, it can generate ten thousand jokes
with no problem, although some of them are repeats:

```
$ botfriend.test.stress number-jokes
# Why is 2 afraid of 3? Because 3 ate 5!
# Why is 7 afraid of 8? Because 8 ate 10!
# Why is 1 afraid of 2? Because 2 ate 4!
# Why is 4 afraid of 5? Because 5 ate 7!
# Why is 4 afraid of 5? Because 5 ate 7!
# Why is 7 afraid of 8? Because 8 ate 10!
# ... etc. etc. ...
```

If you've got a complicated bot, it can be a good idea to run
`botfriend.test.stress` on it a couple of times before using it for real.

# How to publish

There are a few more interesting features of Botfriend, but let's take
a minute to talk about the boring features. It's easy to make a bot
write its posts to a file, but nobody's going to see that. What you
really need is to get some Twitter or Mastodon credentials.

TODO: link to instructions on how to actually do this.

In your bot's `bot.yaml` file, add your credentials to the `publish`
configuration setting. This will give your bot additional ways to
publish its posts to the Internet.

Here's an example. This is what the configuration for Number Jokes
would look like if it had Twitter and Mastodon connections set up, in
addition to writing everything to a file.

```
name: Number Jokes
schedule: 60
publish:
  file:
    filename: number-jokes.txt
  mastodon:
    api_base_url: 'https://botsin.space/'
    client_id: cc13bf3de67fb399475c315e4a9bf5dd4dfb7ea0f3a521fca72a9c8bf02075ab
    client_secret: 0946d2634d7b6aa1ea93af4b183fccf14e9df2e2b55db8fcdb0c8a5f267ff312
    access_token: c76eb18c5c0dc7c1fe09b53ac175b3b9ed081b0e43ea4d60e94ee721b83c1eda
  twitter:
    consumer_key: t7CfbbNLB3jfoAKI
    consumer_secret: 2teOyqqgFpFpytFanuOXzfjvR3vEmYH3
    access_token: 3341062559-SbUlEDFCDn6k6vHHDWGwqlK0wyZ0fKRegaZMyS9lwBa4L5VXY5fdl
    access_token_secret: ALc2CPkkrSBf33swYluxEgdC0GNueQK3x6D4pEr8GGDpqrmed
```

(These credentials won't work -- I made them up to resemble real
Twitter and Mastodon credentials.)

## `botfriend.test.publisher`: test your publishing credentials

The `botfriend.test.publisher` script tries out all of your bots'
publishing credentials to make sure they work. If a bot is having
trouble posting, the problem will show up here.

For every bot with a publishing technique that works, you'll get a
line that starts with `GOOD`. For every publishing technique that's
broken, you'll get a line that starts with `FAIL`.

In this example, writing to a file works fine, but since the Twitter
and Mastodon credentials are made up, Twitter and Mastodon won't
actually accept them.

```
$ botfriend.test.publisher
# GOOD Number Jokes file
# FAIL Number Jokes twitter: [{u'message': u'Bad Authentication data.', u'code': 215}]
# FAIL Number Jokes mastodon: {u'error': u'The access token is invalid'}
```

## Publish to a file

This is the simplest publication technique, and it's really only good
for testing and for keeping a log. The `file` publisher takes one
configuration setting: `filename`, the name of the file to write to.

```
publish:
    file:
        filename: "anniversary.txt"
```

## Twitter

To get your bot on Twitter, you need to create a Twitter account for
the bot, and then get four different values: `consumer_key`,
`consumer_secret`, `access_token` and `access_token_secret`. These
four values, when inserted into `bot.yaml`, give you the ability to
post to a specific Twitter account using the Twitter API.

There's help on the web for getting these four values;  the
[Build-a-Bot
Workshop](https://spinecone.gitbooks.io/build-a-bot-workshop/content/set_up_twitter.html)
has some good instructions.

## Mastodon

To connect your bot to Mastodon, you create a Mastodon account for the
bot, and then get four values. First, `api_base_url`-- this is easy,
it's just the URL to the Mastodon instance that hosts the account you
created. I like to use [botsin.space](https://botsin.space/), a
Mastodon instance created especially for bots.

Then you need to get `client_id`, `client_secret`, and
`access_token`. Allison Parrish has [a useful
tutorial](https://gist.github.com/aparrish/661fca5ce7b4882a8c6823db12d42d26)
for getting these three values for your Mastodon account.

Once you have these four values, put them into `bot.yaml`, and your bot
will be able to post to your Mastodon account.

Okay, now back to the cool bots you can write with Botfriend.

# Bots that keep a backlog: Boat Names

Some comedians can come up with original content on the fly, over and
over again again. Others keep a Private Joke File: a list of jokes
assembled ahead of time which they can dip into as necessary.

Instead of putting a bunch of generator code in a Botfriend bot, you
can generate a _backlog_ of posts, however you like -- you can even
write the backlog yourself in a text editor. It's easy to create a bot
that simply posts items from its backlog, in order, one at a time.

Let's make a simple backlog bot that posts interesting names for
boats. (This is exactly how my real bot [Boat
Names](https://botsin.space/@BoatNames) works.)

First, make a directory for the bot:

```
$ mkdir bots/boat-names
```

As you'll see, this bot is so simple that you don't even need an
`__init__.py` to program its behavior. But you still need a `bot.yaml`
to configure its schedule and where it should post.

Create `bots/boat-names/bot.yaml` and put this text in there:

```
name: Boat Names
publish:
    file: boat-names.txt
schedule:
    mean: 480
    stdev: 15
```

The schedule here is a little different than in the first example
bot. The first example's posts will come exactly one hour apart. This
bot posts every eight hours (480 minutes), _on average_, but there is
some random variation--usually up to fifteen or thirty minutes in
either direction.

Now, running `botfriend.bots` will list both of your bots.

```
$ botfriend.bots
# boat-names
# number-jokes
```

Running `botfriend.post` will tell both bots to post something if they
want, but Boat Names can never post anything, because it has no
backlog and no logic for generating new posts.

## `botfriend.backlog.load`

The `botfriend.backlog.load` script lets you add items to a bot's
backlog from a file. The simplest way to do this is with a text file
containing one post per line.

Let's create a backlog file. This can go into anywhere, but I
recommend keeping it in the same directory as the rest of the bot, in
case something goes wrong and you need to recreate it. Open up a file
`bots/boat-names/backlog.txt` and put some boat names in it:

```
Honukele
LA PARISIENNE
Stryss
Cozy Cat
Hull # 14
Always On Vacation
Sea Deuce
Bay Viewer
Tanden
Clean Livin'
Goodnight Moon
SPECIAL OCASSION
Innocent Dream
```

Now you can load the backlog:

```
$ botfriend.backlog.load boat-names --file=bots/boat-names/backlog.txt
# LOG | Backlog load script | Appended 13 items to backlog.
# LOG | Backlog load script | Backlog size now 13 items
```

Once there are items in the backlog, `botfriend.post` will work:

```
# botfriend.post
# LOG | Boat Names | file | Published 2019-01-20 03:15 | Honukele
```

## `botfriend.backlog.show`

The `botfriend.backlog.show` script will summarize a bot's current
backlog. It'll show you how many items are in the backlog and what the
next item is.

```
$ botfriend.backlog.show boat-names
# Boat Names | 12 posts in backlog
# Boat Names | LA PARISIENNE
```

## `botfriend.backlog.clear`

The `botfriend.backlog.clear` script will completely erase a bot's
backlog.

```
$ bin/backlog.clear boat-names
# Boat Names | About to clear the backlog for Boat Names.
# Boat Names | Sleeping for 2 seconds to give you a chance to Ctrl-C.

$ bin/backlog.show boat-names
#  Boat Names | No backlog.
```

# Bots that keep state: Web Words

Sometimes a bot needs to do something that takes a long time, or
something that might be annoying to someone else if it happened
frequently. Botfriend allows this difficult or annoying thing to be
done rarely, and the results stored in the bot's _state_ for
consultation later.

Let's create one more example bot. This one's called "Web Words". Its
job is to download random web pages and pick random phrases from them.

```
$ mkdir bots/web-words
```

We're going to split the "download a random web page" part of the bot
from the "pick a random phrase" part. The "pick a random phrase" part
will run every time the bot is asked to post something, but the
"download a random web page" part will run once a day.

This time let's start with the `bot.yaml` file:

```
name: "Web Words"
schedule: 60
state_update_schedule: 1440
publish:
    file:
      filename: "web-words.txt"
```

This bot will post according to its `schedule`, once an hour. But
there's another thing that's going to happen once a day (every 1440
minutes): a "state update".

Here's the `__init__.py` file:

```
import random
import re
from olipy import corpora
import requests
from botfriend.bot import TextGeneratorBot

class WebWords(TextGeneratorBot):
    """A bot that pulls random words from a random webpage."""

    def update_state(self):
        """Choose random domain names until we find one that hosts a web page
        larger than ten kilobytes.
        """
        new_state = None
        while not new_state:

            # Make up a random URL.
            word = random.choice(corpora.words.english_words['words'])
            domain = random.choice(["com", "net", "org"])
            url = "http://www.%s.%s/" % (word, domain)
            
            try:
                self.log.info("Trying to get new state from %s" % url)
                response = requests.get(url, timeout=5)
                potential_new_state = response.content
                if len(potential_new_state) < 1024 * 10:
                    # This is probably a generic domain parking page.
                    self.log.info("That was too small, trying again.")
                    continue
                new_state = response.content
                self.log.info("Success!")
            except Exception, e:
                self.log.info("That didn't work, trying again.")
        return new_state

    def generate_text(self):
        """Choose some words at random from a webpage."""
        webpage = self.model.state
       
        # Choose a random point in the web page that's not right at the end.
        total_size = len(webpage)
        near_the_end = int(total_size * 0.9)
        starting_point = random.randint(0, near_the_end)

        # Find some stuff in the webpage that looks like words, rather than HTML.
        some_words = re.compile("([A-Za-z\s]{10,})")

        match = some_words.search(webpage[starting_point:])
        if not match:
            # Because we didn't find anything, we're choosing not to post
            # anything right now.
            return None
        data = match.groups()[0].strip()
        return data

Bot = WebWords
```

The first time you tell Botfriend to post something for this bot,
Botfriend will call the `update_state()` method. This method may try
several times to find a web page it can use, but it will eventually
succeed.

```
$ botfriend.post web-words
Web Words | Trying to get new state from http://www.stenographical.com/
Web Words | That was too small, trying again.
Web Words | Trying to get new state from http://www.bronchologic.org/
Web Words | That didn't work, trying again.
Web Words | Trying to get new state from http://www.dentonomy.org/
Web Words | That was too small, trying again.
Web Words | Trying to get new state from http://www.crummy.com/
Web Words | Success!
Web Words | file | Published 2019-01-10 01:45 | e experimental group
```

Once the state is in place, running `botfriend.post` again won't
download a whole new web page every time. Instead, Web Words will
choose another random string from the webpage it's already downloaded.

```
$ botfriend.post web-words --force
Web Words | file | Published 2019-01-10 01:46 an old superstition
```

This bot's state expires in one day (this was set in its
`bot.yaml`). 24 hours after `update_state()` is called for the first
time, running `botfriend.post` will cause Botfriend to call that
method again. A brand new web page will be downloaded, and for the
next 24 horus all of the Web Words posts will come from that web page.

## `botfriend.state.show` - Showing the state

This script simply prints out a bot's current state.

```
$ botfriend.state.show web-words
```

## `botfriend.state.refresh` - Refreshing the state

You can use this script to forcibly refresh a bot's state, even if the
bot's configured `state_update_schedule` says it's not time yet.

```
$ botfriend.state.refresh web-words
```

## `botfriend.state.set` - Setting the state to a specific value

You can use this script to set a bot's state to a specific value,
rather than setting the state by calling the `update_state()`
method. Here, instead of telling Web Words to pick random strings from
a web page, we're telling it to pick random strings from its own
source code.

```
$ bin/state.set web-words --file=bots/web-words/__init__.py

$ bin/post web-words --force
Web Words | file | Published 2018-01-21 1:56 | while not new
```

--- Stopping here

# Loading up more examples

To see how this works, started, check out [the `botfriend`
project repository](https://github.com/leonardr/botfriend/)
and copy its `sample-bots` directory into your virtual environment:

```
git clone git@github.com:leonardr/botfriend.git
cp -r botfriend/bots.sample/* bots
```

The `bots.sample` directory contains over ten sample bots--the ones I
describe in this document and several based on real bots that I run.

You can see what your bots are up to with the `botfriend.dashboard` script. 

```
botfriend.dashboard
```

This script will tell you the last time each bot posted, the next time
it's going to post, what it last posted, and (if known) the next thing
it's going to post. Since you just installed botfriend, running
`dashboard` won't do much.

To tell the bots to actually post something, run the `post` script:

```
$ botfriend.post
```

At this point, some of the sample bots will raise a `NothingToPost`
exception. [Roy's
Postcards](https://github.com/leonardr/botfriend/tree/master/bots.sample/postcards)
and [Deathbot
3000](https://github.com/leonardr/botfriend/tree/master/bots.sample/roller-derby)
are examples. These bots can't just make up something to post--they
can only post from a backlog. Since they start out with an empty
backlog, they have nothing to post.

Other bots, like [Best of
RHP](https://github.com/leonardr/botfriend/tree/master/bots.sample/best-of-rhp)
and [Ingsoc Party
Slogans](https://github.com/leonardr/botfriend/tree/master/bots.sample/ingsoc-slogans),
will raise `TweepError` at this point. These bots need working Twitter
credentials to function, and none of the sample bots are supplied with
valid Twitter credentials.

But most of the bots in `bots.sample` will work fine. They will
generate one or more posts and publish them. These posts won't be
published to Twitter or Mastodon, since none of the sample bots have
working credentials. Instead, the posts will be published to files in
the bot subdirectories.

For example, [A Dull
Bot](https://github.com/leonardr/botfriend/tree/master/bots.sample/a-dull-bot)
will write one post at a time to
`bots.sample/a-dull-bot/a-dull-bot.txt`.

[Crowd Board
Games](https://github.com/leonardr/botfriend/tree/master/bots.sample/crowd-board-games)
will convert an entire RSS feed into a series of posts, and write them
all to `bots.sample/crowd-board-games/crowd-board-games.txt`.

[Anniversary
Materials](https://github.com/leonardr/botfriend/tree/master/bots.sample/anniversary)
can get extra data from Twitter if it has Twitter credentials, but it
won't crash or give up if those credentials are missing. When you run
`post` this bot will come up with a gift idea using the built-in
datasets from `olipy` and `corpora`, and write its idea to
`bots.sample/anniversary/anniversary.txt`.

If you run `botfriend.dashboard` after running `botfriend.post`,
you'll see that everything that just happened was archived in the
database at `bots/botfriend.sqlite`.

The second time you run `botfriend.post`, you'll see a
lot less action. The bots that crashed the first time will crash the
second time, but most of the bots will be silent. This is because they
just posted something. Every bot has some kind of scheduling rules
that limit how often it posts. Generally you don't want bots to post
more than once an hour.

If you want to force a specific bot to post, despite its schedule, you
can use the `--force` option, like this:

```
botfriend.post --bot=anniversary --force
```

# Bot anatomy

So, you've got about ten examples to look at in `bots.sample/`. Each
bot directory contains two special files: `bot.yaml` for configuration
and `__init__.py` for source code.

# `bot.yaml`

This is a YAML file which contains all the information necessary to
configure your bot. You can put anything you want in here, but you
need to supply three pieces of information:

* The name of your bot
* How often the bot should post things
* The specific services to post to, and API credentials that can be used
  on those services.

Configuration options are covered in more detail below, but here's a
simple example: the `bot.yaml` for [A Dull Bot](https://github.com/leonardr/botfriend/tree/master/bots.sample/a-dull-bot).

```
name: "A Dull Bot"
schedule: 60
publish:
    file:
      filename: "a-dull-bot.txt"
```

This is saying:

* The name of the bot is "A Dull Bot".

* The bot should publish something exactly once an hour.

* When it's time for the bot to publish something, the 'something'
  should be written to the file `a-dull-bot.txt`.

# `__init__.py`

This is where the bot's source code lives. There's only one rule here:
you have to define a class called `Bot`. The Botfriend scripts are
going to load that class, instantiate it, and use it to make that
bot's contribution to whatever the script is supposed to do. The bot
may be asked to post something, list previous posts, or whatever.

For a complex bot, you can subclass `BasicBot` and override its
`new_post` method.

```
from bot import BasicBot
class Bot(BasicBot):
    def new_post(self):
      # Generate a new Post object
      ...
```

But for most common types of bot, there's a more specific class you
can subclass, which will eliminate most of the work and any need to
deal with the Botfriend object model.

## Bots that generate posts on demand

The simplest type of bot generates a totally new string every time
it's invoked. There's no connection between one post and the
next. Each post is the result of running the same random process.

To create this kind of bot, subclass
[`TextGeneratorBot`](https://github.com/leonardr/botfriend/blob/master/bot.py)
and implement the `generate_text()` method to return the string you
want to post. Every time your bot needs to post something, Botfriend
will call its `generate_text()` method, and post whatever is returned.

Several examples of `TextGeneratorBot` are included with botfriend,
the simplest being [A Dull
Bot](https://github.com/leonardr/botfriend/tree/master/bots.sample/a-dull-bot).
Here's the _entire_ code of A Dull Bot:

```
from olipy.typewriter import Typewriter
from olipy.integration import pad
from bot import TextGeneratorBot

class TypewriterBot(TextGeneratorBot):

    def generate_text(self):
        typewriter = Typewriter()
        text = typewriter.type("All work and no play makes Jack a dull boy.")
        return pad(text)
        
Bot = TypewriterBot
```

Whenever it's time to post something, A Dull Bot instantiates a
simulated typewriter, types a phrase on the typewriter (which usually
introduces some typos), and returns the phrase (typos and all). You
can see the results on [Twitter](https://twitter.com/a_dull_bot) or
[Mastodon](http://botsin.space/adullbot). 

Although it's very simple, creating this bot was a fair amount of
work. But all of the work went into the fun part: creating an accurate
software model of the typewriter from _The Shining_. There's no code
for making sure the bot posts once an hour, or for pushing the
typewritten text through the Twitter or Mastodon APIs. Botfriend takes
care of all that stuff.

[Euphemism
Bot](https://github.com/leonardr/botfriend/tree/master/bots.sample/euphemism)
and [Serial
Entrepreneur](https://github.com/leonardr/botfriend/tree/master/bots.sample/serial-entrepreneur)
have more complex generative logic, but they're still very simple
`TextGeneratorBot`s.

[I Am A
Bot. AMA!](https://github.com/leonardr/botfriend/tree/master/bots.sample/ama)
is a more complicated `TextGeneratorBot` that stores state between
invocations to minimize load on the Twitter API.

## Bots that know what they'll post in advance

Instead of creating a new post every time it's time for the bot to
act, you can list out all the things the bot should post ahead of
time, and have the bot post items off the list as appropriate.

There are two ways to do this. If you need posts to be published at
specific dates and times, you can schedule posts with
`ScriptedBot`. Most of the time, though, you need posts to be
published at rough intervals but not at specific times. In that case,
it's easier to subclass the base `Bot` class and load in a backlog from a text file.

### Bots that post from a backlog

When you have a large number of posts and the exact timing isn't
particularly important, the best thing to do is to create an empty
subclass of `Bot` and use the command line to dump all those posts
into your bot's backlog. Every time your bot is asked to post
something, it will pick the top item off the backlog and send it to be
published.

[Boat Name
Bot](https://github.com/leonardr/botfriend/tree/master/bots.sample/boat-names)
is a bot that has no custom code whatsoever. It does nothing but post
strings from its backlog. The `__init__.py` for Boat Name Bot contains
no code, just a placeholder:

```
from bot import Bot
```

The real action is in the `backload-load` script, which loads items
into the backlog from a text file. Boat Name Bot comes with a [sample backlog
of boat names](https://github.com/leonardr/botfriend/blob/master/bots.sample/boat-names/backlog.txt), which you can load with a command like this:

```
./backlog-load --config=bots.sample --bot=boat-names --file=bots.sample/boat-names/input.txt
```

Once you've loaded a backlog, the `./post` script will start posting
items off the backlog, gradually, one at a time. With a backlog bot,
there's almost no risk of something going wrong while the bot is in
operation, because you've done all the work ahead of time. The bot is
going to post the things from that text file, one at a time.

[Death Bot
3000](https://github.com/leonardr/botfriend/tree/master/bots.sample/roller-derby)
is a backlog bot that is a little more complicated. Instead of keeping
its backlog in a text file, it expects its backlog in the form of
[JSON
lists](https://github.com/leonardr/botfriend/blob/master/bots.sample/roller-derby/backlog.ndjson).
To convert from a JSON list to a string, Death Bot 3000 subclasses the
`object_to_post` method.

### Bots that act out a script

The default assumption of Botfriend is that a bot should post at a
certain interval, but that the exact times aren't important. Most of
the time you _want_ a little variation in the posting time, so that
your bot doesn't post at the exact same time all the time, like a
clock striking the hour.

But sometimes the exact timing _is_ important. If it's important that
your bost posts at specific times, you can have schedule its posts at
precise times, by subclassing `ScriptedBot`.

[Frances
Daily](https://github.com/leonardr/botfriend/tree/master/bots.sample/frances-daily)
posts journal entries on specific dates, exactly thirty years after the
journal entries were originally written. On some days, there is no post
at all. So there's no simple rule like "every day, publish the next
post." Each post is associated with a specific date and time.

Frances Daily doesn't have any special code -- it uses [a simple JSON
script
format](https://github.com/leonardr/botfriend/blob/master/bots.sample/frances-daily/script.ndjson)
to explain what should be published when. Most of the times, you can
create schedule posts without writing any special Botfriend code at
all. All the work goes into generating your script.

To load a list of scheduled posts, use the `scheduled-load`
script. Here's an example that loads the script for Frances Daily:

```
./scheduled-load --config=bots.sample --bot=frances-daily --file=bots.sample/frances-daily/script.ndjson
```

Once you load the scheduled posts, the `./post` script will start
posting them. Note that it won't post items that are more than a few
days old.

### Bots can post images

Bots can post images as well as text. [Roy's
Postcards](https://github.com/leonardr/botfriend/tree/master/bots.sample/postcards)
is an example of a bot that loads a combination of images and text
into its backlog. In this case the images are loaded off disk when
it's time to post them -- only the filenames are stored in the
database backlog. The images will be uploaded as attachments to the
posts; you can see this in action on
[Twitter](https://twitter.com/RoyPostcards/) or
[Mastodon](https://botsin.space/@RoysPostcards).

# Tracking bot-specific state

[I Am A
Bot. AMA!](https://github.com/leonardr/botfriend/tree/master/bots.sample/ama)
gets its data by using the Twitter API to run searches on strings like
"i am a" and "i am an". These searches take a long time and can cause
Twitter to rate-limit the bot if you run them too often. Rather than
go through this process every single time the bot wants to post, the
`IAmABot` class implements the `update_state` method. This method
performs the searches against the Twitter API, and returns the raw data
in a JSON string that's stored in the database.

When it's time for the bot to post something, the current value of the
bot's state is accessible through the property `self.model.state`. 

You can store whatever you want in the bot's state. Anything that can
be cached and is time-consuming to calculate is a good
choice. [Anniversary
Materials](https://github.com/leonardr/botfriend/tree/master/bots.sample/anniversary)
is another example--it also uses the Twitter search API to gather raw
data from Twitter, which it stores in its state.

# Configuration

Let's take another look at [A Dull Bot's bot.yaml](https://github.com/leonardr/botfriend/tree/master/bots.sample/a-dull-bot/bot.yaml):

```
name: "A Dull Bot"
schedule: 60
publish:
    file:
      filename: "a-dull-bot.txt"
```

The `name` option should be self-explanatory -- it's the human-readable name of
the bot. Now let's take a detailed look at the other two options.

## `schedule`

The `schedule` configuration option controls how often your bot should
post. There are basically three strategies.

1. Set `schedule` to a number of minutes. (This is what A Dull Bot
does.) Your bot will post at exact
intervals, with that number of minutes between posts.
2. Give `schedule` a `mean` number of minutes. Your bot will post at
randomly determined intervals, with _approximately_ that number of
minutes between posts, but with a fair amount of random variation.
3. To fine-tune the randomness, you can specify a `stdev` to go along
with the mean. This sets the standard deviation used when calculating
when the next post should be. Set it to a low number, and posts will
nearly `mean` minutes apart. Set it to a high number, and the posting
schedule will vary widely.

You can omit `schedule` if your bot schedules all of its posts ahead
of time (like [Frances
Daily](https://github.com/leonardr/botfriend/tree/master/bots.sample/frances-daily)
does).

### `state_update_schedule`

There's a related option, `state_update_schedule`, which you only need
to set if your bot keeps internal state, like [I Am A
Bot. AMA!](https://github.com/leonardr/botfriend/tree/master/bots.sample/ama)
does. This option works the same way as `schedule`, but instead of
controlling how often the bot should post, it controls how often your
`update_state()` method is called.

## Other publication options

`schedule`, and `publish` are only the most common
configuration options.  [Best of
RHP](https://github.com/leonardr/botfriend/tree/master/bots.sample/best-of-rhp),
a subclass of `RetweetBot`, has a special configuration setting called
`retweet-user`, which controls the Twitter account that the bot
retweets. Your bot can have its own configuration options--the
configuration object is parsed as YAML and passed into the `Bot`
constructor as the `config` argument. You can look in there and pick
out whatever information you want.

## Defaults

If you put a file called `default.yaml` in your Botfriend directory
(next to `botfriend.sqlite`), all of your bots will inherit the values
in that file.

Almost all my bots use the same Mastodon and Twitter client keys (but
different application keys), and all my Mastodon bots are hosted at
botsin.space. I keep these configuration settings in `default.yaml` so
I don't have to repeat them in every single `bot.yaml` file. My
`default.yaml` looks like this:

```
publish:
  mastodon: {api_base_url: 'https://botsin.space/', client_id: a, client_secret: b}
  twitter: {consumer_key: c, consumer_secret: d}
```

This way, inside a given `bot.yaml` file, I only have to fill in the
information that's not specified in `default.yaml`:

```
name: My Bot
publish:
 mastodon:
  access_token: efg
 twitter:
  access_token: hij
  access_token_secret: klm
schedule:
 mean: 120
```

## Programmatic access to an API

Sometimes you'll need to communicate with an API for more than just
posting. You can always access a publisher from inside a `Bot` as
`self.publishers`, and the raw API will always be available as
`Publisher.api`.

See the [`IAmABot`
constructor](https://github.com/leonardr/botfriend/blob/master/bots.sample/ama/__init__.py)
for an example. This bot needs a Twitter API client to get its data,
so it looks through `self.publishers` until it finds the Twitter
publisher, and grabs its `.api`, storing it for later.

# Posting on a regular basis

Once you have a few bots, you'll need to run the `post` script
regularly to keep new content flowing. The best way to do this is to
set up a cron job to schedule the `post` script to run every few
minutes. Don't worry about posting too often. Bots that need to post
something will post when they're ready. Bots that don't need to post
anything right when `post` is run will be quiet, and bide their time.

Here's what my cron script looks like:

```
#!/bin/bash
export ROOT=$HOME/scripts/botfriend
source $ROOT/env/bin/activate
$ROOT/post
```

Here's how I use a cron job to run it every five minutes

```
*/5 * * * * /home/leonardr/scripts/botfriend/cron 2> /home/leonardr/scripts/botfriend_err
```

Any errors that happen during the run are appended to a file,
`botfriend_err`, which I can check periodically.

# Conclusion

There are a lot of features of Botfriend that I've barely touched or
not mentioned at all: bots that retweet other Twitter accounts, bots
that get their posts by scraping a web page for their content, scripts
for showing or clearing the backlog, scripts for managing the stored
state if it should get screwed up, scripts for republishing posts that
weren't posted properly the first time.

But the features I've covered are the main ones you need to get
started and to see the power of Botfriend. I hope you enjoy it!
