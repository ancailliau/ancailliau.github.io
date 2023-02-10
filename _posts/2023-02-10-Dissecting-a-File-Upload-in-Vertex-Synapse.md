---
title: Dissecting a File Upload in Vertex Synapse
tags: synapse storm
---

In this post, we'll explore how files get stored in a Vertex Axon and recorded
on the Vertex Synapse platform. The Axon is a tool for storing binary data
securely within the Synapse framework; it indexes binaries using SHA-256 hash
so that no duplicate storage occurs. By default, blobs are stored inside an
LMDB Slab.

I created this post to gain a more extensive comprehension of file uploads,
allowing me to rebuild the feature within my custom Telepath client in C#. In
a following blog entry, I will look into how one can download files from
Synapse/Axon.

## LMDB?

LMDB (Lightning Memory-Mapped Database) is a powerful software library that
facilitates lightning-fast transactions for applications requiring massive
amounts of data storage and access. It's specifically designed to be fast,
reliable, and endlessly scalable - rendering it the optimal selection for
programs needing substantial performance levels with minimal latency.

LMDB employs a revolutionary technique known as memory mapping, which gives the
system permission to process a file just like it were part of its internal
memory. As a result, it becomes simple and quick for users to access data
stored in the database since no copying or transferring is needed — data can be
acquired directly from memory!

The transactional nature of LMDB is integral, providing a reliable way to
manage database updates and ensure that they remain consistent. This durability
is especially essential in applications where data integrity can’t be
compromised, as it guarantees the preservation of an unchanged state - even if
there's a power failure or system crash!

LMDB offers an impressive, dynamic database solution fit for applications that
need rapid and reliable access to enormous amounts of data. This technology is
utilized all over the world in a wide range of contexts, from advanced web apps
to embedded systems and even databases themselves.

## Test bed

For the purpose of this post, we will be uploading two files. The first one is
an empty file named `test.txt` and contains nothing inside it; while the second
one is titled `hello-world.txt`, which only has a string that says `Hello
World!`.

```
$ shasum -a 256 test.txt 
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855  test.txt
$ shasum -a 256 hello-world.txt 
7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069  hello-world.txt
```

To initiate transmission of our two data files, we first launched a Vertex
Synapse node (via Docker) on port 8903 to interact with the Telepath API. We
then fired up Wireshark to capture the traffic between server and client before
running the following commands in a Storm client:

```
$ python3 -m synapse.tools.storm tcp://root:***@localhost:8903/

Welcome to the Storm interpreter!

Local interpreter (non-storm) commands may be executed with a ! prefix:
    Use !quit to exit.
    Use !help to see local interpreter commands.

storm> !pushfile test.txt
uploading file: test.txt
.........
file:bytes=sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
        :name = test.txt
        :sha256 = e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
        .created = 2023/02/10 07:32:06.671
complete. 1 nodes in 105 ms (9/sec).
storm>  !pushfile hello-world.txt
uploading file: hello-world.txt
.........
file:bytes=sha256:7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069
        :name = hello-world.txt
        :sha256 = 7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069
        .created = 2023/02/10 08:01:12.755
complete. 1 nodes in 25 ms (40/sec).

storm> !quit
o/
```

We paused the Wireshark capture and scrutinized the relevant packets, paying
special attention to their msgpack-encoded messages. By copying these as a hex
stream in Wireshark, we were able to [decode them
online](https://msgpack.solder.party/). with ease.

## Exchanges Telepath Messages

To begin, we must fetch a handle to write onto and save. The best way to do
this is by sending out the `t2:init` message for the method [`getAxonUpload`](https://synapse.docs.vertex.link/en/latest/synapse/autodocs/synapse.html?highlight=getAxonUpload#synapse.cortex.CoreApi.getAxonUpload). With
that done, we can write our desired data.

```
[
  "t2:init",
  {
    "todo": [
      "getAxonUpload",
      [],
      {}
    ],
    "name": null,
    "sess": "865026c78fe7e6fa304394d460738505"
  }
]
```

The server will return a response containing various key values, with one of
the most important being `iden` which we'll need to upload our file.

```
[
  "t2:share",
  {
    "iden": "730bc4268e211064c973c27510acd2d1",
    "sharinfo": {
      "meths": {},
      "syn:commit": "92fc8df3c249f39c26d76ff2db2a48b2b5abd41f",
      "syn:version": [
        2,
        122,
        0
      ],
      "classes": [
        "synapse.axon.UpLoadProxy",
        "synapse.lib.share.Share",
        "synapse.lib.base.Base"
      ]
    }
  }
]
```

To start, we will be uploading the empty file `test.txt`. We have nothing to
write in this case, thus all that is needed is a call of `save` on our handle
(which you should remember as `synapse.axon.UpLoadProxy`).

```
[
  "t2:init",
  {
    "todo": [
      "save",
      [],
      {}
    ],
    "name": "730bc4268e211064c973c27510acd2d1",
    "sess": "865026c78fe7e6fa304394d460738505"
  }
]
```

The server provides the following affirmative notifications, confirming that we
have completed our task.

```
[
  "t2:fini",
  {
    "retn": [
      true,
      [
        0,
        {
          "type": "Buffer",
          "data": [
            227, 176, 196, 66, 152, 252, 28, 20, 154, 251, 244, 200, 153, 111, 
            185, 36, 39, 174, 65, 228, 100, 155, 147, 76, 164, 149, 153, 27, 
            120, 82, 184, 85
          ]
        }
      ]
    ]
  }
]
[
  "share:fini",
  {
    "share": "730bc4268e211064c973c27510acd2d1"
  }
]
```

The value in the field `data` provides us with the SHA256 hash of the file,
as you can decode for example with [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Decimal('Comma',false)To_Hex('None',0)&input=MjI3LCAxNzYsIDE5NiwgNjYsIDE1MiwgMjUyLCAyOCwgMjAsIDE1NCwgMjUxLCAyNDQsIDIwMCwgMTUzLCAxMTEsIDE4NSwgMzYsIDM5LCAxNzQsIDY1LCAyMjgsIDEwMCwgMTU1LCAxNDcsIDc2LCAxNjQsIDE0OSwgMTUzLCAyNywgMTIwLCA4MiwgMTg0LCA4NQ).

For the second instance, we are uploading a text file titled `hello-world.txt`
which isn't blank; it carries the string `Hello World!` that is translated as:

```
$ hexdump -C hello-world.txt
00000000  48 65 6c 6c 6f 20 57 6f  72 6c 64 21              |Hello World!|
0000000c
```

That is, in decimal

```
$ hexdump -e'"%07.8_ad  " 8/1 "%d " "  " 8/1 "%d " "  |"' -e'16/1  "%_p"  "|\n"' hello-world.txt 
00000000  72 101 108 108 111 32 87 111 114 108 100 33      |Hello World!|
```

By using the `write` method on our handle, we can store the bytes contained in
the field `data`, which are expressed as decimals.

```
[
  "t2:init",
  {
    "todo": [
      "write",
      [
        {
          "type": "Buffer",
          "data": [
            72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100, 33
          ]
        }
      ],
      {}
    ],
    "name": "730bc4268e211064c973c27510acd2d1",
    "sess": "865026c78fe7e6fa304394d460738505"
  }
]
```

After executing the operation, we received confirmation that it was a success.

```
[
  "t2:fini",
  {
    "retn": [
      true,
      null
    ]
  }
]
```

We can now save the file, like demonstrated above with the empty file.

Uploading the file to Axon does not register in Synapse's server. Thus, there
is a need for us to record a `file:bytes` node in Synapse and we do this by
sending a `t2:init` message for the
[`storm`](https://synapse.docs.vertex.link/en/latest/synapse/autodocs/synapse.html?highlight=write#synapse.cortex.CoreApi.storm) method with query `[
file:bytes=$sha256 ] { -:name [ :name = $name] }` along with its corresponding
mapping variable.


```
[
  "t2:init",
  {
    "todo": [
      "storm",
      [
        "[ file:bytes=$sha256 ] { -:name [:name=$name] }"
      ],
      {
        "opts": {
          "repr": true,
          "vars": {
            "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
            "name": "test.txt"
          }
        }
      }
    ],
    "name": null,
    "sess": "865026c78fe7e6fa304394d460738505"
  }
]
```

In return, we get a generator and the node. For brevity in our explanation,we
will omit any message related to node-edits from this output.

```
[
  "t2:genr",
  {}
]
[
  "t2:yield",
  {
    "retn": [
      true,
      [
        "init",
        {
          "tick": 1676014326577,
          "text": "[ file:bytes=$sha256 ] { -:name [:name=$name] }",
          "task": "453997a728583c8038263f8116cab3ca"
        }
      ]
    ]
  }
]
...
(skipping two node-edit message)
...
[
  "t2:yield",
  {
    "retn": [
      true,
      [
        "node",
        [
          [
            "file:bytes",
            "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
          ],
          {
            "iden": "3fda2bf653ccbf15df887152bcf2fa86a0a95375cefd45e47dbe1f76051e2b87",
            "tags": {},
            "props": {
              ".created": 1676014326671,
              "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
              "name": "test.txt"
            },
            "tagprops": {},
            "nodedata": {},
            "reprs": {
              ".created": "2023/02/10 07:32:06.671"
            },
            "tagpropreprs": {},
            "path": {}
          }
        ]
      ]
    ]
  }
]
...
[
  "t2:yield",
  {
    "retn": [
      true,
      [
        "fini",
        {
          "tock": 1676014326682,
          "took": 105,
          "count": 1
        }
      ]
    ]
  }
]
```

Our Synapse instance has just added a new node, which our `t2:yield` with node
details. We can verify that the files exist in our client:

```
storm> file:bytes
file:bytes=sha256:7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069
        :name = hello-world.txt
        :sha256 = 7f83b1657ff1fc53b92dc18148a1d65dfc2d4b1fa3d677284addd200126d9069
        .created = 2023/02/10 08:01:12.755
file:bytes=sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
        :name = test.txt
        :sha256 = e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
        .created = 2023/02/10 07:32:06.671
complete. 2 nodes in 5 ms (400/sec).
```

It is possible to create a model of any given file in Synapse without having to
manually upload the file. This comes in handy when you are working on malicious
intrusions, for example: *“sample xxx drops yyy”* where both *xxx* and *yyy*
refer to specific hashes. We can then design the files as `file:bytes` with
their corresponding hash properties set up. If at some point we decide to
upload these files (e.g., from VirusTotal), Synapse will automatically detect
this and avoid recreating it if its original hashes match those on record.
