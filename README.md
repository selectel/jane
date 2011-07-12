JANE
====
JS's Acknowledgement of Native Erlang types
-------------------------------------------

Rationale
---------
Nowadays, Erlang is getting a huge attention. This attention reveals a growing need for an interaction with JavaScript and other languages. Despite of Erlang's concise syntax and neat type annotations (which are used by every conscious Erlang programmer), JavaScript is the only language for the Web (for now, at least). This document represents an attempt to change the way how JS and Erlang interact by specifying how Erlang's type annotated records can be represented using JSON. The translated type information can be used to draw edit forms, check types of ingoing and outgoing messages and control the way how data is represented.

Format
------
The reference implementation takes a .hrl file containing typed record definitions and translates it into a .json file containing JSON-encoded record definitions. Each record is encoded as an entry in a first-level dictionary. Then all fields are specified - again as a dictionary where key is a field name and value is a field specification.

Record field specification consists of "type" field and optional "default" field. "default" field contains a default value (obviously) and "type" field is encoded as follows:

*type name is set as a key, arguments list is set as a value;
*each type argument can be a value or a dictionary that contains type as a key and their arguments list as a value;
*these list+dictionary constructs can go deeper.

By default all record fields contain a union of their actual type and atom "undefined". The reference implementation ignores these unions if flag "ignore_undefined" is set.

Implementation
--------------
Current implementation of JANE is an escript file that can be used inside of rebar post-hook:

```erlang
{post_hooks, [{'compile', './priv/recordparser ignore_undefined include/test.hrl'}]}.
```

By default, all resulting .json files will be located in priv/records/test.json, where test is a name of .hrl file.

Example
-------
Here is an example .hrl file with typed records:

```erlang
-record(params_ping, {host :: nonempty_string()}).
-record(params_tcp, {host :: list(atom()),
                     port = 80 :: pos_integer(),
                     timeout :: pos_integer()}).
```

Here is how it is translated to JSON with ignore_undefined:

```javascript
{
    "params_ping": {
        "host": {
            "type": {
                "nonempty_string": []
            }
        }
    },
    "params_tcp": {
        "host": {
            "type": {
                "list": [
                    {
                        "atom": []
                    }
                ]
            }
        },
        "port": {
            "type": {
                "pos_integer": []
            },
            "default": 80
        },
        "timeout": {
            "type": {
                "pos_integer": []
            }
        }
    }
}
```

And here is without ignore_undefined:

```javascript
{
    "params_ping": {
        "host": {
            "type": {
                "union": [
                    {
                        "atom": [
                            "undefined"
                        ]
                    },
                    {
                        "nonempty_string": []
                    }
                ]
            }
        }
    },
    "params_tcp": {
        "host": {
            "type": {
                "union": [
                    {
                        "atom": [
                            "undefined"
                        ]
                    },
                    {
                        "nonempty_string": []
                    }
                ]
            }
        },
        "port": {
            "type": {
                "pos_integer": []
            },
            "default": 80
        },
        "timeout": {
            "type": {
                "union": [
                    {
                        "atom": [
                            "undefined"
                        ]
                    },
                    {
                        "pos_integer": []
                    }
                ]
            }
        }
    }
}
```

License
-------
Copyright 2011 Selectel. All rights reserved.

Redistribution and use in source and binary forms, with or without modification, are
permitted provided that the following conditions are met:

   1. Redistributions of source code must retain the above copyright notice, this list of
      conditions and the following disclaimer.

   2. Redistributions in binary form must reproduce the above copyright notice, this list
      of conditions and the following disclaimer in the documentation and/or other materials
      provided with the distribution.

THIS SOFTWARE IS PROVIDED BY SELECTEL ``AS IS'' AND ANY EXPRESS OR IMPLIED
WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> OR
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

The views and conclusions contained in the software and documentation are those of the
authors and should not be interpreted as representing official policies, either expressed
or implied, of Selectel.
