# Gerenciamento de bancos de dados (MongoDB)

Esta seção cobre **criação, configuração e verificação** de bancos e collections — as camadas “de fora para dentro” antes de aprofundar em comandos que manipulam mais os dados (por exemplo, na parte de CRUD). Em projetos reais, você costuma criar o banco poucas vezes; mesmo assim, dominar esses comandos evita erros de contexto (banco ou collection errados).


---

## 1. Visão geral da seção

- **Banco de dados** e **collections** são conceitos distintos; o **documento** é o registro dentro da collection.
- **`db`** no shell sempre se refere ao **banco atual**. Confira o contexto antes de inserir ou apagar dados.
- Bancos “de sistema” que aparecem na listagem: **`admin`**, **`config`**, **`local`** — não devem ser removidos em uso normal.

---

## 2. Listar bancos

```text
show dbs
```

Lista todos os bancos persistidos. Você verá os três bancos internos do MongoDB e os bancos que você criou (desde que tenham dados — ver item 3).

---

## 3. Criar banco e “persistência” na lista

Um banco **só passa a aparecer** em `show dbs` quando existe **pelo menos uma collection com pelo menos um documento** (criação implícita ao inserir).

Fluxo típico:

1. **`use <nomeDoBanco>`** — seleciona o banco (se não existir como entidade utilizável ainda, o shell passa a usar esse contexto).
2. **`db.<collection>.insertOne({ ... })`** — insere um documento; isso cria collection e materializa o banco na listagem.

Exemplo de ideia (nomes livres):

```text
use meuBanco
db.minhaColecao.insertOne({ campo: "valor" })
show dbs
```

---

## 4. Qual banco estou usando?

```text
db
```

Exibe o nome do **banco atual**. Use sempre que estiver administrando bancos para não rodar comandos no contexto errado.

---

## 5. Trocar (ou “entrar” em) outro banco

```text
use outroBanco
```

- Serve para **mudar o banco atual**.
- Também é o comando usado no fluxo de **criação** ao escolher o nome e em seguida inserir dados.

---

## 6. Collections e inserção

- Você **não é obrigado** a criar a collection antes: **`insertOne`** (ou inserções equivalentes) **cria a collection automaticamente** se ela não existir.
- **`db.<nomeDaCollection>.insertOne({ ... })`** sempre age sobre a collection **do banco atual**.

É possível ter **várias collections** no mesmo banco, criando-as conforme insere dados em nomes diferentes.

---

## 7. Consultar documentos — `find` e filtro

```text
db.<collection>.find()
db.<collection>.find({ campo: "valorExato" })
```

- Sem filtro (documento vazio ou omitindo o filtro conforme o que você já viu na aula), retorna **todos** os documentos da collection.
- Com filtro `{ campo: "valor" }`, a busca é por **igualdade exata** — não é “contém” (tipo LIKE). Valores diferentes do filtro não aparecem.

---

## 8. Saída legível — `pretty()`

Para documentos com vários campos, arrays ou documentos aninhados, use:

```text
db.<collection>.find().pretty()
db.<collection>.find({ ... }).pretty()
```

Ajuda a ler o resultado no terminal.

---

## 9. Criar collection com opções — `createCollection`

Quando você quer **configurar** a collection (além da criação implícita por insert):

```text
db.createCollection("nomeDaCollection", { ... opções ... })
```

Exemplo citado na aula — collection **limitada (capped)** com tamanho máximo e número máximo de documentos (útil para cenários como **logs**, mantendo só os últimos N registros):

```text
db.createCollection("minhaColecao", { capped: true, size: 1000, max: 50 })
```

- **`capped: true`** habilita o comportamento de coleção limitada.
- **`size`**: limite de tamanho (em bytes) para a collection.
- **`max`**: número máximo de documentos.

Quando o limite é atingido, em uma collection capped o comportamento pode **substituir documentos antigos** (como uma fila circular) — atenção ao significado na sua versão do servidor e ao uso pretendido.

---

## 10. Listar collections do banco atual

```text
show collections
```

Mostra as collections **do banco em que você está** (`use ...`).  
Observação da aula: uma collection criada só com **`createCollection`** pode aparecer mesmo **sem documentos**; já a criada só por insert só “ganha vida” quando há inserção.

---

## 11. Campo `_id`

- Todo documento recebe **`_id`** se você não fornecer um.
- Valores gerados automaticamente costumam ser **`ObjectId`**, únicos, com parte baseada em timestamp de criação — adequados para identificar registros e relacionar collections.
- Há **índice** no `_id`, o que tende a tornar buscas por `_id` muito eficientes em relação a campos sem índice.

---

## 12. Remover uma collection

```text
db.<nomeDaCollection>.drop()
```

- Remove a collection e **todos os documentos**.
- Retorno indica sucesso (`true`) ou que a collection não existia (`false`). **Cuidado em produção** — faça backup quando necessário.

---

## 13. Remover o banco atual

```text
db.dropDatabase()
```

- **Sensível a maiúsculas/minúsculas:** use **`dropDatabase`** com **D** maiúsculo como na documentação do método.
- Apaga o **banco atual** e suas collections. Após remover, `show dbs` não lista mais esse banco; o shell pode ainda mostrar um contexto “virtual” até você mudar com `use`.

---

## Exercícios (resumo)

Os detalhes e variações estão em **`comandos.md`**. Resumo do que as aulas pedem:

| Exercício | Objetivo |
|-----------|----------|
| **2** | Criar um **novo banco** (com `use` + insert que materialize o banco), depois **`show dbs`** para ver todos os bancos. |
| **3** | Criar uma collection com dados de **nome** e **salário** (mínimo ~3 documentos), **`find`** para conferir, **`show collections`** para validar. |
| **4** | **`drop`** na collection indicada (ex.: salários), depois **`db.dropDatabase()`** no banco onde ela estava; se precisar refazer o cenário, recrie banco, collection e um documento antes de repetir o fluxo. |

---

## Importação e exportação (Database Tools)

Instale os **MongoDB Database Tools** (`mongoimport`, `mongoexport`, `mongodump`, `mongorestore`) e use-os no **terminal do sistema** (pasta do projeto aberta no terminal facilita os caminhos dos arquivos). Se algum comando não for encontrado, confira o PATH e a instalação das tools.

### Importar JSON — `mongoimport`

Cria/uso de banco e collection conforme parâmetros; exemplo alinhado ao material:

```text
mongoimport books.json -d booksData -c books
```

### Exportar uma collection — `mongoexport`

```text
mongoexport -c books -d booksData -o booksExample.json
```

### Várias collections — `mongodump` e `mongorestore`

- **`mongodump`**: exporta o banco para um **diretório** (várias collections, metadados).

```text
mongodump -d meuBanco -o meuBanco
```

- **`mongorestore`**: restaura a partir desse diretório.

```text
mongorestore meuBanco
```

Se o banco de destino **já existir** com dados, a restauração pode **falhar ou conflitar** — na aula, a solução foi **`use meuBanco`** + **`db.dropDatabase()`** e então rodar o `mongorestore` de novo.

---

## Monitoramento — `mongostat`

- Rode **`mongostat`** no **terminal**, **fora** do `mongosh` (senão pode não funcionar como esperado).
- Atualiza métricas em tempo real (consultas, rede, etc.). Útil para investigar lentidão.
- **Ctrl+C** encerra e devolve o terminal ao modo normal.

---

## Limpar vários bancos de teste (script no shell)

Para ambientes de curso, às vezes você cria muitos bancos de teste. É possível rodar um **loop em JavaScript** no shell que:

1. Obtém os nomes dos bancos.
2. **Não remove** `admin`, `config` e `local`.
3. Chama **`dropDatabase()`** nos demais.

Guarde esse snippet só para laboratório — **nunca** use em produção sem política clara de backup. Ajuste a API se o seu cliente for `mongosh` e o método global diferir (`getMongo()`, `getSiblingDB()`, etc.).

---

## Boas práticas rápidas

- Confira **`db`** e **`show collections`** antes de operações destrutivas.
- **`drop`** e **`dropDatabase()`** são **irreversíveis** sem backup.
- Separe mentalmente: **banco** → **collection** → **documento** → campos e **`_id`**.

Próximos tópicos do curso costumam aprofundar CRUD, filtros avançados e índices; esta base de gerenciamento evita confusão de contexto nas aulas seguintes.
