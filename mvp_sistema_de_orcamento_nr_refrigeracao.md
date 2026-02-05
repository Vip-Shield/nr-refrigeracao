# MVP – Sistema de Orçamento NR Refrigeração (IMPLEMENTAÇÃO FREE)

Este documento agora apresenta **a implementação prática** do sistema gratuito, pronta para rodar em produção simples.

---

## 1. Estrutura de Pastas

```
/nr-refrigeracao
 ├─ public/
 │   ├─ index.html
 │   ├─ simulador.html
 │   ├─ style.css
 │   └─ js/simulador.js
 ├─ server.js
 ├─ db.sqlite
 └─ config.js
```

---

## 2. Configuração FREE (preparada para PRO)

```js
// config.js
module.exports = {
  MODE: "FREE",
  VALOR_KM: 2.5,
  BASE: "Rua José da Costa, 264 - Mariópolis - Nilópolis - RJ"
};
```

---

## 3. Backend FREE (Node.js + Express)

```js
// server.js
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const cors = require('cors');
const config = require('./config');

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static('public'));

const db = new sqlite3.Database('./db.sqlite');

db.serialize(() => {
  db.run(`CREATE TABLE IF NOT EXISTS simulacoes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    cliente TEXT,
    endereco TEXT,
    problema TEXT,
    valor REAL,
    distancia REAL,
    data DATETIME DEFAULT CURRENT_TIMESTAMP
  )`);
});

app.post('/simulacao', (req, res) => {
  const { cliente, endereco, problema, valor, distancia } = req.body;
  db.run(
    `INSERT INTO simulacoes (cliente, endereco, problema, valor, distancia)
     VALUES (?,?,?,?,?)`,
    [cliente, endereco, problema, valor, distancia]
  );
  res.json({ status: 'ok' });
});

app.get('/simulacoes', (req, res) => {
  db.all('SELECT * FROM simulacoes ORDER BY data DESC', (err, rows) => {
    res.json(rows);
  });
});

app.listen(3000, () => console.log('Sistema NR rodando na porta 3000'));
```

---

## 4. Simulador (Frontend integrado ao site)

```html
<!-- simulador.html -->
<section class="section-pad bg-light">
  <div class="container">
    <h2 class="section-title">Simulador de Orçamento</h2>

    <div class="service-card">
      <input id="cliente" placeholder="Seu nome" />
      <input id="endereco" placeholder="Endereço completo" />
      <textarea id="problema" placeholder="Descreva o problema"></textarea>

      <select id="equipamento">
        <option value="150">Split</option>
        <option value="180">Split Inverter</option>
      </select>

      <button class="btn btn-primary" onclick="calcular()">Calcular</button>

      <div id="resultado"></div>
      <button id="confirmar" class="btn btn-outline" style="display:none" onclick="enviar()">
        Confirmar no WhatsApp
      </button>
    </div>
  </div>
</section>

<script src="https://maps.googleapis.com/maps/api/js?key=SUA_API_KEY&libraries=places"></script>
<script src="js/simulador.js"></script>
```

---

## 5. Lógica do Simulador + Google Maps

```js
// public/js/simulador.js
const BASE = "Rua José da Costa, 264 - Mariópolis - Nilópolis";

function calcular() {
  const endereco = document.getElementById('endereco').value;
  const equipamento = Number(document.getElementById('equipamento').value);
  const service = new google.maps.DistanceMatrixService();

  service.getDistanceMatrix({
    origins: [BASE],
    destinations: [endereco],
    travelMode: 'DRIVING'
  }, (res) => {
    const km = res.rows[0].elements[0].distance.value / 1000;
    const deslocamento = km * 2 * 2.5;
    const total = equipamento + deslocamento;

    document.getElementById('resultado').innerHTML = `Valor estimado: R$ ${total.toFixed(2)}`;
    window.valorFinal = total;
    window.distancia = km;

    document.getElementById('confirmar').style.display = 'block';
  });
}

function enviar() {
  const dados = {
    cliente: cliente.value,
    endereco: endereco.value,
    problema: problema.value,
    valor: window.valorFinal,
    distancia: window.distancia
  };

  fetch('/simulacao', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(dados)
  });

  window.open(`https://wa.me/5521995151440?text=Orçamento estimado R$ ${window.valorFinal}`);
}
```

---

## 6. Geração de PDF (Frontend)

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script>
function gerarPDF() {
  const pdf = new jspdf.jsPDF();
  pdf.text('NR Refrigeração', 20, 20);
  pdf.text(`Valor estimado: R$ ${window.valorFinal}`, 20, 40);
  pdf.save('orcamento-nr.pdf');
}
</script>
```

---

## 7. O que está PRONTO agora

- Sistema 100% gratuito
- Sem login
- Banco local
- Google Maps automático
- WhatsApp integrado
- Preparado para virar PRO

---

## 8. Evolução futura (sem refazer nada)

- Ativar MODE=PRO
- Criar painel admin
- Relatórios
- Pagamentos
- Multi-técnicos

---

**Este é o sistema real, funcional e escalável da NR Refrigeração.**

