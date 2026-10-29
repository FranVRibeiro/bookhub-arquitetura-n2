# Avaliação N2 — Projeto e Arquitetura de Software
## BookHub: DDD, arquitetura orientada a eventos e princípios SOLID

**Estudante:** Francieli Ribeiro

## 1. Domínio de aplicação

O BookHub é uma plataforma de leitura digital voltada para leitores e autores independentes. O sistema centraliza o catálogo de livros, publicação de PDFs e capas, leitura, descoberta por feed e busca, interação social por curtidas e comentários e relacionamento com autores. O domínio possui regras de identidade e autorização: uma pessoa deve estar autenticada para publicar, editar ou excluir suas obras; somente uma curtida por pessoa e livro é permitida; comentários pertencem a um livro e a um autor; e o proprietário pode editar sua obra. O problema de negócio é oferecer uma experiência de leitura e descoberta com controle do acervo e feedback social mensurável.

## 2. Questão 1 — Transformação para DDD e arquitetura hexagonal

A proposta separa o domínio das tecnologias externas. O núcleo não conhece Express, Sequelize, MySQL, Multer ou Axios. A entrada HTTP é um adaptador; o banco, o armazenamento de arquivos e o mensageiro também são adaptadores. Casos de uso orquestram as regras e dependem de portas, não de implementações concretas.

### 2.1 Bounded Contexts

**Identidade e Acesso:** cadastro, login, JWT, perfil, bloqueio, papéis e autorização.

**Catálogo e Leitura:** livro, autor, publicação, PDF, capa, categorias, busca, feed, visualizações e leitura.

**Interação Social:** curtida, comentário, contador e feed social.

**Administração:** moderação, exclusão, bloqueio e métricas administrativas.

O agregado principal do contexto Catálogo é `Book`. `Book` mantém invariantes como título/descrição válidos e relação com o proprietário. `Like` é uma entidade associativa com identidade própria e restrição única `(userId, bookId)`. `Comment` pertence a um `Book` e a um `User`, com texto limitado e não vazio.

### 2.2 Camadas

- **Domain:** entidades, value objects, eventos de domínio e contratos de repositório.
- **Application:** casos de uso, DTOs, portas de entrada e transações como unidade de trabalho.
- **Adapters/In:** controllers HTTP, validators, autenticação e upload.
- **Adapters/Out:** repositórios Sequelize, armazenamento local/S3, JWT e mensageria.
- **Infrastructure:** Express, MySQL, Sequelize, filesystem e configuração.

### 2.3 Fluxo de publicação

1. `PublishBookController` recebe multipart e autenticação.
2. O caso de uso `PublishBook` valida comando e chama as portas `BookRepository` e `FileStorage`.
3. O agregado `Book` é criado e emite `BookPublished`.
4. A unidade de trabalho confirma banco e arquivos segundo uma política definida.
5. O adaptador de mensageria publica o evento depois do commit usando Outbox.
6. Consumidores atualizam projeções, notificações e métricas sem acoplar o caso de uso ao canal.

### 2.4 Organização DDD proposta

A árvore completa está em `architecture/ddd/README.md`. Ela não substitui imediatamente o código Web: é uma arquitetura alvo, conforme permitido pelo enunciado, e serve para orientar uma migração incremental.

## 3. Questão 2 — Camada de comunicação orientada a eventos

### 3.1 Necessidade de negócio

O BookHub precisa reagir a fatos sem tornar a publicação dependente de todos os consumidores. Quando um livro é publicado, leitores interessados podem receber uma notificação, o catálogo de busca pode atualizar sua projeção e uma métrica de publicação pode ser registrada. Quando um livro recebe uma curtida ou comentário, contadores, notificações e recomendações podem evoluir independentemente.

A mensageria reduz o acoplamento temporal entre o módulo de Catálogo e os módulos de Notificação, Busca e Analytics. O autor recebe resposta rápida; uma falha transitória em uma notificação não precisa impedir a persistência do livro.

### 3.2 Escolha técnica

A proposta usa um broker compatível com AMQP, como RabbitMQ, com exchanges por domínio e filas por consumidor. A persistência local utiliza o padrão Outbox: o evento é salvo na mesma transação do agregado em `outbox_events`; um publicador entrega eventos pendentes ao broker e marca o registro como publicado. Isso evita o problema de confirmar no banco e falhar antes de publicar.

Eventos são fatos imutáveis, versionados e idempotentes. Cada evento possui `eventId`, `eventType`, `version`, `occurredAt`, `aggregateId` e `payload`. Consumidores guardam `eventId` processado ou usam operações idempotentes. O broker deve ter retry com backoff e dead-letter queue.

### 3.3 Eventos principais

- `BookPublished.v1`: livro publicado; consumidores: Search Projection, Notification e Analytics.
- `BookUpdated.v1`: metadados ou capa alterados; consumidores: Search Projection e Cache.
- `BookDeleted.v1`: livro removido; consumidores: Search Projection, Storage Cleanup e Analytics.
- `BookLiked.v1` / `BookUnliked.v1`: curtida alterada; consumidores: Notification e Analytics.
- `CommentAdded.v1`: comentário criado; consumidores: Notification, Moderation e Analytics.
- `UserFollowed.v1`: relacionamento de acompanhamento; consumidores: Feed e Notification.

### 3.4 C4

O C4 em níveis de contexto e contêiner está representado em `architecture/c4/bookhub-events.mmd` e em `architecture/c4/bookhub-events.png`. O sistema web continua oferecendo HTTP ao frontend, mas passa a publicar eventos em uma camada de comunicação. O broker não substitui o banco: ele transporta fatos; a consistência do agregado continua sendo garantida pela transação.

## 4. Questão 3 — Metodologia da inspeção SOLID

A inspeção foi feita sobre a versão do backend utilizada nas Atividades 09 e 10. O procedimento foi reproduzível: (1) listar módulos e dependências; (2) identificar funções públicas e efeitos colaterais; (3) classificar cada módulo pelo papel que desempenha; (4) localizar dependências concretas de banco, filesystem, JWT e HTTP; (5) comparar o desenho com SRP, OCP, LSP, ISP e DIP; (6) registrar trecho, impacto, princípio e refatoração sugerida. A análise trata módulos e funções como equivalentes funcionais de classes e métodos, conforme a orientação do enunciado.

Foram encontradas seis ocorrências documentadas em `analysis/solid-inspection.md`. As propostas são sugestões acadêmicas e não precisam ser implementadas no projeto Web atual.

### 4.1 Síntese das ocorrências

1. `bookService.js` concentra consultas, regra de autorização, transação, filesystem e contadores — SRP.
2. `userService.js` concentra cadastro, login, JWT, perfil e filesystem — SRP.
3. `bookService.js` depende diretamente de Sequelize e `fs` — DIP.
4. O registro de módulos e rotas em `app.js` exige alteração do núcleo para cada novo adaptador — OCP.
5. O módulo `bookService` expõe uma interface ampla de operações para consumidores com necessidades diferentes — ISP.
6. `BookService.remove` mistura política de autorização de proprietário/admin com persistência e limpeza física — LSP/contrato de substituição e DIP, pois uma política alternativa não pode ser substituída sem conhecer detalhes concretos.

A classificação não afirma que JavaScript tenha herança clássica nesse trecho; LSP é analisado como contrato comportamental. A sugestão é separar portas e políticas para que implementações substitutas preservem pré-condições, pós-condições e erros esperados.

## 5. Conclusão

A transformação proposta preserva o que já funciona no BookHub, mas cria limites explícitos para o crescimento: regras de negócio no domínio, casos de uso na aplicação, adaptadores nas bordas e eventos para integração assíncrona. A inspeção SOLID mostra que o código funcional modular já possui separação inicial, mas ainda mistura efeitos e detalhes de infraestrutura em services. O próximo passo natural seria migrar um caso de uso, como `PublishBook`, usando portas, Outbox e testes de contrato antes de mover todo o sistema.
