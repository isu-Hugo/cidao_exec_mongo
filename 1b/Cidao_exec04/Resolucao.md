Aula sobre $cond
create, collmod, insert update, delete, aggregate

```
db.restaurants.aggregate([
  {$unwind: '$grades'},
  {$project:{
    borough:1,
    score: '$grades.score',
    
    status: {
      $cond: {
        if: {$lt: ['$grades.score', 10] },
        then: 'Excelente',
        else: 'Ruim'
      }
    }
    
  }},
  {$limit:5},
])
```

```
db.restaurants.aggregate([
  {$unwind: '$grades'},
  {$project: {
    _id:0,
    name:1,
    grade: '$grades.grade',
    isA: {
      $cond: {
        if: {$eq: ['$grades.grade', 'A'] },
        then: true,
        else: false
      }
    }
  }},
  {$limit:5}
])
```

```
db.restaurants.aggregate([
  {$unwind: '$grades'},
  {$project:{
    _id:0,
    score: '$grades.score',
    classificacao:{
      $cond:{
        if: {$lt:['$grades.score', 5] },
        then: 'Excelente',
        else: {
          $cond: {
            if: {$lt: ['$grades.score', 15] },
            then: 'Bom',
            else: 'Ruim'
          }
        }
      }
    }
  }}
])
```

```
db.restaurants.aggregate([
  {$unwind: '$grades'},
  {$project:{
    _id:0,
    score: '$grades.score',
    classificacao: {
      $cond: {
        if: {$lt: ['$grades.score', 5] },
        then: 'Excelente',
        else: {
          $cond: {
            if: {$lt: ['$grades.score', 15]},
            then: 'Bom',
            else: {
              $cond: {
                if: {$lt:['$grades.score', 30] },
              	then: 'Regular',
              	else: 'Crítico'
              }
            }
          }
        }
      }
    }
  }}
])
```

```
db.restaurants.aggregate([
  {$unwind: '$grades'},
  {$group: {_id: '$restaurant_id',
            nome: {$first: '$name'},
            total: {$sum: 1},
            totalA: {$sum: {
              $cond: [{$eq:['$grades.grade', 'A']}, 1, 0 ]
            }}
           }
  },
  {$sort: {totalA:-1}},
  {$limit:5}
])
```

