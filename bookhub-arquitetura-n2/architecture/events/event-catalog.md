# Catálogo de eventos

| Evento | Agregado | Quando | Consumidores | Idempotência |
|---|---|---|---|---|
| `BookPublished.v1` | Book | após publicação confirmada | busca, notificações, analytics | `eventId` único |
| `BookUpdated.v1` | Book | após edição | busca, cache | versão por agregado |
| `BookDeleted.v1` | Book | após exclusão | busca, storage, analytics | delete idempotente |
| `BookLiked.v1` | Like | após curtida | notificações, analytics | chave user+book |
| `CommentAdded.v1` | Comment | após comentário | autor, moderação, analytics | `eventId` único |
| `UserFollowed.v1` | Follow | após seguir autor | feed, notificações | chave follower+followee |

## Envelope

```json
{
  "eventId": "uuid",
  "eventType": "BookPublished",
  "version": 1,
  "occurredAt": "2026-10-29T10:00:00Z",
  "aggregateType": "Book",
  "aggregateId": 1,
  "payload": { "title": "...", "ownerId": 1 }
}
```

O produtor grava o envelope na Outbox dentro da transação do agregado. O relay publica e marca `publishedAt`. Falhas são tentadas novamente e depois enviadas para uma dead-letter queue.
