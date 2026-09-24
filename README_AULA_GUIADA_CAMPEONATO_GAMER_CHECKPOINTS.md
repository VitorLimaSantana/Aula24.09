# 🎮 Aula guiada — Vamos criar uma API de Campeonato Gamer

**Turma:** 2º ano do Ensino Médio  
**Tecnologias:** Node.js, Express, JavaScript e Postman ou Insomnia  
**Entrega:** projeto `campeonato-gamer` com o arquivo `server.js`

> Professor: mostre primeiro a situação-problema. Escreva cada bloco com a turma e faça a pausa de teste antes de mostrar o próximo. O arquivo começa pequeno e cresce até virar um CRUD.

## A situação-problema

Nossa turma vai organizar partidas de jogos. Queremos registrar **jogo**, **time A**, **time B**, **placar** e **status**. Uma partida começa `agendada` com placar `0 x 0`. Depois podemos atualizar o resultado para `finalizada`.

Ao final, a API terá estas rotas:

| Método | Rota | O que faz |
| --- | --- | --- |
| GET | `/` | Confirma que a API está funcionando |
| GET | `/partidas` | Lista e filtra partidas |
| GET | `/partidas/:id` | Busca uma partida |
| POST | `/partidas` | Cadastra uma partida |
| PUT | `/partidas/:id` | Altera uma partida e o placar |
| DELETE | `/partidas/:id` | Exclui uma partida |
| GET | `/estatisticas` | Resume o campeonato |
| GET | `/classificacao` | Mostra os pontos dos times |

**Combinado para esta aula:** usaremos um array no lugar do banco de dados. Os cadastros feitos durante a execução são perdidos quando reiniciamos o servidor.

---

## Etapa 1 — Criar o projeto

No terminal, digite **uma linha por vez**:

```bash
mkdir campeonato-gamer
cd campeonato-gamer
npm init -y
npm install express cors
code .
```

Explique: `npm init -y` cria o `package.json`; `npm install` instala as bibliotecas; `code .` abre a pasta no VS Code.

**⏸️ Pausa 1:** confira se há `package.json`, `package-lock.json` e `node_modules`. Pergunte: qual arquivo registra as dependências do projeto?

## Etapa 2 — Fazer o servidor responder

Crie `server.js`. Escreva:

```js
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

app.get("/", (req, res) => {
  res.json({ mensagem: "Campeonato Gamer no ar!" });
});

const PORTA = 3000;
app.listen(PORTA, () => {
  console.log(`Servidor em http://localhost:${PORTA}`);
});
```

Explique enquanto escreve: `app` é a aplicação; `app.get` cria uma rota; `req` representa o pedido; `res` envia a resposta; `express.json()` permite ler JSON enviado no corpo da requisição.

Execute:

```bash
node server.js
```

**⏸️ Pausa 2:** abra `http://localhost:3000`. Resultado esperado: `{"mensagem":"Campeonato Gamer no ar!"}`. Quando editar o arquivo nas próximas etapas, use `Ctrl+C` no terminal e rode `node server.js` novamente.

## Etapa 3 — Criar dados de exemplo

**Onde escrever:** antes da rota `app.get("/", ...)`, adicione:

```js
let PARTIDAS = [
  {
    id: 1,
    jogo: "Arena Pixel",
    timeA: "Falcões",
    timeB: "Dragões",
    pontosA: 3,
    pontosB: 2,
    status: "finalizada"
  },
  {
    id: 2,
    jogo: "Corrida Turbo",
    timeA: "Lobos",
    timeB: "Falcões",
    pontosA: 0,
    pontosB: 0,
    status: "agendada"
  }
];
```

Pergunte: por que cada partida precisa de um `id`? Qual a diferença entre o número `0` e o texto `"0"`?

## Etapa 4 — Listar as partidas (READ)

**Onde escrever:** depois da rota `/` e **antes** de `const PORTA`, adicione:

```js
app.get("/partidas", (req, res) => {
  res.json(PARTIDAS);
});
```

**⏸️ Pausa 3:** reinicie o servidor. Acesse `GET http://localhost:3000/partidas`. Devem aparecer as duas partidas. Pergunte: qual parte de CRUD corresponde ao `GET`?

## Etapa 5 — Buscar pelo ID (READ)

**Onde escrever:** abaixo da rota que lista as partidas:

```js
app.get("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const partida = PARTIDAS.find(p => p.id === id);

  if (!partida) {
    return res.status(404).json({ mensagem: "Partida não encontrada" });
  }

  res.json(partida);
});
```

Explique: `/partidas/2` fornece o parâmetro `id`; `req.params.id` chega como texto; `Number()` converte; `find()` procura o objeto; `404` indica que não foi encontrado. O `return` impede que a rota tente responder duas vezes.

**⏸️ Pausa 4:** teste `GET /partidas/1`, `GET /partidas/2` e `GET /partidas/99`. Peça que os alunos expliquem o resultado de cada requisição.

## Etapa 6 — Cadastrar uma partida (CREATE)

Agora usaremos **Postman ou Insomnia** para enviar JSON. Abaixo da busca por ID, escreva:

```js
app.post("/partidas", (req, res) => {
  const { jogo, timeA, timeB } = req.body;

  if (!jogo || !timeA || !timeB) {
    return res.status(400).json({
      mensagem: "Informe jogo, timeA e timeB"
    });
  }

  if (timeA.trim().toLowerCase() === timeB.trim().toLowerCase()) {
    return res.status(400).json({
      mensagem: "Os times devem ser diferentes"
    });
  }

  const novoId = PARTIDAS.length > 0
    ? Math.max(...PARTIDAS.map(p => p.id)) + 1
    : 1;

  const novaPartida = {
    id: novoId,
    jogo: jogo.trim(),
    timeA: timeA.trim(),
    timeB: timeB.trim(),
    pontosA: 0,
    pontosB: 0,
    status: "agendada"
  };

  PARTIDAS.push(novaPartida);
  res.status(201).json({
    mensagem: "Partida cadastrada",
    partida: novaPartida
  });
});
```

Escreva em partes e pergunte: de onde vêm `jogo`, `timeA` e `timeB`? Por que o placar começa em zero? O que `push()` faz? `map()` obtém os IDs e `Math.max()` escolhe o maior; somamos 1. Usamos `400` para dados inválidos e `201` para criação.

**⏸️ Pausa 5:** selecione **POST** `http://localhost:3000/partidas`, Body → JSON:

```json
{
  "jogo": "Batalha Neon",
  "timeA": "Raios",
  "timeB": "Cometas"
}
```

Confira o status `201` e faça `GET /partidas`. Em seguida, tente cadastrar sem `jogo` e observe o `400`. Use nomes de jogos criados pelos alunos na próxima tentativa.

## Etapa 7 — Atualizar o placar (UPDATE)

Uma requisição `PUT` enviará **todos os dados da partida**. Abaixo do `POST`, escreva:

```js
app.put("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const indice = PARTIDAS.findIndex(p => p.id === id);

  if (indice === -1) {
    return res.status(404).json({ mensagem: "Partida não encontrada" });
  }

  const { jogo, timeA, timeB, pontosA, pontosB, status } = req.body;

  if (!jogo || !timeA || !timeB ||
      !Number.isInteger(pontosA) || pontosA < 0 ||
      !Number.isInteger(pontosB) || pontosB < 0 ||
      !["agendada", "finalizada"].includes(status)) {
    return res.status(400).json({
      mensagem: "Envie jogo, times, placares válidos e status"
    });
  }

  PARTIDAS[indice] = {
    id,
    jogo,
    timeA,
    timeB,
    pontosA,
    pontosB,
    status
  };

  res.json({
    mensagem: "Partida atualizada",
    partida: PARTIDAS[indice]
  });
});
```

Explique: `find()` traz o objeto; `findIndex()` traz a **posição** do objeto no array. `Number.isInteger()` impede placares como `2.5` e `"2"`. O `id` vem da URL; os novos dados vêm do JSON. Um empate é permitido.

**⏸️ Pausa 6:** envie **PUT** `http://localhost:3000/partidas/2` com Body → JSON:

```json
{
  "jogo": "Corrida Turbo",
  "timeA": "Lobos",
  "timeB": "Falcões",
  "pontosA": 4,
  "pontosB": 1,
  "status": "finalizada"
}
```

Confira com `GET /partidas/2`. Tente atualizar `/partidas/99` e teste um placar negativo.

## Etapa 8 — Excluir uma partida (DELETE)

Abaixo do `PUT`, escreva:

```js
app.delete("/partidas/:id", (req, res) => {
  const id = Number(req.params.id);
  const indice = PARTIDAS.findIndex(p => p.id === id);

  if (indice === -1) {
    return res.status(404).json({ mensagem: "Partida não encontrada" });
  }

  const [partidaExcluida] = PARTIDAS.splice(indice, 1);
  res.json({
    mensagem: "Partida excluída",
    partida: partidaExcluida
  });
});
```

Explique: `splice(indice, 1)` remove **um** item daquela posição. Como devolve um array, usamos `[partidaExcluida]` para pegar o item removido.

**⏸️ Pausa 7:** envie `DELETE /partidas/1`, depois `GET /partidas/1`. O segundo teste deve retornar `404`.

## Etapa 9 — Pesquisar com Query String

**Onde alterar:** encontre a rota antiga `app.get("/partidas", ...)` da etapa 4. **Substitua essa rota inteira** pelo código abaixo. Não mantenha duas rotas `GET /partidas`.

```js
app.get("/partidas", (req, res) => {
  const { jogo, time, status, limite } = req.query;
  let resultado = [...PARTIDAS];

  if (jogo) {
    resultado = resultado.filter(p =>
      p.jogo.toLowerCase().includes(jogo.toLowerCase())
    );
  }

  if (time) {
    resultado = resultado.filter(p =>
      p.timeA.toLowerCase().includes(time.toLowerCase()) ||
      p.timeB.toLowerCase().includes(time.toLowerCase())
    );
  }

  if (status) {
    resultado = resultado.filter(p => p.status === status);
  }

  if (limite !== undefined) {
    const quantidade = Number(limite);
    if (!Number.isInteger(quantidade) || quantidade < 1) {
      return res.status(400).json({
        mensagem: "limite deve ser um número inteiro maior que zero"
      });
    }
    resultado = resultado.slice(0, quantidade);
  }

  res.json(resultado);
});
```

Explique: `req.query` lê o que aparece depois do `?`; `filter()` cria um array filtrado; `toLowerCase()` ignora maiúsculas na pesquisa por jogo e time; `includes()` procura um trecho do texto; `slice(0, quantidade)` seleciona os primeiros resultados. A cópia `[...PARTIDAS]` preserva os dados originais.

**⏸️ Pausa 8 — teste no navegador:**

```text
GET http://localhost:3000/partidas?jogo=arena
GET http://localhost:3000/partidas?time=falc
GET http://localhost:3000/partidas?status=finalizada
GET http://localhost:3000/partidas?status=finalizada&limite=1
GET http://localhost:3000/partidas?limite=abc
```

Reiniciar o servidor restaura as duas partidas iniciais. Pergunte: que diferença há entre `/partidas/2` e `/partidas?limite=2`?

## Checkpoint 1 — Estatísticas ao vivo

**Situação:** a turma quer saber quantas partidas já terminaram. Antes de escrever, peça aos alunos que calculem os resultados esperados com os dois registros iniciais: **total 2, agendadas 1 e finalizadas 1**.

**Onde escrever:** acrescente esta rota depois de `app.get("/partidas", ...)` e antes de `const PORTA`:

```js
app.get("/estatisticas", (req, res) => {
  const total = PARTIDAS.length;
  const agendadas = PARTIDAS.filter(p => p.status === "agendada").length;
  const finalizadas = PARTIDAS.filter(p => p.status === "finalizada").length;

  res.json({ total, agendadas, finalizadas });
});
```

Explique: `filter()` seleciona partidas de um status; `.length` conta quantas foram selecionadas. Os valores vêm do array em tempo real.

**⏸️ Checkpoint:** faça `GET http://localhost:3000/estatisticas`. Cadastre uma partida nova e consulte novamente: o total e as agendadas devem aumentar em 1. Finalize essa nova partida com o `PUT` e consulte mais uma vez: agendadas deve diminuir em 1 e finalizadas aumentar em 1.

**Para completar no quadro:** `PARTIDAS.filter(p => p.status === "__________").length`.

## Checkpoint 2 — Pesquisar quem venceu

**Situação:** alguém pergunta: “Quais partidas os Falcões ganharam?” Vamos criar `?vencedor=Falcões` usando as partidas **finalizadas**. Empate e partida agendada não têm vencedor.

**Onde alterar:** na rota `app.get("/partidas", ...)` da etapa 9, adicione `vencedor` na desestruturação:

```js
const { jogo, time, status, limite, vencedor } = req.query;
```

**Ainda dentro da mesma rota**, coloque o bloco abaixo **antes** do bloco `if (limite !== undefined)`. Não crie outra rota `GET /partidas`:

```js
if (vencedor) {
  resultado = resultado.filter(p => {
    if (p.status !== "finalizada" || p.pontosA === p.pontosB) {
      return false;
    }

    const timeVencedor = p.pontosA > p.pontosB ? p.timeA : p.timeB;
    return timeVencedor.toLowerCase().includes(vencedor.toLowerCase());
  });
}
```

Explique: primeiro descartamos partidas que ainda não terminaram e empates. O operador `? :` escolhe o time com maior placar. Depois pesquisamos no nome do vencedor.

**⏸️ Checkpoint:** reinicie o servidor para recuperar as duas partidas iniciais. Teste `GET /partidas?vencedor=Falcões` (deve trazer a partida 1) e `GET /partidas?vencedor=Lobos` (deve trazer um array vazio). Finalize a partida 2 com vitória dos Lobos e teste novamente.

**Pergunta à turma:** o que aconteceria se removêssemos a condição `p.pontosA === p.pontosB`?

## Checkpoint 3 — Classificação dos times

**Situação:** vamos atribuir **3 pontos por vitória**, **1 por empate** e **0 por derrota**. Partidas agendadas não contam. A rota mostrará uma tabela ordenada por pontos.

**Onde escrever:** acrescente a rota abaixo **antes de `const PORTA`**. Ela usa o mesmo array `PARTIDAS`:

```js
app.get("/classificacao", (req, res) => {
  const tabela = {};

  for (const partida of PARTIDAS) {
    if (!tabela[partida.timeA]) tabela[partida.timeA] = 0;
    if (!tabela[partida.timeB]) tabela[partida.timeB] = 0;

    if (partida.status !== "finalizada") continue;

    if (partida.pontosA > partida.pontosB) {
      tabela[partida.timeA] += 3;
    } else if (partida.pontosB > partida.pontosA) {
      tabela[partida.timeB] += 3;
    } else {
      tabela[partida.timeA] += 1;
      tabela[partida.timeB] += 1;
    }
  }

  const classificacao = Object.entries(tabela)
    .map(([time, pontos]) => ({ time, pontos }))
    .sort((a, b) => b.pontos - a.pontos || a.time.localeCompare(b.time));

  res.json(classificacao);
});
```

Explique por partes: `tabela` guarda os pontos pelo nome do time; `continue` pula partidas agendadas; `Object.entries()` transforma a tabela em pares; `map()` cria objetos `{ time, pontos }`; `sort()` coloca os mais pontuados primeiro e usa o nome para desempatar.

**⏸️ Checkpoint:** reinicie o servidor e acesse `GET http://localhost:3000/classificacao`. Como só a partida 1 terminou, o esperado é **Falcões: 3, Dragões: 0, Lobos: 0**. Atualize a partida 2 para empate e confira: Falcões passa a 4 e Lobos a 1. Exclua a partida 1 e consulte novamente: Falcões e Lobos ficam com 1 ponto; Dragões deixa de aparecer, pois só estava na partida excluída. Use essa pergunta para discutir de onde vêm os times da classificação.

**Desafio rápido:** como você acrescentaria `vitorias` e `empates` à resposta de cada time?

## Etapa 10 — Quadro de revisão

| Código | Vem de onde? | Exemplo |
| --- | --- | --- |
| `req.params.id` | Caminho da URL | `/partidas/2` |
| `req.query.time` | Depois do `?` | `/partidas?time=Lobos` |
| `req.body.jogo` | JSON enviado pelo cliente | `POST /partidas` |

| Status HTTP | Significado |
| --- | --- |
| `200` | Consulta, atualização ou exclusão concluída |
| `201` | Partida cadastrada |
| `400` | Dados inválidos |
| `404` | Partida não encontrada |

**Perguntas para a turma:**

1. Qual método HTTP usamos para cadastrar uma partida?
2. Por que `req.params.id` passa por `Number()`?
3. Qual método usamos para encontrar a posição de uma partida?
4. O que ocorre com os novos cadastros ao reiniciar o servidor?
5. Como pesquisar partidas de um time sem saber o ID?

## Desafio individual

Amplie a rota `GET /estatisticas` para também responder com `empates`:

```json
{
  "total": 2,
  "agendadas": 1,
  "finalizadas": 1,
  "empates": 0
}
```

Os números devem ser **calculados a partir do array**, não escritos manualmente. Conte empates apenas entre partidas finalizadas (`pontosA === pontosB`).

**Entrega:** pasta do projeto com `server.js` e `package.json`, mais uma captura de tela de uma requisição `POST`, uma `GET` com filtro e uma `PUT` no Postman ou Insomnia. A pasta `node_modules` não precisa ser enviada.