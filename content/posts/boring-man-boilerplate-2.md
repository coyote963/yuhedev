---
title: "Boring Man Boilerplate Tutorial Part 2"
date: 2020-01-03T20:36:27-05:00
draft: false
toc: false
images:
tags: 
  - boring man
  - python
  - programming
  - rcon
---

Welcome to the second part of my two part Boring Man Boilerplate Rcon tutorial. If you haven't already check out the first part for a basic introduction to interfacing with Boring Man Rewrite servers. This tutorial will show how to do some more advanced things: sending multiple comamnds.

# Command to make someone Overpowered #

For this part we will make a command for users to be able to get every powerup. Let's do this when someone types "!op" in chat. As usual we need to set up the function

```python
def op(event_id, message_string, sock):
  """Handles the case where a user types !op in chat"""
  if event_id == if event_id == rcon_event.chat_message.value:
    js = json.loads(message_string)
    if js['Message'].startswith("!op"):
      # Here we will need to add all the powerups
```
Now we can check the [documentation of the boilerplate](https://github.com/coyote963/bm-boilerplate) and see that the command we want is `powerup : Give a specified player a specified power up. 1 = triple damage, 2 = super speed, 3 = regen, 4 = invis, 5 = bfg [Requires Mutators]` And we can loop through all 5 of them and give to the issuer of the command. 

```python
def op(event_id, message_string, sock):
  """Handles the case where a user types !op in chat"""
  if event_id == rcon_event.chat_message.value:
    js = json.loads(message_string)
    if js['Message'].startswith("!op"):
      for i in range(1, 6):
        send_request(sock, 'powerup "{}" "{}"'.format(js['PlayerID'], i), rcon_receive.command.value)
```

But if we were to test it out, this only gives one powerup. What gives?

The reason is that we are sending out the requests too quickly. So we need to add some delay between the powerups. One way to do this is by importing a library called `time`. And then forcing the code to sleep. To be safe we need to sleep 0.1 seconds.

```python
import time

def op(event_id, message_string, sock):
  """Handles the case where a user types !op in chat"""
  if event_id == rcon_event.chat_message.value:
    js = json.loads(message_string)
    if js['Message'].startswith("!op"):
      for i in range(1, 6):
        send_request(sock, 'powerup "{}" "{}"'.format(js['PlayerID'], i), rcon_receive.command.value)
        time.sleep(0.1)
```

Remember to add it to the functionlist. Hmm... still doesn't really work... it says mutators must be on for the command to be sent. Mutators means that the connected players can't earn experience, but its not a big deal. To enable mutators we just send the command `enablemutators`. A good time to do so is when the boring man rcon client first connects. Add a new handler for when the rcon client connects. Checking Spasman's github for such event yields `rcon_logged_in`.

```python
def enablemutator(event_id, message_string, sock):
  if event_id == rcon_event.rcon_logged_in.value:
    send_request(sock, "enablemutators", rcon_receive.command.value)
```

And we are all set! 

## That's all folks ##

Now if you test it, rcon should give you all five powerups. Let me know what other topics you might want for future RCON lessons but this should be enough for you to write some pretty amazing scripts. Amaze everyone with what you can do!