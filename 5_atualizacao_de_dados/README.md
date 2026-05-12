# Atualização de dados (MongoDB)

Esta sessão trata do **U** do CRUD — **Update**: alterar documentos que já existem no banco. Na prática, **leitura** costuma ser a operação mais frequente; **atualização** aparece no mesmo patamar que **criação** para muitos sistemas — **remoção** tende a ser menos usada.

O tema é amplo: esta parte cobre o **fundamento** (`updateOne`, `updateMany`, `$set`, campos novos, `replaceOne`, `$push`, atualização global). **Outras sessões** do curso aprofundam métodos e operadores mais avançados.

Para comandos copiáveis e tarefas, use **`comandos.md`** nesta pasta.

**Pré-requisito:** banco **`booksCollection`** com collection **`books`** (ex.: importação com `mongoimport` conforme a seção de leitura de dados).

---

## 1. Estrutura dos comandos de atualização

A lógica é parecida com a do **`find`**:

1. **Primeiro argumento** — **filtro** (documento): define **quais** documentos serão afetados (como no “select antes de atualizar”).
2. **Segundo argumento** — o que fazer na atualização; em geral usa-se o operador **`$set`** para definir ou alterar valores de campos.

```text
db.books.updateOne(
  { /* filtro — critérios para achar o(s) documento(s) */ },
  { $set: { /* campos e novos valores */ } }
)
```

- **`updateOne`** — no máximo **um** documento que combine com o filtro (o primeiro encontrado, segundo as regras do servidor).
- **`updateMany`** — **todos** os documentos que combinarem com o filtro.

Sem um filtro bem definido, você pode alterar mais registros do que pretendia — sempre revise o filtro.

---

## 2. `updateOne` com `$set`

Exemplo da aula: atualizar **um** livro identificado pelo **`_id`** (ajuste o número ao seu dataset):

```text
db.books.updateOne(
  { _id: 314 },
  { $set: { pageCount: 1000 } }
)
```

O shell costuma indicar se houve **documento modificados**. Confira com:

```text
db.books.findOne({ _id: 314 }).pretty()
```

Assim você valida que, por exemplo, **`pageCount`** passou de 600 para 1000.

**Ideia central:** o **filtro** escolhe **qual** registro atualizar; **`$set`** diz **quais campos** mudar (podem ser **vários campos** no mesmo `$set`).

---

## 3. Exercício 9

1. Localizar o livro com **`_id`** igual a **`20`** (ou o critério que o curso usar).
2. Atualizar o **`title`** para **`"Meu primeiro update"`**.
3. Buscar de novo com **`find`** / **`findOne`** para confirmar a alteração.

Referência em **`comandos.md`** (EXERCICIO 9).

---

## 4. `updateMany`

Mesma ideia de dois argumentos (filtro + atualização), mas **todos** os documentos que satisfizerem o filtro são atualizados.

Exemplo conceitual da aula: livros cuja categoria inclui **Java** — alterar **`status`** para um novo valor (no material do projeto aparece algo como **`UNPUBLISHED`**):

```text
db.books.updateMany(
  { categories: "Java" },
  { $set: { status: "UNPUBLISHED" } }
)
```

O retorno pode mostrar **quantos documentos** foram modificados (ex.: dezenas de livros de uma vez).

**Observação:** se o filtro só encontrar **um** documento, só **um** será atualizado — o método continua sendo `updateMany`, mas o efeito é sobre um registro. Quando você **sabe** que quer um só, **`updateOne`** deixa a intenção mais clara.

---

## 5. Adicionar campos novos com `$set`

No modelo de documentos do MongoDB, **não** é obrigatório que todos os documentos tenham os mesmos campos. Você pode **criar um campo novo** em documentos já existentes usando **`$set`** com uma chave que ainda não existia.

Exemplo da aula: todos os livros do autor **`Vikram Goyal`** passam a ter **`downloads: 1000`**:

```text
db.books.updateMany(
  { authors: "Vikram Goyal" },
  { $set: { downloads: 1000 } }
)
```

Antes do update, os documentos podiam não ter **`downloads`**; depois, o campo aparece só onde o filtro bateu. Isso evita o trabalho de “criar coluna em toda a tabela” típico de modelos relacionais rígidos.

---

## 6. Exercício 10

1. Para **todos** os livros com **`pageCount` maior que 500**, adicionar ou definir um campo (ex.: **`bestseller: true`** — booleano).
2. Verificar a alteração com **`find`** nos mesmos critérios ou usando **`count`** / contagens para ver se o número de documentos condiz com o que o `updateMany` reportou.

Referência em **`comandos.md`** (EXERCICIO 10).

---

## 7. `replaceOne` — substituir o documento inteiro

**`replaceOne`** não apenas altera alguns campos: ele **substitui o documento** pelo novo documento informado (exceto **`_id`**, que em geral **se mantém** como referência).

```text
db.books.replaceOne(
  { _id: 120 },
  { foi: "substituido" }
)
```

Depois disso, aquele documento pode conter **apenas** as chaves que você passou no segundo argumento (mais o **`_id`** preservado). **Todo o resto** (título, ISBN, categorias, etc.) **some** — por isso use com muito cuidado.

É útil quando você **quer mesmo** trocar o conteúdo inteiro por outro documento; para ajustes pontuais, prefira **`$set`**.

---

## 8. Arrays: `$push` vs `$set`

- Com **`$set`** em um campo que é **array**, você pode **substituir o array inteiro**. Se passar uma **string** em vez de array, o campo pode deixar de ser array — **erro de modelagem**.
- Para **acrescentar um elemento** a um array existente, use **`$push`**:

```text
db.books.updateOne(
  { _id: 201 },
  { $push: { categories: "PHP" } }
)
```

Isso **mantém** os valores que já estavam em **`categories`** e **adiciona** `"PHP"`.

Resumo: **atualizar um array “somando” um item** ⇒ **`$push`** (ou operadores de array específicos em sessões avançadas); **trocar o array todo** ⇒ **`$set`** com o novo array completo.

---

## 9. Atualizar “todo o banco” — filtro vazio

```text
db.books.updateMany(
  {},
  { $set: { atualizado: true } }
)
```

Um filtro **`{}`** significa **todos os documentos** da collection — equivalente à ideia de um `find` sem critério. Isso pode alterar **centenas ou milhares** de registros de uma vez.

Use **somente** quando for **intencional** (ex.: migração, flag global). Evite `updateMany({}, …)` “sem querer”: pode ser difícil desfazer sem backup.

---

## 10. Boas práticas: `find` antes do `update`

Antes de **`updateOne`** / **`updateMany`** / **`replaceOne`** em dados sensíveis:

1. Rode um **`find`** (ou **`findOne`**) com o **mesmo filtro** que você pretende usar no update.
2. Confira se são **exatamente** os documentos que devem mudar.
3. Só então execute o update.

Assim você reduz risco de atualizar **`_id`** ou critérios errados e precisar corrigir depois. Em administração pelo shell, esse “passo de selects first” costuma evitar acidentes; na aplicação, a mesma disciplina vale para queries parametrizadas.

---

## Resumo rápido

| Método | Efeito típico |
|--------|----------------|
| **`updateOne`** | Até **um** documento que combine com o filtro |
| **`updateMany`** | **Todos** os documentos que combinarem com o filtro |
| **`$set`** | Define ou altera valores de campos (inclui **criar** campos novos) |
| **`replaceOne`** | Substitui o documento inteiro (cuidado: perde campos não enviados) |
| **`$push`** | Adiciona um elemento a um **array** |

Operações de atualização são poderosas: combine **filtro preciso**, **`find` prévio** quando houver dúvida, e evite **filtro vazio** salvo quando for objetivo atualizar toda a collection.

Para a **tarefa** da pasta (ex.: status de um título específico, livros curtos, `push` em categorias), veja **`comandos.md`** (TAREFA 04).
