- find() Só filtra e retorna
- aggregate() Transforma dados em informação
- pipeline = esteira: saída de estagio = entrada de um próximo
- cada estágio é um objeto {$match {...} }


```
db.funcionarios.aggregate([
	{ $match: {ativo: true} } // O filtro, where
	{ $group: {_id: {departamento: "$departamento", cargo: "$cargo"} } } // Agrupa por um id, este ID é para controle do $group
	{ $count: "qtd"} // Apresenta a quantidade de documentos
	{ $group:{_id: "$departamento", total:{$sum:1} } }
	{ $sort: -1 ou 1} // em crescente ou decrescente, 
	{ $sort: departamento:1, salario:-1}
])
```

```
db.funcionarios.aggregate([
  {$group: {_id:"$departamento", total:{$sum:1}, folha:{$sum:'$salario'} }} // Quando usamos '$departamento' é para definirmos que é uma coluna e não um apelido
])
```
