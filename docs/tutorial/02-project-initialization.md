---
title: "Tutorial step #1 - Project initialization"
sidebar_label: "Step #1: Project initialization"
slug: step-1
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Project initialization

The first goal is to set up a simple HTML webpage that serves out a form and a list of messages. We’re going to use the Node.JS web framework `express` to this end. Make sure [Node.JS](https://nodejs.org) is installed.

First let’s create a `package.json` manifest file that describes our project. I recommend you place it in a dedicated empty directory (I’ll call mine `socket-chat-example`).

<Tabs groupId="lang">
  <TabItem value="cjs" label="CommonJS" default>

```json
{
  "name": "socket-chat-example",
  "version": "0.0.1",
  "description": "my first socket.io app",
  "type": "commonjs",
  "dependencies": {}
}
```

  </TabItem>
  <TabItem value="mjs" label="ES modules">

```json
{
  "name": "socket-chat-example",
  "version": "0.0.1",
  "description": "my first socket.io app",
  "type": "module",
  "dependencies": {}
}
```

  </TabItem>
</Tabs>

:::caution

The "name" property must be unique, you cannot use a value like "socket.io" or "express", because npm will complain when installing the dependency.

:::

Now, in order to easily populate the `dependencies` property with the things we need, we’ll use `npm install`:

```
npm install express@4
```

Once it's installed we can create an `index.js` file that will set up our application.

<Tabs groupId="lang">
  <TabItem value="cjs" label="CommonJS" default>

```js
const express = require('express');
const { createServer } = require('node:http');

const app = express();
const server = createServer(app);

app.get('/', (req, res) => {
  res.send('<h1>Hello world</h1>');
});

server.listen(3000, () => {
  console.log('server running at http://localhost:3000');
});
```

  </TabItem>
  <TabItem value="mjs" label="ES modules">

```js
import express from 'express';
import { createServer } from 'node:http';

const app = express();
const server = createServer(app);

app.get('/', (req, res) => {
  res.send('<h1>Hello world</h1>');
});

server.listen(3000, () => {
  console.log('server running at http://localhost:3000');
});
```

  </TabItem>
</Tabs>

This means that:

- Express initializes `app` to be a function handler that you can supply to an HTTP server (as seen in line 5).
- We define a route handler `/` that gets called when we hit our website home.
- We make the http server listen on port 3000.

If you run `node index.js` you should see the following:

<img src="/images/chat-1.png" alt="A console saying that the server has started listening on port 3000" />

And if you point your browser to `http://localhost:3000`:

<img src="/images/chat-2.png" alt="A browser displaying a big 'Hello World'" />

So far, so good!

:::info

<Tabs groupId="lang">
  <TabItem value="cjs" label="CommonJS" default attributes={{ className: 'display-none' }}>

You can run this example directly in your browser on:

- [CodeSandbox](https://codesandbox.io/p/sandbox/github/socketio/chat-example/tree/cjs/step1?file=index.js)
- [StackBlitz](https://stackblitz.com/github/socketio/chat-example/tree/cjs/step1?file=index.js)


  </TabItem>
  <TabItem value="mjs" label="ES modules" attributes={{ className: 'display-none' }}>

You can run this example directly in your browser on:

- [CodeSandbox](https://codesandbox.io/p/sandbox/github/socketio/chat-example/tree/esm/step1?file=index.js)
- [StackBlitz](https://stackblitz.com/github/socketio/chat-example/tree/esm/step1?file=index.js)


  </TabItem>
</Tabs>

:::
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Multiplayer Car Racing</title>
  <style>
    body {
      background: #111;
      margin: 0;
      font-family: sans-serif;
      overflow-x: hidden;
    }
    #gameArea {
      width: 400px;
      height: 600px;
      margin: 20px auto;
      background: repeating-linear-gradient(to bottom, #333 0 40px, #444 40px 80px);
      position: relative;
      border: 2px solid #fff;
      overflow: hidden;
    }
    .car {
      width: 50px;
      height: 100px;
      border-radius: 10px;
      position: absolute;
      bottom: 20px;
      transition: left 0.1s linear;
    }
    #info {
      color: white;
      text-align: center;
      margin-bottom: 10px;
      font-size: 20px;
    }
    .controls {
      width: 400px;
      margin: 10px auto;
      display: flex;
      justify-content: center;
      gap: 20px;
    }
    button {
      padding: 10px 20px;
      font-size: 20px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      user-select: none;
    }
  </style>
</head>
<body>
  <div id="info">Multiplayer Car Racing</div>
  <div id="gameArea"></div>
  <div class="controls">
    <button id="leftBtn">◀️</button>
    <button id="rightBtn">▶️</button>
  </div>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io();
    const gameArea = document.getElementById("gameArea");
    const leftBtn = document.getElementById("leftBtn");
    const rightBtn = document.getElementById("rightBtn");

    const players = {};

    function createCar(id, color) {
      const car = document.createElement("div");
      car.classList.add("car");
      car.style.background = color;
      car.style.left = "175px";
      gameArea.appendChild(car);
      players[id] = car;
    }

    function removeCar(id) {
      if(players[id]) {
        gameArea.removeChild(players[id]);
        delete players[id];
      }
    }

    function updatePosition(id, pos) {
      if(players[id]) {
        players[id].style.left = pos.x + "px";
      }
    }

    socket.on("currentPlayers", (serverPlayers) => {
      for (const id in serverPlayers) {
        if (!players[id]) {
          createCar(id, id === socket.id ? "red" : "yellow");
          updatePosition(id, serverPlayers[id]);
        }
      }
    });

    socket.on("newPlayer", ({ id, position }) => {
      createCar(id, "yellow");
      updatePosition(id, position);
    });

    socket.on("playerMoved", ({ id, position }) => {
      updatePosition(id, position);
    });

    socket.on("playerDisconnected", (id) => {
      removeCar(id);
    });

    function moveLeft() {
      socket.emit("move", "left");
    }
    function moveRight() {
      socket.emit("move", "right");
    }

    document.addEventListener("keydown", (e) => {
      if(e.key === "ArrowLeft") moveLeft();
      else if(e.key === "ArrowRight") moveRight();
    });

    leftBtn.addEventListener("click", moveLeft);
    rightBtn.addEventListener("click", moveRight);
  </script>
</body>
</html>
npm init -y
npm install express socket.io
node server.js
