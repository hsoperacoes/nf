<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Consulta de Nota Fiscal</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            color: #2c3e50;
            text-align: center;
            margin-bottom: 30px;
        }
        .container {
            background-color: white;
            padding: 25px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .input-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        input, button {
            padding: 12px;
            font-size: 16px;
            width: 100%;
            box-sizing: border-box;
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
            margin-top: 30px;
            padding: 20px;
            border-radius: 5px;
            background-color: white;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .sucesso {
            background-color: #dff0d8;
            border-left: 4px solid #3c763d;
        }
        .erro {
            background-color: #f2dede;
            border-left: 4px solid #a94442;
        }
        .detalhes-nf {
            margin-top: 20px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #f2f2f2;
            font-weight: bold;
        }
        .status-valido {
            color: #3c763d;
            font-weight: bold;
        }
        .status-invalido {
            color: #a94442;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Consulta de Nota Fiscal</h1>
        
        <div class="input-group">
            <label for="chaveAcesso">Chave de Acesso (44 caracteres):</label>
            <input type="text" id="chaveAcesso" placeholder="Digite a chave de acesso completa" autofocus>
        </div>
        
        <button id="btnConsultar">Consultar Nota Fiscal</button>
        
        <div id="resultado">
            <p>Informe a chave de acesso da nota fiscal para consultar.</p>
        </div>
        
        <div class="detalhes-nf" id="detalhesNF" style="display: none;">
            <h2>Detalhes da Nota Fiscal</h2>
            <div id="dadosNF"></div>
            
            <h2>Histórico de Consultas</h2>
            <table id="tabelaHistorico">
                <thead>
                    <tr>
                        <th>Número NF</th>
                        <th>Data</th>
                        <th>Valor Total</th>
                        <th>Itens</th>
                        <th>Status</th>
                    </tr>
                </thead>
                <tbody id="corpoHistorico">
                    <!-- Dados serão inseridos aqui -->
                </tbody>
            </table>
        </div>
    </div>

    <script>
        // Configurações
        const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
        const historico = [];
        
        // Elementos da página
        const btnConsultar = document.getElementById("btnConsultar");
        const inputChave = document.getElementById("chaveAcesso");
        const divResultado = document.getElementById("resultado");
        const divDetalhesNF = document.getElementById("detalhesNF");
        const divDadosNF = document.getElementById("dadosNF");
        const corpoHistorico = document.getElementById("corpoHistorico");
        
        // Eventos
        btnConsultar.addEventListener("click", consultarNota);
        inputChave.addEventListener("keypress", function(e) {
            if (e.key === "Enter") consultarNota();
        });
        
        // Função principal
        async function consultarNota() {
            const chave = inputChave.value.trim();
            
            // Validação básica
            if (!chave) {
                mostrarResultado("Por favor, digite uma chave de acesso", "erro");
                return;
            }
            
            if (chave.length !== 44) {
                mostrarResultado("A chave deve ter exatamente 44 caracteres", "erro");
                return;
            }
            
            mostrarResultado("Consultando nota fiscal...", "sucesso");
            
            try {
                const dados = await consultarWebApp(chave);
                processarResposta(dados);
            } catch (error) {
                console.error("Erro na consulta:", error);
                mostrarResultado("Erro na consulta: " + error.message, "erro");
                registrarErroNoHistorico(error, chave);
            }
        }
        
        // Função para consultar o WebApp
        async function consultarWebApp(chave) {
            // Usando proxy para contornar CORS
            const proxyUrl = `https://corsproxy.io/?${encodeURIComponent(`${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`)}`;
            
            const response = await fetch(proxyUrl, {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            });
            
            if (!response.ok) {
                throw new Error("Erro na conexão com o servidor");
            }
            
            const data = await response.json();
            
            if (!data.success) {
                throw new Error(data.message || "Erro ao consultar nota fiscal");
            }
            
            return data;
        }
        
        // Processa a resposta do WebApp
        function processarResposta(data) {
            const nota = data.data;
            
            // Formata os dados para exibição
            const htmlDetalhes = `
                <div class="info-item">
                    <strong>Filial:</strong> ${nota.filial || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>Número NF:</strong> ${nota.numeroNF || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>Data:</strong> ${formatarData(nota.data) || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>Razão Social:</strong> ${nota.razaoSocial || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>CNPJ Remetente:</strong> ${formatarCNPJ(nota.cnpjRemetente) || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>CFOP:</strong> ${nota.cfop || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>Quantidade de Itens:</strong> ${nota.quantidadeItens || '0'}
                </div>
                <div class="info-item">
                    <strong>Valor Total:</strong> R$ ${formatarMoeda(nota.valorTotal) || '0,00'}
                </div>
                <div class="info-item">
                    <strong>Código Peça:</strong> ${nota.codPeca || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>Cor Peça:</strong> ${nota.corPeca || 'N/A'}
                </div>
                <div class="info-item">
                    <strong>Valor Unitário:</strong> R$ ${formatarMoeda(nota.valorUnitario) || '0,00'}
                </div>
                <div class="info-item">
                    <strong>Chave de Acesso:</strong> ${nota.chaveAcesso || 'N/A'}
                </div>
            `;
            
            divDadosNF.innerHTML = htmlDetalhes;
            mostrarResultado("Nota fiscal encontrada com sucesso!", "sucesso");
            divDetalhesNF.style.display = "block";
            
            // Adiciona ao histórico
            adicionarAoHistorico({
                numeroNF: nota.numeroNF,
                data: nota.data,
                valorTotal: nota.valorTotal,
                quantidadeItens: nota.quantidadeItens,
                status: "VÁLIDA"
            });
        }
        
        // Funções auxiliares
        function formatarMoeda(valor) {
            if (!valor) return "0,00";
            const num = typeof valor === 'string' ? 
                parseFloat(valor.replace(',', '.')) : 
                valor;
            return isNaN(num) ? "0,00" : num.toLocaleString('pt-BR', {
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            });
        }
        
        function formatarData(dataString) {
            if (!dataString) return 'N/A';
            try {
                const data = new Date(dataString);
                return data.toLocaleDateString('pt-BR');
            } catch {
                return dataString;
            }
        }
        
        function formatarCNPJ(cnpj) {
            if (!cnpj) return 'N/A';
            // Remove caracteres não numéricos
            const nums = cnpj.replace(/\D/g, '');
            return nums.replace(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/, '$1.$2.$3/$4-$5');
        }
        
        function mostrarResultado(mensagem, tipo) {
            divResultado.innerHTML = `<p>${mensagem}</p>`;
            divResultado.className = tipo;
        }
        
        function adicionarAoHistorico(item) {
            historico.unshift(item);
            if (historico.length > 10) historico.pop();
            atualizarHistorico();
        }
        
        function atualizarHistorico() {
            corpoHistorico.innerHTML = historico.map(item => `
                <tr>
                    <td>${item.numeroNF || 'N/A'}</td>
                    <td>${formatarData(item.data) || 'N/A'}</td>
                    <td>R$ ${formatarMoeda(item.valorTotal) || '0,00'}</td>
                    <td>${item.quantidadeItens || '0'}</td>
                    <td class="${item.status === 'VÁLIDA' ? 'status-valido' : 'status-invalido'}">
                        ${item.status || 'N/A'}
                    </td>
                </tr>
            `).join('');
        }
        
        function registrarErroNoHistorico(error, chave) {
            adicionarAoHistorico({
                numeroNF: chave.substring(25, 34) || 'ERRO',
                data: new Date().toISOString(),
                valorTotal: "0,00",
                quantidadeItens: "0",
                status: "ERRO: " + error.message.substring(0, 20)
            });
        }
    </script>
</body>
</html>
