# Matriz física inicial — V1

**Atualizado em:** 2026-10-10
**Escopo:** tabelas de domínio e suporte necessárias à V1; presença, importação Excel e notificações da V2 ficam fora.

Esta matriz traduz as decisões registradas no [modelo conceitual](README.md) e as definições para a V1 em uma proposta PostgreSQL, distinguindo regras definidas de escolhas técnicas ainda propostas. A matriz serviu de base para o schema inicial; itens marcados como pendentes continuam sem decisão e precisam ser resolvidos antes de qualquer mudança correspondente no DDL.

## Convenções físicas

- Nomes de tabela e coluna em `snake_case`; são nomes propostos para o schema.
- Entidades independentes usam `BIGINT IDENTITY`. Extensões 1:1 compartilham a chave do usuário; tabelas de associação e snapshots usam chaves compostas quando a combinação identifica naturalmente a linha.
- Datas acadêmicas e períodos usam `DATE`; instantes de evento usam `TIMESTAMPTZ` e são enviados/tratados em UTC.
- Estados e tipos finitos usam `TEXT` com `CHECK`; valores permitidos são comparados em minúsculas.
- FKs usam `ON DELETE RESTRICT`; os fluxos aprovados usam exclusão lógica onde indicado e operações transacionais para remover relações permitidas.
- Não criar tabelas de log ou histórico de alterações de resultado. `ativado_em`, `cancelada_em` e `instante_confirmacao_realizacao` representam o estado/instante de negócio aprovado, não um histórico de eventos.
- `DEFAULT —` significa que a API fornece o valor. `Pendente` identifica regra ainda não aprovada ou precisão ainda não fechada.

## 1. `usuario`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_usuario` | `BIGINT IDENTITY` | NN / gerado | PK; `UNIQUE (id_usuario, tipo_perfil)` para FKs compostas das extensões | Chave aprovada |
| `tipo_perfil` | `TEXT` | NN / — | `CHECK` em `aluno`, `professor`, `administrador`; `UNIQUE (id_usuario, tipo_perfil)` para FKs compostas | Valores aprovados |
| `nome_completo` | `TEXT` | NN / — | Não vazio após `btrim`; `CHECK (char_length(nome_completo) BETWEEN 1 AND 255)` | Campo e limite de 255 caracteres aprovados |
| `email_institucional` | `TEXT` | NN / — | `UNIQUE` global, inclusive contas excluídas logicamente; `CHECK (email_institucional = lower(email_institucional))` | Normalização e unicidade aprovadas |
| `email_pendente` | `TEXT` | NULL / `NULL` | Endereço novo aguardando confirmação; normalizado em minúsculas. Impedir colisão com qualquer endereço atual ou pendente. | Aprovado |
| `telefone_contato` | `TEXT` | NULL / — | Armazenar normalizado em E.164; validar e normalizar na API | Opcional e formato E.164 aprovados |
| `hash_senha` | `TEXT` | NULL / `NULL` | Armazenar PHC Argon2id; nunca senha em texto puro | Algoritmo e parâmetros de referência aprovados: 19 MiB, 2 iterações, paralelismo 1; benchmark no runtime Azure permanece pendente |
| `ativado_em` | `TIMESTAMPTZ` | NULL / `NULL` | NULL indica conta pendente; não há histórico de ativações | Regra aprovada |
| `excluido_em` | `TIMESTAMPTZ` | NULL / `NULL` | Exclusão lógica; preserva vínculos e bloqueia login | Regra aprovada |
| `falhas_login_na_janela` | `INTEGER` | NN / `0` | `CHECK >= 0` | Bloqueio por conta aprovado |
| `inicio_janela_falhas_login` | `TIMESTAMPTZ` | NULL / `NULL` | Início da janela de 15 min | Regra aprovada |
| `bloqueado_ate` | `TIMESTAMPTZ` | NULL / `NULL` | Bloqueio temporário após 5 falhas | Regra aprovada |
| `contador_pedidos_redefinicao` | `INTEGER` | NN / `0` | `usuario_contador_pedidos_redefinicao_ck CHECK (contador_pedidos_redefinicao >= 0)`; até três pedidos por usuário em janela fixa de 24 horas iniciada no primeiro pedido; sem limite por IP neste fluxo. A constraint só impede valor negativo; a API aplica o máximo de três. Zerar junto com `inicio_janela_redefinicao` após redefinição concluída ou login bem-sucedido. Pedido excedente mantém resposta genérica `200 OK` e não envia e-mail. Incremento e abertura da janela atômicos sob concorrência. | Campo e constraint incluídos no schema inicial; máximo de três aplicado pela API |
| `inicio_janela_redefinicao` | `TIMESTAMPTZ` | NULL / `NULL` | Instante do primeiro pedido contabilizado na janela fixa de 24 horas; limpar junto com o contador após redefinição concluída ou login bem-sucedido. | Campo incluído no schema inicial |

**Índices e integridade:** `UNIQUE(email_institucional)`; índice único parcial em `email_pendente` quando não nulo; verificação transacional no banco para impedir colisão cruzada entre endereços atuais e pendentes, inclusive em alterações concorrentes. Índice para listagens administrativas por perfil/nome, restrito a usuários não excluídos. Não indexar campos de bloqueio sem evidência de consulta.

Ao iniciar a troca, `ativado_em` fica nulo e a conta não aceita login. O endereço atual permanece até a confirmação do novo endereço pelo fluxo de ativação; então `email_pendente` substitui `email_institucional`, volta a NULL e a conta é ativada. As sessões são revogadas após a confirmação. A unicidade cruzada será protegida por função/trigger no banco com advisory transaction locks sobre e-mails normalizados, adquiridos em ordem estável, e verificação conjunta de e-mails atuais e pendentes.

## 2. `aluno` e `professor`

`administrador` é representado apenas por `usuario.tipo_perfil`; não há tabela `administrador` aprovada.

| Tabela | Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|---|
| `aluno` | `id_usuario` | `BIGINT` | NN / — | PK e FK para usuário | Extensão 1:1 aprovada |
| `aluno` | `tipo_perfil` | `TEXT` | NN / `'aluno'` | `CHECK (tipo_perfil = 'aluno')`; FK composta para `usuario(id_usuario, tipo_perfil)`, diferida | FKs compostas aprovadas |
| `aluno` | `telefone_responsavel` | `TEXT` | NN / — | Não vazio após `btrim` | Obrigatório aprovado |
| `professor` | `id_usuario` | `BIGINT` | NN / — | PK e FK para usuário | Extensão 1:1 aprovada |
| `professor` | `tipo_perfil` | `TEXT` | NN / `'professor'` | `CHECK (tipo_perfil = 'professor')`; FK composta diferida para `usuario` | FKs compostas aprovadas |

Um constraint trigger diferido deve verificar ao fim da transação que todo usuário `aluno` tem somente extensão `aluno`, todo usuário `professor` tem somente extensão `professor` e usuários `administrador` não têm extensão. A criação do usuário e de sua extensão ocorre na mesma transação.

Campos pessoais da V1 confirmados: nome completo, e-mail institucional e telefone de contato opcional em `usuario`; telefone do responsável obrigatório em `aluno`. `professor` não terá atributos cadastrais próprios adicionais nesta versão.

## 3. `materia`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_materia` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| `nome` | `TEXT` | NN / — | Duplicidade de matérias ativas bloqueada; nome reutilizável após exclusão lógica | Regra aprovada; comparar sem diferenciar caixa ou espaços externos, mantendo acentos significativos |
| `descricao` | `TEXT` | NULL / `NULL` | Sem constraint de conteúdo definida | Opcional aprovado |
| `excluido_em` | `TIMESTAMPTZ` | NULL / `NULL` | Exclusão lógica | Regra aprovada |

**Índice aprovado:** unicidade de matérias ativas por `lower(btrim(nome))`, com predicado `excluido_em IS NULL`; preservar a grafia exibida em `nome`. A comparação ignora caixa e espaços externos, mas considera acentos diferentes.

## 4. `turma`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_turma` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| `serie` | `INTEGER` | NN / — | `CHECK (serie IN (1, 2, 3))`; `UNIQUE (serie, ano_letivo)` | Série e unicidade aprovadas |
| `ano_letivo` | `INTEGER` | NN / — | `UNIQUE (serie, ano_letivo)` | Campo aprovado |

Não foi aprovado campo de exclusão lógica de turma; a API bloqueia edição de série/ano quando há vínculos. Na V1, a turma terá somente série e ano letivo; não haverá turno, nome adicional ou outro campo complementar.

## 5. `matricula`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_matricula` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| `id_usuario_aluno` | `BIGINT` | NN / — | FK para `aluno(id_usuario)` | Nomenclatura aprovada |
| `id_turma` | `BIGINT` | NN / — | FK para `turma(id_turma)` | Nomenclatura aprovada |
| `inicio_vigencia` | `DATE` | NN / fornecido pela API | Início inclusivo | Regra aprovada |
| `fim_vigencia` | `DATE` | NULL / `NULL` | Primeiro dia em que o aluno já não está matriculado; fim exclusivo; `CHECK (fim_vigencia IS NULL OR fim_vigencia > inicio_vigencia)` | Regra aprovada; data efetiva informada pelo administrador |
| `cancelada_em` | `TIMESTAMPTZ` | NULL / `NULL` | Instante UTC em que o cancelamento foi confirmado; não armazena o administrador autor | Regra aprovada; independente da data efetiva |

**Índices e integridade aprovados:** busca por aluno e turma; índice único parcial em `id_usuario_aluno` para matrícula sem `fim_vigencia`, evitando duas matrículas abertas do mesmo aluno. Para criar ou alterar matrícula, a API bloqueia a linha do aluno dentro da transação, verifica sobreposição usando intervalos `[início, fim)` e rejeita conflito antes de gravar.

## 6. `turma_disciplina` e `turma_disciplina_professor`

| Tabela | Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|---|
| `turma_disciplina` | `id_turma_disciplina` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| `turma_disciplina` | `id_turma` | `BIGINT` | NN / — | FK para `turma` | Aprovado conceitualmente |
| `turma_disciplina` | `id_materia` | `BIGINT` | NN / — | FK para `materia` | Aprovado conceitualmente |
| `turma_disciplina` | par turma/matéria | — | — | `UNIQUE (id_turma, id_materia)` | Oferta única aprovada |
| `turma_disciplina_professor` | `id_turma_disciplina` | `BIGINT` | NN / — | FK para `turma_disciplina` | Associação atual |
| `turma_disciplina_professor` | `id_usuario_professor` | `BIGINT` | NN / — | FK para `professor(id_usuario)` | Associação atual |
| `turma_disciplina_professor` | par oferta/professor | — | — | PK composta; não manter histórico | Vários professores e ausência de histórico aprovados |

Ao remover a oferta de uma matéria da turma, a API remove também, na mesma transação, as associações atuais dessa oferta com professores. Os cadastros dos professores são preservados e não se mantém histórico. Simulados futuros devem ser atualizados para retirar a matéria antes da remoção; simulado realizado bloqueia a remoção conforme a regra aprovada.

## 7. `sessao`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_sessao` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| `id_usuario` | `BIGINT` | NN / — | FK para `usuario` | Aprovado |
| `hash_token_sha256` | `BYTEA` | NN / — | `UNIQUE`; `CHECK (octet_length(hash_token_sha256) = 32)` | Armazenar somente os 32 bytes do hash SHA-256 aprovado |
| `criada_em` | `TIMESTAMPTZ` | NN / instante atual | Início da sessão | Aprovado |
| `ultima_atividade_em` | `TIMESTAMPTZ` | NN / instante atual | Atualizada a cada atividade autenticada | Aprovado |
| `expira_em` | `TIMESTAMPTZ` | NN / — | Expiração deslizante; não posterior a `expira_absoluta_em` | 15 min aprovado |
| `expira_absoluta_em` | `TIMESTAMPTZ` | NN / — | Limite absoluto da sessão | 8 h aprovado |
| `revogada_em` | `TIMESTAMPTZ` | NULL / `NULL` | Nulo enquanto a sessão não for revogada | Aprovado |

**Índices aprovados:** único no hash; `(id_usuario, expira_em)` para sessões não revogadas, com predicado `revogada_em IS NULL`. Não armazenar token original, IP ou user-agent.

## 8. `token_ativacao` e `token_redefinicao_senha`

| Tabela | Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|---|
| ambas | `id_token` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| ambas | `id_usuario` | `BIGINT` | NN / — | FK para `usuario` | Aprovado |
| ambas | `hash_token_sha256` | `BYTEA` | NN / — | `UNIQUE`; `CHECK (octet_length(hash_token_sha256) = 32)` | Hash SHA-256 armazenado como 32 bytes aprovado |
| ambas | `expira_em` | `TIMESTAMPTZ` | NN / fornecido pela API | Validade | Aprovado |

**Índices e ciclo de vida aprovados:** `UNIQUE (id_usuario)` e unicidade do hash em cada tabela. Antes de emitir token novo, apagar o anterior; conferir a expiração na API e remover tokens expirados por limpeza diária. Ao consumir o token, bloquear o registro e concluir a operação e sua exclusão na mesma transação, impedindo reutilização concorrente. O token de ativação confirma o endereço pendente quando houver troca de e-mail. Não manter histórico de tokens usados, invalidados ou expirados.

## 9. `foto_perfil`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_usuario` | `BIGINT` | NN / — | PK e FK para `usuario`; uma foto atual por usuário | Relação 1:1 aprovada |
| `chave_objeto` | `TEXT` | NN / — | `UNIQUE`; chave no container privado Azure Blob | Aprovado conceitualmente |
| `tipo_midia` | `TEXT` | NN / — | `CHECK` em `image/jpeg`, `image/png` | Formatos aprovados |
| `tamanho_bytes` | `INTEGER` | NN / — | `CHECK (tamanho_bytes BETWEEN 1 AND 5242880)` | Limite 5 MB aprovado |
| `criada_em` | `TIMESTAMPTZ` | NN / instante atual | Primeiro envio | Aprovado |
| `atualizada_em` | `TIMESTAMPTZ` | NN / instante atual | Atualizar ao substituir a foto; não é histórico de versões | Atualização e ausência de histórico aprovadas |

O arquivo binário não fica no PostgreSQL. Ao substituir ou excluir logicamente a conta, apagar a metadata e o objeto anterior conforme a regra aprovada; não manter histórico de fotos.

## 10. `simulado`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_simulado` | `BIGINT IDENTITY` | NN / gerado | PK | Convenção aprovada |
| `id_turma` | `BIGINT` | NN / — | FK para `turma`; compõe chave única `(id_simulado, id_turma)` usada pelas FKs compostas | Aprovado conceitualmente |
| `numero` | `INTEGER` | NN / — | `CHECK (numero > 0)`; número único por turma e ano letivo | Regra aprovada |
| `tipo` | `TEXT` | NN / — | `CHECK` em `objetivo`, `dissertativo` | Valores aprovados |
| `bimestre` | `SMALLINT` | NN / — | `CHECK (bimestre BETWEEN 1 AND 4)` | Regra aprovada |
| `data_realizacao` | `DATE` | NN / — | Data de negócio; pode estar no passado com aviso | Aprovado |
| `instante_confirmacao_realizacao` | `TIMESTAMPTZ` | NULL / `NULL` | Preenchido na confirmação pelo administrador; evento em UTC | Aprovado |
| `excluido_em` | `TIMESTAMPTZ` | NULL / `NULL` | Exclusão lógica; número pode ser reutilizado depois | Regra aprovada |

**Índices aprovados:** `UNIQUE (id_turma, numero) WHERE excluido_em IS NULL`; `UNIQUE (id_simulado, id_turma)` para FK composta; índice para consultas por turma/bimestre/data. Exclusão lógica somente antes da confirmação de realização e se não existir nenhum registro em `resultado`, independentemente do estado; após congelar participantes, bloquear exclusão. A API retorna o erro `SIMULATION_HAS_LINKED_ANSWERS`; status e corpo detalhado ainda serão fechados no contrato do endpoint.

## 11. `simulado_disciplina`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_simulado` | `BIGINT` | NN / — | Parte da PK; FK composta para simulado e turma | Aprovado conceitualmente |
| `id_turma` | `BIGINT` | NN / — | Parte da PK; FK composta para a oferta da matéria | Denormalização aprovada para integridade |
| `id_materia` | `BIGINT` | NN / — | Parte da PK; deve ser matéria oferecida na turma | Aprovado conceitualmente |
| `total_questoes` | `INTEGER` | NN / — | `CHECK (total_questoes > 0)` | Inteiro positivo, sem limite máximo adicional de negócio aprovado |
| `peso` | `NUMERIC(5,4)` | NN / — | Proporção de `0.0001` a `1.0000`; `0.3333` equivale a `33,33%`; `CHECK (peso > 0 AND peso <= 1)` | Regra e tipo físico aprovados; precisão comporta passos de 0,01% |

PK aprovada: `(id_simulado, id_turma, id_materia)`. FKs: `(id_simulado, id_turma)` para `simulado`; `(id_turma, id_materia)` para `turma_disciplina`. A interface aceita `xx,xx%`, exibe o percentual ainda disponível ao cadastrar e apresenta o peso junto à identificação de cada simulado. O peso disponível é calculado a partir dos simulados não excluídos da mesma turma, matéria e bimestre, não armazenado como coluna. Valores parciais podem ser cadastrados, mas não podem ultrapassar o saldo; a distribuição fica completa automaticamente em 100%, sem ação de fechamento. Enquanto faltar percentual, a média é provisória e identificada como distribuição incompleta. `0,00%` não é permitido. Para evitar concorrência acima de 100%, a API bloqueia a linha de `turma_disciplina` da matéria durante a transação, recalcula o saldo e só então grava a alteração.

## 12. `simulado_aluno`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_simulado` | `BIGINT` | NN / — | Parte da PK; FK para `simulado` | Aprovado conceitualmente |
| `id_usuario_aluno` | `BIGINT` | NN / — | Parte da PK; FK para `aluno(id_usuario)` | Aprovado conceitualmente |

PK aprovada: `(id_simulado, id_usuario_aluno)`. A API cria o snapshot na confirmação da realização, dentro da mesma transação, conforme matrícula vigente na data do simulado. A data da confirmação já fica em `simulado.instante_confirmacao_realizacao`; não repetir um timestamp por participante.

## 13. `resultado`

| Campo | Tipo proposto | Nulo / padrão | Chave e regra | Estado |
|---|---|---|---|---|
| `id_simulado` | `BIGINT` | NN / — | Parte da PK e de FKs compostas | Aprovado conceitualmente |
| `id_usuario_aluno` | `BIGINT` | NN / — | Parte da PK; FK composta para `simulado_aluno` | Aprovado conceitualmente |
| `id_turma` | `BIGINT` | NN / — | Parte da PK; permite FK composta para `simulado_disciplina` | Denormalização aprovada para integridade entre o mesmo simulado/turma |
| `id_materia` | `BIGINT` | NN / — | Parte da PK; FK composta para `simulado_disciplina` | Aprovado conceitualmente |
| `estado` | `TEXT` | NN / `'pendente'` | `CHECK` em `pendente`, `avaliado`, `ausente` | Valores e estado inicial aprovados |
| `acertos` | `INTEGER` | NULL / `NULL` | `CHECK (acertos IS NULL OR acertos >= 0)`; zero é válido | Aprovado; a API valida `acertos <= total_questoes` na transação |

PK aprovada: `(id_simulado, id_usuario_aluno, id_turma, id_materia)`. FKs compostas para participante `(id_simulado, id_usuario_aluno)` e matéria avaliada `(id_simulado, id_turma, id_materia)` impedem cruzar simulados. `CHECK` exige `acertos` preenchido somente quando `estado='avaliado'`; em `pendente` e `ausente`, `acertos` fica nulo. Como a comparação com `total_questoes` envolve outra tabela, a API bloqueia a linha de `simulado_disciplina` na mesma transação e valida `acertos <= total_questoes`. Somente o administrador pode converter `ausente` em `avaliado`. Nota calculada não é armazenada. Não haverá tabela de histórico/auditoria de alterações de resultado.

## Estado da matriz

O peso é armazenado como proporção decimal e apresentado como percentual; a soma é controlada por turma, matéria e bimestre, com distribuição parcial até 100% e sem peso zero. Esta matriz descreve a estrutura física de referência para o schema V1. As migrations devem preservar as regras e constraints aqui descritas; alterações de regra ou estrutura precisam ser revisadas antes de serem aplicadas.

O seed de teste poderá usar uma senha genérica somente em ambiente de teste isolado. A forma de fornecer a senha inicial do administrador em produção fica pendente para a versão final. A definição formal do escopo e da etapa de implementação continua pendente.

## Referências técnicas

- [PostgreSQL 18 — tipos de dados](https://www.postgresql.org/docs/18/datatype.html)
- [PostgreSQL 18 — constraints](https://www.postgresql.org/docs/18/ddl-constraints.html)
- [PostgreSQL 18 — índices parciais](https://www.postgresql.org/docs/18/indexes-partial.html)
- [OWASP — Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
