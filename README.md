# irclog couchapp

An assumption is that some kind of an irc bot, stores irc messages in CouchDB.
The bot itself is out of scope here (see references below), but what is important
is the shape of the messages (a document in couchdb parlance).

Each message is a json document:

```json
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

# HowTo

```sh
uvx --from CMSCouchapp couchapp push -p $PWD -c http://admin:password@localhost:5984/irclog
```

# API

Global setup:
```js
const baseUrl = new URL("https://db.softver.org.mk/irclog");
```

List of channels:
```js
const listChannels = new URL("/ddoc/_view/channel", baseUrl);
query = new URLSearchParams({"update_seq": "true", "reduce": "true", "group_level": "1"});
listChannels.search = query.toString();
await fetch(listChannels, {
    "credentials": "omit",
    "method": "GET",
    "mode": "cors"
});
```

A page of last 100 message for a channel:
```js
const last100 = new URL("/ddoc/_view/channel", baseUrl);
query = {
  "limit": 100,
  "include_docs": true,
  "update_seq": true,
  "reduce": false,
  "descending": true,
  "startkey": ["lugola", {}],
  "endkey": ["lugola", 0],
};

res = await fetch(last100, {
    credentials: "omit",
    mode: "cors",
    method: "POST",
    body: JSON.stringify(query),
    headers: {'Content-Type': 'application/json'}
});

data = await res.json();
update_seq = data.update_seq
```

Get a feed of new messages since the last page (you need the update_seq from the previous request):
```js
const feedUrl = new URL("/irclog/_changes", baseUrl);
const query = new URLSearchParams({feed: "longpoll", timeout: "90000", include_docs: "true",
                    filter: "_selector", since: update_seq
});
feedUrl.search = query.toString();

res = await fetch(feedUrl, {
    credentials: "omit",
    method: "POST",
    mode: "cors",
    headers: {"content-type":"application/json"},
    body: JSON.stringify({selector: {channel: "spodeli"}}),
});

data = await res.json();
update_seq = data.last_seq
```
This can be run in a loop, to get even newer messages as they appear.


# References:

* [Apache CouchDB](http://couchdb.apache.org/)
* [What is a couchapp](http://couchapp.readthedocs.io/en/latest/intro/what-is-couchapp.html)
* [a bot and its plugin that logs messages to couchdb](https://github.com/gdamjan/erlang-irc-bot-skopjehacklab/blob/master/src/ircbot_plugin_couch_log.erl)
* [webapp implementations in different frontend frameworks](https://github.com/irclogs)
* [cli access to the logs](https://github.com/irclogs/cli)
* [uvx](https://docs.astral.sh/uv/#tool-management)
