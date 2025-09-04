
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Cold Storage Stock Report (3-09-2025)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels"></script>
  <style>
    body {
      margin: 0;
      font-family: 'Segoe UI', sans-serif;
      background-color: #f4fdfd;
    }

    header {
      background-color: #dff6f0;
      padding: 20px 30px 10px;
      border-bottom: 1px solid #ccc;
    }

    header h1 {
      margin: 0;
      color: #00665c;
    }

    #top-menu {
      display: flex;
      gap: 10px;
      margin-top: 15px;
      flex-wrap: wrap;
    }

    .cold-item {
      padding: 10px 16px;
      border: none;
      background-color: #fff;
      cursor: pointer;
      border-radius: 5px;
      font-weight: bold;
      color: #00665c;
      border: 1px solid #b2e2d5;
    }

    .cold-item.active {
      background-color: #b6ece0;
      color: #00594d;
    }

    #content {
      padding: 30px;
    }

    .box {
      background-color: white;
      border-radius: 10px;
      padding: 20px;
      box-shadow: 0 0 10px rgba(0,0,0,0.05);
      margin-bottom: 30px;
    }

    h2 {
      margin-top: 0;
      color: #00665c;
      font-size: 22px;
    }

    h3 {
      color: #008060;
      margin-top: 20px;
      font-size: 18px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
    }

    th, td {
      border: 1px solid #ccc;
      padding: 10px;
      text-align: center;
    }

    th {
      background-color: #e8f8f5;
    }

    .charts-row {
      display: flex;
      justify-content: space-around;
      gap: 20px;
      flex-wrap: wrap;
    }

    .chart-container {
      width: 400px;
      height: 400px;
      flex-shrink: 0;
    }

    canvas {
      max-width: 100%;
      max-height: 100%;
    }

    @media (max-width: 900px) {
      .charts-row {
        flex-direction: column;
        align-items: center;
      }

      .chart-container {
        width: 90vw;
        max-width: 400px;
        height: 300px;
        margin-bottom: 30px;
      }
    }
  </style>
</head>
<body>

<header>
  <h1>Cold Storage Stock Report (3-09-2025)</h1>
  <div id="top-menu">
    <button class="cold-item active" onclick="showData('boraste', this)">Boraste</button>
    <button class="cold-item" onclick="showData('bandhan', this)">Bandhan</button>
    <button class="cold-item" onclick="showData('rizvi', this)">Rizvi</button>
  </div>
</header>

<div id="content">
  <div class="box" id="data-box"></div>
  <div class="charts-row">
    <div class="chart-container">
      <canvas id="handChart"></canvas>
    </div>
    <div class="chart-container">
      <canvas id="kgChart"></canvas>
    </div>
  </div>
</div>

<script>
  const dataSet = {
    boraste: {
      name: "BORASTE COLD STOCK",
      kg13: { "4H": 217, "5H": 968, "6H": 3486, "8H": 1699 },
      kg14: { "4H": 27, "5H": 185, "6H": 815, "8H": 379 },
      kg7: {},
      kg5: {}
    },
    bandhan: {
      name: "BANDHAN COLD STOCK",
      kg13: { "4H": 5, "5H": 16, "6H": 18, "8H": 312 },
      kg14: { "4H": 398, "5H": 483, "6H": 6212, "8H": 2639 },
      kg7: { "3H": 3175, "4H": 638 },
      kg5: { "3H": 732 }
    },
    rizvi: {
      name: "RIZVI COLD STOCK",
      kg13: { "4H": 185, "5H": 753, "6H": 2059, "8H": 1627 },
      kg14: {},
      kg7: {},
      kg5: {}
    }
  };

  let kgChartInstance;
  let handChartInstance;

  function showData(key, btn) {
    document.querySelectorAll('.cold-item').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');

    const storage = dataSet[key];
    const box = document.getElementById('data-box');
    box.innerHTML = `<h2>${storage.name}</h2>`;

    const kgTotals = {};
    const handTotals = {};
    const kgLabels = {
      kg13: "Roshana 13 KG",
      kg14: "Roshana 14 KG",
      kg7: "Laibaah 7 KG",
      kg5: "Haniya 5 KG"
    };

    for (let kg in kgLabels) {
      const data = storage[kg];
      if (Object.keys(data).length === 0) continue;

      const total = Object.values(data).reduce((a, b) => a + b, 0);
      kgTotals[kgLabels[kg]] = total;

      let html = `<h3>${kgLabels[kg]}</h3>
        <table>
          <tr>${Object.keys(data).map(h => `<th>${h}</th>`).join('')}<th>Total</th></tr>
          <tr>${Object.values(data).map(val => `<td>${val}</td>`).join('')}<td>${total}</td></tr>
        </table>`;
      box.innerHTML += html;

      for (let hand in data) {
        handTotals[hand] = (handTotals[hand] || 0) + data[hand];
      }
    }

    if (Object.keys(kgTotals).length === 0) {
      box.innerHTML += `<p>No data available for this cold storage.</p>`;
    }

    const kgCtx = document.getElementById("kgChart").getContext("2d");
    if (kgChartInstance) kgChartInstance.destroy();

    kgChartInstance = new Chart(kgCtx, {
      type: "pie",
      data: {
        labels: Object.keys(kgTotals),
        datasets: [{
          data: Object.values(kgTotals),
          backgroundColor: ['#3498db', '#2ecc71', '#f1c40f', '#e67e22']
        }]
      },
      options: {
        responsive: true,
        plugins: {
          datalabels: {
            formatter: (value) => value,
            color: '#fff',
            font: {
              weight: 'bold',
              size: 14
            }
          },
          title: {
            display: true,
            text: "Total Boxes by KG Category",
            font: {
              size: 22,
              weight: 'bold'
            }
          },
          legend: {
            position: 'bottom'
          }
        }
      },
      plugins: [ChartDataLabels]
    });

    const handCtx = document.getElementById("handChart").getContext("2d");
    if (handChartInstance) handChartInstance.destroy();

    handChartInstance = new Chart(handCtx, {
      type: "bar",
      data: {
        labels: Object.keys(handTotals),
        datasets: [{
          label: "Total Boxes",
          data: Object.values(handTotals),
          backgroundColor: '#3498db'
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          datalabels: {
            anchor: 'end',
            align: 'top',
            formatter: Math.round,
            font: {
              weight: 'bold',
              size: 12
            }
          },
          title: {
            display: true,
            text: "Total Boxes by Hand (H)",
            font: {
              size: 22,
              weight: 'bold'
            }
          },
          legend: {
            display: false
          }
        },
        scales: {
          x: {
            grid: { display: false }
          },
          y: {
            beginAtZero: true,
            grid: { display: false },
            ticks: { precision: 0 }
          }
        }
      },
      plugins: [ChartDataLabels]
    });
  }

  window.onload = () => showData('boraste', document.querySelector('.cold-item'));
</script>

</body>
</html>
