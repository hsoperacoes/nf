<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Consulta NF-e</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #121212;
      color: white;
      margin: 0;
      padding: 20px;
    }

    .container {
      max-width: 800px; /* AUMENTEI A LARGURA */
      margin: 0 auto;
      background-color: #1e1e1e;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }

    h2, h3 {
      text-align: center;
    }

    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }

    input {
      width: 100%;
      padding: 10px;
      margin-bottom: 20px;
      border: none;
      border-radius: 5px;
      background-color: #2a2a2a;
      color: white;
    }

    button {
      width: 100%;
      padding: 12px;
      background-color: #673ab7;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
    }

    .hidden {
      display: none;
    }

    .result, .history {
      margin-top: 20px;
      background-color: #2e2e2e;
      padding: 15px;
      border-radius: 5px;
    }

    .error {
      color: #f44336;
      margin-top: 15px;
    }

    .logout {
      text-align: right;
      margin-top: -20px;
      margin-bottom: 10px;
    }

    .logout button {
      padding: 5px 10px;
      background-color: #444;
      font-size: 14px;
    }
  </style>
</head>
<body>
  <!-- TELA DE LOGIN -->
  <div id="login" class="container">
    <h2>Login da Filial</h2>
    <label for="codigo">Código da Filial</label>
    <input type="text" id="codigo" placeholder="Ex: 293 ou 488" />
    <button onclick="entrar()">Entrar</button>
  </div>

  <!-- TELA PRINCIPAL -->
  <div id="principal" class="container hidden">
    <div class="logout">
      <button onclick="sair()">Sair</button>
    </div>
    <h2>Consulta de Nota Fiscal</h2>
    <label for="chave">Chave de Acesso (44 dígitos)</label>
    <input type="text" id="chave" placeholder="Digite a chave completa" maxlength="44"/>
    <button onclick="consultarNota()">Consultar</button>

    <div id="resultado" class="result hidden"></div>
    <div id="erro" class="error"></div>

    <div class="history">
      <h3>Histórico da Filial</h3>
      <ul id="historicoLista"></ul>
    </div>
  </div>

  <script>
    const URL_SCRIPT = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";

    function entrar() {
      const codigo = document.getElementById('codigo').value.trim();
      if (!codigo) {
        alert("Informe o código da filial.");
        return;
      }
      localStorage.setItem('filial', codigo);
      document.getElementById('login').classList.add('hidden');
      document.getElementById('principal').classList.remove('hidden');
      carregarHistorico(codigo);
    }

    function sair() {
      localStorage.removeItem('filial');
      location.reload();
    }

    function consultarNota() {
      const filial = localStorage.getItem('filial');
      const chave = document.getElementById('chave').value.trim();
      const resultado = document.getElementById('resultado');
      const erro = document.getElementById('erro');

      resultado.classList.add('hidden');
      erro.innerText = '';

      if (!filial || !chave || chave.length !== 44) {
        erro.innerText = 'Preencha corretamente a chave com 44 dígitos.';
        return;
      }

      fetch(`${URL_SCRIPT}?chave=${chave}&filial=${filial}`)
        .then(res => res.json())
        .then(data => {
          if (data.success) {
            resultado.classList.remove('hidden');
            resultado.innerHTML = `
              <p><strong>Valor Total:</strong> ${data.data.valorTotal}</p>
              <p><strong>Quantidade Total:</strong> ${data.data.quantidadeTotal}</p>
              <p><strong>Status:</strong> ✅ Válida</p>
            `;
            carregarHistorico(filial);
          } else {
            erro.innerText = data.message || 'Erro ao buscar nota fiscal.';
            carregarHistorico(filial);
          }
        })
        .catch(() => {
          erro.innerText = 'Erro de comunicação com o servidor.';
        });
    }

    function formatarDataBr(iso) {
      const dt = new Date(iso);
      return dt.toLocaleDateString('pt-BR');
    }

    function carregarHistorico(filial) {
      const historicoLista = document.getElementById('historicoLista');
      historicoLista.innerHTML = '<li>Carregando...</li>';

      fetch(`${URL_SCRIPT}?acao=historico&filial=${filial}`)
        .then(res => res.json())
        .then(data => {
          if (data.success && data.data.length > 0) {
            historicoLista.innerHTML = '';
            data.data.forEach(registro => {
              const dataFormatada = formatarDataBr(registro.dataHora);
              const numeroNF = registro.numeroNF || '-';
              const qtd = registro.quantidade || '-';
              const status = registro.status || '---';

              historicoLista.innerHTML += `
                <li>${dataFormatada} - NF ${numeroNF} - ${qtd} itens - ${status}</li>
              `;
            });
          } else {
            historicoLista.innerHTML = '<li>Nenhum histórico encontrado.</li>';
          }
        })
        .catch(() => {
          historicoLista.innerHTML = '<li>Erro ao carregar histórico.</li>';
        });
    }

    // Carrega automático se já estiver logado
    window.onload = function () {
      const filial = localStorage.getItem('filial');
      if (filial) {
        document.getElementById('login').classList.add('hidden');
        document.getElementById('principal').classList.remove('hidden');
        carregarHistorico(filial);
      }
    };
  </script>
</body>
</html>
