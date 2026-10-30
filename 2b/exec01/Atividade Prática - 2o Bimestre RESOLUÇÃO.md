Exercício 1 - $lookup - Enriquecendo dados (1,5 pontos) - FÁCIL
a)
``` js
db.campeoes.aggregate([
  {
    $lookup:{
      from: 'regioes_info',
      localField: 'regiao',
      foreignField: '_id',
      as: 'regiao_info'
    }
  },
  {$unwind: '$regiao_info'},
  {
    $project:{
      _id:0,
      nome: 1,
      regiao: 1,
      lider: '$regiao_info.lider',
      caracteristicas: '$regiao_info.caracteristica',
    }
  },
  {
    $sort: {regiao: -1}
  }
])
```
b)
``` js
db.pokemons.aggregate([
  {
    $lookup: {
      from: 'geracoes_info',
      localField: 'generation',
      foreignField: '_id',
      as: 'geracao'
    }
  },
  {$unwind: '$geracao'},
  {
    $project:{
      _id:0,
      name:1,
      generation:1,
      nomeGeracao: '$geracao.nome',
      ano: '$geracao.ano',
      lendario: '$geracao.lendario'
    }
  },
  {
    $limit: 10
  }
])
```

Exercício 2 - $switch - Classificação com Texto + Quantidade (1,5 pontos) - MÉDIO
a)
``` js
db.pokemons.aggregate([
  {
    $addFields:{
      classe: {
        $switch:{
          branches: [
            {case: {$lt:['$attack', 50]}, then: 'Fraco'},
            {case: {$lt:['$attack', 80]}, then: 'Médio'},
            {case: {$lt:['$attack', 100]}, then: 'Forte'},
          ],
          default: 'Lendário'
        }
      }
    }
  },
  {
    $group:{
      _id: '$classe',
      total: {$sum:1},
      mediaHP: {$avg: '$hp'}
    }
  },
  {
    $project:{
      _id:0,
      classe: '$_id',
      total: 1,
      mediaHP: {$round: ['$mediaHP', 1]}
    }
  }
])
```
b)
``` js
db.campeoes.aggregate([
  {
    $addFields:{
      classe: {
        $switch:{
          branches:[
            {case: {$lt:['$estatisticas.vida', 550]}, then: 'Frágil'},
            {case: {$lt:['$estatisticas.vida', 600]}, then: 'Médio'},
            {case: {$lt:['$estatisticas.vida', 650]}, then: 'Tanque'}
          ],
          default: 'Super Tanque'
        }
      }
    }
  },
  {
    $group:{
      _id: '$classe',
      total: {$sum:1},
      mediaDano: {$avg: '$estatisticas.dano_ataque'}
    }
  },
  {
    $project:{
      _id:0,
      classe: '$_id',
      mediaDano: {$round: ['$mediaDano', 1]},
      total:1
    }
  }
])
```

Exercício 3 - $bucket + $lookup - Histograma com Texto e Dica Extra (2,0 pontos) - AVANÇADO
a)
Aqui precisamos usar o resultado do bucket com os mesmos valores do _id do faixas_ataque
``` js
db.pokemons.aggregate([
  {
    $bucket:{
      groupBy: '$attack',
      boundaries: [0, 50, 80, 100, 200],
      default: '200',
      output: {
        total: {$sum:1},
        mediaHP: {$avg: '$hp'}
      }
    }
  },
  {
    $lookup:{
      from: 'faixas_ataque',
      localField: '_id',
      foreignField: '_id',
      as: 'faixa_ataque'
    }
  },
  {
    $unwind: '$faixa_ataque'
  },
  {
    $project:{
      _id:0,
      faixa_min: '$faixa_ataque.min',
      classe: '$faixa_ataque.classe',
      total:1,
      mediaHP: {$round: ['$mediaHP', 1]},
      dica: '$faixa_ataque.dica',
      cor: '$faixa_ataque.cor'
    }
  },
  {
    $sort: {faixa_min: -1}
  },
])
```
b)
``` js
db.campeoes.aggregate([
  {
    $bucket:{
      groupBy: '$estatisticas.vida',
      boundaries: [0, 550, 600, 650, 1000],
      default: '1000',
      output:{
        total: {$sum:1},
        nomes: {$push: '$nome'}
      }
    }
  },
  {
    $lookup:{
      from: 'faixa_vida',
      localField: '_id',
      foreignField: '_id',
      as: 'faixa'
    }
  },
  {
    $unwind: '$faixa'
  },
  {
    $project: {
      _id:0,
      total:1,
      classe: '$faixa.classe',
      estrategia: '$faixa.estrategia',
      nomes: {$slice: ['$nomes', 3] }
    }
  }
])
```

Exercício 4 - $bucketAuto - Clustering Automático (2,0 pontos) - AVANÇADO
a)
``` js
db.pokemons.aggregate([
  {
    $facet:{
      b3:[
        {
        $bucketAuto:{
          groupBy: '$hp',
          buckets: 3,
          output:{
            total: {$sum:1},
            mediaAtaque: {$avg: '$attack'}
          }
        }
      }
      ],
		b5: [
        {
        $bucketAuto:{
          groupBy: '$hp',
          buckets: 5,
          output:{
            total: {$sum:1},
            mediaAtaque: {$avg: '$attack'}
          }
        }
      }
    ] 
    }
  }
  // O uso de quantidades diferentes de buckets faz com o que as faixas mudem dinamicamente
])
```
b)
``` js
db.campeoes.aggregate([
  {
    $bucketAuto: {
      groupBy: '$estatisticas.vida',
      buckets: 3,
      output:{
        total: {$sum:1},
        mediaDano: {$avg: '$estatisticas.dano_ataque'}
      }
    }
  }
])

db.campeoes.aggregate([
  {
    $bucketAuto: {
      groupBy: '$estatisticas.vida',
      buckets: 4,
      output:{
        total: {$sum:1},
        mediaDano: {$avg: '$estatisticas.dano_ataque'}
      }
    }
  }
])
```
c)
``` text
	O bucket possui as faixas fixas, enquanto o bucketAuto possúi faixas variadas escolhidas automaticamente pelo mongo.
	O bucketAuto, com sua faixa variavel, faz com o que a quantidade fique equilibrada, o que depende dos valores e quantidades dos documentos.
	Para decidir quando usar cada, basta pensar no resultado... Eu preciso de valores '_id's exatos, ou quero um resultado distribuido?
```

Exercício 5 - $facet - Dashboard com Vários Relatórios em 1 Query (2,0 pontos) - DESAFIO
``` js
db.pokemons.aggregate([
  {
    $facet: {
      porAtaque:[
        {
          $bucket:{
            groupBy: '$attack',
            boundaries: [0, 50, 80, 100, 200],
            default: '> 200',
            output: {
              total: {$sum:1}
            }
          }
        },
        {
          $project:{
            _id:0,
            faixaAtaque: '$_id',
            total:1
          }
        }
      ],
      
      porHP:[
        {
          $bucket: {
            groupBy: '$hp',
            boundaries: [0, 50, 80, 255],
            default: '> 255',
            output: {
              total: {$sum:1}
            }
          }
        },
        {
          $project:{
            _id:0,
            faixaHP: '$_id',
            total:1
          }
        }
      ],
      
      porDefense:[
        {
          $bucket: {
            groupBy: '$defense',
            boundaries: [0, 50, 80, 200],
            default: '> 200',
            output: {
              total: {$sum:1}
            }
          }
        },
        {
          $project:{
            _id:0,
            faixaDefesa: '$_id',
            total:1
          }
        }
      ],
      
      mediaGeral:[
        {
          $group:{
            _id: null,
            HP: {$avg: '$hp'},
            ATK: {$avg: '$attack'},
            DEF: {$avg: '$defense'},
            total: {$sum:1}
          }
        },
        {
          $project: {
            _id:0,
            mediaHP: {$round: ['$HP', 2]},
            mediaAtaque: {$round: ['$ATK', 2]},
            mediaDefesa: {$round: ['$DEF', 2]},
            total: 1
          }
        }
      ]

      
    }
  }
])
```

Exercício 6 - DESAFIO FINAL BÔNUS - Tudo Junto: $facet + $bucket + $bucketAuto + $lookup + $switch (1,0 ponto bônus) - NINJA
``` js
db.pokemons.aggregate([
  {
    $facet:{
      porAtaqueManual: [
        {
          $bucket: {
            groupBy: '$attack',
            boundaries: [0, 50, 80, 100, 200],
            default: '>200',
            output: {
              total: {$sum:1}
            }
          }
        },
        {
          $lookup:{
            from: 'faixas_ataque',
            localField: '_id',
            foreignField: '_id',
            as: 'faixaAtaque'
          }
        },
        {$unwind: '$faixaAtaque'},
        {
          $project:{
            _id: 0,
            faixa: '$_id',
            total:1,
            classe: '$faixaAtaque.classe',
            dica: '$faixaAtaque.dica',
          }
        }
      
      ],
      porAtaqueAuto: [
        {
          $bucketAuto:{
            groupBy: '$attack',
            buckets: 4,
            output: {
              total: {$sum:1}
            }
          }
        },
        {
          $project: {
            _id:0,
            faixa_inicio: '$_id.min',
            faixa_fim: '$_id.max',
            total:1
          }
        }
      ],
      porHP: [
        {
          $bucket:{
            groupBy: '$hp',
            boundaries: [0, 50, 80, 255],
            output:{
              total: {$sum:1}
            }
          }
        },
        {
          $project:{
            _id:0,
            faixa: '$_id',
            total:1
          }
        }
      ],
      classificacaoSwitch: [
        {
          $addFields:{
            classe: {
              $switch:{
                branches: [
                  {case: {$lt:['$attack', 50]}, then: 'Fraco'},
                  {case: {$lt:['$attack', 80]}, then: 'Médio'},
                  {case: {$lt:['$attack', 100]}, then: 'Forte'}
                ],
                default: 'Lendário'
              }
            }
          }
        },
        {
          $group: {
            _id: '$classe',
            total: {$sum:1}
          }
        },
        {$sort: {total:-1}},
        {
          $project:{
            _id:0,
            classe: '$_id',
            total:1
          }
        }
      ]
    }
  }
])
```