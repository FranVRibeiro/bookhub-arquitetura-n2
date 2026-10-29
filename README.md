# BookHub — Arquitetura de Software N2

Este repositório contém a proposta arquitetural do BookHub para a avaliação N2 de Projeto e Arquitetura de Software. O projeto parte da plataforma de leitura digital desenvolvida em Programação Web e organiza a evolução para uma arquitetura inspirada em DDD, com uma camada de comunicação orientada a eventos e uma inspeção SOLID do código atual.

## Entregáveis

- `docs/avaliacao-n2-bookhub.md`: documento completo que pode ser convertido para PDF.
- `docs/guia-estudo-apresentacao.md`: roteiro de estudo e defesa oral.
- `docs/guia-publicacao-github.md`: passo a passo para publicar este repositório e os projetos Web.
- `architecture/ddd/`: organização proposta para DDD/Arquitetura Hexagonal.
- `architecture/c4/`: diagramas C4 e decisões da camada de mensageria.
- `architecture/events/`: catálogo de eventos de domínio e integração.
- `analysis/solid-inspection.md`: inspeção com seis ocorrências do código atual e propostas de refatoração.

## Domínio

O BookHub é uma plataforma de leitura digital. Usuários criam contas, publicam livros em PDF com capa, exploram o catálogo, visualizam detalhes, leem obras, curtem e comentam livros, acompanham autores e consultam seu histórico. Autores administram suas próprias obras; administradores podem atuar sobre o catálogo conforme as regras de autorização.

## Como visualizar o diagrama

O diagrama editável está em `architecture/c4/bookhub-events.mmd` e pode ser renderizado com Mermaid. A versão PNG fica em `architecture/c4/bookhub-events.png`.
