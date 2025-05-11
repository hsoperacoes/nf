function consultarNotaPorChave(chaveAcesso) {
  const spreadsheetId = '1R1Hq4kp7eaf9XfEVNCNFpUFjz3eLFcWqNl6QQx-QAew';
  const sheetName = 'ARTUR';
  const spreadsheet = SpreadsheetApp.openById(spreadsheetId);
  const planilha = spreadsheet.getSheetByName(sheetName);
  
  if (!planilha || planilha.getLastRow() < 2) {
    return { valido: false };
  }
  
  const dados = planilha.getDataRange().getValues();
  const cabecalhos = dados[0];
  
  // Encontra os índices das colunas necessárias
  const idxChave = cabecalhos.indexOf('Chave Acesso (44)');
  const idxNumeroNF = cabecalhos.indexOf('Número NF');
  const idxTotalItens = cabecalhos.indexOf('Total Itens NF');
  const idxValorTotal = cabecalhos.indexOf('Valor Total NF');
  
  if (idxChave === -1 || idxNumeroNF === -1 || idxTotalItens === -1 || idxValorTotal === -1) {
    throw new Error('Estrutura da planilha não encontrada');
  }
  
  // Pula o cabeçalho
  for (let i = 1; i < dados.length; i++) {
    const linha = dados[i];
    if (linha[idxChave] === chaveAcesso) {
      return {
        valido: true,
        numeroNF: linha[idxNumeroNF],
        totalItens: linha[idxTotalItens],
        valorTotal: linha[idxValorTotal]
      };
    }
  }
  
  return { valido: false };
}
