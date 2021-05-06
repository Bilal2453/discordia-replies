# Discordia-Replies

This is an addon module for the library [Discordia](https://github.com/SinisterRectus/Discordia/) v2, aims at providing support for the new Discord replies. This module will be deprecated when Discordia version 3 is released (which will support replies).

## Documentation

When used with default options, this module will mainly do two things A) will add `Message:newReply` method B) will add `Message.repliesTo` getter.

### The module

When required, this module will return a table value that contains the following proprieties:

|---------|----------|:------------|
| index   | type     | description |
| reply   | function | A function that accepts two parameters, the message object you want to reply to, and the message content (which can be a string or a table). |
| module  | string   | The Git repo name of this module. |
| version | string   | This module's current version. |

Even though this module do return `reply` **it isn't the recommended way**. The returned table also have a `__call` meta-method set, when called, it will accept a single optional parameter of type table, that should be either the Discordia module you want to patch, or a table for further configuration of how the module behave (See [Options](#options)).

----

### Methods

#### Message:newReply(message, content)

Sends a new message with a Discord reply to `Message`.  

|---------|----------|:------------|
| param   | type     | description |
| message | Message  | The message you are replying to. |
| content | string/table | The reply contents for the reply message. A string value means the content is string only, identical to `{content = value}`, a table value can be passed for further options, (See [TextChannel.send](#WIP)).

Note: This method is almost identical to `TextChannel.send` except it pre-sets `message_reference` for you, therefor you can read Discordia wiki for more information about `TextChannel.send`.
Note: Unlike Discordia's `send`, this method will try to cast parameter `content` to string if it wasn't a string/table value. 

----

### Structures

#### Message.repliesTo

A table value that represents a reply associated with `Message`. If the message does not reply to anything **this will be nil**.

|---------|----------|:------------|
| index   | type     | description |
| message | Message/nil  | The message object this message replies to, if the message was deleted (or cannot be fetched for some reason) this will be nil. Note that this module will try to fetch the message if it wasn't cached already by default (See [Options](#options) if you wish to change that behavior). |
| channel | TextChannel | The TextChannel object of where the message was sent. |
| guild   | Guild    | The Guild object of where the reply channel is located under. |
| client  | Client   | The respective client object. |

#### Options

A table value that allows the user to provide further configuration and customization of how the module behave.

|-----------------|----------|------------|:------------|
| index           | type     | default    | description |
| replyIndex      | any      | `newReply` | What index to assign the reply method to under the `Message` class. |
| sendIndex       | any      | `send`     | If `patchSend` is true, this will specify what index to assign the module patched version of `TextChannel.send` to under TextChannel class. |
| replaceOriginal | boolean  | `false`    | Whether or not to patch `Message:reply` with the module version. |
| patchGetters    | boolean  | `true`     | Whether or not to include the `repliesTo` getter. |
| fetchMessage    | boolean  | `true`     | Whether or not try to fetch the replied to message object if wasn't already cached. |
| failIfNotExists | boolean  | `true`     | Passes `fail_if_not_exists` for all replies requests. See [Discords docs](https://discord.com/developers/docs/resources/channel#message-reference-object-message-reference-structure) for more info. |

## General Examples

- Sending a reply

```lua
local discordia = require("discordia-replies")() -- Require and patch Discordia

local client = discordia.Client()
client:on("messageCreate", function(msg)
  if msg.content == "ping!" then
    msg:newReply("Pong!")
  end
end)

client:run('Bot ...')
```

- Reading a message reply

```lua
local discordia = require("discordia") -- Require Discordia
require("discordia-replies")(discordia) -- Patch Discordia

local client = discordia.Client()
client:on("messageCreate", function(msg)
  if msg.repliesTo and msg.repliesTo.message then
    msg:reply('Detected a reply message that says "' .. msg.repliesTo.message.content .. '"!')
  end
end)

client:run('Bot ...')
```

- Different ways of requiring the addon with different options

```lua
local discordia = require("discordia-replies") {
  patchSend = true,
  replyIndex = "reply_to_this_message_pls"
  -- other options
}

local client = discordia.Client()
client:on("messageCreate", function(msg)
  if msg.content == "ping!" then
    msg:reply_to_this_message_pls("Pong!")
  end
end)

client:run('Bot ...')
```

## Specific Examples

Examples for each use. Sorted by most recommended to least. 

1. Requiring the extension

- Passing Discordia as a parameter

```lua
local discordia = require("discordia")
require("discordia-replies")(discordia)
```

- Letting the extension require Discordia for you

```lua
local discordia = require("discordia-replies")()
```

- Using the extension without patching Discordia (Not recommended. For sending only.)

```lua
local reply = require("discordia-replies").reply
```

- Providing both options and your Discordia

```lua
local discordia = require("discordia")
require("discordia-replies") {
  discordia = discordia,
  patchSend = true,
  -- etc
}
```

2. Replying to messages

Note that in all listed examples a value of table (instead of string) is supported.

- Using the patched `newReply` method (recommended)

```lua
message:newReply("Hello World!")
message:newReply({
  embed = {
    title = "Embeds supported too!"
  }
})
```

- Using the returned `reply` function

```lua
reply(message, "Hello World!")
```

- Using the patched `send` method (Not recommended. Not enabled by default, see [Options](#options).)

```lua
message.channel:send({
  content = "Hello World!",
  messageReference = {message_id = message.id},
})
```

3. Using the module options

Check [Options](#options) for the full usage.

- Patching the original `Message:reply` method

```lua
local discordia = require("discordia-replies") {
  replaceOriginal = true
}

local client = discordia.Client()
client:on("messageCreate", function(msg)
  msg:reply("Hello World!") -- Will do an actual Discord reply
end)

client:run('...')
```

- Patching Discordia with your own index for the patched method

```lua
local discordia = require("discordia-replies") {
  replyIndex = "my_fancy_reply_method_name_that_technically_doesnt_have_to_be_string",
}

local client = discordia.Client()
client:on("messageCreate", function(msg)
  msg:my_fancy_reply_method_name_that_technically_doesnt_have_to_be_string("Hello World!")
  -- Sorry.. no one asked for that I know...
end)

client:run('...')
```

- Patching `TextChannel:send` to support message_reference using your own index

```lua
local discordia = require("discordia-replies") {
  patchSend = true,
  sendIndex = "fancySend"
}

local client = discordia.Client()
client:on("messageCreate", function(msg)
  msg:fancySend({
    content = "Hello World!",
    message_reference = {message_id = message.id}
  })
end)

client:run('...')
```

- Using the module without `.repliesTo` getter (Send only)

```lua
local discordia = require("discordia-replies") {
  patchGetters = false
}
```