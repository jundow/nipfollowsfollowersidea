NIP-0A
======

Discrete query messages - "FOLLOWS" and "FOLLOWERS"
---------------------------------------------------

'draft' 'optional'

## Discrete query messages

Discrete query messages aim for relays to provide essential, quick and efficient searching methods instead of using the "REQ" message.

Each discrete query message should have a single function and be simple enough to obtain results from tables with indexes to ensure quick searching. 

This NIP determines two discreet query messages, "FOLLOWS" and "FOLLOWERS."

### "FOLLOWS" 

"FOLLOWS" has the messaging pattern following. 

- ["FOLLOWS", <subscription_id>, ["npub1", "npub2", ...] ]

Once the message is received, the relay should return each npub's follow list based on the latest kind 3 event.

The result is much simpler than the original kind 3 event such as:

``` json
["FOLLOWS", <subscription_id>, 
 {
        "pubkey": "npub1",
        "follows": [
            ["91cf9..4e5ca", "wss://alicerelay.com/", "alice"],
            ["14aeb..8dad4", "wss://bobrelay.com/nostr", "bob"],
            ["612ae..e610f", "ws://carolrelay.com/ws", "carol"]
        ]
 },
 {
        "pubkey": "npub2",
        "follows": [
            ...
 ]
 },
 {
        "pubkey": "npub3",
        "follows": [
            ...
 ]
 }
    ...
]
```
### "FOLLOWERS"

"FOLLOWERS" has the following messaging pattern. 

- ["FOLLOWERS", <subscription_id>, ["npub1", "npub2", ...] ]

Once the message is received, the relay should return the follower list of each npub based on aggregating all the latest kind 3 events of all the npubs.
The result from the relay has the following pattern.

``` json
["FOLLOWERS", <subscription_id>, 
 {
        "pubkey": "npub1",
        "followers": [
            ["91cf9..4e5ca", "wss://alicerelay.com/", "alice"],
            ["14aeb..8dad4", "wss://bobrelay.com/nostr", "bob"],
            ["612ae..e610f", "ws://carolrelay.com/ws", "carol"]
    },
 {
        "pubkey": "npub2",
        "followers": [ 
            ... 
        ]
 },
 {
        "pubkey": "npub3",
        "followers": [ 
            ... 
        ]
 },
    ...
]
```
## Communication between clients and relays

### From client to relay:

The following message will be used to manage a query session, including "FOLLOWS" and "FOLLOWERS" already described above.

- ["CLOSE", <subscription_id>], used to stop the ongoing querying session.

### From relay to client

Relays can send the following messages.

["EOSE", <subscription_id>], used to indicate the end of searching and the beginning of a new searching session.
["CLOSED", <subscription_id>, <message>], used to indicate that the subscription was ended on the server side.
["NOTICE", <message>], used to send a human-readable message in case of an error.

## Appendix - managing the latest follows and followers of each npub

To obtain follows and followers of each npub from a relay, it should have an aggregated list of all the follows of the npubs that stored kind-3 to the relay such as:

| npub(indexed) | follow(indexed) | main_relay | petname |
|:-:|:-:|:-:|:-:|
| **npub1** | npub2 | wss://relay1.net | Bob |
| **npub1** | npub3 | wss://relay1.net | Carol |
| **npub1** | npub4 | wss://relay2.net | Dave |
| npub2 | npub1 | wss://relay1.net | Alice |
| npub2 | npub3 | wss://relay1.net | Carol |
| npub2 | npub4 | wss://relay2.net | David |
| npub4 | npub1 | wss://relay1.net |  |
| npub4 | npub2 | wss://relay1.net |  |

Both the "npub" and "follow" columns must be indexed in the database. That way, by searching "npub" with a specific "follow," the relay can search for followers quickly. 

| npub(indexed) | follow(indexed) | main_relay | petname |
|:-:|:-:|:-:|:-:|
| *npub2* | **npub1** | wss://relay1.net | Alice |
| *npub4* | **npub1** | wss://relay1.net |  |
| npub1 | npub2 | wss://relay1.net | Bob |
| npub4 | npub2 | wss://relay1.net |  |
| npub1 | npub3 | wss://relay1.net | Carol |
| npub2 | npub3 | wss://relay1.net | Carol |
| npub1 | npub4 | wss://relay2.net | Dave |
| npub2 | npub4 | wss://relay2.net | David |

("npub1" is followed by "npub2" and "npub4")