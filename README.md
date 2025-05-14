<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Consulta de NF</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 30px;
      background-color: #f5f5f5;
    }
    h1 {
      color: #333;
    }
    input[type="text"], input[type="password"] {
      padding: 8px;
      font-size: 16px;
      width: 320px;
    }
    button {
      padding: 8px 14px;
      font-size: 16px;
      cursor: pointer;
    }
    .resultado {
      margin-top: 20px;
      padding: 15px;
      border-radius: 6px;
    }
    .sucesso {
      background-color: #d4edda;
      color: #155724;
    }
    .erro {
      background-color: #f8d7da;
      color: #721c24;
    }
    table {
      margin-top: 30px;
      width: 100%;
      border-collapse: collapse;
      background-color: white;
    }
    th, td {
      border: 1px solid #ddd;
      padding: 10px;
      text-align: left;
    }
    th {
      background-color: #f2f2f2;
    }
    .status-valido {
      color: green;
      font-weight: bold;
    }
    .status-invalido {
      color: red;
      font-weight: bold;
    }
    #senhaContainer {
      display: none;
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <h1>Consulta de Nota Fiscal</h1>

  <input type="text" id="inputChave" placeholder="Digite a chave de 44 dígitos" maxlength="44"/>
  <button onclick="consultarNota()">Consultar</button>

  <div id="resultado" class="resultado"></div>

  <h2>Histórico de Consultas</h2>
  <table>
    <thead>
      <tr>
        <th>Número NF</th>
        <th>Valor Total</th>
        <th>Qtd. Itens</th>
        <th>Status</th>
        <th>Data/Hora</th>
      </tr>
    </thead>
    <tbody id="corpoHistorico"></tbody>
  </table>

  <button onclick="mostrarCampoSenha()">Limpar Histórico</button>

  <div id="senhaContainer">
    <input type="password" id="inputSenha" placeholder="Digite a senha para limpar"/>
    <button onclick="limparHistorico()">Confirmar</button>
  </div>

  <script>
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
    const HISTORICO_KEY = "historicoConsultasNF";
    const SENHA_LIMPEZA = "hs2024";

    const inputChave = document.getElementById("inputChave");
    const inputSenha = document.getElementById("inputSenha");
    const divResultado = document.getElementById("resultado");
    const divSenhaContainer = document.getElementById("senhaContainer");
    const corpoHistorico = document.getElementById("corpoHistorico");

    let historico = [];

    async function consultarNota() {
      const chave = inputChave.value.trim();

      if (chave.length !== 44) {
        mostrarResultado("A chave deve conter exatamente 44 dígitos.", "erro");
        return;
      }

      try {
        const data = await consultarWebApp(chave);

        if (data.mensagemJaConsultada) {
          mostrarResultado(data.mensagemJaConsultada, "erro");
          return;
        }

        processarResposta(data, chave);

      } catch (error) {
        mostrarResultado(error.message, "erro");
        registrarErroNoHistorico(error, chave);
      }

      inputChave.value = "";
    }

    async function consultarWebApp(chave) {
      const url = `${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`;
      const response = await fetch(url);

      if (!response.ok) throw new Error("Erro ao consultar o servidor");

      const json = await response.json();

      if (!json.success) {
        if (json.message && json.message.includes("já foi consultada")) {
          return { mensagemJaConsultada: json.message };
        }
        throw new Error(json.message || "Nota fiscal inválida ou não encontrada");
      }

      return json.data;
    }

    function processarResposta(data, chaveOriginal) {
      mostrarResultado("Consulta realizada com sucesso!", "sucesso");

      const novaEntrada = {
        numeroNF: data.numeroNF || 'N/A',
        valorTotal: data.valorTotal || '0,00',
        quantidadeItens: data.quantidadeTotal || '0',
        status: "Válido",
        dataHora: data.dataRegistro,
        chaveOriginal: chaveOriginal
      };

      historico.push(novaEntrada);
      salvarHistorico();
      atualizarHistorico();
    }

    function registrarErroNoHistorico(error, chaveOriginal) {
      historico.push({
        numeroNF: "",
        valorTotal: "0,00",
        quantidadeItens: "0",
        status: "Erro",
        dataHora: new Date().toLocaleString('pt-BR'),
        chaveOriginal: chaveOriginal
      });
      salvarHistorico();
      atualizarHistorico();
    }

    function mostrarResultado(msg, tipo) {
      divResultado.textContent = msg;
      divResultado.className = `resultado ${tipo}`;
    }

    function salvarHistorico() {
      localStorage.setItem(HISTORICO_KEY, JSON.stringify(historico));
    }

    function carregarHistorico() {
      const salvo = localStorage.getItem(HISTORICO_KEY);
      historico = salvo ? JSON.parse(salvo) : [];
    }

    function atualizarHistorico() {
      corpoHistorico.innerHTML = "";

      historico.forEach(item => {
        const tr = document.createElement("tr");

        tr.innerHTML = `
          <td>${item.numeroNF}</td>
          <td>R$ ${item.valorTotal}</td>
          <td>${item.quantidadeItens}</td>
          <td class="${item.status === "Válido" ? "status-valido" : "status-invalido"}">${item.status}</td>
          <td>${item.dataHora}</td>
        `;

        corpoHistorico.appendChild(tr);
      });
    }

    function mostrarCampoSenha() {
      divSenhaContainer.style.display = "block";
      inputSenha.focus();
    }

    function limparHistorico() {
      const senha = inputSenha.value.trim();

      if (senha !== SENHA_LIMPEZA) {
        alert("Senha incorreta. Tente novamente.");
        return;
      }

      localStorage.removeItem(HISTORICO_KEY);
      historico = [];
      atualizarHistorico();
      divSenhaContainer.style.display = "none";
      inputSenha.value = "";
    }

    carregarHistorico();
    atualizarHistorico();
  </script>
</body>
</html>
