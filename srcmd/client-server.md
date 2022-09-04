---
title: "Client-Server Interactions in BreadQuest"
date: 2022-09-03T22:03:36-00:00
draft: true
---

The interactions require HTTP POST, cookies, and WebSockets.

## Authentication

Basically, a client sends a POST request to the BreadQuest server which contains the username and password, and the server responds with a "set-cookie" header.  This cookie will then be used for opening the WebSocket for playing the game.

The payload for the POST request is sent to `/loginAction`.  Its payload (body) looks like this:

    username=a&password=a

Additionally, the `Content-Length` header must be in accord with the payload, and I think the ostracod server is using HTTP/1.1.

This means that the whole POST request looks like this:

    POST /loginAction HTTP/1.1
    Host: ostracodapps.com:2626
    Content-type: application/x-www-form-urlencoded
    Content-Length: 21
    
    username=a&password=a

(with CRLF / 0d0a / \r\n, of course)

After this, you'll get a response with a `set-cookie` header.

I have no idea how to create a WebSocket in your language (in Rust, I used `tokio-tungstenite`), but you'll need to use that cookie.

NOTE: If you're building an in-browser client, it'll use the cookie automatically if you're logged in on the server's website, but you will probably need to disable some securiy features.  If you're logged in at the official server, but not [my web client](https://munvoseli.github.io/breadquest-client/socket.html), that security has yet to be disabled.

The default client is served at `/game`, but the WebSocket needs to be at `/gameAction`.

## The WebSocket and JSON

Here, you'll be making requests on the WebSocket.  They are written in JSON.  Commands come a bit later, but some are listed here.

Here is something that the server might send to the client.  It is almost a real request, it's just that `setTiles` has been abbreviated, there were actually five `addEntity` commands, and formatting stuffs.

    {
        "success": true,
        "commandList":[
            {"commandName":"removeAllEntities"},
            {
                "commandName":"addEntity",
                "entityInfo":{
                    "className":"Enemy",
                    "id":26693,
                    "pos":{"x":-3017,"y":-2236}
                }
            },
            {
                "commandName":"addEntity",
                "entityInfo":{
                    "className":"Enemy",
                    "id":26694,
                    "pos":{"x":-3042,"y":-2291}
                }
            },
            {
                "commandName":"setTiles",
                "pos":{"x":-3048,"y":-2280},
                "tileList":[143,143,128...143,143],
                "size":30
            },
            {"commandName":"removeAllOnlinePlayers"},
            {
                "commandName":"addOnlinePlayer",
                "username":"Melody"
            },
            {
                "commandName":"setStats",
                "health":5,
                "isInvincible":false
            }
        ]
    }

Here's something that a client might send to the server:

    [
        {
            "commandName":"assertPos",
            "pos":{"x":-3033,"y":-2265}
        },
        {"commandName":"getEntities"},
        {"commandName":"getTiles","size":30},
        {"commandName":"getChatMessages"},
        {"commandName":"getOnlinePlayers"},
        {"commandName":"getInventoryChanges"},
        {"commandName":"getRespawnPosChanges"},
        {"commandName":"getStats"},
        {"commandName":"getAvatarChanges"}
    ]

## The commands

This section will detail the commands the server and client say to each other.

The client-to-server commands are listed in the code [here](https://github.com/ostracod/breadquest/blob/master/utils/game.js#L174).

The server-to-client commands are listed in the code [here](https://github.com/ostracod/breadquest/blob/master/public/javascript/game.js#L113).

I won't get to this now.  It's on the to-do list.
