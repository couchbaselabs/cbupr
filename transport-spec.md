
#UPR Transport Specification

##Terminology

* **Consumer** - The endpoint in a connection that is responsible for requesting different kinds of data. The consumer is responsible for doing some sort of processing with the data sent from the producer.
* **Producer** - The endpoint in a connection that is responsible for producing data for given requests.
* **Stream** - A stream is a series of messages sent over a given period of time that are generate from a particular request message.
* **Snapshot** - A unique sequence of keys that is sent by a stream.

##Messages

UPR utilize the Memcached binary protocol as the transport protocol
(see https://code.google.com/p/memcached/wiki/BinaryProtocolRevamped
for more information about the protocol layout), and defines a set
opcodes with new commands.

It does differ from the standard Memcached connections in the way that
a UPR connection is full duplex while the normal connections is
simplex (the client send a command, the server respond etc).

The typical scenario is that the client start requesting a stream and
upon success the server will start sending *command* messages back to
the client for mutations/deletions/expirations etc. The client may at
any time send additional commands to the server to start additional
UPR streams etc.

###Failover Log Request (opcode 0x51)

The Failover log request is used by the consumer to request all known
failover ids a client may use to continue from. A failover id consists
of the vbucket UUID and a sequence number. If a client can't find a
known failover id, it should select the vbucket with the highest
sequence number since that is the stream with the shortest path
to completion.

The request:
* Must not have extras
* Must not have key
* Must not have value

The response:
* Must not have extras
* Must not have key
* Must have value on Success

The following example requests the failover log for vbucket 0:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x51          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_GET_FAILOVER_LOG command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x51
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

If the command executes successful (see the status field), the
following packet is returned from a server which have 4 different
failover ids available:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x51          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x40          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x54          | 0x32          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      44| 0x00          | 0xde          | 0xca          | 0xfe          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0x01          | 0x34          | 0x32          | 0x14          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0xfe          | 0xed          | 0xfa          | 0xce          |
        +---------------+---------------+---------------+---------------+
      64| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      68| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      72| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      76| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      80| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      84| 0x00          | 0x00          | 0x65          | 0x24          |
        +---------------+---------------+---------------+---------------+
    UPR_GET_FAILOVER_LOG response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x51
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0000
    Total body   (8-11) : 0x00000040
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      vb UUID    (24-31): 0x00000000feeddeca
      vb seqno   (32-39): 0x0000000000005432
      vb UUID    (40-47): 0x0000000000decafe
      vb seqno   (48-55): 0x0000000001343214
      vb UUID    (56-63): 0x00000000feedface
      vb seqno   (64-71): 0x0000000000000004
      vb UUID    (72-79): 0x00000000deadbeef
      vb seqno   (80-87): 0x0000000000006524

There are multiple reason's why the request may fail (see the status
field), but the one that's most likely to expect is "not my vbucket"
if the requested vbucket isn't located on the server.

###Stream Request (opcode 0x50)

Sent by the consumer side to the producer specifying that the consumer
want some piece of data (Ex. XDCR). In order to initial a stream from
a vbucket the consumer must send the following command below. In order
to initiate multiple stream the consumer needs to send multiple
commands. The value specified in opaque in the stream request packet
will be used as opaque field in all commands sent for the stream.

The request:
* Must have extras
* May have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| Flags                                                         |
       +---------------+---------------+---------------+---------------+
      4| RESERVED                                                      |
       +---------------+---------------+---------------+---------------+
      8| Start sequence number                                         |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     16| End sequence number                                           |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     24| VBucket UUID                                                  |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     32| High sequence number                                          |
       | High sequence number                                          |
       +---------------+---------------+---------------+---------------+
       Total 40 bytes

* **Flags** - Used to specify extra information added in the extra
    section for modifying what the stream send.
* **Start By Seqno** - Specified the last by sequence number that has
    been seen by the consumer.
* **End By Seqno** - Specifies that the stream should be closed when
    the sequence number with this ID has been sent.
* **VBucket UUID** - A unique identifier that is generated that is
    assigned to each VBucket. This number is generated on an unclean
    shutdown or when a Vbucket becomes active.
* **High Seqno** - The high sequence number at the time that the
    VBucket UUID was generated.

If a key is specified, it represents the **Group ID** (a name that can
be given to a stream).

The response:
* Must not have extras
* Must not have key
* May have value

The following example tries to initiate a stream for vbucket 0 that
continues from a given point in time, but the server can't continue
from that point and tells the client to roll back to a different
sequence. The client retries with the information that the server
replied with, and the stream is established successfully.

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x50          | 0x00          | 0x0a          |
        +---------------+---------------+---------------+---------------+
       4| 0x28          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x32          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0xff          | 0xee          | 0xdd          |
        +---------------+---------------+---------------+---------------+
      40| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      44| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      64| 0x76 ('v')    | 0x62 ('b')    | 0x73 ('s')    | 0x74 ('t')    |
        +---------------+---------------+---------------+---------------+
      68| 0x72 ('r')    | 0x65 ('e')    | 0x61 ('a')    | 0x6d ('m')    |
        +---------------+---------------+---------------+---------------+
      72| 0x2d ('-')    | 0x30 ('0')    |
        +---------------+---------------+
    UPR_STREAM_REQ command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x50
    Key length   (2,3)  : 0x000a
    Extra length (4)    : 0x28
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000032
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      flags      (24-27): 0x00000000
      reserved   (28-31): 0x00000000
      start seqno(32-39): 0x0000000000ffeedd
      end seqno  (40-47): 0xffffffffffffffff
      vb UUID    (48-55): 0x00000000feeddeca
      high seqno (56-63): 0x0000000000000000
      key        (64-73): vbstream-0

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x50          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x08          | 0x00          | 0x00          | 0x23          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x08          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_REQ response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x50
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x08
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0023 (Rollback)
    Total body   (8-11) : 0x00000008
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      rollback # (24-31): 0x0000000000000000


      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x50          | 0x00          | 0x0a          |
        +---------------+---------------+---------------+---------------+
       4| 0x28          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x32          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      40| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      44| 0xff          | 0xff          | 0xff          | 0xff          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0xfe          | 0xed          | 0xde          | 0xca          |
        +---------------+---------------+---------------+---------------+
      56| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      60| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      64| 0x76 ('v')    | 0x62 ('b')    | 0x73 ('s')    | 0x74 ('t')    |
        +---------------+---------------+---------------+---------------+
      68| 0x72 ('r')    | 0x65 ('e')    | 0x61 ('a')    | 0x6d ('m')    |
        +---------------+---------------+---------------+---------------+
      72| 0x2d          | 0x30          |
        +---------------+---------------+
    UPR_STREAM_REQ command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x50
    Key length   (2,3)  : 0x000a
    Extra length (4)    : 0x28
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000032
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      flags      (24-27): 0x00000000
      reserved   (28-31): 0x00000000
      start seqno(32-39): 0x0000000000000000
      end seqno  (40-47): 0xffffffffffffffff
      vb UUID    (48-55): 0x00000000feeddeca
      high seqno (56-63): 0x0000000000000000
      key        (64-73): vbstream-0

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x81          | 0x50          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_REQ response
    Field        (offset) (value)
    Magic        (0)    : 0x81
    Opcode       (1)    : 0x50
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Status       (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

As always you may receive other error messages, where "not my vbucket"
(meaning you sent the request to the wrong server) or "key not found"
meaning that the server don't know the vbucket uuid.

###Stream Start (opcode 0x52)

After the stream is set up with Stream Request the *server* starts the
stream by sending the Stream Start command to the client.

The request:
* Must not have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x52          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_START command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x52
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

###Stream End (opcode 0x53)

Sent to tell the consumer that the producer will has no more messages to stream.

The request:
* Must have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x53          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x04          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x04          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_STREAM_END command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x53
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x04
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000004
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      flag       (24-27): 0x0000000000000000

The flag may have the following values:

* OK (0x00) - The stream has finished without error.
* State Changed (0x01) - The state of the VBucket that is being streamed has changed to state that the consumer does not want to receive.


###Snapshot Start (opcode 0x54)

Sent by the producer to tell the consumer that a new snapshot is being
sent. A snaphot is simply a series of commands that is guarenteed to
contain a unique set of keys.

The request:
* Must not have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x54          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_SNAPSHOT_START command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x54
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

###Snapshot End (opcode 0x55)
Sent by the producer to tell the consumer that the current snapshot is finshed.

The request:
* Must not have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x55          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_SNAPSHOT_END command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x55
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

###Mutation (0x56)
Tells the consumer that the message contains a key mutation.

The request:
* Must have extras
* Must have key
* May have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| by_seqo                                                       |
       |                                                               |
       +---------------+---------------+---------------+---------------+
      8| rev seqno                                                     |
       |                                                               |
       +---------------+---------------+---------------+---------------+
     16| flags                                                         |
       +---------------+---------------+---------------+---------------+
     20| expiration                                                    |
       +---------------+---------------+---------------+---------------+
     24| lock_time                                                     |
       +---------------+---------------+---------------+---------------+
       Total 28 bytes

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x56          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       4| 0x1c          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x27          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      40| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      44| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      48| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      52| 0x68 ('h')    | 0x65 ('e')    | 0x6c ('l')    | 0x6c ('l')    |
        +---------------+---------------+---------------+---------------+
      56| 0x6f ('o')    | 0x77 ('w')    | 0x6f ('o')    | 0x72 ('r')    |
        +---------------+---------------+---------------+---------------+
      60| 0x6c ('l')    | 0x64 ('d')    |
        +---------------+---------------+
    UPR_MUTATION command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x56
    Key length   (2,3)  : 0x0005
    Extra length (4)    : 0x1c
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000027
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000001
      by seqno   (24-31): 0x0000000000000000
      rev seqno  (32-39): 0x0000000000000000
      flags      (40-43): 0x00000000
      expiration (44-47): 0x00000000
      lock time  (48-51): 0x00000000
      key        (52-56): hello
      value      (57-62): world

###Deletion (0x57)
Tells the consumer that the message contains a key deletion.

The request:
* Must have extras
* Must have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| by_seqo                                                       |
       |                                                               |
       +---------------+---------------+---------------+---------------+
      8| rev seqno                                                     |
       |                                                               |
       +---------------+---------------+---------------+---------------+
       Total 16 bytes

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x57          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       4| 0x10          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x15          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      40| 0x68 ('h')    | 0x65 ('e')    | 0x6c ('l')    | 0x6c ('l')    |
        +---------------+---------------+---------------+---------------+
      44| 0x6f ('o')    |
        +---------------+
    UPR_DELETION command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x57
    Key length   (2,3)  : 0x0005
    Extra length (4)    : 0x10
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000015
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000001
      by seqno   (24-31): 0x0000000000000000
      rev seqno  (32-39): 0x0000000000000000
      key        (40-44): hello

###Expiration (0x58)
Tells the consumer that the message contains a key expiration.

The request:
* Must have extras
* Must have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| by_seqo                                                       |
       |                                                               |
       +---------------+---------------+---------------+---------------+
      8| rev seqno                                                     |
       |                                                               |
       +---------------+---------------+---------------+---------------+
       Total 16 bytes

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x58          | 0x00          | 0x05          |
        +---------------+---------------+---------------+---------------+
       4| 0x10          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x15          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      24| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      28| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      32| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      36| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      40| 0x68 ('h')    | 0x65 ('e')    | 0x6c ('l')    | 0x6c ('l')    |
        +---------------+---------------+---------------+---------------+
      44| 0x6f ('o')    |
        +---------------+
    UPR_EXPIRATION command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x58
    Key length   (2,3)  : 0x0005
    Extra length (4)    : 0x10
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000015
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000001
      by seqno   (24-31): 0x0000000000000000
      rev seqno  (32-39): 0x0000000000000000
      key        (40-44): hello

###Flush (0x59)
Tells the consumer to delete all of its data for a given vbucket.

The request:
* Must not have extras
* Must not have key
* Must not have value

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x59          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
    UPR_FLUSH command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x59
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x00
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000000
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000

###Set VBucket State (0x5A)

The Set VBucket message is used during the VBucket takeover process to
hand off ownership of a VBucket between two nodes. The message format
as well as the state values for this operation is below.

The request:
* Must have extras
* Must not have key
* Must not have value

Extra looks like:

     Byte/     0       |       1       |       2       |       3       |
        /              |               |               |               |
       |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
       +---------------+---------------+---------------+---------------+
      0| state         |               |               |               |
       +---------------+---------------+---------------+---------------+
       Total 1 byte

State may have the following values:

* Active (0x01) - Changes the VBucket on the consumer side to active state.
* Pending (0x02) - Changes the VBucket on the consumer side to pending state.
* Replica (0x03) - Changes the VBucket on the consumer side to replica state.
* Dead (0x04) - Changes the VBucket on the consumer side to dead state.

The client should not send a reply to this command. The following
example shows the breakdown of the message:

      Byte/     0       |       1       |       2       |       3       |
         /              |               |               |               |
        |0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|0 1 2 3 4 5 6 7|
        +---------------+---------------+---------------+---------------+
       0| 0x80          | 0x5a          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       4| 0x01          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
       8| 0x00          | 0x00          | 0x00          | 0x01          |
        +---------------+---------------+---------------+---------------+
      12| 0xde          | 0xad          | 0xbe          | 0xef          |
        +---------------+---------------+---------------+---------------+
      16| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      20| 0x00          | 0x00          | 0x00          | 0x00          |
        +---------------+---------------+---------------+---------------+
      24| 0x04          |
        +---------------+
    UPR_SET_VBUCKET_STATE command
    Field        (offset) (value)
    Magic        (0)    : 0x80
    Opcode       (1)    : 0x5a
    Key length   (2,3)  : 0x0000
    Extra length (4)    : 0x01
    Data type    (5)    : 0x00
    Vbucket      (6,7)  : 0x0000
    Total body   (8-11) : 0x00000001
    Opaque       (12-15): 0xdeadbeef
    CAS          (16-23): 0x0000000000000000
      state      (24)   : 0x4 (dead)
