
Exercício 1 - $bucket - Pokémon por Ataque - FÁCIL - Igual Q9 mas em 3 linhas!
``` js
db.pokemons.aggregate([
  {
    $bucket: {
      groupBy: '$attack',
      boundaries: [0, 50, 80, 100, 200],
      default: 'Lendário',
      output: {
        total: {$sum:1},
        mediaHP: {$avg: '$hp'},
        nomes: {$push: '$name'}
      }
    }
  },
  {
    $set: {
      nomes: { $slice: ['$nomes', 3]  }
    }
  }
])
```

Exercício 2 - $bucket - Pokémon por HP - Outra faixa
``` js
db.pokemons.aggregate([
  {
    $bucket:{
      groupBy: '$hp',
      boundaries: [0, 50, 80, 255],
      default: 'HP Extremo',
      output: {
        total: {$sum: 1},
        mediaAtque: {$avg: '$attack'},
        maiorAtaque: {$max: '$attack'}
      }
    }
  }
])
```

Exercício 3 $bucketAuto - Clustering Automático - 3 Clusters!
Ex. com 3 buckets
``` js
db.pokemons.aggregate([
  {
    $bucketAuto:{
      groupBy: '$attack',
      buckets: 3,
      output:{
        total: {$sum: 1},
        mediaHP: {$avg: '$hp'},
        exemplos: {$push: '$name'}
      }
    }
  },
  {
    $set: {
      exemplos: {$slice: ['$exemplos', 5]}
    }
  }, 
])
```
Ex. com 5 buckets
``` js
db.pokemons.aggregate([
  {
    $bucketAuto:{
      groupBy: '$attack',
      buckets: 5,
      output:{
        total: {$sum: 1},
        mediaHP: {$avg: '$hp'},
        exemplos: {$push: '$name'}
      }
    }
  },
  {
    $set: {
      exemplos: {$slice: ['$exemplos', 5]}
    }
  }
  
])
```

Exercício 4 - Classificação com $switch - Alternativa limpa ao $cond da prova
``` js
db.pokemons.aggregate([
  {
    $addFields:{
      classe:{
        $switch:{
          branches: [
            {case: {$lt: ['$attack', 50]}, then: 'Fraco'},
            {case: {$lt: ['$attack', 80]}, then: 'Médio'},
            {case: {$lt: ['$attack', 100]}, then: 'Forte'},
          ],
          default: 'Lendário'
        }
      }
    }
  },
  {
    $group:{
      _id: '$classe',
      total: {$sum: 1},
      mediaHP: {$avg: '$hp'},
    }
  },
  {
    $sort: {total: -1}
  },
  {
    $project:{
      _id:0,
      classe: '$_id',
      total:1,
      mediaHP: {$round: ['$mediaHP', 2]}
    }
  },
])
```

Exercício 5 - Classificação com $switch aplicada após o bucket.
``` js
db.pokemons.aggregate([
  {
    $bucket: {
      groupBy: '$hp',
      boundaries: [0, 50, 80, 100],
      default: "Lendário",
      output: {
        total: {$sum:1},
        mediaHP: {$avg: '$hp'}
      }
    }
  },
  {
    $addFields:{
      classTotal: {
        $switch:{
          branches: [
            {case: {$gt:['$total', 500]}, then: 'Rapaiz, é muito pokemom'},
            {case: {$gt:['$total', 400]}, then: 'Onde cabe tudo isso?'},
            {case: {$gt:['$total', 300]}, then: 'Mas ai é pra poca'},
            {case: {$gt:['$total', 200]}, then: 'Aqui já fica mais tranquilo'},
            {case: {$gt:['$total', 100]}, then: 'Aqui já da uns times de futebol'},
          ],
          default: "Só isso?"
        }
      }
    }
  }
])
```
