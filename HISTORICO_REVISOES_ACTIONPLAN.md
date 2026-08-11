# Histórico de Revisões — ActionPlan

Sistema de gestão de planos de ação, ações corretivas e Curso de 1 Tema para ambiente automotivo/industrial. Aplicação SPA em arquivo único (HTML/JS), sem build, hospedada no GitHub Pages, com Firebase Firestore + Storage como backend.

> Reconstruído a partir do histórico de conversas do projeto. Versões e datas anteriores à v19.0 são aproximadas, com base no conteúdo das sessões — os detalhes técnicos estão corretos, a numeração exata de sub-versões desse período pode não ser exaustiva.

---

## Fase inicial (v5 – v8) — Fundamentos

- Unificação do fluxo de edição: o lápis na Listagem passa a abrir o formulário de cadastro pré-preenchido, em vez de um modal separado.
- Remoção da antiga página "Gestão de Problemas" (funcionalidade absorvida pela Listagem + Novo Cadastro).
- Numeração sequencial persistente da ação (`numeroAcao`), exibida na Listagem.
- Correção de problemas sem nenhuma ação corretiva ficando invisíveis na Listagem (linha placeholder com editar/excluir).
- Suporte a colar imagem (Ctrl+V) na evidência.
- Colunas calculadas "Abertura" e "Idade" na Listagem.
- Campo Responsável do Follow-up convertido de texto livre para lista suspensa.
- Página "Análise" com o "Resumo PDM" (bloco de texto em fonte monoespaçada, ordenado por Vencidas → Vencem hoje → A vencer).
- Coluna "Último FUP" (data do follow-up mais recente, com fallback pra data de abertura).
- Colunas ordenáveis por clique no cabeçalho (Nº Ação, Ação Corretiva, Abertura, Idade, Status).
- Correção de filtro de responsáveis ativos (`r.ativo !== false`, incluindo registros legados sem o campo).

## v9 – v11 — Evolução contínua

- Ajustes incrementais de UI e correções de bugs pontuais no fluxo de cadastro e listagem, preparando o terreno para os módulos maiores das versões seguintes.

## v12 — Controle de Presença e Taxa de Aderência

- Novo módulo **Controle de Presença**: filtro por data/plano, tabela de responsáveis com checkboxes de Presente/Necessário, persistência com ID estruturado (`YYYY-MM-DD` ou `YYYY-MM-DD_planoId`) evitando duplicidade por dia.
- Card **Taxa de Aderência** no Dashboard, com semáforo (🟢 ≥90% · 🟡 75–89% · 🔴 <75%).
- Coleção `presencas` incluída no Backup/Restauração.
- Separadores visuais por categoria no Resumo PDM (Vencidas / Vencem Hoje / A Vencer).
- Histórico de presenças editável, indicador de quantas reuniões foram computadas.

## v13 — Estabilização e automações de Presença

- Correção de bug fatal em `renderListagem()`.
- Botões ativo/inativo dinâmicos nas 4 telas de cadastro (Clientes, Responsáveis, Planos, Sub-Tópicos).
- Automação de UX: marcar Presente auto-marca e desabilita o campo Necessário.
- Persistência dos filtros da Listagem via `localStorage`.

## v15 — Gráficos empilhados por Plano

- Os três gráficos "por Responsável" (Pendências, Vencidas, Fechadas) passam a ser gráficos de barra empilhada, agrupados por Plano/Cliente, com paleta de cores dinâmica.

## v16 — Segurança e Autenticação

- Sanitização de dados do Firestore antes de qualquer `innerHTML` (mitigação de XSS), cobrindo os 5 caracteres perigosos.
- Ofuscação das credenciais do Firebase (Base64).
- **Login obrigatório** via Firebase Authentication (e-mail/senha), com tela de login cheia, `onAuthStateChanged` controlando o boot do app, e botão de Sair.
- Regras de segurança do Firestore exigindo `request.auth != null` em toda leitura/escrita.

## Curso de 1 Tema (GP12) e Reincidências

- Criação automática de **Curso de 1 Tema** para problemas de planos GP12, copiando sub-tópico, cliente, evidência e demais dados do problema no momento da criação.
- Suporte a **reincidências**: campos de Operador, Quantidade, Data de Produção e Turno (T1/T2/T3) por ocorrência, cada uma gerando seu próprio Curso de 1 Tema.
- Checkpoints T1/T2/T3 com finalização automática do curso quando os três são marcados.

---

# v19.0 — Autenticação completa, níveis de acesso e permissões

- Perfil de usuário (`_userProfile`) carregado do Firestore após login, com 5 níveis de acesso (1 = admin completo, 5 = somente consulta).
- Matriz de permissões (`podeVer()`) aplicada em toda a UI — cadastro, exclusão, follow-up, configurações.
- Painel de **Gestão de Usuários** (criar/editar/resetar senha) restrito a nível 1, acessado via Shift+clique no botão de tema (removendo os antigos `prompt()` de senha fixa).
- Histórico de alterações por Shift+clique no lápis da Listagem.
- Checagem otimista de edição simultânea (`updated_at`/`updated_by`).
- Paginação da Listagem (50/página, "Carregar mais").
- Timer de inatividade (30 min) com logout automático.

# v19.1 — Firebase Storage e migração de fotos

- Fotos e PDFs deixam de ser gravados como base64 no Firestore — passam a subir para o **Firebase Storage**, com upload/exclusão em cascata nos 4 pontos de evidência do sistema.
- Rotina de **migração** de fotos já existentes (base64 → Storage), reentrante e segura (desliga os listeners em tempo real durante a operação para evitar sobrecarga de escrita).
- Backup em 2 modos (com/sem fotos).
- Template de Excel do Curso de 1 Tema armazenado no Storage, compartilhado entre todos os usuários.
- Renomear um Responsável passa a propagar automaticamente para todo o histórico de ações (`_substituirResponsavelEmAcoes`).

# v19.3 — Importação Fast Response

- Novo fluxo de importação em massa (Excel) para o plano "Fast Response": cada importação apaga e recria os problemas desse plano a partir da planilha, com tela de revisão prévia (responsáveis sem correspondência, sub-tópicos novos) antes de confirmar.
- Correlação de responsáveis com sugestão por similaridade de nome.

# v19.4 — Visualização Pública (link direto sem login)

- Login anônimo via Firebase Auth permite abrir um link direto (`#acao=...`) e visualizar uma ação específica **sem precisar de conta** — layout com foto, dados do problema e follow-ups, botão para fazer login e editar.
- Regras do Firestore/Storage ajustadas para diferenciar `get` (documento específico) de `list` (consulta à coleção inteira), liberando o primeiro para visitantes anônimos sem expor a base toda.

# v19.5 — Índice de consulta por Número da Ação

- Nova coleção `indice_numeroacao`, permitindo buscar o prazo e a data de fechamento de uma ação pelo número, via planilha Excel (macro VBA), sem precisar de login — só o número da ação.

# v19.6 – v19.9 — Notas, aviso de dados não salvos e correções de dados

- Campo **Notas/Observações** (rich text) no cadastro do problema, com colar de imagens direto no texto (upload automático pro Storage).
- Aviso ao tentar sair de um cadastro com alterações não salvas.
- Diversas correções de UI: card do Dashboard "Sem Prazo" contando errado, alinhamento de botões, barra de rolagem horizontal da Listagem.

# v19.10 – v19.13 — Unificação dos filtros do Dashboard

- Filtros de Semana, Top N e Peças/Ocorrências unificados entre os gráficos de Defeitos por Turno/Operador/Sub-tópico (antes eram independentes por gráfico).
- Diversas idas e vindas na correção do "gráfico aparecendo atrás do filtro" ao rolar a tela — resolvida definitivamente na v19.14, tirando a barra de filtro do fluxo de rolagem por completo (elemento fixo próprio, no mesmo nível do topbar, abandonando de vez `position:sticky`).

# v19.14 — Card de Curso 1 Tema pendente + Data Efetiva automática

- Novo card no Dashboard com a contagem de Cursos de 1 Tema pendentes.
- Data Efetiva de uma ação passa a ser preenchida automaticamente com a data do dia ao enviar a evidência de fechamento.

# v19.16 — Aprendizado de turno por operador

- O sistema passa a "aprender" o turno de cada operador: uma vez marcado manualmente, o código do operador sozinho já sugere o turno da próxima vez.
- Importação inicial em lote via planilha (Configurações).

# v19.17 – v19.19 — Ajustes finos e migração de fotos esquecidas

- Suporte a múltiplos operadores no mesmo campo (`/` ou `;`), com divisão de quantidade por distribuição de resto (não duplicação).
- Descoberta e correção de uma lacuna real na migração de fotos: `cursos1tema[].evidencia_problema` (cópia de reserva da foto do problema original) nunca era migrada — só a evidência própria do curso (`evidenciaCursoUrl`) era coberta.
- Rastreamento da troca de evidência do problema no histórico de alterações.

# v19.20 – v19.22 — Refinamentos de Configurações e Curso 1 Tema

- Layout de Configurações reorganizado (6 painéis numa única grade).
- Ressincronização automática do Curso de 1 Tema quando o Sub-tópico do problema é corrigido (sem perder progresso já feito).
- Planilha de turnos por operador passa a aceitar "D" (demitido) e "?" (não identificado), além de uma coluna de nome.
- Comentário automático "245-Nome - T1" adicionado às Notas do problema quando há operador identificado, e no *mouse-over* do gráfico "Defeitos por Operador".

# v19.23 – v19.27 — Código do Sub-tópico e ajustes de Listagem

- Nova coluna "Emissão" no Curso de 1 Tema — descoberta de que já existia `criado_em` desde sempre (equivalente ao campo `data_emissao` criado por engano nessa faixa de versões, removido em seguida).
- Novo campo **Código(s) do Subitem** no cadastro de Sub-Tópico (múltiplos códigos separados por `/`), com busca integrada na Listagem Geral (`#codigo` ou o próprio número) e coluna própria (depois movida só para a tela de cadastro de Sub-tópicos, com quebra de linha forçada a cada `/`).
- Scripts de exportação/importação em lote dos códigos de sub-tópico (planilha).
- Coluna "Abertura" renomeada para "Ocorrência" (Listagem e Curso 1 Tema).
- Coluna calculada "Idade" corrigida para nunca mostrar valor negativo.
- Ordenação por clique no cabeçalho da tabela de Sub-Tópicos.

# v19.31 – v19.34 — Modo Auditoria

- Novo **Modo Auditoria**: checkbox em Configurações, sincronizado globalmente via Firestore (afeta todos os usuários vendo o app no momento, não só quem ativou) — oculta todas as ações vencidas da Listagem e do Dashboard.
- Indicador visual no topbar (ícone 🕵️, clicável para desativar direto, restrito a quem tem permissão).
- Painel "Atividade na Semana" oculto por completo enquanto o modo está ativo (evita vazar informação de vencidas por ali).

# v19.35 – v19.36 — Relatório PDF do Plano de Ações

- Novo espaço em Configurações para upload do logo da empresa.
- Botão dedicado na Listagem (🖨️, ao lado do Excel) que abre um modal de opções (Plano, Sub-Tópico opcional, incluir fotos, incluir follow-ups) e gera uma tela formatada — pronta para impressão/"Salvar como PDF" pelo navegador, replicando um modelo de referência fornecido pelo usuário (logo, título, cabeçalho do Plano/Cliente, tabela com Sub-Tópico e Problema mesclados verticalmente por grupo).
- Correção de causa raiz real: `html`, `body`, `#app`, `#main` e `#page` tinham `overflow:hidden`/altura travada em `100vh`, impedindo a paginação da impressão além da primeira página — corrigido liberando `overflow`/`height` de toda a cadeia de containers durante a impressão.

# v19.37 – v19.39 — Evidência do problema: de obrigatória a alerta, e bug real de exclusão

- Campo de evidência do problema passou por 3 fases: obrigatório (bloqueava salvar) → alerta de confirmação (pergunta mas não bloqueia) → indicador "(recomendado)" removido do rótulo.
- Shift+clique (nível 1) no botão de visualizar evidência permite removê-la (ex: enviada por engano).
- **Bug real descoberto e corrigido**: a exclusão de evidência definia o campo como `null`, mas a lógica de salvamento testava `!= null` para decidir se devia gravar a mudança — como `null != null` é falso, a exclusão nunca persistia de verdade (o arquivo sumia do Storage, mas o link quebrado continuava no Firestore). Corrigido com uma flag dedicada (`_evidenciaAlterada`) em vez de inferir pela igualdade a `null`.
- Contador de itens filtrados na Listagem Geral ("Mostrando X de Y ações"), com correção de bug de pluralização ("açãoões" → "ações").
- Botões "Expandir/Colapsar Follow-ups" convertidos para ícone único, sem texto.

# v19.40 — Notificação por e-mail (mailto) e clique na linha

- Botão "✉️ Notificar" em cada ação corretiva do Novo Cadastro (só quando o problema já foi salvo) — monta um `mailto:` com assunto, corpo e link direto da ação, usando o e-mail cadastrado do responsável. Sem confirmação, sem rastreamento de "já notificado" — decisão deliberada do usuário para manter simples.
- Clicar em qualquer parte de uma linha da Listagem (fora dos botões) abre a edição, igual ao lápis.

# v19.41 – v19.45 — Backup com fotos: robustez e correção de UI reaproveitada

- Nova tentativa automática (retry com espera crescente) ao baixar cada foto do Storage durante o backup "com fotos" — investigação com dados reais mostrou um padrão de "quebra numa hora específica e nunca recupera" (rede/limite temporário), não CORS.
- Aviso de "fotos não baixadas" trocado de toast passageiro para modal persistente, listando **quais** problemas/cursos específicos falharam (não só a contagem).
- Backup "com fotos" passa a também baixar e embutir imagens coladas dentro do campo Notas (antes só cobria evidência de problema/fechamento/curso).
- Nova função `showAlert()` — aviso de reconhecimento com um único botão de verdade, substituindo o truque de usar `showConfirm()` com `okText`/`cancelText` iguais (que sempre renderizava dois botões duplicados).

# v19.46 – v19.50 — Limpeza de UI e correções no Dashboard

- Vários ajustes pontuais: texto "(recomendado)" removido, botão de notificar com texto ao lado do ícone, engrenagem redundante removida da tela de Curso 1 Tema.
- **Bug real corrigido**: gráficos "Pendências/Fechadas/Vencidas por Responsável" mostravam responsáveis com zero atividades — causa: inicialização de mapas auxiliares criava a chave do responsável mesmo sem nenhum valor de verdade.
- Painel "Atividade na Semana" refatorado para cálculo real baseado nas datas de corte da semana selecionada (Segunda a Domingo) — antes usava uma fórmula reversa que só funcionava para a semana atual. Identidade matemática (`Fechamento + Adicionadas − Fechadas = Pendentes`) validada com 5.000 simulações aleatórias e, depois, corrigida uma segunda vez ao encontrar um caso real de dado inválido (`data_efetiva` anterior à `data_abertura`) que quebrava a soma — fórmula agora resiliente a esse tipo de erro de digitação.
- Busca por número exato da ação na Listagem (`#327`).
- Tecla ESC fecha o visualizador de evidência ou sai da edição de ação (respeitando o aviso de alterações não salvas).
- Os 3 gráficos de defeito (Turno/Operador/Sub-tópico) passam a ficar realmente ocultos — não só vazios — quando o Plano selecionado não exige Operador/Turno.

# v19.51 – v19.53 — Classificação 6M e reorganização do Dashboard

- Novo campo **Classificação 6M** no cadastro do problema (Máquina, Método, Mão de Obra, Medição, Material, Meio Ambiente), com scripts de exportação/importação em lote para classificar a base retroativamente.
- Novo gráfico **Defeitos por Categoria (6M)** — disponível para qualquer Plano (diferente dos outros 3, que exigem Operador/Turno), contando por padrão 1 por problema; soma Quantidade de Peças quando esse modo está disponível.
- Layout do Dashboard reorganizado: Defeitos por Turno + 6M na mesma linha, Defeitos por Operador + Sub-Tópico na linha de baixo (6M ocupa a linha inteira sozinho quando Turno está oculto).
- Dropdown "Quantidade de Peças/Ocorrências" oculto por padrão (só aparece para Planos que exigem Operador/Quantidade), com "Ocorrências" como novo padrão.

---

## Scripts utilitários de manutenção (rodados via Console do navegador)

Ao longo dessas versões, diversos scripts avulsos foram criados para diagnóstico e correção em lote — sempre em modo `DRY_RUN` por padrão, reaproveitando funções já existentes no app quando possível:
ressincronização de Cursos 1 Tema por sub-tópico, atualização retroativa de comentários de operador em Notas, exportação/importação de códigos de sub-tópico, detecção de evidências com link quebrado no Storage, diagnóstico de ações sem Data de Ocorrência, diagnóstico de planos órfãos, diagnóstico detalhado de atividade semanal por responsável, e exportação/importação em lote da Classificação 6M.

---

*Documento gerado a partir do histórico de conversas do projeto ActionPlan — cobre da fase inicial até a v19.53.*
