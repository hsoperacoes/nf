<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de NF-e</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #1e1e1e;
      color: #f0f0f0;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 900px;
      margin: 0 auto;
      padding: 20px;
    }
    .card {
      background-color: #2c2c2c;
      padding: 20px;
      margin-bottom: 20px;
      border-radius: 8px;
    }
    .hidden {
      display: none;
    }
    .btn {
      background-color: #6a0dad;
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      margin-top: 10px;
    }
    .btn-danger {
      background-color: #c62828;
    }
    h2 {
      margin-top: 0;
    }
  </style>
</head>
<body>
  <div class="container">
    <div id="login" class="card">
      <h2>Login da Filial</h2>
      <input type="text" id="codigoFilial" placeholder="Digite o código da filial (ex: 293)" />
      <button class="btn" onclick="loginFilial()">Entrar</button>
    </div>

    <div id="principal" class="hidden">
      <div class="card">
        <input type="text" id="chave" placeholder="Digite a chave da NF-e (44 dígitos)" style="width: 100%" />
        <button class="btn" onclick="consultarNota()">Consultar</button>
        <p id="erro" style="color: red;"></p>
      </div>

      <div class="card hidden" id="resultado"></div>

      <div class="card">
        <h2>Histórico da Filial</h2>
        <ul id="historicoLista"></ul>
        <button class="btn btn-danger" onclick="limparHistoricoLocal()">🗑 Limpar Histórico Local</button>
      </div>
    </div>
  </div>

  <script>
    const URL_SCRIPT = 'https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec';

    function loginFilial() {
      const codigo = document.getElementById('codigoFilial').value.trim();
      if (codigo.length > 0) {
        localStorage.setItem('filial', codigo);
        document.getElementById('login').classList.add('hidden');
        document.getElementById('principal').classList.remove('hidden');
        carregarHistorico(codigo);
      }
    }

    function consultarNota() {
      const chave = document.getElementById('chave').value.trim();
      const filial = localStorage.getItem('filial');
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
              <p><strong>Valor Total:</strong> ${formatarValor(data.data.valorTotal)}</p>
              <p><strong>Quantidade Total:</strong> ${data.data.quantidadeTotal}</p>
              <p><strong>Status:</strong> ✅ ${data.data.status}</p>
            `;
            salvarHistoricoLocal(filial, data.data);
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

    function salvarHistoricoLocal(filial, dados) {
      const historico = JSON.parse(localStorage.getItem(`historico_${filial}`) || '[]');
      historico.unshift({
        dataHora: new Date().toISOString(),
        numeroNF: dados.numeroNF,
        valorTotal: dados.valorTotal,
        quantidade: dados.quantidadeTotal,
        status: dados.status
      });
      localStorage.setItem(`historico_${filial}`, JSON.stringify(historico.slice(0, 100)));
    }

    function formatarDataBr(iso) {
      const dt = new Date(iso);
      return dt.toLocaleDateString('pt-BR');
    }

    function formatarValor(valor) {
      if (!valor || typeof valor !== 'string') return valor;
      return valor.replace('.', ',');
    }

    function carregarHistorico(filial) {
      const historicoLista = document.getElementById('historicoLista');
      historicoLista.innerHTML = '';

      const historico = JSON.parse(localStorage.getItem(`historico_${filial}`) || '[]');

      if (historico.length === 0) {
        historicoLista.innerHTML = '<li>Nenhum histórico encontrado.</li>';
        return;
      }

      historico.forEach(registro => {
        const dataFormatada = formatarDataBr(registro.dataHora);
        let linha = `${dataFormatada}`;
        if (registro.numeroNF) linha += ` - NF ${registro.numeroNF}`;
        if (registro.valorTotal) linha += ` - ${formatarValor(registro.valorTotal)}`;
        if (registro.quantidade) linha += ` - ${registro.quantidade} itens`;
        if (registro.status) linha += ` - ${registro.status}`;
        historicoLista.insertAdjacentHTML('afterbegin', `<li>${linha}</li>`);
      });
    }

    function limparHistoricoLocal() {
      const filial = localStorage.getItem('filial');
      if (filial) {
        localStorage.removeItem(`historico_${filial}`);
        carregarHistorico(filial);
      }
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
