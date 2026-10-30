Iniciando o mongo com os dados estáticos 
```
use faculdade
db.alunos.drop()
db.alunos.insertMany([
	{ nome: "Ana", idade: NumberInt(21), curso: "Sistemas de Informação" },
	{ nome: "Bruno", idade: NumberInt(22) },
	{ nome: "Carla", idade: "20", curso: "Administração" },
	{ nome: "Diego", idade: NumberInt(19), curso: "Sistemas de Informação" }
])
```

#### Conhecendo os documentos com find()
1. Exiba todos os documentos da coleção alunos.
	`db.alunos.find()`
	O primeiro parâmetro é a condição de busca `where`, quando vazio aplica a todos.

2.  Exiba somente nome, idade e curso, sem mostrar _id.
	`db.alunos.find({}, {_id : 0})`
	Mantendo o `where` vazio e aplicando o filtro para esconder o campo `_id` 
	
3. Localize os alunos que não possuem o campo curso.
	`db.alunos.find({curso: {$exists: false} })`
	O parâmetro `where` também possui propriedades especiais, usadas com `$`.

4. Localize os alunos cuja idade está armazenada como int.
	`db.alunos.find({idade: {$type: "int"} })`
	Podemos filtrar pelo tipo de dados
	
5. Localize os alunos cuja idade está armazenada como string.
	`db.alunos.find({idade: {$type: 'string'} })`
	
6. Localize documentos que possuem idade int e também possuem curso.
	`db.alunos.find({idade: {$type: 'int'}, curso: {$exists: true} })`
	

#### Criando uma coleção validada

7. Crie a coleção turmas. codigo, disciplina e semestre devem ser obrigatórios.
```
db.createCollection("turmas", {
  validator: {
	$jsonSchema: {
	  bsonType: "object",
	  required: ["codigo", "disciplina", "semestre"],   
	}
  }
})
```
Teste de insert
`db.turmas.insertOne({codigo: "ESOFT", disciplina: "NOSQL", semestre: "6"})` valido
`db.turmas.insertOne({codigo: "ESOFT", disciplina: "NOSQL"})` inválido

8. codigo e disciplina devem ser string; semestre deve ser int e possuir minimum 1.
```
	db.runCommand({
	  collMod: "turmas",
	  validator: { $jsonSchema: {
	      bsonType: "object",
	      required: ["codigo", "disciplina", "semestre"],
	      properties: {
	        codigo: {bsonType: 'string'},
	        disciplina: {bsonType: 'string'},
	        semestre: {bsonType: 'int', minimum: 1}
		  }
	}},
  validationLevel: "strict",
  validationAction: "error"
})
```
Teste
```
db.turmas.updateOne(
  {codigo: "ESOFT"}, 
  {$set: {codigo: "ESOFTNA"}}
)
```
Causa erro pois o semestre foi inserido como `"6"` e precisa ser `int`. Após:
```
db.turmas.updateOne(
  {codigo: "ESOFT"}, 
  {$set: {semestre: 6}}
)
```
Volta a ser válido

9. vagas deve ser int, possuir minimum 0 e ainda não será obrigatória.
```
db.runCommand({
  collMod: 'turmas',
  validator: { $jsonSchema: {
	      bsonType: "object",
	      required: ["codigo", "disciplina", "semestre"],
	      properties: {
	        codigo: {bsonType: 'string'},
	        disciplina: {bsonType: 'string'},
	        semestre: {bsonType: 'int', minimum: 1},
	        vagas: {bsonType: 'int', minimum: 0}
		  }
	}}
})
```
Teste
```
db.turmas.insertOne({codigo: "ADS", disciplina: "GO", semestre: 1})
```
Funcionou por conta que 'vagas' não são obrigatórias
```
db.turmas.updateOne(
  {codigo: 'ADS'},
  {$set: {vagas: -1}}
)
```
Falhou por conta do valor mínimo
```
db.turmas.updateOne(
  {codigo: 'ADS'},
  {$set: {vagas: 10}}
)
```
Funcionou
10. Configure validationLevel como strict e validationAction como error.
```
db.runCommand({
  collMod: 'turmas',
  validator: {
    $jsonSchema: {
	      bsonType: "object",
	      required: ["codigo", "disciplina", "semestre"],
	      properties: {
	        codigo: {bsonType: 'string'},
	        disciplina: {bsonType: 'string'},
	        semestre: {bsonType: 'int', minimum: 1},
	        vagas: {bsonType: 'int', minimum: 0}
		  }
	}
  },
  validationLevel: 'strict',
  validationAction: 'error'
})
```

11. Teste um documento válido, um sem disciplina, um com semestre string e um com vagas negativa.
Valido
```
db.turmas.insertOne(
  {codigo: 'FARMA', disciplina: 'ANATOMIA', semestre: 4}
)
```
Sem disciplina (Document failed validation)
```
db.turmas.insertOne(
  {codigo: 'ENG_MEC', semestre: 1}
)
```
Semestre em string (Document failed validation)
```
db.turmas.insertOne(
  {codigo: 'ENG_MEC', disciplina: 'CALCULO_I', semestre: '2'}
)
```
Vagas negativas (Document failed validation)
```
db.turmas.insertOne(
  {codigo: 'ENG_MEC', disciplina: 'CALCULO_I', semestre: 2, vagas: -2}
)
```

#### Evoluindo o validador com collMod

12. Torne vagas obrigatória.
Modificada a linha
```
required: ["codigo", "disciplina", "semestre", "vagas"],
```
Adicionando vagas, e testando com
```
db.turmas.insertOne(
  {codigo: 'ENG_MEC', disciplina: 'CALCULO_I', semestre: 2}
) 
#Document failed validation

db.turmas.insertOne(
  {codigo: 'ENG_MEC', disciplina: 'CALCULO_I', semestre: 2, vagas: 0}
) 
#successo
```

13. Acrescente o campo obrigatório ativa, do tipo bool.
Alterado
```
required: ["codigo", "disciplina", "semestre", "vagas", "ativa"],
ativa: {bsonType: 'bool'}
```
Ao tentar alterar uma outra propriedade recebemos o erro
`[MongoServerError:](#) Plan executor error during update :: caused by :: Document failed validation`
Por conta que agora o campo 'ativa' é obrigatório

14. Preserve todas as regras anteriores no novo validator.

15. Continue usando strict + error.

16. Teste uma turma válida, uma sem ativa e uma com ativa: "sim".
Valida
```
db.turmas.insertOne(
  {codigo: 'PSICO', disciplina: 'ANATO', semestre: 6, vagas: 30, ativa: true}
)
```
Sem ativa
```
db.turmas.insertOne(
  {codigo: 'DIREITO', disciplina: 'COD. PENAL', semestre: 6, vagas: 12}
)
#Document failed validation
```
Com ativa 'sim'
```
db.turmas.insertOne(
  {codigo: 'DIREITO', disciplina: 'COD. PENAL', semestre: 6, vagas: 12, ativa: 'sim'}
)
#Document failed validation
```
Erro
```
propertyName: 'ativa',
            details: [
              {
				operatorName: 'bsonType',
				specifiedAs: {
				  bsonType: 'bool'
				},
				reason: 'type did not match',
				consideredValue: 'sim',
				consideredType: 'string'
			   }
			]
```

#### A armadilha do validador incompleto
17. Execute um collMod contendo somente a regra obrigatória ativa: bool.
```
db.runCommand({
  collMod: 'turmas',
  validator:{
    $jsonSchema: {
      bsonType: 'object',
      required: ["ativa"],
      properties: {
        ativa: {bsonType: 'bool'}
      }
    }
  },
  validationLevel: 'strict',
  validationAction: 'error'
})
```

18. Tente inserir apenas { ativa: true }
```
db.turmas.insertOne(
  {ativa: true}
)
#Sucesso
```

19. Explique por que a inserção foi aceita.
	A inserção foi aceita pois o collMod acima sobrescreveu o `validator` definido anteriormente. Todas as configurações foram perdidas e a única no momento é a que, precisa conter um campo `ativa` do tipo boolean
	

#### moderate + error em dados legados

21. Na coleção alunos, exija nome string, idade int com minimum 0 e curso string; os três campos devem ser obrigatórios.
```
db.runCommand({
  collMod: 'alunos',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['nome', 'idade', 'curso'],
      properties: {
        nome: {bsonType: 'string'},
        idade: {bsonType: 'int', minimum: 0},
        curso: {bsonType: 'string'}
      }
    }
  },
  validationLevel: 'moderate',
  validationAction: 'error'
})
```

22.  Configure moderate + error.
	Já feito no passo anterior.
	
23. Antes de executar, classifique cada operação como PERMITE ou BLOQUEIA: 
	[[Validation Level e Action]]
	
	a) inserir um novo aluno sem curso; 
		BLOQUEAR
	b) alterar somente o nome de Bruno; 
		PERMITIR
	c) alterar somente o nome de Ana; 
		PERMITIR
	d) remover curso de Ana; 
		BLOQUEAR
	e) corrigir idade de Carla sem alterar outro campo.
		PERMITIR

24. Comprovar questão anterior:
a) inserir um novo aluno sem curso
```
db.alunos.insertOne(
  {nome: 'Zé', idade: 22}
)
#Document failed validation
```
	
b) alterar somente o nome de Bruno; 
```
db.alunos.updateOne(
  {nome: 'Bruno'},
  {$set: {nome: 'Bruno Ernesto'} }
)
#Sucesso
```

c) alterar somente o nome de Ana; 
```
db.alunos.updateOne(
  {nome: 'Ana'},
  {$set: {nome: 'Ana Julia'} }
)
#Sucesso
```

d) remover curso de Ana; 
```
db.alunos.updateOne(
  {nome: 'Ana Julia'},
  {$unset: {curso: ''} }
)
# missingProperties 'curso'
```

e) corrigir idade de Carla sem alterar outro campo.
```
db.alunos.updateOne(
  {nome: 'Carla'},
  {$set: {idade: 20} }
)
#Sucesso
```

#### Alterando error para warn
25. Mantenha validationLevel: moderate e o validator completo.
Nenhuma alteração feita.

26. Altere validationAction para warn.
```
db.runCommand({
  collMod: 'alunos',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['nome', 'idade', 'curso'],
      properties: {
        nome: {bsonType: 'string'},
        idade: {bsonType: 'int', minimum: 0},
        curso: {bsonType: 'string'}
      }
    }
  },
  validationLevel: 'moderate',
  validationAction: 'warn'
})
```

27. Insira Eduardo com idade int, mas sem curso.
```
db.alunos.insertOne({
  nome: 'Eduardo', idade: 49
})
# Sucesso
```

28. Use find() para verificar se Eduardo foi armazenado.
Foi armazenado
```
db.alunos.find(
  {nome: 'Eduardo'}
)
{
  _id: ObjectId('6a870f17f8b3bc9d98733ca0'),
  nome: 'Eduardo',
  idade: 49
}
```

29. Explique a diferença entre error e warn e onde a advertência é registrada.
Warn: Vai apenas apresentar uma mensagem de aviso e registrar um LOG. Permitindo a adição ou alteração no banco.
Error: Apresenta mensagem de erro e bloqueia a ação, não salvando ou alterando informações.

Os erros e warns são salvos nos logs do mongo, eles podem ser encontrados em 
(Linux) `/var/log/mongodb/mongod.log` 
ou via docker com `docker logs -f <nome_do_container_mongo>` 


#### Migração de moderate para strict
30. Localize documentos sem curso usando find() e $exists.
```
db.alunos.find(
  {curso: {$exists: false} }
)
```

31. Localize idades armazenadas como string usando find() e $type.
```
db.alunos.find(
  {idade: {$type: 'string'} }
)
```

Combinando as questões 30 e 31:
```
db.alunos.find(
  { $or : [
	  {curso: {$exists: false} }, 
	  {idade: {$type: 'string'} }
	  ] 
   })
```

32. Corrija Bruno e Carla.
Bruno
```
db.alunos.updateOne(
  {nome: 'Bruno Ernesto'},
  {$set: {curso: 'ESOFT'} }
)
```

Carla
```
db.alunos.updateOne(
  {nome: 'Carla'},
  {$set: {idade: 20} }
)
```

33. Confirme que as consultas de pendências não retornam documentos.
Sim, retornou o usuário 
```
{
  _id: ObjectId('6a870f17f8b3bc9d98733ca0'),
  nome: 'Eduardo',
  idade: 49
}
```
e foi ajustado
```
db.alunos.updateOne(
  {nome: 'Eduardo'},
  {$set: {curso: 'PSICO'} }
)
```

34. Reenvie o validator completo usando strict + error.
```
db.runCommand({
  collMod: 'alunos',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['nome', 'idade', 'curso'],
      properties: {
        nome: {bsonType: 'string'},
        idade: {bsonType: 'int', minimum: 0},
        curso: {bsonType: 'string'}
      }
    }
  },
  validationLevel: 'strict',
  validationAction: 'error'
})
```

35. Tente remover curso de um aluno válido e explique o resultado.
```
db.alunos.updateOne(
  {nome: 'Eduardo'},
  {$unset: {curso: ''} }
)
#Erro
missingProperties: [
  'curso'
]
```
Ao tentar remover um curso com a nova regra de validação o banco nega a alteração pois, o `strict` emite a validação onde um documento se tornara inválido e o `error` bloqueia a execução.

#### Desafio de aplicação
36. Crie a coleção produtos com nome string obrigatório, estoque int obrigatório com minimum 0 e preco double obrigatório com minimum 0.
```
db.createCollection('produtos', {
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['nome', 'estoque', 'preco'],
      properties: {
        nome: {bsonType: 'string'},
        estoque: {bsonType: 'int', minimum: 0},
        preco: {bsonType: 'double', minimum: 0}
      }
    }
  }
})
```

37. Use strict + error.
```
db.runCommand({
  collMod: 'produtos',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['nome', 'estoque', 'preco'],
      properties: {
        nome: {bsonType: 'string'},
        estoque: {bsonType: 'int', minimum: 0},
        preco: {bsonType: 'double', minimum: 0}
      }
    }
  },
  validationLevel: 'strict',
  validationAction: 'error'
})
```

38. Insira dois produtos válidos.
Inseri separados para testar o `Double()`
```
db.produtos.insertOne(
  {nome: 'Erva mate pf', preco: 20.2, estoque: 50}
)
db.produtos.insertOne(
  {nome: 'Erva mate lc', preco: Double(20), estoque: 50}
)
```

39. Use find() e $type para localizar preços do tipo double e estoques do tipo int.
```
db.produtos.find(
  {estoque: {$type: 'int'} }
)
# 2 resultados
db.produtos.find(
  {preco: {$type: 'double' } }
)
# 2 resultados
```

40. Use collMod para acrescentar o campo obrigatório ativo, do tipo bool, preservando todas as regras.
```
db.runCommand({
  collMod: 'produtos',
  validator: {
    $jsonSchema: {
      bsonType: 'object',
      required: ['nome', 'estoque', 'preco', 'ativo'],
      properties: {
        nome: {bsonType: 'string'},
        estoque: {bsonType: 'int', minimum: 0},
        preco: {bsonType: 'double', minimum: 0},
        ativo: {bsonType: 'bool'}
      }
    }
  },
  validationLevel: 'strict',
  validationAction: 'error'
})
```
