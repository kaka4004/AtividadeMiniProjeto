# Architectural Decision Record (ADR)

## Título: ADR 0004 - Implementação de Padrões de Resiliência no EduVerse
**Status:** Aceito  
**Data:** 01 de Junho de 2026  
**Autor:** Caua Barbosa de Alcântara  

---

### Contexto
Em uma arquitetura de microsserviços distribuídos integrada a sistemas externos (como LMS institucionais e APIs de IA), a falha de um único componente pode gerar um efeito cascata. Se o motor de IA ou o LMS externo apresentar lentidão ou ficar fora do ar durante uma avaliação, as requisições da API do EduVerse acumularão, gerando esgotamento de memória e derrubando toda a plataforma, violando o RNF de disponibilidade e tempo de resposta.

### Decisão
Decidimos implementar de forma mandatória os padrões de resiliência **Circuit Breaker (Disjuntor)**, **Fallback** e **Bulkhead (Mamparos)** através da API Gateway (AWS App Mesh / Kong):
1. **Circuit Breaker:** Se o microsserviço de IA registrar uma taxa de falhas superior a 20% ou respostas acima de 1,5 segundos em uma janela de tempo, o circuito se abre. Novas requisições não chegam ao serviço instável.
2. **Fallback:** Quando o circuito estiver aberto, a API Gateway intercepta a requisição e retorna uma resposta alternativa imediata: uma trilha de aprendizagem padrão pré-calculada armazenada em cache no **Redis**.
3. **Bulkhead:** Alocação de pools de threads e recursos isolados para cada microsserviço. Problemas de lentidão no módulo de IA não poderão consumir os recursos destinados ao módulo de login ou integração com o LMS.

### Alternativas Rejeitadas
*   **Apenas Estratégias de Retry (Reatentativas Lineares):** Rejeitado. Se um serviço está sofrendo com sobrecarga de acessos, aplicar retries automáticos e síncronos funciona como um ataque de negação de serviço (DoS) autoinfligido, piorando o estado de degradação do sistema.

### Justificativa Teórica e Trade-offs
Conforme defendido por **Michael Nygard (2018)** em *Release It!*, sistemas distribuídos modernos precisam aceitar que falhas vão acontecer e devem ser projetados para falhar de forma segura (*Design for Failure*). O padrão Circuit Breaker protege a saúde sistêmica da aplicação, impedindo o esgotamento de recursos preciosos (como sockets e conexões de banco de dados). 

O uso do Redis como estratégia de Fallback garante que, mesmo que a IA inovadora falhe temporariamente, o aluno não receba uma tela de erro (HTTP 500) no meio de seu estudo, mantendo a experiência do usuário fluida e a usabilidade estável.

**Consequências Positivas:**
*   Eliminação do efeito cascata de falhas na arquitetura.
*   Garantia da operação contínua do sistema em modo degradado (*graceful degradation*).
*   Manutenção do SLA de disponibilidade em 99,5%.

**Consequências Negativas / Trade-offs:**
*   Aumento da complexidade no desenvolvimento e nos testes de integração.
*   Durante a falha, o aluno recebe recomendações estáticas (temporariamente menos personalizadas) até que o circuito se feche novamente.