#### INSERÇÃO - insertOne e insertMany
1. Insira um único funcionário chamado 'Rafael Souza', 26 anos, Desenvolvedor Mobile, departamento Tecnologia, salario 8200, ativo true, com habilidade ['Flutter']. Use insertOne
```
db.funcionarios.insertOne({
  nome: 'Rafael Souza',
  idade: 26,
  cargo: 'Desenvolvedor Mobile',
  departamento: 'Tecnologia',
  salario: 8200,
  ativo: true,
  habilidades: ['Flutter']
})
```

2. Insira 2 novos funcionários de uma vez só no departamento 'Marketing', ambos inativos, contratados hoje. Invente os dados. Use insertMany.
```
db.funcionarios.insertMany([
  {nome: 'Hugo', idade: 20, departamento: 'Marketing', ativo: false, salario: 7000, habilidades: ['Linux']},
  {nome: 'Ana Julia', idade: 18, departamento: 'Marketing', ativo: false, salario: 7000, habilidades: ['Aplicar Vacina']}
])
```

3. Insira um funcionário chamado 'Teste Upsert' sem campo contato. Ele será usado no exercício de upsert.
```
db.funcionarios.insertOne({
  nome: 'Teste Upsert',
  departamento: 'teste', ativo: true, salario: 5000, habilidades: ['Teste']
})
```

#### LEITURA - find() com Operadores Lógicos e de Array

4. Liste todos os funcionários do departamento 'Tecnologia'.
```
db.funcionarios.find({
    departamento: 'Tecnologia'
})
```

5. AND Implícito Liste quem é da Tecnologia E ganha mais de 8000.
```
db.funcionarios.find({
  departamento: 'Tecnologia',
  salario: {$gt: 8000}
})
```

6. AND Explícito com $and Use $and para listar quem tem idade entre 30 e 40 anos e está ativo.
```
db.funcionarios.find({
  $and: [
    {idade: {$gte: 30, $lte: 40}},
    {ativo: true}
  ]
})
```

7. OR Liste quem é do departamento 'RH' OU 'BI'. Use $or.
```
db.funcionarios.find({
  $or: [
    {departamento: 'RH'},
    {departamento: 'BI'}
  ]
})
```

8.  IN Liste quem tem cargo dentro desta lista: ['Analista de Dados', 'Analista de RH', 'Gerente de Projetos']. Use $in
```
db.funcionarios.find({
  cargo: {$in: ['Analista de Dados', 'Analista de RH', 'Gerente de Projetos']}
})
```

9. NIN Liste quem NÃO está no departamento Tecnologia. Use $nin.
```
db.funcionarios.find({
  departamento: {$nin: ['Tecnologia']}
})
```

10. EXISTS Liste quem NÃO tem telefone cadastrado dentro de contato.telefone. Use $exists: false. 
```
db.funcionarios.find({
'contato.telefone': {$exists: false}  
})
```

11. Busca exata em Array Liste quem tem a habilidade 'React' (busca simples no array). 
```
db.funcionarios.find({
  habilidades: {$eq: 'React'}
})
# OU
db.funcionarios.find({
  habilidades: {$in: ['React']}
})
```

12. Busca com $all Liste quem sabe 'React' E 'Node.js' ao mesmo tempo.
```
db.funcionarios.find({
  habilidades: {$all: ['React', 'Node.js']}
})
```

13. Busca com $size Liste funcionários que possuem exatamente 4 habilidades.
```
db.funcionarios.find({
  habilidades: {$size: 4}
})
```

14. Busca com $elemMatch + Regex Liste quem tem alguma habilidade que começa com 'Java'. Deve trazer Java e JavaScript.
```
db.funcionarios.find({
  habilidades: {$elemMatch: {$regex: /^Java/} }
})
```

#### MANIPULAÇÃO DE ARRAYS E SUBDOCUMENTOS

15.  PUSH simples Adicione a habilidade 'TypeScript' para a Ana Silva. Use updateOne + $push.
```
db.funcionarios.updateOne(
  {nome: 'Ana Silva'},
  {
    $push: {habilidades: 'TypeScript'}
  }
)
```

16. PUSH com $each + $position + $sort Para o André Marques, adicione de uma vez só ['GraphQL', 'Redis'] na POSIÇÃO 0 do array de habilidades, e ordene o array em ordem alfabética.
Ao usar o $sort, -1 é decrescente e 1 é crescente
```
db.funcionarios.updateOne(
  {nome: 'André Marques'},
  {
    $push: {
      habilidades:{
        $each: ['GraphQL', 'Redis'],
        $position: 0,
        $sort: 1
      }
    }
  }
)
```

17. ADDTOSET Tente adicionar 'React' novamente para a Ana Silva, mas sem duplicar. Use $addToSet.
```
db.funcionarios.updateOne(
  {nome: 'Ana Silva'},
  {
    $addToSet: {habilidades: 'React'}
  }
)
```

18.  PULL Remova a habilidade 'Legislação' da Eduarda Lima. Use $pull
```
db.funcionarios.updateOne(
  {nome: 'Eduarda Lima'},
  {
    $pull: {habilidades: 'Legislação' }
  }
)
```

19. PULL com condição em Many Remova de TODOS os funcionários da Tecnologia a habilidade 'AWS'. Use updateMany + $pull.
Se usar o updateOne, não passa por todos da tecnologia
```
db.funcionarios.updateMany(
  {
    departamento: 'Tecnologia',
  },
  {
    $pull: {habilidades: 'AWS'}
  }
)
```

20. POP Remova a ÚLTIMA habilidade do Bruno Costa com $pop: 1. Depois remova a PRIMEIRA com $pop: -1
```
db.funcionarios.updateOne(
  {nome: 'Bruno Costa'},
  {
    $pop: {habilidades: 1}
  }
)
# E depois
db.funcionarios.updateOne(
  {nome: 'Bruno Costa'},
  {
    $pop: {habilidades: -1}
  }
)
```

21.  Subdocumento - Dot Notation Adicione o telefone '11-99999-0000' para o Bruno Costa, que hoje só tem e-mail. Use $set em 'contato.telefone'.
```
db.funcionarios.updateOne(
  {nome: 'Bruno Costa'},
  {
    $set: {'contato.telefone': '11-99999-0000'}
  }
)
```

22. Operador posicional $ Substitua 'Spring' por 'Spring Boot' no array do Bruno Costa. Filtro: {nome: 'Bruno Costa', habilidades: 'Spring'} e update: {'habilidades.$': 'Spring Boot'}.
O carácter `$` é atribuído ao índice do 'Spring' na lista, por conta que estava no filtro de busca. Ele é automaticamente atribuído ao índice no qual o item foi encontrado no array pelo filtro
```
db.funcionarios.updateOne(
	{nome: 'Bruno Costa', habilidades: 'Spring'},
  {
    $set: {'habilidades.$': 'Spring Boot'}
  }
)
```

23. arrayFilters - DESAFIO Substitua 'PowerBI' por 'Tableau' no array do Daniel Farias usando set com 'habilidades.$[elem]' e arrayFilters.
```
db.funcionarios.updateOne(
  {nome: 'Daniel Farias', habilidades: 'PowerBI'},
  {
    $set: {
      'habilidades.$': 'Tableau'
    }
  }
)
```

#### ATUALIZAÇÃO COM OPTIONS

24. updateMany + $mul Dê um aumento de 10% para TODO o departamento de Tecnologia. Use $mul: { salario: 1.10 }
```
db.funcionarios.updateMany(
  {departamento: 'Tecnologia'},
  {
    $mul: {salario: 1.10}
  }
)
```

25. updateOne + upsert: true Atualize 'Teste Upsert' setando ativo: true. Se ele não existir, deve criar. Use option { upsert: true }.  
```
db.funcionarios.updateOne(
  {nome: 'Teste Upsert'},
  {$set: {ativo: true} },
  {upsert: true}
)
```

26. updateMany + $set + $currentDate Desative todos contratados antes de 01/01/2019, setando ativo: false e criando ultima_atualizacao com $currentDate. 
Não tenho certeza se está certo
```
db.funcionarios.updateMany(
  {data_contratacao: {$lt: new Date('01/01/2019')} },
  {
    $set: {ativo: false},
    $currentDate: {ultima_atualizacao: true}
  }
)
```


27. updateOne + $rename Renomeie o campo cargo para funcao APENAS da Carla Dias.
Não  tenho certeza se está correto pois o campo não existia
```
db.funcionarios.updateOne(
  {nome: 'Carla Dias'},
  {
    $rename: {cargo: 'funcao'}
  }
)
```

28. updateOne + $unset Remova completamente o campo telefone do contato da Ana Silva.
```
db.funcionarios.updateOne(
  {nome: 'Ana Silva'},
  {
    $unset: {'contato.telefone': ''}
  }
)
```

#### EXCLUSÃO - deleteOne e deleteMany

29. deleteOne Apague o funcionário 'Teste Upsert'. 
```
db.funcionarios.deleteOne(
  {nome: 'Teste Upsert'}
)
```

30. deleteMany Apague TODOS os funcionários com ativo: false.
```
db.funcionarios.deleteMany(
  {ativo: false}
)
```

#### DESAFIO FINAL - Juntando tudo.

31. Crie uma consulta que retorne 
nome, cargo e salario 
APENAS dos ativos da Tecnologia que ganham > 8000 e 
possuem 'React' OU 'MongoDB'. 
Use projeção {nome:1, cargo:1, salario:1, _id:0} e .sort({salario: -1}). 

```
db.funcionarios.find(
  {
    $and: [
    {ativo: true}, 
    {departamento: 'Tecnologia'}, 
    {salario: {$gt: 8000}}, 
    {habilidades: {$in: ['React', 'MongoDB'] } }],
  },
  {nome:1, cargo:1, salario:1, _id:0}
).sort({salario: -1})
```

32. Faça um updateMany que para quem tem 'Docker' nas habilidades, adicione também 'Kubernetes' se ainda não tiver (use $addToSet) e aumente o salário em RS 500 (use $inc).
```
db.funcionarios.updateMany(
  {habilidades: 'Docker'},
  {
    $addToSet: {habilidades: 'Kubernets'},
    $inc: {salario: 500}
  }
)
```
