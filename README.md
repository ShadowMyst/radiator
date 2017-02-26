radiator
========

![radiator](http://www.steemimg.com/images/2016/08/19/RadiatorCoolingFan-54in-Webfdcb1.png)[![Build Status](https://travis-ci.org/inertia186/radiator.svg?branch=master)](https://travis-ci.org/inertia186/radiator) [![Code Climate](https://codeclimate.com/github/inertia186/radiator/badges/gpa.svg)](https://codeclimate.com/github/inertia186/radiator) [![Test Coverage](https://codeclimate.com/github/inertia186/radiator/badges/coverage.svg)](https://codeclimate.com/github/inertia186/radiator)

STEEM Ruby API Client

### Installation

Add the gem to your Gemfile:

    gem 'radiator', github: 'inertia186/radiator'
    
Then:

    $ bundle install

### Usage

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
response = api.get_followers('inertia', 0, 'blog', 100)
response.result.map(&:follower)
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

#### Streaming

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

Transactions are supported:

```ruby
stream.transactions do |tx|
  puts tx.to_json
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
stream.blocks do |bk|
  puts bk.to_json
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

Radiator now supports transaction signing, so you can use it to vote:

```ruby
transaction = Radiator::Transaction.new(wif: '5JLw5dgQAx6rhZEgNN5C2ds1V47RweGshynFSWFbaMohsYsBvE8')
vote = {
  type: :vote,
  voter: 'xeroc',
  author: 'xeroc',
  permlink: 'piston',
  weight: 10000
}

operation = Radiator::Operation.new(vote)
transaction.operations << operation
transaction.process(true)
```

You can also post/comment:

```ruby
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

tx = Radiator::Transaction.new(wif: 'Your Wif Here')
op = Radiator::Operation.new(comment)
tx.operations << op
tx.process(true)
```

There's a complete list of operations known to Radiator in [`broadcast_operations.json`](https://github.com/inertia186/radiator/blob/master/lib/radiator/broadcast_operations.json).  In the current version, only "flat" operations are broadcasted correctly.  These are individual operations with one level of nesting, like voting and commenting.  Operations that have nested datatypes, like `custom_json`, will probably be supported in the next release.

## Tests

* Clone the client repository into a directory of your choice:
  * `git clone https://github.com/inertia186/radiator.git`
* Navigate into the new folder
  * `cd radiator`
* Basic tests can be invoked as follows:
  * `rake`
* To run tests with parallelization and local code coverage:
  * `HELL_ENABLED=true rake`

## Get in touch!

If you're using Radiator, I'd love to hear from you.  Drop me a line and tell me what you think!  I'm @inertia on STEEM.
  
## License

I don't believe in intellectual "property".  If you do, consider Radiator as licensed under a Creative Commons [![CC0](http://i.creativecommons.org/p/zero/1.0/80x15.png)](http://creativecommons.org/publicdomain/zero/1.0/) License.
