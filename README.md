<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta NF-e</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #121212;
      color: #fff;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
      flex-direction: column;
    }
    .container {
      width: 100%;
      max-width: 700px;
      margin: 20px auto;
      padding: 20px;
    }
    .card {
      background-color: #2a2a2a;
      border-radius: 8px;
      padding: 20px;
      margin-bottom: 20px;
    }
    button, select, input {
      padding: 10px;
      margin-top: 10px;
      font-size: 16px;
      width: 100%;
    }
    .btn-red {
      background-color: #d32f2f;
      color: white;
      border: none;
      border-radius: 4px;
    }
    .btn-purple {
      background-color: #673ab7;
      color: white;
      border: none;
      border-radius: 4px;
    }
    .hidden { display: none; }
    ul { padding-left: 20px; }
  </style>
</head>
<body>
  <div class="container">
    <div id="login" class="card">
      <h2>Informe o código da filial</h2>
      <select id="filial">
        <option value="">Selecione</option>
        <option value="293">ARTUR</option>
        <option value="488">FLORIANO</option>
      </select>
      <button class="btn-purple" onclick="entrar()">Entrar</button>
    </div>

    <div id="principal" class="card hidden">
      <input type="text" id="chave" placeholder="Digite a chave da NF (44 dígitos)" maxlength="44">
      <button class="btn-purple" onclick="consultar()">Consultar</button>
      <p id="erro" style="color: red;"></p>
      <div id="resultado" class="hidden"></div>
    </div>

    <div class="card">
      <h3>Histórico da Filial</h3>
      <ul id="historicoLista">
        <li>Nenhum histórico encontrado.</li>
      </ul>
      <button class="btn-red" onclick="limparHistorico()">🗑 Limpar Histórico Local</button>
    </div>
  </div>

  <script>
    const URL_SCRIPT = 'https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec';

    function entrar() {
      const filial = document.getElementById('filial').value;
      if (!filial) return alert('Selecione uma filial.');
      localStorage.setItem('filial', filial);
      document.getElementById('login').classList.add('hidden');
      document.getElementById('principal').classList.remove('hidden');
      carregarHistorico(filial);
    }

    function consultar() {
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
              <p><strong>Número da NF:</strong> ${data.data.numeroNF}</p>
              <p><strong>Valor Total:</strong> ${data.data.valorTotal}</p>
              <p><strong>Quantidade Total:</strong> ${data.data.quantidadeTotal}</p>
              <p><strong>Status:</strong> ✅ ${data.data.status}</p>
            `;

            const historicoKey = `historico_${filial}`;
            const historico = JSON.parse(localStorage.getItem(historicoKey)) || [];
            historico.unshift({
              dataHora: new Date().toISOString(),
              numeroNF: data.data.numeroNF,
              valorTotal: data.data.valorTotal,
              quantidade: data.data.quantidadeTotal,
              status: data.data.status
            });
            localStorage.setItem(historicoKey, JSON.stringify(historico));
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
      const historicoKey = `historico_${filial}`;
      const historico = JSON.parse(localStorage.getItem(historicoKey)) || [];

      if (historico.length > 0) {
        historicoLista.innerHTML = '';
        historico.forEach(registro => {
          const dataFormatada = formatarDataBr(registro.dataHora);
          const numeroNF = registro.numeroNF || '';
          const valor = registro.valorTotal || '';
          const qtd = registro.quantidade || '';
          const status = registro.status || '';
          let linha = `${dataFormatada}`;
          if (numeroNF && numeroNF !== '-') linha += ` - NF ${numeroNF}`;
          if (valor && valor !== '-') linha += ` - ${valor}`;
          if (qtd && qtd !== '-') linha += ` - ${qtd} itens`;
          if (status) linha += ` - ${status}`;
          historicoLista.insertAdjacentHTML('beforeend', `<li>${linha}</li>`);
        });
      } else {
        historicoLista.innerHTML = '<li>Nenhum histórico encontrado.</li>';
      }
    }

    function limparHistorico() {
      const filial = localStorage.getItem('filial');
      localStorage.removeItem(`historico_${filial}`);
      carregarHistorico(filial);
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
