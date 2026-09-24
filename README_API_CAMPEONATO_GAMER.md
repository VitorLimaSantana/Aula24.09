# 🎮 Projeto prático — API de Campeonato Gamer

## Node.js + Express | 2º ano do Ensino Médio

Sua turma vai organizar um campeonato de jogos. Cada partida terá um jogo, dois times, placar e status. A API permitirá cadastrar, consultar, editar e excluir partidas, além de pesquisar partidas por jogo, time ou status. Exemplos de jogos são fictícios: troque pelos títulos escolhidos pela turma.

**Objetivo:** praticar API REST, CRUD, `req.params`, `req.body`, `req.query`, arrays e códigos HTTP. Primeiro construiremos cada rota; ao final há o código completo para conferir.

> Nesta versão, os dados ficam em um array: alterações desaparecem ao reiniciar o servidor. Não há banco de dados nem autenticação.

## 1. Preparação

No terminal:

```bash
mkdir campeonato-gamer
cd campeonato-gamer
npm init -y
npm install express cors
code .
```

Crie `server.js` e comece com:

```js
const express = require("express");
const cors = require("cors");
const app = express();

app.use(cors());
app.use(express.json());

const PORTA = 3000;
app.get("/", (req, res) => res.json({ mensagem: "API do Campeonato Gamer funcionando!" }));
app.listen(PORTA, () => console.log(`API em http://localhost:${PORTA}`));
```

Rode `node server.js` e visite `http://localhost:3000`. Ao acrescentar código, pare com `Ctrl+C` e inicie novamente.

**⏸️ Pausa 1:** Qual é a função de `express.json()`? Ele permite ler o JSON enviado no corpo da requisição.

## 2. Nosso banco temporário

Acima de `app.get("/", ...)`, adicione:

```js
let PARTIDAS = [
  { id: 1, jogo: "Corrida Turbo", timeA: "Falcões", timeB: "Dragões", pontosA: 3, pontosB: 2, status: "finalizada" },
  { id: 2, jogo: "Arena Pixel", timeA: "Lobos", timeB: "Falcões", pontosA: 0, pontosB: 0, status: "agendada" },
  { id: 3, jogo: "Arena Pixel", timeA: "Dragões", timeB: "Lobos", pontosA: 1, pontosB: 1, status: "finalizada" }
];
```

Cada objeto representa uma partida. O `id` identifica a partida; `pontosA` e `pontosB` são números; `status` pode ser `agendada` ou `finalizada`.

## 3. READ: listar e pesquisar

Acima de `app.listen`, crie a rota:

```js
app.get("/partidas", (req, res) => {
  const { jogo, time, status, limite } = req.query;
  let resultado = [...PARTIDAS];

  if (jogo) resultado = resultado.filter(p => p.jogo.toLowerCase().includes(jogo.toLowerCase()));
  if (time) resultado = resultado.filter(p =>
    p.timeA.toLowerCase().includes(time.toLowerCase()) ||
    p.timeB.toLowerCase().includes(time.toLowerCase())
  );
  if (status) resultado = resultado.filter(p => p.status.toLowerCase() === status.toLowerCase());
  if (limite !== undefined) {
    const quantidade = Number(limite);
    if (!Number.isInteger(quantidade) || quantidade < 1) {
      return res.status(400).json({ mensagem: "limite deve ser um inteiro positivo" });
    }
    resultado = resultado.slice(0, quantidade);
  }

  res.json(resultado);
});
```

`req.query` lê informações após `?`; `filter()` mantém os itens que atendem à condição; `includes()` permite pesquisa parcial; `slice()` limita o resultado. A cópia `[...PARTIDAS]` evita alterar o array original durante a consulta.

**⏸️ Pausa 2 — teste no navegador ou Insomnia:**

```text
GET http://localhost:3000/partidas
GET http://localhost:3000/partidas?jogo=arena
GET http://localhost:3000/partidas?time=falc
GET http://localhost:3000/partidas?status=finalizada&limite=1
GET http://localhost:3000/partidas?limite=abc
```

No último caso, espere status `400`.

## 4. READ: buscar uma partida pelo ID

```js
app.get("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const partida = PARTIDAS.find(p => p.id === id);
  if (!partida) return res.status(404).json({ mensagem: "Partida não encontrada" });
  res.json(partida);
});
```

`req.params.id` vem do caminho `/partidas/2`, como texto; `Number()` converte para número. `find()` retorna o objeto encontrado.

**⏸️ Pausa 3:** teste `GET /partidas/2` e `GET /partidas/99`. Explique o `404`.

## 5. CREATE: cadastrar uma partida

No Postman ou Insomnia, selecione método **POST**, Body → **JSON**. Adicione ao arquivo:

```js
app.post("/partidas", (req, res) => {
  const { jogo, timeA, timeB } = req.body;
  if (typeof jogo !== "string" || !jogo.trim() ||
      typeof timeA !== "string" || !timeA.trim() ||
      typeof timeB !== "string" || !timeB.trim()) {
    return res.status(400).json({ mensagem: "Informe jogo, timeA e timeB" });
  }
  if (timeA.trim().toLowerCase() === timeB.trim().toLowerCase()) {
    return res.status(400).json({ mensagem: "Escolha dois times diferentes" });
  }

  const novoId = PARTIDAS.length ? Math.max(...PARTIDAS.map(p => p.id)) + 1 : 1;
  const partida = {
    id: novoId, jogo: jogo.trim(), timeA: timeA.trim(), timeB: timeB.trim(),
    pontosA: 0, pontosB: 0, status: "agendada"
  };
  PARTIDAS.push(partida);
  res.status(201).json({ mensagem: "Partida cadastrada", partida });
});
```

O placar inicial é `0 x 0`; a partida começa como `agendada`. `map()` pega os IDs, `Math.max()` encontra o maior e `push()` adiciona o registro. O `201` significa criado.

**⏸️ Pausa 4:** envie `POST http://localhost:3000/partidas` com:

```json
{ "jogo": "Batalha Neon", "timeA": "Raios", "timeB": "Cometas" }
```

Faça `GET /partidas` para confirmar. Tente cadastrar dois times com o mesmo nome e observe o `400`.

## 6. UPDATE: registrar ou corrigir o placar

Use **PUT** com todos os campos. Adicione:

```js
app.put("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const indice = PARTIDAS.findIndex(p => p.id === id);
  if (indice === -1) return res.status(404).json({ mensagem: "Partida não encontrada" });

  const { jogo, timeA, timeB, pontosA, pontosB, status } = req.body;
  if (typeof jogo !== "string" || !jogo.trim() ||
      typeof timeA !== "string" || !timeA.trim() ||
      typeof timeB !== "string" || !timeB.trim() ||
      timeA.trim().toLowerCase() === timeB.trim().toLowerCase() ||
      !Number.isInteger(pontosA) || pontosA < 0 ||
      !Number.isInteger(pontosB) || pontosB < 0 ||
      !["agendada", "finalizada"].includes(status)) {
    return res.status(400).json({ mensagem: "Dados inválidos: confira times, placar e status" });
  }

  PARTIDAS[indice] = {
    id, jogo: jogo.trim(), timeA: timeA.trim(), timeB: timeB.trim(),
    pontosA, pontosB, status
  };
  res.json({ mensagem: "Partida atualizada", partida: PARTIDAS[indice] });
});
```

`findIndex()` retorna a posição no array. O ID vem de `req.params`; os dados novos vêm de `req.body`. Uma partida finalizada pode terminar empatada.

**⏸️ Pausa 5:** `PUT http://localhost:3000/partidas/2` com:

```json
{
  "jogo": "Arena Pixel", "timeA": "Lobos", "timeB": "Falcões",
  "pontosA": 4, "pontosB": 2, "status": "finalizada"
}
```

Confirme com `GET /partidas/2`. Depois experimente enviar `"pontosA": -1`.

## 7. DELETE: excluir uma partida

```js
app.delete("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const indice = PARTIDAS.findIndex(p => p.id === id);
  if (indice === -1) return res.status(404).json({ mensagem: "Partida não encontrada" });

  const [partida] = PARTIDAS.splice(indice, 1);
  res.json({ mensagem: "Partida excluída", partida });
});
```

`splice(indice, 1)` remove um item a partir da posição encontrada e devolve o item removido.

**⏸️ Pausa 6:** `DELETE /partidas/3`; depois `GET /partidas/3` deve retornar `404`.

## 8. Resumo para o quadro

| Operação | Método e rota | Informação recebida |
| --- | --- | --- |
| Listar e filtrar | `GET /partidas?time=Lobos` | `req.query` |
| Buscar uma | `GET /partidas/2` | `req.params.id` |
| Cadastrar | `POST /partidas` | `req.body` |
| Atualizar | `PUT /partidas/2` | `req.params.id` e `req.body` |
| Excluir | `DELETE /partidas/2` | `req.params.id` |

| Status | Significado neste projeto |
| --- | --- |
| `200` | Consulta, alteração ou exclusão concluída |
| `201` | Partida criada |
| `400` | Dados da requisição inválidos |
| `404` | Partida não encontrada |

## 9. Desafios para fazer em duplas

1. Criar o filtro `GET /partidas?vencedor=Falcões`: considerar só partidas finalizadas e ignorar empates.
2. Criar `GET /partidas?empate=true`: retornar partidas finalizadas com placares iguais.
3. Criar `GET /partidas?ordem=jogo`: ordenar alfabeticamente pelo jogo sem modificar `PARTIDAS`.
4. Criar `GET /estatisticas`: informar total de partidas, agendadas e finalizadas.
5. Extra: criar um front-end simples que liste as partidas usando `fetch()`.

## 10. Código final completo — `server.js`

Copie esta versão apenas para conferir o resultado construído nas etapas anteriores.

```js
const express = require("express");
const cors = require("cors");
const app = express();
app.use(cors());
app.use(express.json());

let PARTIDAS = [
  { id: 1, jogo: "Corrida Turbo", timeA: "Falcões", timeB: "Dragões", pontosA: 3, pontosB: 2, status: "finalizada" },
  { id: 2, jogo: "Arena Pixel", timeA: "Lobos", timeB: "Falcões", pontosA: 0, pontosB: 0, status: "agendada" },
  { id: 3, jogo: "Arena Pixel", timeA: "Dragões", timeB: "Lobos", pontosA: 1, pontosB: 1, status: "finalizada" }
];

app.get("/", (req, res) => res.json({ mensagem: "API do Campeonato Gamer funcionando!" }));

app.get("/partidas", (req, res) => {
  const { jogo, time, status, limite } = req.query;
  let resultado = [...PARTIDAS];
  if (jogo) resultado = resultado.filter(p => p.jogo.toLowerCase().includes(jogo.toLowerCase()));
  if (time) resultado = resultado.filter(p =>
    p.timeA.toLowerCase().includes(time.toLowerCase()) ||
    p.timeB.toLowerCase().includes(time.toLowerCase())
  );
  if (status) resultado = resultado.filter(p => p.status.toLowerCase() === status.toLowerCase());
  if (limite !== undefined) {
    const quantidade = Number(limite);
    if (!Number.isInteger(quantidade) || quantidade < 1) {
      return res.status(400).json({ mensagem: "limite deve ser um inteiro positivo" });
    }
    resultado = resultado.slice(0, quantidade);
  }
  res.json(resultado);
});

app.get("/partidas/:id", (req, res) => {
  const partida = PARTIDAS.find(p => p.id === Number(req.params.id));
  if (!partida) return res.status(404).json({ mensagem: "Partida não encontrada" });
  res.json(partida);
});

app.post("/partidas", (req, res) => {
  const { jogo, timeA, timeB } = req.body;
  if (typeof jogo !== "string" || !jogo.trim() ||
      typeof timeA !== "string" || !timeA.trim() ||
      typeof timeB !== "string" || !timeB.trim()) {
    return res.status(400).json({ mensagem: "Informe jogo, timeA e timeB" });
  }
  if (timeA.trim().toLowerCase() === timeB.trim().toLowerCase()) {
    return res.status(400).json({ mensagem: "Escolha dois times diferentes" });
  }
  const novoId = PARTIDAS.length ? Math.max(...PARTIDAS.map(p => p.id)) + 1 : 1;
  const partida = {
    id: novoId, jogo: jogo.trim(), timeA: timeA.trim(), timeB: timeB.trim(),
    pontosA: 0, pontosB: 0, status: "agendada"
  };
  PARTIDAS.push(partida);
  res.status(201).json({ mensagem: "Partida cadastrada", partida });
});

app.put("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const indice = PARTIDAS.findIndex(p => p.id === id);
  if (indice === -1) return res.status(404).json({ mensagem: "Partida não encontrada" });
  const { jogo, timeA, timeB, pontosA, pontosB, status } = req.body;
  if (typeof jogo !== "string" || !jogo.trim() ||
      typeof timeA !== "string" || !timeA.trim() ||
      typeof timeB !== "string" || !timeB.trim() ||
      timeA.trim().toLowerCase() === timeB.trim().toLowerCase() ||
      !Number.isInteger(pontosA) || pontosA < 0 ||
      !Number.isInteger(pontosB) || pontosB < 0 ||
      !["agendada", "finalizada"].includes(status)) {
    return res.status(400).json({ mensagem: "Dados inválidos: confira times, placar e status" });
  }
  PARTIDAS[indice] = { id, jogo: jogo.trim(), timeA: timeA.trim(), timeB: timeB.trim(), pontosA, pontosB, status };
  res.json({ mensagem: "Partida atualizada", partida: PARTIDAS[indice] });
});

app.delete("/partidas/:id", (req, res) => {
  const indice = PARTIDAS.findIndex(p => p.id === Number(req.params.id));
  if (indice === -1) return res.status(404).json({ mensagem: "Partida não encontrada" });
  const [partida] = PARTIDAS.splice(indice, 1);
  res.json({ mensagem: "Partida excluída", partida });
});

const PORTA = 3000;
app.listen(PORTA, () => console.log(`API em http://localhost:${PORTA}`));
```

**Checklist final:** liste partidas, filtre por time, busque ID inexistente, cadastre, atualize o placar e exclua uma partida. Observe método, URL, JSON e status em cada teste.
