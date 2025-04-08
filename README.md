<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Simulatore Payoff Opzioni</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f7f7f7;
      margin: 20px;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .form-container, .options-table {
      text-align: center;
      margin-bottom: 20px;
    }
    .form-container input, .form-container select {
      margin: 5px;
      padding: 8px;
      font-size: 14px;
    }
    .form-container button {
      padding: 8px 16px;
      background-color: #28a745;
      color: white;
      border: none;
      cursor: pointer;
    }
    .form-container button:hover {
      background-color: #218838;
    }
    table {
      margin: 0 auto;
      border-collapse: collapse;
      width: 90%;
    }
    th, td {
      padding: 8px;
      border: 1px solid #ccc;
    }
    canvas {
      display: block;
      margin: 20px auto;
    }
  </style>
</head>
<body>

  <h1>Simulatore Payoff Opzioni</h1>

  <div class="form-container">
    <input type="number" id="strike" placeholder="Strike">
    <input type="number" id="premio" placeholder="Premio">
    <input type="number" id="quantita" placeholder="Quantità">
    <input type="number" id="scadenza" placeholder="Scadenza (giorni)">
    <input type="number" id="prezzo" placeholder="Prezzo sottostante attuale" value="100">
    <select id="tipo">
      <option value="call">Call</option>
      <option value="put">Put</option>
    </select>
    <select id="posizione">
      <option value="long">Long</option>
      <option value="short">Short</option>
    </select>
    <button onclick="aggiungiOpzione()">Aggiungi Opzione</button>
  </div>

  <div class="options-table">
    <table id="tabellaOpzioni">
      <thead>
        <tr>
          <th>Tipo</th>
          <th>Strike</th>
          <th>Premio</th>
          <th>Quantità</th>
          <th>Scadenza</th>
          <th>Posizione</th>
          <th>Azioni</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <div style="text-align: center;">
    <button onclick="aggiornaGrafico()">Genera Grafico Payoff</button>
  </div>

  <canvas id="graficoPayoff" width="800" height="400"></canvas>

  <script>
    let opzioni = [];
    let colori = ['green', 'red', 'blue', 'orange', 'purple', 'brown', 'cyan', 'magenta'];

    function aggiungiOpzione() {
      const strike = parseFloat(document.getElementById("strike").value);
      const premio = parseFloat(document.getElementById("premio").value);
      const quantita = parseInt(document.getElementById("quantita").value);
      const scadenza = parseInt(document.getElementById("scadenza").value);
      const prezzo = parseFloat(document.getElementById("prezzo").value);
      const tipo = document.getElementById("tipo").value;
      const posizione = document.getElementById("posizione").value;

      if (isNaN(strike) || isNaN(premio) || isNaN(quantita) || isNaN(scadenza) || isNaN(prezzo)) {
        alert("Inserisci tutti i valori numerici!");
        return;
      }

      opzioni.push({ strike, premio, quantita, scadenza, tipo, posizione, prezzo });
      aggiornaTabella();
    }

    function rimuoviOpzione(index) {
      opzioni.splice(index, 1);
      aggiornaTabella();
    }

    function aggiornaTabella() {
      const tbody = document.querySelector("#tabellaOpzioni tbody");
      tbody.innerHTML = "";
      opzioni.forEach((opzione, i) => {
        const row = `<tr>
          <td>${opzione.tipo.toUpperCase()}</td>
          <td>${opzione.strike}</td>
          <td>${opzione.premio}</td>
          <td>${opzione.quantita}</td>
          <td>${opzione.scadenza} gg</td>
          <td>${opzione.posizione}</td>
          <td><button onclick="rimuoviOpzione(${i})">Rimuovi</button></td>
        </tr>`;
        tbody.innerHTML += row;
      });
    }

    function calcolaPayoffOpzione(opzione, prezzi) {
      return prezzi.map(p => {
        let payoff = 0;
        if (opzione.tipo === "call") {
          payoff = Math.max(0, p - opzione.strike) - opzione.premio;
        } else if (opzione.tipo === "put") {
          payoff = Math.max(0, opzione.strike - p) - opzione.premio;
        }
        if (opzione.posizione === "short") payoff = -payoff;
        return payoff * opzione.quantita;
      });
    }

    function aggiornaGrafico() {
      if (opzioni.length === 0) {
        alert("Aggiungi almeno un'opzione prima di generare il grafico.");
        return;
      }

      const prezzoBase = opzioni[0].prezzo; // Usa il prezzo del sottostante dalla prima opzione
      const prezzi = [];
      const min = prezzoBase - 50, max = prezzoBase + 50, step = 5;
      for (let p = min; p <= max; p += step) prezzi.push(p);

      const datasets = [];
      let payoffTotale = prezzi.map(() => 0);

      opzioni.forEach((opzione, idx) => {
        const payoff = calcolaPayoffOpzione(opzione, prezzi);
        payoffTotale = payoffTotale.map((val, i) => val + payoff[i]);

        datasets.push({
          label: `${opzione.tipo.toUpperCase()} ${opzione.posizione} @${opzione.strike} (${opzione.scadenza}gg)`,
          data: payoff,
          borderColor: colori[idx % colori.length],
          borderWidth: 2,
          fill: false
        });
      });

      datasets.push({
        label: "Payoff Totale",
        data: payoffTotale,
        borderColor: "black",
        borderDash: [5, 5],
        borderWidth: 2,
        fill: false
      });

      const ctx = document.getElementById("graficoPayoff").getContext("2d");
      if (window.chart) window.chart.destroy();

      window.chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: prezzi,
          datasets: datasets
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              display: true
            }
          },
          scales: {
            x: {
              title: {
                display: true,
                text: "Prezzo Sottostante"
              }
            },
            y: {
              title: {
                display: true,
                text: "Payoff"
              }
            }
          }
        }
      });
    }
  </script>

</body>
</html>
