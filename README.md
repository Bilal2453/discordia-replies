# Discordia-Replies

***This have been deprecated! Use Discordia 3 built-in replies implementation instead.***

This is an addon module for the library [Discordia](https://github.com/SinisterRectus/Discordia/) v2, aims at providing support for the new Discord replies. This module will be deprecated when Discordia version 3 is released (which will support replies).

## Index

- [Examples](#examples)
  - [General Examples](#general-examples)
  - [Various Examples](#various-examples)
- [Documentation](#documentation)
  - [The Module](#the-module)
  - [Methods](#methods)
  - [Structures](#structures)

## Examples

### General Examples

- Sending a reply

```lua
local discordia = require("discordia") -- Require Discordia
require("discordia-replies")() -- Patch Discordia

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
-- You could also do:
local discordia = require("discordia-replies")() -- Require and patch Discordia

local client = discordia.Client()
client:on("messageCreate", function(msg)
  if msg.repliesTo and msg.repliesTo.message then
    msg:reply('Detected a reply to a message that said "' .. msg.repliesTo.message.content .. '"!')
  end
end)

client:run('Bot ...')
```

- Different ways of requiring the addon with different options

```lua
local discordia = require("discordia-replies") {
  replyMention = false,
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

### Various Examples

Examples for each use. Sorted by most recommended to least. 

1. Requiring the extension

  - Require then patch Discordia

```lua
local discordia = require("discordia")
require("discordia-replies")()
```

  - Letting the extension require Discordia for you after patching it

```lua
local discordia = require("discordia-replies")()
```

  - Using the extension without patching Discordia (Not recommended. For sending only.)

```lua
local replies = require("discordia-replies")
local reply = replies.reply -- A method that takes a message object and text/table
local reply = replies.send  -- A method that takes a channel object and text/table
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

  - Using the returned `reply` or `send` function

```lua
reply(message, "Hello World!")
send(channel, "Accept a table value too!")
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

  - Patching Discordia with your own index for the patched `Message:newReply()`

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

## Documentation

When used with default options, this module will mainly do two things A) will add `Message:newReply` method B) will add `Message.repliesTo` getter. To do that the extension must patch and inject some stuff into Discordia, mainly into the `Message` class, optionally the `TextChannel` class.

### The module

When required, this module will return a table value that contains the following proprieties:

| index     | type       | description   |
| --------- | ---------- | :------------ |
| reply     | function   | A function that accepts two parameters, the Message object you want to reply to, plus the message content (which can be a string or a table, see [newReply](#messagenewreplymessage-content)). |
| send      | function   | See [TextChannel.text](https://github.com/SinisterRectus/Discordia/wiki/textChannel#sendcontent) documentation.<br>This only differs from `TextChannel:send` in one thing, that it supports `message_reference` & `allowed_mentions` fields.|
| module    | string     | The Git repo name of this module. |
| version   | string     | This module's current version. |

The returned table also have a `__call` meta-method set, when called, it can optionally accept a table value for further configuration of how the module behave (See [Options](#options)), and will do the necessarily injections. It will also return a single table value, which is the patched Discordia module (can be ignored).

### Methods

#### Message:newReply(message, content)

Sends a new message with a Discord reply referring to `Message`.  

| param   | type     | description |
|---------|----------|:------------|
| message | [Message](https://github.com/SinisterRectus/Discordia/wiki/message)  | The message you are replying to. |
| content | string/table | The reply contents for the reply message. Regular `TextChannel:send()` rules apply (See [TextChannel.send](https://github.com/SinisterRectus/Discordia/wiki/TextChannel#sendcontent)). Using this makes `message_reference` & `allowed_mentions` available for use.

Note: This method is almost identical to `Message.reply` except it pre-sets `message_reference` for you, therefor you can read Discordia wiki for more information about it.

Note: Unlike Discordia's `send`, this method will try to cast parameter `content` to string if it wasn't a string/table value already.

----

### Structures

#### Message.repliesTo

A table value that represents a reply associated with `Message`. If the message does not reply to anything **this will be nil**.

| index   | type     | description |
|---------|----------|:------------|
| message | [Message](https://github.com/SinisterRectus/Discordia/wiki/message)/nil | The message object this message replies to, if the message was deleted (or cannot be fetched for some reason) this will be nil. Note that this module will try to fetch the message if it wasn't cached already by default (See [Options](#options) if you wish to change this behavior). |
| channel | [TextChannel](https://github.com/SinisterRectus/Discordia/wiki/TextChannel) | The TextChannel object of where the message was sent. |
| guild   | [Guild](https://github.com/SinisterRectus/Discordia/wiki/guild) | The Guild object of where the reply channel is located under. |
| client  | [Client](https://github.com/SinisterRectus/Discordia/wiki/Client) | The respective client object. |

#### Options

A table value that allows the user to provide further configuration and customization of how the module behave.

| index           | type     | default    | description |
|-----------------|----------|------------|:------------|
| replyIndex      | any      | `newReply` | What index to assign the reply method to under the `Message` class. |
| sendIndex       | any      | `send`     | If `patchSend` is true, this will specify what index to assign the module patched version of `TextChannel.send` to under TextChannel class. |
| replaceOriginal | boolean  | `false`    | Whether or not to patch `Message:reply` with `newReply`. |
| replyMention    | boolean  | `true`     | Whether or not to mention the user you're replying to. |
| patchGetters    | boolean  | `true`     | Whether or not to patch in the `repliesTo` getter. |
| fetchMessage    | boolean  | `false`    | Whether or not try to fetch the replied to message object if wasn't already cached. |
| failIfNotExists | boolean  | `true`     | Passes `fail_if_not_exists` for all replies requests. See [Discords docs](https://discord.com/developers/docs/resources/channel#message-reference-object-message-reference-structure) for more info. |
