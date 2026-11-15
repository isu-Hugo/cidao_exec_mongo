# Apply Schema Desigh Patterns
### 1. Padrão de heranças
%%Exemplo utilizado: Coleções de livros (físicos, e-books e áudio books)%%
Os dados são salvos juntos e são lidos juntos, e no mongo a funcionalidade em destaque é o polimorfismo de documentos semelhantes, como por exemplo os livros, possuem autor e titulo em comum, mas seus tipos `{product_type: 'audiobook'}`, são diferentes!
Os campos em comuns, são reaproveitados, mas de acordo com o campo de tipo, a aplicação pode trata-los de forma diferente.
#### Usando pipeline para aplicar a herança
Usando a agregação, podemos de forma eficiente aplicar a herança entre documentos em toda uma coleção, transformando os campos e padronizando caso tenham chaves diferentes ou inexistentes
```js
db.books.aggregate([
{
	$project:{
		_id: "_id",
		product_type: {
			$ifNull: ["$product", "Unspecified"]
		},
		description: {
			$ifNull: [
				"$desc",
				"$description",
				"$details",
				"Unspecified"
			]
		}		
	}
},
{
	$merge: {
		into: "books",
		on: "_id",
		whenMatched: "replace",
		whenNotMatched: "discart"
	}
}
])
```
Este foi um exemplo um pouco resumido, mas funciona da mesma forma, e após a padronização, podemos tratar os documentos com campos `Unspecified`
```js
[{
	$match: {
		$and: [{product_type: "Unspecified"}, {length_minutes: {$gt: 0} }]
	}
},
{
	$set: {product_type: "audiobook"}
},
{
	$merge: {
		into: "books",
		on: "_id",
		whenMatched: "replace",
		whenNotMatched: "discart"
	}
}]
```
Podemos aplicar dois demais pipelines adaptados para `e-books` e `printed books`, transformando a `collection` em um padrão consistente e com herança de campos entre documentos.

> [!NOTE] Nota
> Para ter certeza da finalização, podemos usar um `find({product_type: "Unspecified"})`, que não deve retornar documentos

### 2 Computed Pattern
Nesta aula, o foco são operações matemáticas e pre-computação antes de leituras, o exemplo citado foi, imagine calcular a média de estrelas de um livro a cada vez que um usuário carrega um livro, isso causaria lentidão e maior custo de processamento.
A solução é manter os dados já calculados, por exemplo: a avaliação de um livro pode ser calculada apenas quando existir uma nova, e este valor fica armazenado a um sub-documento deste livro.
Também citado o Roll-up, que é uma forma de visualizar pequenas categorias em grupos maiores, como relatórios por semanas, meses, anos ou categorias como aplicando aos livros
```js
[{
	$group: {
		_id: "$product_type",
		count: { $sum: 1 },
		mediaDeAutores: { $avg: { $size: "$autors" } }
	}	
}]
```
Listando os livros por tipo, contando a quantidade e média de autores, esse relatório pode ser gerado apenas uma vez por dia, e não a cada consulta, variando com as regras de negócios.

### 3 Extended Reference Pattern
As consultas que precisam relacionar varias coleções, também conhecidas como `JOINS` podem ser realizadas usando o `$lookup`, mas aplicar a cada consulta pode causar perda de performance, então entra a duplicidade de dados, onde podemos adicionar ao documento principal, uma informação que precise ser acessada constantemente.
O requisito era buscar reviews pelo titulo do livro, nota ou nome do usuário, para isso, seriam necessários cerca de 200 `$lookups` por segundo (requisito de exemplo), então, a solução foi adicionar os campos necessários a coleção de `review`
```js
{
product:{
	_id: "book-123",
	title: "return 0 in C"
},
reviwer:{
	_id: "456",
	username: "I like Linux BTW"
}
review:{
	rating: 5,
	date: "2026-11-16"
}
}
```

Este padrão de uso, permite aumentar a performance em consultas, mesmo causando a duplicidade de dados e, para sincronização normalmente podem ser executadas 'depois' e cada alteração não necessita uma atualização completa (regra de negócios).
A regra é priorizar apenas os campos realmente necessários e, preferencia por valores com baixa taxa de atualizações.

Exemplo oficial, realizando `$lookup` para agrupar as informações, `$unwind` para desmanchar os arrays, `$project` para estruturar os novos documentos e `$merge` para aplicar as alterações aos documentos de mesmo `_id`:
```js 
[
  {
    $lookup: {
      from: "books",
      localField: "product_id",
      foreignField: "product_id",
      as: "product_info",
    },
  },
  {
    $unwind: {
      path: "$product_info",
      includeArrayIndex: "string",
      preserveNullAndEmptyArrays: false,
    },
  },
  {
    $project: {
      _id: "$_id",
      "product.product_id": "$product_id",
      "product.product_type":
        "$product_info.product_type",
      "product.title": "$product_info.title",
      "review.user_id": "$user_id",
      "review.reviewTitle": "$reviewTitle",
      "review.reviewBody": "$reviewBody",
      "review.date": "$date",
      "review.stars": "$stars",
    },
  },
  {
    $merge: {
      into: "reviews",
      on: "_id",
      whenMatched: "replace",
      whenNotMatched: "discard",
    },
  },
]
```

# Identify Anti-patterns

### 1. Unbounded Arrays
%%Arrays sem limites%%
Um livro pode conter uma infinidade de reviews, o que causa um problema onde um `array` pode crescer indefinidamente, a implementação do mongo, permite apenas um documento expandir até `16MB`, então para casos de livros muito famosos, este problema pode surgir com facilidade.
Para resolver este problema, e ainda manter o conceito de que dados escritos juntos são lidos juntos, podemos analisar as regras de negócios. A quantidade de avaliações a mostra precisa ser fixamente três, então em nosso documento de livros, mantemos um `array` fixo de três itens, enquanto salvamos todas as reviews em sua própria coleção.

### 2. Bloated Documents
%%Documentos muito volumosos%%
O conceito de dados armazenados juntos são lidos juntos podem ainda se tornar um problema, no caso específico apresentado, a maquina possuía um total de `2GB RAM`, o que permitia ao cache interno do mongo, utilizar `256MB RAM`, porem a `collection` possuía um total de `500MB`.
Assim causando uma enorme perda de performance ao processo, a solução foi separar em uma outra coleção, os valores com menor frequência de consulta. A coleção que possui maior taxa de leitura ficou apenas com `35MB` de tamanho, assim sendo possível ser cacheada inteiramente.
O calculo de tamanho aproximado pode ser feito com
```js
db.coll.stats().count
db.coll.stats().avgObjSize
```

### Objetivos aprendidos
Aplicar padrões de design para schemas, definindo campos comuns, herança, padrão de dados já computados e entender as aplicações de cada uma.
Identificar quando os padrões estão sendo quebrados, como documentos 'inchados' e suas manutenções
