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
      max-width: 800px;
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

    .history button {
      background-color: #c62828;
      margin-top: 20px;
    }

    .loading {
      text-align: center;
      font-style: italic;
      margin-top: 10px;
    }

    .logo {
      display: block;
      margin: 0 auto 20px;
      max-width: 100px;
    }
  </style>
</head>
<body>
  <img src="logo.png.png" alt="Logo" class="logo">

  <div id="login" class="container">
    <h2>Login da Filial</h2>
    <label for="codigo">Código da Filial</label>
    <input type="text" id="codigo" placeholder="Digite sua senha" />
    <button onclick="entrar()">Entrar</button>
  </div>

  <div id="principal" class="container hidden">
    <div class="logout">
      <button onclick="sair()">Sair</button>
    </div>
    <h2>Consulta de Nota Fiscal</h2>
    <label for="chave">Chave de Acesso (44 dígitos)</label>
    <input type="text" id="chave" placeholder="Digite a chave completa" maxlength="44"/>
    <button onclick="consultarNota()">Consultar</button>

    <div id="loading" class="loading hidden">⏳ Consultando nota fiscal...</div>
    <div id="resultado" class="result hidden"></div>
    <div id="erro" class="error"></div>

    <div class="history">
      <h3>Histórico da Filial</h3>
      <ul id="historicoLista"></ul>
      <button onclick="limparHistoricoLocal()">🗑 Limpar Histórico Local</button>
    </div>
  </div>

  <script>
    const URL_SCRIPT = "https://script.google.com/macros/s/AKfycbwfoYOgleHUcmbr_1B8tV_NG6cEZxcHm5zBSrJ0ItgRV_Cp7tumh3GjBzsvzTSNJ5sbmA/exec";

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

    function limparHistoricoLocal() {
      const senha = prompt("Digite a senha para limpar o histórico:");
      if (senha !== "hs") {
        alert("Senha incorreta. Operação cancelada.");
        return;
      }

      const filial = localStorage.getItem('filial');
      localStorage.removeItem(`historico_${filial}`);
      document.getElementById('historicoLista').innerHTML = '';
      alert("Histórico local apagado.");
    }

    function consultarNota() {
      const filial = localStorage.getItem('filial');
      const chave = document.getElementById('chave').value.trim();
      const resultado = document.getElementById('resultado');
      const erro = document.getElementById('erro');
      const loading = document.getElementById('loading');
      const botao = document.querySelector("button[onclick='consultarNota()']");

      resultado.classList.add('hidden');
      erro.innerText = '';

      if (!filial || !chave || chave.length !== 44) {
        erro.innerText = 'Preencha corretamente a chave com 44 dígitos.';
        return;
      }

      const historico = JSON.parse(localStorage.getItem(`historico_${filial}`)) || [];
      const chaveExistente = historico.find(h => h.chave === chave);
      if (chaveExistente) {
        erro.innerText = 'Essa chave já foi consultada anteriormente!';
        return;
      }

      botao.disabled = true;
      loading.classList.remove('hidden');

      fetch(`${URL_SCRIPT}?chave=${chave}&filial=${filial}`)
        .then(res => res.json())
        .then(data => {
          if (data.success) {
            const valorFormatado = data.data.valorTotal;
            resultado.classList.remove('hidden');
            resultado.innerHTML = `
              <p><strong>Número da NF:</strong> ${data.data.numeroNF}</p>
              <p><strong>Valor Total:</strong> ${valorFormatado}</p>
              <p><strong>Quantidade Total:</strong> ${data.data.quantidadeTotal}</p>
              <p><strong>Status:</strong> ✅ ${data.data.status}</p>
            `;

            historico.push({
              chave: chave,
              dataHora: new Date().toISOString(),
              numeroNF: data.data.numeroNF,
              valorTotal: valorFormatado,
              quantidade: data.data.quantidadeTotal
            });
            localStorage.setItem(`historico_${filial}`, JSON.stringify(historico));
            carregarHistorico(filial);
          } else {
            erro.innerText = data.message || 'Erro ao buscar nota fiscal.';
          }
        })
        .catch(() => {
          erro.innerText = 'Erro de comunicação com o servidor.';
        })
        .finally(() => {
          loading.classList.add('hidden');
          botao.disabled = false;
        });
    }

    function formatarDataBr(iso) {
      const dt = new Date(iso);
      return dt.toLocaleDateString('pt-BR');
    }

    function carregarHistorico(filial) {
      const historicoLista = document.getElementById('historicoLista');
      const historico = JSON.parse(localStorage.getItem(`historico_${filial}`)) || [];

      if (historico.length === 0) {
        historicoLista.innerHTML = '<li>Nenhum histórico local encontrado.</li>';
        return;
      }

      historicoLista.innerHTML = '';
      historico.slice().reverse().forEach(registro => {
        const dataFormatada = formatarDataBr(registro.dataHora);
        let linha = `${dataFormatada} - NF ${registro.numeroNF} - ${registro.valorTotal} - ${registro.quantidade} itens`;
        historicoLista.insertAdjacentHTML('beforeend', `<li>${linha}</li>`);
      });
    }

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
