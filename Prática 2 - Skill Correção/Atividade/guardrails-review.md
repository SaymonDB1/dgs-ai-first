# Revisão dos Guardrails do Assistente NovaTech (v1)

24/09/2026 · Product Specialist · revisão de `guardrails-v1.md`

## Resumo

A v1 tinha 50 guardrails e cobria todos os temas exigidos, mas apresentava quatro tipos de problema:

- **Genéricos ou redundantes:** 3 regras eram requisitos de pipeline e não comportamento de domínio, e 1 duplicava outra.
- **Enforcement fraco:** 11 regras estavam só em PROMPT, embora tivessem uma verificação determinística viável.
- **Enforcement de código mal desenhado:** 3 regras poderiam bloquear respostas corretas ou deixar passar respostas erradas.
- **Riscos sem cobertura:** 8 riscos do domínio não tinham guardrail. Cada um ganhou um incidente novo (INC-19 a INC-26), extraído do FAQ, do recorte de domínio, do Mapa de Riscos e da jornada.

A revisão também encontrou uma divergência entre a tabela de áreas do `05` e a POL-001 §3.2. Ela é corrigida nos guardrails e fica registrada para o `05`.

A versão final (`guardrails-final.md`) tem 61 guardrails: 23 DEVE, 19 NÃO DEVE e 19 QUANDO EM DÚVIDA. São 26 incidentes, todos cobertos, e nenhuma regra ficou só em PROMPT quando havia verificação determinística possível.

## 1. Regras genéricas ou redundantes

| ID v1 | Problema | Decisão na versão final |
| --- | --- | --- |
| DV-17 | "Cada resposta traz suas próprias citações" vale para qualquer RAG. Não diz o que muda no Teams para o domínio NovaTech: o reaproveitamento de dados do caso (data de abertura, tier, peso, região) entre mensagens. | Reescrita: dado do caso só é reaproveitado dentro da janela do pedido de informação (C-43), e valor de regra dito antes é citado de novo. |
| QD-16 | Orçamento de contexto é requisito técnico do pipeline (C-31, ADR-0002), não comportamento que o atendente percebe. | Retirada dos guardrails; continua na C-31 do requirements. |
| QD-13 | Fallback por tempo é genérico, e o limite de 30 s ainda não tem origem confirmada (DT-17). | Mantida, porque protege contra "prazo dito às pressas", com nota sobre a DT-17. |
| ND-14 | Duplicava o DV-02: ambos proíbem citar chunk ou resumo. | Incorporada ao DV-02; o ID ND-14 foi retirado. |

## 2. Regras que deveriam ter enforcement por código

Todas passam de PROMPT para PROMPT + CÓDIGO na versão final.

| ID | Por que só prompt é insuficiente | Verificação determinística proposta |
| --- | --- | --- |
| ND-04 | Classificar a classe 8 como "não perigosa" é o erro mais provável de um LLM treinado na norma da ANTT. | Detectar classe 7, 8 ou 9 ou produto perigoso sem classe; bloquear "está incluída", "está excluída", "não é considerada perigosa"; forçar o alerta. |
| ND-10 | SLA confundido com prazo de entrega é a interpretação incorreta mais citada no recorte (T-19). | Rejeitar parte classificada como prazo de entrega (BC-06) que cite o SLA-2024; bloquear "entrega mais rápida" junto de nome de tier. |
| ND-11 | Classificar incidente crítico é decisão do atendente (C-59). | Bloquear "é um incidente crítico", "classificado como incidente crítico"; permitir "os dados atendem ao critério". |
| ND-12 | Não há integração com o tracking; qualquer previsão de entrega é dedução. | Bloquear "provavelmente em trânsito", "deve chegar em", "previsão de entrega é" em qualquer resposta. |
| ND-13 | A fusão dos três reembolsos é o erro do exemplo da própria JORNADA. | Bloquear "reembolso do frete" e "direito a reembolso" fora de parte de devolução com trecho que contenha "reembolso"; bloquear "crédito" associado a "atraso". |
| QD-02 | O gatilho é objetivo: peso igual a 500 kg. | Detectar 500 kg na dúvida e exigir os dois trechos (§1 e §2) e a área Comercial. |
| QD-03 | O gatilho é objetivo: menção a peso cubado ou cubagem. | Detectar o termo e exigir o bloco de lacuna; bloquear fator de peso sem esse bloco. |
| QD-05 | A regra do título neutro (C-54) era só instrução. | Verificar que a primeira frase de parte S4 não contém número presente no trecho do FAQ. |
| QD-09 | Divergência não registrada depende de o LLM perceber. | Comparar valores numéricos do mesmo parâmetro em trechos de documentos diferentes; valor diferente fora do registro gera "Possível divergência". |
| QD-10 | Mesmo risco do ND-11. | Mesma lista de bloqueio do ND-11. |
| QD-20 (novo) | Frete reverso depende da versão do frete original. | Exigir a data de referência do frete original antes de liberar multiplicador. |

## 3. Enforcement de código com falha de desenho

| ID | Falha | Correção |
| --- | --- | --- |
| ND-07 | A lista de bloqueio incluía "aprovado". Ela bloquearia a resposta correta sobre carga acima de 5.000 kg ("requer aprovação prévia do gerente de operações regional"). | A lista passa a conter só frases de concessão: "tem direito a desconto", "desconto automático", "está aprovado", "fica concedido", "concedo". |
| ND-01 e DV-09 | "Bloqueia valor em R$" bloquearia valores legítimos citados nos documentos: R$ 100.000 (SLA-2024 §3), R$ 500.000 (SLA-2024 §1), R$ 50.000 (FAQ #27). | O bloqueio passa a valer para valor em R$ que não aparece literalmente num trecho citado. |
| DV-14 | A lista de áreas não cobria as exceções de devolução da POL-001 §3.2 além da carga perigosa. | A área de cadeia de frio rompida e lacre violado passa a ser a Gestão de Riscos (ver seção 5). |

## 4. Incidentes com cobertura fraca ou sem cobertura

Os 18 incidentes da v1 tinham pelo menos um guardrail. Dois estavam mal cobertos:

- **INC-12:** tratava a ingestão manual, mas não o chunk que corta tabela. Novo DV-19: citação de tabela sempre por célula.
- **INC-18:** coberto só por regras PROMPT. QD-02 e QD-03 ganham código.

Oito riscos do domínio não tinham incidente nem guardrail. Os incidentes novos vêm de materiais já recebidos:

| Incidente novo | Fonte | Guardrail final |
| --- | --- | --- |
| INC-19 · Atendente promete exceção de devolução de carga perigosa porque "Riscos já autorizou" | FAQ #3; jornada da Tarefa 1, exemplo do F4 | ND-17 |
| INC-20 · Resposta automática "estamos verificando" contada como primeira resposta de SLA | FAQ #41; `05` T-20 | ND-18 |
| INC-21 · Crédito de penalidade oferecido já na primeira violação ou como compensação por atraso de entrega | `05` T-23 | DV-23 |
| INC-22 · Devolução de carga refrigerada aceita pelo relato do cliente, sem sensor, e lacre violado recusado sem checar a documentação da entrega | Mapa de Riscos R07; `05` T-33, T-34 | DV-20 |
| INC-23 · Embarque acima de 5.000 kg confirmado sem mencionar a aprovação prévia | Mapa de Riscos R15; `05` T-17 | DV-21 |
| INC-24 · Prazo de devolução contado da data informada pelo cliente ou da nota fiscal, não da data confirmada no tracking | Mapa de Riscos R12; `05` T-29 | DV-22, QD-18 |
| INC-25 · Consulta enviada com CPF e endereço do destinatário copiados do chamado | Jornada da Tarefa 1, P2; JORNADA G6 | ND-20 |
| INC-26 · SLA-2024 sinalizado como desatualizado só pelo nome | Jornada da Tarefa 1, G5 (superado); Mapa de Riscos R11 | ND-19 |

## 5. Riscos do domínio ainda não tratados na v1

| Risco | Evidência no Anexo A | Tratamento na versão final |
| --- | --- | --- |
| Exceções de devolução além da carga perigosa (cadeia de frio, lacre violado) sem critério exato | POL-001 §3.2: "mais de 30 minutos contínuos, conforme registro do sensor IoT"; lacre "salvo quando a violação for documentada no ato de entrega" | DV-20 |
| Área errada para essas exceções | POL-001 §3.2 manda as três categorias para a Gestão de Riscos, ramal 4500. O `05` §2.4 (linha 10) as envia a Operações. | DV-20 e DV-14 usam Gestão de Riscos. **Corrigir a linha 10 do `05` §2.4.** |
| Prazo expirado recusado em vez de encaminhado | POL-001 §3.5: "Encaminhar ao Comercial para negociação caso a caso" | DV-20 |
| Carga acima de 5.000 kg sem menção à aprovação | PROC-042 v1 §4 e v2 §4, iguais nas duas versões | DV-21 |
| Marco errado do prazo de devolução | POL-001 §3.1: "data de recebimento confirmada no sistema de tracking" | DV-22 |
| Penalidade de SLA aplicada fora da contagem mensal | SLA-2024 §4 | DV-23 |
| Primeira resposta definida pelo FAQ | FAQ #41; SLA-2024 não define o termo | ND-18 |
| Aritmética ambígua do desconto e contagem "para o mesmo cliente" | PROC-042 v2 §4: "5% sobre o multiplicador regional" | QD-19 |
| Feriados regionais e horas úteis | POL-001 §3.1 cita só feriados nacionais; SLA-2024 §5 define horário comercial | QD-18 |
| Frete reverso sem versão definida | POL-001 §3.5: "mesmos multiplicadores do frete original" | QD-20 |
| Citação de PROC-042 sem versão | Chunk PROC-042-A do Anexo B identifica só "versão original" | QD-17 |
| Dados pessoais na consulta | JORNADA G6 | ND-20 (na v1, deixado fora por falta de incidente) |

## 6. Mudanças da v1 para a versão final

| Tipo | IDs |
| --- | --- |
| Reescritos | DV-02 (absorve ND-14), DV-09, DV-10, DV-14, DV-17, ND-01, ND-07 |
| Enforcement elevado para PROMPT + CÓDIGO | ND-04, ND-10, ND-11, ND-12, ND-13, QD-02, QD-03, QD-05, QD-09, QD-10 |
| Novos | DV-19 a DV-23, ND-17 a ND-20, QD-17 a QD-20 |
| Retirados | ND-14 (incorporado ao DV-02), QD-16 (requisito técnico, C-31) |
| Incidentes novos | INC-19 a INC-26 |

## 7. Pendências fora dos guardrails

- [ ] Corrigir o `05` §2.4, linha 10: cadeia de frio rompida e lacre violado vão à Gestão de Riscos (POL-001 §3.2), não a Operações.
- [ ] Confirmar a origem do limite de 30 s (DT-17), usado no QD-13.
- [ ] Decidir a área de carga perigosa no frete (DT-10), usada em DV-10 e QD-11.
- [ ] Remapear os incidentes se houver uma lista oficial diferente deste catálogo.
