Anotações dos módulos presentes no curso

## Perform CRUD operations
---

#### 1. Inserindo valores em uma coleção

Para inserir, usamos `insertOne({chave: 'valor'})` ou vários com 
`insertMany([ {...}, {...} ])`.
`db.<collection>.insertOne({...})`
O mongo cria a `collection` caso ela não exista, e cada documento tem seu _id gerado automaticamente pelo Mongo.

#### 2. Encontrando documentos

Para encontrar os documentos usamos `db.collection.find()`. Isso retorna UM CURSOR para todos os documentos presentes na coleção,
o que permite usar '> it' para continuar carregando os resultados encontrados.
Para buscas reais, usamos parâmetros na chamada de função, como:
 `find( {campo: 'valor'} )` que se comporta igual a `find( {campo: {$eq: 'valor'} } )`, esta função retorna os documentos em que os valores são exatos, 
mas para fazer uma consulta com mais valores em um mesmo campo usamos
`find( {campo: {$in: ['valor1', 'valor2'] } } )`, retorna todos os documentos em que `campo` seja igual a 'valor1' OU 'valor2'
Para retornos de apenas um documento, podemos usar `findOne()`

#### 3. Atualizando documentos
Para atualizar documentos no mongodb, usamos `updadeOne()` ou `updateMany()`;
Ambas funções esperam 2 parâmetros com um opcional
{ campo: valor } -> O campo do find, para escolhermos o documento que vamos modificar
{ $set: {campo: valor} } -> Quando encontrarmos, o que vamos fazer com o documento
{ upsert: true } -> Campo opcional para configuração

#### 4. Foco no updateMany
O updateMany é usado para atualizar múltiplos documentos em uma coleção, seguindo os parâmetros de 
{ filtro },
{ modificação },
{ opções }.
Ele não é aconselhado para uso corporativo, pois não possui all-or-nothing, então caso falhe os documentos podem ficar atualizados pela metade,
já que ele não faz o rollback caso não executado novamente, assim seria necessário rodar novamente para a conclusão da atualização.

#### 5. Excluindo documentos
Para excluir documentos, usamos `deleteMany()`, ou `deleteOne()`.
No exemplo, precisamos remover um documento duplicado, então usamos o `deleteOne()` com o `_id` do objeto,
mas para deletar todos os documentos que possuem um mesmo valor, usamos o `deleteMany()`,
O próprio curso indica o uso de um `find()`, com a mesma query que será usada no `deleteMany()`, para que antes de 
deletar todos, os dados sejam consultados e conferidos. (se tem placa, tem história), apagar valores as cegas nunca sera recomendado.
No curso é citado sobre o objeto de opções, mas não é mostrado. Nele possuem funções de log de mensagem para rastreio, otimizações
definição de idioma para buscas por strings, também opções para ecossistemas distribuídos, onde outros nós precisam 'concordas' com a exclusão.

## Desenvolvendo buscas eficientes

#### 1. Operadores relacionais 
Podemos usar queries com operadores 
`$gt, $gte, $lt, $lte, $eq, $ne, $in: [], $nin: []`.
Agora com documentos mais complexos precisamos usar `"dot.notation"`. Assim podemos acessar campos em subdocumentos.
`{ 'campo.sub': {$gte: 10} }`

#### 2. Uso de and e or
Operadores `$and` e `$or` em queries, o find possui um `$and` implícito por padrão, mas como mencionado no curso, precisamos tomar cuidado com 
a sobrescrita de campos num JSON.
O operador `$and:[ {...}, {...} ]` espera um array de objetos para filtro de busca, os valores retornados são os que cumprem todos os requisitos.
O operador `$or: [ {...}, {...} ]` também espera um array, mas retorna documentos que pelo menos uma regra seja cumprida.
Estes operadores permitem consultas muito bem estruturadas, onde podem ser adicionados uns dentro dos outros como
``` js
$and: [
{$or [ {...}, {...} ] },
{$or [ {...}, {...} ] }
].
```
Neste caso em específico, o uso do and implícito pelo find causaria problemas, já que o segundo `$or`, poderia sobrescrever a chave de consulta do anterior!

#### 3. Trabalhando com arrays em documentos
Para procurar um documento que tenha o valor dentro de um array, ainda podemos usar o find, mas o uso pode ser limitado em algumas situações. Em casos comuns
podemos usar `find( {campo: 'valor'} )` e todos os documentos que possuem 
`campo: 'valor' ou campo: [ 'valor', 'outro valor'  ]` serão listados.

Para trabalharmos com uma consulta em arrays apenas, usamos o `elemMatch`
``` js
db.collection.find({
	$elemMatch: {
		$eq: "valor"
	}
})
```

Mas o uso do `elemMatch` é necessário quanto precisamos encontrar um sub-documento que está em um `array`,  como por ex.:
```js
// BASE DE EXEMPLO
{
_id: 'uuid',
lista: [
	{
		nome: 'string',
		quantidade: 3,
		preco: 10
	}
]
}
```
Neste caso, um simples find não seria capaz de encontrar o documento certo, já que o filtro seria aplicado a todos os itens do `array` de uma vez, o correto seria:
```js
db.collection.find({
	$elemMatch: {
		nome: 'string',
		preco: {$gt: 5},
		quantidade: {$lte: 3}
	}
})
```
Desta forma, procuramos pelo sub-documento que único cumpra todos os requisitos da busca.

## Modificando resultados de queryes

#### 1. Sorteamento e limitando
Os métodos de `find`, retornam cursores, objetos que funcionam como ponteiros para os dados, permitindo enviar resultados em lotes e manipulação das informações sem carregar todos os valores na memória instantaneamente

Usando o cursor, podemos sortear os dados em `ascendente` ou `descendente` escolhendo o campo para ordenação, mas não se limitando a apenas um único, podemos ordenar por valor e quando forem iguais decidir na data de criação por exêmplo.
```js
asc = 1;
desc = -1;
.find({...}).sort({campo: asc})
```

E com o método find, podemos adicionar a projeção, escolher quais valores queremos apresentar na consulta, para isso usamos o segundo parâmetro:
```js
.find({...}, {campo:1, _id:0} )
```

Mas para fins de domínio e otimização, podemos definir um limite máximo de documentos retornados 
```js
.find({...}).limit(3)
```

#### 2. Especificando campos para retorno usando projeção
Ao usar `find`, podemos usar dois parâmetros, o primeiro É o filtro de busca, e o segundo (opcional), é a projeção, os valores que serão apresentados ao resultado da consulta
```js
.find(
{...}, // Obj de consulta
{campo:1, _id:0} // Obj de projeção
)
```
Uma regra importante é, podemos usar a inclusão, ou a exclusão em uma projeção, mas não os dois simultaneamente, com uma exceção do `_id`, que como no exemplo acima precisa explicitamente da exclusão.

Para usar um `find` com múltiplos valores em um mesmo campo, podemos usar o `campo: {$in: ['v1', v2]}`. Então se o `campo` conter um valor em `[...]` é o que eu procuro
```js
.find({campo: {$in: ['v1','v2']} }, {campo:1})
```
%%Para múltiplos valores em um campo, usamos $in, para varias condições em campos diferentes usamos $or%%

#### 3. Contando documentos
Para contar documentos em uma coleção, podemos usar
```js
db.collection.countDocuments(<query>, <options>)
```

Semelhante ao `find`, nossa `query` será o filtro de busca e, todos os documentos encontrados serão contados e retornados como resposta da função.
O segundo parâmetro de `<options>` é usado para otimizações como

| Comando   | Ação                                            |
| --------- | ----------------------------------------------- |
| limit     | Define a contagem máxima                        |
| skip      | Ignora os primeiros `n` documentos              |
| hint      | Dica de performance                             |
| maxTimeMS | Define um tempo máximo para execução do comando |
| collation | Regras de idioma para buscas (ignorar acentos)  |

![[Pasted image 20261109205519.png]]