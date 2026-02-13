# High-Performance-Transactional-Wallet-API

![Java 21](https://img.shields.io/badge/Java-21-orange)
![Spring Boot 3](https://img.shields.io/badge/Spring_Boot-3.2-green)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue)
![Coverage](https://img.shields.io/badge/Coverage-90%25-brightgreen)

## 📖 Visão Geral do Projeto

Este projeto implementa uma API RESTful de alta performance para transações financeiras (carteira digital), com foco obsessivo em **consistência ACID**, **controle de concorrência** e **idempotência**.

Diferente de aplicações CRUD simples, este sistema foi engenheirado para lidar com cenários financeiros do "Mundo Real", como condições de corrida (race conditions) durante saques simultâneos e garantia de integridade de dados em ambientes distribuídos.

A arquitetura segue estritamente os princípios da **Arquitetura Hexagonal (Ports and Adapters)** e **Domain-Driven Design (DDD)**, garantindo que o núcleo da aplicação seja agnóstico a frameworks e infraestrutura.

---

## 🏗️ Design Arquitetural

A aplicação foi desenhada utilizando um **Modelo de Domínio Rico**. A lógica *core* reside na camada de domínio e não possui dependências de frameworks (Spring) ou bibliotecas externas.

### Camadas da Arquitetura Hexagonal

1.  **Domain (Core):** Entidades Java puras, Value Objects (Money, CPF) e Exceções de Domínio. Regras como "saldo não pode ficar negativo" vivem aqui.
2.  **Application (Use Cases):** Orquestra o fluxo de dados. Define as `Portas de Entrada` (interfaces) e `Portas de Saída` (interfaces para persistência/notificação).
3.  **Infrastructure (Adapters):** Implementações concretas das portas.
    * *Driving Adapters:* Controladores REST (Spring Web).
    * *Driven Adapters:* Repositórios Postgres (Spring Data JPA), Serviços de Notificação.

## Mermaid
```
graph TD
    subgraph "Infrastructure (Adapters)"
        Controller[REST Controller] -->|DTO| InputPort
        RepoImpl[JPA Repository] -.->|Implementa| OutputPort
    end

    subgraph "Application (Hexagon)"
        InputPort[Caso de Uso / Porta] --> Domain
        UseCaseImpl -->|Chama| OutputPort[Porta de Saída]
    end
```
   
## 🚀 Desafios de Engenharia & Soluções

### 1. Concorrência & Race Conditions
O Problema: Em um ambiente altamente concorrente, duas requisições para transferir dinheiro da mesma carteira podem chegar simultaneamente. Sem proteção, ambas poderiam ler o mesmo saldo inicial, resultando em "gasto duplo" e inconsistência financeira.

A Solução: Implementação de Pessimistic Locking (SELECT ... FOR UPDATE) no nível do banco de dados.

Ao iniciar uma transação, a linha da carteira é bloqueada (travada).

Requisições subsequentes aguardam até que o bloqueio seja liberado (commit ou rollback).

Isso garante acesso estritamente serializado à lógica de modificação de saldo.

### 2. Precisão Monetária
O Problema: O uso de double ou float para valores monetários leva a erros de precisão de ponto flutuante (ex: 0.1 + 0.2 != 0.3).

A Solução: Uso estrito de BigDecimal para todas as operações financeiras, com modos de arredondamento centralizados na camada de Domínio.

### 3. Confiabilidade & Estratégia de Testes
A qualidade é inegociável em Fintechs. Este projeto emprega uma "Pirâmide de Testes" rigorosa:

Testes Unitários (JUnit 5 + Mockito): Validam regras de negócio dentro do Domínio Rico isolado. Execução em milissegundos.

Testes de Integração (Testcontainers): Validam a interação com um banco PostgreSQL real. Isso garante que queries SQL, constraints e Pessimistic Locks funcionem como esperado em um ambiente idêntico à produção. Não utilizamos bancos em memória (H2).

Testes E2E (RestAssured): Testes caixa-preta dos endpoints da API, validando códigos de status HTTP, serialização JSON e tratamento de erros.

## 🛠️ Stack Tecnológica
### Linguagem: Java 21.
### Framework: Spring Boot 3.2.
### Banco de Dados: PostgreSQL 16.
### Persistência: Spring Data JPA + Flyway.
### Mapeamento: MapStruct.
### Testes: JUnit 5, Mockito, Testcontainers, RestAssured, Instancio.
### Documentação: SpringDoc OpenAPI.
### Containerização: Docker & Docker Compose.
