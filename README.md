# Blueprint de Arquitetura: Padrão Saga no E-commerce

## 1. Contexto de Negócios e Decisão Arquitetural
A arquitetura adota o Padrão Saga via Orquestração para resolver o fluxo de checkout distribuído entre Order Service, Inventory Service e Payment Service[cite: 2]. O Order Service atua como o Saga Execution Coordinator (SEC)[cite: 2]. Essa decisão reduz o acoplamento cíclico, centraliza o rastreamento do estado em caso de falha (Rollback) e facilita a adição de novos microsserviços no futuro[cite: 2].

## 2. Diagramas de Arquitetura
Os desenhos de arquitetura mapeando os fluxos síncronos e assíncronos encontram-se na pasta `/diagrams`:

* **`saga-architecture-overview.png`**: Visão geral topológica mostrando os domínios isolados com seus respectivos bancos de dados (Database-per-service) e a comunicação centralizada via Message Broker (RabbitMQ/Kafka).
* **`sequence-success-flow.png`**: Mapeamento do caminho feliz, onde o pedido é registrado, o estoque é reservado e o gateway de pagamento aprova a transação.
* **`sequence-compensation-flow.png`**: Demonstração do mecanismo de reversão. Quando o Payment Service emite um evento de recusa, o Orquestrador comanda o Inventory Service a executar a transação compensatória de `ReleaseStock` para manter a consistência eventual.