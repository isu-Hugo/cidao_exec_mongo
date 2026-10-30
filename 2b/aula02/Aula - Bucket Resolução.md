
Ex1 - $bucket por ataque [0,50,80,100,200] 
``` js
db.pokemons.aggregate([
  {$bucket: {
    groupBy: '$attack',
    boundaries: [0, 50, 80, 100, 200],
    default: 'HP lendário',
    output: {
      total: {$sum:1}
    }
  }}
])
#
{
  _id: 0,
  total: 134
} ...
```

Ex2 - $bucket por HP [0,50,80,255] 
``` js
db.pokemons.aggregate([
  {$bucket: {
    groupBy: '$hp',
    boundaries: [0, 50, 80, 255],
    default: 'Fora da faixa',
    output: {
      total: {$sum: 1}
    }
  }}
])
#
{
  _id: 0,
  total: 189
} ...
```

Ex3 - $bucketAuto 3 clusters por attack 
``` js
db.pokemons.aggregate([
  {$bucketAuto: {
    groupBy: '$attack',
    buckets: 3,
    output: {
      total: {$sum:1}
    }
  }}
])
#
{
  _id: {
    min: 20,
    max: 81
  },
  total: 270
} ...
```

Ex4 - $switch classificando Fraco/Médio/Forte/Lendário + $group 
``` js
db.pokemons.aggregate([
  {$addFields: {
    classificacao: {
      $switch: {
        branches: [
          {case: {$lt:['$attack', 50]}, then: 'Fraco' },
          {case: {$lt:['$attack', 80]}, then:'Médio'},
          {case: {$lt:['$attack', 100]}, then: 'Forte'},
          {case: {$lt:['$attack', 200]}, then:'Lendário'}
        ],
        default: 'Infinito'
      }
    }
  }},

  {$group:{
    _id: '$classificacao', total: {$sum:1}
  }}
])
#
{
  _id: 'Forte',
  total: 114
} ...
```

Ex5 - LoL - $bucket por vida [0,550,600,650,1000] + $bucketAuto por dano
``` js
db.campeoes.aggregate([
  {$facet:{
    porVida:[
      {
        $bucket:{
          groupBy: '$estatisticas.vida',
          boundaries: [0, 550, 600, 650, 1000],
          default: "Maior que 1.000",
          output: {
            total: {$sum: 1}}}
      }
    ],
    porDano:[
      {
        $bucketAuto: {
          groupBy: '$estatisticas.dano_ataque',
          buckets: 3,
          output: {
            total: {$sum:1}
          }}}
    ],
  }}
])
#
porVida: [
    {
      _id: 0,
      total: 17
    }...
  ],
porDano: [
    {
      _id: {
        min: 51,
        max: 57
      },
      total: 14
    }...
]
```
