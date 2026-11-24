# Restaurante Flor Imperatriz · CQRS em .NET 10

> 🚧 Projeto em construção. É o código de exemplo de um artigo no Medium sobre CQRS.

Uma Minimal API em **.NET 10** para reservar mesas de restaurante. O cliente escolhe data, horário e número de pessoas e, na mesma reserva, já pode pedir **entradas** e **bebidas**, que chegam à mesa quando ele sentar.

> "Reservar mesa é fácil. E se o cliente quiser chegar com a caipirinha na mesa?"

## O que o projeto mostra

- **CQRS sem MediatR:** commands e queries passam por um dispatcher próprio. CQRS é um padrão, não uma biblioteca.
- **Escrita e leitura com modelos diferentes:** os commands carregam entidades com tracking e aplicam as regras de domínio. As queries usam `AsNoTracking` e projetam direto para DTOs feitos para cada tela.
- **Vertical slices:** cada caso de uso fica numa pasta própria (command/query, handler, validator e endpoint).
- **Regras de negócio testáveis:** `TimeProvider` injetado e `FakeTimeProvider` nos testes.

## Stack

- .NET 10 (SDK fixado via `global.json`), Minimal APIs
- EF Core 10 com SQLite (um único banco; a separação está no modelo)
- FluentValidation
- OpenAPI + Scalar
- xUnit

## Como rodar

```bash
dotnet build
dotnet test
dotnet run --project src/Restaurante.Api
```

A especificação OpenAPI fica em `/openapi/v1.json`. O banco é criado e populado (mesas e cardápio) automaticamente na primeira execução.

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| GET | `/mesas/disponiveis?dataHora=&pessoas=&area=` | Mesas livres para o horário |
| GET | `/cardapio/pre-reserva?categoria=` | Entradas e bebidas disponíveis |
| POST | `/reservas` | Cria uma reserva |
| GET | `/reservas/{id}` | Detalhe da reserva, com itens e total |
| GET | `/reservas?data=` | Reservas do dia (visão da cozinha e do bar) |
| POST | `/reservas/{id}/itens` | Adiciona uma entrada ou bebida |
| DELETE | `/reservas/{id}/itens/{itemId}` | Remove um item |
| POST | `/reservas/{id}/confirmar` | Confirma a reserva |
| POST | `/reservas/{id}/cancelar` | Cancela a reserva |

Erros seguem `ProblemDetails`: **400** para validação, **404** para recurso inexistente, **409** para mesa já ocupada e **422** para as demais regras de negócio.

## Principais regras

- A mesa precisa comportar o número de pessoas e não pode ter outra reserva ativa numa janela de 2h.
- Só **entradas** e **bebidas** podem ser pré-reservadas.
- Os itens podem ser alterados até 2h antes do horário, e o cancelamento é permitido até 1h antes.
- O preço do item é copiado para a reserva no momento do pedido: se o cardápio mudar, a reserva mantém o valor combinado.

## Estrutura

```
src/Restaurante.Api/
├─ Abstractions/    → ICommand, IQuery, Dispatcher, Result
├─ Domain/          → Mesa, ItemCardapio, Reserva, ItemReserva
├─ Infrastructure/  → AppDbContext, mapeamentos e seed
└─ Features/        → Reservas, Mesas, Cardapio (uma pasta por caso de uso)
tests/Restaurante.Tests/
```

## Licença

MIT
