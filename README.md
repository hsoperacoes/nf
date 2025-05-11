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
    const historico = []; // Armazena o histórico localmente
    
    function consultarNota() {
      const chaveAcesso = document.getElementById('chaveAcesso').value.trim();
      
      if (!chaveAcesso) {
        exibirMensagem('Digite uma chave de acesso válida', 'erro');
        return;
      }
      
      if (chaveAcesso.length !== 44) {
        exibirMensagem('A chave de acesso deve ter 44 caracteres', 'erro');
        return;
      }
      
      google.script.run
        .withSuccessHandler(processarResposta)
        .withFailureHandler(exibirErro)
        .consultarNotaPorChave(chaveAcesso);
    }
    
    function processarResposta(dados) {
      if (dados && dados.valido) {
        // Adiciona ao histórico
        adicionarAoHistorico({
          numeroNF: dados.numeroNF,
          totalItens: dados.totalItens,
          valorTotal: dados.valorTotal,
          status: 'VÁLIDA'
        });
        
        exibirMensagem(`NOTA FISCAL VÁLIDA E LANÇADA - ${dados.numeroNF}`, 'sucesso');
      } else {
        // Adiciona ao histórico como inválida
        adicionarAoHistorico({
          numeroNF: 'N/A',
          totalItens: '0',
          valorTotal: '0,00',
          status: 'INVÁLIDA'
        });
        
        exibirMensagem('NOTA FISCAL INVÁLIDA', 'erro');
      }
      
      // Limpa o campo de entrada
      document.getElementById('chaveAcesso').value = '';
      document.getElementById('chaveAcesso').focus();
    }
    
    function exibirMensagem(mensagem, tipo) {
      const div = document.getElementById('resultado');
      div.innerHTML = `<p class="destaque">${mensagem}</p>`;
      div.className = tipo;
    }
    
    function exibirErro(error) {
      exibirMensagem('Erro: ' + error.message, 'erro');
    }
    
    function adicionarAoHistorico(item) {
      // Adiciona no início do array
      historico.unshift(item);
      
      // Atualiza a tabela de histórico
      atualizarHistorico();
    }
    
    function atualizarHistorico() {
      const corpo = document.getElementById('corpoHistorico');
      corpo.innerHTML = '';
      
      // Limita a 10 itens no histórico
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
    
    // Permite pressionar Enter para consultar
    document.getElementById('chaveAcesso').addEventListener('keypress', function(e) {
      if (e.key === 'Enter') consultarNota();
    });
  </script>
</body>
</html>
