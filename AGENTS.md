# AGENTS.md — Commizo

## 1. Contexto e objetivo

O **Commizo** é uma aplicação web SaaS multiusuário que permite a desenvolvedores autenticarem-se com o GitHub e registrarem, pelo navegador (inclusive no celular), atividades reais de desenvolvimento ou estudo em um repositório autorizado. O backend usa a API do GitHub para criar ou atualizar um arquivo e gerar um commit, sem clonar o repositório nem executar Git no servidor.

O produto deve priorizar **segurança, simplicidade, transparência e experiência mobile**. A marca se chama **Commizo** (não StreakKeeper), com nome visual em minúsculas: `commizo`. Seu objetivo é facilitar contribuições legítimas, inclusive quando o usuário estiver longe do computador. Não prometer que qualquer commit manterá o streak: a contabilização é decisão do GitHub e depende das regras da plataforma.

**Idioma:** comunique-se com o usuário em português do Brasil. Use nomes de classes, métodos, variáveis, APIs e identificadores em inglês. Não traduza nomes de bibliotecas, protocolos ou convenções de código.

## 2. Stack e decisões já tomadas

- **Backend obrigatório:** Java 21, Spring Boot, Maven, Spring Web, Spring Security e OAuth2 Client.
- **Banco:** PostgreSQL, Spring Data JPA e Flyway.
- **Integração GitHub:** GitHub App, GitHub REST API e, quando necessário, GraphQL; preferir `RestClient` do Spring para HTTP síncrono.
- **Frontend:** Next.js, TypeScript e Tailwind CSS; responsivo e mobile-first.
- **Testes backend:** JUnit 5, Mockito, MockMvc e Testcontainers para integrações reais.
- **Documentação:** OpenAPI/Swagger para a API própria.
- **Infraestrutura:** Docker/Compose em desenvolvimento; pipelines com GitHub Actions quando houver necessidade.
- **Arquitetura:** monólito modular. **Não** introduzir microsserviços, RabbitMQ, Redis, Kubernetes ou filas sem justificativa concreta.
- **Não utilizar NestJS, Node.js ou TypeScript no backend.** O Node.js pode existir somente como runtime e ferramenta do frontend.
- Não adicionar dependências, bibliotecas ou infraestrutura antes de demonstrar a necessidade.

## 3. Identidade visual e experiência do produto

A identidade da marca foi aprovada e **não deve ser redesenhada sem solicitação explícita**.

- **Marca:** Commizo; no logotipo/wordmark usar `commizo` em letras minúsculas.
- **Slogan:** **“Commits de qualquer lugar”** (pode ser omitido em espaços pequenos).
- **Símbolo:** letra **C construída com exatamente seis quadrados arredondados** (tiles de contribuições do GitHub), sem contorno, sem moldura e sem elementos extras. Geometria conceitual, com posições iguais e espaçamento regular:

  ```text
  · ■ ■
  ■ · ·
  ■ · ·
  · ■ ■
  ```

- Os quadrados têm **mesmo tamanho** e **cantos suavemente arredondados**. Os tons de verde progridem de **mais claros na parte superior para mais escuros na parte inferior**, evocando a intensidade do gráfico de contribuições. Manter forma e contagem exatas nas versões de ícone, favicon e logo; não adicionar quadrados de fundo que alterem o C.
- **Estilo:** minimalista, geométrico, contemporâneo, orientado a produto SaaS; evitar gradientes excessivos, efeitos 3D, mascotes ou ornamentos desnecessários.
- **Base cromática proposta para a interface:** fundo `#0B0F14`, verde claro `#A8FF60`, verde médio `#22C55E`, verde escuro `#15803D`, superfície `#1F2937`, texto claro `#F8FAFC`. Usar esses valores como referência inicial da UI; reutilizar tokens de tema e ajustar contraste conforme acessibilidade. Não supor que estes hexadecimais sejam uma amostragem exata dos pixels da arte final.
- **Tipografia de interface sugerida:** Inter (com fontes sans-serif de fallback). Wordmark em peso forte e caixa baixa.
- **Versões de uso:** principal sobre fundo escuro; alternativa sobre fundo claro; ícone sem texto para favicon/PWA. Preferir SVG para vetores e PNG apropriado para imagens rasterizadas.
- **Assets:** antes de inserir logo no frontend, verificar quais arquivos reais já existem no repositório. Não inventar caminhos de assets nem recriar uma logo divergente a partir de uma descrição quando houver arte aprovada disponível.
- **UI:** responsiva e mobile-first; estados de carregamento, erro, sucesso, autorização revogada e contribuição ainda não contabilizada devem ser claros. Evitar a mensagem enganosa “streak garantido”: commit criado e contribuição contabilizada são estados diferentes.
- **Conteúdo:** interface em português do Brasil na primeira versão, mensagens curtas e objetivas; nomes de código e identificadores em inglês.

## 4. Escopo do MVP

1. Autenticação de usuários com o GitHub.
2. Instalação/autorização da GitHub App com acesso somente aos repositórios selecionados.
3. Listagem e seleção de repositórios autorizados, inicialmente com preferência por repositório privado dedicado a registros.
4. Registro de uma atividade curta com data, descrição e repositório de destino.
5. Criação ou atualização de arquivo Markdown em um diretório reservado (`logs/YYYY-MM-DD.md`) no repositório selecionado, seguida do commit via API do GitHub.
6. Persistência do resultado (sucesso/falha, SHA, URL, horário e contexto seguro) e prevenção de duplicações por reenvio.
7. Consulta e exibição de contribuições reais e sequência de dias, distinguindo **commit criado** de **contribuição contabilizada**.
8. Interface web intuitiva e responsiva com mensagens claras de erro e sucesso.

**Fora do MVP:** commits agendados ou automáticos, edição arbitrária de arquivos de código, gamificação complexa, planos pagos, microsserviços e funcionalidades sociais. Só adicionar após solicitação explícita.

## 5. Arquitetura e organização

Inspecionar a estrutura real do repositório primeiro. Se ainda estiver vazio, propor uma estrutura simples como:

```text
commizo/
├── backend/                # Java 21 + Spring Boot + Maven
│   ├── pom.xml
│   └── src/
├── frontend/               # Next.js + TypeScript
├── compose.yaml            # Infra local, quando necessária
├── README.md
├── AGENTS.md
└── CLAUDE.md
```

Se o projeto já tiver um package root diferente, **não renomear pacotes automaticamente**; propor a mudança antes. No backend, organizar **por funcionalidade**, não criar uma floresta de camadas globais:

```text
com.commizo
├── auth/                   # Sessão, OAuth e autorização
├── user/                   # Perfil e preferências, incluindo fuso horário
├── github/                 # Cliente HTTP, GitHub App e DTOs externos
├── repository/             # Repositórios autorizados e seleção
├── contribution/           # Registro, commit, histórico e streak
└── common/                 # Tratamento de erros/configurações compartilhadas
```

Usar pacotes internos (`controller`, `service`, `repository`, `domain`, `dto`) apenas quando fizer sentido. Manter controllers enxutos, regras de negócio em services e integração externa isolada de entidades JPA. Evitar abstrações desnecessárias.

## 6. Regras de integração com o GitHub

- Usar **GitHub App** para autenticação e autorização granular. Diferenciar OAuth de usuário, instalação do app, token de instalação e user access token; escolher credenciais conforme a operação e as permissões exigidas.
- Nunca solicitar Personal Access Token em formulário como fluxo padrão e nunca expor tokens ao frontend.
- Para escrita, exigir autorização explícita do usuário e `Contents: read/write` somente nos repositórios selecionados. Solicitar outras permissões apenas se o endpoint escolhido realmente exigir.
- Verificar que o repositório solicitado pertence ao conjunto de repositórios autorizados para aquele usuário, em cada operação de escrita. Não confiar em `owner`, `repo`, `installationId` ou `userId` vindos do cliente.
- Descobrir a **branch padrão** do repositório; não presumir `main` ou `master`. Rejeitar forks ou explicar que suas contribuições podem não ser contabilizadas.
- Usar o endpoint de conteúdos para criar/atualizar Markdown: `PUT /repos/{owner}/{repo}/contents/{path}`. Para atualizar um arquivo, obter e enviar seu SHA atual; lidar com conflitos `409`, erros `401/403/404`, rate limits e falhas transitórias.
- Garantir autoria atribuível à conta do usuário: validar identidade e e-mail apropriado/autorizado, inclusive endereço `noreply` associado, respeitando as regras do GitHub. **Não atribuir commits indevidamente a um bot ou a outra pessoa.** Validar a estratégia em teste real com duas contas.
- Não afirmar que o streak foi mantido imediatamente após receber sucesso do endpoint. O status de contribuição no gráfico deve ser obtido do GitHub e pode demorar a atualizar.
- Considerar fuso horário configurado pelo usuário e regras de datas do GitHub; usar `Instant` para eventos e `LocalDate` + `ZoneId` para a experiência do usuário. Não forçar/backdate commits nem manipular datas para simular contribuições anteriores.
- Evitar alterar arquivos de aplicação do usuário. O Commizo escreve apenas na área previamente aprovada (`logs/`), com caminho e conteúdo validados.
- Respeitar rate limits, termos e políticas do GitHub. Não implementar contornos de limites ou geração em massa de contribuições artificiais.

## 7. Segurança e privacidade (obrigatórias)

- Aplicação multiusuário: toda consulta e mutação de dados deve validar o usuário autenticado e o acesso ao recurso. Impedir IDOR e vazamento entre contas.
- Usar permissões mínimas; proteger credenciais em armazenamento seguro, com criptografia em repouso quando persistidas, rotação/expiração e possibilidade de revogação/desconexão.
- Nunca registrar tokens, cookies, `Authorization`, códigos OAuth, chaves privadas ou descrições sensíveis em logs. Nunca salvar segredos no Git.
- Usar `state` e proteções adequadas ao fluxo OAuth; proteger sessões com cookies `HttpOnly`, `Secure` e `SameSite` conforme a arquitetura. Aplicar CSRF se autenticação baseada em cookies e CORS restritivo.
- Segredos somente via variáveis de ambiente/secret manager. Manter `.env.example` sem valores reais e `.gitignore` apropriado.
- Validar entradas com Bean Validation, limitar comprimento de descrições e tamanho de arquivos, escapar conteúdo quando renderizado e prevenir path traversal.
- Nunca enviar o conteúdo de repositórios privados a serviços terceiros fora do estritamente necessário para a funcionalidade aprovada.
- Tratar chamadas à API do GitHub com timeout explícito, erros tipados e mensagens seguras. Não expor stack traces ao cliente.
- Seguir princípio de confirmação explícita para commits: **nenhuma escrita em repositórios sem ação clara ou consentimento previamente configurado**.

## 8. Modelagem inicial (sujeita à evolução)

Entidades/tabelas candidatas:

- `users`: ID imutável do GitHub, login, avatar, preferências e `zone_id`.
- `github_connections`: vínculo entre usuário e instalação/autorização GitHub; guardar apenas dados necessários e segredos protegidos.
- `repositories`: ID imutável do repositório no GitHub, owner, name, seleção, status de autorização, branch padrão (revalidada quando necessário).
- `contribution_logs`: usuário, repositório, data local pretendida, descrição, estado (`PENDING`, `COMMITTED`, `FAILED`), commit SHA/URL, timestamps e chave de idempotência.

- Usar migrações Flyway versionadas; nunca depender de `ddl-auto=update` como estratégia de migração em produção.
- Modelar erros, concorrência e reenvios: uma resposta perdida após commit não pode criar duplicatas silenciosas.
- Não usar a tabela de registros locais como verdade absoluta do gráfico de contribuições do GitHub.

## 9. Contrato da API e comportamento

Antes de criar endpoints, definir DTOs e respostas coerentes. Sugestões (ajustar após discutir autenticação e analisar o repositório):

- `GET /api/v1/me` — dados do usuário autenticado.
- `GET /api/v1/repositories` — repositórios permitidos ao usuário.
- `POST /api/v1/contributions` — valida e solicita registro de atividade; exige autenticação, repositório autorizado e idempotência.
- `GET /api/v1/contributions` — histórico paginado do usuário.
- `GET /api/v1/contributions/summary` — visão do streak e evidência/fonte de contabilização.

Preferir `ProblemDetail` para erros HTTP, códigos adequados, paginação quando pertinente e DTOs específicos. Não expor entidades JPA diretamente pela API.

## 10. Qualidade e testes

- Escrever testes para regras de autorização, isolamento entre usuários, validação de repositórios, criação/atualização de arquivo, SHA, conflito, rate limit, duplicidade, fuso horário e falhas da API externa.
- Usar **JUnit 5 + Mockito** para regras isoladas; **MockMvc** para controllers/segurança; **Testcontainers + PostgreSQL** para persistência e migrações; testes com cliente HTTP simulado para GitHub.
- Não usar tokens reais em testes automatizados nem depender de API externa para CI.
- Manter um procedimento **manual e seguro** para validar a atribuição real de contribuições com contas/repositórios de teste; documentar que o gráfico pode atualizar com atraso.
- Executar testes e build dos módulos alterados antes de concluir; se um comando não puder ser executado, explicar o motivo e não afirmar que passou.
- Evitar `@SpringBootTest` para tudo: usar testes menores quando possível.

## 11. Padrões de implementação

- Código legível, coeso e simples; preferir composição a hierarquias artificiais.
- Java: `record` para DTOs imutáveis quando apropriado; `Optional` para ausência em retorno, não para campos JPA indiscriminadamente; sem `catch (Exception)` genérico para esconder erros.
- Definir limites transacionais com cuidado: **não manter transação de banco aberta durante chamada HTTP externa**. Persistir estado e reconciliar resultados quando necessário.
- TypeScript com `strict` habilitado, componentes acessíveis e feedback de carregamento, sucesso e erro no frontend.
- Usar Conventional Commits (`feat:`, `fix:`, `test:`, `docs:`, `refactor:`) se o repositório não definir outro padrão.
- Nunca fazer `git push`, criar PR, modificar configurações remotas, instalar dependências globais ou executar operações destrutivas sem autorização do usuário.
- Atualizar o README e `.env.example` quando introduzir comandos, configurações ou requisitos de execução.

## 12. Como o agente deve trabalhar

1. **Inspecionar o estado real do repositório** antes de criar arquivos ou assumir que uma estrutura já existe.
2. Para tarefas grandes, apresentar um plano curto, apontando decisões pendentes e dependências; para tarefas pequenas, implementar diretamente.
3. Implementar em incrementos verticais pequenos: API + regra + persistência/integração + testes conforme necessário.
4. Não criar funcionalidades fora do pedido nem trocar a stack/arquitetura definida por preferência própria.
5. Em decisões relevantes (tipo de token GitHub, sessão, autoria do commit, rate limit, privacidade), explicar trade-offs e consultar documentação oficial quando necessário.
6. Antes de modificar código existente, ler os arquivos envolvidos e preservar os padrões que já funcionam.
7. Ao concluir, informar objetivamente: arquivos alterados, comportamento entregue, comandos/testes executados, limitações e próximo passo recomendado.
8. Não inventar resultados de testes, métricas, endpoints prontos ou decisões que ainda não foram implementadas.
9. Perguntar somente por decisões realmente bloqueantes. Para detalhes secundários, escolher a alternativa mais simples e explicitar a escolha.

## 13. Ordem de implementação recomendada

1. Bootstrap do backend Java/Spring Boot e infraestrutura local com PostgreSQL/Flyway.
2. Fluxo GitHub App: autenticação, instalação e autorização em repositórios selecionados.
3. Listagem segura de repositórios do usuário.
4. Primeiro commit em repositório privado de teste, atribuído corretamente ao usuário.
5. Histórico persistido e idempotência.
6. Consulta real de contribuições e regras de streak.
7. Frontend Next.js mobile-first.
8. Testes de segurança, observabilidade básica e deploy.

**Critério de sucesso inicial:** duas contas GitHub distintas conseguem autorizar repositórios diferentes e registrar commits atribuídos corretamente, sem acessar recursos uma da outra. O sistema mostra separadamente a confirmação do commit e a contabilização da contribuição no GitHub.
