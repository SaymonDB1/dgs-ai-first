# Product Rules · Guardrails do Assistente NovaTech

24/09/2026 · Product Specialist · versão final auditada · substitui `guardrails-v1.md`, `guardrails-final.md` e `product-rules-guardrails-v1.md` · auditoria em `product-rules-guardrails-audit.md`

## Escopo e convenções

São 64 guardrails do assistente NovaTech: 23 DEVE, 21 NÃO DEVE e 20 QUANDO EM DÚVIDA. Os IDs da v1 foram mantidos; ND-14 foi incorporado ao DV-02 e QD-16 foi retirado por ser requisito técnico do pipeline (C-31). Cada um está ligado a pelo menos um incidente documentado (INC-xx) e alinhado ao `06-requirements-final.md`.

**Fontes usadas:** cenário e guardrails informais do `exercicio-fase-1-entendimento.md`; Anexo A (POL-001 v3.1, PROC-042 v1 e v2, SLA-2024 v2024.1, FAQ-Atendimento); Anexo B (chunks e armadilhas); JORNADA (G1 a G6); jornada da Tarefa 1; Mapa de Riscos; `05-bounded-contexts-final.md`.

**Origem dos incidentes:** não foi fornecida uma lista oficial de incidentes. O catálogo abaixo reúne as falhas descritas nos materiais: respostas simuladas avaliadas no exercício de QA, armadilhas do Anexo B, proposta de RAG revisada pelo Tech Lead, práticas do FAQ que contradizem documentos formais e casos da JORNADA. Se houver uma lista oficial de incidentes, os vínculos podem ser remapeados.

**Enforcement:**

- **PROMPT:** instrução ao LLM; comportamento probabilístico, verificado por testes de avaliação.
- **CÓDIGO:** validação determinística antes de entregar a resposta (esquema, listas fechadas, registro de conflitos, metadados, comparação literal do trecho).
- **PROMPT + CÓDIGO:** o LLM produz o conteúdo e o código bloqueia ou corrige o que viola a regra.

**Categorias:** os 13 temas exigidos (fonte obrigatória, source_document, prazos, valores, SLAs, multiplicadores, carga perigosa, tiers, versão vigente, documentos contraditórios, ausência de informação, baixa confiança, escalação), mais "Exceções de devolução" (cadeia de frio, lacre, prazo expirado) e "Dados pessoais". Cada guardrail tem uma única categoria; a matriz de cobertura é derivada dessa coluna.

**Campo `source_document`:** código do documento de cada citação na resposta estruturada. Valores aceitos: `POL-001`, `PROC-042 v1`, `PROC-042 v2`, `SLA-2024`, `FAQ-Atendimento`. Qualquer outro valor é rejeitado pelo código.

## Catálogo de incidentes

Vinte e seis incidentes sustentam os guardrails. Cada um descreve uma falha concreta do domínio NovaTech e a fonte em que foi observada.

| ID | Incidente | Onde foi observado |
| --- | --- | --- |
| INC-01 | Resposta atribui ao tier "Platinum" SLA de 1h de resposta e 12h de resolução; o tier não existe | Exercício de QA 1.2, resposta 3; Anexo B, armadilha 3 |
| INC-02 | Resposta diz que carga perigosa pode ser devolvida em 7 dias úteis, invertendo a exceção da POL-001 §3.2 | Exercício de QA 1.2, resposta 4; Anexo B, armadilha 4 |
| INC-03 | Frete de 600 kg para Manaus respondido só com o multiplicador 1.8, sem fator de peso, sem valor base, sem alertar a coexistência com a v1 e citando "seção 2" em vez da §2.1 | Exercício de QA 1.2, resposta 2 |
| INC-04 | Multiplicador do Sudeste informado como 1.1 sem sinalizar o 1.0 da v1, sem pedir a data do chamado e citando "seção 2" em vez da §2.1 | Exercício de QA 1.2, resposta 5; Anexo B, mapa de cobertura |
| INC-05 | Prazo de devolução citado como POL-001 "seção 3.2", quando a regra de 7 dias úteis está na §3.1 | Exercício de QA 1.2, resposta 1 |
| INC-06 | Resposta mistura multiplicadores da v1 e da v2 quando o retriever traz chunks das duas versões | Anexo B, armadilha 1; Anexo A, contradições 1 a 3 |
| INC-07 | FAQ #32 (frete expresso de carga perigosa) e FAQ #38 (carga danificada) usados com confiança alta como se fossem regra | Anexo B, armadilha 2; Anexo A, contradição 4 e gap 1 |
| INC-08 | Valor de frete inventado para carga de 300 kg, tema sem documento na base | Anexo B, armadilha 5; Anexo A, gap 3 |
| INC-09 | Atendimento escolhe a PROC-042 "mais recente" ou a tabela do contrato do cliente, ignorando a regra de transição da v2 §5 | FAQ #8; Anexo A, nota da contradição 1 |
| INC-10 | Cliente com mais de 10 fretes/mês recebe promessa de "desconto automático na tabela" | FAQ #45 × PROC-042 v1 §4; Mapa de Riscos, R03 |
| INC-11 | PROC-042 v2 descrita num resumo foi tratada como documento verificado antes de o arquivo estar na base | Reflexão sobre Progressive Disclosure, camadas 0 a 2 |
| INC-12 | Base desatualizada por ingestão manual "quando alguém lembra" e chunking fixo de 512 tokens sem overlap, que pode cortar tabelas | Proposta de RAG revisada no exercício do Tech Lead 1.3 |
| INC-13 | Prioridade alta de rastreamento do FAQ #27 (Gold ou carga acima de R$ 50.000) confundida com incidente crítico do SLA-2024 §3 | FAQ #27; Mapa de Riscos, R06 |
| INC-14 | "Entrega atrasou e a caixa chegou amassada: tenho direito a reembolso do frete?" respondido com uma promessa única que junta três reembolsos diferentes | JORNADA, exemplo de dúvida; `05` §2.2 |
| INC-15 | SLA lido como prazo de entrega: cliente Gold informado de que terá entrega mais rápida | `05`, T-03 e T-19 (interpretações incorretas registradas) |
| INC-16 | Na quinta pergunta de uma sessão no Teams, a resposta repete valor do histórico sem nova citação | Exercício de QA 1.1, critério "falha de contexto" |
| INC-17 | Em 15% dos casos o atendente não encontra resposta e escala ao supervisor sem motivo nem área registrados | Dados de discovery do exercício do Product Specialist 1.2 |
| INC-18 | Carga de exatamente 500 kg, ou com peso cubado diferente do real, enquadrada de forma diferente por cada atendente | Mapa de Riscos, R10; análise comparativa da etapa 2 |
| INC-19 | Atendente promete exceção de devolução de carga perigosa porque "Riscos já autorizou" | FAQ #3; jornada da Tarefa 1, exemplo do F4 |
| INC-20 | Resposta automática "estamos verificando" contada como primeira resposta de SLA | FAQ #41; `05` T-20 |
| INC-21 | Crédito de penalidade oferecido já na primeira violação ou como compensação por atraso de entrega | `05` T-23 |
| INC-22 | Devolução de carga refrigerada aceita pelo relato do cliente, sem registro do sensor, e lacre violado recusado sem checar a documentação da entrega | Mapa de Riscos, R07; `05` T-33, T-34 |
| INC-23 | Embarque acima de 5.000 kg confirmado sem mencionar a aprovação prévia do gerente de operações regional | Mapa de Riscos, R15; `05` T-17 |
| INC-24 | Prazo de devolução contado da data informada pelo cliente ou da nota fiscal, não da data confirmada no tracking | Mapa de Riscos, R12; `05` T-29 |
| INC-25 | Consulta enviada ao assistente com CPF e endereço do destinatário copiados do chamado | Jornada da Tarefa 1, P2; JORNADA G6 |
| INC-26 | SLA-2024 sinalizado como possivelmente desatualizado só pelo nome | Jornada da Tarefa 1, G5 (superado); Mapa de Riscos, R11 |

# DEVE

O assistente sempre fundamenta, identifica a versão e sinaliza o risco antes de o atendente falar com o cliente.

| ID | Categoria | Regra | Enforcement | Justificativa | Incidente relacionado |
| -- | --------- | ----- | ----------- | ------------- | --------------------- |
| DV-01 | Fonte obrigatória | Toda afirmação de prazo, valor, percentual, multiplicador, critério ou procedimento traz citação própria: `source_document`, seção, versão, data da fonte, status de vigência e trecho literal. | PROMPT + CÓDIGO | O código rejeita resposta com afirmação de regra sem citação e confere o trecho, após normalização, no arquivo individual do Anexo A (C-20, C-38). | INC-05, INC-11 |
| DV-02 | source_document | O `source_document` é um dos cinco documentos do Anexo A, e o trecho citado vem do texto canônico desse documento. Chunk do Anexo B, resumo, e-mail ou documento citado e ausente (PROC-043, PROC-088) não são fonte nem trecho. | CÓDIGO | Lista fechada validada no esquema; trecho sem correspondência no arquivo individual é rejeitado. Os chunks do Anexo B são condensados e não reproduzem o documento (C-58). | INC-11, INC-05 |
| DV-03 | Fonte obrigatória | A seção citada é a que contém o trecho: prazo de 7 dias úteis → POL-001 §3.1; exceções de carga → §3.2; triagem de 4 horas úteis → §3.3; multiplicadores regionais → PROC-042 §2.1, não §2. | CÓDIGO | O código localiza o trecho no documento e compara com a seção declarada. Citar §3.2 para o prazo geral fez a resposta parecer correta com fonte errada. | INC-05, INC-03, INC-04 |
| DV-04 | Prazos | Prazos saem com número, unidade e marco do documento: "7 (sete) dias úteis após a data de recebimento confirmada no sistema de tracking"; "+2 dias úteis" (v1) ou "+3 dias úteis" (v2) sobre o prazo padrão da rota. Dias úteis nunca viram corridos. | PROMPT + CÓDIGO | O código verifica que número e unidade da resposta aparecem no trecho citado (VC-12). Trocar úteis por corridos é a leitura do CDC que o `05` registra como erro (T-28). | INC-14, INC-15 |
| DV-05 | SLAs | O SLA só é informado com tier e categoria definidos: primeira resposta em chamado geral ("Até 2h úteis", "Até 4h úteis", "Até 8h úteis") é diferente da primeira resposta em incidente crítico ("Até 30min", "Até 1h", "Até 2h", sem "úteis"); o mesmo vale para resolução. SLA mede atendimento de chamado. | PROMPT + CÓDIGO | O código exige os dois dados antes de liberar um prazo de SLA; sem eles, a resposta vira pedido de informação (S7). A tabela do SLA-2024 §2 tem unidades diferentes por categoria. | INC-15, INC-01 |
| DV-06 | Tiers Gold, Silver e Standard | Os únicos tiers são Gold, Silver e Standard. Ao citar critério, reproduz o "OU" do SLA-2024 §1: basta um dos dois critérios (valor anual ou operações/mês). | PROMPT + CÓDIGO | Lista fechada no código. Exigir os dois critérios juntos é a interpretação incorreta registrada no `05` (T-02). | INC-01 |
| DV-07 | Multiplicadores | Multiplicador regional sempre com versão e data de referência: chamado aberto antes de 01/12/2023 e ainda em processamento → v1; a partir de 01/12/2023 → v2 (PROC-042 v2 §5). A coexistência com a outra versão é sempre exibida. | PROMPT + CÓDIGO | O código confere que o multiplicador vem marcado com versão e acompanhado do alerta de coexistência. As duas versões seguem ativas no SharePoint (Anexo A, contradição 1). | INC-04, INC-09 |
| DV-08 | Documentos contraditórios | Onde os valores diferem, fator de peso (acima de 1.000 kg: 1.2/1.5 na v1 × 1.15/1.4 na v2), prazo adicional (+2 × +3) e desconto por volume aparecem com v1 e v2 lado a lado, sem escolher. Na faixa de 500 a 1.000 kg o fator é 1.0 nas duas versões e é citado com as duas, sem alerta de divergência. | PROMPT + CÓDIGO | A regra de transição da v2 §5 cobre só multiplicadores. O registro de conflitos do código obriga os dois blocos (C-12); a comparação de valores evita alerta falso na faixa idêntica (C-39). | INC-06, INC-03 |
| DV-09 | Valores | Frete especial é apresentado como fórmula completa (valor base × multiplicador regional × fator de peso), com a declaração de que a tabela mensal de fretes, fonte do valor base, não está na base. | PROMPT + CÓDIGO | Responder só com o multiplicador parece um preço. O código bloqueia valor em R$ que não aparece literalmente num trecho citado (C-17, DP-14). | INC-03 |
| DV-10 | Carga perigosa | Qualquer menção a carga perigosa, classe de risco (1 a 9) ou produto perigoso (inflamável, explosivo, corrosivo, tóxico, gás, infectante) exibe "Requer validação humana". Devolução: não elegível pelo processo padrão, contato com a Gestão de Riscos, ramal 4500 (POL-001 §3.2). Frete e frete expresso: seguem a PROC-043, ausente e em revisão pelo Compliance, que é a área indicada (provisório, DT-10). | PROMPT + CÓDIGO | O código detecta classes e termos e força o alerta, independentemente da confiança (C-26). O erro tem impacto regulatório e de segurança. | INC-02, INC-07 |
| DV-11 | Documentos contraditórios | Conflitos registrados (PROC-042 v1 × v2; PROC-042 × FAQ #45; PROC-042 v2 §5 × FAQ #8 (versão pelo contrato); POL-001 §3.5 × FAQ #38; FAQ #32 sem fonte formal) geram alerta mesmo quando só uma das fontes foi recuperada. | CÓDIGO | O retriever pode trazer só a v2; o alerta vem do registro de conflitos, não da recuperação (C-09). | INC-06, INC-10, INC-09 |
| DV-12 | Baixa confiança | Trecho do FAQ aparece no próprio bloco com o rótulo "Fonte informal · não validada por Compliance ou Operações" e confiança Baixa. | CÓDIGO | Rótulo e teto de confiança vêm do metadado do documento, não do LLM. O texto repete o cabeçalho do próprio FAQ. | INC-07, INC-10 |
| DV-13 | Baixa confiança | A confiança segue a evidência: Alta só com fonte formal vigente (hoje, POL-001 e SLA-2024 por premissa), trecho explícito e seção sem conflito; PROC-042 no máximo Média; só FAQ, Baixa. | PROMPT + CÓDIGO | O LLM indica se o trecho é explícito ou exige interpretação; o código aplica o teto pelos metadados (C-23, C-24). | INC-07, INC-04 |
| DV-14 | Escalação | Em ausência de evidência, conflito sem precedência, tema sensível ou documento ausente bloqueante, exibe "Área indicada" com motivo e uma única área, pela tabela do `05` §2.4: Comercial, Diretoria Comercial, Operações (gerente de operações regional para carga acima de 5.000 kg), Gestão de Riscos (exceções de devolução da POL-001 §3.2), Compliance, Jurídico ou Supervisão SAC N2 (área padrão). | PROMPT + CÓDIGO | A área vem da tabela, aplicada pelo código; o LLM redige o motivo. Hoje a escalação ao supervisor ocorre sem registro de motivo. | INC-17, INC-10 |
| DV-15 | Ausência de informação | Quando a regra depende de documento ausente, a resposta começa pela ausência e nomeia o que falta: PROC-043, PROC-088, tabela mensal de fretes, tabela de prazo por rota, tabela de frete padrão. | PROMPT + CÓDIGO | A lista de documentos ausentes é mantida pelo BC-02 e consultada pelo código (C-16). A ausência no fim da resposta, como ressalva, passa despercebida. | INC-08, INC-03 |
| DV-16 | Versão vigente | Toda citação exibe o status de vigência: POL-001 e SLA-2024 "vigente por premissa P-01"; PROC-042 v1 e v2 "sem indicação formal de vigência"; FAQ "não controlada". | CÓDIGO | Nenhum documento declara data de vigência; o G3 da JORNADA exige vigência visível (TP-41). | INC-09, INC-11 |
| DV-17 | Fonte obrigatória | Em sessão no Teams, dado do caso informado antes (data de abertura do chamado, tier, peso, região) só é reaproveitado dentro da janela de 30 minutos do pedido de informação (provisório, DT-01) e aparece repetido na resposta. Valor de regra já dito é recuperado e citado de novo. | PROMPT + CÓDIGO | O código confere citação por turno e a origem de cada dado do caso (mensagem atual ou pedido de informação vinculado, C-43). Valor repetido do histórico perde fonte e versão. | INC-16 |
| DV-18 | Prazos | Dúvida com duas categorias recebe uma parte por categoria, cada uma com fonte, confiança e alertas próprios. A triagem da devolução (4 horas úteis, POL-001 §3.3) e a primeira resposta do SLA (SLA-2024 §2) são prazos distintos. | PROMPT + CÓDIGO | O esquema da resposta exige partes separadas. O `05` registra que os dois prazos medem coisas diferentes, sem regra de relação. | INC-14 |
| DV-19 | Fonte obrigatória | Valor de tabela é citado por célula (tabela, linha, coluna, valor): SLA-2024 §1 (tier × critério), SLA-2024 §2 (métrica × tier) e PROC-042 §2.1 (região × multiplicador). O fator de peso, que está em texto corrido na §2, é citado como trecho literal da faixa inteira. Nunca linha parcial nem tabela cortada. | CÓDIGO | O validador confere se a célula existe no documento. Chunk que corta tabela produz valor sem o rótulo da linha ou da coluna (C-38). | INC-12, INC-03 |
| DV-20 | Exceções de devolução | Na devolução, informa as três exceções da POL-001 §3.2 com o critério exato: carga perigosa (classes 1 a 6); cadeia de frio rompida (fora da faixa da nota fiscal por mais de 30 minutos contínuos, conforme sensor IoT); lacre violado, salvo violação documentada na entrega com assinatura do motorista e do recebedor. As três vão à Gestão de Riscos, ramal 4500. Prazo expirado vai ao Comercial para negociação caso a caso (§3.5). | PROMPT + CÓDIGO | O código aplica a área e o alerta de exceção de devolução (tema sensível). Aceitar relato do cliente sem o registro do sensor, ou recusar lacre violado documentado, contraria a política. | INC-22 |
| DV-21 | Escalação | Carga acima de 5.000 kg sempre traz a aprovação prévia do gerente de operações regional (PROC-042 v1 §4 e v2 §4, iguais nas duas versões). O assistente não confirma embarque nem prazo sem mencioná-la. | PROMPT + CÓDIGO | O código detecta peso acima de 5.000 kg na dúvida e exige o bloco da aprovação. O procedimento não define canal, prazo nem efeito da recusa (T-17). | INC-23 |
| DV-22 | Prazos | O prazo de devolução conta da data de recebimento confirmada no sistema de tracking (POL-001 §3.1). A data informada pelo cliente, a da nota fiscal ou a da abertura do chamado não são o marco. | PROMPT | O marco muda o último dia do prazo. A data vem do tracking, que o assistente não consulta: ele informa qual data vale, não a calcula. | INC-24 |
| DV-23 | SLAs | Penalidade de SLA segue a contagem de violações no mesmo mês (SLA-2024 §4): primeira, registro interno sem impacto contratual; segunda, crédito de 5% sobre o valor do frete do chamado afetado; terceira ou mais, crédito de 10% e reunião com o gerente de conta (Gold) ou de operações (Silver e Standard). O crédito é por violação de SLA de atendimento, não por atraso de entrega. | PROMPT + CÓDIGO | O código confere percentuais contra o trecho citado. Oferecer crédito na primeira violação, ou como compensação de atraso, gera promessa sem base (T-23, T-40). | INC-21, INC-14 |

# NÃO DEVE

O assistente nunca completa lacuna, escolhe versão sozinho nem decide pelo negócio.

| ID | Categoria | Regra | Enforcement | Justificativa | Incidente relacionado |
| -- | --------- | ----- | ----------- | ------------- | --------------------- |
| ND-01 | Valores | Não informa valor final de frete em R$ nem estima valor base por kg, km ou referência de mercado. | CÓDIGO | A tabela mensal de fretes não está na base; qualquer valor seria inventado (G1, DP-14). O código bloqueia valor em R$ que não aparece literalmente num trecho citado, o que preserva critérios como os R$ 100.000 do SLA-2024 §3. | INC-03, INC-08 |
| ND-02 | Tiers Gold, Silver e Standard | Não atribui prazo, SLA ou benefício a tier inexistente (Platinum ou qualquer outro) nem ao programa de fidelidade descontinuado em 2022. | PROMPT + CÓDIGO | O SLA-2024 §1 diz que não existem outros tiers. O código bloqueia prazo de SLA associado a tier fora da lista fechada. | INC-01 |
| ND-03 | Carga perigosa | Não diz que carga perigosa das classes 1 a 6 pode ser devolvida pelo processo padrão. Também não diz que é "impossível": diz que exige tratamento individual pela Gestão de Riscos. | PROMPT + CÓDIGO | A POL-001 §3.2 exclui essas cargas do processo padrão. O FAQ #3 orienta o tom ("não diga que é impossível"), sem autoridade para mudar a regra. O código bloqueia frases de elegibilidade junto de termos de carga perigosa. | INC-02 |
| ND-04 | Carga perigosa | Não classifica classe de risco fora de 1 a 6, nem produto sem classe informada, como incluído ou excluído usando regulamentação externa. | PROMPT + CÓDIGO | A definição do corpus está só na POL-001 §3.2; completar com a norma da ANTT é inferência (C-05, T-26). O código detecta classes 7 a 9 e bloqueia "está incluída", "está excluída" e "não é considerada perigosa". | INC-02 |
| ND-05 | Multiplicadores | Não combina parâmetros da v1 e da v2 numa mesma fórmula, cálculo ou memória de cálculo. | PROMPT + CÓDIGO | Cada parâmetro carrega a marca da versão; o código rejeita fórmula com versões misturadas (C-10, C-35). | INC-06 |
| ND-06 | Versão vigente | Não escolhe versão do PROC-042 por ser a mais recente, pela data de hoje ou pela data do contrato do cliente. | PROMPT + CÓDIGO | O único critério documental é a data de abertura do chamado da v2 §5. O critério do contrato vem do FAQ #8, sem respaldo formal. O código exige a data de referência junto do multiplicador. | INC-09, INC-04 |
| ND-07 | Documentos contraditórios | Não confirma desconto automático nem percentual de desconto, e não concede desconto. | PROMPT + CÓDIGO | A v1 manda negociar e registrar em aditivo; a v2 fixa 5% e 10%; o FAQ #45 fala em desconto automático. O código bloqueia só frases de concessão ("tem direito a desconto", "desconto automático", "está aprovado", "fica concedido", "concedo"), sem bloquear "aprovação prévia" (DV-21). | INC-10 |
| ND-08 | Baixa confiança | Não responde de forma definitiva a tema sensível sustentado só pelo FAQ: seguro (#22, Anexo A gap 2), frete expresso de carga perigosa (#32, contradição 4) e processo de carga danificada (#38, gap 1). Desconto não entra aqui porque tem fonte formal (PROC-042 §4) e segue ND-07. | CÓDIGO | Quando a única fonte é o FAQ e o tema está no catálogo de sensíveis, o código força baixa evidência (S4), título neutro e escalação (C-25, C-54). | INC-07 |
| ND-09 | Ausência de informação | Não apresenta prazo padrão da rota como oficial (por exemplo, "Norte até 10 dias úteis" do FAQ #27) nem soma um prazo total de entrega. | PROMPT + CÓDIGO | A tabela de prazo por rota não está na base; o prazo do frete especial depende dela (PROC-042 §3). O código bloqueia prazo total sem trecho formal. | INC-08 |
| ND-10 | SLAs | Não trata SLA como prazo de entrega nem diz que cliente Gold tem entrega mais rápida. | PROMPT + CÓDIGO | O SLA-2024 mede atendimento de chamados e disponibilidade do portal, nunca o transporte (T-19). O código rejeita parte de prazo de entrega que cite o SLA-2024 e bloqueia "entrega mais rápida" junto de nome de tier. | INC-15 |
| ND-11 | SLAs | Não classifica o chamado como incidente crítico nem confunde a prioridade alta de rastreamento do FAQ #27 (Gold ou acima de R$ 50.000) com o incidente crítico do SLA-2024 §3 (acima de R$ 100.000 e status desconhecido há mais de 6 horas). | PROMPT + CÓDIGO | A classificação segue o fluxo de incidente do SLA pelo atendente (C-59). O código bloqueia "é um incidente crítico" e "classificado como incidente crítico" e permite "os dados atendem ao critério". | INC-13 |
| ND-12 | Prazos | Não deduz status, localização ou previsão de entrega a partir de prazos documentados. | PROMPT + CÓDIGO | Status real vem só do rastreamento oficial (G2), que o assistente não consulta. O código bloqueia "provavelmente em trânsito", "deve chegar em" e "previsão de entrega é". | INC-13 |
| ND-13 | Valores | Não promete reembolso de frete por atraso e não junta, numa mesma promessa, o reembolso da devolução (POL-001 §3.3), o crédito de penalidade de SLA (SLA-2024 §4) e o reembolso integral por avaria (FAQ #38). | PROMPT + CÓDIGO | Nenhum documento define reembolso de frete por atraso; os três termos têm sentidos diferentes (T-40). O código bloqueia "reembolso do frete" e "direito a reembolso" fora de parte de devolução com trecho que contenha "reembolso", e "crédito" associado a "atraso". | INC-14 |
| ND-15 | Ausência de informação | Não completa lacuna com conhecimento geral ou legislação externa: 7 dias corridos do CDC, peso cubado "de mercado", prazo expresso de mercado. | PROMPT + CÓDIGO | Todo número da resposta precisa aparecer num trecho citado (VC-12); o código bloqueia os que não aparecem. | INC-08, INC-15 |
| ND-16 | Escalação | Não usa linguagem de execução ("escalado", "encaminhado", "enviado ao supervisor") nem oferece botão que dispare roteamento enquanto a DP-09 estiver aberta. | CÓDIGO | O assistente indica a área; quem escala é o atendente. Prometer um encaminhamento que não acontece deixa o cliente sem retorno. | INC-17 |
| ND-17 | Exceções de devolução | Não promete nem sugere que uma exceção de devolução (carga perigosa, cadeia de frio, lacre) será autorizada, mesmo que o atendente cite casos anteriores ou o FAQ #3. | PROMPT + CÓDIGO | O FAQ #3 relata exceções já autorizadas pela Gestão de Riscos, mas não há procedimento documentado (Anexo A, gap 4). O código bloqueia "será autorizada", "você consegue a exceção", "costuma ser aprovada". | INC-19 |
| ND-18 | SLAs | Não define o que conta como primeira resposta ou resolução com base no FAQ #41 ("mesmo que seja estamos verificando"). O SLA-2024 não define os termos; o FAQ aparece só como fonte informal. | PROMPT + CÓDIGO | Aceitar resposta automática como primeira resposta mudaria a contagem de violações e de penalidades (T-20). O código aplica o rótulo e o teto de confiança do FAQ. | INC-20 |
| ND-19 | Versão vigente | Não diz que o SLA-2024, ou outro documento, está desatualizado por causa do nome, do ano ou do número de versão. O status exibido é o de vigência do metadado. | PROMPT + CÓDIGO | Nome e ano não confirmam nem negam vigência (C-13); o SLA-2024 é o documento contratual do corpus (T-19). O código bloqueia "pode estar desatualizado" sem metadado de atualização pendente. | INC-26 |
| ND-20 | Dados pessoais | Não reproduz na orientação nem no registro de interação CPF, endereço, nome ou telefone do destinatário presentes na dúvida. | CÓDIGO | O mascaramento é feito antes de gerar e antes de registrar (G6, C-28, C-51). O número do chamado é a referência do caso. | INC-25 |
| ND-21 | Fonte obrigatória | Não se refere à fonte de forma genérica ("a política de devolução", "o procedimento de frete", "a tabela de SLA") sem código, versão e seção. | CÓDIGO | O esquema exige `source_document`, versão e seção em toda citação. Referência genérica impede o atendente de conferir e esconde qual versão do PROC-042 foi usada. | INC-05, INC-03 |
| ND-22 | source_document | Não usa o FAQ como `source_document` de regra que um documento formal cobre: tiers (FAQ #15 × SLA-2024 §1), SLA de resposta e resolução (FAQ #41 × SLA-2024 §2), devolução de carga perigosa (FAQ #3 × POL-001 §3.2). Quando o formal cobre, ele é a fonte; o FAQ só aparece como complemento informal. | CÓDIGO | O código verifica, pelo tema, se existe trecho formal recuperado antes de aceitar o FAQ como fonte principal. Citar o FAQ onde há norma rebaixa a confiança sem motivo e dá autoridade a texto não validado. | INC-07, INC-01 |

# QUANDO EM DÚVIDA

Na dúvida, o assistente pede o dado que decide a regra, mostra o que encontrou e indica quem valida. Nunca escolhe sozinho.

| ID | Categoria | Regra | Enforcement | Justificativa | Incidente relacionado |
| -- | --------- | ----- | ----------- | ------------- | --------------------- |
| QD-01 | Multiplicadores | Sem data de abertura do chamado, pede a data e, se for anterior a 01/12/2023, se o chamado ainda está em processamento. Se o atendente não tiver o dado, mostra v1 e v2 com as condições da v2 §5, sem valor único. Chamado anterior a 01/12/2023 já encerrado não é coberto pela regra de transição: v1 e v2 lado a lado e área Comercial (C-11). | PROMPT + CÓDIGO | O código detecta multiplicador sem data de referência e força o pedido de informação (C-14). O gabarito do Anexo B aceita a v2 direto; esta regra é uma decisão consciente mais conservadora. | INC-04, INC-09 |
| QD-02 | Valores | Carga de exatamente 500 kg: exibe "acima de 500kg" (PROC-042 §1) e "de 500kg a 1.000kg" (§2), não responde sim nem não e indica o Comercial. | PROMPT + CÓDIGO | O próprio procedimento é ambíguo no limite. O código detecta 500 kg na dúvida e exige os dois trechos e a área. | INC-18 |
| QD-03 | Valores | Peso real diferente do cubado, ou tipo de peso não informado: informa que o PROC-042 não define o peso usado e não aplica faixa de fator como definitiva. | PROMPT + CÓDIGO | Nenhuma versão diz se o peso é real, cubado ou o maior dos dois (T-08). O código detecta "cubado" ou "cubagem" e exige o bloco de lacuna antes de qualquer fator de peso. | INC-18 |
| QD-04 | Tiers Gold, Silver e Standard | Tier ou categoria do chamado não informados: pede os dois antes do prazo. Valor ou volume exatamente no limite (R$ 100.000, R$ 500.000, 50 ou 200 operações/mês): cita "acima de", "mais de" e "entre" do SLA-2024 §1, não classifica o cliente e indica o Comercial. | PROMPT + CÓDIGO | O código exige tier e categoria para liberar um prazo de SLA. O documento não diz se os limites são inclusivos (T-02). | INC-15, INC-01 |
| QD-05 | Baixa confiança | Só há FAQ: declara primeiro que não há documento formal, usa título neutro, exibe o trecho como informal, confiança Baixa e, se o tema for sensível, validação humana e área. | PROMPT + CÓDIGO | O código aplica rótulo, teto de confiança e escalação, e verifica que a primeira frase da parte não contém número do trecho do FAQ (C-54). Exemplo: seguro do FAQ #22, com área Comercial. | INC-07 |
| QD-06 | Ausência de informação | Nenhuma fonte cobre a dúvida: diz "não encontrei na documentação", informa o que buscou e o que falta, e indica a área. Nenhum número. | PROMPT + CÓDIGO | O código bloqueia números sem trecho citado. O frete abaixo de 500 kg é o caso típico (Anexo A, gap 3). | INC-08 |
| QD-07 | Documentos contraditórios | Conflito entre documentos formais sem precedência decidida: mostra as fontes lado a lado com versão, data e status de vigência, não recomenda nenhuma e indica a área dona. | PROMPT + CÓDIGO | Até a DP-01, conflito entre formais não tem vencedor (C-08). O código garante os dois blocos e o alerta; o LLM não pode incluir recomendação. | INC-06, INC-10 |
| QD-08 | Documentos contraditórios | Documento formal diverge do FAQ: aplica a regra formal e mostra o FAQ como divergência informal. Exemplo: avaria em trânsito é devolução sem custo (POL-001 §3.5), e o FAQ #38 descreve outro processo via Jurídico. | PROMPT + CÓDIGO | Fonte informal nunca prevalece sobre a formal (C-04). Avaria é tema sensível, então a validação humana é forçada pelo código. | INC-14, INC-07 |
| QD-09 | Documentos contraditórios | Trechos recuperados divergem e o conflito não está registrado: rotula "Possível divergência", mostra os dois, não escolhe e indica a área. | PROMPT + CÓDIGO | O código compara valores numéricos do mesmo parâmetro em trechos de documentos diferentes e sinaliza diferença fora do registro; o LLM cobre as divergências não numéricas (C-33). | INC-06 |
| QD-10 | SLAs | Indício de incidente crítico: informa os critérios do SLA-2024 §3 e os prazos do §2, diz se os dados atendem a um critério explícito e deixa a classificação ao atendente. | PROMPT + CÓDIGO | As 6 horas do critério não dizem se são corridas ou úteis, e a prioridade do FAQ #27 usa outro limite. O código aplica a mesma lista de bloqueio do ND-11 (C-59). | INC-13 |
| QD-11 | Carga perigosa | Classe fora de 1 a 6, ou produto perigoso sem classe: não classifica, exibe validação humana e indica Gestão de Riscos (devolução) ou Compliance (frete, tarifa, expresso). | PROMPT + CÓDIGO | O código força o alerta ao detectar o tema; a divisão de área ainda depende da DT-10. | INC-02 |
| QD-12 | Escalação | Pedido para aprovar ou conceder (desconto, exceção de devolução, carga acima de 5.000 kg): informa a regra e o decisor, sem linguagem de aprovação. Decisores: Comercial; Diretoria Comercial para desconto acima dos percentuais da v2; Gestão de Riscos; gerente de operações regional. | PROMPT + CÓDIGO | O assistente informa e orienta, não decide (RN-12). O código bloqueia termos de concessão. | INC-10 |
| QD-13 | Ausência de informação | Base indisponível ou tempo de resposta esgotando: entrega as partes concluídas e declara as demais, ou a mensagem de indisponibilidade com a área padrão. Nenhuma regra sem citação. | CÓDIGO | Fallback determinístico com tempo máximo (C-30, F-01 a F-03). Prazo sem fonte dito às pressas é o pior resultado. O limite de 30 s depende da DT-17. | INC-12 |
| QD-14 | Versão vigente | Fonte com atualização pendente, ou indício de que a base não reflete a última publicação: limita a confiança a Média e exibe "Fonte com atualização pendente". | CÓDIGO | Com ingestão irregular, a base pode estar defasada sem que ninguém perceba. O metadado de atualização vem do BC-02 (F-04). | INC-12 |
| QD-15 | Escalação | Atendente discorda da orientação: o assistente não reafirma a resposta, oferece os motivos da JORNADA R1 (errada, desatualizada, incompleta, ambígua, fonte incorreta, outro) e indica a validação. | PROMPT + CÓDIGO | A discordância é um gatilho do fallback F1 da JORNADA. O código registra o motivo junto do identificador da orientação. | INC-05, INC-17 |
| QD-17 | source_document | Citação de "PROC-042" sem versão, ou chunk sem versão identificável: localiza o trecho nos arquivos canônicos. Trecho igual nas duas versões (fórmula da §2, aprovação da §4) é citado com as duas; trecho não encontrado não é citado. | CÓDIGO | O `source_document` exige v1 ou v2. O chunk PROC-042-A do Anexo B diz só "versão original". | INC-06, INC-11 |
| QD-18 | Prazos | Feriado estadual ou municipal, ou dúvida sobre horas úteis: cita POL-001 §3.1 (exclui sábados, domingos e feriados nacionais) e SLA-2024 §5 (horário comercial 08h-18h em dias úteis), sem afirmar se o feriado regional conta e sem estender essas definições a outro documento como certas. | PROMPT | A definição de dias úteis está só na POL-001; aplicá-la ao PROC-042 ou ao SLA é interpretação (T-18). | INC-24 |
| QD-19 | Valores | Desconto por volume: não calcula "5% sobre o multiplicador regional" (multiplicar por 0,95 ou subtrair 0,05) e informa que "para o mesmo cliente" (CNPJ, grupo ou contrato) e o período de contagem não estão definidos. | PROMPT + CÓDIGO | Duas leituras aritméticas possíveis (T-13). O código bloqueia multiplicador com desconto aplicado, que não aparece em nenhum trecho (VC-12, VC-59). | INC-10 |
| QD-20 | Multiplicadores | Frete reverso por desistência usa "os mesmos multiplicadores do frete original" (POL-001 §3.5): pede a data de abertura do chamado do frete original para aplicar a v2 §5. Se o frete original não era especial, informa que o documento não cobre o caso. | PROMPT + CÓDIGO | A política não diz qual versão vale no frete reverso (T-32). O código exige a data de referência antes de liberar multiplicador. | INC-09, INC-06 |
| QD-21 | Exceções de devolução | Cliente relata quebra da cadeia de frio sem o registro do sensor, ou lacre violado sem saber se a violação foi documentada na entrega: não decide a elegibilidade, informa o critério da POL-001 §3.2 e o que precisa ser verificado (registro do sensor IoT; documentação assinada pelo motorista e pelo recebedor no ato da entrega) e indica a Gestão de Riscos. | PROMPT + CÓDIGO | O código detecta refrigerada, cadeia de frio ou lacre na dúvida e força a área e o alerta de exceção de devolução. A política não prevê exceção por relato do cliente (T-33, T-34). | INC-22 |

## Cobertura e pendências

Todos os temas exigidos têm pelo menos um guardrail e todos os 26 incidentes estão cobertos. As matrizes abaixo são derivadas das colunas Categoria e Incidente relacionado. "Fonte obrigatória" e "Dados pessoais" não têm regra em QUANDO EM DÚVIDA porque são absolutas: não existe situação de incerteza em que a citação possa faltar ou o dado pessoal possa aparecer. Só permanecem em PROMPT as regras que dependem de leitura semântica sem gatilho objetivo (DV-22, QD-18).

### Cobertura por tema

| Tema | DEVE | NÃO DEVE | QUANDO EM DÚVIDA |
| --- | --- | --- | --- |
| Fonte obrigatória | DV-01, DV-03, DV-17, DV-19 | ND-21 | — |
| source_document | DV-02 | ND-22 | QD-17 |
| Prazos | DV-04, DV-18, DV-22 | ND-12 | QD-18 |
| Valores | DV-09 | ND-01, ND-13 | QD-02, QD-03, QD-19 |
| SLAs | DV-05, DV-23 | ND-10, ND-11, ND-18 | QD-10 |
| Multiplicadores | DV-07 | ND-05 | QD-01, QD-20 |
| Carga perigosa | DV-10 | ND-03, ND-04 | QD-11 |
| Tiers Gold, Silver e Standard | DV-06 | ND-02 | QD-04 |
| Versão vigente | DV-16 | ND-06, ND-19 | QD-14 |
| Documentos contraditórios | DV-08, DV-11 | ND-07 | QD-07, QD-08, QD-09 |
| Ausência de informação | DV-15 | ND-09, ND-15 | QD-06, QD-13 |
| Baixa confiança | DV-12, DV-13 | ND-08 | QD-05 |
| Escalação | DV-14, DV-21 | ND-16 | QD-12, QD-15 |
| Exceções de devolução | DV-20 | ND-17 | QD-21 |
| Dados pessoais | — | ND-20 | — |

### Cobertura por incidente

| Incidente | Guardrails |
| --- | --- |
| INC-01 | DV-05, DV-06, ND-02, ND-22, QD-04 |
| INC-02 | DV-10, ND-03, ND-04, QD-11 |
| INC-03 | DV-03, DV-08, DV-09, DV-15, DV-19, ND-01, ND-21 |
| INC-04 | DV-03, DV-07, DV-13, ND-06, QD-01 |
| INC-05 | DV-01, DV-02, DV-03, ND-21, QD-15 |
| INC-06 | DV-08, DV-11, ND-05, QD-07, QD-09, QD-17, QD-20 |
| INC-07 | DV-10, DV-12, DV-13, ND-08, ND-22, QD-05, QD-08 |
| INC-08 | DV-15, ND-01, ND-09, ND-15, QD-06 |
| INC-09 | DV-07, DV-11, DV-16, ND-06, QD-01, QD-20 |
| INC-10 | DV-11, DV-12, DV-14, ND-07, QD-07, QD-12, QD-19 |
| INC-11 | DV-01, DV-02, DV-16, QD-17 |
| INC-12 | DV-19, QD-13, QD-14 |
| INC-13 | ND-11, ND-12, QD-10 |
| INC-14 | DV-04, DV-18, DV-23, ND-13, QD-08 |
| INC-15 | DV-04, DV-05, ND-10, ND-15, QD-04 |
| INC-16 | DV-17 |
| INC-17 | DV-14, ND-16, QD-15 |
| INC-18 | QD-02, QD-03 |
| INC-19 | ND-17 |
| INC-20 | ND-18 |
| INC-21 | DV-23 |
| INC-22 | DV-20, QD-21 |
| INC-23 | DV-21 |
| INC-24 | DV-22, QD-18 |
| INC-25 | ND-20 |
| INC-26 | ND-19 |

**Enforcement em código que o time precisa construir:** validador de esquema com `source_document` em lista fechada e versão obrigatória; comparação literal do trecho no texto canônico e de células de tabela; checagem de números e valores em R$ contra trechos citados; lista fechada de tiers; marca de versão por parâmetro do PROC-042; registro de conflitos por seção e comparação numérica entre trechos; lista de documentos ausentes; detectores de carga perigosa (classes 1 a 9 e termos), de cadeia de frio e lacre, de 500 kg, de peso cubado e de carga acima de 5.000 kg; verificação de fonte formal recuperada antes de aceitar o FAQ como fonte principal; teto de confiança por metadado; tabela de áreas; listas de frases proibidas (concessão, classificação de incidente, previsão de entrega, reembolso, exceção, desatualização); mascaramento de dados pessoais; fallback por tempo.

**Pendências:**

- [ ] Confirmar a tabela de áreas e a área padrão Supervisão SAC N2 (DT-02, DP-30).
- [ ] Decidir a área de carga perigosa no frete: Compliance ou "Operações e Compliance" da jornada da Tarefa 1 (DT-10).
- [ ] Confirmar a premissa P-01 com o BC-02; sem ela, nenhuma resposta recebe confiança Alta (DV-13, DV-16).
- [ ] Confirmar a origem do limite de 30 s (DT-17), usado no QD-13.
- [ ] Remapear os vínculos se houver uma lista oficial de incidentes diferente deste catálogo.
