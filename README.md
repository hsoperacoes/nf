<!DOCTYPE html>
<html>
<head>
  <base target="_top">
  <style>
    body { font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto; padding: 20px; }
    input, button { padding: 10px; font-size: 16px; width: 100%; box-sizing: border-box; }
    #resultado { margin-top: 20px; padding: 15px; border: 1px solid #ddd; }
    .sucesso { background-color: #dff0d8; }
    .erro { background-color: #f2dede; }
  </style>
</head>
<body>
  <h1>Registro de Entrada de Mercadoria</h1>
  <input type="text" id="codigoBarras" placeholder="Leia o código de barras ou digite o número da NF" autofocus>
  <div id="resultado"></div>
  
  <script>
    document.getElementById('codigoBarras').addEventListener('change', buscarNota);
    
    function buscarNota() {
      const codigo = document.getElementById('codigoBarras').value.trim();
      if (!codigo) return;
      
      google.script.run
        .withSuccessHandler(exibirResultado)
        .withFailureHandler(exibirErro)
        .buscarNotaPorCodigo(codigo);
    }
    
    function exibirResultado(dados) {
      const div = document.getElementById('resultado');
      if (dados && dados.length > 0) {
        div.innerHTML = `
          <h3>Nota Fiscal ${dados[0].numeroNF} - ${dados[0].filial}</h3>
          <p>Emitente: ${dados[0].emitente}</p>
          <p>Itens: ${dados.length}</p>
          <button onclick="registrarEntrada('${dados[0].chaveUnica}')">Registrar Entrada</button>
        `;
        div.className = 'sucesso';
      } else {
        div.innerHTML = 'Nota fiscal não encontrada no banco de dados';
        div.className = 'erro';
      }
      document.getElementById('codigoBarras').value = '';
      document.getElementById('codigoBarras').focus();
    }
    
    function exibirErro(error) {
      document.getElementById('resultado').innerHTML = 'Erro: ' + error.message;
      document.getElementById('resultado').className = 'erro';
    }
    
    function registrarEntrada(chaveUnica) {
      google.script.run
        .withSuccessHandler(() => {
          document.getElementById('resultado').innerHTML = 'Entrada registrada com sucesso!';
          document.getElementById('resultado').className = 'sucesso';
        })
        .registrarEntrada(chaveUnica);
    }
  </script>
</body>
</html>
