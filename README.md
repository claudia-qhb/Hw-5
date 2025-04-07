<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Opzioni - Payoff</title>
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
    .form-container {
      text-align: center;
      margin-bottom: 20px;
    }
    .option-input {
      margin: 10px auto;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
    }
    .option-input input,
    .option-input select {
      padding: 6px;
      font-size: 14px;
      border: 1px solid #ccc;
    }
    button {
      padding: 10px 20px;
      margin-top: 10px;
      background-color: #28a745;
      color: white;
      border: none;
      cursor: pointer;
      font-size: 16px;
    }
    button:hover {
      background-color: #218838;
    }
    canvas {
      display: block;
      margin: 30px auto;
    }
  </style>
</head>
<body>

<h1>Simulatore Payoff Opzioni</h1>

<div class="form-container">
  <div id="opzioniContainer">
    <!-- Le opzioni saranno inserite qui dinamicamente -->
  </div>
  <button onclick="aggiungiOpzione()">+ Aggiungi Opzione</button><br><br>
  <button onclick="aggiornaGrafico()">Genera Grafico</button>
</div>

<canvas id="graficoPayoff" width="700" height="400"></canvas>

<script>
  let index = 0;

  function aggiungiOpzione() {
    const container = document.getElementById('opzioniContainer');
    const div = document.createElement('div');
    div.className = 'option-input';
    div.innerHTML = `
      <label>Tipo: 
        <select id="tipo-${index}">
          <option value="call">Call</option>
          <option value="put">Put</option>
        </select>
      </label>
      <input type="number" id="strike-${index}" placeholder="Strike" />
      <input type="number" id="premio-${index}" placeholder="Premio" />
      <input type="number" id="scadenza-${index}" placeholder="Scadenza (giorni)" />
      <input type="number" id="quantita-${index}" placeholder="Quantità" />
      <select id="posizione-${index}">
        <option value="long">Long</option>
        <option value="short">Short</option>
      </select>
    `;
    container.appendChild(div);
    index++;
  }

  function calcolaPayoff(strike, premio, quantita, tipo, posizione, prezzoMin, prezzoMax) {
    let payoff = [];
    for (let i = prezzoMin; i <= prezzoMax; i += 1) {
      let val = tipo === "call" ? Math.max(0, i - strike) : Math.max(0, strike - i);
      val -= premio;
      if (posizione === "short") val *= -1;
      payoff.push(val * quantita);
    }
    return payoff;
  }

  function aggiornaGrafico() {
    const prezzoMin = 50;
    const prezzoMax = 150;
    const labels = Array.from({length: prezzoMax - prezzoMin + 1}, (_, i) => prezzoMin + i);
    let datasets = [];

    for (let i = 0; i < index; i++) {
      const tipo = document.getElementById(`tipo-${i}`);
      const strike = document.getElementById(`strike-${i}`);
      const premio = document.getElementById(`premio-${i}`);
      const quantita = document.getElementById(`quantita-${i}`);
      const posizione = document.getElementById(`posizione-${i}`);
      if (!tipo || !strike || !premio || !quantita || !posizione) continue;

      const payoff = calcolaPayoff(
        parseFloat(strike.value),
        parseFloat(premio.value),
        parseInt(quantita.value),
        tipo.value,
        posizione.value,
        prezzoMin,
        prezzoMax
      );

      datasets.push({
        label: `${tipo.value.toUpperCase()} ${posizione.value} - Strike ${strike.value}`,
        data: payoff,
        borderWidth: 2,
        borderColor: tipo.value === "call" ? "green" : "red",
        backgroundColor: "rgba(0,0,0,0)",
      });
    }

    const ctx = document.getElementById('graficoPayoff').getContext('2d');
    if (window.chart) window.chart.destroy();

    window.chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: datasets
      },
      options: {
        responsive: true,
        scales: {
          x: {
            title: {
              display: true,
              text: 'Prezzo Sottostante'
            }
          },
          y: {
            title: {
              display: true,
              text: 'Payoff'
            }
          }
        }
      }
    });
  }

  // Inizializza con una opzione di default
  aggiungiOpzione();
</script>

</body>
</html>

      });
    }
  </script>

</body>
</html>
