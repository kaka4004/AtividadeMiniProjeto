# Architectural Decision Record (ADR)

## Título: ADR 0003 - Estratégia de Nuvem e Escalabilidade para o EduVerse
**Status:** Aceito  
**Data:** 01 de Junho de 2026  
**Autor:** Caua Barbosa de Alcântara  

---

### Contexto
O EduVerse precisa suportar uma carga sazonal agressiva, com picos de até 5.000 usuários simultâneos durante períodos de avaliações acadêmicas. O sistema atual necessita de garantias de disponibilidade (99,5% de uptime) e tempo de resposta inferior a 2 segundos. Precisávamos decidir entre uma abordagem de Infraestrutura como Serviço (IaaS), Plataforma como Serviço (PaaS) ou Serverless para a implantação dos microsserviços e do motor de IA.

### Decisão
Decidimos adotar uma abordagem híbrida focada em **PaaS** e **Serverless** utilizando a nuvem da **AWS (Amazon Web Services)**:
1. **PaaS (AWS ECS com AWS Fargate):** Para a hospedagem das APIs principais (FastAPI e Node.js) dentro de containers Docker, sem a necessidade de gerenciar instâncias de servidores (Serverless Container).
2. **Serverless (AWS Lambda):** Para o processamento sob demanda do motor de recomendação de Inteligência Artificial em Python.
3. **Escalabilidade Horizontal Automatizada (Auto Scaling):** Configurada para disparar com base no consumo de CPU (>70%) e volume de conexões simultâneas, adicionando novos containers de forma elástica.

### Alternativas Rejeitadas
*   **IaaS puro (AWS EC2 / Máquinas Virtuais tradicionais):** Rejeitado devido ao alto custo operacional de configuração, manutenção de patches de segurança e lentidão no tempo de reação do Auto Scaling (subir uma VM nova demora minutos, enquanto um container Fargate ou uma função Lambda sobem em segundos).

### Justificativa Teórica e Trade-offs
Segundo **Bass, Clements e Kazman (2012)** em *Software Architecture in Practice*, a escalabilidade horizontal (adicionar mais instâncias idênticas de um componente) é a tática de desempenho mais eficiente para lidar com picos imprevisíveis de tráfego, superando as limitações físicas da escalabilidade vertical (aumento de hardware em uma única máquina). 

Além disso, a arquitetura Serverless (AWS Lambda) mitiga o custo ocioso da infraestrutura. O motor de IA consome muitos recursos computacionais; mantê-lo rodando em servidores dedicados 24/7 geraria custos proibitivos. Com Serverless, o cliente paga apenas pelos milissegundos em que o algoritmo de IA está gerando recomendações aos alunos.

**Consequências Positivas:**
*   Alta elasticidade para suportar mais de 5.000 usuários simultâneos.
*   Redução drástica do custo operacional de infraestrutura (NoOps).
*   Isolamento completo do processamento pesado de IA.

**Consequências Negativas / Trade-offs:**
*   *Cold Start* (atraso inicial) nas funções AWS Lambda quando não são invocadas frequentemente, o que requer otimização do tamanho do pacote de código da IA.
*   Dependência do ecossistema do provedor de nuvem (*Vendor Lock-in* da AWS).