# Web-HTML

<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kalkulator BMI</title>
  <style>
    body {
      font-family: sans-serif;
      max-width: 800px;
      margin: auto;
      padding: 2rem;
      background: #f9f9f9;
    }
    h1, h2 {
      text-align: center;
      color: #2c3e50;
    }
    label {
      display: block;
      margin-top: 1rem;
    }
    input, select, button {
      width: 100%;
      padding: 0.5rem;
      margin-top: 0.5rem;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    .result, .history {
      margin-top: 2rem;
      padding: 1rem;
      background: #ecf0f1;
      border-radius: 5px;
    }
    .warning {
      color: red;
    }
    .nav {
      text-align: center;
      margin-bottom: 1rem;
    }
    .nav button {
      margin: 0.5rem;
      padding: 0.5rem 1rem;
      background-color: #3498db;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    table {
      width: 100%;
      margin-top: 1rem;
      border-collapse: collapse;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 0.5rem;
      text-align: center;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
  <h1>Aplikasi BMI</h1>
  <div class="nav">
    <button onclick="tampilkanHalaman('kalkulator')">Kalkulator BMI</button>
    <button onclick="tampilkanHalaman('histori')">Histori BMI</button>
  </div>

  <div id="kalkulator">
    <h2>Kalkulator BMI</h2>
    <label for="berat">Berat Badan (kg):</label>
    <input type="number" id="berat" step="0.1">

    <label for="tinggi">Tinggi Badan (cm):</label>
    <input type="number" id="tinggi" step="0.1">

    <label for="gender">Jenis Kelamin:</label>
    <select id="gender">
      <option value="Laki-laki">Laki-laki</option>
      <option value="Perempuan">Perempuan</option>
    </select>

    <label for="kondisi">Kondisi Khusus:</label>
    <select id="kondisi">
      <option value="Tidak Ada">Tidak Ada</option>
      <option value="Atlet">Atlet</option>
      <option value="Ibu Hamil">Ibu Hamil</option>
    </select>

    <button onclick="hitungBMI()">Hitung BMI</button>
    <div id="output" class="result"></div>
  </div>

  <div id="histori" style="display:none">
    <h2>Histori BMI Anda</h2>
    <div class="history">
      <table id="tabelHistori">
        <thead>
          <tr>
            <th>Tanggal</th>
            <th>Berat</th>
            <th>Tinggi</th>
            <th>BMI</th>
            <th>Kategori</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
      <canvas id="grafikBMI" height="200"></canvas>
    </div>
  </div>

  <script>
    function tampilkanHalaman(id) {
      document.getElementById('kalkulator').style.display = id === 'kalkulator' ? 'block' : 'none';
      document.getElementById('histori').style.display = id === 'histori' ? 'block' : 'none';
      if (id === 'histori') tampilkanHistori();
    }

    function hitungBMI() {
      const berat = parseFloat(document.getElementById("berat").value);
      const tinggi = parseFloat(document.getElementById("tinggi").value);
      const gender = document.getElementById("gender").value;
      const kondisi = document.getElementById("kondisi").value;
      const output = document.getElementById("output");

      if (tinggi <= 0 || berat <= 0) {
        output.innerHTML = '<p class="warning">Masukkan berat dan tinggi yang valid.</p>';
        return;
      }

      const tinggiM = tinggi / 100;
      const bmi = berat / (tinggiM ** 2);
      let kategori = "";

      if (kondisi === "Atlet") kategori = "BMI tidak akurat untuk atlet";
      else if (kondisi === "Ibu Hamil") kategori = "BMI tidak berlaku untuk ibu hamil";
      else {
        if (gender === "Perempuan") {
          if (bmi < 18.0) kategori = "Kurus";
          else if (bmi < 24.0) kategori = "Normal (Ideal)";
          else if (bmi < 29.0) kategori = "Berat Badan Berlebih";
          else kategori = "Obesitas";
        } else {
          if (bmi < 18.5) kategori = "Kurus";
          else if (bmi < 25) kategori = "Normal (Ideal)";
          else if (bmi < 30) kategori = "Berat Badan Berlebih";
          else kategori = "Obesitas";
        }
      }

      const bmiTarget = (gender === "Laki-laki") ? 22 : 21.5;
      const beratIdeal = bmiTarget * (tinggiM ** 2);
      const selisih = berat - beratIdeal;

      let rekomendasi = "";
      if (kategori === "Kurus") {
        rekomendasi = "Konsumsi makanan bergizi dan olahraga kekuatan.";
      } else if (kategori === "Normal (Ideal)") {
        rekomendasi = "Pertahankan pola hidup sehat dan seimbang.";
      } else if (kategori === "Berat Badan Berlebih") {
        rekomendasi = "Kurangi asupan kalori dan mulai olahraga rutin.";
      } else if (kategori === "Obesitas") {
        rekomendasi = "Konsultasi dengan ahli gizi dan perbaiki pola hidup.";
      }

      output.innerHTML = `
        <p><strong>BMI Anda:</strong> ${bmi.toFixed(2)}</p>
        <p><strong>Kategori:</strong> ${kategori}</p>
        <p><strong>Berat Ideal:</strong> ${beratIdeal.toFixed(1)} kg</p>
        ${kategori === "Berat Badan Berlebih" || kategori === "Obesitas" ? `<p><strong>Disarankan menurunkan berat sekitar:</strong> ${selisih.toFixed(1)} kg</p>` : ""}
        <p><strong>Rekomendasi:</strong> ${rekomendasi}</p>
      `;

      const histori = JSON.parse(localStorage.getItem('historiBMI')) || [];
      histori.push({
        tanggal: new Date().toLocaleString(),
        berat, tinggi, bmi: bmi.toFixed(2), kategori
      });
      localStorage.setItem('historiBMI', JSON.stringify(histori));
    }

    function tampilkanHistori() {
      const data = JSON.parse(localStorage.getItem('historiBMI')) || [];
      const tbody = document.querySelector('#tabelHistori tbody');
      tbody.innerHTML = data.map(d => `
        <tr>
          <td>${d.tanggal}</td>
          <td>${d.berat}</td>
          <td>${d.tinggi}</td>
          <td>${d.bmi}</td>
          <td>${d.kategori}</td>
        </tr>
      `).join('');

      const ctx = document.getElementById('grafikBMI').getContext('2d');
      if (window.chartBMI) window.chartBMI.destroy();
      window.chartBMI = new Chart(ctx, {
        type: 'line',
        data: {
          labels: data.map(d => d.tanggal),
          datasets: [{
            label: 'BMI dari waktu ke waktu',
            data: data.map(d => parseFloat(d.bmi)),
            borderColor: 'blue',
            backgroundColor: 'rgba(0, 0, 255, 0.1)',
            tension: 0.2
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: false
            }
          }
        }
      });
    }
  </script>
</body>
</html>

<!-- Bagian baru: Analisis Tubuh -->
<div id="analisis" style="display:none">
  <h2>Analisis Tambahan Tubuh</h2>
  <label for="beratInput">Berat Badan (kg):</label>
  <input type="number" id="beratInput" step="0.1">

  <label for="tinggiInput">Tinggi Badan (cm):</label>
  <input type="number" id="tinggiInput" step="0.1">

  <label for="usiaInput">Usia:</label>
  <input type="number" id="usiaInput">

  <label for="genderInput">Jenis Kelamin:</label>
  <select id="genderInput">
    <option value="Laki-laki">Laki-laki</option>
    <option value="Perempuan">Perempuan</option>
  </select>

  <label for="aktivitasInput">Aktivitas:</label>
  <select id="aktivitasInput">
    <option value="Sedentari">Sedentari</option>
    <option value="Ringan">Ringan</option>
    <option value="Sedang">Sedang</option>
    <option value="Berat">Berat</option>
    <option value="Ekstrem">Ekstrem</option>
  </select>

  <label for="lingkarPinggang">Lingkar Pinggang (cm):</label>
  <input type="number" id="lingkarPinggang" step="0.1">

  <label for="lingkarLeher">Lingkar Leher (cm):</label>
  <input type="number" id="lingkarLeher" step="0.1">

  <div id="inputPinggulWrapper" style="display:none">
    <label for="lingkarPinggul">Lingkar Pinggul (cm):</label>
    <input type="number" id="lingkarPinggul" step="0.1">
  </div>

  <label for="tujuanKalori">Tujuan Kalori:</label>
  <select id="tujuanKalori">
    <option value="netral">Pertahankan Berat Badan</option>
    <option value="defisit">Penurunan Berat Badan (Defisit 500 kkal)</option>
    <option value="surplus">Kenaikan Massa (Surplus 500 kkal)</option>
  </select>

  <button onclick="hitungAnalisisTubuh()">Hitung</button>
  <div id="hasilAnalisis" class="result"></div>
  <canvas id="grafikLemak" height="200"></canvas>
</div>

<script>
  document.getElementById('genderInput').addEventListener('change', function() {
    document.getElementById('inputPinggulWrapper').style.display = this.value === 'Perempuan' ? 'block' : 'none';
  });

  function hitungAnalisisTubuh() {
    try {
      const berat = parseFloat(document.getElementById("beratInput").value);
      const tinggi = parseFloat(document.getElementById("tinggiInput").value);
      const usia = parseInt(document.getElementById("usiaInput").value);
      const gender = document.getElementById("genderInput").value;
      const aktivitas = document.getElementById("aktivitasInput").value;
      const lingkarPinggang = parseFloat(document.getElementById("lingkarPinggang").value);
      const lingkarLeher = parseFloat(document.getElementById("lingkarLeher").value);
      const lingkarPinggul = gender === 'Perempuan' ? parseFloat(document.getElementById("lingkarPinggul").value) : 0;
      const tujuan = document.getElementById("tujuanKalori").value;

      if (berat <= 0 || tinggi <= 0 || usia <= 0 || lingkarPinggang <= 0 || lingkarLeher <= 0 || (gender === 'Perempuan' && lingkarPinggul <= 0)) {
        throw new Error("Semua input harus terisi dan valid.");
      }

      // Hitung BMR
      const bmr = gender === "Laki-laki"
        ? 10 * berat + 6.25 * tinggi - 5 * usia + 5
        : 10 * berat + 6.25 * tinggi - 5 * usia - 161;

      const aktivitasMap = {
        Sedentari: 1.2,
        Ringan: 1.375,
        Sedang: 1.55,
        Berat: 1.725,
        Ekstrem: 1.9
      };
      const tdee = bmr * aktivitasMap[aktivitas];
      const kalori = tujuan === 'defisit' ? tdee - 500 : (tujuan === 'surplus' ? tdee + 500 : tdee);

      // Lemak tubuh (%): rumus U.S. Navy
      const tinggiLog = Math.log10(tinggi);
      let bodyFat;
      if (gender === "Laki-laki") {
        bodyFat = 495 / (1.0324 - 0.19077 * Math.log10(lingkarPinggang - lingkarLeher) + 0.15456 * tinggiLog) - 450;
      } else {
        bodyFat = 495 / (1.29579 - 0.35004 * Math.log10(lingkarPinggang + lingkarPinggul - lingkarLeher) + 0.221 * tinggiLog) - 450;
      }

      const whr = gender === 'Perempuan' ? lingkarPinggang / lingkarPinggul : lingkarPinggang / lingkarLeher;
      const risiko = gender === 'Perempuan' ? (whr >= 0.85 ? "Risiko Tinggi" : "Normal") : (whr >= 0.90 ? "Risiko Tinggi" : "Normal");

      // Tampilkan hasil
      document.getElementById("hasilAnalisis").innerHTML = `
        <p><strong>BMR:</strong> ${bmr.toFixed(2)} kkal/hari</p>
        <p><strong>TDEE:</strong> ${tdee.toFixed(2)} kkal/hari</p>
        <p><strong>Kalori Harian:</strong> ${kalori.toFixed(2)} kkal</p>
        <p><strong>Lemak Tubuh:</strong> ${bodyFat.toFixed(2)}%</p>
        <p><strong>WHR:</strong> ${whr.toFixed(2)} → ${risiko}</p>
      `;

      const ctx = document.getElementById('grafikLemak').getContext('2d');
      if (window.chartLemak) window.chartLemak.destroy();
      window.chartLemak = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: ['Lemak Anda', 'Rata-rata Ideal'],
          datasets: [{
            data: [bodyFat, gender === 'Laki-laki' ? 18 : 25],
            backgroundColor: ['orange', 'green']
          }]
        },
        options: {
          plugins: { legend: { display: false } },
          scales: {
            y: {
              beginAtZero: true,
              title: { display: true, text: '% Lemak Tubuh' }
            }
          }
        }
      });
    } catch (e) {
      document.getElementById("hasilAnalisis").innerHTML = `<p class='warning'>${e.message}</p>`;
    }
  }

  // Tambah navigasi ke menu
  const navDiv = document.querySelector('.nav');
  const btn = document.createElement('button');
  btn.textContent = 'Analisis Tubuh';
  btn.onclick = () => tampilkanHalaman('analisis');
  navDiv.appendChild(btn);
</script>

<!-- Tambahkan ke dalam div#histori sebelum penutup -->
<button onclick="resetHistori()" style="margin-top: 1rem; background-color: #e74c3c;">🗑️ Reset Histori BMI</button>

<script>
  function resetHistori() {
    if (confirm("Apakah Anda yakin ingin menghapus semua histori BMI?")) {
      localStorage.removeItem('historiBMI');
      tampilkanHistori();
    }
  }
</script>
