<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    /* Seus estilos permanecem os mesmos */
  </style>
</head>
<body>
  <!-- Seu HTML permanece o mesmo -->
  
  <script>
    const historico = [];
    // URL alternativa com redirecionamento forçado para evitar CORS
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwUa5DLhtKpa2kUAMxicHQsPlIG3gsLW-D3Scq6WUjAw42JIcUerAgy4f1H3TxsJLTB/exec?chave=";
    
    async function consultarNota() {
      const chaveAcesso = document.getElementById('chaveAcesso').value.trim();
      
      if (!validarChave(chaveAcesso)) return;
      
      exibirMensagem('Consultando nota fiscal...', 'sucesso');
      
      try {
        // Método 1: Tentativa com JSONP (para contornar CORS)
        await consultarComJSONP(chaveAcesso);
        
        // Se falhar, tentamos o método 2:
        // await consultarComProxy(chaveAcesso);
        
      } catch (error) {
        console.error('Falha na consulta:', error);
        exibirErro(error);
      }
    }
    
    function validarChave(chave) {
      if (!chave) {
        exibirMensagem('Digite uma chave de acesso válida', 'erro');
        return false;
      }
      if (chave.length !== 44) {
        exibirMensagem('A chave deve ter 44 caracteres', 'erro');
        return false;
      }
      return true;
    }
    
    function consultarComJSONP(chave) {
      return new Promise((resolve, reject) => {
        const script = document.createElement('script');
        const callbackName = `jsonp_${Date.now()}`;
        
        window[callbackName] = (response) => {
          delete window[callbackName];
          document.body.removeChild(script);
          processarResposta(response);
          resolve(response);
        };
        
        script.src = `${WEB_APP_URL}${encodeURIComponent(chave)}&callback=${callbackName}`;
        script.onerror = () => {
          reject(new Error('Falha ao carregar o script'));
        };
        
        document.body.appendChild(script);
      });
    }
    
    async function consultarComProxy(chave) {
      // Usando um proxy CORS simples (exemplo com crossorigin.me)
      const proxyUrl = `https://cors-anywhere.herokuapp.com/${WEB_APP_URL}`;
      
      const response = await fetch(proxyUrl, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'X-Requested-With': 'XMLHttpRequest'
        },
        body: `chave=${encodeURIComponent(chave)}`
      });
      
      if (!response.ok) throw new Error(`Erro HTTP: ${response.status}`);
      
      const data = await response.json();
      processarResposta(data);
    }
    
    // As demais funções permanecem iguais (processarResposta, exibirMensagem, etc.)
  </script>
</body>
</html>
