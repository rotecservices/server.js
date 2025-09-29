const express = require('express');
const puppeteer = require('puppeteer');
const app = express();

app.get('/proxy', async (req, res) => {
  const url = req.query.url;
  if (!url) return res.status(400).send('URL ausente');

  try {
    const browser = await puppeteer.launch({ headless: 'new' });
    const page = await browser.newPage();
    await page.goto(url, { waitUntil: 'networkidle2' });

    const imagem = await page.evaluate(() => {
      const card = document.querySelector('.poly-card--large img');
      return card ? card.src : null;
    });

    await browser.close();

    if (!imagem) return res.send('<h2>Imagem não encontrada</h2>');

    res.send(`
      <html>
        <head><style>body{margin:0;padding:0;text-align:center;}</style></head>
        <body>
          <img src="${imagem}" style="max-width:100%;height:auto;" />
        </body>
      </html>
    `);
  } catch (err) {
    res.status(500).send('Erro ao acessar a página');
  }
});

app.listen(3000, () => console.log('Servidor rodando'));
