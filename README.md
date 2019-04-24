# irclog couchapp

An assumption is that some kind of an irc bot, stores irc messages in CouchDB.
The bot itself is out of scope here (see references below), but what is important
is the shape of the messages (a document in couchdb parlance).

Each message is a json document:

```
{
  "message": "lorem ipsum",
  "sender": "johndo",
  "channel": "freenode",
  "timestamp": 1556136148.123456
}
```

Now, just a bag of messages is not enough, we need to index them, sort them, filter them,
and search through them. This is where CouchDB and its map/reduce views and filters come
into play, and what this repo provides. Each view and filter in the `views/` and
`filters/` directory becomes an API endpoint when added to a CouchDB database as
a "design document".

Some benefits of couchdb that we get for free are:
* the API is accessed via http and json, no special protocols required
* it can also serve static web pages (a webapp), that can then access the API (a couchapp)
* supports change notifications via longpoll or eventsource

# howto

To setup this couchapp, add your frontend of choice in the `_attachments/` directory, and just run:

```sh
couchapp push . https://your-couchdb-server.example.net/
```

# API

… FIXME …


# References:

* [Apache CouchDB](http://couchdb.apache.org/)
* [What is a couchapp](http://couchapp.readthedocs.io/en/latest/intro/what-is-couchapp.html)
* [a bot and its plugin that logs messages to couchdb](https://github.com/gdamjan/erlang-irc-bot-skopjehacklab/blob/master/src/ircbot_plugin_couch_log.erl)
* [webapp implementations in different frontend frameworks](https://github.com/irclogs)
* [cli access to the logs](https://github.com/irclogs/cli)
