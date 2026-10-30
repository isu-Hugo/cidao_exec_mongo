/^Ana/ começa com ana
/Julia$/ termina com Silva
/Molinari/ Contem Molinari na frase
/^An(a|dré)/ Começa com An e depois vem a ou dré -> Ana ou André
/i desconsiderar upper ou down case

^ inicio da String
$ Fim da String
. Um caracter qualquer (ana, ata, aba)
[AEIOU] Verificar se tem caracter na lista
[^ ] Caracter fora da lista
| ou

\* Zero ou mais vezes
\+ Um ou mais vezes
? Zero ou uma vez
\d um digito
\w um caracter (letra, numero ou _ )
\w+ uma palavra
\s espaço
{n} exatamente n vezes
{n, } n ou mais
{n, m} entre n e m
