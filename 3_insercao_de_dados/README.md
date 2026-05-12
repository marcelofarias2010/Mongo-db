# Inserção de dados (MongoDB)

Esta é a **primeira parte do CRUD**: **Create** — criar e inserir documentos nas collections. O curso divide o CRUD em **quatro blocos** (inserção, leitura, atualização e remoção). Leitura e atualização costumam ter **mais variações de parâmetros** (filtros, relatórios, atualizações parciais); por isso há outras sessões dedicadas a esses temas. Aqui o foco é **inserir dados** com os métodos principais e boas práticas.

Para comandos curtos de referência e exercícios, use também **`comandos.md`** nesta pasta.

---

## 1. O que é CRUD?

**CRUD** é o acrônimo de **Create**, **Read**, **Update** e **Delete** (às vezes o “D” é interpretado como *Destroy* — mesma ideia).

| Operação | No dia a dia da aplicação |
|----------|---------------------------|
| **C**reate | Cadastrar cliente, produto, pedido… |
| **R**ead | Listar ou buscar registros |
| **U**pdate | Alterar endereço, status, preço… |
| **D**elete | Excluir conta, remover item… |

No **SQL** você associa ideias a `INSERT`, `SELECT`, `UPDATE`, `DELETE`. No **MongoDB**, cada operação pode ter **vários métodos e parâmetros**, por isso o curso separa bem as sessões.

Quase todo projeto com banco e backend usa CRUD em alguma intensidade — com frequência **leitura** pesa mais que as demais operações.

---

## 2. Documentos (terminologia)

No MongoDB, estruturas entre **`{ }`** usadas em **filtros**, **dados** ou **opções de método** são chamadas de **documentos** (na documentação oficial), não “objetos” — embora a sintaxe lembre JavaScript.

Expressões comuns:

- **Inserir um documento na collection**
- Passar um **documento** como filtro ou como opções (`writeConcern`, etc.)

Analogia com SQL: o **documento** corresponde, em grosso modo, à ideia de uma **linha/dado**; a **collection** lembra uma **tabela**, com a diferença de que o modelo é orientado a documentos.

---

## 3. Inserir um único documento — `insertOne`

**Forma geral:**

```text
db.<nomeDaCollection>.insertOne({ campo: valor, ... })
```

- **`db`** → banco atual (definido com `use nomeDoBanco`).
- **`nomeDaCollection`** → onde os dados serão gravados (equivale ao “destino” da inserção).
- Dentro das chaves: **pares campo: valor**, separados por **vírgula**.
- **Strings** entre aspas; **números** sem aspas; **booleanos** `true` / `false`; **arrays** com `[ ]`; valores aninhados são **subdocumentos**.

**Tipos comuns na mesma inserção:** texto, número, booleano, array (ex.: `hobbies`), etc.

Após inserir, você pode conferir com:

```text
db.<nomeDaCollection>.find()
```

### Esquema flexível na mesma collection

Diferente do modelo relacional estrito, **dois documentos na mesma collection não precisam ter os mesmos campos**. Um registro pode ter só `nome` e `idade`; outro pode ter mais campos — o MongoDB **não obriga** todos a seguirem o mesmo “molde”.

Isso é poderoso, mas pode gerar **inconsistência** se não houver **modelagem** e **aplicação** alinhadas. Boas práticas:

- Definir **quais campos** cada collection deve ter (quando o negócio exige uniformidade).
- Usar a flexibilidade **a favor** do projeto (ex.: novo campo em novos cadastros sem `ALTER TABLE`).

**Exemplo citado na aula:** alunos com `nome` e `notas`; um novo aluno (**José**) pode ser cadastrado **só com `nome`**, sem `notas`, se ainda não houver provas — sem erro por “falta de coluna”.

---

## 4. Exercício 5 — collection `provas`

Objetivo:

1. Criar/usar uma collection chamada **`provas`**.
2. Inserir **dois documentos**, cada um com:
   - nome do aluno;
   - propriedade **`notas`** como **array** com várias notas.
3. Exibir todos os dados com **`find`** (opcional: **`.pretty()`** para leitura mais clara).

Referência de ideia em **`comandos.md`** (exercício 5).

---

## 5. Inserir vários documentos — `insertMany`

**Forma geral:**

```text
db.<nomeDaCollection>.insertMany([ { ... }, { ... }, ... ])
```

- O argumento é um **array** `[ ]`.
- Cada elemento é um **documento** `{ }`, separados por **vírgula**.

**Observações da aula:**

- Em **inserções simultâneas**, os **`_id`** gerados automaticamente são **distintos** (coordenação por tempo nos instantes próximos).
- Você **pode** usar `insertMany` com **um único** documento no array — funciona — mas, para **um** registro só, **`insertOne`** costuma ser mais simples e legível.

---

## 6. Exercício 6 — collection `mercado`

Objetivo:

1. Usar **`insertMany`** na collection **`mercado`**.
2. Inserir **vários produtos** com campos como:
   - **nome**
   - **preço** (números decimais com **ponto**, ex.: `4.50`, `2.99`)
   - **disponibilidade** (quantidade em estoque ou disponível — conforme você modelar)
3. Conferir com **`find()`** e, se quiser, **`.pretty()`**.

---

## 7. Método legado — `insert`

```text
db.<collection>.insert(documentoOuArray)
```

- Aceita **um documento** ou **array de documentos** (inserção única ou múltipla).
- **Retrocompatibilidade:** ainda existe em muitos ambientes, mas **não é o recomendado** para código novo.

Para projetos atuais, prefira:

- **`insertOne`** — um documento;
- **`insertMany`** — vários documentos.

O retorno de **`insert`** difere dos métodos novos (menos detalhado em alguns casos). Em **manutenção** de sistemas antigos, você pode encontrar **`insert`**.

---

## 8. `_id` personalizado

Por padrão, o MongoDB gera **`_id`** (geralmente **`ObjectId`** único).

Você pode **definir o `_id` você mesmo** no documento:

```text
db.<collection>.insertOne({ _id: "1001", nome: "Produto", preco: 14.99 })
```

**Quando faz sentido:** identificadores de negócio estáveis — exemplo da aula: **número de pedido** (`1001`, `1002`…) para comunicação entre setores (ex.: SAC). Aí você controla a sequência ou o formato.

**Cuidado:** valores de **`_id`** devem ser **únicos** na collection; duplicar **`_id`** gera erro.

---

## 9. Opções de escrita — `writeConcern` (ex.: em `insertMany`)

Além do array de documentos, métodos como **`insertMany`** aceitam um **segundo argumento**: um **documento de opções**.

Exemplo discutido na aula — reconhecimento de escrita e tempo máximo de espera:

```text
db.mercado.insertMany(
  [
    { nome: "tesoura", preco: 12.99 },
    { nome: "peito de frango", preco: 14.99 },
    { nome: "ameixa", preco: 3.50 }
  ],
  {
    w: "majority",
    wtimeout: 200
  }
)
```

- **`w`**: nível de confirmação da escrita (ex.: **`"majority"`** — maioria dos membros do replica set confirma).
- **`wtimeout`**: tempo máximo em **milissegundos**; se a operação ultrapassar, pode ocorrer **timeout** — útil para **não travar** o servidor em inserções pesadas ou rede lenta.

Na demonstração com poucos dados e tempo curto, o timeout pode **não** disparar; o importante é saber que existe para **produção** e cargas grandes. Consulte a documentação oficial de **Write Concern** para detalhes e valores.

Mais exemplos em **`comandos.md`** (seção write concern).

---

## 10. Dica: comandos longos no editor (VS Code)

Para **`insertMany`** grandes ou vários documentos aninhados:

1. Abra um arquivo **`.js`** no editor.
2. Escreva o comando com **quebras de linha** e indentação (como código JavaScript).
3. Revise parênteses, colchetes e chaves com ajuda do editor.
4. **Copie** tudo e **cole** no **`mongosh`** (no Windows, às vezes **botão direito** cola no terminal).

Isso reduz erros de sintaxe em relação a digitar tudo em uma linha só no shell.

---

## Resumo rápido

| Método | Uso principal |
|--------|----------------|
| **`insertOne`** | Um documento por chamada |
| **`insertMany`** | Vários documentos (array) |
| **`insert`** | Legado; um ou vários — evitar em código novo |

**Conceitos:** **`db`** = banco atual; dados e opções em formato de **documento** `{ }`; esquema **flexível** exige **disciplina de modelagem**; **`_id`** pode ser automático ou **manual**; **`writeConcern`** ajuda a controlar **durabilidade** e **timeout** da escrita.

Nas próximas sessões do curso entram **leitura**, **atualização** e **remoção** em profundidade.
