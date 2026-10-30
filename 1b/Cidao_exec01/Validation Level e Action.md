#### Validation Level
É o nível de restrição em que o mongo vai atuar, nós temos dois cenários que são:

**Strict:** Independente se um documento já existia ou não, após criado ou alterado ele precisa estar válido.

**moderate:** Semelhante ao strict, mas em documentos inválidos você pode modificar valores **VALIDOS** sem precisar arrumar o documento. Desde que não adicione outra propriedade inválida.

#### Validation Action
É a ação tomada um nível é quebrado:

**Warn:** Vai apenas apresentar uma mensagem de aviso e registrar um LOG. Permitindo a adição ou alteração no banco.

**Error:** Apresenta mensagem de erro e bloqueia a ação, não salvando ou alterando informações.
