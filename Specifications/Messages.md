# Messages

Each message consists of one "header" byte that describes the purpose of the
message, and a "body" of zero or more bytes that contain the actual data of the
message.

## Header byte

The header byte is what gives the rest of a message meaning, and must be read
before the body.

The first bit of the header byte is the "type" bit that indicates which parties
interact with each other via the message.

A message of server type has the first bit set (`0b_1XXX_XXXX`). These messages
can be set from a client, and might signal an attempted move in the game. They
can also be sent from the server, and might signal an actual move by clients.

A message of client type leaves the first bit unset (`0b_0XXX_XXXX`). These
message can only be created by clients, and might contain a chat message to
other clients. When the server receives one of these it must forward it to all
other clients without changing it.

This leaves room for 128 header codes for each message type that can be read by
either masking away the type bit, or reading negative/positive numbers. What
these codes mean is described in the [Message codes](#Message-codes) section.

## Message codes

### Server types

All following codes are for the server type of codes.

#### `0`: Connect

This message is the first thing a client should send after connecting to the
server. Before sending receiving this message the server should not interact
with the give client.

After receiving this message the server must respond with a message with the
same code with information about the current session. If the server can't host a
game right now, the rest of the response will be gibberish.

Header: `0b_1000_0000`

Body (client to server):  
Body should be empty. Server should ignore any present body.

Body (server to client):
| Length | Description                                        | Type           |
|-------:|----------------------------------------------------|----------------|
|    `1` | Indicating if the server can host a game right now | Boolean        |
|    `1` | Byte length of Player integer (`pi`)               | Unsigned byte  |
|   `pi` | The clients Player id                              | Player integer |
|   `pi` | Number of players currently in game                | Player integer |

#### `1`: Peer (dis)connected

Server notifying clients when another peer connects or disconnects.

Header: `0b_1000_0001`

Body:
| Length | Description                                     | Type           |
|-------:|-------------------------------------------------|----------------|
|    `1` | Weather the peer is connecting or disconnecting | Boolean        |
|   `pi` | New player count                                | Player integer |
|   `pi` | Player id of affected peer                      | Player integer |

#### `2`: Start game

The client with player id zero can tell the server to start the game. The server
can tell the clients that the game is starting.

Header: `0b_1000_0010`

Body:  
Body should be empty. Server and clients should ignore any present body.

#### `3`: Next piece

Sent by the server indicating which piece is to drop next. One such message per
client is sent.

Header: `0b_1000_0011`

Body:
| Length | Description                      | Type           |
|-------:|----------------------------------|----------------|
|   `pi` | Player id of the affected sender | Player integer |
|    `1` | The type of piece                | Piece enum     |

#### `4`: Tick

Sent by the server to all clients indicating a tick in the game.

Header: `0b_1000_0100`

Body:  
Body should be empty. Clients should ignore any present body.

#### `5`: Move

Clients send these message to the server whenever a move is made. Server
forwards it with the player id added to all clients, including the original
sender. Replies to the original sender that the provided move is not allowed.

Header: `0b_1000_0101`

Body (client to server):
| Length | Description      | Type      |
|-------:|------------------|-----------|
|    `1` | The type of move | Move enum |

Body (server to client):
| Length | Description                      | Type           |
|-------:|----------------------------------|----------------|
|   `pi` | Player id of the affected sender | Player integer |
|    `1` | The type of move                 | Move enum      |

#### `6`: Points update

Sent by the server to all clients when one client earns points.

Header: `0b_1000_0110`

Body:
| Length | Description                  | Type           |
|-------:|------------------------------|----------------|
|   `pi` | Player id of affected player | Player integer |
|    `4` | New total points             | Unsigned int   |

#### `7`: State update

Clients can request the state of peers from the server.

The playfield is represented by a `10x40` matrix where each element is a Piece
enum telling which piece the block originally came from.

The floating piece is not present in the playfield matrix, and is instead
described as follows.  
X-position is measured from the left edge to the right [0, 10).  
Y-position is measured from the top edge to the bottom [0, 40).  
Rotation is measured in number of 90-degree clockwise turns.

Header: `0b_1000_0111`

Body (client to server):
| Length | Description                  | Type           |
|-------:|------------------------------|----------------|
|   `pi` | Player id of the sought peer | Player integer |

Body (server to client):  
| Length | Description                       | Type                          |
|-------:|-----------------------------------|-------------------------------|
|   `pi` | Player id of the sought peer      | Player integer                |
|  `200` | The playfield in a `10x40` matrix | Half byte array of piece enum |
|    `1` | Currently floating piece          | Piece enum                    |
|    `1` | X-position of floating piece      | Unsigned byte                 |
|    `1` | Y-position of floating piece      | Unsigned byte                 |
|    `1` | Rotation  of floating piece       | Unsigned byte                 |

### Client types

All following codes are for the client type of codes.

#### `0`: Chat Message

Send chat messages between clients.

Header: `0b_0000_0000`

Body:
| Length | Description             | Type           |
|-------:|-------------------------|----------------|
|   `pi` | Sender player index     | Player integer |
|    `2` | Length of message (`n`) | Unsigned short |
|    `n` | Message text            | UTF-8 array    |