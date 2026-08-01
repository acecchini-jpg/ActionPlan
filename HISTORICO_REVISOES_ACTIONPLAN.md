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

---

*Documento gerado a partir do histórico de conversas do projeto ActionPlan — cobre da fase inicial até a v19.22.*
