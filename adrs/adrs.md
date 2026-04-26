# ADR 002 — Adoção da Arquitetura Hexagonal no EduVerse

## Status
Aceita

---

## Contexto

Durante a modelagem inicial do sistema, foi identificado que a abordagem tradicional em camadas (N-Tier) poderia gerar alto acoplamento entre a lógica de negócio e a infraestrutura, dificultando a evolução do sistema.

Além disso, o EduVerse possui forte dependência de integrações externas (IA, LMS) e necessidade de constante evolução dos algoritmos de recomendação.

Esse cenário cria um conflito entre:

- Facilidade de implementação (N-Tier)
- Flexibilidade e manutenção a longo prazo

---

## Decisão

Adotar a Arquitetura Hexagonal (Ports and Adapters), onde:

- O domínio de negócio é isolado no centro do sistema
- As integrações externas são realizadas por meio de adaptadores
- As dependências apontam sempre para o domínio

---

## Justificativa

Essa decisão garante:

- Maior desacoplamento
- Facilidade de testes
- Flexibilidade para trocar tecnologias sem impactar o domínio

De acordo com Martin (2017), essa abordagem reduz o acoplamento e melhora a qualidade arquitetural do sistema.

---

## Consequências

### Positivas

- Melhor manutenibilidade
- Maior escalabilidade
- Facilidade de evolução do sistema

### Negativas

- Aumento da complexidade inicial
- Necessidade de maior disciplina arquitetural