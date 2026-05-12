# Removendo dados (MongoDB)

Esta sessão cobre o **D** do CRUD — **Delete** (às vezes chamado de **Destroy** em materiais em inglês): **remover documentos** do banco. A ideia é **muito parecida com atualização**: você usa um **filtro** (documento) para dizer **quais** registros serão afetados; o que muda é o método — em vez de alterar campos, os documentos são **apagados** da collection.

Os padrões **`One`** / **`Many`** repetem os de **`insertOne`** / **`insertMany`** e **`updateOne`** / **`updateMany`**.

Para comandos de referência, veja **`comandos.md`** nesta pasta.

---

## 1. `deleteOne` — remover um documento

Remove **no máximo um** documento que combine com o filtro.

```text
db.books.deleteOne({ _id: 20 })
```

O shell costuma retornar **`deletedCount`** (ex.: `1`) se algo foi removido.

Para conferir que sumiu:

```text
db.books.find({ _id: 20 })
```

Não deve retornar documentos.

**O filtro não precisa ser só `_id`.** Qualquer critério válido de consulta serve — por exemplo, **`isbn`** com o valor exato (string entre **aspas** se o campo for texto):

```text
db.books.deleteOne({ isbn: "1933988713" })
```

(Ajuste o valor ao **ISBN real** do exercício ou do seu `books.json`.)

---

## 2. Operação destrutiva e alternativas

Remover é **irreversível** sem backup — **`delete`** não manda para “lixeira”. Em muitos sistemas prefere-se:

- **Soft delete:** marcar o registro como inativo (`ativo: false`, `deletedAt`, etc.) com **`update`** e **não** listar esses registros nas consultas normais.

Assim mantém-se **histórico** e auditoria. Ainda assim, **hard delete** (remover de verdade) é comum quando a regra de negócio exige.

---

## 3. `deleteMany` — remover vários documentos

Remove **todos** os documentos que satisfizerem o filtro.

Exemplo da aula: remover todos os livros cuja categoria seja **Java**:

```text
db.books.deleteMany({ categories: "Java" })
```

O retorno indica **quantos** documentos foram removidos (na demonstração, dezenas de registros). Se o filtro só encontrar **um** documento, apenas **um** será apagado — mas o método continua sendo **`deleteMany`**.

**Cuidado:** filtros amplos podem apagar muito mais dados do que você imaginava. Revise o filtro antes (use o mesmo critério em **`find`**).

---

## 4. Exercício 11

1. Remover **um** livro pelo **`isbn`** igual ao valor indicado no enunciado/slide (use **`deleteOne`** e aspas se for string).
2. Remover **todos** os livros da categoria **PowerBuilder** (na correção da aula o nome aparece **junto**: `PowerBuilder` — confira no seu dataset como está gravado).
3. Usar **`find`** (ou contagens) para verificar que os registros não existem mais.

Detalhes numéricos exatos dependem do arquivo **`books.json`** do curso; compare com a lista de comandos em **`comandos.md`** se o instrutor atualizar os valores.

---

## 5. `deleteMany` com filtro vazio — esvaziar a collection

```text
db.books.deleteMany({})
```

Um filtro **`{}`** não impõe nenhuma condição: **todos** os documentos da collection são candidatos à remoção — equivalente a um “deletar tudo” na collection. Na aula, isso removeu **centenas** de registros de uma vez; um **`find`** depois não retorna nada.

**Extremamente perigoso** em produção (compare com um **`DELETE` sem `WHERE`** no SQL). Use **somente** quando for **intencional** (limpar ambiente de teste, refazer importação, etc.).

---

## Resumo rápido

| Método | Efeito |
|--------|--------|
| **`deleteOne`** | Remove **até um** documento que combine com o filtro |
| **`deleteMany`** | Remove **todos** os documentos que combinarem com o filtro |
| **`deleteMany({})`** | Remove **todos** os documentos da collection (**filtro vazio**) |

**Boas práticas:**

- Antes de **`deleteMany`**, rode **`find`** com o **mesmo filtro** e confira quantos documentos seriam afetados.
- Evite filtros vazios `{}` salvo quando o objetivo for **realmente** limpar toda a collection.
- Mantenha **backup** ou política de **soft delete** quando o histórico for importante.

Com isso, você fecha o ciclo **CRUD** no MongoDB em nível introdório: **criar**, **ler**, **atualizar** e **remover** dados.
