<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>MilfCup</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-chart-matrix@1.3.0/dist/chartjs-chart-matrix.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
    <style>
      .glow {
        box-shadow:
          0 0 8px rgba(59, 130, 246, 0.6),
          0 0 16px rgba(99, 102, 241, 0.4);
        transition: all 0.2s ease-in-out;
      }
      .glow:hover {
        box-shadow:
          0 0 12px rgba(59, 130, 246, 0.8),
          0 0 20px rgba(99, 102, 241, 0.6);
        transform: scale(1.05);
      }
      #matchChart,
      #eloChart {
        filter: drop-shadow(0 0 6px #273e64);
      }
    </style>
  </head>
  <body class="bg-gray-900 text-gray-100 p-6 font-sans">
    <canvas id="bgCanvas" class="fixed top-0 left-0 w-full h-full z-0"></canvas>
    <div class="grid gap-6 max-w-5xl mx-auto relative z-10">
      <!-- Add Player -->
      <div class="rounded-2xl shadow-lg bg-gray-800 p-4">
        <h1 class="text-2xl font-bold mb-3">Add Player</h1>
        <div class="flex gap-2">
          <input
            id="playerName"
            type="text"
            placeholder="Player Name"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          />
          <button
            id="addPlayer"
            class="bg-gradient-to-r from-blue-600 to-indigo-500 hover:from-blue-400 hover:to-indigo-400 text-white p-2 rounded w-28 glow text-lg"
          >
            Add
          </button>
        </div>
      </div>
      <!-- Leaderboard -->
      <div class="rounded-2xl shadow-lg bg-gray-800 p-4">
        <h1 class="text-2xl font-bold mb-2">MilfCup Leaderboard</h1>
        <ul id="leaderboard" class="space-y-1 text-lg"></ul>
      </div>
      <!-- Record Match -->
      <div class="rounded-2xl shadow-lg bg-gray-800 p-4">
        <h1 class="text-2xl font-bold mb-3">Record Match (FT5)</h1>
        <div class="flex gap-2 mb-2">
          <select
            id="leftPlayer"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          ></select>
          <select
            id="rightPlayer"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          ></select>
        </div>
        <div class="flex gap-2 mb-2">
          <select
            id="leftScore"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          >
            <option value="0">0</option>
            <option value="1">1</option>
            <option value="2">2</option>
            <option value="3">3</option>
            <option value="4">4</option>
            <option value="5">5</option>
          </select>
          <select
            id="rightScore"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          >
            <option value="0">0</option>
            <option value="1">1</option>
            <option value="2">2</option>
            <option value="3">3</option>
            <option value="4">4</option>
            <option value="5">5</option>
          </select>
        </div>
        <button
          id="recordMatch"
          class="bg-gradient-to-r from-green-600 to-emerald-500 hover:from-green-400 hover:to-emerald-400 text-white p-2 rounded w-full glow text-lg"
        >
          Record
        </button>
        <!-- Match List Table -->
        <div class="mt-2 overflow-x-auto">
          <table class="min-w-full table-auto text-left border border-gray-700">
            <thead>
              <tr class="bg-gray-700">
                <th class="px-2 py-1">Left Player</th>
                <th class="px-2 py-1">Score</th>
                <th class="px-2 py-1">Right Player</th>
                <th class="px-2 py-1">Time</th>
              </tr>
            </thead>
            <tbody id="matchList" class="text-lg"></tbody>
          </table>
        </div>
      </div>
      <!-- Player Stats -->
      <div class="rounded-2xl shadow-lg bg-gray-800 p-4">
        <h1 class="text-2xl font-bold mb-3">Player Statistics</h1>
        <select
          id="statPlayer"
          class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full mb-2 text-lg"
        >
          <option value="">Select Player</option>
        </select>
        <div id="statsOutput" class="text-gray-300 space-y-1 text-lg"></div>
      </div>

      <!-- Match History Graph -->
      <div class="rounded-2xl shadow-lg bg-gray-800 p-4">
        <h1 class="text-2xl font-bold mb-3">Match History Graph</h1>
        <div class="flex gap-2 mb-2">
          <select
            id="graphA"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          ></select>
          <select
            id="graphB"
            class="border border-gray-700 bg-gray-800 text-gray-100 p-2 rounded w-full text-lg"
          ></select>
        </div>
        <div class="w-full h-96">
          <canvas id="matchChart" class="w-full h-full"></canvas>
        </div>
      </div>
      <!-- ELO Over Time Graph -->
      <div class="rounded-2xl shadow-lg bg-gray-800 p-4">
        <h1 class="text-2xl font-bold mb-3">ELO Over Time</h1>
        <div class="w-full h-96">
          <canvas id="eloChart" class="w-full h-full"></canvas>
        </div>
      </div>
    </div>
    <script>
      const K = 32;
      const players = [];
      const matches = [];
      let matchChart, eloChart;

      const playerNameInput = document.getElementById("playerName");
      const addPlayerBtn = document.getElementById("addPlayer");
      const leftPlayer = document.getElementById("leftPlayer");
      const rightPlayer = document.getElementById("rightPlayer");
      const leftScore = document.getElementById("leftScore");
      const rightScore = document.getElementById("rightScore");
      const recordMatchBtn = document.getElementById("recordMatch");
      const matchList = document.getElementById("matchList");
      const graphA = document.getElementById("graphA");
      const graphB = document.getElementById("graphB");
      const statPlayer = document.getElementById("statPlayer");
      const statsOutput = document.getElementById("statsOutput");
      const leaderboard = document.getElementById("leaderboard");
      const ctxMatch = document.getElementById("matchChart").getContext("2d");
      const ctxElo = document.getElementById("eloChart").getContext("2d");

      function saveData() {
        localStorage.setItem("players", JSON.stringify(players));
        localStorage.setItem("matches", JSON.stringify(matches));
      }

      function updatePlayerSelects() {
        const options =
          '<option value="">Select Player</option>' +
          players.map((p) => `<option>${p.name}</option>`).join("");
        [statPlayer, graphA, graphB].forEach(
          (sel) => (sel.innerHTML = options),
        );
        updateLeftRightOptions();
      }

      function updateLeftRightOptions() {
        const leftVal = leftPlayer.value;
        const rightVal = rightPlayer.value;
        leftPlayer.innerHTML =
          `<option value="">Select Left Player</option>` +
          players
            .filter((p) => p.name !== rightVal)
            .map((p) => `<option>${p.name}</option>`)
            .join("");
        rightPlayer.innerHTML =
          `<option value="">Select Right Player</option>` +
          players
            .filter((p) => p.name !== leftVal)
            .map((p) => `<option>${p.name}</option>`)
            .join("");
        leftPlayer.value = leftVal;
        rightPlayer.value = rightVal;
      }

      function updateLeaderboard() {
        leaderboard.innerHTML = players
          .sort((a, b) => b.elo - a.elo)
          .map((p, i) => {
            let bgClass = "";
            if (i === 0) bgClass = "bg-yellow-700/50";
            else if (i === 1) bgClass = "bg-gray-600/40";
            else if (i === 2) bgClass = "bg-gray-600/30";
            return `<li class="flex justify-between items-center p-2 rounded ${bgClass} hover:bg-blue-600/30 transition">
                 <span class="flex items-center gap-2"><span class="w-6 text-gray-400 font-semibold">${i + 1}.</span>${p.name}</span>
                 <div class="flex gap-2">
                   <span class="text-gray-300 font-mono">${Math.round(p.elo)}</span>
                   <button onclick="deletePlayer('${p.name}')" class="text-red-500 hover:text-red-400 font-bold px-2 py-0.5 rounded">✕</button>
                 </div>
               </li>`;
          })
          .join("");
      }

      function deletePlayer(name) {
        if (!confirm(`Delete player "${name}" and all their matches?`)) return;

        // Remove player
        const index = players.findIndex((p) => p.name === name);
        if (index !== -1) players.splice(index, 1);

        // Remove matches involving this player
        for (let i = matches.length - 1; i >= 0; i--) {
          if (matches[i].left === name || matches[i].right === name)
            matches.splice(i, 1);
        }

        updatePlayerSelects();
        updateLeaderboard();
        renderMatches();
        renderChart();
        renderEloChart();
        renderStats();
        saveData(); // persist deletion
      }

      function addPlayer() {
        const name = playerNameInput.value.trim();
        if (!name || players.some((p) => p.name === name)) return;
        if (!confirm(`Add player "${name}"?`)) return;
        players.push({ name, elo: 1500, eloHistory: [1500] });
        playerNameInput.value = "";
        updatePlayerSelects();
        updateLeaderboard();
        saveData();
      }

      function recordMatch() {
        const left = leftPlayer.value,
          right = rightPlayer.value;
        const lScore = parseInt(leftScore.value),
          rScore = parseInt(rightScore.value);
        if (!left || !right || left === right) return;
        if (!((lScore === 5 && rScore < 5) || (rScore === 5 && lScore < 5))) {
          alert("In FT5, one player must reach 5 and the other less than 5.");
          return;
        }
        if (!confirm(`Record match: ${left} ${lScore}-${rScore} ${right}?`))
          return;

        const timestamp = Date.now();
        matches.push({
          left,
          right,
          leftScore: lScore,
          rightScore: rScore,
          timestamp,
        });
        updateElo(
          left,
          right,
          lScore > rScore ? 1 : 0, // leftWin
          rScore > lScore ? 1 : 0, // rightWin
          lScore,
          rScore,
        );

        leftPlayer.value = rightPlayer.value = "";
        leftScore.value = rightScore.value = "0";
        renderMatches();
        renderChart();
        renderEloChart();
        renderStats();
        updateLeaderboard();
        updateLeftRightOptions();

        saveData();
      }

      function updateElo(
        leftName,
        rightName,
        leftWin,
        rightWin,
        leftScore,
        rightScore,
      ) {
        const p1 = players.find((p) => p.name === leftName);
        const p2 = players.find((p) => p.name === rightName);

        // Expected scores
        const expected1 = 1 / (1 + Math.pow(10, (p2.elo - p1.elo) / 400));
        const expected2 = 1 / (1 + Math.pow(10, (p1.elo - p2.elo) / 400));

        // Margin factor
        const margin = Math.abs(leftScore - rightScore);
        const maxScore = 5;
        const scoreMultiplier = 1 + margin / maxScore; // 1–2

        // Apply ELO change safely
        const delta1 = K * (leftWin - expected1) * scoreMultiplier;
        const delta2 = K * (rightWin - expected2) * scoreMultiplier;

        p1.elo += delta1;
        p2.elo += delta2;

        // History
        p1.eloHistory.push(p1.elo);
        p2.eloHistory.push(p2.elo);
      }

      function renderMatches() {
        const lastMatches = matches.slice(-20).reverse();
        matchList.innerHTML = lastMatches
          .map((m) => {
            const time = new Date(m.timestamp).toLocaleString();
            const leftClass =
              m.leftScore > m.rightScore
                ? "text-green-400 font-bold"
                : m.leftScore < m.rightScore
                  ? "text-red-500"
                  : "";
            const rightClass =
              m.rightScore > m.leftScore
                ? "text-green-400 font-bold"
                : m.rightScore < m.leftScore
                  ? "text-red-500"
                  : "";
            return `<tr>
               <td class="px-2 py-1 ${leftClass}">${m.left}</td>
               <td class="px-2 py-1 text-center">${m.leftScore} - ${m.rightScore}</td>
               <td class="px-2 py-1 ${rightClass}">${m.right}</td>
               <td class="px-2 py-1">${time}</td>
               
               
             </tr>`;
          })
          .join("");
      }

      // Match History Chart
      function renderChart() {
        const a = graphA.value,
          b = graphB.value;
        if (!a || !b) return;

        const relevantMatches = matches.filter(
          (m) =>
            (m.left === a && m.right === b) || (m.left === b && m.right === a),
        );

        const labels = relevantMatches.map((m, i) => `Match ${i + 1}`);
        const maxBlocks = 5; // FT5 max score

        const leftData = relevantMatches.map((m) =>
          m.left === a ? m.leftScore : m.rightScore,
        );
        const rightData = relevantMatches.map((m) =>
          m.left === b ? m.leftScore : m.rightScore,
        );

        if (matchChart) matchChart.destroy();

        matchChart = new Chart(ctxMatch, {
          type: "bar",
          data: {
            labels,
            datasets: [
              // For dataset colors
              {
                label: a,
                data: leftData,
                backgroundColor: playerColor(a),
                borderColor: playerColor(a),
                borderWidth: 2,
                stack: "stack1",
                borderRadius: 0,
              },
              {
                label: b,
                data: rightData,
                backgroundColor: playerColor(b),
                borderColor: playerColor(b),
                borderWidth: 2,
                stack: "stack2",
                borderRadius: 0,
              },
            ],
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
              legend: { labels: { color: "#fff" } },
              tooltip: {
                callbacks: {
                  label: function (ctx) {
                    const idx = ctx.dataIndex;
                    const m = relevantMatches[idx];
                    const leftScore = m.left === a ? m.leftScore : m.rightScore;
                    const rightScore =
                      m.left === b ? m.leftScore : m.rightScore;
                    return `${a}: ${leftScore} | ${b}: ${rightScore}`;
                  },
                },
              },
            },
            scales: {
              x: {
                stacked: false,
                ticks: { color: "#eee" },
                grid: { color: "#444" },
              },
              y: {
                stacked: true,
                ticks: { color: "#eee", stepSize: 1 },
                grid: { color: "#444" },
                beginAtZero: true,
                suggestedMax: maxBlocks,
              },
            },
          },
        });
      }

      // ELO Chart
      function renderEloChart() {
        if (players.length === 0) return;

        // X-axis: timestamps of matches
        const timestamps = matches.map((m) => m.timestamp);
        const labels = timestamps.map((t) => {
          const d = new Date(t);
          return `${d.getDate().toString().padStart(2, "0")}/${(d.getMonth() + 1).toString().padStart(2, "0")} ${d.getHours().toString().padStart(2, "0")}:${d.getMinutes().toString().padStart(2, "0")}`;
        });

        const datasets = [];

        players.forEach((p) => {
          const lineColor = playerColor(p.name);
          const pointColor = playerBrightColor(p.name);
          const data = [];
          const pointColors = [];
          const pointRadii = [];

          let eloIndex = 0;
          matches.forEach((m) => {
            if (m.left === p.name || m.right === p.name) {
              data.push(p.eloHistory[eloIndex + 1]);
              pointRadii.push(6);
              pointColors.push(pointColor);
              eloIndex++;
            } else {
              data.push(data[data.length - 1] || 1500);
              pointRadii.push(0);
              pointColors.push("transparent");
            }
          });

          datasets.push({
            label: p.name,
            data,
            fill: false,
            borderColor: lineColor,
            backgroundColor: lineColor, // solid rectangle in legend
            tension: 1,
            stepped: true,
            pointRadius: pointRadii,
            pointHoverRadius: 8,
            borderWidth: 8,
            pointBackgroundColor: pointColors,
          });
        });

        if (eloChart) eloChart.destroy();
        eloChart = new Chart(ctxElo, {
          type: "line",
          data: { labels, datasets },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: {
              legend: {
                labels: {
                  color: "#fff", // white text
                  boxWidth: 40,
                  font: {
                    size: 18,
                  },
                },
              },
              tooltip: { enabled: false },
            },
            interaction: { mode: "nearest", intersect: false },
            scales: {
              x: { ticks: { color: "#eee" }, grid: { color: "#444" } },
              y: { ticks: { color: "#eee" }, grid: { color: "#444" } },
            },
          },
        });
      }

      // Global map to store colors
      const playerColorsMap = {};
      const playerBrightColorsMap = {};

      // Fixed distinct hues for players
      const fixedHues = [
        0, // red
        120, // green
        60, // yellow
        240, // blue
        30, // orange
        280, // purple
        180, // cyan/teal
        320, // magenta
        90, // lime
        15, // pink
        200, // teal-dark
        270, // lavender
        25, // brown
        50, // beige
        0, // maroon (can wrap)
        150, // mint
        60, // olive
        20, // apricot
        240, // navy
        0, // gray
      ];

      // Pick a deterministic hue based on player index
      function getBaseHue(name) {
        const index = players.findIndex((p) => p.name === name);
        if (index === -1) return 0; // fallback
        return fixedHues[index % fixedHues.length];
      }

      // Standard player color
      function playerColor(name) {
        if (playerColorsMap[name]) return playerColorsMap[name];

        const baseHue = getBaseHue(name);
        const saturation = 60 + (Math.abs(baseHue) % 30); // 70-79%
        const lightness = 40 + (Math.abs(baseHue) % 30); // 45-54%
        const color = `hsl(${baseHue}, ${saturation}%, ${lightness}%)`;

        playerColorsMap[name] = color;
        return color;
      }

      // Brighter variant for points/hover
      function playerBrightColor(name) {
        if (playerBrightColorsMap[name]) return playerBrightColorsMap[name];

        const baseHue = (getBaseHue(name) + 15) % 360;
        const saturation = 85 + (Math.abs(baseHue) % 10); // 85-94%
        const lightness = 55 + (Math.abs(baseHue) % 10); // 55-64%
        const color = `hsl(${baseHue}, ${saturation}%, ${lightness}%)`;

        playerBrightColorsMap[name] = color;
        return color;
      }

      // Player Stats
      function renderStats() {
        const name = statPlayer.value;
        if (!name) {
          statsOutput.innerHTML = "";
          return;
        }

        const p = players.find((pl) => pl.name === name);
        if (!p) return;

        const pMatches = matches.filter(
          (m) => m.left === name || m.right === name,
        );

        // Basic stats
        let wins = 0,
          losses = 0,
          roundsWon = 0,
          roundsLost = 0;
        let currentWin = 0,
          longestWin = 0,
          sweeps = 0;
        let leftWins = 0,
          rightWins = 0,
          gamesLeft = 0,
          gamesRight = 0;

        // Opponent stats
        const opponentStats = {};

        pMatches.forEach((m) => {
          const score = m.left === name ? m.leftScore : m.rightScore;
          const oppScore = m.left === name ? m.rightScore : m.leftScore;
          const won = score > oppScore;

          // Win/Loss counts
          wins += won ? 1 : 0;
          losses += won ? 0 : 1;

          // Round totals
          roundsWon += score;
          roundsLost += oppScore;

          // Sweep
          if (won && score === 5 && oppScore === 0) sweeps++;

          // Side stats
          if (m.left === name) {
            gamesLeft++;
            if (won) leftWins++;
          }
          if (m.right === name) {
            gamesRight++;
            if (won) rightWins++;
          }

          // Streaks
          currentWin = won ? currentWin + 1 : 0;
          longestWin = Math.max(longestWin, currentWin);

          // Opponent stats
          const opp = m.left === name ? m.right : m.left;
          if (!opponentStats[opp])
            opponentStats[opp] = { games: 0, totalDiff: 0 };
          opponentStats[opp].games++;
          opponentStats[opp].totalDiff += score - oppScore;
        });

        // Favourite Opponent = most games played against
        let favOpponent = "N/A";
        let maxGames = 0;
        Object.keys(opponentStats).forEach((opp) => {
          if (opponentStats[opp].games > maxGames) {
            maxGames = opponentStats[opp].games;
            favOpponent = opp;
          }
        });

        const totalGames = wins + losses;
        const winPct = totalGames ? ((wins / totalGames) * 100).toFixed(1) : 0;
        const roundPct =
          roundsWon + roundsLost
            ? ((roundsWon / (roundsWon + roundsLost)) * 100).toFixed(1)
            : 0;
        const favSide =
          leftWins + rightWins
            ? leftWins >= rightWins
              ? "Left"
              : "Right"
            : "N/A";

let hardestDiff = Infinity;
let hardestRival = "N/A";
Object.keys(opponentStats).forEach((opp) => {
    const avgDiff = opponentStats[opp].totalDiff / opponentStats[opp].games;
    if (avgDiff < hardestDiff) {
        hardestDiff = avgDiff;
        hardestRival = opp;
    }
});

        const hardestMargin = hardestDiff.toFixed(2);

// Closest rival by smallest magnitude, but keep sign
let closestDiff = Infinity,
    closestRival = "N/A";

Object.keys(opponentStats).forEach((opp) => {
  const avgDiff = opponentStats[opp].totalDiff / opponentStats[opp].games; // signed
  if (Math.abs(avgDiff) < Math.abs(closestDiff)) {
    closestDiff = avgDiff;  // signed
    closestRival = opp;
  }
});
const closestMargin = closestDiff.toFixed(2); // can be negative if they beat you more

// Color: green if positive (you win more), red if negative (you lose more)
const closestColor = closestDiff >= 0 ? "text-green-400" : "text-red-400";



        // Generate HTML with grid & colored cards
        statsOutput.innerHTML = `
           <div class="grid grid-cols-2 md:grid-cols-3 gap-3">
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Games Played:</span><span>${totalGames}</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Games (W/L):</span><span class="text-green-400">${wins}<span class="text-white"> / <span class="text-red-400"> ${losses}</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Win %:</span><span>${winPct}%</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Biggest Win Streak:</span><span>${longestWin}</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Rounds W/L:</span><span class="text-green-400">${roundsWon}<span class="text-white"> / <span class="text-red-400">${roundsLost}</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Round Win %:</span><span>${roundPct}%</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Sweeps (5-0):</span><span>${sweeps}</span></div>   
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Games Left/Right:</span><span>${gamesLeft}/${gamesRight}</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Lucky Side:</span><span>${favSide}</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white"><span>Hardest Rival:</span><span class="text-red-400">${hardestRival} (${hardestMargin})</span></div>
             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white">  <span>Closest Rival:</span><span class="${closestColor}">${closestRival} (${closestMargin})</span>
</div>

             <div class="bg-gray-700 p-2 rounded shadow flex justify-between text-white">  <span>Favourite Rival:</span><span class="text-green-400">${favOpponent} (${maxGames} games)</span></div>
             </div>
         
         
           `;
      }

      // Event Listeners
      addPlayerBtn.addEventListener("click", addPlayer);
      recordMatchBtn.addEventListener("click", recordMatch);
      // For match dropdowns (left/right player)
      [leftPlayer, rightPlayer].forEach((el) =>
        el.addEventListener("change", () => {
          updateLeftRightOptions();
          renderChart();
          renderEloChart();
          renderStats();
        }),
      );

      // For graph dropdowns (graphA / graphB)
      [graphA, graphB].forEach((el) =>
        el.addEventListener("change", () => {
          updateGraphOptions(); // prevent same selection
          renderChart(); // redraw chart
        }),
      );

      // Stats player dropdown
      statPlayer.addEventListener("change", () => {
        renderStats();
      });

      // --- Load saved data on page load ---
      window.addEventListener("DOMContentLoaded", () => {
        const savedPlayers = JSON.parse(
          localStorage.getItem("players") || "[]",
        );
        const savedMatches = JSON.parse(
          localStorage.getItem("matches") || "[]",
        );

        players.push(...savedPlayers);
        matches.push(...savedMatches);

        players.forEach((p) => {
          if (!p.eloHistory || !Array.isArray(p.eloHistory)) {
            p.eloHistory = [p.elo || 1500];
          }
        });

        updatePlayerSelects();
        updateLeaderboard();
        renderMatches();
        renderChart();
        renderEloChart();
      });

      function updateGraphOptions() {
        const aVal = graphA.value;
        const bVal = graphB.value;

        graphA.innerHTML =
          `<option value="">Select Player A</option>` +
          players
            .filter((p) => p.name !== bVal)
            .map((p) => `<option>${p.name}</option>`)
            .join("");

        graphB.innerHTML =
          `<option value="">Select Player B</option>` +
          players
            .filter((p) => p.name !== aVal)
            .map((p) => `<option>${p.name}</option>`)
            .join("");

        graphA.value = aVal;
        graphB.value = bVal;
      }
    </script>
  </body>
</html>
