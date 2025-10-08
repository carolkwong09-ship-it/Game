# Game
const express = require("express");
const app = express();
const http = require("http").createServer(app);
const io = require("socket.io")(http);
const PORT = process.env.PORT || 3000;

app.use(express.static("public"));

let players = [];

io.on("connection", (socket) => {
  console.log("新玩家連線");

  socket.on("joinGame", (name) => {
    players.push({ id: socket.id, name, score: 0, usedDouble: false, activatedDouble: false });
    io.emit("updateLeaderboard", players);
  });

  socket.on("useDouble", (name) => {
    const player = players.find(p => p.name === name);
    if (player && !player.usedDouble) {
      player.usedDouble = true;
      player.activatedDouble = true;
      socket.emit("doubleActivated");
    }
  });

  socket.on("answer", ({ name, correct }) => {
    const p = players.find(p => p.name === name);
    if (!p) return;
    let points = correct ? 100 : 0;
    if (p.activatedDouble && correct) points *= 2;
    p.score += points;
    p.activatedDouble = false;
    io.emit("updateLeaderboard", players.sort((a,b)=>b.score-a.score));
  });

  socket.on("disconnect", () => {
    players = players.filter(p => p.id !== socket.id);
    io.emit("updateLeaderboard", players);
  });
});

app.get("/", (req,res)=>{
  res.send(`<h2>伺服器運作中 ✅</h2>
            <p>請前往 <a href="/player.html">/player.html</a> 測試</p>`);
});

http.listen(PORT, ()=> console.log("伺服器運作於 port", PORT));
