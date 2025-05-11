<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    /* [Seus estilos CSS permanecem os mesmos] */
  </style>
</head>
<body>
  <!-- [Seu HTML permanece o mesmo] -->
  
  <script>
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec";

    async function consultarNota() {
      const chave = document.getElementById('chaveAcesso').value.trim();
      const resultadoDiv = document.getElementById('resultado');
      
      // Validação básica
      if (!chave || chave.length !== 44) {
        resultadoDiv.innerHTML = 'Chave inválida! Deve ter exatamente 44 caracteres';
        resultadoDiv.className = 'erro';
        return;
      }
      
      resultadoDiv.innerHTML = 'Consultando nota fiscal...';
      resultadoDiv.className = 'sucesso';
      
      try {
        // 1. Primeiro tentamos diretamente (sem proxy)
        let data = await tentarConsultaDireta(chave);
        
        // 2. Se falhar, tentamos com proxy CORS
        if (!data) {
          data = await tentarConsultaComProxy(chave);
        }
        
        // Processamento dos dados
        if (data && data.success) {
          exibirDadosCorretos(data);
        } else {
          throw new Error(data?.message || 'Nota fiscal não encontrada');
        }
        
      } catch (error) {
        console.error("Erro detalhado:", error);
        resultadoDiv.innerHTML = `Erro: ${error.message}`;
        resultadoDiv.className = 'erro';
        
        adicionarAoHistorico({
          numeroNF: 'ERRO',
          totalItens: '0',
          valorTotal: '0,00',
          status: 'FALHA',
          observacao: error.message.substring(0, 30)
        });
      }
    }

    async function tentarConsultaDireta(chave) {
      try {
        const response = await fetch(`${WEB_APP_URL}?chave=${encodeURIComponent(chave)}`, {
          method: 'GET',
          redirect: 'follow'
        });
        
        if (!response.ok) return null;
        return await response.json();
      } catch {
        return null;
      }
    }

    async function tentarConsultaComProxy(chave) {
      const proxyUrl = 'https://cors-anywhere.herokuapp.com/';
      try {
        const response = await fetch(`${proxyUrl}${WEB_APP_URL}`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
          },
          body: `chave=${encodeURIComponent(chave)}`
        });
        
        return await response.json();
      } catch (error) {
        throw new Error('Falha ao conectar com o servidor');
      }
    }

    function exibirDadosCorretos(data) {
      const resultadoDiv = document.getElementById('resultado');
      const nfData = data.data; // Ajuste conforme a estrutura do seu retorno
      
      // Verificação crítica dos dados
      if (!nfData || !nfData.numeroNF) {
        throw new Error('Estrutura de dados inválida');
      }
      
      // Formata os valores corretamente
      const valorFormatado = parseFloat(nfData.valorTotal || 0).toLocaleString('pt-BR', {
        style: 'currency',
        currency: 'BRL'
      });
      
      // Adiciona ao histórico
      adicionarAoHistorico({
        numeroNF: nfData.numeroNF || 'N/A',
        totalItens: nfData.totalItens || '0',
        valorTotal: valorFormatado,
        status: nfData.valido ? 'VÁLIDA' : 'INVÁLIDA'
      });
      
      // Exibe resultado
      resultadoDiv.innerHTML = `
        <p class="destaque">NF ${nfData.numeroNF} - ${nfData.valido ? 'VÁLIDA' : 'INVÁLIDA'}</p>
        <p>Itens: ${nfData.totalItens}</p>
        <p>Valor Total: ${valorFormatado}</p>
        ${nfData.observacao ? `<p>Obs: ${nfData.observacao}</p>` : ''}
      `;
      resultadoDiv.className = nfData.valido ? 'sucesso' : 'erro';
    }

    // [Suas outras funções permanecem aqui]
  </script>
</body>
</html>
