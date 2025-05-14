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
      color: #ffffff;
      padding: 20px;
      margin: 0;
    }

    .container {
      max-width: 500px;
      margin: 0 auto;
      background-color: #1e1e1e;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }

    h2 {
      text-align: center;
      margin-bottom: 30px;
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

    .result {
      margin-top: 30px;
      background-color: #2e2e2e;
      padding: 15px;
      border-radius: 5px;
    }

    .error {
      color: #f44336;
      margin-top: 15px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Consulta de Nota Fiscal</h2>
    <label for="codigo">Código da Filial</label>
    <input type="text" id="codigo" placeholder="Ex: 293 ou 488" />

    <label for="chave">Chave de Acesso (44 dígitos)</label>
    <input type="text" id="chave" placeholder="Digite a chave completa" maxlength="44"/>

    <button onclick="consultarNota()">Consultar</button>

    <div id="resultado" class="result" style="display:none;"></div>
    <div id="erro" class="error"></div>
  </div>

  <script>
    function consultarNota() {
      const codigo = document.getElementById('codigo').value.trim();
      const chave = document.getElementById('chave').value.trim();
      const resultado = document.getElementById('resultado');
      const erro = document.getElementById('erro');

      resultado.style.display = 'none';
      erro.innerText = '';

      if (!codigo || !chave || chave.length !== 44) {
        erro.innerText = 'Preencha corretamente o código da filial e a chave com 44 dígitos.';
        return;
      }

      fetch(`https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec?chave=${chave}&codigo=${codigo}`)
        .then(res => res.json())
        .then(data => {
          if (data.sucesso) {
            resultado.style.display = 'block';
            resultado.innerHTML = `
              <p><strong>Quantidade de Itens:</strong> ${data.qtd}</p>
              <p><strong>Valor Total:</strong> R$ ${data.total}</p>
            `;
          } else {
            erro.innerText = data.mensagem || 'Erro ao buscar nota fiscal.';
          }
        })
        .catch(() => {
          erro.innerText = 'Erro de comunicação com o servidor.';
        });
    }
  </script>
</body>
</html>
