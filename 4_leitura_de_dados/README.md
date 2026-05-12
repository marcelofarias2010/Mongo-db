# Leitura de dados (MongoDB)

Esta seção cobre o **R** do CRUD — **Read** — em especial o método **`find`**, com filtros e **operadores**. O curso divide o tema em **várias sessões**: primeiro o essencial para montar uma aplicação com consultas razoáveis; depois, operadores e padrões mais avançados. Em projetos reais, **leitura costuma ser a operação mais frequente** em relação às demais.

Para comandos copiáveis e exercícios, use também **`comandos.md`** nesta pasta.

---

## 1. Preparação: importar o banco de livros

Para praticar com **muitos documentos** sem cadastrar manualmente, importe o arquivo **`books.json`** (material do curso — pasta desta seção).

No terminal **na pasta** onde está o arquivo:

```text
mongoimport books.json -d booksCollection -c books
```

Depois, no **`mongosh`**:

```text
show dbs
use booksCollection
db.books.find()
```

O nome do banco (`booksCollection`) e da collection (`books`) devem bater com o que você usou no `mongoimport`. Ajuste se no seu material os nomes forem outros.

---

## 2. `find()` — todos os documentos

```text
db.books.find()
db.books.find({})
```

- Sem filtro (ou com **documento vazio `{}`**), o MongoDB retorna **todos** os documentos que a consulta enxergar (sujeito a limites de cursor — ver abaixo).
- **`pretty()`** formata a saída no terminal:

```text
db.books.find().pretty()
```

Útil quando há textos longos (ex.: descrições de livros).

---

## 3. Cursor e lotes (`it`)

**`find()`** (sem ser `findOne`) devolve um **cursor**, não “todos os documentos de uma vez” em uma única impressão. O shell pode mostrar um **primeiro lote** de resultados; para pedir **mais**, digite:

```text
it
```

Isso ajuda o servidor e o cliente a não descarregarem de uma vez um volume enorme de dados — ideia parecida com **paginação**.

**`findOne()`** devolve **um** documento (ou `null`) e encerra — comportamento diferente do fluxo com cursor + `it`.

---

## 4. Filtro por igualdade — primeiro argumento do `find`

O **primeiro argumento** de `find` é o **filtro** (um **documento**). Campos no mesmo nível funcionam como **E lógico** (AND): todos os critérios precisam ser verdadeiros no **mesmo** documento.

### Exemplo: número exato de páginas

```text
db.books.find({ pageCount: 362 })
```

### Exemplo: título exato

```text
db.books.find({ title: "The Internet in Action" })
```

Strings entre **aspas**. A busca é por **valor igual** ao informado.

Para um único resultado esperado, você pode usar **`findOne`** em vez de `find`.

---

## 5. Exercício 7

1. Encontrar o livro cujo **`title`** seja **`MongoDB in Action`** (ajuste aspas/título exatamente como no dataset).
2. Encontrar livros do autor **`Jason R. Weiss`**.

**Dica:** o campo **`authors`** pode ser um **array** com vários nomes. No MongoDB, ao filtrar com `{ authors: "Jason R. Weiss" }`, basta que **um** dos elementos do array seja igual ao valor — não precisa ser o único autor.

Solução de referência em **`comandos.md`** (EXERCICIO 7).

---

## 6. Operador `$in` — um entre vários valores

Útil quando um campo pode ter **vários valores possíveis** e você quer documentos em que o campo seja **qualquer um** da lista (OR **dentro do mesmo campo**).

```text
db.books.find({
  categories: { $in: ["Java", "Internet"] }
}).pretty()
```

Sintaxe típica: **`$in`** com **array** de valores. Strings continuam entre aspas.

**Atenção às chaves:** o filtro tem documentos aninhados — conferir abertura/fechamento de `{}` e `[]`. Para comandos grandes, use um editor (VS Code), como na dica da seção de inserção de dados.

---

## 7. Várias condições no mesmo documento (AND)

No **mesmo** documento de filtro, **várias chaves** separadas por vírgula significam **todas** verdadeiras:

```text
db.books.find({ pageCount: 592, _id: 63 }).pretty()
```

Só entram resultados que tenham **592 páginas** **e** **`_id` igual a 63** (no seu dataset, tipos numéricos ou ObjectId conforme o schema).

Se você alterar só um dos critérios e não existir documento que satisfaça **ambos**, o resultado vem **vazio**.

---

## 8. Operador `$gt` — maior que (*greater than*)

Operadores de comparação costumam usar **`$`** no nome e ficam em um **subdocumento** do campo:

```text
db.books.find({ pageCount: { $gt: 450 } }).pretty()
```

Livros com **`pageCount` estritamente maior que 450**.

Você pode combinar com outros campos no mesmo filtro (AND), por exemplo categoria **e** página:

```text
db.books.find({
  pageCount: { $gt: 450 },
  categories: "Java"
}).pretty()
```

(Se `categories` for array, a igualdade por string também casa quando um dos elementos coincide — comportamento padrão.)

---

## 9. Exercício 8

1. Livros com **mais de 800 páginas**: use **`pageCount`** com **`$gt: 800`**.
2. Livros cujo **`_id`** seja **maior que 122** (tratado como comparável numericamente no exemplo — depende de como o `_id` está no arquivo importado).

Referência em **`comandos.md`** (EXERCICIO 8).

---

## 10. Operador `$lt` — menor que (*less than*)

```text
db.books.find({ pageCount: { $lt: 120 } }).pretty()
```

Mesmo padrão do `$gt`, trocando o operador.

### Importante: não repita a mesma chave duas vezes no mesmo objeto

Em JavaScript/MongoDB, um documento é um objeto: **se você escrever o mesmo campo duas vezes**, por exemplo:

```text
{ pageCount: { $gt: 1 }, pageCount: { $lt: 120 } }
```

**só vale o último** — o primeiro é sobrescrito. Por isso **intervalo** (“entre X e Y”) no **mesmo campo** deve ir **no mesmo subdocumento**:

```text
db.books.find({ pageCount: { $gt: 1, $lt: 120 } })
```

Para combinações mais complexas, existem **`$and`**, **`$or`**, etc. (outras sessões reforçam isso.)

---

## 11. Operador `$or` — uma condição **ou** outra (campos diferentes)

Quando você quer documentos que satisfaçam **pelo menos uma** entre várias condições **entre campos diferentes** (ou critérios independentes), use **`$or`** com um **array** de documentos:

```text
db.books.find({
  $or: [
    { pageCount: { $gt: 500 } },
    { _id: { $lt: 5 } }
  ]
}).pretty()
```

Retorna livros com **mais de 500 páginas** **ou** com **`_id` menor que 5** (ajuste conforme o tipo real do `_id` no seu banco).

---

## 12. Combinar AND implícito com `$or`

Você pode exigir **uma condição fixa** (AND) **e** ainda assim permitir **alternativas** com `$or`:

```text
db.books.find({
  status: "PUBLISH",
  $or: [
    { pageCount: 500 },
    { authors: "Robi Sen" }
  ]
}).pretty()
```

Interpretação típica da aula: **status** deve ser publicado **e** ( **páginas exatamente 500** **ou** **autor** igual ao nome indicado ). Os nomes exatos dos campos (`status`, `authors`, `pageCount`) dependem do JSON importado — confira com `findOne()` em um documento real.

*(No arquivo `comandos.md` do projeto aparece `"Robi Sen"`; na fala da aula pode soar como “Rob 100” — use sempre o valor que existir nos seus dados.)*

---

## 13. Contar resultados — `count`

Para saber **quantos documentos** batem com o filtro, encadeie **`.count()`** ao cursor:

```text
db.books.find({ pageCount: { $gt: 600 } }).count()
```

Retorna um **número inteiro** (ex.: quantos livros têm mais de 600 páginas). Útil para dashboards (“quantos livros cadastrados com esse critério?”).

**Nota:** em versões recentes do MongoDB, a API recomendada em aplicações costuma ser **`countDocuments()`** no driver ou no shell; em material antigo do curso aparece **`.count()`** no cursor — mantenha compatibilidade com o que o instrutor usar na versão do servidor/shell.

---

## Resumo dos operadores citados

| Operador | Significado (nesta seção) |
|----------|---------------------------|
| `$in` | Valor do campo é um dos listados no array |
| `$gt` | Maior que |
| `$lt` | Menor que |
| `$or` | Pelo menos uma das condições do array é verdadeira |

**Regras rápidas:**

- Várias chaves **no mesmo nível** do filtro ⇒ **AND**.
- **`$or`** ⇒ OR entre subdocumentos listados.
- Intervalo no mesmo campo ⇒ **`{ campo: { $gt: a, $lt: b } }`**, não duas chaves `campo` repetidas.

---

## Próximos passos do curso

Operadores adicionais, agregações e consultas mais elaboradas aparecem nas **próximas sessões** de leitura — esta parte cobre o fundamental para **filtrar**, **combinar critérios** e **contar** resultados sobre um dataset real (livros).
