
// server.js (Heroku-compatible Node.js server with WebSockets)
const WebSocket = require('ws');
const express = require('express');
const http = require('http');
const app = express();

const port = process.env.PORT || 8080;
const server = http.createServer(app);
const wss = new WebSocket.Server({ server });

let players = [];
let currentBid = 0;
let currentBidder = null;

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    const data = JSON.parse(message);

    switch (data.type) {
      case 'join':
        players.push({ id: ws, name: data.name, budget: 100, team: [] });
        broadcast({ type: 'player-list', players: players.map(p => p.name) });
        break;

      case 'bid':
        if (data.amount > currentBid) {
          currentBid = data.amount;
          currentBidder = data.name;
          broadcast({ type: 'bid-update', amount: currentBid, bidder: currentBidder });
        }
        break;

      case 'finalize-auction':
        finalizeAuction();
        break;
    }
  });

  ws.on('close', () => {
    players = players.filter(p => p.id !== ws);
    broadcast({ type: 'player-list', players: players.map(p => p.name) });
  });
});

function broadcast(data) {
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(data));
    }
  });
}

function finalizeAuction() {
  broadcast({ type: 'auction-end', message: 'Auction is over! Match simulation will begin.' });
}

app.use(express.static('public'));  // Serve static files from the "public" directory

server.listen(port, () => {
  console.log(`WebSocket server is running on http://localhost:${port}`);
});{
  "name": "ipl-auction-game",
  "version": "1.0.0",
  "description": "Multiplayer IPL Auction Game using Node.js and WebSockets",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.17.1",
    "ws": "^7.4.6"
  },
  "engines": {
    "node": ">=12.0.0"
  }
}
# IPL Auction Game

A multiplayer IPL Auction game where players can participate in an auction and simulate matches based on the chosen players' skills.

## How to Play
1. Each player starts with a budget of 100 crores.
2. Join the auction and bid on players.
3. Finalize the auction and simulate a cricket match based on the chosen players.

## One-Click Deploy to Heroku
Click the button below to deploy the game to Heroku instantly.

[![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

## Requirements
- Node.js 12+
- WebSockets for real-time communication

## Local Installation
1. Clone the repository:
    ```bash
    git clone https://github.com/yourusername/ipl-auction-game.git
    cd ipl-auction-game
    ```
2. Install dependencies:
    ```bash
    npm install
    ```
3. Start the server:
    ```bash
    npm start
    ```
