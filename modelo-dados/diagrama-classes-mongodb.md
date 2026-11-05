# NotasMAX — Diagrama de classes do modelo de persistência (MongoDB)

Fonte: exportação `NotasMax_05_11_2026.mdm` do MongoDB Compass, fornecida em 05/11/2026.

```mermaid
classDiagram
    direction LR
    class Usuario {
      +ObjectId _id
      +int __v
      +string nome
      +string email
      +string senha
      +string tipo_usuario
      +string nome_responsavel
      +string telefone_contato
      +string telefone_responsavel
      +null resetToken
      +null resetTokenExpiry
      +date createdAt
      +date updatedAt
    }
    class Turma {
      +ObjectId _id
      +int __v
      +ObjectId[] alunos
      +int ano
      +int serie
      +date createdAt
      +date updatedAt
    }
    class Materia {
      +ObjectId _id
      +int __v
      +string nome
      +date createdAt
      +date updatedAt
    }
    class TurmaDisciplina {
      +ObjectId _id
      +int __v
      +ObjectId materia_id
      +ObjectId professor_id
      +ObjectId turma_id
      +date createdAt
      +date updatedAt
    }
    class Simulado {
      +ObjectId _id
      +int __v
      +int bimestre
      +int numero
      +string tipo
      +date data_realizacao
      +ObjectId turma_id
      +Conteudo[] conteudos
      +date createdAt
      +date updatedAt
    }
    class Conteudo {
      +ObjectId _id
      +int peso
      +int quantidade_questoes
      +ObjectId turma_disciplina_id
      +Resultado[] resultados
      +date createdAt
      +date updatedAt
    }
    class Resultado {
      +ObjectId _id
      +ObjectId aluno_id
      +int acertos
      +int nota
      +string notificacao_enviada
      +date createdAt
      +date updatedAt
    }

    Turma "0..*" --> "0..*" Usuario : alunos[]
    TurmaDisciplina "0..*" --> "1" Materia : materia_id
    TurmaDisciplina "0..*" --> "1" Usuario : professor_id
    TurmaDisciplina "0..*" --> "1" Turma : turma_id
    Simulado "0..*" --> "1" Turma : turma_id
    Simulado "1" *-- "0..*" Conteudo : conteudos[]
    Conteudo "1" *-- "0..*" Resultado : resultados[]
    Conteudo "0..*" --> "1" TurmaDisciplina : turma_disciplina_id
    Resultado "0..*" --> "1" Usuario : aluno_id
```

## Descrição do diagrama de classes

O diagrama de classes do **NotasMAX** representa os principais modelos de persistência do sistema, seus atributos e seus relacionamentos, com base na estrutura observada no MongoDB Compass. São sete classes: **`Usuario`**, **`Turma`**, **`Materia`**, **`TurmaDisciplina`**, **`Simulado`**, **`Conteudo`** e **`Resultado`**. `Usuario` reúne os perfis de alunos, professores e administradores; `Turma` e `Materia` organizam as informações acadêmicas; e `TurmaDisciplina` associa turma, professor e disciplina.

`Simulado` contém objetos `Conteudo`, que por sua vez contêm objetos `Resultado`, representando a composição de documentos incorporados no MongoDB. As demais associações são feitas por identificadores **`ObjectId`**. Assim, o diagrama facilita a compreensão dos modelos e de suas ligações, sem representar tabelas ou chaves estrangeiras de um banco relacional.

## Interpretação

- **Associação (`-->`)**: vínculo lógico por `ObjectId`, sem chave estrangeira imposta pelo MongoDB.
- **Composição (`*--`)**: subdocumentos incorporados; `Conteudo` pertence ao array `conteudos` de `Simulado`, e `Resultado` pertence ao array `resultados` de `Conteudo`.
- `Usuario` representa todos os tipos de usuário; `professor_id` e `aluno_id` referenciam usuários com papéis distintos, não coleções próprias.
- As multiplicidades das referências indicam a interpretação lógica esperada. O arquivo Compass marca os relacionamentos como **inferidos**, não como regras de integridade verificadas.
- Os tipos são os tipos BSON identificados no arquivo `.mdm`. `resetToken` e `resetTokenExpiry` apareceram como `null` nos dados inspecionados, o que **não comprova** que esses campos aceitem somente `null` no schema da aplicação.
- `__v` é um campo de versionamento usual do Mongoose. `createdAt` e `updatedAt` são campos de data observados nos documentos.

> Este é um diagrama de classes **dos dados persistidos**, não um diagrama de todas as classes, métodos, serviços e telas da aplicação. A estrutura foi extraída do modelo do Compass, não de uma validação dos schemas atuais do código-fonte.
