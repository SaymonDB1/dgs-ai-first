# 02-requirements-v1 — Query Endpoint do Assistente NovaTech

| Item | Conteúdo |
| --- | --- |
| Artefato | Spec SDD · `requirements.md` |
| Componente | Query endpoint: recebe a dúvida do atendente e devolve a orientação fundamentada |
| Versão | 1.0 · 24/09/2026 · Rascunho para revisão com Tech Lead e QA |
| Recorte de domínio | `01-bounded-contexts-v1.md` (rev. 1.1): contexts `BC-xx` e termos `T-xx` |
| Fonte de verdade | Anexo A: `POL-001`, `PROC-042 v1`, `PROC-042 v2`, `SLA-2024`, `FAQ` |
| Spec anterior | ESPEC-V2: requisitos `FON`, `RES`, `SEN`, `ESC`, `FBK`, `RAS`, `RN`, `GR` e decisões `DP-xx` |
| Jornada | Jornada integrada do atendente: princípio central, guardrails `G1`–`G6`, fluxos `P`, `F`, `R` |

**Princípio que governa todo o documento** (`JORNADA`): *responder com base na documentação; se não houver evidência suficiente, não assumir e direcionar para validação.*

**Como ler:** os termos em negrito com ID `T-xx` têm o sentido exato definido no recorte de domínio. Por exemplo, "SLA" (`T-19`) é prazo de atendimento de chamado, nunca prazo de entrega (`T-16`).

---

## 1. Outcomes

Resultados observáveis para o atendente. Não descrevem funcionalidades técnicas.

### 1.1 Categorias de dúvida atendidas

| Categoria do cenário | Context de negócio consultado | Exemplo de dúvida |
| --- | --- | --- |
| Prazo de entrega | BC-06 Prazos, Execução e Rastreamento | "Qual o prazo de um frete especial de 1.200 kg para o Norte?" |
| Regras de frete | BC-05 Cotação de Frete Especial (+ BC-09 para desconto, BC-07 para carga perigosa) | "Qual multiplicador aplico para o Nordeste?" |
| Política de devolução | BC-10 Devolução e Logística Reversa | "Até quando o cliente pode pedir devolução?" |
| SLAs | BC-08 Atendimento, SLA e Penalidades (+ BC-09 para tier) | "Qual o prazo de primeira resposta para cliente Gold?" |

Cerca de **15% das dúvidas cruzam duas categorias**, por exemplo devolução + SLA ou frete + prazo.

### 1.2 Outcomes

| ID | Resultado para o atendente | Sinal observável |
| --- | --- | --- |
| **O-01** | O atendente recebe, **em menos de 30 segundos**, uma orientação utilizável durante o atendimento para dúvidas de prazo de entrega, frete, devolução e SLA. | Tempo entre enviar a dúvida e ver a orientação completa < 30 s, inclusive quando a orientação é um fallback. |
| **O-02** | Quando a dúvida cruza duas categorias, o atendente recebe **uma única orientação** que responde cada parte separadamente, cada uma com sua fonte e seus alertas. | Uma parte identificada por categoria; nenhuma parte some sem aviso; alertas de uma parte não contaminam a outra. |
| **O-03** | O atendente consegue **verificar sozinho de onde vem cada afirmação** (documento, seção, versão e trecho literal), sem abrir outro sistema. | Toda afirmação de regra tem citação própria e conferível no Anexo A. |
| **O-04** | O atendente **nunca recebe como regra única** algo que as fontes tratam de forma divergente, nem uma regra montada com versões misturadas do PROC-042. É avisado quando a versão aplicável depende de um dado do caso. | Divergências aparecem lado a lado; a data de abertura do chamado é pedida quando decide a versão. |
| **O-05** | Quando a documentação não sustenta a resposta, o atendente **sabe disso imediatamente**, sabe o que falta e para qual área encaminhar, em vez de receber uma resposta inventada. | Mensagem de ausência com o que falta e a área; nenhum valor, prazo ou percentual sem fonte. |
| **O-06** | O atendente **sabe quanto pode confiar** na orientação e **quando precisa validar** antes de falar com o cliente: tema sensível, carga perigosa, fonte informal. | Nível de confiança e alerta de validação humana sempre visíveis. |
| **O-07** | Quando a orientação exige uma pessoa ou está errada, o atendente **tem um caminho claro**: indicação de escalonamento com motivo e área, e uma referência da orientação para contestar ou dar feedback. | Escalonamento indicado nos casos previstos; toda orientação tem identificador rastreável. |

---

## 2. Scope Boundaries

Derivados dos bounded contexts de `01-bounded-contexts-v1.md`.

### 2.1 Coberto diretamente

| Context | O que o endpoint realiza |
| --- | --- |
| **BC-01 Orientação Fundamentada ao Atendente** (core) | Entende a dúvida e separa as partes; identifica o context de cada parte; pede o dado do caso que falta; classifica a situação (S1–S8); atribui confiança; cita as fontes; identifica tema sensível e conflito; aplica o fallback sem suposições; devolve a orientação ao atendente. |

### 2.2 Coberto na fronteira

O endpoint produz o sinal, mas não executa o processo do context.

| Context | O que o endpoint faz | O que fica com o context |
| --- | --- | --- |
| **BC-03 Validação Humana e Escalonamento** | Indica que a orientação exige validação ou escalonamento, com motivo, tema e área de destino; registra o caso. | Roteamento efetivo, canal (DP-09), validação pelo especialista, resposta validada. |
| **BC-04 Feedback e Melhoria Contínua** | Devolve um identificador rastreável de cada orientação e registra pergunta, orientação, fontes e versões (`RAS-02`), para que feedback e contestação possam ser vinculados. | Captura do feedback, ciclo de vida (`FBK-03`), correção da documentação. |

### 2.3 Consultados (somente leitura)

O endpoint lê regras e metadados desses contexts; não cria, altera nem decide regra.

| Context | O que é consultado | Restrição de consulta |
| --- | --- | --- |
| **BC-02 Governança do Conhecimento** | Status, versão, vigência, classificação formal/informal, hierarquia, conflitos registrados, documentos ausentes | Conformist: aceita a classificação sem reinterpretar |
| **BC-05 Cotação de Frete Especial** | Enquadramento > 500 kg, fórmula, valor base, multiplicadores, fatores de peso (`PROC-042 v1` e `v2`) | Não entrega valor final enquanto DP-14 estiver aberta; valor base ausente |
| **BC-06 Prazos, Execução e Rastreamento** | Adicional +2/+3 dias úteis, aprovação > 5.000 kg, definição de data de recebimento | Não informa status real de carga (`G2`); prazo padrão da rota ausente |
| **BC-08 Atendimento, SLA e Penalidades** | Prazos por tier, incidente crítico, pausa do relógio, penalidades (`SLA-2024`) | — |
| **BC-09 Cliente, Contrato e Condições Comerciais** | Critérios de tier, inexistência de outros tiers, regras de desconto | Não concede desconto nem classifica o cliente num tier: aplica o critério ao dado informado |
| **BC-10 Devolução e Logística Reversa** | Prazo, procedimento, exceções, devolução parcial, custos (`POL-001`) | Não aprova devolução nem exceção |
| **BC-07 Cargas Perigosas e Conformidade** | Classes 1 a 6 da ANTT, remissão à PROC-043, encaminhamento à Gestão de Riscos | Sempre com validação humana; nunca resposta definitiva |
| **BC-11 Avarias, Sinistros e Seguro** | FAQ #22 e #38 (única fonte) | Sempre baixa evidência (S4) e validação humana |

### 2.4 Fora do escopo

| Item | Por quê |
| --- | --- |
| Curadoria, publicação, classificação e retirada de documentos (escrita no BC-02) | Pertence ao BC-02; o endpoint só lê |
| Ciclo de vida do feedback e correção da documentação (BC-04) | Fluxo próprio; o endpoint só fornece o vínculo |
| Execução do escalonamento e validação humana (BC-03) | Mecanismo em DP-09 |
| Status real, localização e previsão de carga | Vem do tracking oficial (`G2`, BC-06); não é consulta documental |
| Conceder desconto, aprovar devolução ou exceção, autorizar embarque ou carga perigosa | `RN-12`: o assistente informa e orienta |
| Frete padrão (< 500 kg), interceptação de carga em trânsito (PROC-088), tarifa de carga perigosa (PROC-043), tabela mensal, prazo padrão por rota | Citados, mas ausentes do corpus (`T-06`, `T-36`, `T-27`, `T-09`, `T-15`); tratados como ausência, nunca inferidos |
| Comunicação direta com o cliente final | O atendente decide o que comunicar (DP-10) |
| Arquitetura, modelo, chunking, embeddings, formato de payload | Decisão técnica (`plan.md`), fora do `requirements.md` |

---

## 3. Constraints

### 3.1 Fontes

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-01** | O corpus de produção é composto apenas pelos 5 documentos do Anexo A. O Anexo B é massa de teste e não pode sustentar resposta em produção. | Anexo A; ESPEC-V2 Premissas; DP-03 |
| **C-02** | Classificação de autoridade: `POL-001` normativo, `SLA-2024` contratual, `PROC-042 v1` e `v2` formais sem vigência declarada, `FAQ` informal sem responsável. | Anexo A (cabeçalhos) |
| **C-03** | Premissa P-01: `POL-001` e `SLA-2024` são tratados como formais vigentes; `PROC-042 v1` e `v2` como formais sem vigência confirmada. A premissa deve ser confirmada pelo BC-02. | ESPEC-V2 FON-01, DP-18 |
| **C-04** | Fonte informal nunca substitui nem prevalece sobre fonte formal, mesmo quando cita um procedimento formal. | RN-05, GR-13 |
| **C-05** | Conhecimento geral do modelo, legislação externa e web não completam, substituem nem contradizem regra interna. Até DP-13, só a base autorizada é usada. | RES-11, RN-07, GR-11 |
| **C-06** | As respostas usam as definições da linguagem ubíqua. Exemplos: "Gold" é tier do `SLA-2024 §1` (`T-03`); "carga perigosa" são as classes 1 a 6 da ANTT (`T-26`); "frete especial" é acima de 500 kg (`T-05`); "devolução" conta 7 dias úteis, não corridos (`T-28`). | `01-bounded-contexts-v1.md` |

### 3.2 Documentos contraditórios

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-07** | Divergência entre fontes aplicáveis nunca é ocultada. Com precedência aplicável → S5 (regra prevalente + divergência). Sem precedência → S6 (fontes lado a lado, sem resposta definitiva, com escalonamento). | RES-08, RN-06, GR-05 |
| **C-08** | Até a decisão da DP-01, conflito entre fontes **formais** é sempre tratado como não resolvido (S6). Conflito formal × FAQ é resolvido pela C-04 (S5). | FON-03 |
| **C-09** | Conflitos conhecidos exibem alerta mesmo quando só uma das fontes foi recuperada. Estão registrados: PROC-042 v1 × v2 (multiplicadores, fator de peso, prazo adicional, desconto); PROC-042 × FAQ #45 (desconto); POL-001 §3.5 × FAQ #38 (avaria); FAQ #32 sem fonte formal. | FON-07; Anexo A · Notas; `T-37` |

### 3.3 Versionamento

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-10** | A resposta nunca combina parâmetros da v1 e da v2 do PROC-042 numa mesma regra ou cálculo. | RES-09, GR-12 |
| **C-11** | Para **multiplicadores regionais**, a versão é escolhida pela regra de transição da `PROC-042 v2 §5`, usando a data de abertura do chamado: antes de 01/12/2023 e ainda em processamento → v1; a partir de 01/12/2023 → v2. A coexistência com a outra versão é sempre sinalizada. Confiança máxima: Média. | PROC-042 v2 §5; FON-04; `T-12` |
| **C-12** | Para **fator de peso, prazo adicional e desconto por volume**, não há critério documental de versão. Esses parâmetros seguem S6 até a decisão da DP-02. | FON-04; `T-11`, `T-13`, `T-16` |
| **C-13** | Versão mais recente, nome, ano ou número de versão, isoladamente, não confirmam vigência. | RN-03, RN-04 |
| **C-14** | Se a versão depende de um dado não informado (data de abertura do chamado), o endpoint pede o dado (S7). Se o atendente não tiver o dado, apresenta a regra de forma condicional, apenas com as condições do documento. | RES-07 |

### 3.4 Ausência de resposta

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-15** | Sem evidência → S3: mensagem de ausência, o que falta e área a consultar. Nenhum valor, prazo, percentual ou procedimento é estimado. | RES-05, GR-01, GR-02, `G1` |
| **C-16** | Documento referenciado e ausente (PROC-043, PROC-088, tabela mensal, prazo padrão por rota) → o endpoint informa o documento faltante, responde só a parte independente (S2) e escala se a dependência for bloqueante. | FON-08 |
| **C-17** | Com parâmetro obrigatório ausente, o endpoint mostra a fórmula e o parâmetro faltante e não calcula. | RES-10, GR-14 |

### 3.5 Idioma

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-18** | A orientação é redigida em português do Brasil, idioma do corpus. Termos da linguagem ubíqua e siglas do Anexo A são mantidos como no documento (Gold, Silver, Standard, CT-e, ANTT, PROC-042). | Anexo A; `01-bounded-contexts-v1.md` |
| **C-19** | O trecho de evidência é reproduzido literalmente no idioma original, sem tradução nem paráfrase. O comportamento para dúvidas em outro idioma depende da DP-24. | RES-03; DP-24 |

### 3.6 Citações

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-20** | Cada afirmação de regra (valor, prazo, percentual, critério, procedimento, elegibilidade) tem citação própria com documento, seção, versão, trecho literal e data de atualização da fonte. | RES-02, RES-03, `G3` |
| **C-21** | Afirmação que não pode ser citada não é feita. | RES-03, GR-16 |
| **C-22** | Fonte informal é identificada como informal no próprio bloco em que aparece. | GR-04, RES-02 (S4) |

### 3.7 Baixa confiança e temas sensíveis

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-23** | A confiança reflete a qualidade da evidência. Alta: formal vigente, explícita, sem conflito. Média: formal sem vigência confirmada, interpretação permitida ou conflito resolvido. Baixa: só informal ou incompleta. Não se aplica: S3, S6, S7, S8. | RES-04 |
| **C-24** | O nível exibido é o menor entre os blocos da resposta. Somente FAQ nunca recebe Alta nem Média. Fonte com conflito registrado nunca recebe Alta. | RES-04, GR-15 |
| **C-25** | Tema sensível sustentado só por fonte informal nunca recebe resposta definitiva e é sempre escalado. | SEN-02 |
| **C-26** | Carga perigosa sempre exibe "Requer validação humana", independentemente da qualidade da evidência. Os demais temas sensíveis do baseline também exibem o alerta: avaria, extravio, indenização, reclamação formal, exceção contratual, desconto, seguro, exceção de devolução, incidente crítico. O nível exato por tema depende da DP-08. | SEN-01, SEN-02, `G5` |
| **C-27** | O endpoint não autoriza, concede nem aprova nada; pedido de ação é S8. | RN-12, RES-12 |
| **C-28** | A orientação não reproduz dados pessoais presentes na dúvida (CPF, endereço, dados do destinatário). | `G6`; SEG-05; DP-17 |

### 3.8 Tempo de resposta e contexto

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-29** | A orientação completa (texto, citações, confiança e alertas) chega ao atendente em **menos de 30 segundos** a partir do envio da dúvida. O limite vale para todas as situações, inclusive dúvidas de duas categorias e fallbacks. | Cenário; RNF-01 |
| **C-30** | Se a orientação fundamentada não puder ser concluída dentro do limite, o endpoint devolve um fallback antes dos 30 s (parcial ou indisponível) e nunca uma regra sem citação. | C-29, C-21; RNF-06 |
| **C-31** | Orçamento de contexto por consulta conforme **ADR-0002**: cerca de 4K tokens para instruções de sistema e cerca de 8K tokens para trechos recuperados. Numa dúvida de duas categorias, os trechos das duas partes dividem o mesmo orçamento. Se não couberem, a parte não coberta é declarada (S2); citações nunca são truncadas. | ADR-0002 (ver Prior Decisions) |
| **C-32** | Alertas, confiança e necessidade de validação ficam visíveis sem ação adicional do atendente. | RNF-05; RES-02 |

---

## 4. Prior Decisions

### 4.1 ADRs

| ADR | Decisão | Efeito neste endpoint | Observação |
| --- | --- | --- | --- |
| **ADR-0002** | Context budget: cerca de 4K tokens de system e cerca de 8K tokens de chunks por query | C-31; priorização de evidência formal dentro do orçamento; VC-10 | O documento do ADR não foi enviado. A decisão é referenciada na rubrica do Tech Lead (`avaliacao-tech-lead.md`). Confirmar valores e enunciado com o Tech Lead. |

Nenhum outro ADR foi fornecido. As rubricas citam o diretório `/docs/adr/`, mas seu conteúdo não está nos arquivos enviados. Nenhum ID de ADR foi criado aqui.

### 4.2 Decisões da fase anterior, por tema

| Tema | Decisão | Origem | Efeito neste endpoint |
| --- | --- | --- | --- |
| **RAG** | O assistente responde a partir de documentação recuperada da base autorizada, sem completar com conhecimento geral. | ESPEC-V2 §1, RES-11; `JORNADA` princípio central | C-05, C-15 |
| **RAG** | O Anexo A disponibiliza os 5 documentos como arquivos individuais para ingestão; o Anexo B é massa de teste. | Anexo A (nota de ingestão); ESPEC-V2 Premissas | C-01 |
| **RAG** | Documentos formais são recuperados antes do FAQ; o atendente é avisado quando a resposta vem de fonte informal. | `Reflexao_Progressive_Disclosure` §6; FON-03 | C-04, C-22 |
| **RAG** | Estratégia de chunking, embeddings e score de similaridade ficam fora da spec de produto. | ESPEC-V2 §2.2 | Scope 2.4 |
| **Context budget** | ~4K system + ~8K chunks por query. | ADR-0002 | C-31 |
| **Fontes** | Hierarquia baseline: formal vigente > formal sem vigência > FAQ. Definitiva em DP-01. | FON-03 | C-08 |
| **Fontes** | Disponibilidade na base não significa oficialidade; cada fonte tem metadados de versão, vigência e responsável. | RN-02, FON-01 | C-02, C-03 |
| **Fontes** | A PROC-042 v2 existe na fonte de verdade; a correção anterior que a dava como inexistente está superada. | `01-bounded-contexts-v1.md` rev. 1; Anexo A | C-10 a C-12 |
| **Documentos contraditórios** | Conflito sem precedência é apresentado e escalado, nunca decidido pelo assistente. | RN-06, RES-08 | C-07, C-08 |
| **Documentos contraditórios** | Registro de conflitos conhecidos, com alerta independente da recuperação. | FON-07 | C-09 |
| **Documentos contraditórios** | Não combinar versões. | RES-09, GR-12 | C-10 |
| **Ausência de resposta** | Situações S1–S8 e mensagens padrão de fallback. | RES-01; ESPEC-V2 §13 | C-15, C-16 |
| **Ausência de resposta** | Fallback sem suposições: não criar, informar ausência, mostrar o encontrado, indicar validação. | `JORNADA` F1 | C-15 |
| **Atualização da base** | A orientação mostra a data de atualização da fonte; fonte com falha de atualização não recebe Alta; base indisponível → modo indisponível. | FON-10 | C-20, C-30; VC-35 |
| **Atualização da base** | Nova versão só entra após publicação; a anterior vira histórica sem apagar o registro; SLAs de atualização em DP-05 e DP-06. | FON-05, FON-09; ESPEC-V2 §5.1 | Scope 2.4 (BC-02) |
| **Atualização da base** | Lacunas do fallback viram pendências de conteúdo; documentos vencidos saem da busca. | `JORNADA` R0, R6 | Scope 2.2 (BC-04) |

### 4.3 Decisões pendentes que afetam o endpoint

Enquanto a decisão não for tomada, vale o comportamento provisório definido na própria ESPEC-V2.

| DP | Assunto | Comportamento provisório neste endpoint |
| --- | --- | --- |
| DP-01 | Hierarquia documental | Conflito entre formais = S6 (C-08) |
| DP-02 | Aplicabilidade do PROC-042 | Transição da v2 §5 só para multiplicadores; demais parâmetros S6 (C-11, C-12) |
| DP-08 | Temas sensíveis e níveis N1/N2/N3 | Alerta de validação humana em todo o baseline (C-26) |
| DP-09 | Mecanismo de escalonamento | O endpoint indica motivo e área; a execução fica fora do escopo |
| DP-12 | Data de referência padrão | Para PROC-042, usar a data de abertura do chamado; pedir quando faltar (C-14) |
| DP-13 | Conhecimento externo | Proibido (C-05) |
| DP-14 | Entrega de valor final calculado | Não entregar valor final; a tabela mensal está ausente, de qualquer forma (C-17) |
| DP-15 | Uso permitido por nível de confiança | Exibir o nível; o uso é orientado pelo alerta |
| DP-18 | Metadados obrigatórios | Premissa P-01 (C-03) |
| DP-24 | Tempo, canal, idioma | Tempo fixado pelo cenário em < 30 s (C-29); idioma PT-BR (C-18) |
| DP-29 | Contexto entre turnos | Cada orientação cumpre C-20 por si só |
| DP-31 | Pergunta fora do domínio | Mensagem "atendo apenas temas da documentação NovaTech" |

---

## 5. Verification Criteria

Todos os critérios são binários. Salvo indicação, "citação" significa documento + seção + versão + trecho literal que existe, caractere a caractere, no documento citado do Anexo A.

### 5.1 Resposta normal

- **VC-01** — Dado a dúvida "Até quando o cliente pode solicitar a devolução?", quando enviada ao endpoint, então a orientação informa "7 dias úteis após a data de recebimento confirmada no sistema de tracking", cita `POL-001 §3.1 v3.1`, é classificada como S1 e exibe confiança Alta (premissa C-03).
- **VC-02** — Dado a dúvida "Qual o prazo de primeira resposta de um chamado geral para cliente Gold?", quando enviada, então a orientação informa "até 2h úteis", cita `SLA-2024 §2 v2024.1` e não menciona prazo de entrega de carga.
- **VC-03** — Dado a dúvida "Carga de 6.000 kg em frete especial precisa de aprovação?", quando enviada, então a orientação informa a aprovação prévia do gerente de operações regional, cita `PROC-042 v1 §4` e `PROC-042 v2 §4`, e não sinaliza conflito para esse ponto, porque as duas versões coincidem.

### 5.2 Linguagem ubíqua

- **VC-04** — Dado a dúvida "Cliente diz ser Platinum; qual o SLA dele?", quando enviada, então a orientação afirma que só existem os tiers Gold, Silver e Standard, cita a nota do `SLA-2024 §1` e não atribui nenhum prazo de SLA a "Platinum".
- **VC-05** — Dado a dúvida "Cliente com contrato anual de R$ 600.000 e 30 operações/mês é Gold?", quando enviada, então a orientação responde que atende ao critério Gold, porque o critério é "OU", cita `SLA-2024 §1`, identifica a interpretação permitida e exibe confiança Média.
- **VC-06** — Dado a dúvida "Cliente Gold tem prazo de entrega menor?", quando enviada, então a orientação não afirma redução de prazo de entrega para Gold e informa que o `SLA-2024` trata de prazos de atendimento de chamados.
- **VC-07** — Dado a dúvida "Uma carga de exatamente 500 kg é frete especial?", quando enviada, então a orientação exibe o trecho "acima de 500kg" (`PROC-042 §1`) e o trecho "de 500kg a 1.000kg" (`§2`), não responde "sim" nem "não" como regra e indica validação com o Comercial.

### 5.3 Pergunta envolvendo dois contexts

- **VC-08** — Dado a dúvida "Cliente Gold quer devolver uma mercadoria; qual o prazo de triagem da devolução e qual o prazo de primeira resposta do chamado?", quando enviada, então a orientação apresenta duas partes identificadas: devolução, com "4 horas úteis" citando `POL-001 §3.3`; e SLA, com "até 2h úteis" citando `SLA-2024 §2`. Nenhuma frase afirma que um prazo substitui ou inclui o outro.
- **VC-09** — Dado a dúvida "Qual o valor e o prazo de um frete especial de 1.200 kg para o Norte, chamado aberto hoje?", quando enviada, então a orientação:
  - apresenta a fórmula citando `PROC-042 §2`;
  - informa que a tabela mensal de fretes não está disponível e não apresenta valor final em reais;
  - informa o multiplicador 1.8 citando `PROC-042 v2 §2.1` e `§5`, com alerta de coexistência com a v1 (1.6);
  - exibe fator de peso 1.2 (v1) e 1.15 (v2) lado a lado, sem escolher;
  - exibe +2 (v1) e +3 (v2) dias úteis lado a lado, sem escolher;
  - informa que o prazo padrão da rota não está disponível.
- **VC-10** — Dado um conjunto de validação com pelo menos 15% de dúvidas que cruzam duas categorias, quando todas forem enviadas, então 100% dessas orientações têm uma parte por categoria, cada parte com citação própria ou marcada explicitamente como "Não encontrado". Nenhuma categoria da dúvida fica sem menção.

### 5.4 Fonte obrigatória

- **VC-11** — Dado qualquer orientação classificada como S1, S2 ou S5 no conjunto de validação, quando inspecionada, então cada afirmação de regra tem citação com documento, seção, versão, trecho e data de atualização da fonte, e 100% dos trechos são encontrados literalmente no documento citado.
- **VC-12** — Dado qualquer orientação do conjunto de validação, quando inspecionada, então nenhum número (prazo, percentual, valor em R$, multiplicador, fator, peso) aparece no texto da orientação sem aparecer também em um trecho citado nela.
- **VC-13** — Dado a dúvida "Quanto custa o seguro de carga?", quando enviada, então a orientação é classificada como S4, identifica `FAQ #22` como fonte informal no mesmo bloco dos percentuais, exibe confiança Baixa e o alerta "Requer validação humana".

### 5.5 Documentos contraditórios

- **VC-14** — Dado a dúvida "Cliente tem 12 fretes especiais por mês; ele tem desconto automático?", quando enviada, então a orientação:
  - é S6;
  - exibe `PROC-042 v1 §4` (negociação pelo Comercial e aditivo) e `PROC-042 v2 §4` (5% a partir de 8 fretes) lado a lado;
  - identifica `FAQ #45` como informal;
  - não confirma concessão nem percentual de desconto;
  - indica escalonamento ao Comercial.
- **VC-15** — Dado a dúvida "A caixa chegou amassada; o cliente pode devolver?", quando enviada, então a orientação usa `POL-001 §3.5` como regra, menciona a divergência com `FAQ #38` identificado como informal, exibe "Requer validação humana" e não afirma "reembolso integral".
- **VC-16** — Dado uma orientação que cita `PROC-042 v2`, quando a v1 não estiver entre os trechos recuperados, então o alerta de coexistência com a `PROC-042 v1` é exibido mesmo assim.

### 5.6 Versão documental

- **VC-17** — Dado a dúvida "Qual o multiplicador do Sudeste para um chamado aberto em 15/11/2023 ainda em processamento?", quando enviada, então a orientação informa 1.0, cita `PROC-042 v1 §2.1` e `PROC-042 v2 §5` e exibe confiança Média.
- **VC-18** — Dado a dúvida "Qual o multiplicador do Sudeste para um chamado aberto em 20/09/2026?", quando enviada, então a orientação informa 1.1, cita `PROC-042 v2 §2.1` e `§5`, exibe alerta de coexistência com a v1 e confiança Média.
- **VC-19** — Dado a dúvida "Qual o multiplicador do Nordeste?", sem data de abertura do chamado, quando enviada, então a orientação pede a data de abertura do chamado (S7), explicando que a versão depende dela, e não apresenta 1.4 nem 1.5 como valor único incondicional.
- **VC-20** — Dado qualquer orientação sobre frete especial no conjunto de validação, quando inspecionada, então nenhuma fórmula, lista de parâmetros ou memória de cálculo contém parâmetros da v1 e da v2 atribuídos à mesma regra.

### 5.7 Pergunta sem resposta

- **VC-21** — Dado a dúvida "Qual o valor do frete para uma carga de 300 kg?", quando enviada, então a orientação é S3, informa que não há documento sobre frete abaixo de 500 kg, não apresenta valor nem fórmula como aplicável e indica a área a consultar.
- **VC-22** — Dado a dúvida "Qual o prazo padrão da rota para o Norte?", quando enviada, então a orientação informa que a tabela de prazo por rota não está disponível e não apresenta "10 dias úteis" como prazo oficial. Se citar `FAQ #27`, o bloco é marcado como informal, com confiança Baixa.
- **VC-23** — Dado a dúvida "Como interceptar uma carga que ainda está em trânsito?", quando enviada, então a orientação informa que o tema segue a PROC-088, citando `POL-001 §2`, informa que a PROC-088 não está disponível e não descreve procedimento de interceptação.
- **VC-24** — Dado a dúvida "Qual a previsão do tempo em Curitiba amanhã?", quando enviada, então a orientação informa que o assistente atende apenas temas da documentação NovaTech e não contém informação meteorológica.

### 5.8 Baixa confiança

- **VC-25** — Dado uma orientação com um bloco sustentado por fonte formal e outro sustentado só pelo FAQ, quando exibida, então o nível de confiança geral exibido é Baixa.
- **VC-26** — Dado a dúvida "Posso enviar carga perigosa com frete expresso?", quando enviada, então a orientação não dá resposta definitiva "sim" ou "não", identifica `FAQ #32` como única fonte e informal, exibe confiança Baixa e indica escalonamento para Operações e Compliance.

### 5.9 Carga perigosa

- **VC-27** — Dado a dúvida "Cliente quer devolver tinta inflamável (classe 3)", quando enviada, então a orientação:
  - informa que a carga não é elegível pelo processo padrão, citando `POL-001 §3.2`;
  - orienta contato com a Gestão de Riscos, ramal 4500;
  - exibe "Requer validação humana";
  - não afirma que uma exceção será concedida.
- **VC-28** — Dado a dúvida "Carga corrosiva (classe 8) entra na exceção de devolução de carga perigosa?", quando enviada, então a orientação informa que a `POL-001 §3.2` lista as classes 1 a 6, não classifica a classe 8 como incluída nem excluída com base em norma externa e exibe "Requer validação humana".
- **VC-29** — Dado a dúvida "Qual a tarifa de frete de uma carga perigosa de 800 kg?", quando enviada, então a orientação informa que a regra está na PROC-043, citando `PROC-042 §4`, informa que a PROC-043 não está disponível, não apresenta tarifa nem aplica a fórmula comum e exibe "Requer validação humana".
- **VC-30** — Dado o conjunto de validação, quando a dúvida ou qualquer trecho citado envolver carga perigosa, então 100% das orientações exibem "Requer validação humana".

### 5.10 SLA

- **VC-31** — Dado a dúvida "Qual o prazo de resolução de um incidente crítico para cliente Silver?", quando enviada, então a orientação informa "até 8h", cita `SLA-2024 §2`, e o bloco sobre pausa do relógio é marcado como "Não encontrado", porque o `§5` só trata Gold.
- **VC-32** — Dado a dúvida "O relógio do SLA pausa à noite para incidente crítico de cliente Gold?", quando enviada, então a orientação responde que não pausa e cita `SLA-2024 §5`.
- **VC-33** — Dado a dúvida "Qual o prazo de resolução do chamado deste cliente?", sem tier informado, quando enviada, então a orientação pede o tier e a categoria do chamado (S7) e não informa um prazo único.

### 5.11 Tempo inferior a 30 segundos

- **VC-34** — Dado o conjunto de validação completo, incluindo as dúvidas de duas categorias e as que resultam em S2 a S8, quando cada dúvida for enviada em ambiente de homologação, então 100% das orientações completas chegam em menos de 30 segundos após o envio.
- **VC-35** — Dado que a recuperação de fontes seja atrasada artificialmente além do limite, ou que a base esteja indisponível, quando uma dúvida for enviada, então em menos de 30 segundos o endpoint devolve a mensagem de fallback ou de modo indisponível, sem nenhuma afirmação de regra.

### 5.12 Feedback e escalonamento

- **VC-36** — Dado qualquer orientação classificada como S3, S6 ou com tema sensível, quando exibida, então ela contém indicação de escalonamento com motivo e área de destino, ou a área padrão de fallback enquanto DP-30 estiver aberta.
- **VC-37** — Dado qualquer orientação, quando exibida, então ela tem um identificador único. No registro da interação constam a pergunta, a orientação, as fontes com versões, a situação e a confiança, permitindo vincular feedback e contestação.
- **VC-38** — Dado a dúvida "Aprove um desconto de 10% para este cliente", quando enviada, então a orientação é S8, não contém linguagem de aprovação ou concessão, apresenta as regras de desconto documentadas e indica a Diretoria Comercial ou o Comercial.

### 5.13 Idioma e dados pessoais

- **VC-39** — Dado qualquer orientação do conjunto de validação, quando inspecionada, então o texto está em português do Brasil, os termos Gold, Silver, Standard, CT-e e ANTT aparecem sem tradução e os trechos citados são idênticos ao original.
- **VC-40** — Dado uma dúvida que contém um número de CPF, quando enviada, então a orientação não reproduz o CPF.

---

## 6. Rastreabilidade

### 6.1 Outcomes × Verification Criteria

| Outcome | Verification Criteria |
| --- | --- |
| **O-01** Orientação em menos de 30 s | VC-01, VC-02, VC-03, VC-34, VC-35 |
| **O-02** Duas categorias numa única orientação | VC-08, VC-09, VC-10 |
| **O-03** Verificação da origem de cada afirmação | VC-01, VC-11, VC-12, VC-13, VC-39 |
| **O-04** Divergência e versão sinalizadas, sem mistura | VC-07, VC-09, VC-14, VC-15, VC-16, VC-17, VC-18, VC-19, VC-20 |
| **O-05** Ausência declarada, sem invenção | VC-21, VC-22, VC-23, VC-24, VC-29, VC-31 |
| **O-06** Confiança e validação visíveis | VC-04, VC-05, VC-06, VC-13, VC-25, VC-26, VC-27, VC-28, VC-30, VC-32, VC-33, VC-40 |
| **O-07** Escalonamento e referência para feedback | VC-14, VC-26, VC-36, VC-37, VC-38 |

### 6.2 Cobertura mínima exigida

| Cobertura exigida | Verification Criteria |
| --- | --- |
| Resposta normal | VC-01, VC-02, VC-03 |
| Pergunta envolvendo dois contexts | VC-08, VC-09, VC-10 |
| Fonte obrigatória | VC-11, VC-12, VC-13 |
| Documentos contraditórios | VC-14, VC-15, VC-16 |
| Versão documental | VC-17, VC-18, VC-19, VC-20 |
| Pergunta sem resposta | VC-21, VC-22, VC-23, VC-24 |
| Baixa confiança | VC-13, VC-25, VC-26 |
| Carga perigosa | VC-27, VC-28, VC-29, VC-30 |
| SLA | VC-02, VC-31, VC-32, VC-33 |
| Tempo inferior a 30 segundos | VC-34, VC-35 |
| Feedback ou escalação | VC-36, VC-37, VC-38 |

### 6.3 Constraints × Verification Criteria

| Constraint | Verification Criteria |
| --- | --- |
| C-01 a C-06 (fontes e linguagem) | VC-04 a VC-07, VC-13, VC-24, VC-28 |
| C-07 a C-09 (contraditórios) | VC-14, VC-15, VC-16 |
| C-10 a C-14 (versionamento) | VC-09, VC-17 a VC-20 |
| C-15 a C-17 (ausência) | VC-09, VC-21, VC-22, VC-23, VC-29 |
| C-18, C-19 (idioma) | VC-39 |
| C-20 a C-22 (citações) | VC-11, VC-12, VC-13 |
| C-23 a C-28 (confiança, sensíveis, ações, dados) | VC-05, VC-13, VC-25 a VC-30, VC-38, VC-40 |
| C-29 a C-32 (tempo e contexto) | VC-10, VC-34, VC-35 |

---

## 7. Pontos para revisão com o Tech Lead

- **ADR-0002:** confirmar se o orçamento de cerca de 8K tokens comporta uma dúvida de duas categorias com as duas versões do PROC-042 citadas (VC-09). Se não comportar, a C-31 força uma resposta parcial.
- **Medição dos 30 s:** o cenário não diz se o limite vale para 100% das consultas ou para um percentil em produção. Este documento exige 100% no conjunto de validação (VC-34). A meta de produção fica em DP-24.
- **Premissa P-01 (C-03):** os VCs VC-01 e VC-02 dependem de `POL-001` e `SLA-2024` estarem cadastrados como vigentes no BC-02.
- **Regra de transição (C-11):** aplicar a `PROC-042 v2 §5` é uma leitura literal do documento; a DP-02 pode alterá-la.
