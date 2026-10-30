Usando o bucket
``` js
db.pokemons.aggregate([
  {
    $bucket: {
      groupBy: '$attack',
      boundaries: [0,50,80,100,200],
      default:'Lendário >200',
      output: {
        total: {$sum: 1},
        mediaAtaque: {$avg: '$attack'},
        maisForte: {$max: '$attack'}
        // nomes: {$push: '$name'}
      }
    }
  },
  {$project: {mediaAtaque: {$round: ['$mediaAtaque', 2]}, total:1, maisForte:1}}
])
```
o output é opcional, mas permite estruturar os dados da mesma forma ao usar um groupBy

mas também podemos deixar o mongo decidir as faixas usando o `autoBucket`, e definimos a quantidade de buckets que queremos, o mongo se encarrega por definir a faixa com quantidades semelhantes
``` js
db.pokemons.aggregate([
  {$bucketAuto:{
    groupBy: '$attack',
    buckets: 4,
    output: {total: {$sum:1} }
  }}
])
```


``` js
db.pokemons.aggregate([
  {
    $bucket: {
      groupBy: '$attack',
      boundaries: [0,50,80,100,200],
      default:'Lendário >200',
      output: {
        total: {$sum: 1},
        mediaAtaque: {$avg: '$attack'},
        maisForte: {$max: '$attack'},
      }}},

  {$addFields: {
    classe: {
          $switch: {
            branches: [
              {case: {$eq: ['$_id', 0]}, then: 'Fraco' },
              {case: {$lt: ['$_id', 80]}, then: 'Médio' },
              {case: {$lt: ['$_id', 100]}, then: 'Forte' },
            ],
            default: 'Lendário'
          }
        }
  }}
])
```