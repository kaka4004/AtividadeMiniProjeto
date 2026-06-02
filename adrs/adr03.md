# Architectural Decision Record (ADR)

## Título: ADR 0005 - Modelo de Comunicação Síncrono e Assíncrono no EduVerse
**Status:** Aceito  
**Data:** 01 de Junho de 2026  
**Autor:** Caua Barbosa de Alcântara  

---

### Contexto
O EduVerse lida com duas naturezas distintas de operações: interações que exigem respostas imediatas (ex: autenticação do aluno, navegação de telas, visualização de notas) e processamentos pesados de longa duração (ex: análise de performance do aluno por IA, processamento de grandes volumes de dados acadêmicos enviados pelo LMS). Utilizar o mesmo modelo de comunicação para ambos os cenários comprometeria severamente o desempenho global do sistema.

### Decisão
Decidimos adotar um **modelo de comunicação híbrido**, separando de forma clara as interações síncronas das assíncronas:
1. **Comunicação Síncrona (HTTP/REST):** Utilizada estritamente entre o Frontend (React Web App) e o API Gateway, e em rotas críticas onde a resposta imediata é um requisito de negócio inescapável (como login e envio de respostas diretas de questões).
2. **Comunicação Assíncrona Baseada em Eventos (RabbitMQ):** Utilizada para toda a comunicação que envolve o processamento de IA e integrações densas. Quando o aluno conclui um bloco de atividades, a API publica um evento `AtividadeConcluidaEvent` no **RabbitMQ** e libera o usuário imediatamente. O motor de IA (Python Worker) consome essa mensagem da fila em segundo plano, processa a nova trilha de aprendizado e atualiza a base de dados.

### Alternativas Rejeitadas
*   **Comunicação Totalmente Síncrona (Cadeia de chamadas HTTP REST ponta a ponta):** Rejeitado. Se a API de backend dependesse de esperar síncronamente o processamento do algoritmo de IA terminar para devolver a resposta ao cliente, o tempo de resposta estouraria com facilidade os 2 segundos exigidos pelo RNF, gerando *timeouts* generalizados na aplicação.

### Justificativa Teórica e Trade-offs
Segundo os conceitos de **Richards e Ford (2020)** em *Fundamentals of Software Architecture*, o desacoplamento temporal fornecido por arquiteturas orientadas a eventos (*Event-Driven Architecture*) é um fator crucial para alcançar alta performance e escalabilidade. O uso do broker **RabbitMQ** atua como um amortecedor (*buffer*) de requisições. Se 5.000 alunos enviarem atividades simultaneamente, o sistema não cai; as requisições são enfileiradas com segurança, protegendo o motor de IA e garantindo que o backend principal responda de imediato ao usuário final.

**Consequências Positivas:**
*   Tempo de resposta ao usuário final mantido drasticamente abaixo do limite de 2 segundos.
*   Desacoplamento total entre os microsserviços (o backend não precisa saber se o motor de IA está online ou lento para aceitar as requisições dos alunos).
*   Garantia de entrega de mensagens e persistência de dados em caso de quedas temporárias de componentes internos.

**Consequências Negativas / Trade-offs:**
*   **Consistência Eventual:** Os dados de recomendação podem demorar alguns poucos segundos para aparecerem atualizados na tela do aluno após o término de um teste (o sistema passa a ser eventualmente consistente).
*   Complexidade acrescida no rastreamento e monitoramento de transações distribuídas (necessidade de IDs de correlação para tracing).