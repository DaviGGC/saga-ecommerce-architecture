# Decisões Arquiteturais e Matriz de Resiliência

## 1. Matriz de Resiliência e Idempotência
Para garantir a confiabilidade na comunicação assíncrona, as seguintes estratégias foram adotadas:

* **Idempotent Consumer:** Cada serviço valida o `transaction_id` antes de processar, garantindo que retentativas do Message Broker (At-Least-Once delivery) não resultem em dupla cobrança ou baixa em duplicidade no estoque[cite: 2].
* **Saga State Machine (Timeouts):** O Order Service escuta timeouts[cite: 2]. Se o pagamento demorar mais de 30 segundos após 3 tentativas, a Saga é abortada e a compensação (Release Stock) é acionada[cite: 2].
* **Dead-Letter Queues (DLQ):** Mensagens não processáveis (ex: erro de payload) vão para uma DLQ após as retentativas, disparando alertas para intervenção técnica[cite: 2].
* **Circuit Breaker:** Protege o Payment Service de gargalos caso o gateway financeiro externo fique instável, falhando rápido (Fail-Fast)[cite: 2].

## 2. Resolução de Risco Arquitetural: O Padrão Outbox
Para garantir que o comando de compensação (`ReleaseStock`) não se perca caso a infraestrutura do Node.js caia antes da mensagem atingir o RabbitMQ/Kafka, aplicamos o Transactional Outbox Pattern combinado com conciliação[cite: 2]:

1. O Order Service atualiza o estado do pedido para CANCELADO e, na mesma transação de banco de dados (ACID), insere a mensagem "ReleaseStock" em uma tabela relacional chamada "outbox"[cite: 2].
2. Se a aplicação sofrer um crash no meio do processo, o banco de dados executa o rollback[cite: 2]. Nenhuma mensagem incompleta vaza[cite: 2].
3. Um processo em segundo plano (CDC via Debezium ou um Worker Cron) lê a tabela "outbox" e despacha a mensagem para o Message Broker com garantia "At-Least-Once"[cite: 2].
4. **Safety Net (Reconciliação):** Como rede de segurança adicional, o Inventory Service roda uma rotina assíncrona (Cron) a cada 15 minutos expirando as reservas ("Soft Locks") órfãs que nunca receberam confirmação, garantindo que o produto volte para a vitrine do e-commerce[cite: 2].