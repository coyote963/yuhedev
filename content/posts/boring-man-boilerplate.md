---
title: "Boring Man Boilerplate Tutorial Part One"
date: 2020-01-02T21:07:55-05:00
draft: false
toc: false
images:
tags: 
  - boring man
  - python
  - programming
  - rcon
---
This tutorial will show you how to use the Boring Man Boilerplate (bm-boilerplate).

## Install the boilerplate ##
First install the boilerplate. You need to first have git installed, get it [here](https://git-scm.com/downloads). Once you have it installed, open a cmd prompt and enter
```
git clone https://github.com/coyote963/bm-boilerplate
cd bm-boilerplate
```

For this tutorial we will be making a rcon script that allows you to skip to a survival wave when a user types a command. This will show the basic functionality of the code.

## A look inside the installed boilerplate ##
Here is a list of files that are in the boilerplate. Try to follow along as I demonstrate the purpose of every file. 

The `bm_parser.py` has the function start_parser, which loops forever taking in data from a server. This function allows you to pass a list of functions (handlers) that *handle* the different server events.

`startparsing.py` is the code you will execute to run all your handlers. You can see that this is multithreaded to allow you to run rcon clients on multiple servers (gamemodes). 

That brings us to the `parseconfigs.py` file. This file has all the settings that change the behavior of the application. The ip address is by default set to `127.0.0.1` which allows you to connect to a server on your own computer. `gamemode_ports` has only a single entry: `'svl' : 42070`, but if you have multiple servers feel free to add multiple. Also notice how `'svl'` matches the array of strings in `startprocessing.py`. `blocking` shouldn't be touched unless you know what you are doing. 

## Actually writing the application ##

Lets open `exampleapp.py` and see what we have there:

```python
# import the enum type for rcon events
from rcontypes import rcon_event
# import json parsing to translate server messages into JSON type
import json

def handle_chat(event_id, message_string, sock):
    # if passed in event_id is a chat_message
    if event_id == rcon_event.chat_message.value:
        # parse the json
        js = json.loads(message_string)
        # if the server message is from a player
        if 'PlayerID' in js and js['PlayerID'] != '-1':
            # print it into console
            print("{} said {}".format(js['Name'],js['Message']))

example_functions = [handle_chat] # include handle_cache if you are using it
```

First lets add a function that replies to users in game! Define a function in the file and call it `ping`:

```python
def ping(event_id, message_string, sock): 
```

A common mistake is not to include your defined functions in the `example_functions` and then wonder why your function never gets called. Change it to

```python
example_functions = [handle_chat, ping]
```

Here `event_id` gets passed into all the functions, telling the function what *kind* of packet it is. The game has over 70 different packet types and you want your function to only deal with the packet type it wants. The information on each packet type is documented [here](https://github.com/Spasman/rcon_example). When the event_id comes into the function it is stored as an `int`, but in the boilerplate we store the different event types as `enums`. Luckily every `enum` has a value attribute which converts them into their `int` value. If the packet isn't a `chat_message` we don't do anything.

```python
def ping(event_id, message_string, sock):
  """Responds in game with pong when user types !ping"""
  if event_id == rcon_event.chat_message.value:
    js = json.loads(message_string)
    if 'PlayerID' in js and js['PlayerID'] != -1:
      if js['Message'].startswith("!ping")
```

Whew, thats a lot of ifs, but here in the nested if, we can tell the rcon code to tell the game to respond with Pong!

But in order to do that we need some additional libraries. At the top of the file add this:

```python
# import another enum from rcontypes
from rcontypes import rcon_event, rcon_receive
from helpers import send_request
```

When sending commands to the servers you can use the function called `send_packet`. It takes 3 arguments, the first is a socket object. Don't worry about this, sock is passed into `ping` and is made for you. The second argument is the data you want to send the server, and the last value is telling the server what kind of comamnd it is. For the data we need to tell the server to `rawsay` (tell everyone) Pong! A full list of the commands are found at the [boilerplate github page](https://github.com/coyote963/bm-boilerplate) So the command is `rawsay Pong!`. `rawsay` command takes two parameters that are stored in double quotes. The first one is the actual message and the second is the color you want it to be displayed in. 

Hmm.. what color am I feeling today... lets take a look at the [gamemaker documentation](https://docs2.yoyogames.com/source/_build/3_scripting/4_gml_reference/drawing/colour/index.html). I am feeling like olive. Olive's value is 32896.

```python
def ping(event_id, message_string, sock):
  """Responds in game with pong when user types !ping"""
  if event_id == rcon_event.chat_message.value:
    if 'PlayerID' in js and js['PlayerID'] != -1:
      if js['Message'].startswith("!ping"):
        send_request(sock, 'rawsay "Pong!" "32896"', rcon_receive.command.value)
```

Ok... but this isn't very good. If someone says !ping the entire server shouldn't hear the Pong! Lets have it so that the server only talks to the person sending the !ping command. Again, let's consult the [bm-boilerplate readme](https://github.com/coyote963/bm-boilerplate). Seems like `pm` does exactly what we need. so lets modify the body of the function:

```python
send_request(sock, 'pm "{}" "Pong!" "32896"'.format(js['PlayerID']), rcon_receive.command.value)
```
Here we call format so that the curly brackets `{}` get replaced with the pinger's ID. Awesome now we have enough code to go say ping and pong between users

### Wrap it up ###

So we saw 

1. The main loop of the application
2. The settings file to connect to your game
3. rcontypes file to tell you what kind of packets the game is sending/receiving
4. A single module `exampleapp` that will allow you to attach handlers to all kinds of server events

Here an exercise for the reader. 

1. When a kill occurs tell the killer how many blocks away they were from the victim. 
2. Or when a player takes a player revive mission list out in chat who is the best player to revive and then kill them.
3. Add a greetings message to every user that joins

### What will be covered in Part 2 ###

I will cover 

1. Sending multiple commands in the same handler
2. Storing information on a player in the player cache

Stay tuned 