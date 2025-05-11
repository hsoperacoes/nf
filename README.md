<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Consulta de Nota Fiscal</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
      background-color: #f9f9f9;
      line-height: 1.6;
    }
    h1 {
      color: #2c3e50;
      text-align: center;
    }
    input, button {
      padding: 10px;
      font-size: 16px;
      width: 100%;
      box-sizing: border-box;
      margin-bottom: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    button {
      background-color: #3498db;
      color: white;
      border: none;
      cursor: pointer;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #2980b9;
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
      color: #3c763d;
    }
    .erro {
      background-color: #f2dede;
      border-color: #ebccd1;
      color: #a94442;
    }
    #historico {
      margin-top: 30px;
      background-color: white;
      padding: 15px;
      border-radius: 5px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      padding: 10px;
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
  <button id="btnConsultar">Consultar</button>
  
  <div id="resultado">
    <p>Digite a chave de acesso da nota fiscal e clique em Consultar.</p>
  </div>
  
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
    // Verificação inicial
    console.log("Script carregado com sucesso!");
    document.addEventListener('DOMContentLoaded', function() {
      console.log("DOM totalmente carregado!");
    });

    // Histórico de consultas
    const historicoConsultas = [];
    
    // Elementos da interface
    const btnConsultar = document.getElementById('btnConsultar');
    const inputChave = document.getElementById('chaveAcesso');
    const divResultado = document.getElementById('resultado');
    const corpoHistorico = document.getElementById('corpoHistorico');
    
    // URL do WebApp - substituída pela sua URL fornecida
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
    
    // Event Listeners
    btnConsultar.addEventListener('click', consultarNota);
    inputChave.addEventListener('keypress', function(e) {
      if (e.key === 'Enter') consultarNota();
    });
    
    // Função principal de consulta
    async function consultarNota() {
      const chave = inputChave.value.trim();
      
      // Validação básica
      if (!chave) {
        mostrarResultado('Por favor, digite uma chave de acesso', 'erro');
        return;
      }
      
      if (chave.length !== 44) {
        mostrarResultado('A chave deve ter exatamente 44 caracteres', 'erro');
        return;
      }
      
      mostrarResultado('Consultando nota fiscal...', 'sucesso');
      
      try {
        // Chamada real ao WebApp
        const dadosNota = await consultarWebApp(chave);
        
        // Adiciona ao histórico
        adicionarAoHistorico(dadosNota);
        
        // Mostra resultado
        mostrarResultado(
          `NF ${dadosNota.numeroNF} encontrada: ${dadosNota.totalItens} itens, Valor Total R$ ${dadosNota.valorTotal}`,
          dadosNota.status === 'VÁLIDA' ? 'sucesso' : 'erro'
        );
        
      } catch (error) {
        console.error("Erro na consulta:", error);
        mostrarResultado(`Erro: ${error.message}`, 'erro');
        adicionarAoHistorico({
          numeroNF: 'ERRO',
          totalItens: '0',
          valorTotal: '0,00',
          status: 'FALHA'
        });
      }
    }
    
    // Função para consultar o WebApp real
    async function consultarWebApp(chave) {
      // Usando um proxy CORS para contornar restrições
      const proxyUrl = 'https://cors-anywhere.herokuapp.com/';
      const urlCompleta = `${proxyUrl}${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`;
      
      const response = await fetch(urlCompleta, {
        method: 'GET',
        headers: {
          'X-Requested-With': 'XMLHttpRequest'
        }
      });
      
      if (!response.ok) {
        throw new Error('Erro na resposta do servidor');
      }
      
      const data = await response.json();
      
      // Verifica se a resposta tem a estrutura esperada
      if (!data || !data.success || !data.data) {
        throw new Error(data?.message || 'Resposta inválida do servidor');
      }
      
      return {
        numeroNF: data.data.numeroNF || 'N/A',
        totalItens: data.data.totalItens || '0',
        valorTotal: formatarMoeda(data.data.valorTotal) || '0,00',
        status: data.data.status || 'VÁLIDA'
      };
    }
    
    // Função para formatar valores monetários
    function formatarMoeda(valor) {
      if (!valor) return '0,00';
      
      // Converte para número se for string
      const numero = typeof valor === 'string' ? 
        parseFloat(valor.replace('.', '').replace(',', '.')) : 
        valor;
      
      return numero.toLocaleString('pt-BR', {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      });
    }
    
    // Função para mostrar resultados
    function mostrarResultado(mensagem, tipo) {
      divResultado.innerHTML = `<p class="destaque">${mensagem}</p>`;
      divResultado.className = tipo;
    }
    
    // Função para adicionar ao histórico
    function adicionarAoHistorico(item) {
      // Limita o histórico a 10 itens
      if (historicoConsultas.length >= 10) {
        historicoConsultas.pop();
      }
      
      // Adiciona no início do array
      historicoConsultas.unshift(item);
      
      // Atualiza a tabela
      atualizarHistorico();
    }
    
    // Função para atualizar a tabela de histórico
    function atualizarHistorico() {
      corpoHistorico.innerHTML = '';
      
      historicoConsultas.forEach(item => {
        const linha = document.createElement('tr');
        
        linha.innerHTML = `
          <td>${item.numeroNF}</td>
          <td>${item.totalItens}</td>
          <td>R$ ${item.valorTotal}</td>
          <td class="${item.status === 'VÁLIDA' ? 'sucesso' : 'erro'}">${item.status}</td>
        `;
        
        corpoHistorico.appendChild(linha);
      });
    }
  </script>
</body>
</html>
