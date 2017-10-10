[![Build Status](https://travis-ci.org/inertia186/radiator.svg?branch=master)](https://travis-ci.org/inertia186/radiator)
[![Code Climate](https://codeclimate.com/github/inertia186/radiator/badges/gpa.svg)](https://codeclimate.com/github/inertia186/radiator)
[![Test Coverage](https://codeclimate.com/github/inertia186/radiator/badges/coverage.svg)](https://codeclimate.com/github/inertia186/radiator)
[![Inline docs](http://inch-ci.org/github/inertia186/radiator.svg?branch=master&style=shields)](http://inch-ci.org/github/inertia186/radiator)

[radiator](https://github.com/inertia186/radiator)
========

#### STEEM Ruby API Client

Radiator is an API Client for interaction with the STEEM network using Ruby.

#### Changes in v0.3.0

* Gem updates
* Added failover subroutines (see Failover section, below).
* Added method closures support (aka passing a block to yield).
* You can now stream virtual operations (see Streaming section, below).
* Added more [documentation](http://www.rubydoc.info/gems/radiator).
* Added/expanded more api namespaces: `::BlockApi`, `::CondenserApi`, `::TagApi`
* Addressed an issue with logging on certain Windows configurations.

#### Fixes in v0.2.3

* Gem updates
* Added low-level support for persistence and retrying API requests.
* Now using exponential back-off for retries.
* Detecting presence of `transaction_ids` (if enabled by the node).
* Default for `Hashie` warnings now go to `/dev/null`, where they belong.
* Added stray methods/operations.

#### Fixes in v0.2.2

* Gem updates
* Improved support for datatypes and handlers.
  * UTF-8 handled more smoothly.
  * Simplified operation construction.
* Improved keep-alive defaults.
  * Better streaming reliability.

---

Also see: [Documentation](http://www.rubydoc.info/gems/radiator)

---

### Quick Start

Add the gem to your Gemfile:

```ruby
gem 'radiator'
```

Then:

```bash
$ bundle install
```

If you don't have `bundler`, see the next section.
    
### Prerequisites

`minimum ruby version: 2.2`

#### Linux

```bash
$ sudo apt-get install ruby-full git openssl libssl1.0.0 libssl-dev
$ gem install bundler
```

#### macOS

```
$ gem install bundler
```

### Usage

```ruby
require 'radiator'

api = Radiator::Api.new
api.get_dynamic_global_properties do |properties|
  properties.virtual_supply
end
=> "135377049.603 STEEM"
```

... or ...

```ruby
require 'radiator'

api = Radiator::Api.new
response = api.get_dynamic_global_properties
response.result.virtual_supply
=> "135377049.603 STEEM"
```

#### Follower API

```ruby
api = Radiator::FollowApi.new
api.get_followers('inertia', 0, 'blog', 100) do |followers|
  followers.map(&:follower)
end
=> ["a11at",
 "abarefootpoet",
 "abit",
 "alexgr",
 "alexoz",
 "andressilvera",
 "applecrisp",
 "arrowj",
 "artificial",
 "ash",
 "ausbitbank",
 "beachbum",
 "ben99",
 "benadapt",
 .
 .
 .
 "steemzine"]
```

#### Streaming

Here's an example of how to use a streaming instance to listen for votes:

```ruby
require 'radiator'

stream = Radiator::Stream.new

stream.operations(:vote) do |op|
  print "#{op.voter} voted for #{op.author}"
  puts " (weight: #{op.weight / 100.0}%)"
end
```

The output would look like this and continue until interrupted.

```
richman voted for krnel (weight: 100.0%)
rainchen voted for rainchen (weight: 100.0%)
richman voted for exploretraveler (weight: 100.0%)
jlufer voted for michaelstobiersk (weight: 100.0%)
jlufer voted for michaelstobiersk (weight: 100.0%)
patelincho voted for borishaifa (weight: 100.0%)
richman voted for vetvso (weight: 100.0%)
jlufer voted for michaelstobiersk (weight: 100.0%)
richman voted for orcish (weight: 100.0%)
demotruk voted for skeptic (weight: -100.0%)
photorealistic voted for oecp85 (weight: 100.0%)
meesterboom voted for rubenalexander (weight: 100.0%)
thecurator voted for robyneggs (weight: 40.0%)
richman voted for originate (weight: 100.0%)
helikopterben voted for etcmike (weight: 100.0%)
.
.
.
```

You can also just stream all operations like this:

```ruby
stream.operations do |op|
  puts op.to_json
end
```

Example of the output:

```json
{
   "vote":{
      "voter":"abudar",
      "author":"rangkangandroid",
      "permlink":"the-kalinga-tattoo-maker",
      "weight":10000
   }
}
{
   "vote":{
      "voter":"shenburen",
      "author":"masteryoda",
      "permlink":"daily-payouts-leaderboards-september-16",
      "weight":10000
   }
}
{
   "vote":{
      "voter":"stiletto",
      "author":"fyrstikken",
      "permlink":"everybody-hating-me",
      "weight":2500
   }
}
{
   "comment":{
      "parent_author":"mariandavp",
      "parent_permlink":"re-onceuponatime-re-mariandavp-the-bridge-original-artwork-by-mariandavp-20160906t182016608z",
      "author":"onceuponatime",
      "permlink":"re-mariandavp-re-onceuponatime-re-mariandavp-the-bridge-original-artwork-by-mariandavp-20160917t054726763z",
      "title":"",
      "body":"https://www.steemimg.com/images/2016/09/17/oldcomputerpics551cb14c.jpg",
      "json_metadata":"{\"tags\":[\"art\"],\"image\":[\"https://www.steemimg.com/images/2016/09/17/oldcomputerpics551cb14c.jpg\"]}"
   }
}
{
   "vote":{
      "voter":"abudar",
      "author":"rangkangandroid",
      "permlink":"the-journey-north-through-the-eyes-of-kalinga-tradition",
      "weight":10000
   }
}
{
   "limit_order_cancel":{
      "owner":"fnait",
      "orderid":2755220300
   }
}
.
.
.
```

You can also stream virtual operations:

```ruby
stream.operations(:producer_reward) do |op|
  puts "#{op.producer} got a reward: #{op.vesting_shares}"
end
```

Example of the output:

```
anyx got a reward: 390.974648 VESTS
gtg got a reward: 390.974647 VESTS
someguy123 got a reward: 390.974646 VESTS
jesta got a reward: 390.974646 VESTS
blocktrades got a reward: 390.974645 VESTS
timcliff got a reward: 390.974644 VESTS
bhuz got a reward: 1961.046504 VESTS
.
.
.
```

Transactions are supported:

```ruby
stream.transactions do |tx, trx_id|
  puts "[#{trx_id}] #{tx.to_json}"
end
```

Example of the output:

```json
{
   "ref_block_num":59860,
   "ref_block_prefix":2619183808,
   "expiration":"2016-09-17T06:03:21",
   "operations":[
      [
         "custom_json",
         {
            "required_auths":[

            ],
            "required_posting_auths":[
               "acidpanda"
            ],
            "id":"follow",
            "json":"[\"follow\",{\"follower\":\"acidpanda\",\"following\":\"gavvet\",\"what\":[\"blog\"]}]"
         }
      ]
   ],
   "extensions":[],
   "signatures":[
      "2048d7e32cc843adea0e11aa617dc9cdc773d0e9a0a0d0cd58d67a9fcd8fa2d2305d1bb611ac219fbd3b6a77ab60071df94fe193aae33591ee669cc7404d4e4ec4"
   ]
}
.
.
.
```

Even whole blocks:

```ruby
stream.blocks do |bk, num|
  puts "[#{num}] #{bk.to_json}"
end
```

Example of the output:

```json
{
   "previous":"004cea0d46a4b91cffe7bb71763ad2ab854c6efd",
   "timestamp":"2016-09-17T06:05:51",
   "witness":"boatymcboatface",
   "transaction_merkle_root":"0000000000000000000000000000000000000000",
   "extensions":[],
   "witness_signature":"2034b0d7398ed1c0d7511ac76c6dedaf227e609dc2676d13f926ddd1e9df7fa9cb254af122a4a82dc619a1091c87293cbd9e2db1b51404fdc8fb62f8e5f37b4625",
   "transactions":[]
}
.
.
.
```

#### Transaction Signing

Radiator supports transaction signing, so you can use it to vote:

```ruby
tx = Radiator::Transaction.new(wif: 'Your Wif Here')
vote = {
  type: :vote,
  voter: 'xeroc',
  author: 'xeroc',
  permlink: 'piston',
  weight: 10000
}

tx.operations << vote
tx.process(true)
```

You can also post/comment:

```ruby
tx = Radiator::Transaction.new(wif: 'Your Wif Here')
comment = {
  type: :comment,
  parent_permlink: 'test',
  author: 'your-account',
  permlink: 'something-unique',
  title: 'Radiator Can Post Comments!',
  body: 'Yep, this post was created by Radiator in `ruby`.',
  json_metadata: '',
  parent_author: ''
}

tx.operations << comment
tx.process(true)
```

Transfers:

```ruby
tx = Radiator::Transaction.new(wif: 'Your Wif Here')
transfer = {
  type: :transfer,
  from: 'ned',
  to: 'inertia',
  amount: '100000.000 SBD',
  memo: 'Wow, inertia!  Radiator is great!'
}

tx.operations << transfer
tx.process(true)
```

#### Golos

Radiator also supports Golos.  To use the Golos blockchain, provide a node and chain_id:

```ruby
tx = Radiator::Transaction.new(wif: 'Your Wif Here', chain: :golos, url: 'https://node.golos.ws')
vote = {
  type: :vote,
  voter: 'xeroc',
  author: 'xeroc',
  permlink: 'piston',
  weight: 10000
}

tx.operations << vote
tx.process(true)
```

There's a complete list of operations known to Radiator in [`broadcast_operations.json`](https://github.com/inertia186/radiator/blob/master/lib/radiator/broadcast_operations.json).

## Failover

Radiator supports failover for situations where a node has, for example, become unresponsive.  When creating a new instance of `::Api`, `::Stream`, and `::Transaction`, you may provide a list of alternative nodes, or leave them out to use the default list.  For example:

```ruby
options = {
  ur: 'https://steemd.steemit.com',
  failover_urls: [
    'https://steemd.steemitstage.com',
    'https://gtg.steem.house:8090'
  ]
}

api = Radiator::Api.new(options)
```

In a nutshell, the way this works is Radiator will try a node and proceed until it encounters an error, then retry the request.  If it encounters a second error within 5 minutes, it will abandon the node and try a random one from `failover_urls`.

It'll keep doing this until it runs out of failovers, then it will reset the configuration and go back to the original node.

Radiator uses an exponential back-off subroutine to avoid slamming nodes when they act up.

There's an additional behavior in `::Stream`.  When a node responds with a block out of sequence, it will use the failover logic above.  Although this is not a network layer failure, it is a bad result that may indicate a problem on the node, so a new node is picked.

There is another rare scenario involving `::Transaction` broadcasts that's handled by the failover logic: When a node responds with a network error *after* a signed transaction is accepted, Radiator will do a look-up to find the accepted signature in order to avoid triggering a `dupe_check` error from the blockchain.  This subroutine might take up to five minutes to execute in the worst possible situation.  To disable this behavior, use the `recover_transactions_on_error` and set it to `false`, e.g.:

```ruby
tx = Radiator::Transaction.new(wif: wif, recover_transactions_on_error: false)
```

## Tests

* Clone the client repository into a directory of your choice:
  * `git clone https://github.com/inertia186/radiator.git`
* Navigate into the new folder
  * `cd radiator`
* Basic tests can be invoked as follows:
  * `rake`
* To run tests with parallelization and local code coverage:
  * `HELL_ENABLED=true rake`

---

<center>
  <img src="http://www.steemimg.com/images/2016/08/19/RadiatorCoolingFan-54in-Webfdcb1.png" />
</center>

See my previous Ruby How To posts in: [#radiator](https://steemit.com/created/radiator) [#ruby](https://steemit.com/created/ruby)

## Get in touch!

If you're using Radiator, I'd love to hear from you.  Drop me a line and tell me what you think!  I'm @inertia on STEEM.
  
## License

I don't believe in intellectual "property".  If you do, consider Radiator as licensed under a Creative Commons [![CC0](http://i.creativecommons.org/p/zero/1.0/80x15.png)](http://creativecommons.org/publicdomain/zero/1.0/) License.
