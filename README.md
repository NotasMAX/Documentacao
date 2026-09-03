# NotasMax — Sistema de Organização de Notas

**Centro Paula Souza — Faculdade de Tecnologia de Jahu**
Curso de Tecnologia em Desenvolvimento de Software Multiplataforma
Documentação do Projeto Interdisciplinar (PI)

Jahu, SP — 6º semestre/2026

**Autores:** Vinícius Gimenes, Vinícius Nascimento, Lucas Nono, Evelyn Cassinotte e Amauri Barbieri Filho.

---

## Sumário

1. [Descrição da Aplicação Web](#1-descrição-da-aplicação-web)
2. [Objetivos](#2-objetivos)
3. [Documento de Requisitos](#3-documento-de-requisitos)
4. [Casos de Uso](#4-casos-de-uso)
5. [Userflow](#5-userflow)
6. [Personas](#6-personas)
7. [Estudo de Viabilidade](#7-estudo-de-viabilidade)
8. [Modelo de Dados](#8-modelo-de-dados)
9. [Design](#9-design)
10. [Aplicação](#10-aplicação)
11. [Considerações Finais](#11-considerações-finais)
12. [Referências Bibliográficas](#referências-bibliográficas)

---

## 1. Descrição da Aplicação Web

### 1.1 Introdução

Reconhecendo a necessidade de melhorar a organização das notas do colégio, apresentamos o projeto NotasMax. Este software permitirá que administradores possam lançar de forma online as notas dos alunos de todas as turmas, permitindo assim que todos os alunos tenham um acesso melhorado de suas respectivas notas. O NotasMax visa eliminar esta dificuldade atual do colégio.

### 1.2 Métodos Utilizados

O cronograma de desenvolvimento foi registrado com uso do Jira para melhor organização. O modelo escolhido pela equipe foi o de prototipação, por ser considerado mais adequado para o projeto.

O wireframe e o protótipo foram desenvolvidos no Figma. A codificação da aplicação web do 4º semestre foi feita no Visual Studio Code, utilizando:

- **Frontend:** JavaScript, React, Tailwind, HTML, CSS
- **Backend:** JavaScript, Node.js
- **Banco de dados:** MongoDB

Já a aplicação mobile, referente ao 5º semestre, utiliza:

- **Frontend:** .NET MAUI, C#
- **Backend:** JavaScript, Node.js
- **Banco de dados:** MongoDB

**Link do protótipo (Figma):** https://www.figma.com/design/3tUP5eB55kFrgwesGN6qAk/NotasMax

---

## 2. Objetivos

### 2.1 Geral

Organizar as notas de simulados dos alunos para que eles possam acompanhar suas notas mais de perto e facilitar a organização delas por parte dos professores/diretores.

### 2.2 Específicos

- Identificar aplicações semelhantes.
- Levantar requisitos funcionais.

---

## 3. Documento de Requisitos

Um documento de requisitos descreve as funcionalidades, características e restrições que um sistema deve ter para atender às necessidades dos usuários e stakeholders. Serve para orientar os desenvolvedores, designers e demais membros da equipe, oferecendo uma visão detalhada dos requisitos funcionais e não funcionais que moldarão a construção do software.

O documento abrange especificações técnicas, interfaces do usuário e critérios de desempenho, estabelecendo uma base sólida para implementação, teste e validação, assegurando a precisão entre a visão do cliente e a solução final.

### 3.1 Histórias do Usuário

- Como aluno, quero visualizar minhas notas dos simulados realizados para acompanhar meu desempenho ao longo dos bimestres por meio de gráficos.
- Como professor, quero visualizar gráficos de desempenho das turmas vinculadas a mim para acompanhar o progresso coletivo e identificar possíveis dificuldades.
- Como professor, quero visualizar gráficos de desempenho individual de cada aluno para entender o progresso e as necessidades específicas de cada um.
- Como administrador, quero cadastrar alunos, professores, turmas e matérias para manter o sistema atualizado e organizado.
- Como administrador, quero atualizar e modificar registros (notas, cadastros de alunos/professores, simulados, matérias e turmas) para garantir que as informações estejam sempre corretas.
- Como administrador, quero importar planilhas do Excel com as notas dos simulados para agilizar o processo de atualização e exibir automaticamente os resultados aos alunos e professores.

### 3.2 Requisitos Funcionais

#### Requisitos funcionais do 4º semestre (aplicação web)

| # | Requisito | Descrição |
|---|---|---|
| RF 1 | Cadastrar Aluno | Admin: cadastro de alunos. Campos: Nome Completo, E-mail institucional, telefone para contato e telefone do responsável. |
| RF 2 | Cadastrar Professor | Admin: cadastro de professor. Campos: Nome Completo, E-mail institucional, Turmas, telefone para contato. |
| RF 3 | Cadastrar Admin *(implementação futura)* | Cadastro de administradores. Campos: Nome Completo, E-mail institucional. |
| RF 4 | Cadastrar Turmas | Admin: cadastro de turmas. Campos: Ano, Alunos, Professores, Matérias. |
| RF 5 | Cadastrar Matérias | Admin: cadastro de matérias. Campo: Descritivo. |
| RF 6 | Importar Planilhas *(implementação futura)* | Admin: importação de planilhas do Excel. |
| RF 7 | Registrar Notas | Admin: lançamento de notas por aluno. Campos: Professor, Simulado, Matéria, Número de acertos, Número de questões e peso. |
| RF 8 | Exibir turmas | Admin: exibição das turmas cadastradas. |
| RF 9 | Exibir Alunos | Admin: exibição dos alunos cadastrados. |
| RF 10 | Exibir Professores | Admin: exibição dos professores cadastrados. |
| RF 11 | Alterar Registros | Admin: atualizações/modificações de registros. RF11.1 modificar notas dentro do prazo; RF11.2 editar cadastro de aluno; RF11.3 editar cadastro de professor. |
| RF 12 | Acessar notas — Admin | Admin: visualizar todas as notas registradas no sistema. |
| RF 13 | Exibir Gráficos do aluno | Aluno e Admin: gráfico com diferenças de desempenho do aluno por bimestre. |
| RF 14 | Exibir Gráficos bimestrais — Admin | Gráfico comparando a média das classes ao longo dos bimestres. |
| RF 15 | Exibir Gráficos de comparação — Admin | Gráfico comparando a média de um aluno com a da classe. |
| RF 16 | Exibir Gráficos por matéria — Admin | Gráfico comparando os acertos da turma em uma matéria. |
| RF 17 | Redefinir senha | Aluno, Professor e Admin: redefinição de senha. RF 17.1: Admin pode alterar a senha de alunos. |
| RF 18 | Cadastrar Simulados | Admin: cadastro de simulados. Campos: Tipo (objetivo ou dissertativo), Data, Numeração, Turma, Matérias, Professores e Bimestre. |
| RF 19 | Identificar os tipos de usuário | O sistema identifica Administrador, Aluno e Professor. |

#### Requisitos funcionais do 5º semestre (aplicação mobile)

| # | Requisito | Descrição |
|---|---|---|
| RF 20 | Alterar Notas — Professor | Professor pode editar notas dos alunos das matérias sob sua responsabilidade no ano atual. |
| RF 21 | Logar na aplicação | Professor e Aluno podem fazer login na aplicação mobile. |
| RF 22 | Acessar notas — Aluno | Aluno acessa notas de simulados filtradas para sua turma no ano atual. |
| RF 23 | Acessar notas — Professor | Professor visualiza notas dos alunos das turmas sob sua responsabilidade. |
| RF 24 | Exibir Gráficos de comparação — Aluno | Gráfico comparando a média do aluno com a da classe. |
| RF 25 | Exibir Gráficos de comparação entre alunos — Professor | Gráfico comparando médias dos alunos de uma turma em uma disciplina lecionada pelo professor. |
| RF 26 | Exibir Calendário — Aluno | Calendário com datas de simulados, filtrado pela turma do aluno. |
| RF 27 | Recuperar senha | Recuperação via código enviado ao e-mail institucional. |
| RF 28 | Exibir Matérias — Aluno | Exibe as matérias lecionadas na turma do aluno. |
| RF 29 | Exibir Perfil | Exibe o perfil do usuário logado. |
| RF 30 | Exibir Turmas — Professor | Exibe as turmas em que o professor tem disciplinas cadastradas no ano atual. |

### 3.3 Requisitos Não Funcionais

- **RNF 1** — A aplicação deve ser de fácil navegação.
- **RNF 2** — A aplicação deve ser segura.
- **RNF 3** — A aplicação deve estar sempre disponível.

### 3.4 Regras de Negócio

Para a elaboração do modelo de negócio, foi utilizado o Modelo de Negócio Canvas, permitindo planejar de forma clara e visual o público-alvo, a proposta de valor, os canais de distribuição, a estrutura de custos e outros elementos essenciais.

**Modelo de Negócio Canvas (resumo):**

- **Parcerias principais:** Colégio Max
- **Atividades principais:** Desenvolvimento da plataforma web; Registro de notas; Geração de relatórios e dashboards de desempenho
- **Recursos principais:** Equipe de Desenvolvimento; Internet; Infraestrutura
- **Proposta de valor:** Gestão de simulados escolares; Acompanhamento do desempenho dos alunos; Relatórios automáticos e visuais de desempenho individual e coletivo
- **Relacionamento com clientes:** Contato direto com professores em sala de aula
- **Segmento de clientes:** Colégio MAX; Professores; Alunos; Administradores
- **Canais:** Plataforma Web; Aplicação Mobile
- **Estrutura de custo:** Desenvolvimento e manutenção da plataforma; Hospedagem; Infraestrutura; Tempo
- **Fontes de receita:** Nenhuma prevista (natureza educacional/acadêmica do projeto)

#### 3.4.1 O que será elaborado?

**Proposta de valor**: Será desenvolvida uma plataforma web educacional denominada Notas Max, voltada à gestão de notas de simulados. O sistema permitirá que administradores cadastrem turmas, matérias, simulados e notas, com o cálculo automático das notas bimestrais. A proposta central é digitalizar, automatizar e otimizar o acompanhamento do desempenho escolar, oferecendo mais transparência e praticidade para alunos, professores e gestores. 

#### 3.4.2 Como será elaborado?

**Atividades principais:** As principais atividades envolvem: Desenvolvimento da plataforma web; Registro e gerenciamento de notas; Geração de relatórios e dashboards de desempenho. 

**Parcerias principais:** A parceria fundamental será com o Colégio MAX, que fornecerá feedback e orientações para a evolução da plataforma, garantindo sua usabilidade e adequação pedagógica. 

**Recursos principais:** Equipe de Desenvolvimento; Internet; Infraestrutura.

#### 3.4.3 Para quem será elaborado?

**Relação com o cliente:** O relacionamento será mantido por meio de contato direto dos alunos com os professores no ambiente da escola. 

**Segmento de clientes:** O sistema é voltado para a instituição de ensino: Colégio Max, no qual abrange administradores, professores e alunos. 

**Canais:** Os canais de acesso serão o website para os administradores e a aplicação mobile para os alunos e professores.

#### 3.4.4 Quanto vai custar?

**Estrutura de custos:** A estrutura de custo contempla o desenvolvimento da plataforma, hospedagem e infraestrutura, além do tempo dedicado pela equipe de desenvolvimento.

**Fontes de renda:** Por se tratar de um projeto educacional e acadêmico, não há previsão de fontes de receita. 

### 3.5 Diagrama BPMN

O Diagrama BPMN ilustra a implementação do sistema baseado no modelo de negócio do projeto. O diagrama abaixo mostra o fluxo mais importante, que é a criação de simulados e como os usuários acessam e interagem com eles no sistema. 

> *Ver Figura 2 – BPMN no documento original em PDF.*

---

## 4. Casos de Uso

O diagrama de casos de uso (NotaMax) mapeia as interações dos atores **Aluno**, **Professor** e **Admin** com o sistema — login, visualização de notas e gráficos individuais/de turma, notificações, busca, importação de planilhas, e cadastro/edição de simulados, alunos, professores, turmas e matérias.

> *Ver Figura 3 – Casos de Uso no documento original em PDF.*

### 4.1 Casos de Uso Resumidos

**Realizar cadastro de alunos**
O admin acessa o formulário de cadastro de novo aluno (Nome completo, E-mail institucional, Telefone para contato, Telefone do responsável). Se válido, o sistema cadastra o aluno com sucesso.

**Realizar login**
O usuário informa e-mail institucional e senha; se corretos, é redirecionado à tela inicial.

**Cadastrar simulado**
O admin insere Tipo (objetivo ou dissertativo), Data, Numeração, Bimestre, Turma, Professores e Matérias; se válido, o simulado é cadastrado.

**Cadastrar professor**
O admin informa Nome completo, E-mail institucional e Telefone para contato; se válido, o professor é cadastrado.

**Editar aluno**
O admin altera Nome completo, E-mail institucional, Turma, Telefone para contato e Telefone do responsável; se válido, os dados são atualizados.

**Acessar notas – Aluno**
O aluno acessa os simulados já realizados com notas lançadas, visualizando desempenho por matéria e por professor.

**Visualizar gráficos de desempenho bimestral – Aluno**
O aluno acessa os gráficos das notas lançadas, visualizando seu desempenho bimestral de forma gráfica.

### 4.2 Casos de Uso Detalhados

#### 4.2.1 Cadastrar Aluno

**Ator principal:** Administrador

**Interesses e interessados:**
- **Administrador:** Deseja cadastrar novos alunos no sistema Notas Max, garantindo que todos os estudantes possuam acesso individual à plataforma. 
- **Aluno:** Deseja ter um cadastro ativo no sistema para acessar suas notas, gráficos de desempenho e notificações personalizadas. 

**Pré-condições:**
- Ator autenticado como Administrador válido.
- Sistema em funcionamento e com acesso à internet para envio de e-mails.

**Pós-condições:**
- Aluno cadastrado com sucesso na base de dados.
- Sistema gera senha aleatória temporária.
- Essa senha é enviada para o e-mail institucional do aluno, permitindo seu primeiro acesso à plataforma. 

**Cenário de sucesso principal:**
1. O administrador acessa o sistema Notas Max. 
2. O administrador seleciona a opção “Cadastrar Aluno” no painel administrativo. 
2. O sistema exibe o formulário de cadastro de aluno. 
2. O administrador preenche os seguintes campos obrigatórios: 
    - Nome Completo 
    - E-mail institucional 
    - Telefone para contato 
    - Telefone do responsável 
2.  O administrador confirma o envio do formulário. 
2. O sistema valida os dados inseridos e verifica se o e-mail institucional não está duplicado. 
2. O sistema gera automaticamente uma senha aleatória e a associa ao novo cadastro. 
2. O sistema envia um e-mail automático ao endereço institucional do aluno contendo: 
    - Uma mensagem de boas-vindas; 
    - O login (e-mail institucional); 
    - A senha temporária e instruções para redefinição no primeiro acesso. 
2. O sistema exibe a mensagem: “Aluno cadastrado com sucesso. As credenciais foram enviadas por e-mail.” 
2. O novo aluno é adicionado à base de dados e pode ser posteriormente vinculado a uma turma. 

**Fluxos alternativos:**
- **1A – Campos incorretos:** "Preencha todos os campos corretamente para continuar." — retorna ao formulário.
- **2A – E-mail já cadastrado:** "E-mail já cadastrado. Verifique as informações."
- **3A – Falha no envio do e-mail:** aluno é cadastrado, mas é exibido alerta para reenvio.
- **4A – Falha de conexão/servidor:** "Erro ao cadastrar aluno. Tente novamente mais tarde." — cadastro não concluído.

**Requisitos relacionados:** RF 1 – Cadastrar Aluno; RF 19 – Logs de modificação.

#### 4.2.2 Registrar Notas

**Ator principal:** Administrador

**Interesses e interessados:**
- Administrador: lançar notas de alunos por simulado, garantindo registro correto.
- Aluno: receber notas corretamente e acompanhar desempenho.
- Professor: consultar e validar notas lançadas em suas turmas.

**Pré-condições:**
- Administrador autenticado.
- Simulado e alunos previamente cadastrados.
- Matérias e professores responsáveis definidos.

**Pós-condições:**
- Notas registradas e associadas ao simulado, matéria, professor e peso.
- Alunos notificados automaticamente sobre a nova nota.

**Cenário de sucesso principal:**
1. O administrador acessa o sistema Notas Max. 
1. Seleciona a opção “Registrar Notas” no painel administrativo. 
1. Escolhe a turma, o simulado e o aluno para lançar a nota. 
1. Preenche os seguintes campos: 
    - Professor responsável 
    - Matéria 
    - Número de acertos 
    - Número total de questões 
    - Peso da avaliação 
1. O sistema calcula automaticamente a nota final do aluno com base nos dados inseridos. 
1. O administrador confirma o lançamento da nota. 
1. O sistema salva a informação no banco de dados. 
1. O aluno recebe uma notificação automática informando sobre a nova nota registrada. 
1. O administrador recebe uma mensagem de confirmação: “Nota registrada com sucesso.” 

**Fluxos alternativos:**
- **1A – Campos obrigatórios não preenchidos:** "Preencha todos os campos obrigatórios antes de continuar."
- **2A – Aluno ou simulado não encontrado:** "Registro não encontrado. Verifique os dados e tente novamente."
- **3A – Erro no cálculo da nota:** "Erro ao calcular a nota. Revise as informações e tente novamente."
- **4A – Falha na notificação:** nota é registrada, mas exibe aviso para reenviar a notificação.

**Requisitos relacionados:** RF 7 – Registrar Notas; RF 20 – Enviar Notificação; RF 12 – Acessar notas; RF 19 – Logs de modificação.

---

## 5. Userflow

O userflow ilustra as possíveis rotas do usuário ao usar o sistema — desde a abertura do sistema e login, passando por navegação (simulados recentes/anteriores, gráficos, calendário, perfil) até logout — e também os cenários de dificuldade, como falha de login e recuperação de senha.

> *Ver Figura 4 – Userflow no documento original em PDF.*

---

## 6. Personas

### 6.1 Persona 1 — "Diretor Carlos"

**Perfil do Gestor Escolar**
- **Nome:** Carlos Mendonça
- **Idade:** 44 anos
- **Cargo:** Diretor pedagógico de escola privada de médio porte
- **Formação:** Pedagogia + Pós-graduação em Gestão Escolar
- **Tecnologia:** Usuário intermediário — utiliza computador diariamente, busca soluções que simplificam a rotina

**Objetivos**
- Ter visão completa e atualizada do desempenho dos alunos por turma, disciplina e bimestre.
- Reduzir retrabalho com planilhas, conferências manuais e consolidação de notas.
- Padronizar o registro de notas entre professores.
- Facilitar comunicações com pais, alunos e corpo docente usando dados claros.
- Garantir justiça, transparência e coerência no processo avaliativo.

**Dores e frustrações**
- Perder horas consolidando notas em formatos diferentes (PDF, Excel, papel etc.).
- Riscos de erros humanos nos cálculos das médias.
- Falta de histórico integrado da evolução dos alunos.
- Dificuldade em identificar rapidamente quedas de desempenho por turma.
- Pressão de pais e mantenedores por dados atualizados.

**Expectativas e necessidades**
- Sistema simples, visual e confiável.
- Dashboard com visão geral e detalhamentos por aluno, turma e simulado.
- Cálculo automático de médias, ranking, curva de desempenho e estatísticas.
- Relatórios exportáveis para reuniões e conselhos de classe.
- Informações para intervenções pedagógicas mais assertivas.

**Comportamento digital**
- Consulta o dashboard diariamente pela manhã.
- Analisa gráficos e alertas antes de reuniões com coordenadores.
- Usa relatórios para embasar conversas com pais e docentes.
- Valoriza interfaces com dados claros e objetivos.

**Mensagem-chave:** "Tenha controle total sobre o desempenho da sua escola com dados claros e automáticos – menos retrabalho, mais decisões pedagógicas assertivas."

### 6.2 Persona 2 — "Aluno Lucas"

**Perfil do Estudante**
- **Nome:** Lucas Andrade
- **Idade:** 16 anos
- **Série:** 2º ano do Ensino Médio
- **Objetivo acadêmico:** Passar em vestibulares concorridos
- **Tecnologia:** Avançado — usa celular para tudo: estudos, rotina, comunicação

**Objetivos**
- Acompanhar sua evolução ao longo dos bimestres.
- Entender em quais habilidades está indo bem e onde precisa melhorar.
- Comparar simulados anteriores para ajustar sua estratégia de estudo.
- Receber notas rápidas, sem precisar esperar semanas.
- Diminuir ansiedade sobre resultados.

**Dores e frustrações**
- Não entender de onde vem sua média final.
- Resultados demorados e pouco claros.
- Ter de pedir nota ao professor quando não é disponibilizada.
- Não saber exatamente o que estudar para melhorar o desempenho.
- Falta de previsibilidade (ex.: "qual nota preciso para fechar o bimestre?").

**Expectativas e necessidades**
- Acesso rápido às notas do simulado (objetivo e dissertativo).
- Visualização clara do desempenho por matéria e habilidade.
- Gráficos de evolução ao longo dos meses.
- Sistema transparente, humano e com linguagem acessível.
- Cálculo automático do que falta para alcançar uma meta (ex.: média 7).

**Comportamento digital**
- Acessa pelo celular no dia a dia.
- Confere notas logo após a divulgação.
- Usa dados para se organizar para próximas provas e simulados.
- Compartilha resultados com amigos e familiares.

**Mensagem-chave:** "Veja sua evolução de forma clara e entenda exatamente o que fazer para melhorar. Suas notas sempre atualizadas e acessíveis."

### 6.3 Persona 3 — "Professor Henrique"

**Perfil do Docente**
- **Nome:** Henrique Duarte
- **Idade:** 39 anos
- **Disciplina:** Matemática
- **Formação:** Licenciatura em Matemática + Pós-graduação em Ensino da Matemática
- **Tecnologia:** Usa ferramentas digitais com naturalidade, mas prefere interfaces simples e diretas

**Objetivos**
- Visualizar rapidamente a quantidade de acertos dos alunos nos simulados ao longo do bimestre.
- Acompanhar a evolução geral da turma.
- Identificar tendências de melhora ou queda no desempenho.
- Planejar aulas de revisão e atividades extras.
- Facilitar diálogos com coordenação, direção, pais e alunos usando dados consolidados.

**Dores e frustrações**
- Não ter local centralizado para ver acertos dos alunos por simulado.
- Precisar recorrer à direção para consolidar dados.
- Dificuldade em entender o panorama da turma sem relatórios claros.
- Alunos pedirem notas antes que ele tenha acesso atualizado.
- Falta de tendências visuais sobre a evolução da turma.

**Expectativas e necessidades**
- Sistema que apresente: quantidade de acertos por aluno, médias calculadas, evolução por simulado, comparação entre bimestres.
- Interface objetiva, sem excesso de informações.
- Gráficos simples mostrando progresso da turma.
- Relatórios que auxiliem reuniões pedagógicas.
- Dados que ajudem a planejar reforço, mesmo sem detalhamento por questão.

**Comportamento digital**
- Consulta o sistema semanalmente, especialmente após simulados.
- Usa dados para orientar atendimentos individuais com alunos.
- Aprecia gráficos e números claros, sem necessidade de granularidade avançada.
- Usa computador e celular, dependendo da correria do dia.

**Mensagem-chave:** "Tenha uma visão clara do desempenho da sua turma — acompanhe acertos, médias e evolução de forma simples e rápida."

---

## 7. Estudo de Viabilidade

### 7.1 Viabilidade Técnica

O sistema é desenvolvido como plataforma web, garantindo acesso remoto, facilidade de uso e integração com diferentes dispositivos. Principais tecnologias: HTML, TailwindCSS, JavaScript, Node.js e MongoDB — estrutura moderna, segura e escalável.

A equipe ampliou seus conhecimentos técnicos ao longo do desenvolvimento, permitindo implementação eficiente e resolução colaborativa de desafios.

### 7.2 Viabilidade Econômica

Os principais custos concentram-se na hospedagem em nuvem. As tecnologias (Node.js, React, MongoDB) são gratuitas, reduzindo o investimento inicial e de manutenção.

Não há despesas financeiras diretas com mão de obra (desenvolvimento pelos próprios alunos), mas o tempo dedicado configura um custo indireto relevante. Estimativa: ~4 horas semanais por integrante ao longo de um semestre (~80 horas por integrante). A um valor médio de mercado de R$ 25,00/hora (nível júnior), o custo individual seria de ~R$ 2.000,00; para uma equipe de cinco desenvolvedores, o custo total estimado é de ~R$ 10.000,00.

Não é necessário investimento em mão de obra externa. Os benefícios (transparência no acompanhamento de notas, otimização do tempo da equipe pedagógica, melhoria na motivação dos alunos) justificam o investimento frente ao baixo custo de implementação e manutenção.

### 7.3 Viabilidade de Mercado

Há demanda constante por soluções que facilitem o monitoramento das notas bimestrais, tanto pela direção escolar quanto pelos alunos, devido às limitações dos métodos atuais.

O sistema se diferencia por ser projetado exclusivamente para as necessidades do Colégio Max, oferecendo solução personalizada em vez de uma plataforma genérica.

### 7.4 Viabilidade Legal

O projeto está em conformidade com as normas vigentes, especialmente a Lei Geral de Proteção de Dados (LGPD). A plataforma implementará políticas claras de privacidade, consentimento e armazenamento seguro de dados, assegurando confidencialidade das informações pessoais e acadêmicas, além de medidas de proteção da propriedade intelectual do sistema.

### 7.5 Viabilidade Organizacional

A equipe possui capacidade técnica e organizacional, com divisão clara de responsabilidades e flexibilidade para se adaptar a desafios. O baixo custo operacional (limitado basicamente à hospedagem em nuvem) torna a manutenção financeiramente sustentável a longo prazo.

### 7.6 Análise SWOT (FOFA)

| Forças | Oportunidades |
|---|---|
| Foco em necessidade real | Expansão para outras escolas |
| Automatização de cálculos | Integração com sistemas educacionais |
| Visualização por gráficos | Avanços em tecnologias de análise de dados (IA) |
| Sistema centralizado | |
| Baixo custo | |

| Fraquezas | Ameaças |
|---|---|
| Dependência do Colégio Max | Resistência de usuários |
| Funcionalidades incompletas | Exigências da LGPD |
| Poucos testes em escala | Dependência de internet |
| Necessidade de treinamento | Riscos de segurança de dados |

---

## 8. Modelo de Dados

Optou-se pelo uso de um banco de dados não relacional (**MongoDB**) por garantir persistência e integridade dos dados em todas as fases da aplicação. O uso do MongoDB permite estruturar e relacionar as informações de forma consistente e bem-organizada, tornando o sistema mais seguro e confiável.

### 8.1 Migração Planejada para Banco de Dados Relacional
 
Atualmente está em planejamento a migração do banco de dados do NotasMax, que hoje é não relacional (MongoDB), para um banco de dados relacional. A proposta de troca, com o levantamento do novo modelo de dados, está disponível em:
 
**Proposta de migração:** https://github.com/NotasMAX/Documentacao/tree/main/modelo-dados

---

## 9. Design

### 9.1 Paleta de Cores

A paleta foi definida com base nas cores do logotipo do Colégio Max, garantindo harmonia visual e identidade com a marca institucional:

`#FFB90D` `#FFCC00` `#043666` `#1C86EB`
`#FFCF58` `#FFE16B` `#4076A9` `#50AAFF`
`#FFE29A` `#FFEDA6` `#84BCF2` `#96CCFF`

### 9.2 Tipografia

Duas fontes: **Inter** (presente em toda a aplicação) e **Space Grotesk** (presente apenas na logo do site).

### 9.3 Logo

A logo usada no site é a logo do próprio colégio (Colégio Max — Beny Macena).

### 9.4 Wireframe

Telas prototipadas incluem, entre outras:
- Aplicação Web – Tela inicial
- Aplicação Web – Tela Editar Turma
- Aplicação Web – Tela de Notas por Disciplina de um Aluno
- Aplicação Mobile – Tela Inicial – Aluno
- Aplicação Mobile – Tela Inicial – Professor

> *Ver Figuras 12–17 no documento original em PDF.*

---

## 10. Aplicação

Telas elaboradas no sistema, incluindo:
- Aplicação Web – Tela de Listagem de Matérias
- Aplicação Web – Tela de exibição de turmas
- Aplicação Web – Tela de edição de turmas
- Aplicação Mobile – Tela de Login
- Aplicação Mobile – Turmas do Professor
- Aplicação Mobile – Desempenho de uma Turma – Professor

> *Ver Figuras 18–22 no documento original em PDF.*

---

## 11. Considerações Finais

As principais dificuldades deste semestre estiveram relacionadas ao desenvolvimento de um aplicativo mobile, já que em semestres anteriores o foco era web — isso demandou pesquisa e esforço para adaptação a esse novo padrão.

Este semestre também marcou o primeiro contato da equipe com o **.NET MAUI**, embora já houvesse experiência prévia com Visual Studio e C# em um projeto de semestre anterior, o que facilitou a adaptação.

No semestre anterior, a equipe criou um escopo grande e não conseguiu entregar todas as funcionalidades planejadas — o que serviu de aprendizado para definir, neste semestre, um escopo menor e mais realista, permitindo um trabalho mais adequado. Esse tipo de experiência é o que torna o PI importante: aprender na prática o que funciona e o que deve ser reavaliado, evoluindo a cada semestre.

---

## Referências Bibliográficas

- INTER. Fonte utilizada no projeto. Disponível em: https://fonts.google.com/specimen/Inter. Acesso em: 20 de novembro de 2025.
- SPACE GROTESK. Fonte utilizada no projeto. Disponível em: https://fonts.google.com/specimen/Space+Grotesk. Acesso em: 20 de novembro de 2025.
- OPENAI. ChatGPT. Utilizado para geração de personas. Disponível em: https://chat.openai.com/. Acesso em: 20 de novembro de 2025.