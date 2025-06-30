<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Multi-Tracker mit Speicher 💾</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: sans-serif; background: #f9f9f9; padding: 20px; }
    h1 { text-align: center; color: #333; }
    #trackerList, #eingabeBereich { margin-bottom: 20px; text-align: center; }
    select, input, button { padding: 8px; margin: 5px; }
    canvas { max-width: 100%; margin-top: 20px; background: #fff; border: 1px solid #ccc; border-radius: 10px; }
    table { width: 100%; margin-top: 15px; border-collapse: collapse; }
    th, td { border: 1px solid #ccc; padding: 6px; text-align: center; }
  </style>
</head>
<body>

  <h1>📊 Fortschritts-Multi-Tracker mit Speicher</h1>

  <div id="trackerList">
    <label for="trackerSelect">Tracker auswählen:</label>
    <select id="trackerSelect"></select>
    <button onclick="neuerTracker()">➕ Neuer Tracker</button>
  </div>

  <div id="eingabeBereich">
    <input type="date" id="datumInput">
    <input type="number" id="wertInput" placeholder="Wert">
    <button onclick="wertHinzufuegen()">✅ Hinzufügen</button>
  </div>

  <table id="datenTabelle">
    <tr><th>Datum</th><th>Wert</th></tr>
  </table>

  <canvas id="trackerChart"></canvas>

  <script>
    let trackerData = {};
    let aktuellerTracker = "";

    const trackerSelect = document.getElementById("trackerSelect");
    const chartCtx = document.getElementById("trackerChart").getContext("2d");

    const chart = new Chart(chartCtx, {
      type: "line",
      data: {
        labels: [],
        datasets: [{
          label: "Fortschritt",
          data: [],
          borderColor: "rgba(75,192,192,1)",
          backgroundColor: "rgba(75,192,192,0.2)",
          tension: 0.3
        }]
      },
      options: {
        scales: {
          y: { beginAtZero: true }
        }
      }
    });

    function saveData() {
      localStorage.setItem("trackerData", JSON.stringify(trackerData));
      localStorage.setItem("aktuellerTracker", aktuellerTracker);
    }

    function loadData() {
      const data = localStorage.getItem("trackerData");
      if (data) {
        trackerData = JSON.parse(data);
        Object.keys(trackerData).forEach(name => {
          const option = document.createElement("option");
          option.text = name;
          option.value = name;
          trackerSelect.add(option);
        });
        aktuellerTracker = localStorage.getItem("aktuellerTracker") || Object.keys(trackerData)[0];
        if (aktuellerTracker) {
          trackerSelect.value = aktuellerTracker;
          trackerGewechselt();
        }
      }
    }

    function neuerTracker() {
      const name = prompt("Neuer Tracker-Name:");
      if (!name || trackerData[name]) return;
      trackerData[name] = { labels: [], werte: [] };
      const option = document.createElement("option");
      option.text = name;
      option.value = name;
      trackerSelect.add(option);
      trackerSelect.value = name;
      trackerGewechselt();
      saveData();
    }

    function trackerGewechselt() {
      aktuellerTracker = trackerSelect.value;
      updateTabelle();
      updateChart();
      saveData();
    }

    trackerSelect.addEventListener("change", trackerGewechselt);

    function wertHinzufuegen() {
      const datum = document.getElementById("datumInput").value;
      const wert = parseFloat(document.getElementById("wertInput").value);
      if (!datum || isNaN(wert)) {
        alert("Bitte gültige Daten eingeben.");
        return;
      }

      const daten = trackerData[aktuellerTracker];
      daten.labels.push(datum);
      daten.werte.push(wert);

      document.getElementById("datumInput").value = "";
      document.getElementById("wertInput").value = "";

      updateTabelle();
      updateChart();
      saveData();
    }

    function updateTabelle() {
      const tabelle = document.getElementById("datenTabelle");
      tabelle.innerHTML = "<tr><th>Datum</th><th>Wert</th></tr>";

      const daten = trackerData[aktuellerTracker];
      daten.labels.forEach((datum, index) => {
        const zeile = tabelle.insertRow();
        zeile.insertCell(0).innerText = datum;
        zeile.insertCell(1).innerText = daten.werte[index];
      });
    }

    function updateChart() {
      const daten = trackerData[aktuellerTracker];
      chart.data.labels = daten.labels;
      chart.data.datasets[0].data = daten.werte;
      chart.data.datasets[0].label = aktuellerTracker;
      chart.update();
    }

    // 🧠 Direkt beim Start alles laden
    loadData();
  </script>

</body>
</html>
