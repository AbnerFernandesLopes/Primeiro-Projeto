<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <title>Gestão de Implantações - KM Sistemas</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>

  <header>
    <h1>🚀 Gestão de Implantações</h1>
  </header>

  <section class="form">
    <input id="cliente" placeholder="Cliente">
    <input id="responsavel" placeholder="Responsável KM">
    <input id="data" type="date">
    <button onclick="addProjeto()">Adicionar</button>
  </section>

  <section>
    <h2>📊 Implantações</h2>
    <table>
      <thead>
        <tr>
          <th>Cliente</th>
          <th>Responsável</th>
          <th>Data</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody id="lista"></tbody>
    </table>
  </section>

  <script src="script.js"></script>
</body>
</html>
