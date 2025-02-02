# This fork of lua-websockets is adapted to Smartthings Edge usage
## Example usage Smartthings Edge driver
```lua
local ws = require('websocket.client').sync({ timeout = 30 })

local params = {
  mode = "client",
  protocol = "any",
  verify = "none",
  options = "all"
}

function ws_connect()
  local r, code, _, sock = ws:connect('wss://IP:PORT/PATH', 'echo', params)
  print('WS_CONNECT', r, code)

  if r then
    driver:register_channel_handler(sock, function ()
      my_ws_tick()
    end)
  end
end

function my_ws_tick()
  local payload, opcode, c, d, err = ws:receive()
  if opcode == 9.0 then  -- PING 
    print('SEND PONG:', ws:send(payload, 10)) -- Send PONG
  end
  if err then
    ws_connect()   -- Reconnect on error
  end
end

driver:call_with_delay(1, function ()
  ws_connect()
end, 'WS START TIMER')

-- Initialize Driver
driver:run()
```

------------------------------------------------------------------------------
# Not maintained / maintainer wanted !!!!

If someone wants to maintain / take ownership of this project, reach out to me (issue, email). I like Lua very much, but I don't have enough time / resources to stay engaged with it.

# About

This project provides Lua modules for [Websocket Version 13](http://tools.ietf.org/html/rfc6455) conformant clients and servers. 
[![Build Status](https://travis-ci.org/lipp/lua-websockets.svg?branch=master)](https://travis-ci.org/lipp/lua-websockets)
[![Coverage Status](https://coveralls.io/repos/lipp/lua-websockets/badge.png?branch=add-coveralls)](https://coveralls.io/r/lipp/lua-websockets?branch=master)

The minified version is only ~10k bytes in size.

Clients are available in three different flavours:

  - synchronous
  - coroutine based ([copas](http://keplerproject.github.com/copas))
  - asynchronous ([lua-ev](https://github.com/brimworks/lua-ev))

Servers are available as two different flavours:

  - coroutine based ([copas](http://keplerproject.github.com/copas))
  - asynchronous ([lua-ev](https://github.com/brimworks/lua-ev))


A webserver is NOT part of lua-websockets. If you are looking for a feature rich webserver framework, have a look at [orbit](http://keplerproject.github.com/orbit/) or others. It is no problem to work with a "normal" webserver and lua-websockets side by side (two processes, different ports), since websockets are not subject of the 'Same origin policy'.

# Usage
## copas echo server
This implements a basic echo server via Websockets protocol. Once you are connected with the server, all messages you send will be returned ('echoed') by the server immediately.

```lua
local copas = require'copas'

-- create a copas webserver and start listening
local server = require'websocket'.server.copas.listen
{
  -- listen on port 8080
  port = 8080,
  -- the protocols field holds
  --   key: protocol name
  --   value: callback on new connection
  protocols = {
    -- this callback is called, whenever a new client connects.
    -- ws is a new websocket instance
    echo = function(ws)
      while true do
        local message = ws:receive()
        if message then
           ws:send(message)
        else
           ws:close()
           return
        end
      end
    end
  }
}

-- use the copas loop
copas.loop()
```

## lua-ev echo server
This implements a basic echo server via Websockets protocol. Once you are connected with the server, all messages you send will be returned ('echoed') by the server immediately.

```lua
local ev = require'ev'

-- create a copas webserver and start listening
local server = require'websocket'.server.ev.listen
{
  -- listen on port 8080
  port = 8080,
  -- the protocols field holds
  --   key: protocol name
  --   value: callback on new connection
  protocols = {
    -- this callback is called, whenever a new client connects.
    -- ws is a new websocket instance
    echo = function(ws)
      ws:on_message(function(ws,message)
          ws:send(message)
        end)

      -- this is optional
      ws:on_close(function()
          ws:close()
        end)
    end
  }
}

-- use the lua-ev loop
ev.Loop.default:loop()

```

## Running test-server examples

The folder test-server contains two re-implementations of the [libwebsocket](http://git.warmcat.com/cgi-bin/cgit/libwebsockets/) test-server.c example.

```shell
cd test-server
lua test-server-ev.lua
```

```shell
cd test-server
lua test-server-copas.lua
```

Connect to the from Javascript (e.g. chrome's debugging console) like this:
```Javascript
var echoWs = new WebSocket('ws://127.0.0.1:8002','echo');
```

# Dependencies

The client and server modules depend on:

  - luasocket
  - luabitop (if not using Lua 5.2 nor luajit)
  - luasec
  - copas (optionally)
  - lua-ev (optionally)

# Install

```shell
$ git clone git://github.com/lipp/lua-websockets.git
$ cd lua-websockets
$ luarocks make rockspecs/lua-websockets-scm-1.rockspec
```

# Minify

A `squishy` file for [squish](http://matthewwild.co.uk/projects/squish/home) is
provided. Creating the minified version (~10k) can be created with:

```sh
$ squish --gzip
```

The minifed version has be to be installed manually though.


# Tests

Running tests requires:

  - [busted with async test support](https://github.com/lipp/busted)
  - [Docker](http://www.docker.com)

```shell
docker build .
```

The first run will take A WHILE.
