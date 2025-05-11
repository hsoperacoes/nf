<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    body { 
      font-family: Arial, sans-serif; 
      max-width: 800px; 
      margin: 0 auto; 
      padding: 20px; 
      background-color: #f5f5f5;
    }
    input, button { 
      padding: 10px; 
      font-size: 16px; 
      width: 100%; 
      box-sizing: border-box; 
      margin-bottom: 10px; 
    }
    #resultado { 
      margin-top: 20px; 
      padding: 15px; 
      border: 1px solid #ddd; 
      border-radius: 5px; 
      background-color: white;
    }
    .sucesso { 
      background-color: #dff0d8; 
      border-color: #d6e9c6; 
    }
    .erro { 
      background-color: #f2dede; 
      border-color: #ebccd1; 
    }
    #historico { 
      margin-top: 30px; 
    }
    table { 
      width: 100%; 
      border-collapse: collapse; 
      margin-top: 10px; 
      background-color: white;
    }
    th, td { 
      padding: 8px; 
      text-align: left; 
      border-bottom: 1px solid #ddd; 
    }
    th { 
      background-color: #f2f2f2; 
    }
    .destaque { 
      font-weight: bold; 
    }
  </style>
</head>
<body>
  <h1>Consulta de Nota Fiscal</h1>
  <input type="text" id="chaveAcesso" placeholder="Digite a chave de acesso (44 caracteres)" autofocus>
  <button onclick="consultarNota()">Consultar</button>
  
  <div id="resultado">Digite uma chave de acesso e clique em Consultar</div>
  
  <div id="historico">
    <h2>Histórico de Consultas</h2>
    <table id="tabelaHistorico">
      <thead>
        <tr>
          <th>Número NF</th>
          <th>Quantidade</th>
          <th>Valor Total</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody id="corpoHistorico">
        <!-- Dados serão inseridos aqui via JavaScript -->
      </tbody>
    </table>
  </div>
  
  <script>
    // Versão simplificada para teste inicial
    document.addEventListener('DOMContentLoaded', function() {
      console.log('Página carregada com sucesso!');
    });

    function consultarNota() {
      const chave = document.getElementById('chaveAcesso').value;
      const resultadoDiv = document.getElementById('resultado');
      
      if (!chave || chave.length !== 44) {
        resultadoDiv.innerHTML = 'Chave inválida! Deve ter 44 caracteres';
        resultadoDiv.className = 'erro';
        return;
      }
      
      resultadoDiv.innerHTML = 'Consultando...';
      resultadoDiv.className = 'sucesso';
      
      // Simulação de consulta - substitua por sua lógica real
      setTimeout(() => {
        adicionarAoHistorico({
          numeroNF: chave.substring(25, 34),
          totalItens: "10",
          valorTotal: "1500,00",
          status: "VÁLIDA"
        });
        
        resultadoDiv.innerHTML = `Consulta realizada para NF ${chave.substring(25, 34)}`;
      }, 1000);
    }

    function adicionarAoHistorico(item) {
      const corpo = document.getElementById('corpoHistorico');
      const novaLinha = document.createElement('tr');
      
      novaLinha.innerHTML = `
        <td>${item.numeroNF}</td>
        <td>${item.totalItens}</td>
        <td>R$ ${item.valorTotal}</td>
        <td class="${item.status === 'VÁLIDA' ? 'sucesso' : 'erro'}">${item.status}</td>
      `;
      
      // Insere no início da tabela
      corpo.insertBefore(novaLinha, corpo.firstChild);
      
      // Limita a 10 registros no histórico
      if (corpo.children.length > 10) {
        corpo.removeChild(corpo.lastChild);
      }
    }
    
    // Permitir consulta com Enter
    document.getElementById('chaveAcesso').addEventListener('keypress', function(e) {
      if (e.key === 'Enter') consultarNota();
    });
  </script>
</body>
</html>
