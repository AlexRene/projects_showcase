# Sistema de Tesouraria

> Sistema backend para gerenciamento financeiro municipal, com controle de entradas, saídas, autenticação de usuários e regras de negócio específicas da área de tesouraria.

![Java](https://img.shields.io/badge/Java-21-007396?style=flat-square\&logo=java\&logoColor=white)
![Spring Boot](https://img.shields.io/badge/SpringBoot-4.x-6DB33F?style=flat-square\&logo=springboot\&logoColor=white)
![Hibernate](https://img.shields.io/badge/Hibernate-JPA-59666C?style=flat-square\&logo=hibernate\&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=flat-square\&logo=postgresql\&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square\&logo=docker\&logoColor=white)
---

## Sumário

* [Sobre o projeto](#sobre-o-projeto)
* [Arquitetura (Modelo C4)](#arquitetura-modelo-c4)
* [Funcionalidades](#funcionalidades)
* [Decisões técnicas](#decisões-técnicas)
* [Boas práticas adotadas](#boas-práticas-adotadas)
* [Como rodar o projeto](#como-rodar-o-projeto)
* [Variáveis de ambiente](#variáveis-de-ambiente)
* [Estrutura de pastas](#estrutura-de-pastas)
* [Exemplos de rotas](#exemplos-de-rotas)
* [Melhorias futuras](#melhorias-futuras)

---

## Sobre o projeto

O **Sistema de Tesouraria** é uma solução desenvolvida com **Java + Spring Boot**, voltada para atender às necessidades operacionais de uma tesouraria municipal.

O sistema centraliza o controle financeiro, permitindo o registro e acompanhamento de contas a pagar, gestão de fornecedores, controle de status de pagamentos e autorização de usuários com diferentes níveis de acesso.

O projeto foi concebido com foco em:

* **Confiabilidade**
* **Rastreabilidade**
* **Separação clara de responsabilidades**
* **Baixo acoplamento entre domínio e infraestrutura**

---

## Arquitetura (Modelo C4)

### C1 — Contexto

```
[Tesoureiro] ─────► Sistema de Tesouraria ◄───── [Autorizado]
                         │
                         ▼
                   PostgreSQL
```

---

### C2 — Containers

```
┌──────────────────────────────────────────────────────────┐
│                        Docker Network                    │
│                                                          │
│  ┌────────────────────────────┐   JDBC   ┌─────────────┐ │
│  │   API Backend              │◄────────►│ PostgreSQL  │ │
│  │   Spring Boot             │          │ Banco dados │ │
│  │   Java 21                 │          │ Porta 5432  │ │
│  │   Porta 8080              │          └─────────────┘ │
│  └─────────────┬─────────────┘                         │
│                │ HTTP/REST                              │
└────────────────┼────────────────────────────────────────┘
                 │
           [Cliente HTTP]
```

---

### C3 — Componentes (Arquitetura Hexagonal)

```
           ┌──────────────────────────────┐
           │        Controller            │
           │   (Adapter de Entrada)      │
           └─────────────┬────────────────┘
                         │
                         ▼
           ┌──────────────────────────────┐
           │        Use Cases             │
           │   (Application Core)         │
           └─────────────┬────────────────┘
                         │
                 ┌───────▼────────┐
                 │     Ports       │
                 │ (Interfaces)    │
                 └───────┬────────┘
                         │
           ┌─────────────▼─────────────┐
           │      Repository JPA        │
           │   (Adapter de Saída)      │
           └─────────────┬─────────────┘
                         │
                         ▼
                    PostgreSQL
```

---

### C4 — Código

```
src/main/java/com/tesouraria/
├── domain/                 # Regras de negócio puras
│   ├── model/
│   └── exception/
│
├── application/            # Casos de uso (use cases)
│   ├── usecase/
│   └── port/
│       ├── in/             # Ports de entrada
│       └── out/            # Ports de saída
│
├── infrastructure/         # Implementações externas
│   ├── persistence/
│   │   ├── entity/
│   │   ├── repository/
│   │   └── adapter/
│   │
│   ├── web/
│   │   ├── controller/
│   │   └── dto/
│   │
│   └── security/
│
└── TesourariaApplication.java
```

---

## Funcionalidades

* Autenticação com JWT
* Controle de usuários e perfis
* Cadastro de fornecedores
* Gestão de contas a pagar
* Controle de fluxo financeiro
* Relatórios

---

## Decisões técnicas

### Por que Arquitetura Hexagonal?

A arquitetura hexagonal foi adotada para garantir que o **domínio da aplicação seja independente de frameworks e tecnologias externas**.

Principais benefícios:

* O domínio não depende de banco de dados
* O domínio não depende de Spring
* Facilidade para trocar banco ou framework
* Testes mais simples (mock de ports)
* Alta manutenibilidade

### Por que Java + Spring Boot?

* Ecossistema robusto
* Forte suporte corporativo
* Integração com segurança (Spring Security)
* Facilidade para APIs REST

### Por que JPA/Hibernate?

* ORM maduro e consolidado
* Integração com Spring Data
* Redução de código boilerplate

---

## Boas práticas adotadas

* Clean Architecture (inspirada)
* Separação entre domínio, aplicação e infraestrutura
* Ports & Adapters
* DTOs para entrada/saída
* Validação com Bean Validation
* Exceptions de domínio
* Inversão de dependência (Dependency Inversion)

---

## Testes automatizados

O projeto possui testes unitários básicos para validação de algumas regras de negócio e comportamentos esperados da aplicação.

### Tecnologias utilizadas

* JUnit 5

### Abordagem

Os testes foram implementados principalmente na camada de aplicação (use cases), com foco na validação de comportamentos e regras de negócio da aplicação.

Os testes utilizam o JUnit 5 para verificar cenários simples e garantir que as funcionalidades principais estejam funcionando conforme esperado.

```java
class CriarContaUseCaseTest {

    @Test
    void deveCriarContaComSucesso() {
        // implementação do teste
    }
}
```

### Execução dos testes

```bash
./mvnw test
```

ou

```bash
./gradlew test
```

### Observações

* Os testes cobrem cenários simples e principais fluxos da aplicação
* Os testes atualmente não utilizam mocks
* Ainda não há testes de integração ou end-to-end
* O sistema está sendo desenvolvido de forma modular e incremental.  
* Cada módulo implementado passa por validação e, após aprovação, é disponibilizado em produção, permitindo entregas contínuas de valor ao cliente.  
* Os módulos iniciais possuem testes unitários básicos, e a estratégia de testes será evoluída conforme novas funcionalidades forem adicionadas, incluindo testes mais robustos e integração entre componentes.


> A arquitetura adotada facilita a evolução da testabilidade do sistema, permitindo a criação de testes mais abrangentes sem dependência de infraestrutura externa.


## Como rodar o projeto

### Pré-requisitos

* Java 21
* Docker
* Maven ou Gradle

---

### Rodando com Docker

```bash
docker compose up -d
```

A API estará disponível em:

```
http://localhost:8080
```

E o frontend em:

```
http://localhost:3000
```
---

### Rodando localmente

```bash
./mvnw spring-boot:run
```

ou

```bash
./gradlew bootRun
```

---

## Variáveis de ambiente

```env
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/tesouraria
SPRING_DATASOURCE_USERNAME=usuario
SPRING_DATASOURCE_PASSWORD=senha

JWT_SECRET=segredo-super-seguro
JWT_EXPIRATION=86400000
```

---

## Estrutura de pastas

```
domain/           → núcleo da aplicação
application/      → casos de uso
infrastructure/   → adapters e integrações externas
```

---

## Exemplos de rotas

### Login

**POST** `/api/auth/login`

```json
{
  "login": "Jane doe",
  "senha": "ABC"
}
```

---

## Melhorias futuras

* Testes com JUnit + Mockito
* CI/CD
* Observabilidade
* Auditoria
* Cache com Redis

---

<div align="center">
  Desenvolvido para gestão financeira municipal · Sistema de Tesouraria
</div>

