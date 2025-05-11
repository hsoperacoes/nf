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
            background-color: #f5f5f5;
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
        }
        th, td {
            padding: 8px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #f2f2f2;
        }
    </style>
</head>
<body>
    <h1>Consulta de Nota Fiscal</h1>
    
    <input type="text" id="chaveAcesso" placeholder="Digite a chave de acesso (44 caracteres)" autofocus>
    <button id="btnConsultar">Consultar</button>
    
    <div id="resultado">
        Digite a chave de acesso e clique em Consultar
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
                <!-- Dados serão inseridos aqui -->
            </tbody>
        </table>
    </div>

    <script>
        // Configurações
        const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
        const historico = [];
        
        // Elementos da página
        const btnConsultar = document.getElementById("btnConsultar");
        const inputChave = document.getElementById("chaveAcesso");
        const divResultado = document.getElementById("resultado");
        const corpoHistorico = document.getElementById("corpoHistorico");
        
        // Eventos
        btnConsultar.addEventListener("click", consultarNota);
        inputChave.addEventListener("keypress", function(e) {
            if (e.key === "Enter") consultarNota();
        });
        
        // Função principal
        async function consultarNota() {
            const chave = inputChave.value.trim();
            
            if (!chave) {
                mostrarResultado("Digite uma chave de acesso", "erro");
                return;
            }
            
            if (chave.length !== 44) {
                mostrarResultado("A chave deve ter 44 caracteres", "erro");
                return;
            }
            
            mostrarResultado("Consultando...", "sucesso");
            
            try {
                const dados = await consultarWebApp(chave);
                adicionarAoHistorico(dados);
                mostrarResultado(`NF ${dados.numeroNF} encontrada: ${dados.totalItens} itens (R$ ${dados.valorTotal})`, "sucesso");
            } catch (error) {
                console.error("Erro:", error);
                mostrarResultado("Erro na consulta: " + error.message, "erro");
                adicionarAoHistorico({
                    numeroNF: "ERRO",
                    totalItens: "0",
                    valorTotal: "0,00",
                    status: "FALHA"
                });
            }
        }
        
        // Função para consultar o WebApp
        async function consultarWebApp(chave) {
            // Usando proxy para contornar CORS
            const proxyUrl = `https://cors-anywhere.herokuapp.com/${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`;
            
            const response = await fetch(proxyUrl, {
                method: 'GET',
                headers: {
                    'X-Requested-With': 'XMLHttpRequest'
                }
            });
            
            if (!response.ok) {
                throw new Error("Erro na resposta do servidor");
            }
            
            const data = await response.json();
            
            if (!data.success) {
                throw new Error(data.message || "Nota fiscal não encontrada");
            }
            
            return {
                numeroNF: data.data.numeroNF || "N/A",
                totalItens: data.data.totalItens || "0",
                valorTotal: formatarMoeda(data.data.valorTotal) || "0,00",
                status: data.data.status || "VÁLIDA"
            };
        }
        
        // Funções auxiliares
        function formatarMoeda(valor) {
            if (!valor) return "0,00";
            const num = Number(valor.toString().replace(',', '.'));
            return isNaN(num) ? "0,00" : num.toLocaleString('pt-BR', {
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            });
        }
        
        function mostrarResultado(mensagem, tipo) {
            divResultado.textContent = mensagem;
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
                    <td>${item.numeroNF}</td>
                    <td>${item.totalItens}</td>
                    <td>R$ ${item.valorTotal}</td>
                    <td class="${item.status === 'VÁLIDA' ? 'sucesso' : 'erro'}">${item.status}</td>
                </tr>
            `).join("");
        }
    </script>
</body>
</html>
