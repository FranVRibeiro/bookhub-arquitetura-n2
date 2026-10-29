# Guia de estudo e apresentação — BookHub N2

## Fala inicial sugerida

“Meu projeto é o BookHub, uma plataforma de leitura digital. O domínio possui usuários, livros, publicação de PDF, leitura, curtidas e comentários. Para a N2, eu separei o domínio em Identidade, Catálogo e Interação Social, coloquei casos de uso no centro e deixei Express, Sequelize, filesystem e mensageria como adaptadores.”

## Pergunta 1: por que DDD?

DDD ajuda a organizar o software pelo negócio. O BookHub não é apenas uma coleção de rotas: ele tem invariantes, como uma curtida por usuário/livro, propriedade do livro e comentário associado. Bounded Contexts reduzem ambiguidades e agregados protegem regras. A arquitetura hexagonal evita que o domínio conheça detalhes externos.

## Pergunta 2: por que eventos?

Depois que um livro é publicado, busca, notificação e analytics podem reagir sem o catálogo chamar cada módulo diretamente. O evento representa um fato ocorrido. RabbitMQ/AMQP desacopla produtores e consumidores; Outbox evita perder evento entre o commit do banco e a publicação.

## Pergunta 3: por que Outbox?

Se o sistema confirmar o livro e falhar antes de publicar, o livro existe, mas ninguém recebe o fato. Com Outbox, livro e evento são gravados na mesma transação. Um relay publica depois; retries e dead-letter tratam falhas.

## Pergunta 4: qual SOLID aparece no código?

O código atual já separa módulos, mas services ainda acumulam banco, filesystem, autorização e regras. Por isso encontrei SRP em `bookService` e `userService`, DIP por imports diretos de Sequelize/fs, OCP no bootstrap manual, ISP na interface ampla do service e uma política de remoção difícil de substituir, analisada como contrato LSP.

## Pergunta 5: precisa implementar a refatoração?

Não. O enunciado pede inspeção, trecho, princípio e sugestão. A arquitetura DDD é uma proposta organizada; a refatoração SOLID não precisa ser aplicada ao projeto Web.

## Termos para saber explicar

- **Agregado:** limite transacional de consistência.
- **Porta:** contrato interno para algo externo.
- **Adaptador:** implementação de uma porta ou entrada do sistema.
- **Evento de domínio:** fato relevante do negócio.
- **Idempotência:** repetir a mesma mensagem não produz efeito duplicado.
- **SRP:** uma razão principal para mudar.
- **DIP:** núcleo depende de abstrações, detalhes dependem delas.
