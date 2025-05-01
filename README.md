# soma-valores
App PWA para somar valores extraídos de imagens via OCR com Tesseract
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Somar Valores de Imagem</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link rel="manifest" href="manifest.json" />
  <script src="https://cdn.jsdelivr.net/npm/tesseract.js@2.1.5/dist/tesseract.min.js"></script>
  <style>
    body { font-family: sans-serif; padding: 20px; background: #f9f9f9; }
    #output { margin-top: 20px; font-weight: bold; }
    button { padding: 10px 20px; font-size: 16px; }
  </style>
</head>
<body>

  <h2>Somar valores da imagem</h2>
  <input type="file" id="imageInput" accept="image/*"><br><br>
  <button onclick="processarImagem()">Processar Imagem</button>

  <div id="output"></div>

  <script>
    function processarImagem() {
      const file = document.getElementById('imageInput').files[0];
      const output = document.getElementById('output');
      if (!file) {
        output.textContent = "Por favor, selecione uma imagem.";
        return;
      }

      output.textContent = "Lendo imagem, por favor aguarde...";

      Tesseract.recognize(file, 'por', {
        logger: m => console.log(m)
      }).then(({ data: { text } }) => {
        console.log("Texto detectado:", text);
        const regex = /VALOR:\s*R\$\s*(\d+),(\d{2})/g;
        let match;
        let total = 0;

        while ((match = regex.exec(text)) !== null) {
          const reais = parseInt(match[1]);
          const centavos = parseInt(match[2]);
          total += reais + centavos / 100;
        }

        output.textContent = total > 0
          ? `Total encontrado: R$ ${total.toFixed(2)}`
          : "Nenhum valor encontrado no formato 'VALOR: R$ xx,xx'.";
      }).catch(err => {
        output.textContent = "Erro ao processar a imagem.";
        console.error(err);
      });
    }

    // Registrar o service worker
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('service-worker.js')
        .then(() => console.log('Service Worker registrado!'))
        .catch(err => console.error('Erro no SW:', err));
    }
  </script>

</body>
</html>
