# NotasMAX — visão de referência

**Contexto acadêmico:** este documento serve como material de referência sobre o NotasMAX para uma competição didática entre alunos da faculdade. Ele descreve o sistema NotasMAX.

**Atualizado em:** 2026-10-07

## Sumário

- [1. Visão do sistema](#1-visão-do-sistema)
- [2. Requisitos](#2-requisitos)
- [3. Arquitetura](#3-arquitetura)
- [4. Protótipos](#4-protótipos)
- [5. Backlog](#5-backlog)
- [6. Funcionalidades](#6-funcionalidades)
- [7. Estratégia inicial de testes](#7-estratégia-inicial-de-testes)
- [Fontes e rastreabilidade](#fontes-e-rastreabilidade)

## 1. Visão do sistema

O NotasMAX é uma plataforma acadêmica para apoiar a organização de simulados, matérias, turmas e resultados escolares do Colégio Max Beny Macena. Seu objetivo é centralizar o lançamento e a consulta de resultados, permitindo que a administração gerencie os dados e que professores e alunos acompanhem o desempenho conforme seus perfis.

A solução planejada é composta por um website administrativo, uma API compartilhada e uma aplicação mobile para alunos e professores. A nova versão está sendo desenvolvida após versões anteriores web e mobile feitas nos semestres 4 e 5. 

### Perfis e canais

| Perfil | Canal planejado | Principais responsabilidades |
|---|---|---|
| Administrador | Website | Gerenciar cadastros, turmas, matérias, matrículas, simulados e lançamento/consulta de resultados. |
| Professor | Mobile | Consultar turmas e resultados das matérias/turmas associadas no ano atual; alterar notas já lançadas nas associações permitidas. |
| Aluno | Mobile | Consultar matérias, simulados, resultados, médias, gráficos e o próprio perfil conforme a turma atual. |

### Diretrizes de versão

- PostgreSQL é requisito para a nova versão; 

## 2. Requisitos

As tabelas abaixo reúnem os requisitos funcionais RF 1–34 e não funcionais RNF 1–14 registrados na documentação. 

### Requisitos funcionais

| ID | Requisito/decisão atual |
| --- | --- |
| RF 1 | Administrador cadastra aluno com nome completo, e-mail institucional, telefone de contato e telefone do responsável.|
| RF 2 | Administrador cadastra professor. A conta fica pendente e o professor define a própria senha pelo link de ativação; associações são feitas por matéria e turma.|
| RF 3 | Administrador pode cadastrar outros administradores; a pessoa ativa a própria conta.|
| RF 4 | Administrador cria turma com série 1, 2 ou 3 e ano letivo inteiro. Há no máximo uma turma por série/ano, sem divisões A/B; vínculos são feitos em etapas próprias.|
| RF 5 | Administrador cadastra e edita matérias. Exclusão lógica foi aprovada posteriormente; nomes ativos não duplicam e podem ser reutilizados após desativação/exclusão.|
| RF 6 | Importação assistida de planilhas Excel, incluindo prévia, erros, revisão e confirmação.|
| RF 7 | Administrador lança acertos por aluno, matéria e simulado. A nota é calculada por (acertos ÷ total de questões) × 10; detalhes restantes do formulário ainda precisam ser fechados.|
| RF 8 | Administrador consulta turmas; por padrão vê o ano letivo atual e pode selecionar outros anos com turmas cadastradas.|
| RF 9 | Administrador consulta/lista alunos cadastrados.|
| RF 10 | Administrador consulta/lista professores cadastrados.|
| RF 11 | Alteração administrativa de registros limita-se aos casos aprovados nos subitens RF 11.1–11.4.|
| RF 11.1 | Notas podem ser alteradas indefinidamente; substitui o prazo antigo de 15 dias.|
| RF 11.2 | Administrador altera informações cadastrais do aluno.|
| RF 11.3 | Administrador altera informações cadastrais do professor.|
| RF 11.4 | Administrador pode editar turma; campos e efeitos em vínculos existentes ainda precisam ser detalhados.|
| RF 12 | Administrador consulta os resultados registrados.|
| RF 13 | Exibir gráfico de desempenho do aluno por bimestre; métricas e detalhamento de acesso devem seguir as decisões e telas atuais.|
| RF 14 | Comparar médias das turmas por matéria e bimestre. A média da turma é aritmética das médias individuais; médias provisórias devem ser identificadas. Não exibir avisos de distorção pela não participação. O tratamento de quem ingressa no meio do ano fica para versão posterior.|
| RF 15 | Administrador compara a média de um aluno com a média da turma.|
| RF 16 | Administrador compara os acertos da turma em uma matéria.|
| RF 17 | Redefinição de senha por link de e-mail para aluno, professor e administrador. Resposta genérica para e-mail cadastrado ou não, limitação por conta/IP, token de uso único válido por uma hora e invalidação das sessões após redefinir. Política de senha e alguns status HTTP pendentes.|
| RF 17.1 | Administrador altera diretamente a senha de aluno.|
| RF 18 | Administrador cadastra simulados com turma, número, tipo, bimestre, data e matérias. Número inteiro sequencial por turma/ano, reutilizável depois de exclusão; bimestre manual de 1 a 4; matérias da turma selecionadas por padrão; data passada pode ser cadastrada após aviso; edição dos dados até a data de realização. Professores não aparecem na seleção.|
| RF 19 | O sistema distingue os perfis administrador, aluno e professor.|
| RF 20 | Professor altera notas já lançadas nas matérias/turmas associadas a ele no ano atual; lançamento inicial cabe ao administrador.|
| RF 21 | Alunos e professores usam a aplicação mobile.|
| RF 22 | Aluno consulta resultados da turma atual. Após cancelar matrícula, perde acesso aos resultados da turma anterior. Ingressante posterior ao simulado não recebe nota, pendência ou ausência retroativa; efeito na média fica para versão posterior.|
| RF 23 | Professor consulta resultados de alunos nas turmas sob sua responsabilidade no ano atual.|
| RF 24 | Aluno compara a própria média com a média da turma.|
| RF 25 | Professor compara médias dos alunos de uma turma nas matérias que leciona.|
| RF 26 | Aluno consulta calendário de simulados da turma atual; detalhes mostram data, tipo, número e peso separado por matéria.|
| RF 27 | Recuperação mobile por código foi substituída pelo fluxo de link de e-mail do RF 17, com tentativa de abertura do app e fallback web.|
| RF 28 | Aluno consulta matérias oferecidas pela turma à qual está vinculado.|
| RF 29 | Aluno e professor consultam o próprio perfil; edição própria não foi aprovada.|
| RF 30 | Professor consulta turmas do ano atual que tenham matérias associadas a ele.|
| RF 31 | Fotos integram cadastro e edição administrativa dos perfis de aluno, professor e administrador, com envio pela API em multipart/form-data. Tipo, tamanho, acesso, remoção e retenção pendentes.|
| RF 32 | Registrar presença de aluno durante simulado.|
| RF 33 | Identificar aluno para registrar presença; o método ainda não foi escolhido.|
| RF 34 | Consultar presenças do dia do simulado; perfis, filtros e detalhes pendentes.|

### Regras de resultados e médias aprovadas

- Resultado começa como pendente; resultado inexistente não equivale a ausência.
- Ausência é confirmada explicitamente pelo administrador e contribui com nota zero. Zero acertos também é uma nota válida.
- Nota do simulado: `(acertos ÷ total de questões) × 10`.
- Os pesos dos simulados somam 1 (100%) por matéria e bimestre. A média final soma `nota × peso`; valores intermediários mantêm precisão completa e somente a média final é arredondada para duas casas.
- Enquanto houver resultados pendentes, a média é provisória e soma as contribuições resolvidas sem redistribuir os pesos.
- A média da turma é a média aritmética das médias individuais; deve ser identificada como provisória quando houver médias individuais provisórias.
- Notas podem ser alteradas sem prazo final. Não há conversão automática de ausência após 15 dias.

### Requisitos não funcionais


| ID | Requisito legado |
| --- | --- |
| RNF 1 | Navegação fácil, com fluxos intuitivos para cada perfil.|
| RNF 2 | Website responsivo em desktop, tablet e mobile.|
| RNF 3 | Resposta inferior a 3 segundos em conexões comuns.|
| RNF 4 | Aplicação mobile compatível com os principais sistemas, citando Android.|
| RNF 5 | Atender à LGPD, protegendo dados pessoais de alunos e professores.|
| RNF 6 | Usar e-mail institucional para autenticação conforme políticas da instituição.|
| RNF 7 | Autenticação robusta, RBAC e armazenamento seguro de senhas; texto legado cita JWT.|
| RNF 8 | Alta disponibilidade e tolerância a falhas.|
| RNF 9 | Stack antiga: React, Tailwind, Node/Express e MongoDB no web; .NET MAUI/C# no mobile.|
| RNF 10 | Aplicação mobile em MVVM.|
| RNF 11 | API REST com nomes, contratos e erros consistentes.|
| RNF 12 | Versionamento Git, organização, documentação e revisão.|
| RNF 13 | Web e mobile compartilham backend REST.|
| RNF 14 | Integrações futuras, incluindo Excel e presença por crachá/IoT.|

## 3. Arquitetura

### Implementação existente

| Componente | Descrição implementada |
|---|---|
| API | Fundação Azure Functions v4 com Node.js 24 e TypeScript; Knex e `pg` para PostgreSQL. |
| Website | Fundação React, Vite e TypeScript; cliente Axios preparado para uma URL pública de API. Há rotas de demonstração e fallback, ainda sem telas de negócio administrativas. |
| Mobile | Arquitetura do Mobile na nova versão ainda não está definida |
| Banco | PostgreSQL obrigatório para a nova versão. |
| Cloud | Azure |

### Contexto histórico

As versões web e mobile dos semestres 4 e 5 usaram MongoDB/NoSQL.

O [modelo de dados conceitual](../modelo-dados/README.md) detalha perfis, matérias por turma, matrículas, simulados, participantes elegíveis e resultados. Seus diagramas não são schema físico.

## 4. Protótipos

O protótipo do NotasMAX está no [Figma](https://www.figma.com/design/3tUP5eB55kFrgwesGN6qAk/NotasMax).

- [Telas administrativas](https://www.figma.com/design/3tUP5eB55kFrgwesGN6qAk/NotasMax?node-id=0-1): referência visual de cadastros e áreas administrativas.
- [Telas de aluno e professor](https://www.figma.com/design/3tUP5eB55kFrgwesGN6qAk/NotasMax?node-id=35-159): referência visual dos fluxos mobile.

As telas são protótipos, não aprovação automática de cada cartão ou comportamento. Por exemplo, o painel administrativo mostrado no Figma permanece pendente de aprovação; requisitos confirmados e pendentes estão marcados nas seções 2 e 6.

## 5. Backlog

### Versões

A primeira versão V1, focada na API e no website do administrador. 


### Tarefas de backend da API V1

| ID | Tarefa |
| --- | --- |
| API-ADM-01 | Revisar modelo e preparar schema/migrations PostgreSQL, sem migração MongoDB.|
| API-ADM-02 | Completar validação comum e erros RFC 9457 com `code` e `errors[]`.|
| API-ADM-03 | Implementar autenticação, sessão web e autorização server-side.|
| API-ADM-04 | Implementar ciclo de vida de contas, ativação, e-mails e redefinição de senha.|
| API-ADM-05 | Endpoints administrativos de alunos.|
| API-ADM-06 | Endpoints administrativos de professores e associações por matéria/turma.|
| API-ADM-07 | Endpoints administrativos de administradores e regras de exclusão.|
| API-ADM-08 | Endpoints de matérias, edição e exclusão lógica.|
| API-ADM-09 | Endpoints de turmas, anos letivos e vínculos de matérias.|
| API-ADM-10 | Endpoints de matrícula, renovação e cancelamento.|
| API-ADM-11 | Endpoints administrativos de consulta e manutenção de simulados.|
| API-ADM-12 | Lançamento e consulta de resultados administrativos.|
| API-ADM-13 | Consultas de resultados e relatórios.|
| API-ADM-14 | Upload/entrega de fotos de perfil.|
| API-ADM-15 | Cobertura de integração e segurança dos fluxos administrativos.|

### Tarefas do website administrativo V1

| ID | Tarefa |
| --- | --- |
| WEB-ADM-01 | Integrar autenticação e proteger rotas administrativas.|
| WEB-ADM-02 | Criar telas administrativas de contas.|
| WEB-ADM-03 | Criar telas de matérias, turmas e associações de professores.|
| WEB-ADM-04 | Criar fluxos de matrícula, renovação e cancelamento.|
| WEB-ADM-05 | Criar consulta e edição de simulados.|
| WEB-ADM-06 | Criar lançamento e consulta administrativa de resultados.|
| WEB-ADM-07 | Criar gráficos administrativos após definição das métricas; não inclui dashboard pendente.|
| WEB-ADM-08 | Integrar fotos aos formulários após definir política de arquivos.|
| WEB-ADM-09 | Testar os fluxos administrativos ponta a ponta.|

### Fora da V1

- Aplicativo mobile, endpoints específicos de aluno/professor e fluxo de alteração de notas pelo professor: reservar para subversão posterior, ainda sem escopo.
- Dashboard administrativo: não implementar até aprovação dos indicadores e métricas.
- Infraestrutura e publicação Azure: dependem das decisões de nuvem/operação.
- Importação de Excel, presença e notificações: adiadas para próximas versões.

## 6. Funcionalidades

As funcionalidades abaixo sintetizam o catálogo funcionalidades, não necessariamente as referentes a v1.

### Acesso e conta

| ID | Perfil/canal | Funcionalidade |
| --- | --- | --- |
| AUTH-01 | Admin web; aluno/professor mobile | Login com e-mail e senha; área liberada conforme perfil. Cookie web seguro já escolhido; duração e renovação pendentes.|
| AUTH-02 | Todos os perfis | Solicitar redefinição de senha por e-mail, com resposta genérica e limitação por conta/IP.|
| AUTH-03 | Todos os perfis | Definir nova senha por link de uso único, com tentativa de abrir o app e fallback web; validade de uma hora.|
| AUTH-04 | Contas novas | Ativar conta e definir a própria senha por link de uso único, válido por 72 horas; administrador pode reenviar.|

### Website do administrador

| ID | Área | Funcionalidade |
| --- | --- | --- |
| WEB-ADM-01 | Painel | Área inicial com indicadores sugeridos no Figma.|
| WEB-ADM-02 | Alunos | Listar, filtrar, cadastrar, detalhar, editar, associar a turma, cancelar matrícula e excluir logicamente sob as condições aprovadas; administrar ativação e foto.|
| WEB-ADM-03 | Importação | Importação assistida de Excel.|
| WEB-ADM-04 | Professores | Listar, cadastrar, editar, excluir logicamente e associar vários professores a matérias/turmas sem histórico.|
| WEB-ADM-05 | Administradores | Cadastrar, consultar, editar e excluir logicamente; impedir autoexclusão e remoção do último administrador ativo.|
| WEB-ADM-06 | Matérias | Listar, cadastrar, editar e excluir logicamente; permitir reutilizar nome quando desativada.|
| WEB-ADM-07 | Turmas | Consultar turmas por ano letivo, usando ano atual por padrão e permitindo selecionar anos cadastrados.|
| WEB-ADM-08 | Turmas | Criar/editar série e ano, listar/vincular matérias; no máximo uma turma por série/ano.|
| WEB-ADM-09 | Simulados | Consultar/cadastrar simulados, selecionar matérias da turma, definir tipo, número, data e bimestre; editar até a data do simulado.|
| WEB-ADM-10 | Resultados | Lançar acertos e confirmar ausência explicitamente; permitir alteração de notas sem prazo final.|
| WEB-ADM-11 | Resultados | Consultar resultados por aluno, simulado e matéria.|
| WEB-ADM-12 | Gráficos | Comparar médias e acertos conforme métricas do Figma.|
| WEB-ADM-13 | Presença | Consultar presenças registradas.|

### Aplicativo do aluno

| ID | Funcionalidade |
| --- | --- |
| APP-ALU-01 | Consultar resumo de desempenho e acessar notas/gráficos próprios.|
| APP-ALU-02 | Consultar notas por matéria/simulado, estado, peso, contribuição e média provisória.|
| APP-ALU-03 | Consultar gráfico próprio e comparar média com a turma.|
| APP-ALU-04 | Consultar calendário de simulados com data, número, tipo e peso por matéria.|
| APP-ALU-05 | Consultar matérias da turma atual.|
| APP-ALU-06 | Consultar o próprio perfil sem edição própria aprovada.|

### Aplicativo do professor

| ID | Funcionalidade |
| --- | --- |
| APP-PRO-01 | Consultar turmas do ano atual com matérias associadas.|
| APP-PRO-02 | Comparar médias dos alunos nas matérias lecionadas.|
| APP-PRO-03 | Consultar resultados nas associações do ano atual e alterar notas já lançadas.|
| APP-PRO-04 | Consultar o próprio perfil sem edição própria.|
| APP-PRO-05 | Consultar gráfico compartilhado com administradores por bimestre.|

### Recursos comuns

| Recurso | Funcionalidade |
| --- | --- |
| Fotos | Administrador inclui/edita foto de aluno, professor e administrador via API multipart/form-data.|
| Presença | Identificar aluno e registrar/consultar presença dentro da aplicação.|
| Notificações | Avisos de resultados por push, dentro do app/site e e-mail.|

## 7. Estratégia inicial de testes


- **Testes unitários de regras:** fórmula de nota, pesos, médias provisórias, estados de resultado, validações e regras de matrícula quando forem implementadas.
- **Testes de integração da API:** rotas e autorização contra PostgreSQL de teste; migrations; unicidade, vínculos e operações transacionais de matrícula; respostas RFC 9457 sem detalhes internos.
- **Testes web:** componentes/formulários, filtros, paginação, tratamento de erros do cliente HTTP e bloqueio de rotas sem autorização.
- **Testes de ponta a ponta:** fluxos administrativos completos, como login, cadastro/edição, manutenção de turma/matéria, matrícula, simulado e lançamento/consulta de resultado, conforme cada parte seja concluída.
- **Acessibilidade e usabilidade:** avaliar formulários, navegação por teclado, leitura e responsividade com representantes dos perfis relevantes; critérios quantitativos ainda precisam ser definidos.
- **Validação educacional/operacional:** confirmar com responsáveis e usuários da instituição se as informações e relatórios refletem os processos escolares aprovados. Esta validação é do NotasMAX, não das regras da competição didática.
- **Dados de teste:** usar dados sintéticos, sem dados pessoais reais em desenvolvimento, CI ou demonstrações.

Os testes atuais cobrem apenas as fundações existentes. Critérios verificáveis e automação de cada requisito devem ser adicionados junto às implementações correspondentes. A execução remota da CI da API foi deixada para outra fase e não é afirmada como concluída neste documento.
