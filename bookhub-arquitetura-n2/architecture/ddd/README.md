# Arquitetura alvo — BookHub DDD/Hexagonal

```text
src/
├── contexts/
│   ├── identity-access/
│   │   ├── domain/
│   │   │   ├── entities/User.js
│   │   │   ├── value-objects/Email.js
│   │   │   ├── events/UserRegistered.js
│   │   │   └── ports/UserRepository.js
│   │   ├── application/
│   │   │   ├── use-cases/RegisterUser.js
│   │   │   ├── use-cases/LoginUser.js
│   │   │   └── dto/
│   │   └── adapters/
│   │       ├── in/http/UserController.js
│   │       └── out/sequelize/SequelizeUserRepository.js
│   ├── catalog-reading/
│   │   ├── domain/
│   │   │   ├── entities/Book.js
│   │   │   ├── value-objects/BookTitle.js
│   │   │   ├── events/BookPublished.js
│   │   │   └── ports/{BookRepository,FileStorage}.js
│   │   ├── application/
│   │   │   ├── use-cases/{PublishBook,UpdateBook,DeleteBook,GetBookDetail}.js
│   │   │   └── dto/
│   │   └── adapters/{in/http,out/sequelize,out/storage}/
│   ├── social-interaction/
│   │   ├── domain/{entities,events,ports}/
│   │   ├── application/use-cases/{ToggleLike,AddComment,ListComments}.js
│   │   └── adapters/{in/http,out/sequelize}/
│   └── administration/
│       ├── domain/
│       ├── application/
│       └── adapters/
├── shared/
│   ├── domain/{DomainEvent,Entity,Result}.js
│   ├── application/{UnitOfWork,EventBus}.js
│   └── infrastructure/{config,logging,errors}/
└── bootstrap/
    ├── http.js
    ├── dependencies.js
    └── routes.js
```

As portas ficam no lado interno; adaptadores implementam as portas. O domínio não importa Sequelize, Express, Multer, JWT ou RabbitMQ.
