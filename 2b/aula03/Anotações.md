
Usando switch para traduzir o mes
``` js
db.visualizacoes.aggregate([
  {
    $addFields: {
      mes: {$month: '$dataHora'}
    }
  },
  {$addFields: {
    mes: {
      $switch:{
        branches: [
          {case: {$eq: ['$mes', 1]}, then: 'Janeiro'},
          {case: {$eq: ['$mes', 2]}, then: 'Fevereiro'},
          {case: {$eq: ['$mes', 3]}, then: 'Marco'},
          {case: {$eq: ['$mes', 4]}, then: 'Abril'},
          {case: {$eq: ['$mes', 5]}, then: 'Maio'},
          {case: {$eq: ['$mes', 6]}, then: 'Junho'},
          {case: {$eq: ['$mes', 7]}, then: 'Julho'},
          {case: {$eq: ['$mes', 8]}, then: 'Agosto'},
          {case: {$eq: ['$mes', 9]}, then: 'Setembro'},
          {case: {$eq: ['$mes', 10]}, then: 'Outubro'},
          {case: {$eq: ['$mes', 11]}, then: 'Novembro'},
          {case: {$eq: ['$mes', 12]}, then: 'Dezembro'},
        ],
        default: 'Inválido'
      }
    }
  }},
  {$limit: 4}
])
```
Porem esse é o jeito 'carrasco' de se usar, fazendo cada tentativa na força bruta, mas como é sequencial, podemos usar um array!


``` js
db.visualizacoes.aggregate([
  {
    $group: {
      _id: {$month: '$dataHora'},
      minutos: {$sum: '$minutosAssistidos'}
    }
  },
  {$sort: {_id:1}},
  {
    $project: {
      _id:0,
      mes: {
        $arrayElemAt: [
          ["janeiro", "fevereiro", "março", "abril", "maio", "junho", "julho", 
           "agosto", "setembro", "outubro", "novembro", "dezembro"],
          {
            $subtract: ['$_id', 1]
          }
        ]
      },
      minutos:1
    }
  },
  {
    $sort: {minutos:-1}
  }
])
```

Para agrupar e contando em ordem decrescente, podemos usar o `$sortByCount`
``` js
db.titulos.aggregate([
  {$sortByCount: '$tipo'}
])
```
para representa-lo em um group normal seria necessário
``` js
db.titulos.aggregate([
  {
    $group:{
      _id: '$tipo',
      count: {$sum: 1}
    }
  },
  {$sort: {count:-1}}
])
```
Ambos com o mesmo resultado
``` js
{
  _id: 'serie',
  count: 5
},
{
  _id: 'filme',
  count: 7
}
```

Exemplo da aula
``` js
db.titulos.aggregate([
  {
    $lookup: {
      from: 'visualizacoes',
      localField: '_id',
      foreignField: 'tituloId',
      as: 'visualizacoes'
    }
  },
  {$unwind: '$visualizacoes'},
  {
    $sortByCount: '$visualizacoes.dispositivo'
  },
  {
    $project: {
      _id:0,
      dispositivo: '$_id',
      quantidade: '$count'
    }
  }
])
```

Podemos criar uma outra coleção com base no resultado do pipeline usando o `$merge`
``` js
db.visualizacoes.aggregate([
  {
    $group: {
      _id: '$dispositivo',
      minutos: { $sum: '$minutosAssistidos'}
    }
  },
  {
    $merge: {
      into: 'relatorio_dispositivo',
      on: '_id',
      whenMatched: 'replace',
      whenNotMatched: 'insert'
    }
  }
])
```
Após isso, outra coleção vai aparecer no db.