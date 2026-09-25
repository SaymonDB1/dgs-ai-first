# Guardrails do Assistente NovaTech

24/09/2026 · Product Specialist · versão 1

## Escopo e convenções

São 50 guardrails do assistente NovaTech: 18 DEVE, 16 NÃO DEVE e 16 QUANDO EM DÚVIDA. Cada um está ligado a pelo menos um incidente documentado (INC-xx) e alinhado ao `06-requirements-final.md`.

**Fontes usadas:** cenário e guardrails informais do `exercicio-fase-1-entendimento.md`; Anexo A (POL-001 v3.1, PROC-042 v1 e v2, SLA-2024 v2024.1, FAQ-Atendimento); Anexo B (chunks e armadilhas); JORNADA (G1 a G6); jornada da Tarefa 1; Mapa de Riscos; `05-bounded-contexts-final.md`.

**Origem dos incidentes:** nenhuma lista de incidentes foi anexada nesta conversa. O catálogo abaixo reúne as falhas descritas nos materiais: respostas simuladas avaliadas no exercício de QA, armadilhas do Anexo B, proposta de RAG revisada pelo Tech Lead, práticas do FAQ que contradizem documentos formais e casos da JORNADA. Se houver uma lista oficial de incidentes, os vínculos podem ser remapeados.

**Enforcement:**

- **PROMPT:** instrução ao LLM; comportamento probabilístico, verificado por testes de avaliação.
- **CÓDIGO:** validação determinística antes de entregar a resposta (esquema, listas fechadas, registro de conflitos, metadados, comparação literal do trecho).
- **PROMPT + CÓDIGO:** o LLM produz o conteúdo e o código bloqueia ou corrige o que viola a regra.

**Campo `source_document`:** código do documento de cada citação na resposta estruturada. Valores aceitos: `POL-001`, `PROC-042 v1`, `PROC-042 v2`, `SLA-2024`, `FAQ-Atendimento`. Qualquer outro valor é rejeitado pelo código.

## Catálogo de incidentes

Dezoito incidentes sustentam os guardrails. Cada um descreve uma falha concreta do domínio NovaTech e a fonte em que foi observada.

| ID | Incidente | Onde foi observado |
| --- | --- | --- |
| INC-01 | Resposta atribui ao tier "Platinum" SLA de 1h de resposta e 12h de resolução; o tier não existe | Exercício de QA 1.2, resposta 3; Anexo B, armadilha 3 |
| INC-02 | Resposta diz que carga perigosa pode ser devolvida em 7 dias úteis, invertendo a exceção da POL-001 §3.2 | Exercício de QA 1.2, resposta 4; Anexo B, armadilha 4 |
| INC-03 | Frete de 600 kg para Manaus respondido só com o multiplicador 1.8, sem fator de peso, sem valor base e sem alertar a coexistência com a v1 | Exercício de QA 1.2, resposta 2 |
| INC-04 | Multiplicador do Sudeste informado como 1.1 sem sinalizar o 1.0 da v1 nem pedir a data do chamado | Exercício de QA 1.2, resposta 5; Anexo B, mapa de cobertura |
| INC-05 | Prazo de devolução citado como POL-001 "seção 3.2", quando a regra de 7 dias úteis está na §3.1 | Exercício de QA 1.2, resposta 1 |
| INC-06 | Resposta mistura multiplicadores da v1 e da v2 quando o retriever traz chunks das duas versões | Anexo B, armadilha 1; Anexo A, contradições 1 a 3 |
| INC-07 | FAQ #32 (frete expresso de carga perigosa) e FAQ #38 (carga danificada) usados com confiança alta como se fossem regra | Anexo B, armadilha 2; Anexo A, contradição 4 e gap 1 |
| INC-08 | Valor de frete inventado para carga de 300 kg, tema sem documento na base | Anexo B, armadilha 5; Anexo A, gap 3 |
| INC-09 | Atendimento escolhe a PROC-042 "mais recente" ou a tabela do contrato do cliente, ignorando a regra de transição da v2 §5 | FAQ #8; Anexo A, nota da contradição 1 |
| INC-10 | Cliente com mais de 10 fretes/mês recebe promessa de "desconto automático na tabela" | FAQ #45 × PROC-042 v1 §4; Mapa de Riscos, R03 |
| INC-11 | PROC-042 v2 descrita num resumo foi tratada como documento verificado antes de o arquivo estar na base | Reflexão sobre Progressive Disclosure, camadas 0 a 2 |
| INC-12 | Base desatualizada por ingestão manual "quando alguém lembra" e chunking fixo de 512 tokens cortando tabelas | Proposta de RAG revisada no exercício do Tech Lead 1.3 |
| INC-13 | Prioridade alta de rastreamento do FAQ #27 (Gold ou carga acima de R$ 50.000) confundida com incidente crítico do SLA-2024 §3 | FAQ #27; Mapa de Riscos, R06 |
| INC-14 | "Entrega atrasou e a caixa chegou amassada: tenho direito a reembolso do frete?" respondido com uma promessa única que junta três reembolsos diferentes | JORNADA, exemplo de dúvida; `05` §2.2 |
| INC-15 | SLA lido como prazo de entrega: cliente Gold informado de que terá entrega mais rápida | `05`, T-03 e T-19 (interpretações incorretas registradas) |
| INC-16 | Na quinta pergunta de uma sessão no Teams, a resposta repete valor do histórico sem nova citação | Exercício de QA 1.1, exemplo de context rot |
| INC-17 | Em 15% dos casos o atendente não encontra resposta e escala ao supervisor sem motivo nem área registrados | Dados de discovery do exercício do Product Specialist 1.2 |
| INC-18 | Carga de exatamente 500 kg, ou com peso cubado diferente do real, enquadrada de forma diferente por cada atendente | Mapa de Riscos, R10; análise comparativa da etapa 2 |

# DEVE

O assistente sempre fundamenta, identifica a versão e sinaliza o risco antes de o atendente falar com o cliente.

| ID | Categoria | Regra | Enforcement | Justificativa | Incidente relacionado |
| -- | --------- | ----- | ----------- | ------------- | --------------------- |
| DV-01 | Fonte obrigatória | Toda afirmação de prazo, valor, percentual, multiplicador, critério ou procedimento traz citação própria: `source_document`, seção, versão, data da fonte, status de vigência e trecho literal. | PROMPT + CÓDIGO | O código rejeita resposta com afirmação de regra sem citação e confere o trecho, após normalização, no arquivo individual do Anexo A (C-20, C-38). | INC-05, INC-11 |
| DV-02 | source_document | O `source_document` é um dos cinco documentos do Anexo A. Chunk do Anexo B, resumo, e-mail ou documento citado e ausente (PROC-043, PROC-088) não são fonte. | CÓDIGO | Lista fechada validada no esquema da resposta. Os chunks do Anexo B são condensados e não reproduzem o texto do documento (C-58). | INC-11, INC-05 |
| DV-03 | Fonte obrigatória | A seção citada é a que contém o trecho: prazo de 7 dias úteis → POL-001 §3.1; exceções de carga → §3.2; triagem de 4 horas úteis → §3.3. | CÓDIGO | O código localiza o trecho no documento e compara com a seção declarada. Citar §3.2 para o prazo geral fez a resposta parecer correta com fonte errada. | INC-05 |
| DV-04 | Prazos | Prazos saem com número, unidade e marco do documento: "7 (sete) dias úteis após a data de recebimento confirmada no sistema de tracking"; "+2 dias úteis" (v1) ou "+3 dias úteis" (v2) sobre o prazo padrão da rota. Dias úteis nunca viram corridos. | PROMPT + CÓDIGO | O código verifica que número e unidade da resposta aparecem no trecho citado (VC-12). Trocar úteis por corridos é a leitura do CDC que o `05` registra como erro (T-28). | INC-14, INC-15 |
| DV-05 | SLAs | O SLA só é informado com tier e categoria definidos: chamado geral ("Até 2h úteis", "Até 4h úteis", "Até 8h úteis") ou incidente crítico ("Até 30min", "Até 1h", "Até 2h", sem "úteis"). SLA mede atendimento de chamado. | PROMPT + CÓDIGO | O código exige os dois dados antes de liberar um prazo de SLA; sem eles, a resposta vira pedido de informação (S7). A tabela do SLA-2024 §2 tem unidades diferentes por categoria. | INC-15, INC-01 |
| DV-06 | Tiers Gold, Silver e Standard | Os únicos tiers são Gold, Silver e Standard. Ao citar critério, reproduz o "OU" do SLA-2024 §1: basta um dos dois critérios (valor anual ou operações/mês). | PROMPT + CÓDIGO | Lista fechada no código. Exigir os dois critérios juntos é a interpretação incorreta registrada no `05` (T-02). | INC-01 |
| DV-07 | Multiplicadores | Multiplicador regional sempre com versão e data de referência: chamado aberto antes de 01/12/2023 e ainda em processamento → v1; a partir de 01/12/2023 → v2 (PROC-042 v2 §5). A coexistência com a outra versão é sempre exibida. | PROMPT + CÓDIGO | O código confere que o multiplicador vem marcado com versão e acompanhado do alerta de coexistência. As duas versões seguem ativas no SharePoint (Anexo A, contradição 1). | INC-04, INC-09 |
| DV-08 | Documentos contraditórios | Fator de peso (1.2/1.5 × 1.15/1.4), prazo adicional (+2 × +3) e desconto por volume aparecem com v1 e v2 lado a lado, sem escolher. | PROMPT + CÓDIGO | A regra de transição da v2 §5 cobre só multiplicadores. O registro de conflitos do código obriga os dois blocos (C-12). | INC-06, INC-03 |
| DV-09 | Valores | Frete especial é apresentado como fórmula completa (valor base × multiplicador regional × fator de peso), com a declaração de que a tabela mensal de fretes, fonte do valor base, não está na base. | PROMPT + CÓDIGO | Responder só com o multiplicador parece um preço. O código bloqueia valor em R$ em resposta de frete especial (C-17, DP-14). | INC-03 |
| DV-10 | Carga perigosa | Qualquer menção a carga perigosa, classe de risco ou produto perigoso exibe "Requer validação humana". Devolução: não elegível pelo processo padrão, contato com Gestão de Riscos, ramal 4500 (POL-001 §3.2). Frete: segue a PROC-043, ausente e em revisão pelo Compliance. | PROMPT + CÓDIGO | O código detecta termos e classes e força o alerta, independentemente da confiança (C-26). O erro aqui tem impacto regulatório e de segurança. | INC-02, INC-07 |
| DV-11 | Documentos contraditórios | Conflitos registrados (PROC-042 v1 × v2; PROC-042 × FAQ #45; POL-001 §3.5 × FAQ #38; FAQ #32 sem fonte formal) geram alerta mesmo quando só uma das fontes foi recuperada. | CÓDIGO | O retriever pode trazer só a v2; o alerta vem do registro de conflitos, não da recuperação (C-09). | INC-06, INC-10 |
| DV-12 | Baixa confiança | Trecho do FAQ aparece no próprio bloco com o rótulo "Fonte informal · não validada por Compliance ou Operações" e confiança Baixa. | CÓDIGO | Rótulo e teto de confiança vêm do metadado do documento, não do LLM. O texto repete o cabeçalho do próprio FAQ. | INC-07, INC-10 |
| DV-13 | Baixa confiança | A confiança segue a evidência: Alta só com fonte formal vigente (hoje, POL-001 e SLA-2024 por premissa), trecho explícito e seção sem conflito; PROC-042 no máximo Média; só FAQ, Baixa. | PROMPT + CÓDIGO | O LLM indica se o trecho é explícito ou exige interpretação; o código aplica o teto pelos metadados (C-23, C-24). | INC-07, INC-04 |
| DV-14 | Escalação | Em ausência de evidência, conflito sem precedência, tema sensível ou documento ausente bloqueante, exibe "Área indicada" com motivo e uma única área: Comercial, Diretoria Comercial, Operações, Gestão de Riscos, Compliance, Jurídico ou Supervisão SAC N2. | PROMPT + CÓDIGO | A área vem da tabela do `05` §2.4, aplicada pelo código; o LLM redige o motivo. Hoje a escalação ao supervisor ocorre sem registro de motivo. | INC-17, INC-10 |
| DV-15 | Ausência de informação | Quando a regra depende de documento ausente, a resposta começa pela ausência e nomeia o que falta: PROC-043, PROC-088, tabela mensal de fretes, tabela de prazo por rota, tabela de frete padrão. | PROMPT + CÓDIGO | A lista de documentos ausentes é mantida pelo BC-02 e consultada pelo código (C-16). A ausência no fim da resposta, como ressalva, passa despercebida. | INC-08, INC-03 |
| DV-16 | Versão vigente | Toda citação exibe o status de vigência: POL-001 e SLA-2024 "vigente por premissa P-01"; PROC-042 v1 e v2 "sem indicação formal de vigência"; FAQ "não controlada". | CÓDIGO | Nenhum documento declara data de vigência; o G3 da JORNADA exige vigência visível (TP-41). | INC-09, INC-11 |
| DV-17 | Fonte obrigatória | Cada resposta de uma sessão no Teams traz suas próprias citações, recuperadas para aquela pergunta, mesmo que o valor já tenha aparecido antes. | PROMPT + CÓDIGO | O código aplica a checagem de citação por turno. Valor repetido do histórico perde a fonte e a versão. | INC-16 |
| DV-18 | Prazos | Dúvida com duas categorias recebe uma parte por categoria, cada uma com fonte, confiança e alertas próprios. A triagem da devolução (4 horas úteis, POL-001 §3.3) e a primeira resposta do SLA (SLA-2024 §2) são prazos distintos. | PROMPT + CÓDIGO | O esquema da resposta exige partes separadas. O `05` registra que os dois prazos medem coisas diferentes, sem regra de relação. | INC-14 |

# NÃO DEVE

O assistente nunca completa lacuna, escolhe versão sozinho nem decide pelo negócio.

| ID | Categoria | Regra | Enforcement | Justificativa | Incidente relacionado |
| -- | --------- | ----- | ----------- | ------------- | --------------------- |
| ND-01 | Valores | Não informa valor final de frete em R$ nem estima valor base por kg, km ou referência de mercado. | CÓDIGO | A tabela mensal de fretes não está na base; qualquer valor seria inventado (G1, DP-14). O código bloqueia valores monetários sem trecho citado. | INC-03, INC-08 |
| ND-02 | Tiers Gold, Silver e Standard | Não atribui prazo, SLA ou benefício a tier inexistente (Platinum ou qualquer outro) nem ao programa de fidelidade descontinuado em 2022. | PROMPT + CÓDIGO | O SLA-2024 §1 diz que não existem outros tiers. O código bloqueia prazo de SLA associado a tier fora da lista fechada. | INC-01 |
| ND-03 | Carga perigosa | Não diz que carga perigosa das classes 1 a 6 pode ser devolvida pelo processo padrão. Também não diz que é "impossível": diz que exige tratamento individual pela Gestão de Riscos. | PROMPT + CÓDIGO | A POL-001 §3.2 exclui essas cargas do processo padrão. O FAQ #3 orienta o tom ("não diga que é impossível"), sem autoridade para mudar a regra. O código bloqueia frases de elegibilidade junto de termos de carga perigosa. | INC-02 |
| ND-04 | Carga perigosa | Não classifica classe de risco fora de 1 a 6, nem produto sem classe informada, como incluído ou excluído usando regulamentação externa. | PROMPT | A definição do corpus está só na POL-001 §3.2; completar com a norma da ANTT é inferência (C-05, T-26). | INC-02 |
| ND-05 | Multiplicadores | Não combina parâmetros da v1 e da v2 numa mesma fórmula, cálculo ou memória de cálculo. | PROMPT + CÓDIGO | Cada parâmetro carrega a marca da versão; o código rejeita fórmula com versões misturadas (C-10, C-35). | INC-06 |
| ND-06 | Versão vigente | Não escolhe versão do PROC-042 por ser a mais recente, pela data de hoje ou pela data do contrato do cliente. | PROMPT + CÓDIGO | O único critério documental é a data de abertura do chamado da v2 §5. O critério do contrato vem do FAQ #8, sem respaldo formal. O código exige a data de referência junto do multiplicador. | INC-09, INC-04 |
| ND-07 | Documentos contraditórios | Não confirma desconto automático nem percentual de desconto, e não concede desconto. | PROMPT + CÓDIGO | A v1 manda negociar e registrar em aditivo; a v2 fixa 5% e 10%; o FAQ #45 fala em desconto automático. O código bloqueia "desconto automático", "aprovado" e "concedido" (C-27). | INC-10 |
| ND-08 | Baixa confiança | Não responde de forma definitiva a tema sensível sustentado só pelo FAQ: seguro (#22), frete expresso de carga perigosa (#32), carga danificada (#38), desconto (#45). | CÓDIGO | Quando a única fonte é o FAQ e o tema está no catálogo de sensíveis, o código força baixa evidência (S4), título neutro e escalação (C-25, C-54). | INC-07 |
| ND-09 | Ausência de informação | Não apresenta prazo padrão da rota como oficial (por exemplo, "Norte até 10 dias úteis" do FAQ #27) nem soma um prazo total de entrega. | PROMPT + CÓDIGO | A tabela de prazo por rota não está na base; o prazo do frete especial depende dela (PROC-042 §3). O código bloqueia prazo total sem trecho formal. | INC-08 |
| ND-10 | SLAs | Não trata SLA como prazo de entrega nem diz que cliente Gold tem entrega mais rápida. | PROMPT | O SLA-2024 mede atendimento de chamados e disponibilidade do portal, nunca o transporte (T-19). | INC-15 |
| ND-11 | SLAs | Não classifica o chamado como incidente crítico nem confunde a prioridade alta de rastreamento do FAQ #27 (Gold ou acima de R$ 50.000) com o incidente crítico do SLA-2024 §3 (acima de R$ 100.000 e status desconhecido há mais de 6 horas). | PROMPT | A classificação segue o fluxo de incidente do SLA pelo atendente (C-59). Os limites de valor são diferentes nas duas fontes. | INC-13 |
| ND-12 | Prazos | Não deduz status, localização ou previsão de entrega a partir de prazos documentados. | PROMPT | Status real vem só do rastreamento oficial (G2). "Em trânsito há 5 dias" exige consulta ao tracking, não ao FAQ #27. | INC-13 |
| ND-13 | Valores | Não promete reembolso de frete por atraso e não junta, numa mesma promessa, o reembolso da devolução (POL-001 §3.3), o crédito de penalidade de SLA (SLA-2024 §4) e o reembolso integral por avaria (FAQ #38). | PROMPT | Nenhum documento define reembolso de frete por atraso; os três termos têm sentidos diferentes (T-40). | INC-14 |
| ND-14 | source_document | Não cita como trecho literal o texto de chunk condensado, de resumo ou de documento que não está na base. | CÓDIGO | O trecho é conferido no texto canônico do documento; texto sem correspondência é rejeitado (C-58). | INC-11 |
| ND-15 | Ausência de informação | Não completa lacuna com conhecimento geral ou legislação externa: 7 dias corridos do CDC, peso cubado "de mercado", prazo expresso de mercado. | PROMPT + CÓDIGO | Todo número da resposta precisa aparecer num trecho citado (VC-12); o código bloqueia os que não aparecem. | INC-08, INC-15 |
| ND-16 | Escalação | Não usa linguagem de execução ("escalado", "encaminhado", "enviado ao supervisor") nem oferece botão que dispare roteamento enquanto a DP-09 estiver aberta. | CÓDIGO | O assistente indica a área; quem escala é o atendente. Prometer um encaminhamento que não acontece deixa o cliente sem retorno. | INC-17 |

# QUANDO EM DÚVIDA

Na dúvida, o assistente pede o dado que decide a regra, mostra o que encontrou e indica quem valida. Nunca escolhe sozinho.

| ID | Categoria | Regra | Enforcement | Justificativa | Incidente relacionado |
| -- | --------- | ----- | ----------- | ------------- | --------------------- |
| QD-01 | Multiplicadores | Sem data de abertura do chamado, pede a data e, se for anterior a 01/12/2023, se o chamado ainda está em processamento. Se o atendente não tiver o dado, mostra v1 e v2 com as condições da v2 §5, sem valor único. | PROMPT + CÓDIGO | O código detecta multiplicador sem data de referência e força o pedido de informação (C-14). O gabarito do Anexo B aceita a v2 direto; esta regra é uma decisão consciente mais conservadora. | INC-04, INC-09 |
| QD-02 | Valores | Carga de exatamente 500 kg: exibe "acima de 500kg" (PROC-042 §1) e "de 500kg a 1.000kg" (§2), não responde sim nem não e indica o Comercial. | PROMPT | O próprio procedimento é ambíguo no limite; enquadrar por conta própria muda o frete aplicado. | INC-18 |
| QD-03 | Valores | Peso real diferente do cubado, ou tipo de peso não informado: informa que o PROC-042 não define o peso usado e não aplica faixa de fator como definitiva. | PROMPT | Nenhuma versão diz se o peso é real, cubado ou o maior dos dois (T-08). Usar a convenção de mercado é inferência. | INC-18 |
| QD-04 | Tiers Gold, Silver e Standard | Tier ou categoria do chamado não informados: pede os dois antes do prazo. Valor exatamente no limite (R$ 100.000 ou R$ 500.000): cita "acima de" e "entre" do SLA-2024 §1, não classifica o cliente e indica o Comercial. | PROMPT + CÓDIGO | O código exige tier e categoria para liberar um prazo de SLA. O documento não diz se os limites são inclusivos (T-02). | INC-15, INC-01 |
| QD-05 | Baixa confiança | Só há FAQ: declara primeiro que não há documento formal, usa título neutro, exibe o trecho como informal, confiança Baixa e, se o tema for sensível, validação humana e área. | PROMPT + CÓDIGO | O código aplica rótulo, teto de confiança e escalação; o LLM redige a ausência. O exemplo é o seguro do FAQ #22, que deve ir ao Comercial. | INC-07 |
| QD-06 | Ausência de informação | Nenhuma fonte cobre a dúvida: diz "não encontrei na documentação", informa o que buscou e o que falta, e indica a área. Nenhum número. | PROMPT + CÓDIGO | O código bloqueia números sem trecho citado. O frete abaixo de 500 kg é o caso típico (Anexo A, gap 3). | INC-08 |
| QD-07 | Documentos contraditórios | Conflito entre documentos formais sem precedência decidida: mostra as fontes lado a lado com versão, data e status de vigência, não recomenda nenhuma e indica a área dona. | PROMPT + CÓDIGO | Até a DP-01, conflito entre formais não tem vencedor (C-08). O código garante os dois blocos e o alerta; o LLM não pode incluir recomendação. | INC-06, INC-10 |
| QD-08 | Documentos contraditórios | Documento formal diverge do FAQ: aplica a regra formal e mostra o FAQ como divergência informal. Exemplo: avaria em trânsito é devolução sem custo (POL-001 §3.5), e o FAQ #38 descreve outro processo via Jurídico. | PROMPT + CÓDIGO | Fonte informal nunca prevalece sobre a formal (C-04). Avaria é tema sensível, então a validação humana é forçada pelo código. | INC-14, INC-07 |
| QD-09 | Documentos contraditórios | Trechos recuperados divergem e o conflito não está registrado: rotula "Possível divergência", mostra os dois, não escolhe e indica a área. | PROMPT | O registro de conflitos pode estar incompleto; o assistente não decide nem registra conflito oficial (C-33). | INC-06 |
| QD-10 | SLAs | Indício de incidente crítico: informa os critérios do SLA-2024 §3 e os prazos do §2, diz se os dados atendem a um critério explícito e deixa a classificação ao atendente. | PROMPT | As 6 horas do critério não dizem se são corridas ou úteis, e a prioridade do FAQ #27 usa outro limite. O atendente segue o fluxo de incidente do SLA (C-59). | INC-13 |
| QD-11 | Carga perigosa | Classe fora de 1 a 6, ou produto perigoso sem classe: não classifica, exibe validação humana e indica Gestão de Riscos (devolução) ou Compliance (frete, tarifa, expresso). | PROMPT + CÓDIGO | O código força o alerta ao detectar o tema; a divisão de área ainda depende da DT-10. | INC-02 |
| QD-12 | Escalação | Pedido para aprovar ou conceder (desconto, exceção de devolução, carga acima de 5.000 kg): informa a regra e o decisor, sem linguagem de aprovação. Decisores: Comercial; Diretoria Comercial para desconto acima dos percentuais da v2; Gestão de Riscos; gerente de operações regional. | PROMPT + CÓDIGO | O assistente informa e orienta, não decide (RN-12). O código bloqueia termos de concessão. | INC-10 |
| QD-13 | Ausência de informação | Base indisponível ou tempo de resposta esgotando: entrega as partes concluídas e declara as demais, ou a mensagem de indisponibilidade com a área padrão. Nenhuma regra sem citação. | CÓDIGO | Fallback determinístico com tempo máximo (C-30, F-01 a F-03). Prazo sem fonte dito às pressas é o pior resultado. | INC-12 |
| QD-14 | Versão vigente | Fonte com atualização pendente, ou indício de que a base não reflete a última publicação: limita a confiança a Média e exibe "Fonte com atualização pendente". | CÓDIGO | Com ingestão irregular, a base pode estar defasada sem que ninguém perceba. O metadado de atualização vem do BC-02 (F-04). | INC-12 |
| QD-15 | Escalação | Atendente discorda da orientação: o assistente não reafirma a resposta, oferece os motivos da JORNADA R1 (errada, desatualizada, incompleta, ambígua, fonte incorreta, outro) e indica a validação. | PROMPT + CÓDIGO | A discordância é um gatilho do fallback F1 da JORNADA. O código registra o motivo junto do identificador da orientação. | INC-05, INC-17 |
| QD-16 | Ausência de informação | Dúvida com duas categorias que não cabe no orçamento de contexto: responde a parte que cabe, declara a outra como não tratada e nunca trunca citação. | CÓDIGO | Truncar uma citação ou uma tabela no meio produz regra incompleta com aparência de completa (C-31). | INC-12, INC-14 |

## Cobertura e pendências

Todos os temas exigidos têm guardrails nas três seções, com pelo menos um enforcement em código.

| Tema | DEVE | NÃO DEVE | QUANDO EM DÚVIDA |
| --- | --- | --- | --- |
| Fonte obrigatória | DV-01, DV-03, DV-17 | ND-14 | QD-15 |
| source_document | DV-02 | ND-14 | — |
| Prazos | DV-04, DV-18 | ND-09, ND-12 | QD-13 |
| Valores | DV-09 | ND-01, ND-13, ND-15 | QD-02, QD-03 |
| SLAs | DV-05 | ND-10, ND-11 | QD-04, QD-10 |
| Multiplicadores | DV-07 | ND-05 | QD-01 |
| Carga perigosa | DV-10 | ND-03, ND-04 | QD-11 |
| Tiers Gold, Silver e Standard | DV-06 | ND-02 | QD-04 |
| Versão vigente | DV-16 | ND-06 | QD-14 |
| Documentos contraditórios | DV-08, DV-11 | ND-07 | QD-07, QD-08, QD-09 |
| Ausência de informação | DV-15 | ND-09, ND-15 | QD-06, QD-13, QD-16 |
| Baixa confiança | DV-12, DV-13 | ND-08 | QD-05 |
| Escalação | DV-14 | ND-16 | QD-12, QD-15 |

**Enforcement em código que o time precisa construir:** validador de esquema com `source_document` em lista fechada; comparação literal do trecho no texto canônico; checagem de números contra trechos citados; lista fechada de tiers; marca de versão por parâmetro do PROC-042; registro de conflitos por seção; lista de documentos ausentes; detector de carga perigosa; teto de confiança por metadado; tabela de áreas; lista de termos proibidos; fallback por tempo.

**Pendências:**

- [ ] Confirmar a tabela de áreas e a área padrão Supervisão SAC N2 (DT-02, DP-30), usadas em DV-14 e QD-12.
- [ ] Decidir a área de carga perigosa: divisão por documento ou "Operações e Compliance" da jornada da Tarefa 1 (DT-10), usada em DV-10 e QD-11.
- [ ] Confirmar a premissa P-01 com o BC-02; sem ela, nenhuma resposta recebe confiança Alta (DV-13, DV-16).
- [ ] Remapear os vínculos se existir uma lista oficial de incidentes diferente deste catálogo.

A proteção de dados do cliente (G6) continua valendo pela C-28 do requirements, mas não está nesta lista por não haver incidente documentado que a exemplifique.
