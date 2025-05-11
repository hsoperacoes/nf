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
        .input-group {
            margin-bottom: 20px;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        input, button {
            padding: 12px;
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
            margin-top: 15px;
            padding: 15px;
            border-radius: 5px;
            text-align: center;
        }
        .sucesso {
            background-color: #dff0d8;
            color: #3c763d;
        }
        .erro {
            background-color: #f2dede;
            color: #a94442;
        }
        #historico {
            margin-top: 30px;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
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
        .btn-limpar {
            background-color: #e74c3c;
            margin-top: 15px;
        }
        .btn-limpar:hover {
            background-color: #c0392b;
        }
        #senhaContainer {
            display: none;
            margin-top: 15px;
        }
    </style>
</head>
<body>
    <h1>Consulta de Nota Fiscal</h1>
    
    <div class="input-group">
        <input type="text" id="chaveAcesso" placeholder="Digite a chave de acesso (44 caracteres)" autofocus>
        <button id="btnConsultar">Consultar</button>
        <div id="resultado">Digite a chave e clique em Consultar</div>
    </div>
    
    <div id="historico">
        <h2>Histórico de Consultas</h2>
        <table id="tabelaHistorico">
            <thead>
                <tr>
                    <th>Número NF</th>
                    <th>Valor Total</th>
                    <th>Itens</th>
                    <th>Status</th>
                    <th>Data/Hora</th>
                </tr>
            </thead>
            <tbody id="corpoHistorico">
                <!-- Dados serão inseridos aqui -->
            </tbody>
        </table>
        <button id="btnLimpar" class="btn-limpar">Limpar Histórico</button>
        <div id="senhaContainer">
            <input type="password" id="senhaLimpar" placeholder="Digite a senha para limpar">
            <button id="btnConfirmarLimpeza">Confirmar</button>
        </div>
    </div>

    <script>
        // Configurações
        const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
        const HISTORICO_KEY = "historicoNotasFiscais";
        const SENHA_LIMPEZA = "hs"; // Senha para limpar o histórico
        let historico = [];
        
        // Elementos da página
        const btnConsultar = document.getElementById("btnConsultar");
        const btnLimpar = document.getElementById("btnLimpar");
        const btnConfirmar = document.getElementById("btnConfirmarLimpeza");
        const inputChave = document.getElementById("chaveAcesso");
        const inputSenha = document.getElementById("senhaLimpar");
        const divResultado = document.getElementById("resultado");
        const corpoHistorico = document.getElementById("corpoHistorico");
        const divSenhaContainer = document.getElementById("senhaContainer");
        
        // Carrega o histórico ao iniciar
        document.addEventListener("DOMContentLoaded", () => {
            carregarHistorico();
            atualizarHistorico();
        });
        
        // Eventos
        btnConsultar.addEventListener("click", consultarNota);
        btnLimpar.addEventListener("click", mostrarCampoSenha);
        btnConfirmar.addEventListener("click", limparHistorico);
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
                mostrarResultado(`A chave deve ter 44 caracteres (digitados: ${chave.length})`, "erro");
                return;
            }
            
            // Verifica se a chave já foi consultada recentemente
            if (chaveJaConsultada(chave)) {
                mostrarResultado("Esta chave já foi consultada recentemente", "erro");
                inputChave.value = "";
                inputChave.focus();
                return;
            }
            
            mostrarResultado("Consultando nota fiscal...", "sucesso");
            
            try {
                const dados = await consultarWebApp(chave);
                processarResposta(dados, chave);
                inputChave.value = ""; // Limpa o campo após consulta
                inputChave.focus(); // Volta o foco para o campo
            } catch (error) {
                console.error("Erro na consulta:", error);
                mostrarResultado("Erro: " + error.message, "erro");
                registrarErroNoHistorico(error, chave);
            }
        }
        
        // Verifica se a chave já foi consultada
        function chaveJaConsultada(chave) {
            return historico.some(item => 
                item.chaveOriginal === chave || 
                (item.numeroNF && item.numeroNF === chave.substring(25, 34))
            );
        }
        
        // Função para consultar o WebApp
        async function consultarWebApp(chave) {
            const url = `${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`;
            
            try {
                // Tentativa 1: Chamada direta
                let response = await fetch(url);
                
                // Se falhar, tentativa 2 com proxy
                if (!response.ok) {
                    const proxyUrl = `https://corsproxy.io/?${encodeURIComponent(url)}`;
                    response = await fetch(proxyUrl);
                }
                
                if (!response.ok) {
                    throw new Error(`Erro HTTP: ${response.status}`);
                }
                
                const data = await response.json();
                return data;
                
            } catch (error) {
                console.error("Erro na requisição:", error);
                throw new Error("Falha na comunicação com o servidor");
            }
        }
        
        // Processa a resposta do WebApp
        function processarResposta(data, chave) {
            if (!data.success) {
                throw new Error(data.message || "Nota fiscal não encontrada");
            }

            const nota = data.data;
            const nfNumero = nota.numeroNF || chave.substring(25, 34) || 'N/A';
            
            mostrarResultado(`NF ${nfNumero} consultada com sucesso!`, "sucesso");
            
            // Adiciona ao histórico
            adicionarAoHistorico({
                numeroNF: nfNumero,
                valorTotal: nota.valorTotal || 0,
                quantidadeItens: nota.quantidadeItens || 0,
                status: "VÁLIDA",
                dataConsulta: new Date().toLocaleString(),
                chaveOriginal: chave // Armazena a chave original para evitar duplicatas
            });
        }
        
        // Mostra campo de senha para limpeza
        function mostrarCampoSenha() {
            divSenhaContainer.style.display = "block";
            inputSenha.focus();
        }
        
        // Funções de histórico (persistente)
        function carregarHistorico() {
            const historicoSalvo = localStorage.getItem(HISTORICO_KEY);
            if (historicoSalvo) {
                historico = JSON.parse(historicoSalvo);
            }
        }
        
        function salvarHistorico() {
            localStorage.setItem(HISTORICO_KEY, JSON.stringify(historico));
        }
        
        function adicionarAoHistorico(item) {
            historico.unshift(item);
            if (historico.length > 50) { // Limita a 50 registros
                historico = historico.slice(0, 50);
            }
            salvarHistorico();
            atualizarHistorico();
        }
        
        function limparHistorico() {
            const senhaDigitada = inputSenha.value.trim();
            
            if (senhaDigitada !== SENHA_LIMPEZA) {
                mostrarResultado("Senha incorreta", "erro");
                inputSenha.value = "";
                return;
            }
            
            historico = [];
            salvarHistorico();
            atualizarHistorico();
            divSenhaContainer.style.display = "none";
            inputSenha.value = "";
            mostrarResultado("Histórico limpo com sucesso", "sucesso");
        }
        
        // Funções auxiliares
        function formatarMoeda(valor) {
            const num = Number(valor) || 0;
            return num.toLocaleString('pt-BR', { 
                minimumFractionDigits: 2,
                maximumFractionDigits: 2
            });
        }
        
        function mostrarResultado(mensagem, tipo) {
            divResultado.textContent = mensagem;
            divResultado.className = tipo;
        }
        
        function atualizarHistorico() {
            corpoHistorico.innerHTML = historico.map(item => `
                <tr>
                    <td>${item.numeroNF}</td>
                    <td>R$ ${formatarMoeda(item.valorTotal)}</td>
                    <td>${item.quantidadeItens}</td>
                    <td class="${item.status === 'VÁLIDA' ? 'status-valido' : 'status-invalido'}">
                        ${item.status}
                    </td>
                    <td>${item.dataConsulta || new Date().toLocaleString()}</td>
                </tr>
            `).join('');
        }
        
        function registrarErroNoHistorico(error, chave) {
            adicionarAoHistorico({
                numeroNF: chave.substring(25, 34) || 'ERRO',
                valorTotal: 0,
                quantidadeItens: 0,
                status: "ERRO",
                dataConsulta: new Date().toLocaleString(),
                chaveOriginal: chave
            });
        }
    </script>
</body>
</html>
