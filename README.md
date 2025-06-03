<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Consulta NF-e</title>
  <script src="https://unpkg.com/html5-qrcode@2.2.1/html5-qrcode.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #121212;
      color: white;
      margin: 0;
      padding: 20px;
    }
    .container {
      max-width: 800px;
      margin: 0 auto;
      background-color: #1e1e1e;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }
    h2, h3 {
      text-align: center;
    }
    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    input {
      width: 100%;
      padding: 10px;
      margin-bottom: 20px;
      border: none;
      border-radius: 5px;
      background-color: #2a2a2a;
      color: white;
    }
    button {
      width: 100%;
      padding: 12px;
      background-color: #673ab7;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
    }
    .hidden {
      display: none;
    }
    .result, .history {
      margin-top: 20px;
      background-color: #2e2e2e;
      padding: 15px;
      border-radius: 5px;
    }
    .error {
      color: #f44336;
      margin-top: 15px;
    }
    .logout {
      text-align: right;
      margin-top: -20px;
      margin-bottom: 10px;
    }
    .logout button {
      padding: 5px 10px;
      background-color: #444;
      font-size: 14px;
    }
    .history button {
      background-color: #c62828;
      margin-top: 20px;
    }
    .loading {
      text-align: center;
      font-style: italic;
      margin-top: 10px;
    }
    .logo {
      display: block;
      margin: 0 auto 20px;
      max-width: 100px;
    }
    #reader {
      width: 100%;
      max-width: 600px;
      margin: 0 auto 20px;
    }
  </style>
</head>
<body>
<!-- O restante do HTML permanece inalterado -->
