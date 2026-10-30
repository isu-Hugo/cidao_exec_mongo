from: de onde eu quero fazer a ligação
localField: O campo local que será usado para a comparação, o campo que está dentro da coleção que estou usando
foreignField: O campo que vai ser referenciado na tabela de destino
as: Apelido

``` 
db.pedidos.aggregate([
  {
    $lookup: {
      from: 'clientes',
      localField: 'clienteId',
      foreignField: '_id',
      as: 'cliente'
    }
  }
])
#
{
  _id: 101,
  clienteId: 1,
  valor: 250,
  status: 'pago',
  data: 2026-09-10T00:00:00.000Z,
  cliente: [
    {
      _id: 1,
      nome: 'Ana',
      cidade: 'São Paulo'
    }
  ]
}
```

O lookup faz a comparação  e adiciona como array os itens encontrados, para trabalharmos com isso usamos o $unwind
```
db.pedidos.aggregate([
  {
    $lookup: {
      from: 'clientes',
      localField: 'clienteId',
      foreignField: '_id',
      as: 'cliente'
    }
  },
  {$unwind: '$cliente'},
  {$project: {
    _id: 1,
    valor:1,
    status:1,
    cliente: '$cliente.nome',
    cidade: '$cliente.cidade'
  }}
])
#
{
  _id: 101,
  valor: 250,
  status: 'pago',
  cliente: 'Ana',
  cidade: 'São Paulo'
}
```

Para várias condições podemos usar um switch
```
db.pedidos.aggregate([
  {
    $addFields: {
      faixaValor: {
        $switch: {
          branches: [
            {case: {$lt: ['$valor', 200]}, then: 'Baixo' },
            {case: {$lt: ['$valor', 1000]}, then: 'Medio' }
          ],
          default: 'Alto'
        }
      }
    }
  }
])
```

Ainda fazendo:
```
db.pedidos.aggregate([
  {$lookup: {
    from: 'clientes',
    localField: 'clienteId',
    foreignField: '_id',
    as: 'cliente'
  }},
  
  {$unwind: '$cliente'},
  {$group: {
    _id: '$cliente._id', 
    totalGasto: {$sum: '$valor'},
    quantidadePedidos: {$sum:1},
    nome: {$first: '$cliente.nome'}
  }},

  {$addFields: {
    categoria: {
      $switch: {
        branches: [
          {case: {$lt: ['$totalGasto', 500]},
          then: 'Bronze'},
          {case: {$lt: ['$totalGasto', 1500] },
          then: 'Prata'}
        ],
        default: 'Ouro'
      }
    }
  }},

  {$facet: {
    totalCidade: [
      
    ],
    
  }}
])
```

#### Resolução da atividade pratica
``` js
db.pedidos.aggregate([
  //{$match: {status: "pago"}},
  //Relacionar as duas coleções
  {
   $lookup:{
     from: "clientes",
     localField: "clienteId",
     foreignField: "_id",
     as: "cliente"
   } 
  },
  
  //Desmontar os arrays
  {$unwind: "$cliente"},

  //Agrupar os clientes e calcular
  {
    $group: {
      _id: '$clienteId',
      totalGasto: { $sum: '$valor' },
      quantidadePedidos: { $sum: 1  },
      nome: {$first: '$cliente.nome'},
      cidade: {$first: '$cliente.cidade'}
    }
  },

  //Acidionar classificação para o cliente (Bronze, Prata, Ouro)
  {
    $addFields: {
      classificacao: {
        $switch :{
          branches: [
            {case: {$lt: ['$totalGasto', 500]}, then: "Bronze"},
            {case: {$lt: ['$totalGasto', 1500]}, then: "Prata"}
          ],
          default: "Ouro"
        }
      }
    }
  },

  //Usar 3 sub-queryes para informações diferentes
  {
    $facet: {
      totalPorCidade: [
        {
          $group:{
            _id: '$cidade', 
            total: {$sum:1}
          }
        }
      ],
      quantPorCategoria: [
        {
          $group:{
            _id: '$classificacao', 
            total:{$sum:1} 
          }
        }
      ],
      //Apenas projeta o valor anterior para os detalhes
      detalhes: [
        {
          $project: {
            _id:0,
            totalGasto:1,
            quantidadePedidos: 1,
  			nome: 1,
  			cidade: 1,
  			classificacao: 1
          }
        }
      ]
    }
  }
])
```

E para completar o desafio adicional, basta adicionar o pipeline
``` js
  {$match: {status: "pago"}}
```
Ao inicio do comando.