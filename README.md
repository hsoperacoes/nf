<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Consulta de Nota Fiscal</title>
    <style>
        /* [MANTENHA OS ESTILOS ANTERIORES] */
    </style>
</head>
<body>
    <!-- [MANTENHA A ESTRUTURA HTML ANTERIOR] -->

    <script>
        // Configurações do WebApp
        const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";
        const historico = [];

        // Elementos da página
        const btnConsultar = document.getElementById("btnConsultar");
        const inputChave = document.getElementById("chaveAcesso");
        const divResultado = document.getElementById("resultado");
        const corpoHistorico = document.getElementById("corpoHistorico");

        // Eventos
        btnConsultar.addEventListener("click", consultarNota);
        inputChave.addEventListener("keypress", (e) => e.key === "Enter" && consultarNota());

        // Função principal
        async function consultarNota() {
            const chave = inputChave.value.trim();
            
            if (!validarChave(chave)) return;
            
            mostrarResultado("Consultando nota fiscal...", "sucesso");
            
            try {
                const dados = await consultarWebApp(chave);
                processarResposta(dados);
            } catch (error) {
                console.error("Erro na consulta:", error);
                mostrarResultado(`Erro: ${error.message}`, "erro");
                registrarErroNoHistorico(error);
            }
        }

        // Função para consultar o WebApp
        async function consultarWebApp(chave) {
            const url = `${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`;
            
            // Usando proxy CORS para desenvolvimento
            const proxyUrl = `https://corsproxy.io/?${encodeURIComponent(url)}`;
            
            const response = await fetch(proxyUrl, {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            });

            if (!response.ok) throw new Error(`Erro HTTP: ${response.status}`);
            
            const data = await response.json();
            
            if (!data.success) throw new Error(data.message || "Erro no servidor");
            
            return data;
        }

        // Funções auxiliares (ajustadas para sua estrutura)
        function validarChave(chave) {
            if (!chave || chave.length !== 44) {
                mostrarResultado("Chave inválida! Deve ter 44 caracteres", "erro");
                return false;
            }
            return true;
        }

        function processarResposta(data) {
            // AJUSTE ESTAS LINHAS CONFORME SUA ESTRUTURA REAL!
            const nota = {
                numeroNF: data.data.numeroNF || "N/A",
                totalItens: data.data.quantidadeItens || "0",  // Altere para o nome do campo correto
                valorTotal: formatarMoeda(data.data.valor || 0), // Altere para o nome do campo correto
                status: data.data.situacao || "VÁLIDA"          // Altere para o nome do campo correto
            };

            adicionarAoHistorico(nota);
            mostrarResultado(
                `NF ${nota.numeroNF}: ${nota.totalItens} itens (R$ ${nota.valorTotal})`,
                nota.status.includes("VÁLID") ? "sucesso" : "erro" // Ajuste conforme seus status
            );
        }

        function formatarMoeda(valor) {
            const num = Number(valor) || 0;
            return num.toLocaleString('pt-BR', { minimumFractionDigits: 2 });
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
                    <td class="${item.status.includes("VÁLID") ? "sucesso" : "erro"}">${item.status}</td>
                </tr>
            `).join("");
        }

        function mostrarResultado(mensagem, tipo) {
            divResultado.innerHTML = `<p class="destaque">${mensagem}</p>`;
            divResultado.className = tipo;
        }

        function registrarErroNoHistorico(error) {
            adicionarAoHistorico({
                numeroNF: "ERRO",
                totalItens: "0",
                valorTotal: "0,00",
                status: error.message.substring(0, 20)
            });
        }
    </script>
</body>
</html>
