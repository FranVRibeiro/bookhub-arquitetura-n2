# Inspeção SOLID — BookHub

## Metodologia

A inspeção foi realizada por leitura estrutural e análise de responsabilidades. Para cada módulo, foram observados imports, funções exportadas, efeitos colaterais, acesso a infraestrutura e contratos de erro. O procedimento reproduzível é: clonar o projeto, instalar dependências, listar `modules`, ler os services/controllers, marcar cada responsabilidade e comparar com SRP/OCP/LSP/ISP/DIP. As linhas abaixo reproduzem trechos do código analisado; a refatoração é proposta, não implementada.

## 1. Service de Book com muitas razões para mudar — SRP

**Trecho:** `modules/book/bookService.js`

```js
const sequelize = require('../../config/database')
const fs = require('fs')
async function createWithFiles(data, userId, files) { /* valida arquivos, abre transação, grava Book, incrementa User */ }
async function remove(id, userId, isAdmin) { /* autoriza, remove arquivos, destrói registro, decrementa User */ }
```

O mesmo módulo consulta catálogo, decide autorização, controla transação, grava no banco, manipula filesystem e mantém contador. Mudanças de banco, storage, política de permissão ou regra de contador exigem editar o mesmo service.

**Refatoração:** separar `BookRepository`, `BookFileStorage`, `BookAuthorizationPolicy` e casos de uso `PublishBook`/`DeleteBook`. O caso de uso orquestra; cada porta tem uma responsabilidade.

## 2. UserService mistura identidade, autenticação e arquivos — SRP

**Trecho:**

```js
async function loginUser(email, password) { /* bcrypt, bloqueio e JWT */ }
async function updateUserProfile(id, {fullName, bio}, file) { /* banco e fs.unlinkSync */ }
```

Login, hash, emissão de token, perfil e substituição de foto são motivos diferentes para mudança.

**Refatoração:** `RegisterUser`, `AuthenticateUser`, `ProfileService` e `ProfilePictureStorage`, com um `PasswordHasher` e `TokenIssuer` como portas.

## 3. Dependência direta de detalhes — DIP

**Trecho:**

```js
const sequelize = require('../../config/database')
const fs = require('fs')
const Book = require('./bookModel')
```

A regra de aplicação depende de Sequelize e filesystem. Testes precisam de banco/disco reais e trocar MySQL por outro adaptador exige alterar o caso de uso.

**Refatoração:** receber `bookRepository`, `userRepository`, `fileStorage` e `unitOfWork` por injeção; as interfaces/portas pertencem ao núcleo e Sequelize/fs ficam nos adaptadores.

## 4. Registro manual no bootstrap — OCP

**Trecho:**

```js
app.use('/api', userRoutes)
app.use('/api', bookRoutes)
app.use('/api', likeRoutes)
app.use('/api', commentRoutes)
```

Cada novo módulo exige editar o bootstrap. O arquivo não está fechado para modificação: a inclusão de um adaptador altera o núcleo de inicialização.

**Refatoração:** cada módulo exporta um `registerRoutes(router, dependencies)` e um composition root descobre uma lista de módulos/configuração. O bootstrap permanece estável; novos módulos são adicionados como plugins.

## 5. Interface ampla de BookService — ISP

**Trecho:**

```js
module.exports = { list, getDetails, feed, createWithFiles, remove }
```

Um controller que só precisa consultar detalhes depende do módulo que também carrega criação com arquivos e exclusão física. Em termos funcionais, consumidores recebem uma interface maior do que precisam.

**Refatoração:** separar portas estreitas: `BookReader`, `BookPublisher`, `BookEditor` e `BookRemover`. Cada controller importa somente o contrato de sua operação.

## 6. Política de autorização misturada com remoção — LSP/contrato comportamental e DIP

**Trecho:**

```js
async function remove(id, userId, isAdmin) {
  if (book.userId !== userId && !isAdmin) { /* 403 */ }
  // remove arquivos, destrói registro e decrementa contador
}
```

A função exige que o chamador conheça a política concreta (`isAdmin`) e combina duas pré-condições: identidade do solicitante e remoção. Uma política alternativa, como moderador ou proprietário delegado, não pode ser substituída sem alterar a assinatura e o comportamento. Isso viola a ideia de substituir políticas por contratos estáveis.

**Refatoração:** `AuthorizationPolicy.canDelete(actor, book)` retorna uma decisão/erro de domínio; `DeleteBook` recebe essa porta e só executa a remoção quando o contrato de autorização for satisfeito. Políticas diferentes tornam-se substituíveis sem alterar o caso de uso.

## Observação sobre Liskov

Como o projeto é funcional e não usa hierarquia de classes nesse trecho, LSP foi analisado como substituição comportamental de políticas/portas. A violação não é “herança incorreta”; é um contrato que fica preso a `isAdmin` e impede trocar a política por outra implementação compatível.
