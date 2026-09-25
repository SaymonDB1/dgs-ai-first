# 06-requirements-final — Query Endpoint do Assistente NovaTech

| Item | Conteúdo |
| --- | --- |
| Artefato | Spec SDD · `requirements.md` |
| Componente | Query endpoint: recebe a dúvida do atendente e devolve a orientação fundamentada |
| Versão | 2.0 · Final · 24/09/2026 · substitui `02-requirements-v1.md` |
| Base da revisão | `04-tech-lead-review.md` |
| Recorte de domínio | `05-bounded-contexts-final.md`: contexts `BC-xx`, termos de negócio `T-01` a `T-41`, termos do produto `TP-01` a `TP-40`, propriedade de conceitos (§2.3), tabela de áreas (§2.4), catálogo de temas sensíveis (§2.5), registro de conflitos (§2.6) |
| Fonte de verdade | Anexo A: `POL-001`, `PROC-042 v1`, `PROC-042 v2`, `SLA-2024`, `FAQ` |
| Spec anterior | ESPEC-V2: requisitos `FON`, `RES`, `SEN`, `ESC`, `FBK`, `RAS`, `RN`, `GR`, `SEG`, `RNF` e decisões `DP-xx`. **Documento não disponível nesta revisão:** as origens que citam a ESPEC-V2 foram herdadas do `02-requirements-v1.md` e não puderam ser conferidas (DT-17) |
| Jornada | `Output_Tarefa2_Jornada_integrada` (PDF): princípio central, guardrails `G1`–`G6`, fluxos `P`, `F`, `R`. A jornada textual da Tarefa 1 (G1–G9) está superada; sua tabela de escalonamento foi usada como proposta de áreas (`05` §2.4) |
| Outras fontes verificadas | Documentos individuais do Anexo A (texto canônico), Anexo B (chunks e gabarito de retrieval), `exercicio-fase-1-entendimento.md` (cenário, guardrails do Product Specialist), entregáveis da fase 1 (`Reflexao_Progressive_Disclosure`, jornada da Tarefa 1) |
| Histórico | `07-historico-iteracao.md` |

**Princípio que governa todo o documento** (`JORNADA`): *responder com base na documentação; se não houver evidência suficiente, não assumir e direcionar para validação.*

**Como ler:**

- Os termos com ID `T-xx` (negócio) e `TP-xx` (produto) têm o sentido exato definido em `05-bounded-contexts-final.md` §3. Os IDs `T-01` a `T-40` são os mesmos da rev. 1.1 do recorte. Por exemplo, "SLA" (`T-19`) é prazo de atendimento de chamado, nunca prazo de entrega (`T-16`).
- Os IDs de constraints e VCs da v1 foram preservados. Itens novos recebem os próximos números (C-33 em diante, VC-41 em diante) e aparecem na seção temática a que pertencem.
- Decisões pendentes herdadas da ESPEC-V2 usam `DP-xx`. Decisões pendentes abertas por esta revisão usam `DT-xx` (§4.4).

---

## 1. Outcomes

Resultados observáveis para o atendente. Não descrevem funcionalidades técnicas.

### 1.1 Categorias de dúvida e domínio

| Categoria do cenário (`TP-12`) | Context de negócio consultado | Exemplo de dúvida |
| --- | --- | --- |
| Prazo de entrega | BC-06 Prazos, Execução e Rastreamento | "Qual o prazo de um frete especial de 1.200 kg para o Norte?" |
| Regras de frete | BC-05 Cotação de Frete Especial (+ BC-09 para desconto, BC-07 para carga perigosa) | "Qual multiplicador aplico para o Nordeste?" |
| Política de devolução | BC-10 Devolução e Logística Reversa | "Até quando o cliente pode pedir devolução?" |
| SLAs | BC-08 Atendimento, SLA e Penalidades (+ BC-09 para tier) | "Qual o prazo de primeira resposta para cliente Gold?" |

As categorias são agrupamento do cenário. O **domínio do assistente** (`TP-11`) é o conjunto dos BCs consultados (BC-05 a BC-11): dúvidas sobre carga perigosa (BC-07), condições comerciais (BC-09) ou seguro e sinistro (BC-11) estão no domínio mesmo sem pertencer a uma das quatro categorias.

Cerca de **15% das dúvidas cruzam duas categorias**, por exemplo devolução + SLA ou frete + prazo.

### 1.2 Outcomes

| ID | Resultado para o atendente | Sinal observável |
| --- | --- | --- |
| **O-01** | O atendente recebe, **em menos de 30 segundos**, a orientação completa (campos da C-48) para dúvidas do domínio. | Tempo medido conforme C-29 < 30 s, inclusive quando a orientação é um fallback. |
| **O-02** | Quando a dúvida tem duas partes, o atendente recebe **uma única orientação** que responde cada parte separadamente, cada uma com sua fonte, sua situação, seu nível de confiança e seus alertas. | Uma parte por pergunta, até o limite da C-41; nenhuma parte some sem aviso; nível e alertas exibidos por parte. |
| **O-03** | O atendente consegue **verificar sozinho de onde vem cada afirmação** (documento, seção, versão e trecho literal), sem abrir outro sistema. | Toda afirmação de regra tem citação própria e conferível no Anexo A. |
| **O-04** | O atendente **nunca recebe como regra única** algo que as fontes tratam de forma divergente, nem uma regra montada com versões misturadas do PROC-042. É avisado quando a versão aplicável depende de um dado do caso. | Divergências aparecem lado a lado; o dado do caso é pedido quando decide a versão. |
| **O-05** | Quando a documentação não sustenta a resposta, o atendente **sabe disso imediatamente**, sabe o que falta e qual área está indicada, em vez de receber uma resposta inventada. | Mensagem de ausência com o que falta e a área; nenhum valor, prazo ou percentual sem fonte. |
| **O-06** | O atendente vê, **para cada parte**, o nível de confiança e se precisa de validação humana antes de falar com o cliente (tema sensível, carga perigosa, fonte informal). | Nível de confiança por parte e alerta de validação humana visíveis sem ação adicional. |
| **O-07** | Quando a orientação exige uma pessoa ou está errada, o atendente **tem um caminho claro**: indicação de escalonamento com motivo e área, e um identificador da orientação para feedback e contestação. | Escalonamento indicado nos casos previstos; toda orientação tem identificador único. |

---

## 2. Scope Boundaries

Derivados de `05-bounded-contexts-final.md`.

### 2.1 Coberto diretamente

| Context | O que o endpoint realiza |
| --- | --- |
| **BC-01 Orientação Fundamentada ao Atendente** (core) | Entende a dúvida e separa as partes; identifica o BC de cada parte; pede o dado do caso que falta; classifica a situação por parte e da orientação (S0 a S9); atribui confiança por bloco e por parte; cita as fontes; aplica o registro de conflitos e sinaliza divergência não registrada; aplica o catálogo de temas sensíveis e a tabela de áreas; aplica os fallbacks sem suposições; produz o registro de interação e o identificador; devolve a orientação ao atendente. |

### 2.2 Coberto na fronteira

O endpoint produz o sinal, mas não executa o processo do context.

| Context | O que o endpoint faz | O que fica com o context |
| --- | --- | --- |
| **BC-02 Governança do Conhecimento** | Emite pendência para divergência detectada não registrada (`TP-32`). | Triagem, registro de conflito, decisão de precedência. |
| **BC-03 Validação Humana e Escalonamento** | Indica que a orientação exige validação ou escalonamento, com motivo, tema e área de destino da tabela do BC-03. Referencia o registro de interação pelo identificador. | Catálogo de temas sensíveis, tabela de áreas, roteamento efetivo, canal (DP-09), validação pelo especialista. |
| **BC-04 Feedback e Melhoria Contínua** | Produz o identificador único e o registro de interação (`RAS-02`), para que feedback e contestação possam ser vinculados. | Tratamento do feedback depois de registrado: avaliação, ciclo de vida (`FBK-03`), acompanhamento de contestação, correção da documentação. |

### 2.3 Consultados (somente leitura)

O endpoint lê regras e metadados desses contexts; não cria, altera nem decide regra.

| Context | O que é consultado | Restrição de consulta |
| --- | --- | --- |
| **BC-02 Governança do Conhecimento** | Status, versão, vigência, classificação formal/informal, hierarquia, registro de conflitos por seção, documentos ausentes | Conformist: aceita a classificação sem reinterpretar |
| **BC-05 Cotação de Frete Especial** | Enquadramento, fórmula, valor base, multiplicadores, fatores de peso (`PROC-042 v1` e `v2`) | Não entrega valor final enquanto DP-14 estiver aberta; valor base ausente |
| **BC-06 Prazos, Execução e Rastreamento** | Prazo adicional, aprovação > 5.000 kg, definição de data de recebimento | Não informa status real de carga (`G2`); prazo padrão da rota ausente |
| **BC-08 Atendimento, SLA e Penalidades** | Prazos por tier, incidente crítico, pausa do relógio, penalidades (`SLA-2024`) | Consome o tier como dado do BC-09 |
| **BC-09 Cliente, Contrato e Condições Comerciais** | Critérios de tier, inexistência de outros tiers, regra de desconto por volume | Não concede desconto nem classifica o cliente sem o dado informado: aplica o critério ao dado |
| **BC-10 Devolução e Logística Reversa** | Prazo, procedimento, triagem, exceções, devolução parcial, custos, avaria na devolução (`POL-001`) | Não aprova devolução nem exceção |
| **BC-07 Cargas Perigosas e Conformidade** | Classes 1 a 6 (`T-26`, `POL-001 §3.2`), inelegibilidade à devolução padrão, remissão à PROC-043, áreas de destino (Gestão de Riscos e Compliance) | Sempre com validação humana; nunca resposta definitiva |
| **BC-11 Avarias, Sinistros e Seguro** | FAQ #22 e #38 (única fonte) | Sempre baixa evidência (S4) e validação humana |

### 2.4 Fora do escopo

| Item | Por quê |
| --- | --- |
| Curadoria, publicação, classificação, retirada de documentos e registro de conflitos (escrita no BC-02) | Pertence ao BC-02; o endpoint só lê e emite pendência |
| Registro de concordância ou discordância do atendente (operação do BC-01) e tratamento posterior do feedback (BC-04) | O registro do feedback é uma operação própria do BC-01, distinta do query endpoint, com os motivos da `JORNADA` R1 (errada, desatualizada, incompleta, ambígua, fonte incorreta, outro); spec própria (DT-04). O query endpoint só fornece o identificador e o registro de interação. |
| Execução do escalonamento e validação humana (BC-03) | Mecanismo em DP-09 |
| Status real, localização e previsão de carga | Vem do tracking oficial (`G2`, BC-06); não é consulta documental |
| Conceder desconto, aprovar devolução ou exceção, autorizar embarque ou carga perigosa | `RN-12`: o assistente informa e orienta |
| Frete padrão (< 500 kg), interceptação de carga em trânsito (PROC-088), tarifa de carga perigosa (PROC-043), tabela mensal, prazo padrão por rota | Documentos ausentes do corpus (`T-06`, `T-36`, `T-27`, `T-09`, `T-15`). A ausência do documento formal é sempre declarada; nunca são inferidos. Conteúdo do FAQ sobre esses temas só aparece como S4 (C-36). |
| Comunicação direta com o cliente final | O atendente decide o que comunicar (DP-10) |
| Arquitetura, modelo, chunking, embeddings, formato serializado do payload | Decisão técnica (`plan.md`). O **conteúdo lógico** da orientação está na C-48. |

### 2.5 Dependências

| ID | Dependência | Efeito no endpoint |
| --- | --- | --- |
| **D-01** | Integração com o canal Microsoft Teams (premissa P-02, DT-07): o canal deve confirmar o recebimento imediatamente e entregar a orientação em mensagem subsequente, porque a requisição síncrona do bot tem timeout inferior a 30 s (valor a confirmar no ADR de canal); o texto dos trechos literais deve ser escapado para não sofrer formatação; o conteúdo não pode ser truncado nem colapsado de forma a esconder alertas (C-53). | C-29, C-38, C-53; VC-34, VC-47 |
| **D-02** | Identidade do atendente via SSO do Teams (Entra ID). O identificador do atendente é gravado no registro de interação; nenhum outro dado do perfil é gravado. | C-50; VC-37 |
| **D-03** | BC-02 disponibiliza metadados (classificação, versão, vigência, data de atualização), registro de conflitos por seção e lista de documentos ausentes. | C-02, C-03, C-09, C-37, C-39 |
| **D-04** | BC-03 disponibiliza o catálogo de temas sensíveis e a tabela de áreas (`05` §2.4, §2.5). | C-26, C-46 |
| **D-05** | ADR-0002 (orçamento de contexto) confirmado pelo Tech Lead (DT-09). | C-31; VC-46 |
| **D-06** | Ingestão dos 5 documentos do Anexo A como arquivos individuais, preservando tabelas com linha e coluna. | C-01, C-38 |

---

## 3. Constraints

### 3.1 Fontes

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-01** | O corpus de produção é composto apenas pelos 5 documentos do Anexo A. O Anexo B é massa de teste e não pode sustentar resposta em produção. | Anexo A; ESPEC-V2 Premissas |
| **C-02** | Classificação de autoridade: `POL-001` normativo, `SLA-2024` contratual, `PROC-042 v1` e `v2` formais sem vigência declarada, `FAQ` informal, sem responsável formal, declarado no próprio cabeçalho como "NÃO validado por Compliance ou Operações". | Anexo A (cabeçalhos) |
| **C-03** | Premissa P-01: `POL-001` e `SLA-2024` são tratados como formais vigentes; `PROC-042 v1` e `v2` como formais sem vigência confirmada. A premissa deve ser confirmada pelo BC-02 (DT-08). Se não for confirmada, os blocos de `POL-001` e `SLA-2024` passam a nível no máximo Médio e os VCs parametrizados (VC-01, VC-02, VC-08, VC-32) seguem o valor alternativo indicado neles. | ESPEC-V2 FON-01, DP-18 |
| **C-04** | Fonte informal nunca substitui nem prevalece sobre fonte formal, mesmo quando cita um procedimento formal. | RN-05, GR-13 |
| **C-05** | Conhecimento geral do modelo, legislação externa e web não completam, substituem nem contradizem regra interna. Até DP-13, só a base autorizada é usada. | RES-11, RN-07, GR-11 |
| **C-06** | As orientações usam as definições da linguagem ubíqua. Exemplos: "Gold" é tier do `SLA-2024 §1` (`T-03`); "carga perigosa" são as classes 1 a 6 da `POL-001 §3.2` (`T-26`), e outra classe mencionada não é classificada como incluída nem excluída; "frete especial" é acima de 500 kg (`T-05`), com o enquadramento de exatamente 500 kg ambíguo no documento; "devolução" conta 7 dias úteis, não corridos (`T-28`). | `05-bounded-contexts-final.md` |

### 3.2 Documentos contraditórios

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-07** | Divergência entre fontes aplicáveis nunca é ocultada. Com precedência aplicável → S5 (regra prevalente + divergência). Sem precedência → S6 (fontes lado a lado, sem resposta definitiva, com escalonamento indicado). Em ambos os casos, cada fonte aparece com sua versão, sua data (`TP-27`) e seu status de vigência (`TP-41`). | RES-08, RN-06, GR-05; requisitos simulados do Product Specialist (exercício 1.1 do Tech Lead) |
| **C-08** | Até a decisão da DP-01, conflito entre fontes **formais** é sempre tratado como não resolvido (S6). Conflito formal × FAQ é resolvido pela C-04 (S5). Quando a mesma parte tem os dois tipos de conflito, vale a precedência da C-40 (S6). | FON-03; revisão RQ-03 |
| **C-09** | Conflitos registrados (`05` §2.6) exibem alerta mesmo quando só uma das fontes foi recuperada. O registro vale por **seção** (C-39). | FON-07; Anexo A · Notas; `TP-31` |
| **C-33** | Divergência entre trechos recuperados que não consta do registro de conflitos (`TP-32`) é tratada como S6 com o rótulo "Possível divergência", indicação de escalonamento pela tabela de áreas e emissão de pendência para o BC-02. O endpoint não registra o conflito nem decide precedência. | Revisão BC-03 |
| **C-34** | Em S6, a orientação pode incluir uma frase de comparação entre as fontes, com o rótulo "Comparação", desde que: não contenha número, prazo, percentual ou condição que não esteja nos trechos citados; não recomende uma das versões; não estime impacto para o cliente. Provisório até DT-03. | Revisão MK-09 |

### 3.3 Versionamento

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-10** | A orientação nunca combina parâmetros da v1 e da v2 do PROC-042 numa mesma regra ou cálculo. | RES-09, GR-12 |
| **C-11** | Para **multiplicadores regionais**, a versão é escolhida pela regra de transição da `PROC-042 v2 §5` (versão aplicável, `T-12`), usando a data de referência (`TP-04`), isto é, a data de abertura do chamado a que a regra se refere (`T-41`): chamado aberto antes de 01/12/2023 e ainda em processamento → v1; aberto a partir de 01/12/2023 → v2. Chamado aberto antes de 01/12/2023 **e já encerrado** não é coberto pela regra: v1 e v2 lado a lado (S6), com escalonamento indicado para o Comercial. A coexistência com a outra versão é sempre sinalizada. Nível máximo: Média. | PROC-042 v2 §5; FON-04; `T-12`; revisão RQ-08 |
| **C-12** | Para **fator de peso, prazo adicional e desconto por volume**, não há critério documental de versão. Esses parâmetros seguem S6 até a decisão da DP-02. | FON-04; `T-11`, `T-13`, `T-16` |
| **C-13** | Versão mais recente, nome, ano ou número de versão, isoladamente, não confirmam vigência. | RN-03, RN-04 |
| **C-14** | Se a versão depende de um dado do caso não informado, o endpoint pede o dado (S7): a data de referência e, quando ela for anterior a 01/12/2023, se o chamado ainda está em processamento. Se o atendente responder que não tem o dado (C-44), a orientação apresenta cada versão com a sua condição documental e a sua citação, sem indicar valor único. | RES-07; revisão RQ-07, RQ-08 |
| **C-35** | Quando qualquer parâmetro da fórmula do PROC-042 tiver versões divergentes, a fórmula é exibida **sem valores substituídos**; cada parâmetro aparece no seu próprio bloco, com versão e citação. Nenhuma memória de cálculo parcial é apresentada. | RES-09, GR-12; revisão RQ-09 |

### 3.4 Ausência de resposta

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-15** | Sem evidência → S3: mensagem de ausência, o que falta e área indicada. Nenhum valor, prazo, percentual ou procedimento é estimado. | RES-05, GR-01, GR-02, `G1` |
| **C-16** | Documento ausente (`TP-29`: PROC-043, PROC-088, tabela mensal, prazo padrão por rota, frete padrão) → o endpoint informa o documento faltante, responde só os blocos independentes (`TP-21`, S2) e indica escalonamento quando a dependência é bloqueante (`TP-22`). | FON-08; revisão LU-06 |
| **C-17** | Com parâmetro obrigatório ausente, o endpoint mostra a fórmula e o parâmetro faltante e não calcula. | RES-10, GR-14 |
| **C-36** | Quando o documento formal está ausente e o FAQ tem conteúdo sobre o tema, a parte é S4: a ausência do documento formal é declarada primeiro; o conteúdo do FAQ aparece identificado como informal, com nível Baixa, e nunca como regra oficial. Se o FAQ não tiver conteúdo, a parte é S3. | Revisão RQ-18 |

### 3.5 Idioma

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-18** | A orientação é redigida em português do Brasil, formal e acessível, idioma do corpus. Termos da linguagem ubíqua e siglas do Anexo A são mantidos como no documento (Gold, Silver, Standard, CT-e, ANTT, PROC-042). | Anexo A; `05-bounded-contexts-final.md`; guardrail (4) do exercício |
| **C-19** | O trecho de evidência é reproduzido literalmente no idioma original, sem tradução nem paráfrase. Dúvida em outro idioma: provisoriamente, a orientação é redigida em PT-BR com o aviso "O assistente responde em português" (fallback F-09), até a DP-24. | RES-03; DP-24 |

### 3.6 Citações

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-20** | Cada afirmação de regra (valor, prazo, percentual, critério, procedimento, elegibilidade) tem citação própria (`TP-25`) com documento, título, seção, versão, data da fonte com o rótulo do próprio documento (`TP-27`), status de vigência (`TP-41`), classificação formal/informal e trecho literal. | RES-02, RES-03, `G3` ("fonte e vigência sempre visíveis") |
| **C-21** | Afirmação que não pode ser citada não é feita. A frase de comparação da C-34 não é afirmação de regra e segue as restrições daquela constraint. | RES-03, GR-16 |
| **C-22** | Fonte informal é identificada como informal no próprio bloco em que aparece. | GR-04, RES-02 (S4) |
| **C-37** | Metadado ausente (`TP-28`) é exibido com sentinela: versão "não controlada" e data "não informada". Bloco de fonte formal com metadado ausente tem nível no máximo Médio. Fonte informal já tem nível Baixa (C-24). | Revisão RQ-19 |
| **C-38** | O trecho literal é comparado ao texto canônico do documento (arquivo individual do Anexo A) após a normalização: Unicode NFC; marcação de formatação removida (negrito, itálico, emoji como ⚠️, marcadores de lista e de tabela markdown); espaços, tabulações e quebras de linha consecutivos reduzidos a um espaço; hifenização de fim de linha removida; aspas tipográficas e retas equivalentes; travessão, meia-risca e hífen equivalentes. Nenhuma outra alteração é permitida. Trecho de **tabela** é citado como referência de célula: tabela, rótulo da linha, rótulo da coluna e o valor da célula; cada valor é comparado literalmente à célula correspondente. | RES-03; revisão RQ-12 |

### 3.7 Confiança e temas sensíveis

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-23** | A confiança reflete a qualidade da evidência e é atribuída **por bloco**. **Alta:** fonte formal com vigência confirmada, afirmação explícita (`TP-19`), seção sem conflito registrado, metadados completos, sem falha de atualização. **Média:** fonte formal sem vigência confirmada, interpretação permitida (`TP-08`), conflito resolvido por precedência (S5), regra de transição (C-11), metadado ausente (C-37) ou falha de atualização (F-04). **Baixa:** somente fonte informal, ou evidência que cobre a afirmação apenas em parte. **Não aplicável:** bloco de ausência, de divergência sem precedência, de pedido de dado ou de pedido de ação. | RES-04; revisão LU-06 |
| **C-24** | O nível da **parte** é o menor entre os blocos com nível aplicável; se nenhum bloco tiver nível aplicável, é "Não aplicável". Orientação de uma parte exibe o nível da parte. Orientação de duas partes exibe o nível **de cada parte** e não exibe nível global. Somente FAQ nunca recebe Alta nem Média. Seção com conflito registrado nunca recebe Alta. | RES-04, GR-15; revisão RQ-01, RQ-02 |
| **C-39** | Conflito registrado e vigência afetam apenas as **seções** listadas no registro (`05` §2.6). Outras seções do mesmo documento não são rebaixadas por causa desse conflito. | Revisão LU-09 |
| **C-25** | Tema sensível sustentado só por fonte informal nunca recebe resposta definitiva e sempre tem indicação de escalonamento com motivo e área. | SEN-02; revisão RQ-14 |
| **C-26** | Carga perigosa (`T-26`) sempre exibe "Requer validação humana", independentemente da qualidade da evidência. Os demais temas do catálogo do BC-03 (`05` §2.5) também exibem o alerta: avaria, extravio, indenização, reclamação formal, exceção contratual, desconto, seguro, exceção de devolução, incidente crítico. O nível exato por tema depende da DP-08. | SEN-01, SEN-02, `G5` |
| **C-27** | O endpoint não autoriza, concede nem aprova nada; pedido de ação (`TP-24`) é S8. | RN-12, RES-12 |
| **C-28** | A orientação não reproduz dados pessoais presentes na dúvida: CPF, endereço, nome e telefone do destinatário. | `G6`; SEG-05; DP-17 |

### 3.8 Tempo de resposta e contexto

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-29** | A orientação completa (todos os campos da C-48) é entregue em **menos de 30 segundos**. O tempo é medido do recebimento da mensagem do atendente pela integração com o canal até a entrega da mensagem final à API de envio do canal. A renderização no cliente do atendente não entra na medição (D-01). O limite vale para todas as situações, inclusive duas partes e fallbacks. A meta de produção (percentil) depende da DP-24. | Cenário; RNF-01; revisão RQ-21 |
| **C-30** | Se a orientação fundamentada não puder ser concluída dentro do limite, o endpoint entrega o fallback F-01 ou F-02 (C-49) antes dos 30 s, nunca uma regra sem citação. | C-29, C-21; RNF-06 |
| **C-31** | Orçamento de contexto por consulta conforme **ADR-0002**: cerca de 4K tokens para instruções de sistema e cerca de 8K tokens para trechos recuperados. Numa dúvida de duas partes, os trechos das duas partes dividem o mesmo orçamento. Se não couberem, a parte ou o bloco não coberto é declarado (S2); citações nunca são truncadas. Os valores dependem da confirmação do ADR (DT-09). | ADR-0002 |
| **C-32** | Alertas, nível de confiança por parte, necessidade de validação e área indicada ficam visíveis sem ação adicional do atendente. | RNF-05; RES-02 |

### 3.9 Situações

| Situação | Nome | Gatilho | Nível | Escalonamento indicado |
| --- | --- | --- | --- | --- |
| **S0** | Indisponível | Base indisponível ou nenhuma parte concluída no tempo (F-02, F-03) | Não aplicável | Área padrão de fallback |
| **S1** | Resposta completa | Todos os blocos da parte têm citação formal; nenhuma divergência sem precedência | Alta ou Média | Só se tema sensível |
| **S2** | Resposta parcial | Parte com blocos respondidos e ao menos um bloco não coberto (documento ausente, orçamento, parte excedente ou não classificada) | Menor nível dos blocos respondidos | Se dependência bloqueante ou tema sensível |
| **S3** | Sem evidência | Nenhuma fonte, formal ou informal, sustenta a parte | Não aplicável | Sempre |
| **S4** | Baixa evidência | Parte sustentada somente por fonte informal | Baixa | Se tema sensível (C-25) |
| **S5** | Regra prevalente com divergência | Divergência resolvida por precedência (hoje, só formal × informal) | No máximo Média | Se tema sensível |
| **S6** | Conflito não resolvido | Divergência sem precedência, registrada ou detectada (C-33) | Não aplicável para os blocos divergentes | Sempre |
| **S7** | Informação necessária | Dado do caso (`TP-23`) ausente e decisivo, ou dúvida ininteligível | Não aplicável | Não |
| **S8** | Ação fora da alçada | Pedido de ação (`TP-24`) | Não aplicável para a ação; blocos informativos mantêm seu nível | Sempre |
| **S9** | Fora do domínio | Nenhuma parte pertence ao domínio (`TP-11`) | Não aplicável | Não |

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-40** | Cada parte tem a sua situação. A situação exibida no cabeçalho da orientação é a mais restritiva entre as partes, pela ordem: S0 > S8 > S6 > S7 > S3 > S4 > S5 > S2 > S1. S9 só é a situação da orientação quando nenhuma parte está no domínio; parte fora do domínio numa dúvida mista é declarada como não tratada (S2). Dentro de uma parte, a situação é a mais restritiva entre os blocos, pela mesma ordem. | RES-01; revisão LU-05, RQ-03 |

### 3.10 Partes, domínio e segundo turno

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-41** | A orientação trata no máximo **2 partes**. A partir da terceira, as partes excedentes são listadas como "não tratadas nesta orientação; envie separadamente" e a orientação é S2. Duas perguntas do mesmo BC são partes distintas. Limite provisório até DT-13. | Revisão RQ-04 |
| **C-42** | Parte do domínio que não pode ser associada a um BC é declarada como não tratada, com sugestão de reformulação (S2 na orientação). Se for a única parte, a orientação é S7 (pede reformulação). | Revisão RQ-04 |
| **C-43** | Após uma orientação S7, a próxima mensagem do mesmo atendente na mesma conversa, dentro da janela de **30 minutos**, é interpretada como resposta ao pedido de dado e completa a dúvida original. A nova orientação recebe novo identificador e referencia o identificador da orientação S7. Fora da janela, ou se a mensagem for claramente uma nova dúvida, é tratada como dúvida nova. Janela provisória até DP-29/DT-01. | DP-29; revisão RQ-06 |
| **C-44** | Toda orientação S7 oferece ao atendente a resposta "Não tenho essa informação". Com essa resposta, aplica-se o caminho condicional da C-14 (ou, para tier e categoria, a apresentação das alternativas documentadas sem valor único). | RES-07; revisão RQ-06 |
| **C-45** | Dúvida fora do domínio (`TP-11`) é S9 com a mensagem "Atendo apenas temas da documentação NovaTech". Temas dos BCs 07, 09 e 11 estão no domínio mesmo fora das quatro categorias. | DP-31; revisão RQ-05 |

### 3.11 Escalonamento e área de destino

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-46** | A área de destino vem da tabela do BC-03 (`05` §2.4). Cada parte com escalonamento indicado tem **uma única área**. Quando mais de uma linha se aplica, vale a primeira da tabela; as linhas de carga perigosa vêm primeiro (Gestão de Riscos para devolução; Compliance para frete, tarifa, expresso e demais temas). Tema sem área documentada usa a área padrão de fallback (`TP-35`, DP-30). | Revisão BC-02, RQ-16 |
| **C-47** | A indicação de escalonamento (`TP-40`) é exibida como "Área indicada: <área>" com o motivo. Enquanto a DP-09 estiver aberta, a orientação não usa linguagem de execução ("escalado", "encaminhado", "enviado ao supervisor") nem oferece ação que dispare roteamento. Quando o tema não tem área documentada, a área indicada é a área padrão (Supervisão SAC N2), que corresponde à orientação de escalar ao supervisor do guardrail (3) do exercício e à raia 5 da `JORNADA`. | DP-09; revisão MK-01, RQ-14 |

### 3.12 Contrato lógico da orientação

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-48** | Toda orientação contém os campos lógicos abaixo. O formato de serialização é decisão do `plan.md`. | Revisão RQ-27 |

| Campo | Conteúdo | Obrigatório |
| --- | --- | --- |
| Identificador | `TP-37` | Sempre |
| Identificador anterior | Identificador da orientação S7 respondida (C-43) | Quando houver |
| Data e hora | Momento da entrega | Sempre |
| Situação da orientação | S0 a S9 (C-40) | Sempre |
| Partes | Lista de 0 a 2 partes | Sempre (vazia em S0 e S9) |
| Parte · rótulo e BC | Categoria ou tema e BC associado | Por parte |
| Parte · situação | S1 a S8 | Por parte |
| Parte · nível de confiança | Alta, Média, Baixa, Não aplicável (C-24) | Por parte |
| Parte · blocos | Tipo (regra, ausência, divergência, pedido de dado, informativo, comparação), texto, nível, citações | Por parte |
| Citação | Campos da C-20, com sentinelas da C-37 e referência de célula da C-38 | Por afirmação de regra |
| Alertas | Tipo (validação humana, fonte informal, divergência, possível divergência, coexistência de versões, documento ausente, atualização pendente), mensagem e parte a que pertence | Quando houver |
| Escalonamento | Indicado (sim/não), motivo e área, por parte | Quando indicado |
| Dados solicitados | Dados do caso pedidos e opção "Não tenho essa informação" | Em S7 |
| Partes não tratadas | Texto de cada parte excedente, não classificada ou fora do domínio | Quando houver |
| Mensagem de fallback | Texto padrão da C-49 | Em fallback |

### 3.13 Fallbacks

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-49** | Os fallbacks seguem a tabela abaixo. Nenhum fallback contém afirmação de regra sem citação. | RES-01; ESPEC-V2 §13; `JORNADA` F1; revisão RQ-20 |

| Fallback | Gatilho | Comportamento e mensagem | Situação |
| --- | --- | --- | --- |
| **F-01** | Limite de tempo iminente com ao menos uma parte concluída | Entrega as partes concluídas; declara as demais: "Não consegui concluir esta parte a tempo. Envie-a novamente." | S2 |
| **F-02** | Limite de tempo iminente sem nenhuma parte concluída | "Não consegui concluir a consulta a tempo. Tente novamente ou consulte a área indicada." com a área padrão | S0 |
| **F-03** | Base indisponível | "A base de documentação está indisponível no momento. Não responda ao cliente com base em memória; consulte a área indicada." com a área padrão | S0 |
| **F-04** | Fonte com falha de atualização registrada pelo BC-02 | Blocos dessa fonte com nível no máximo Médio e alerta "Fonte com atualização pendente" | Conforme a parte |
| **F-05** | Metadado ausente | Sentinelas da C-37 | Conforme a parte |
| **F-06** | Documento ausente | C-16 | S2 ou S3 |
| **F-07** | Orçamento de contexto excedido | C-31 | S2 |
| **F-08** | Parte excedente ou não classificada | C-41, C-42 | S2 ou S7 |
| **F-09** | Dúvida em outro idioma | Orientação em PT-BR com o aviso "O assistente responde em português"; trechos literais no original (provisório até DP-24) | Conforme a parte |
| **F-10** | Mensagem com anexo ou imagem | Aviso "Anexos não são analisados; considerei apenas o texto". Sem texto, pede a dúvida em texto (S7) | Conforme a parte |
| **F-11** | Falha ao gravar o registro de interação | Entrega a orientação com o identificador e o aviso "Esta referência pode não estar disponível para contestação" | Conforme a parte |
| **F-12** | Fora do domínio | C-45 | S9 |
| **F-13** | Divergência detectada não registrada | C-33 | S6 |
| **F-14** | Mensagem vazia ou ininteligível | "Não entendi a dúvida. Pode reformular?" | S7 |

### 3.14 Registro, identificador e dados pessoais

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-50** | Cada orientação gera **um único** registro de interação (`TP-36`), produzido pelo BC-01, com: identificador, identificador anterior (quando houver), identificador do atendente (D-02), número do chamado (quando informado), dúvida mascarada, orientação, fontes com versões, situações, níveis, alertas, áreas indicadas. BC-03 e BC-04 referenciam o registro pelo identificador e não mantêm registro paralelo. | RAS-02; revisão BC-04; jornada da Tarefa 1 (P2, FB2) |
| **C-51** | Os dados pessoais da C-28 são mascarados **antes** de gravar o registro. Prazo de retenção e perfis de acesso ao registro dependem da DP-17/DT-06. | SEG-05; DP-17; revisão RQ-22 |
| **C-52** | O identificador da orientação é único, não reutilizável, não contém dado pessoal e não permite deduzir o volume de consultas. O formato é decisão do `plan.md` (DT-14). | RAS-02; revisão MK-11 |

### 3.15 Apresentação no canal

Restrições sobre o conteúdo que o canal deve exibir. O layout é do mockup, que deve ser atualizado (DT-15).

| ID | Restrição | Origem |
| --- | --- | --- |
| **C-53** | Ordem de exibição: (1) alertas de validação humana, divergência e fonte informal, cada um com o rótulo da parte; (2) situação e nível por parte; (3) texto e citações; (4) área indicada; (5) identificador. Nenhum recurso de colapsar ou truncar pode esconder os itens 1, 2 e 4. | RNF-05; revisão MK-12 |
| **C-54** | Em S3, S4 e S6, o título da parte não afirma regra. Em S4, o título declara a ausência de fonte formal (ex.: "Não há regra formal sobre seguro de carga"); os valores do FAQ aparecem só dentro do bloco informal. | C-25; revisão MK-03 |
| **C-55** | Os rótulos exibidos seguem `05` §3.4 (termo → rótulo de UI). | Revisão LU-07, MK-06 |
| **C-56** | Toda citação exibe, na mesma ordem e com os mesmos rótulos: código do documento, título conforme cabeçalho, seção, "Versão", a data com o rótulo do próprio documento ("Última atualização" na POL-001 e no SLA-2024; "Data de emissão" no PROC-042; "Diversas" no FAQ) (`TP-27`) e o status de vigência (`TP-41`). A versão é exibida como no cabeçalho do documento; "v1" e "v2" são aliases do PROC-042 usados apenas no texto desta spec. | Revisão MK-07, LU-08; cabeçalhos do Anexo A |
| **C-58** | O trecho literal vem sempre do texto do documento. O chunk recuperado serve para localizar a seção; os chunks do Anexo B são condensados e não são citáveis como trecho. | Anexo B (chunks condensados); RES-03 |
| **C-59** | O assistente informa os critérios e os prazos de incidente crítico (`SLA-2024 §2`, `§3`, `§5`) e pode dizer que os dados informados atendem a um critério explícito (interpretação permitida), mas não classifica o chamado como incidente crítico; a classificação segue o fluxo de incidente do SLA pelo atendente. | Jornada da Tarefa 1 (P1, G6); `T-22` |
| **C-57** | O registro de feedback é uma operação do BC-01 fora deste endpoint (DT-04); quando existir, usa os motivos da `JORNADA` R1. A orientação exibe o identificador com o texto "Use esta referência para feedback ou contestação" e não promete acompanhamento enquanto o FBK-03 não estiver especificado. | Revisão MK-10, MK-11 |

---

## 4. Prior Decisions

### 4.1 ADRs e premissas

| ID | Decisão | Efeito neste endpoint | Observação |
| --- | --- | --- | --- |
| **ADR-0002** | Context budget: cerca de 4K tokens de system e cerca de 8K tokens de chunks por query | C-31; priorização de evidência formal dentro do orçamento; VC-46 | Documento não disponível. Pelo `exercicio-fase-1-entendimento.md`, o ADR-0002 é um entregável do Tech Lead (exercício 1.1); a análise simulada do desenvolvedor estima ~2K tokens de system e chunks de ~500 tokens, o que difere dos ~4K herdados do 02. Confirmar ou escrever o ADR e medir VC-09 e VC-14 (DT-09). |
| **P-01** | `POL-001` e `SLA-2024` vigentes | C-03 | Confirmar com BC-02 (DT-08) |
| **P-02** | Canal Microsoft Teams | D-01; §3.15 | Adotado pelo mockup; formalizar em ADR de canal (DT-07, DP-24) |

Nenhum outro ADR foi fornecido. As rubricas citam o diretório `/docs/adr/`, mas seu conteúdo não está nos arquivos enviados. Nenhum ID de ADR foi criado aqui.

### 4.2 Decisões da fase anterior, por tema

| Tema | Decisão | Origem | Efeito neste endpoint |
| --- | --- | --- | --- |
| **RAG** | O assistente responde a partir de documentação recuperada da base autorizada, sem completar com conhecimento geral. | ESPEC-V2 §1, RES-11; `JORNADA` princípio central | C-05, C-15 |
| **RAG** | O Anexo A disponibiliza os 5 documentos como arquivos individuais para ingestão; o Anexo B é massa de teste. | Anexo A (nota de ingestão); ESPEC-V2 Premissas | C-01, D-06 |
| **RAG** | Documentos formais são recuperados antes do FAQ; o atendente é avisado quando a resposta vem de fonte informal. | `Reflexao_Progressive_Disclosure` §6; FON-03 | C-04, C-22 |
| **RAG** | Estratégia de chunking, embeddings e score de similaridade ficam fora da spec de produto. | ESPEC-V2 §2.2 | Scope 2.4 |
| **Context budget** | ~4K system + ~8K chunks por query. | ADR-0002 | C-31 |
| **Fontes** | Hierarquia baseline: formal vigente > formal sem vigência > FAQ. Definitiva em DP-01. | FON-03 | C-08 |
| **Fontes** | Disponibilidade na base não significa oficialidade; cada fonte tem metadados de versão, vigência e responsável. | RN-02, FON-01 | C-02, C-03, C-37 |
| **Fontes** | A PROC-042 v2 existe na fonte de verdade; a correção anterior que a dava como inexistente está superada. A DP-03 da ESPEC-V2 fica respondida. | `05-bounded-contexts-final.md` (rev. 1); Anexo A | C-10 a C-12 |
| **Fontes** | O limiar de aprovação de 5.000 kg está nas duas versões do PROC-042. A DP-27 fica respondida. | `05-bounded-contexts-final.md` T-17 | VC-03 |
| **Documentos contraditórios** | Conflito sem precedência é apresentado e escalado, nunca decidido pelo assistente. | RN-06, RES-08 | C-07, C-08, C-33 |
| **Documentos contraditórios** | Registro de conflitos conhecidos, com alerta independente da recuperação. | FON-07 | C-09, C-39 |
| **Documentos contraditórios** | Não combinar versões. | RES-09, GR-12 | C-10, C-35 |
| **Ausência de resposta** | Situações S1–S8 e mensagens padrão de fallback. | RES-01; ESPEC-V2 §13 | §3.9 (acrescenta S0 e S9), C-49 |
| **Ausência de resposta** | Fallback sem suposições: não criar, informar ausência, mostrar o encontrado, indicar validação. | `JORNADA` F1 | C-15, C-36 |
| **Atualização da base** | A orientação mostra a data de atualização da fonte; fonte com falha de atualização não recebe Alta; base indisponível → modo indisponível. | FON-10 | C-20, C-23, F-03, F-04; VC-35, VC-52 |
| **Atualização da base** | Nova versão só entra após publicação; a anterior vira histórica sem apagar o registro; SLAs de atualização em DP-05 e DP-06. | FON-05, FON-09; ESPEC-V2 §5.1 | Scope 2.4 (BC-02) |
| **Atualização da base** | Lacunas do fallback viram pendências de conteúdo; documentos vencidos saem da busca. | `JORNADA` R0, R6 | Scope 2.2 (BC-04) |

### 4.3 Decisões pendentes da ESPEC-V2 que afetam o endpoint

Enquanto a decisão não for tomada, vale o comportamento provisório.

| DP | Assunto | Comportamento provisório neste endpoint |
| --- | --- | --- |
| DP-01 | Hierarquia documental | Conflito entre formais = S6 (C-08) |
| DP-02 | Aplicabilidade do PROC-042 | Transição da v2 §5 só para multiplicadores; demais parâmetros S6 (C-11, C-12) |
| DP-05, DP-06 | SLAs de atualização da base | Nova versão disponível no assistente em até 24h após a publicação (requisito simulado do Product Specialist no exercício 1.1 do Tech Lead); fora do escopo deste endpoint (BC-02) |
| DP-08 | Temas sensíveis e níveis N1/N2/N3 | Alerta de validação humana em todo o baseline (C-26) |
| DP-09 | Mecanismo de escalonamento | Só indicação de motivo e área, sem linguagem de execução (C-47) |
| DP-10 | Comunicação com o cliente | O atendente decide o que comunicar (Scope 2.4) |
| DP-12 | Data de referência padrão | Data de abertura do chamado relacionado ao frete; pedir quando faltar (C-14, `TP-04`, `T-41`, DT-11) |
| DP-13 | Conhecimento externo | Proibido (C-05) |
| DP-14 | Entrega de valor final calculado | Não entregar valor final; a tabela mensal está ausente, de qualquer forma (C-17) |
| DP-15 | Uso permitido por nível de confiança | Exibir o nível por parte; o uso é orientado pelo alerta |
| DP-17 | Dados pessoais | Não reproduzir na orientação; mascarar antes de gravar; retenção pendente (C-28, C-51, DT-06) |
| DP-18 | Metadados obrigatórios | Premissa P-01 (C-03); sentinelas para ausentes (C-37) |
| DP-24 | Tempo, canal, idioma | < 30 s no conjunto de validação (C-29); canal Teams como premissa (P-02); PT-BR com aviso para outro idioma (C-19, F-09) |
| DP-29 | Contexto entre turnos | Só para S7: janela de 30 minutos (C-43); demais orientações cumprem C-20 por si só |
| DP-30 | Área padrão de fallback | Supervisão de Atendimento (SAC N2) (`TP-35`), com base na raia 5 da `JORNADA` e no guardrail (3) do exercício |
| DP-31 | Pergunta fora do domínio | S9 com a mensagem da C-45 |

### 4.4 Decisões pendentes abertas por esta revisão

| DT | Decisão | Comportamento provisório | Responsável sugerido | Itens da revisão |
| --- | --- | --- | --- | --- |
| DT-01 | Janela de contexto para responder S7 | 30 minutos, mesmo atendente e conversa (C-43) | Produto + Tech Lead | RQ-06 |
| DT-02 | Tabela tema/BC → área | `05` §2.4 | BC-03 | BC-02, RQ-16 |
| DT-03 | Frase de comparação em S6 | Permitida com as restrições da C-34 | Produto + Compliance | MK-09 |
| DT-04 | Spec da operação de feedback do BC-01 e entrada no MVP | Fora do query endpoint; motivos da `JORNADA` R1 | Produto | MK-10 |
| DT-05 | Concorrência do teste de tempo e percentil de produção | 10 consultas simultâneas no VC-34; percentil em DP-24 | Tech Lead + QA | RQ-21 |
| DT-06 | Retenção e acesso ao registro de interação | Mascaramento obrigatório; retenção a definir | Segurança / DPO | RQ-22 |
| DT-07 | ADR de canal (Teams) | Premissa P-02 e D-01 | Tech Lead | MK-13 |
| DT-08 | Confirmação da premissa P-01 | VCs parametrizados (C-03) | BC-02 | RQ-26 |
| DT-09 | Confirmação do ADR-0002 e medição de VC-09 e VC-14 | Valores de C-31 | Tech Lead | RQ-25 |
| DT-10 | Área de carga perigosa | Gestão de Riscos para exceção de devolução (`POL-001 §3.2`); Compliance para frete, tarifa, expresso e demais temas (`PROC-042 v2 §4`, `FAQ #32`). A jornada da Tarefa 1 propõe "Operações e Compliance", que exige escolher uma área | BC-03 + BC-07 | BC-07, RQ-16 |
| DT-11 | Qual chamado fornece a data de referência do PROC-042 | Data de abertura informada pelo atendente para o chamado relacionado ao frete (`T-41`) | BC-05 + Produto | LU-04 |
| DT-13 | Limite de partes por orientação | 2 partes (C-41) | Produto | RQ-04 |
| DT-14 | Formato do identificador | Requisitos da C-52 | Tech Lead | MK-11 |
| DT-16 | Massa de teste para o VC-43 (divergência não registrada) | O Anexo B não contém esse caso; criar dois documentos de teste só para homologação | QA | Verificação com Anexo B |
| DT-17 | Conferência com a ESPEC-V2 (situações S1 a S8, mensagens do §13, teor das DPs, requisitos citados como origem) e confirmação da origem dos "30 segundos" (O-01, C-29, VC-34) e dos "15% com duas categorias" (§1.1, VC-10), que não aparecem em nenhum arquivo disponível; o único 15% do cenário é o de casos escalados ao supervisor | Valores herdados do `02-requirements-v1.md` | Produto + Tech Lead | Verificação geral |
| DT-15 | Mockup v2 com os estados faltantes (S0, S2, S3, S5, S7, S8, S9, carga perigosa, coexistência, confiança Média, duas partes com nível por parte) e com as C-53 a C-57 | Mockup v1 vale só para os estados que já mostra, com os ajustes da §3.15 | Design | MK-01 a MK-08, MK-12, MK-14 |

---

## 5. Verification Criteria

Todos os critérios são binários.

**Regras gerais de verificação:**

- "Citação" significa os campos da C-20, com o trecho literal verificado pela normalização da C-38.
- "A orientação informa X" significa que os **valores exatos** (número, unidade, marco) e os **elementos obrigatórios** listados estão presentes no texto. O texto livre ao redor não é comparado literalmente.
- "Área indicada" é verificada contra a tabela de `05` §2.4 vigente na execução.
- VCs marcados com **[P-01]** esperam Alta enquanto a premissa estiver confirmada e Média caso contrário (C-03).
- Toda dúvida do conjunto de validação tem data fixa quando a data influencia o resultado.
- **Divergência consciente com o gabarito do Anexo B:** para perguntas sobre multiplicador sem data ("Qual o multiplicador para o Sudeste?", "Frete para 600kg para Manaus?"), o mapa do Anexo B espera a v2 diretamente; esta spec exige S7 (C-14). O mapa do Anexo B continua valendo para avaliar retrieval; o comportamento da resposta é avaliado por estes VCs.

### 5.1 Resposta normal

- **VC-01** — Dado a dúvida "Até quando o cliente pode solicitar a devolução?", quando enviada ao endpoint, então a orientação informa o prazo de 7 dias úteis e o marco "data de recebimento confirmada no sistema de tracking", cita `POL-001 §3.1`, é classificada como S1 e exibe confiança Alta **[P-01]**. O conflito registrado em `POL-001 §3.5` não rebaixa esta seção (C-39).
- **VC-02** — Dado a dúvida "Qual o prazo de primeira resposta de um chamado geral para cliente Gold?", quando enviada, então a orientação informa "até 2h úteis", cita `SLA-2024 §2` como referência de célula (tabela, linha, coluna e valor, C-38), exibe confiança Alta **[P-01]** e não menciona prazo de entrega de carga.
- **VC-03** — Dado a dúvida "Carga de 6.000 kg em frete especial precisa de aprovação?", quando enviada, então a orientação informa a aprovação prévia do gerente de operações regional, cita `PROC-042 v1 §4` e `PROC-042 v2 §4`, não sinaliza conflito para esse ponto, porque as duas versões coincidem, e exibe confiança Média (fonte formal sem vigência confirmada).

### 5.2 Linguagem ubíqua

- **VC-04** — Dado a dúvida "Cliente diz ser Platinum; qual o SLA dele?", quando enviada, então a orientação afirma que só existem os tiers Gold, Silver e Standard, cita a nota do `SLA-2024 §1` e não atribui nenhum prazo de SLA a "Platinum".
- **VC-05** — Dado a dúvida "Cliente com contrato anual de R$ 600.000 e 30 operações/mês é Gold?", quando enviada, então a orientação responde que atende ao critério Gold, porque o critério é "OU", cita `SLA-2024 §1`, identifica a interpretação permitida (`TP-08`) e exibe confiança Média.
- **VC-06** — Dado a dúvida "Cliente Gold tem prazo de entrega menor?", quando enviada, então a orientação não afirma redução de prazo de entrega para Gold e informa que o `SLA-2024` trata de prazos de atendimento de chamados.
- **VC-07** — Dado a dúvida "Uma carga de exatamente 500 kg é frete especial?", quando enviada, então a orientação exibe o trecho "acima de 500kg" (`PROC-042 §1`) e o trecho "de 500kg a 1.000kg" (`§2`), não responde "sim" nem "não" como regra e exibe "Área indicada: Comercial".

### 5.3 Pergunta envolvendo dois contexts

- **VC-08** — Dado a dúvida "Cliente Gold quer devolver uma mercadoria; qual o prazo de triagem da devolução e qual o prazo de primeira resposta do chamado?", quando enviada, então a orientação apresenta duas partes identificadas: devolução, com "4 horas úteis" citando `POL-001 §3.3`; e SLA, com "até 2h úteis" citando `SLA-2024 §2`. Cada parte exibe seu próprio nível (Alta e Alta **[P-01]**) e nenhum nível global é exibido. Nenhuma frase afirma que um prazo substitui ou inclui o outro.
- **VC-09** — Dado a dúvida "Qual o valor e o prazo de um frete especial de 1.200 kg para o Norte, chamado aberto em 24/09/2026?", quando enviada, então a orientação apresenta duas partes:
  - **frete (valor)**, com situação S6 e "Área indicada: Comercial", que:
    - apresenta a fórmula sem valores substituídos, citando `PROC-042 §2`, sem alerta de divergência para a fórmula, que é idêntica nas duas versões (C-35, C-39);
    - informa que a tabela mensal de fretes não está disponível e não apresenta valor final em reais;
    - informa o multiplicador 1.8 citando `PROC-042 v2 §2.1` e `§5`, com alerta de coexistência com a v1 (1.6) e nível Média;
    - exibe fator de peso 1.2 (v1) e 1.15 (v2) em blocos separados, lado a lado, sem escolher;
  - **prazo**, com situação S6 e "Área indicada: Operações", que:
    - exibe +2 (v1) e +3 (v2) dias úteis lado a lado, sem escolher;
    - informa que o prazo padrão da rota não está disponível.
- **VC-10** — Dado um conjunto de validação com pelo menos 15% de dúvidas que cruzam duas categorias, quando todas forem enviadas, então 100% dessas orientações têm uma parte por categoria, cada parte com citação própria ou marcada explicitamente como "Não encontrado". Nenhuma categoria da dúvida fica sem menção.

### 5.4 Fonte obrigatória

- **VC-11** — Dado qualquer orientação classificada como S1, S2, S4 ou S5 no conjunto de validação, quando inspecionada, então cada afirmação de regra tem citação com os campos da C-20 e 100% dos trechos são encontrados no texto canônico do documento citado após a normalização da C-38, nunca apenas no texto de um chunk (C-58) (ou, para tabelas, cada valor de célula confere com a célula referenciada).
- **VC-12** — Dado qualquer orientação do conjunto de validação, quando inspecionada, então nenhum número de regra (prazo, percentual, valor em R$, multiplicador, fator, peso, quantidade) aparece no texto das partes sem aparecer também em um trecho citado na mesma orientação. **Não entram na verificação:** identificador da orientação, números de seção, versão e data de citação, contagem de partes, e números que apenas repetem dados informados na dúvida.
- **VC-13** — Dado a dúvida "Quanto custa o seguro de carga?", quando enviada, então a orientação:
  - é classificada como S4;
  - tem título que declara a ausência de regra formal e não afirma os percentuais (C-54);
  - identifica `FAQ #22` como fonte informal no mesmo bloco dos percentuais;
  - exibe confiança Baixa e o alerta "Requer validação humana";
  - exibe "Área indicada: Comercial" (C-25, C-46; `05` §2.4, linha 11a).

### 5.5 Documentos contraditórios

- **VC-14** — Dado a dúvida "Cliente tem 12 fretes especiais por mês; ele tem desconto automático?", quando enviada, então a orientação:
  - é S6;
  - exibe `PROC-042 v1 §4` (negociação pelo Comercial e aditivo) e `PROC-042 v2 §4` (5% a partir de 8 fretes) lado a lado;
  - identifica `FAQ #45` como informal;
  - não confirma concessão nem percentual de desconto;
  - exibe "Requer validação humana" (desconto) e "Área indicada: Comercial";
  - se contiver frase de comparação, ela atende à C-34.
- **VC-15** — Dado a dúvida "A caixa chegou amassada; o cliente pode devolver?", quando enviada, então a orientação usa `POL-001 §3.5` como regra, é S5 com nível Média, menciona a divergência com `FAQ #38` identificado como informal, exibe "Requer validação humana" e "Área indicada: Operações", e não afirma "reembolso integral".
- **VC-16** — Dado uma orientação que cita `PROC-042 v2`, quando a v1 não estiver entre os trechos recuperados, então o alerta de coexistência com a `PROC-042 v1` é exibido mesmo assim.
- **VC-43** — Dado um documento de teste criado para homologação (DT-16) com divergência não registrada em relação a outro documento de teste, carregado apenas no ambiente de homologação, quando uma dúvida recupera os dois trechos, então a orientação é S6 com o rótulo "Possível divergência", exibe os dois trechos lado a lado, indica área e emite pendência para o BC-02.

### 5.6 Versão documental

- **VC-17** — Dado a dúvida "Qual o multiplicador do Sudeste para um chamado aberto em 15/11/2023 ainda em processamento?", quando enviada, então a orientação informa 1.0, cita `PROC-042 v1 §2.1` e `PROC-042 v2 §5` e exibe confiança Média.
- **VC-18** — Dado a dúvida "Qual o multiplicador do Sudeste para um chamado aberto em 20/09/2026?", quando enviada, então a orientação informa 1.1, cita `PROC-042 v2 §2.1` e `§5`, exibe alerta de coexistência com a v1 e confiança Média.
- **VC-19** — Dado a dúvida "Qual o multiplicador do Nordeste?", sem data de abertura do chamado, quando enviada, então a orientação pede a data de abertura do chamado (S7), explica que a versão depende dela, oferece a resposta "Não tenho essa informação" e não apresenta 1.4 nem 1.5 como valor único.
- **VC-20** — Dado qualquer orientação sobre frete especial no conjunto de validação, quando inspecionada, então nenhuma fórmula, lista de parâmetros ou memória de cálculo contém parâmetros da v1 e da v2 atribuídos à mesma regra, e nenhuma fórmula aparece com valores substituídos quando algum parâmetro tem versões divergentes (C-35).
- **VC-41** — Dado a orientação S7 do VC-19, quando o atendente responder "Não tenho essa informação", então a nova orientação apresenta o multiplicador da v1 e o da v2 em blocos separados, cada um com sua condição da `PROC-042 v2 §5` e sua citação, sem indicar valor único e com nível Média.
- **VC-42** — Dado a dúvida "Qual o multiplicador do Sudeste para um chamado aberto em 10/11/2023 e já encerrado?", quando enviada, então a orientação é S6, exibe o multiplicador da v1 e o da v2 lado a lado, informa que a regra de transição não cobre chamados encerrados e exibe "Área indicada: Comercial".
- **VC-50** — Dado a orientação S7 do VC-19, quando o mesmo atendente responder "20/09/2026" na mesma conversa em até 30 minutos, então a nova orientação informa o multiplicador da v2 citando `PROC-042 v2 §2.1` e `§5`, tem novo identificador e referencia o identificador da orientação S7. Quando a mesma resposta chegar depois de 30 minutos, então ela é tratada como dúvida nova (S7 pedindo reformulação).

### 5.7 Pergunta sem resposta

- **VC-21** — Dado a dúvida "Qual o valor do frete para uma carga de 300 kg?", quando enviada, então a orientação é S3, informa que não há documento sobre frete abaixo de 500 kg, não apresenta valor nem fórmula como aplicável e exibe "Área indicada: Comercial".
- **VC-22** — Dado a dúvida "Qual o prazo padrão da rota para o Norte?", quando enviada, então a orientação declara primeiro que a tabela de prazo por rota não está disponível e não apresenta "10 dias úteis" como prazo oficial. Se exibir `FAQ #27`, a parte é S4, o bloco é marcado como informal com confiança Baixa e o título não afirma o prazo (C-36, C-54); se não exibir, a parte é S3. Em ambos os casos, exibe "Área indicada: Operações".
- **VC-23** — Dado a dúvida "Como interceptar uma carga que ainda está em trânsito?", quando enviada, então a orientação informa que o tema segue a PROC-088, citando `POL-001 §2`, informa que a PROC-088 não está disponível, não descreve procedimento de interceptação e exibe "Área indicada: Operações" (dependência bloqueante).
- **VC-24** — Dado a dúvida "Qual a previsão do tempo em Curitiba amanhã?", quando enviada, então a orientação é S9, informa que o assistente atende apenas temas da documentação NovaTech e não contém informação meteorológica.

### 5.8 Baixa confiança

- **VC-25** — Dado uma parte com um bloco sustentado por fonte formal e outro sustentado só pelo FAQ, quando exibida, então o nível de confiança exibido para essa parte é Baixa.
- **VC-26** — Dado a dúvida "Posso enviar carga perigosa com frete expresso?", quando enviada, então a orientação não dá resposta definitiva "sim" ou "não", identifica `FAQ #32` como única fonte e informal, exibe confiança Baixa, "Requer validação humana" e "Área indicada: Compliance" (provisório até DT-10).

### 5.9 Carga perigosa

- **VC-27** — Dado a dúvida "Cliente quer devolver tinta inflamável (classe 3)", quando enviada, então a orientação:
  - informa que a carga não é elegível pelo processo padrão, citando `POL-001 §3.2`;
  - orienta contato com a Gestão de Riscos, ramal 4500, e exibe "Área indicada: Gestão de Riscos";
  - exibe "Requer validação humana";
  - não afirma que uma exceção será concedida.
- **VC-28** — Dado a dúvida "Carga corrosiva (classe 8) entra na exceção de devolução de carga perigosa?", quando enviada, então a orientação informa que a `POL-001 §3.2` lista as classes 1 a 6, não classifica a classe 8 como incluída nem excluída com base em norma externa, exibe "Requer validação humana" e "Área indicada: Gestão de Riscos".
- **VC-29** — Dado a dúvida "Qual a tarifa de frete de uma carga perigosa de 800 kg?", quando enviada, então a orientação informa que a regra está na PROC-043, citando `PROC-042 §4`, informa que a PROC-043 não está disponível, não apresenta tarifa nem aplica a fórmula comum, exibe "Requer validação humana" e "Área indicada: Compliance".
- **VC-30** — Dado o conjunto de validação, quando a dúvida ou qualquer trecho citado envolver carga perigosa (`T-26`), então 100% das orientações exibem "Requer validação humana" e, na parte correspondente, a área da linha de carga perigosa aplicável (`05` §2.4: Gestão de Riscos para devolução; Compliance para os demais temas).

### 5.10 SLA

- **VC-31** — Dado a dúvida "Qual o prazo de resolução de um incidente crítico para cliente Silver?", quando enviada, então a orientação informa "até 8h", cita `SLA-2024 §2`, a parte é S2 com o bloco sobre pausa do relógio marcado como "Não encontrado" (porque o `§5` só trata Gold), e exibe "Requer validação humana" (incidente crítico) e "Área indicada" com a área padrão de fallback.
- **VC-32** — Dado a dúvida "O relógio do SLA pausa à noite para incidente crítico de cliente Gold?", quando enviada, então a orientação responde que não pausa, cita `SLA-2024 §5`, exibe confiança Alta **[P-01]**, "Requer validação humana" e "Área indicada" com a área padrão de fallback.
- **VC-33** — Dado a dúvida "Qual o prazo de resolução do chamado deste cliente?", sem tier informado, quando enviada, então a orientação pede o tier e a categoria do chamado (S7), oferece "Não tenho essa informação" e não informa um prazo único.

### 5.11 Tempo inferior a 30 segundos

- **VC-34** — Dado o conjunto de validação completo, incluindo as dúvidas de duas partes e as que resultam em S0 a S9, quando executado 3 vezes em homologação, uma vez de forma sequencial e duas com 10 consultas simultâneas (provisório até DT-05), então 100% das orientações completas são entregues em menos de 30 segundos, medidos conforme a C-29.
- **VC-35** — Dado que a recuperação de fontes seja atrasada artificialmente além do limite, quando uma dúvida de uma parte for enviada, então em menos de 30 segundos o endpoint entrega o F-02 (S0) com a mensagem padrão e a área padrão, sem nenhuma afirmação de regra. Dado que a recuperação de uma das partes de uma dúvida de duas partes seja atrasada, então o endpoint entrega o F-01 (S2) com a parte concluída e a outra declarada. Dado que a base esteja indisponível, então o endpoint entrega o F-03 (S0).

### 5.12 Feedback e escalonamento

- **VC-36** — Dado qualquer orientação ou parte classificada como S0, S3, S6 ou S8, com tema sensível ou com dependência bloqueante, quando exibida, então ela contém "Área indicada" com **uma única área** por parte, igual à da tabela de `05` §2.4, e o motivo do escalonamento.
- **VC-37** — Dado qualquer orientação, quando exibida, então ela tem um identificador único que atende à C-52. No registro de interação constam os campos da C-50, com a dúvida mascarada, e existe exatamente um registro por identificador.
- **VC-38** — Dado a dúvida "Aprove um desconto de 10% para este cliente", quando enviada, então a orientação é S8, não contém linguagem de aprovação ou concessão, apresenta as regras de desconto documentadas, exibe "Requer validação humana" (desconto) e "Área indicada: Comercial".
- **VC-55** — Dado o conjunto de validação, quando inspecionadas todas as orientações, então nenhuma contém as expressões "escalado", "encaminhado", "enviado ao supervisor" ou "acompanhar", e toda indicação de escalonamento aparece no formato "Área indicada: <área>" (C-47, C-57).

### 5.13 Idioma e dados pessoais

- **VC-39** — Dado qualquer orientação do conjunto de validação, quando inspecionada, então o texto está em português do Brasil, os termos Gold, Silver, Standard, CT-e e ANTT aparecem sem tradução e os trechos citados atendem à C-38.
- **VC-40** — Dado uma dúvida que contém CPF, endereço, e nome e telefone do destinatário, quando enviada, então a orientação não reproduz nenhum desses dados e o registro de interação os contém apenas mascarados.
- **VC-53** — Dado a dúvida "What is the return deadline?", quando enviada, então a orientação é redigida em PT-BR com o aviso "O assistente responde em português", e os trechos citados aparecem no original (provisório até DP-24).

### 5.14 Partes, orçamento, contrato e fallbacks

- **VC-44** — Dado uma dúvida com três perguntas (prazo de devolução, SLA Gold de primeira resposta e multiplicador do Sudeste para chamado aberto em 20/09/2026), quando enviada, então a orientação trata as duas primeiras partes, lista a terceira como "não tratada nesta orientação; envie separadamente" e é S2.
- **VC-45** — Dado a dúvida "E aquele outro caso, como fica?", quando enviada sem conversa anterior, então a orientação é S7 e pede reformulação, sem afirmação de regra.
- **VC-46** — Dado o VC-09 executado com o orçamento de trechos reduzido artificialmente para que a segunda parte não caiba, quando enviado, então a parte não coberta é declarada (S2) e nenhuma citação exibida está truncada.
- **VC-47** — Dado as orientações dos VC-09, VC-13 e VC-14 exibidas no cliente Microsoft Teams em homologação, quando inspecionadas sem nenhuma interação do atendente, então os alertas, a situação e o nível por parte e a área indicada estão visíveis e aparecem na ordem da C-53.
- **VC-48** — Dado qualquer orientação que cite o FAQ, quando inspecionada, então a citação exibe versão "não controlada" e data "não informada" (ou a data declarada, se houver) e o bloco tem nível Baixa.
- **VC-49** — Dado qualquer orientação do conjunto de validação, quando inspecionada, então ela contém todos os campos obrigatórios da C-48 para a sua situação, e os rótulos exibidos seguem `05` §3.4.
- **VC-51** — Dado que a gravação do registro de interação falhe, quando uma dúvida for enviada, então a orientação é entregue com o identificador e o aviso da F-11.
- **VC-52** — Dado que o BC-02 marque o `SLA-2024` com falha de atualização, quando a dúvida do VC-02 for enviada, então o bloco tem nível Média e exibe o alerta "Fonte com atualização pendente".
- **VC-54** — Dado uma mensagem com uma imagem anexada e o texto "Até quando o cliente pode solicitar a devolução?", quando enviada, então a orientação exibe o aviso da F-10 e responde ao texto como no VC-01. Dado uma mensagem só com imagem, então a orientação é S7 e pede a dúvida em texto.

### 5.15 Vigência, incidente crítico e ambiguidades do recorte de domínio

Estes VCs verificam o comportamento diante das lacunas registradas no `05` (T-02, T-08, T-13, T-18, T-22, T-32, T-40). Em todos, a resposta esperada é expor a lacuna, sem resolvê-la.

- **VC-56** — Dado as orientações dos VC-01, VC-03 e VC-13, quando inspecionadas, então as citações exibem, respectivamente: "Última atualização 15/01/2024" e "vigente por premissa P-01"; "Data de emissão 03/03/2023" e "10/11/2023" com "sem indicação formal de vigência"; "Última atualização: Diversas" e "não controlada".
- **VC-57** — Dado a dúvida "Cliente Gold abriu 6 chamados sobre o mesmo problema nas últimas 24 horas. É incidente crítico?", quando enviada, então a orientação cita o critério "Mais de 5 chamados do mesmo cliente nas últimas 24 horas sobre o mesmo problema" (`SLA-2024 §3`), informa que os dados atendem a esse critério com confiança Média (interpretação permitida), informa os prazos de incidente crítico Gold (`SLA-2024 §2`), não afirma que o chamado está classificado como incidente crítico (C-59) e exibe "Requer validação humana".
- **VC-58** — Dado a dúvida "Carga com 800 kg de peso real e 1.200 kg de peso cubado: qual fator de peso aplico?", quando enviada, então a orientação informa que o PROC-042 não define se o peso é real ou cubado, não escolhe um dos pesos nem uma faixa como definitiva e exibe "Área indicada: Comercial".
- **VC-59** — Dado a dúvida "Com o desconto de 5% da v2, qual fica o multiplicador do Sul?", quando enviada, então a orientação cita `PROC-042 v2 §4` ("desconto de 5% sobre o multiplicador regional"), informa que o documento não define se o desconto multiplica ou subtrai, não apresenta 1.235 nem 1.25, informa o conflito com `PROC-042 v1 §4` (S6), exibe "Requer validação humana" e "Área indicada: Comercial".
- **VC-60** — Dado a dúvida "Cliente com contrato anual de exatamente R$ 500.000 e 30 operações/mês é Gold ou Silver?", quando enviada, então a orientação cita os critérios do `SLA-2024 §1` ("acima de R$ 500.000" para Gold; "entre R$ 100.000 e R$ 500.000" para Silver), informa que o documento não diz se os limites são inclusivos, não classifica o cliente como definitivo e exibe "Área indicada: Comercial".
- **VC-61** — Dado a dúvida "Feriado estadual conta como dia útil no prazo de devolução?", quando enviada, então a orientação cita `POL-001 §3.1` ("exclui sábados, domingos e feriados nacionais"), informa que feriados estaduais e municipais não são mencionados, não afirma que contam nem que não contam e exibe "Área indicada: Operações".
- **VC-62** — Dado a dúvida "Carga de R$ 150.000 com status desconhecido há 6 horas e meia, contando só horário comercial. É incidente crítico?", quando enviada, então a orientação cita o critério do `SLA-2024 §3` ("há mais de 6 horas"), informa que o documento não diz se as horas são corridas ou úteis, não classifica o chamado (C-59) e exibe "Requer validação humana" e "Área indicada" com a área padrão de fallback.
- **VC-63** — Dado a dúvida "É a segunda violação de SLA deste cliente no mês. Qual a penalidade?", quando enviada, então a orientação informa o crédito de 5% sobre o valor do frete do chamado afetado, cita `SLA-2024 §4`, é S1 e exibe confiança Alta **[P-01]**.
- **VC-64** — Dado a dúvida "O cliente desistiu e vai devolver a carga. Qual multiplicador uso no frete reverso?", quando enviada, então a orientação cita `POL-001 §3.5` ("calculado com os mesmos multiplicadores do frete original"), pede a data de abertura do chamado do frete original (S7), explica que a versão do multiplicador depende dela (C-11) e não apresenta multiplicador único.
- **VC-65** — Dado a dúvida "Minha entrega atrasou e a caixa chegou amassada. Tenho direito a reembolso do frete?", quando enviada, então a orientação apresenta duas partes: **atraso**, informando que nenhum documento define reembolso de frete por atraso de entrega e que o prazo padrão da rota não está disponível (S3); e **avaria**, usando `POL-001 §3.5` como regra, com a divergência do `FAQ #38` identificada como informal (S5) e "Requer validação humana". A orientação não promete reembolso do frete, não trata o crédito do `SLA-2024 §4` como reembolso por atraso de entrega e exibe "Área indicada: Operações" nas duas partes.
- **VC-66** — Dado que o retriever retorne o chunk SLA-2024-B do Anexo B para a dúvida do VC-02, quando a orientação for inspecionada, então o trecho citado é a célula do `SLA-2024 §2` ("Até 2h úteis") e não o texto condensado do chunk ("resposta em até 2h úteis").

---

## 6. Rastreabilidade

### 6.1 Outcomes × Verification Criteria

| Outcome | Verification Criteria |
| --- | --- |
| **O-01** Orientação em menos de 30 s | VC-34, VC-35, VC-49, VC-51 |
| **O-02** Duas partes numa única orientação | VC-08, VC-09, VC-10, VC-44, VC-46 |
| **O-03** Verificação da origem de cada afirmação | VC-01, VC-02, VC-11, VC-12, VC-13, VC-39, VC-48, VC-56, VC-66 |
| **O-04** Divergência e versão sinalizadas, sem mistura | VC-07, VC-09, VC-14, VC-15, VC-16, VC-17, VC-18, VC-19, VC-20, VC-41, VC-42, VC-43, VC-50, VC-59, VC-64 |
| **O-05** Ausência declarada, sem invenção | VC-21, VC-22, VC-23, VC-24, VC-29, VC-31, VC-45, VC-53, VC-54, VC-58, VC-60, VC-61, VC-62, VC-65 |
| **O-06** Confiança e validação visíveis por parte | VC-03, VC-05, VC-08, VC-13, VC-25, VC-26, VC-27, VC-28, VC-30, VC-32, VC-47, VC-52, VC-57, VC-63 |
| **O-07** Escalonamento e referência para feedback | VC-14, VC-26, VC-36, VC-37, VC-38, VC-51, VC-55 |

Os VC-04, VC-06, VC-33 e VC-40 não têm outcome próprio: verificam constraints (C-06, C-14, C-28) e estão rastreados na §6.3.

### 6.2 Cobertura mínima exigida

| Cobertura exigida | Verification Criteria |
| --- | --- |
| Resposta normal | VC-01, VC-02, VC-03, VC-63 |
| Pergunta envolvendo dois contexts | VC-08, VC-09, VC-10, VC-44, VC-65 |
| Fonte obrigatória | VC-11, VC-12, VC-13, VC-48, VC-56, VC-66 |
| Documentos contraditórios | VC-14, VC-15, VC-16, VC-43, VC-59 |
| Versão documental | VC-17, VC-18, VC-19, VC-20, VC-41, VC-42, VC-50, VC-64 |
| Pergunta sem resposta | VC-21, VC-22, VC-23, VC-24, VC-58, VC-60, VC-61 |
| Baixa confiança | VC-13, VC-25, VC-26 |
| Carga perigosa | VC-27, VC-28, VC-29, VC-30 |
| SLA | VC-02, VC-31, VC-32, VC-33, VC-57, VC-62, VC-63 |
| Tempo inferior a 30 segundos | VC-34, VC-35 |
| Feedback ou escalação | VC-36, VC-37, VC-38, VC-55 |

### 6.3 Constraints × Verification Criteria

| Constraint | Verification Criteria |
| --- | --- |
| C-01 (corpus) | VC-11, VC-43 |
| C-02, C-03 (classificação e P-01) | VC-01, VC-02, VC-03, VC-08, VC-32 |
| C-04 (formal prevalece) | VC-14, VC-15 |
| C-05 (sem conhecimento externo) | VC-24, VC-28 |
| C-06 (linguagem ubíqua) | VC-04, VC-05, VC-06, VC-07, VC-28, VC-58, VC-60, VC-61 |
| C-07, C-08 (contraditórios) | VC-14, VC-15 |
| C-09 (conflitos registrados) | VC-16 |
| C-10 (sem mistura de versões) | VC-09, VC-20 |
| C-11 (regra de transição) | VC-17, VC-18, VC-42 |
| C-12 (parâmetros sem critério) | VC-09, VC-14, VC-59 |
| C-13 (versão não confirma vigência) | VC-03 |
| C-14 (dado do caso) | VC-19, VC-33, VC-41, VC-64 |
| C-15 (sem evidência) | VC-21, VC-65 |
| C-16 (documento ausente) | VC-09, VC-23, VC-29 |
| C-17 (parâmetro ausente) | VC-09 |
| C-18, C-19 (idioma) | VC-39, VC-53 |
| C-20, C-21, C-22 (citações) | VC-11, VC-12, VC-13, VC-56 |
| C-23, C-24 (confiança) | VC-03, VC-05, VC-08, VC-25 |
| C-25, C-26 (sensíveis) | VC-13, VC-26 a VC-32, VC-38 |
| C-27 (ações) | VC-38 |
| C-28 (dados pessoais) | VC-40 |
| C-29, C-30 (tempo) | VC-34, VC-35 |
| C-31 (orçamento) | VC-46 |
| C-32 (visibilidade) | VC-47 |
| C-33 (divergência não registrada) | VC-43 |
| C-34 (comparação em S6) | VC-14 |
| C-35 (fórmula sem substituição) | VC-09, VC-20 |
| C-36 (FAQ sobre documento ausente) | VC-22 |
| C-37 (metadado ausente) | VC-48 |
| C-38 (normalização e tabelas) | VC-02, VC-11, VC-39, VC-66 |
| C-39 (granularidade por seção) | VC-01 |
| C-40 (precedência de situações) | VC-09, VC-14, VC-44 |
| C-41, C-42 (partes) | VC-44, VC-45 |
| C-43, C-44 (segundo turno) | VC-41, VC-50 |
| C-45 (domínio) | VC-13, VC-24, VC-26 |
| C-46, C-47 (área e indicação) | VC-36, VC-55 |
| C-48 (contrato) | VC-49 |
| C-49 (fallbacks) | VC-35, VC-45, VC-51, VC-52, VC-53, VC-54 |
| C-50, C-51, C-52 (registro, dados, identificador) | VC-37, VC-40 |
| C-53 (ordem de exibição) | VC-47 |
| C-54 (título neutro) | VC-13, VC-22 |
| C-55 (rótulos) | VC-49 |
| C-56 (metadados exibidos) | VC-48, VC-49, VC-56 |
| C-57 (feedback fora do endpoint) | VC-55 |
| C-58 (chunk não citável) | VC-11, VC-66 |
| C-59 (incidente crítico não classificado) | VC-57, VC-62 |

---

## 7. Pendências para a implementação

As decisões pendentes estão em §4.3 (DP) e §4.4 (DT), cada uma com o comportamento provisório que permite implementar. Antes do primeiro sprint, recomenda-se fechar ou aceitar formalmente o provisório de: DT-17 (conferência com a ESPEC-V2 e origem dos 30 s e dos 15%), DT-08 (P-01), DT-09 (ADR-0002), DT-02 e DT-10 (áreas), DT-01 (janela de S7), DT-16 (massa de teste do VC-43) e DT-15 (mockup v2).

## Histórico de Iteração

O histórico completo desta iteração, com a rastreabilidade de cada item da revisão e a reconciliação com o `01-bounded-contexts-v1.md`, está em `07-historico-iteracao.md`.
