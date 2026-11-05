# Modelo de dados — NotasMAX (MongoDB)

A documentação vigente descreve o modelo documental observado no MongoDB Compass em 05/11/2026.

- [Modelo de dados MongoDB](modelo-dados-mongodb.md): diagrama de coleções, estrutura dos subdocumentos, descrição do modelo, campos e tipos BSON.
- [Diagrama de classes UML](diagrama-classes-mongodb.md): classes dos modelos de persistência, relacionamentos e descrição explicativa.

Os diagramas Mermaid são renderizados diretamente pelo GitHub. As referências por `ObjectId` são associações lógicas; as composições representam documentos incorporados.

> O modelo é baseado na exportação do Compass; ele não comprova restrições de validação ou índices existentes no servidor.

## Arquivos históricos

As propostas relacionais descontinuadas foram preservadas apenas para consulta histórica: [modelo PostgreSQL](historico-modelo-postgresql.md) e [matriz física PostgreSQL](historico-matriz-fisica-postgresql.md). **Não representam o banco atualmente utilizado.**
