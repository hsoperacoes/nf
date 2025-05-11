<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
    input, button { padding: 10px; font-size: 16px; width: 100%; box-sizing: border-box; margin-bottom: 10px; }
    #resultado { margin-top: 20px; padding: 15px; border: 1px solid #ddd; border-radius: 5px; }
    .sucesso { background-color: #dff0d8; border-color: #d6e9c6; }
    .erro { background-color: #f2dede; border-color: #ebccd1; }
    #historico { margin-top: 30px; }
    table { width: 100%; border-collapse: collapse; margin-top: 10px; }
    th, td { padding: 8px; text-align: left; border-bottom: 1px solid #ddd; }
    th { background-color: #f2f2f2; }
    .destaque { font-weight: bold; }
  </style>
</head>
<body>
  <h1>Consulta de Nota Fiscal</h1>
  <input type="text" id="chaveAcesso" placeholder="Digite a chave de acesso (44 caracteres)" autofocus>
  <button onclick="consultarNota()">Consultar</button>
  
  <div id="resultado"></div>
  
  <div id="historico">
    <h2>Histórico de Consultas</h2>
    <table id="tabelaHistorico">
      <thead>
        <tr>
          <th>Número NF</th>
          <th>Quantidade de Peças</th>
          <th>Valor Total</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody id="corpoHistorico">
        <!-- Histórico será preenchido aqui -->
      </tbody>
    </table>
  </div>
  
  <script>
    const historico = [];
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
    
    async function consultarNota() {
      const chaveAcesso = document.getElementById('chaveAcesso').value.trim();
      
      if (!chaveAcesso) {
        exibirMensagem('Digite uma chave de acesso válida', 'erro');
        return;
      }
      
      if (chaveAcesso.length !== 44) {
        exibirMensagem('A chave de acesso deve ter 44 caracteres', 'erro');
        return;
      }
      
      exibirMensagem('Consultando nota fiscal...', 'sucesso');
      
      try {
        // Usando POST em vez de GET para evitar problemas com CORS
        const response = await fetch(WEB_APP_URL, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
          },
          body: `chave=${encodeURIComponent(chaveAcesso)}`
        });
        
        if (!response.ok) {
          throw new Error(`Erro HTTP: ${response.status}`);
        }
        
        const data = await response.json();
        processarResposta(data);
      } catch (error) {
        console.error('Erro na requisição:', error);
        exibirErro(error);
      }
    }
    
    function processarResposta(dados) {
      if (dados && dados.valido) {
        adicionarAoHistorico({
          numeroNF: dados.numeroNF,
          totalItens: dados.totalItens,
          valorTotal: dados.valorTotal,
          status: 'VÁLIDA'
        });
        
        exibirMensagem(`NOTA FISCAL VÁLIDA E LANÇADA - ${dados.numeroNF}`, 'sucesso');
      } else {
        adicionarAoHistorico({
          numeroNF: 'N/A',
          totalItens: '0',
          valorTotal: '0,00',
          status: 'INVÁLIDA'
        });
        
        exibirMensagem(dados.mensagem || 'NOTA FISCAL INVÁLIDA', 'erro');
      }
      
      document.getElementById('chaveAcesso').value = '';
      document.getElementById('chaveAcesso').focus();
    }
    
    function exibirMensagem(mensagem, tipo) {
      const div = document.getElementById('resultado');
      div.innerHTML = `<p class="destaque">${mensagem}</p>`;
      div.className = tipo;
    }
    
    function exibirErro(error) {
      exibirMensagem('Erro na consulta: ' + (error.message || 'Servidor não respondeu'), 'erro');
    }
    
    function adicionarAoHistorico(item) {
      historico.unshift(item);
      atualizarHistorico();
    }
    
    function atualizarHistorico() {
      const corpo = document.getElementById('corpoHistorico');
      corpo.innerHTML = '';
      
      const historicoExibir = historico.slice(0, 10);
      
      historicoExibir.forEach(item => {
        const linha = document.createElement('tr');
        
        linha.innerHTML = `
          <td>${item.numeroNF}</td>
          <td>${item.totalItens}</td>
          <td>R$ ${item.valorTotal}</td>
          <td class="${item.status === 'VÁLIDA' ? 'sucesso' : 'erro'}">${item.status}</td>
        `;
        
        corpo.appendChild(linha);
      });
    }
    
    document.getElementById('chaveAcesso').addEventListener('keypress', function(e) {
      if (e.key === 'Enter') consultarNota();
    });
  </script>
</body>
</html>
