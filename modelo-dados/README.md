# Modelo de dados do NotasMAX

## Objetivo e estado

**Atualizado em:** 2026-10-10

Este documento descreve o modelo conceitual da nova versão. As versões web e mobile dos semestres 4 e 5 utilizaram MongoDB/NoSQL; a nova versão terá PostgreSQL como requisito, começará com a base vazia e não migrará registros do MongoDB.

Os diagramas são conceituais. A [matriz física inicial — V1](matriz-fisica-v1.md) detalha tipos, chaves e restrições do schema inicial. O modelo físico inclui a estrutura PostgreSQL e os campos da cota de redefinição. O armazenamento de fotos usa um container privado do Azure Blob Storage; formatos, limites e regras de acesso estão descritos neste documento.

## Regras de domínio aprovadas

### Usuários, perfis e oferta de matérias

- Cada usuário possui um tipo de perfil armazenado em usuario.tipo_perfil e limitado a aluno, professor ou administrador. ALUNO e PROFESSOR são extensões 1:1; o banco deve impedir que a extensão não corresponda ao tipo, e a criação do usuário e da extensão ocorre na mesma transação. Dados específicos podem permanecer nessas tabelas.
- USUARIO armazena nome completo, e-mail institucional e telefone de contato; o telefone é opcional para todos os perfis. ALUNO armazena também o telefone obrigatório do responsável.
- O e-mail institucional é convertido para minúsculas antes de armazenar e comparar; sua unicidade vale entre todas as contas.
- Uma matéria pode ser oferecida em uma turma por meio de turma_disciplina.
- Cada matéria/turma pode ter vários professores. turma_disciplina_professor representa apenas a associação atual; não haverá histórico de atribuições.
- Na criação do simulado, a lista inicial contém as matérias oferecidas pela turma selecionada. O administrador pode remover matérias e restaurar a lista completa. A seleção não apresenta professores. O simulado só pode ser salvo se houver ao menos uma matéria selecionada.

### Matrículas e participantes

- A matrícula registra o período em que o aluno está vinculado à turma.
- O administrador confirma explicitamente a realização. A confirmação registra um instante de auditoria com fuso, armazenado/tratado em UTC (PostgreSQL timestamptz), separado da data do simulado. Em uma transação, a lista de simulado_aluno é congelada usando as matrículas vigentes na data cadastrada do simulado.
- Um aluno que não era elegível na data não recebe resultado, ausência ou pendência para aquele simulado. O tratamento de alunos que ingressam no meio do ano nos cálculos de médias será definido em versão posterior.
- O início da matrícula é inclusivo e o fim é exclusivo: o período usa a convenção [início, fim).

### Simulados

- O número é inteiro e único por turma e ano letivo, independentemente do bimestre. Após remover um simulado, seu número pode ser reutilizado.
- O bimestre é escolhido manualmente entre 1 e 4; o modelo não define datas para os bimestres.
- Uma data anterior à data atual pode ser cadastrada com aviso. Os dados do simulado podem ser editados até a data de realização; as notas podem ser alteradas sem prazo final.
- O tipo do simulado deve ser exibido e será um dos valores aprovados: `objetivo` ou `dissertativo`.

### Resultados e médias

- Cada participante congelado deve ter um resultado por matéria do simulado; cada combinação participante/matéria tem no máximo um resultado conceitual e começa como pendente.
- Os estados são pendente, avaliado e ausente. Ausência é explícita. Um resultado inexistente não equivale a ausência; nota zero é válida.
- Para resultados avaliados, a nota é calculada a partir de (acertos / total de questões) × 10; a nota calculada não será persistida como dado independente.
- Os pesos dos simulados somam 1,0 (100%) por matéria e bimestre. Exemplo: nota 8,0 com peso 0,4 contribui com 3,2 para a média final da matéria.
- A média final do aluno é a soma de cada nota multiplicada pelo respectivo peso. Resultado ausente contribui com zero. Enquanto houver resultado pendente, a média é provisória e mantém os pesos originais, sem redistribuí-los.
- A média do bimestre da turma é a média aritmética das médias finais dos alunos. Deve haver aviso quando existem alunos com notas provisórias. Não devem ser exibidos avisos de possível distorção por ausência ou não participação em simulados.

## MER conceitual

O desenho guarda identidade e tipo de perfil em USUARIO; dados específicos de aluno, professor ou administrador podem ficar em tabelas próprias quando necessários. Também mostra a oferta da matéria, as associações atuais de professores e os participantes definidos pela matrícula vigente:

~~~mermaid
flowchart LR
    U["USUARIO<br/>tipo_perfil"]
    A["ALUNO"]
    P["PROFESSOR"]
    M["MATRICULA<br/>período de validade"]
    T["TURMA<br/>série e ano letivo"]
    MT["MATERIA"]
    TD["TURMA_DISCIPLINA<br/>oferta da matéria"]
    TDP["TURMA_DISCIPLINA_PROFESSOR<br/>associação atual"]
    S["SIMULADO<br/>número, tipo, bimestre e data"]
    SD["SIMULADO_DISCIPLINA<br/>questões e peso"]
    SA["SIMULADO_ALUNO<br/>participante congelado"]
    R["RESULTADO<br/>estado e nota"]

    U -->|"dados específicos se tipo=aluno"| A
    U -->|"dados específicos se tipo=professor"| P
    A -->|"matrícula"| M
    T -->|"recebe"| M
    T -->|"oferece"| TD
    MT -->|"é oferecida em"| TD
    TD -->|"atribuições atuais"| TDP
    P -->|"pode lecionar em várias associações"| TDP
    T -->|"possui"| S
    S -->|"avalia"| SD
    TD -->|"matéria da turma"| SD
    M -->|"vigente na data"| SA
    S -->|"congela elegíveis"| SA
    A -->|"participa"| SA
    SA -->|"resultado por matéria"| R
    SD -->|"matéria avaliada"| R
~~~

## DER conceitual

A estrutura abaixo registra entidades e atributos relevantes às decisões atuais. Identificadores e nomes são rótulos conceituais; não determinam os nomes finais de tabelas ou colunas.

| Entidade | Responsabilidade |
| --- | --- |
| usuario | Nome, e-mail institucional em minúsculas e, durante a troca, e-mail pendente de confirmação; telefone de contato; hash de senha Argon2id anulável até a primeira ativação; data de ativação atual; exclusão lógica; controle de falhas e bloqueio de login; contador de pedidos de redefinição por usuário em janela fixa de 24 horas iniciada no primeiro pedido; até três pedidos, reset após redefinição concluída ou login bem-sucedido, sem limite por IP, com resposta genérica 200 e sem novo e-mail após o limite |
| sessao | Sessão web vinculada ao usuário; token opaco enviado no cookie e somente seu hash SHA-256 armazenado no PostgreSQL, com criação, última atividade, expiração e revogação |
| token_ativacao, token_redefinicao_senha | Tokens aleatórios com hash SHA-256 único e validade; guardar somente o hash. Manter um registro atual por usuário e finalidade; ao emitir outro, remover o anterior. Remover o registro no consumo e limpar expirados diariamente, sem histórico de tokens. Redefinição só emite token para conta ativada; conta pendente segue pelo fluxo de ativação. |
| foto_perfil | Metadata no PostgreSQL (chave do objeto, tipo de mídia, tamanho e datas) e arquivo em container privado do Azure Blob Storage; formatos JPEG/PNG e limite de 5 MB aprovados. Administradores podem ver fotos de todos os usuários; professores podem ver a própria foto e as fotos dos alunos das turmas às quais estão associados; cada aluno pode ver a própria foto. Não haverá ação independente para remover uma foto; o administrador pode substituí-la na edição do perfil, removendo o arquivo anterior do armazenamento ativo. Manter a foto enquanto a conta estiver ativa; ao excluir logicamente a conta, remover arquivo e metadata do armazenamento ativo |
| aluno, professor | Extensões 1:1 com USUARIO para dados e vínculos próprios; ALUNO inclui telefone obrigatório do responsável |
| materia | Matérias cadastradas |
| turma | Série e ano letivo |
| matricula | Vínculo do aluno com período de validade |
| turma_disciplina | Matéria oferecida em uma turma |
| turma_disciplina_professor | Professores associados atualmente à oferta |
| simulado | Número, tipo, bimestre, data marcada e instante de confirmação da realização |
| simulado_disciplina | Matéria selecionada, total de questões e peso |
| simulado_aluno | Participante elegível congelado na ocorrência |
| resultado | Resultado por participante e matéria |

~~~mermaid
erDiagram
    USUARIO ||--o| ALUNO : dados_especificos_se_tipo_aluno
    USUARIO ||--o| PROFESSOR : dados_especificos_se_tipo_professor
    USUARIO ||--o{ SESSAO : possui
    USUARIO ||--o{ TOKEN_ATIVACAO : recebe
    USUARIO ||--o{ TOKEN_REDEFINICAO_SENHA : recebe
    USUARIO ||--o| FOTO_PERFIL : possui

    USUARIO {
        string tipo_perfil
        string nome_completo
        string email_institucional
        string telefone_contato
        string hash_senha
        timestamptz ativado_em
        timestamptz excluido_em
        integer falhas_login_na_janela
        timestamptz inicio_janela_falhas_login
        timestamptz bloqueado_ate
        integer contador_pedidos_redefinicao
        timestamptz inicio_janela_redefinicao
    }

    SESSAO {
        string hash_token_sha256
        timestamptz criada_em
        timestamptz ultima_atividade_em
        timestamptz expira_em
        timestamptz expira_absoluta_em
        timestamptz revogada_em
    }

    TOKEN_ATIVACAO {
        string hash_token_sha256
        timestamptz expira_em
    }

    TOKEN_REDEFINICAO_SENHA {
        string hash_token_sha256
        timestamptz expira_em
    }
    ALUNO {
        string telefone_responsavel
    }

    ALUNO ||--o{ MATRICULA : possui
    TURMA ||--o{ MATRICULA : recebe

    TURMA ||--o{ TURMA_DISCIPLINA : oferece
    MATERIA ||--o{ TURMA_DISCIPLINA : compoe
    TURMA_DISCIPLINA ||--o{ TURMA_DISCIPLINA_PROFESSOR : possui
    PROFESSOR ||--o{ TURMA_DISCIPLINA_PROFESSOR : associado

    TURMA ||--o{ SIMULADO : possui
    SIMULADO ||--|{ SIMULADO_DISCIPLINA : contem
    TURMA_DISCIPLINA ||--o{ SIMULADO_DISCIPLINA : selecionavel

    SIMULADO ||--o{ SIMULADO_ALUNO : congela
    ALUNO ||--o{ SIMULADO_ALUNO : participa
    SIMULADO_ALUNO ||--o{ RESULTADO : recebe
    SIMULADO_DISCIPLINA ||--o{ RESULTADO : avaliada

    SIMULADO {
        integer numero
        string tipo
        integer bimestre
        date data_realizacao
        timestamptz instante_confirmacao_realizacao
    }

    MATRICULA {
        date inicio_vigencia
        date fim_vigencia
    }

    SIMULADO_DISCIPLINA {
        integer total_questoes
        decimal peso
    }

    RESULTADO {
        integer acertos
        string status_resultado
    }
~~~

Cada usuário possui exatamente um valor de tipo de perfil em USUARIO, limitado a aluno, professor ou administrador. As extensões ALUNO e PROFESSOR devem corresponder ao tipo em USUARIO; o banco deve rejeitar divergências. Cada resultado relaciona exatamente um participante congelado (`simulado_aluno`) a uma matéria do simulado (`simulado_disciplina`), e ambos devem pertencer ao mesmo simulado. A unicidade dessa combinação é aprovada. **Regra aprovada para o schema físico:** usar chaves estrangeiras compostas que incluam o identificador do simulado, impedindo combinações cruzadas. `nota_calculada` é derivada dos acertos e do total de questões, não armazenada como dado independente.

## Diagrama de classes

~~~mermaid
classDiagram
    class Usuario {
        +BIGINT idUsuario
        +String nomeCompleto
        +String emailInstitucional
        +String telefoneContato
        +String hashSenha
        +DateTime ativadoEm
        +DateTime excluidoEm
        +Integer falhasLoginNaJanela
        +DateTime inicioJanelaFalhasLogin
        +DateTime bloqueadoAte
        +String tipoPerfil
    }


    class Aluno {
        +String telefoneResponsavel
    }
    class Professor

    class Materia {
        +BIGINT idMateria
        +String nome
    }

    class Turma {
        +BIGINT idTurma
        +Integer serie
        +Integer anoLetivo
    }

    class Matricula {
        +Date inicioVigencia
        +Date fimVigencia
    }

    class TurmaDisciplina
    class TurmaDisciplinaProfessor

    class Simulado {
        +BIGINT idSimulado
        +Integer numero
        +String tipo
        +Integer bimestre
        +Date dataRealizacao
        +DateTime instanteConfirmacaoRealizacaoUtc
    }

    class SimuladoDisciplina {
        +Integer totalQuestoes
        +Decimal peso
    }

    class SimuladoAluno
    class Resultado {
        +Integer acertos
        +String statusResultado
    }

    Usuario "1" --> "0..1" Aluno : dados se tipo aluno
    Usuario "1" --> "0..1" Professor : dados se tipo professor

    Aluno "1" --> "0..*" Matricula : possui
    Turma "1" --> "0..*" Matricula : recebe
    Turma "1" --> "0..*" TurmaDisciplina : oferece
    Materia "1" --> "0..*" TurmaDisciplina : compoe
    TurmaDisciplina "1" --> "0..*" TurmaDisciplinaProfessor : associacao atual
    Professor "1" --> "0..*" TurmaDisciplinaProfessor : leciona

    Turma "1" --> "0..*" Simulado : possui
    Simulado "1" --> "1..*" SimuladoDisciplina : contem
    TurmaDisciplina "1" --> "0..*" SimuladoDisciplina : selecionavel
    Simulado "1" --> "0..*" SimuladoAluno : congela elegiveis
    Aluno "1" --> "0..*" SimuladoAluno : participa
    SimuladoAluno "1" --> "0..*" Resultado : recebe por materia
    SimuladoDisciplina "1" --> "0..*" Resultado : avaliada

    note for Usuario "tipoPerfil aceita aluno, professor ou administrador. ALUNO e PROFESSOR são extensões 1:1, devem corresponder ao tipo e são criados na mesma transação do usuário. ADMINISTRADOR fica representado em USUARIO."
    note for TurmaDisciplinaProfessor "Pode haver varios professores; nao ha historico."
    note for Simulado "Numero unico por turma e ano letivo, sem depender do bimestre."
    note for SimuladoAluno "Elegibilidade congelada pela matricula vigente na data do simulado."
    note for Resultado "Estados: pendente, avaliado ou ausente. Ausencia explicita conta como zero; ausencia de registro nao equivale a ausente."
~~~

## Fotos e presença

Fotos de perfil fazem parte da V1 e se vinculam ao usuário; o envio pela API em multipart/form-data foi aprovado. O arquivo ficará em container privado do Azure Blob Storage, com metadata no PostgreSQL, e o tamanho máximo será 5 MB. Formatos aceitos aprovados: JPEG e PNG. Administradores podem ver fotos de todos os usuários; professores podem ver a própria foto e as fotos dos alunos das turmas às quais estão associados; cada aluno pode ver a própria foto. Não haverá ação independente para remover uma foto; o administrador pode substituí-la na edição do perfil, removendo o arquivo anterior do armazenamento ativo. A foto será mantida enquanto a conta estiver ativa; na exclusão lógica da conta, arquivo e metadata serão removidos do armazenamento ativo. O registro e a consulta de presença foram adiados para a V2 e não entram no schema da V1.

## Pontos de modelagem ainda abertos





- O armazenamento em Azure Blob privado, os formatos JPEG e PNG, o limite de 5 MB, a matriz de acesso e o ciclo de vida da foto estão definidos. Não existe remoção manual da foto; administrador pode substituí-la na edição do perfil. A foto permanece enquanto a conta estiver ativa e é removida do armazenamento ativo quando substituída ou quando a conta é excluída logicamente.

- Definir como alunos que ingressaram depois de um simulado participarão dos cálculos de média em versões futuras.
- A matriz física V1 detalha a estrutura do schema inicial. Os detalhes físicos que ainda dependem de definição estão marcados na [matriz física V1](matriz-fisica-v1.md).

## Contexto histórico

Os modelos e implementações MongoDB das versões anteriores servem como referência histórica para entender requisitos, mas não definem o modelo atual por si só. Esta nova base começa vazia; não há mapeamento de ObjectIds, carga inicial, reconciliação nem plano de migração de dados MongoDB. Este documento e a matriz física correspondente descrevem o modelo de dados da nova versão.

## Direção para o schema inicial

O schema inicial cobre todas as tabelas da V1, inclusive as usadas por subversões posteriores; presença, importação de Excel e notificações ficam fora por serem itens futuros. As tabelas são organizadas em migrations Knex por assunto. O primeiro administrador é provisionado por seed manual e controlado, sem credenciais fixas no código, com senha em Argon2id e recusa se já houver administrador ativo.

## Regras e decisões técnicas

- **Regra aprovada e representada:** usuario.tipo_perfil guarda exatamente um dos valores aluno, professor ou administrador. A restrição de valores foi aprovada.
- **Integridade de perfil aprovada:** as extensões ALUNO e PROFESSOR devem corresponder a usuario.tipo_perfil; criar o usuário e sua extensão na mesma transação. A técnica de constraint será definida no DDL.
- **Dados cadastrais aprovados:** nome completo, e-mail institucional e telefone de contato ficam em USUARIO; o telefone é opcional para todos os perfis. O telefone do responsável fica em ALUNO e é obrigatório no cadastro do aluno.
- **E-mail aprovado:** converter o endereço institucional inteiro para minúsculas antes de armazenar e comparar; aplicar unicidade entre todas as contas.
- **Credencial aprovada:** incluir USUARIO.hash_senha, anulável até a primeira ativação, e armazenar somente o hash Argon2id da senha; ao ativar a conta, gravar o hash da senha escolhida; nunca guardar a senha em texto puro.
- **Ativação aprovada:** USUARIO.ativado_em é anulável; nulo indica conta pendente e a ativação grava a data e hora atuais. Quando a troca confirmada de e-mail exigir nova ativação, limpar o campo, revogar as sessões ativas e recusar logins até a reativação. Manter o hash de senha atual até a reativação ser concluída e então substituí-lo pela senha escolhida. Não manter histórico de ativações.
- **Exclusão lógica aprovada:** USUARIO.excluido_em é anulável; preenchê-lo ao excluir logicamente aluno, professor ou administrador, sem apagar os vínculos acadêmicos. Revogar todas as sessões da conta em SESSAO.revogada_em e recusar novos logins; o prazo de retenção dos dados permanece sem definição.
- **Bloqueio de login aprovado:** guardar em USUARIO a quantidade de falhas na janela de 15 minutos, o início da janela e o horário até o qual a conta está bloqueada. O bloqueio ocorre após 5 falhas na janela e dura 15 minutos. Após login bem-sucedido, zerar o contador e remover o bloqueio. Se a janela expirar sem bloqueio, a próxima falha inicia outra janela com contador 1; depois que um bloqueio termina, a primeira nova falha também inicia uma janela com contador 1. Não limitar tentativas de login por IP.
- **Pedidos de redefinição de senha:** guardar em USUARIO `contador_pedidos_redefinicao` e `inicio_janela_redefinicao`, com até três pedidos por usuário em janela fixa de 24 horas iniciada no primeiro pedido. Não haverá limite por IP para esse fluxo. Após redefinição concluída ou login bem-sucedido, zerar o contador e limpar o início da janela; após 24 horas, o próximo pedido inicia uma nova janela e conta como o primeiro. Pedidos acima do limite na janela recebem a mesma resposta genérica `200 OK`, sem envio de novo e-mail e sem revelar se a conta existe ou está ativa. Os dois campos e a regra de valor não negativo constam da matriz física. O limite máximo de três é aplicado pela API; a atualização conjunta é atômica sob concorrência. Não criar tabela de log ou histórico para essa contagem.
- **Regra física aprovada:** vincular cada resultado ao participante e à matéria do mesmo simulado por chaves estrangeiras compostas que incluam o identificador do simulado.
- **Tipos aprovados:** `objetivo` e `dissertativo`; a representação física (CHECK, enum ou tabela de referência) será definida na implementação do schema.
- **Estratégias físicas adotadas:** impor a correspondência entre tipo_perfil e ALUNO/PROFESSOR com FKs compostas e constraint triggers adiadas ao fim da transação; armazenar sessões no PostgreSQL com hash único, validade e revogação; armazenar tokens com hash e validade, removendo os registros usados/invalidados e limpando expirados diariamente; guardar os arquivos de foto em container privado do Azure Blob Storage e sua chave, tipo de mídia, tamanho e datas no PostgreSQL. A estrutura do schema inicial segue a matriz física V1. A constraint da cota impede valores negativos; a API aplica o máximo de três pedidos. Formatos JPEG/PNG e limite de 5 MB são os parâmetros definidos para fotos.
- **Escopo inicial aprovado:** incluir todas as tabelas da V1 no primeiro conjunto de schema, em migrations por assunto; excluir as entidades de presença, importação de Excel e notificações da V2.
- **Seed do primeiro administrador aprovado:** execução manual/controlada, sem credenciais fixas no repositório, senha protegida por Argon2id e recusa quando já houver administrador ativo. A forma operacional concreta será detalhada no escopo de implementação.

## Decisões complementares — 2026-10-07

- **Senha de teste do seed:** uma senha genérica poderá ser usada somente em testes isolados; essa permissão não se aplica a ambientes locais compartilhados, homologação ou produção. Fora de testes, não haverá valor padrão ou senha fixa no repositório; o fornecimento será detalhado no escopo de implementação.
- **Troca de e-mail:** guardar o novo endereço em `USUARIO.email_pendente`; enquanto aguarda confirmação, a conta não aceita login e `ativado_em` fica nulo. O endereço atual só é substituído após confirmação pelo fluxo de ativação. A unicidade deve cobrir endereços atuais e pendentes com proteção contra concorrência no banco.
- **Correção de ausência:** somente o administrador pode alterar um resultado de ausente para avaliado. Não será mantido histórico ou registro de auditoria das alterações de resultados.
- **Diretrizes físicas:** tipos, nulabilidade, padrões, chaves, constraints e índices estão detalhados na matriz física. Usar `BIGINT IDENTITY` para IDs internos, `DATE` para datas acadêmicas, `TIMESTAMPTZ` para instantes, `NUMERIC` para pesos com validação de faixa, `TEXT` com `CHECK` para valores finitos e FKs restritivas sem exclusão em cascata. A matriz física V1 serve de referência para o schema inicial.
