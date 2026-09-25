# 07-historico-iteracao — Iteração v1 → final

| Item | Conteúdo |
| --- | --- |
| Artefatos de entrada | `01-bounded-contexts-v1.md` (rev. 1.1), `02-requirements-v1.md`, `03-mockup-teams.png` |
| Revisão aplicada | `04-tech-lead-review.md` |
| Artefatos de saída | `05-bounded-contexts-final.md` (mapa de BCs + linguagem ubíqua), `06-requirements-final.md` |
| Data | 24/09/2026 |

## Reconciliação com o 01 original

A revisão do Tech Lead e a primeira versão dos artefatos finais foram feitas sem o `01-bounded-contexts-v1.md`. Com o arquivo disponível, a iteração foi refeita:

- **O `05` foi regenerado a partir do 01**, preservando todo o conteúdo e os IDs `T-01` a `T-40`. A versão anterior do `05`, reconstruída pelas referências do requirements, tinha IDs com sentido diferente do original e perdia boa parte do conteúdo (linhas Rec-01 e Rec-03).
- **Parte dos achados da revisão já estava resolvida no 01.** A causa era a redação do `02-requirements-v1.md`, não o recorte de domínio. Nesses casos, a alteração foi feita só no requirements, e a linha indica "Já resolvido no 01".
- **Algumas decisões da iteração anterior foram revertidas** por contrariarem o 01: a separação de "carga perigosa" em dois termos, a remoção dos valores do glossário, a área única de carga perigosa, o rótulo "Resposta completa" e a atribuição da captura de feedback ao BC-04.
- **Uma linha da iteração anterior estava errada** e foi retirada: não havia colisão no `T-16` (Rec-02).

**Resumo:** 83 linhas: 61 itens da revisão, 4 de reconciliação com o 01 e 18 da verificação contra o Anexo A e os entregáveis da fase 1. Um item da revisão (MK-14) foi revertido por ser improcedente. Pendências abertas: DT-01 a DT-11, DT-13 a DT-17 e as DPs listadas no `06` §4.3.

## Verificação contra o Anexo A e os entregáveis da fase 1

Depois da reconciliação, foram recebidos o Anexo A, os cinco documentos individuais, o Anexo B, o `exercicio-fase-1-entendimento.md`, a JORNADA em PDF (Tarefa 2), a jornada textual (Tarefa 1), a `Reflexao_Progressive_Disclosure` e os demais entregáveis da fase 1. Todas as citações e valores dos VCs foram conferidos com o texto dos documentos e estão corretos. Os achados que exigiram alteração estão nas linhas V-01 a V-18. Continuam indisponíveis a ESPEC-V2 e o `avaliacao-tech-lead.md` (DT-17 e DT-09).

**Como ler:** a coluna "Alteração realizada" começa com **Tratado** quando o problema foi resolvido nos artefatos, com **Já resolvido no 01** quando o recorte original já tratava o ponto, ou com **Pendente** quando foi registrado como decisão do time com comportamento provisório. O Apêndice traz o mapa dos IDs da versão anterior do `05` para os IDs finais.

## Histórico de Iteração

| Problema identificado | Alteração realizada | Artefato afetado |
| --------------------- | ------------------- | ---------------- |
| [BC-01] Mapa de BCs não entregue; requirements v1 citava rev. 1.1 no cabeçalho e rev. 1 na §4.2 | **Tratado:** 01 recebido e `05` regenerado a partir dele. A menção à rev. 1 era legítima: é a revisão em que a PROC-042 v2 foi incorporada; o texto passa a dizer isso. Pendência de reconciliação (DT-12) encerrada. | 05 (cabeçalho, registro de revisão); 06 §4.2, §4.4 |
| [BC-02] Nenhum BC dono do mapeamento tema → área | **Tratado:** o 01 já listava no BC-03 os destinos que o Anexo A nomeia. O `05` §2.4 completa a tabela com esses destinos, as áreas dos donos dos documentos e a área padrão; C-46 exige uma área por parte. **Pendente (DT-02, DP-30):** confirmação da tabela e da área padrão. | 05 BC-03, §2.3, §2.4; 06 C-46, VC-36 |
| [BC-03] Divergência detectada em runtime sem tratamento definido | **Tratado:** termo TP-32; evento BC-01 → BC-02; C-33 (S6 "Possível divergência" com pendência ao BC-02); VC-43 com massa do Anexo B. | 05 BC-01, BC-02, §2.3, TP-32; 06 §2.2, C-33, F-13, VC-43 |
| [BC-04] Registro de interação em BC-03 e BC-04 | **Tratado:** no 01, o "registro do caso" ficava no BC-03 e o feedback registrado no BC-04 (`JORNADA` R2). Passa a existir um registro único produzido pelo BC-01 (TP-36), referenciado pelos dois contexts pelo identificador (TP-37). | 05 BC-01, BC-03, BC-04, §2.3; 06 §2.2, C-50, VC-37 |
| [BC-05] Tier com dono duplo (BC-08 e BC-09) | **Já resolvido no 01:** o BC-09 é dono do tier e o BC-08 é Conformist ("Fora: critério e atribuição de tier"). A sobreposição vinha da redação do requirements v1 §2.3. O `05` §2.3 confirma e o `06` §2.3 foi alinhado. | 05 §2.3; 06 §2.3 |
| [BC-06] Desconto por volume em BC-05 e BC-09 | **Já resolvido no 01:** o BC-05 lista desconto como "Fora (BC-09)". O `05` §2.3 confirma e a relação BC-09 → BC-05 passa a citar o desconto, que na v2 incide sobre o multiplicador. | 05 BC-05, §2.3; 06 §2.3 |
| [BC-07] Carga perigosa em três BCs com áreas diferentes | **Tratado:** o BC-07 já era upstream no 01 e passa a ser dono do alerta e das linhas de área. A área se divide conforme o 01: Gestão de Riscos para exceção de devolução (`POL-001 §3.2`) e Compliance para frete, tarifa e expresso (`PROC-042 v2 §4`, `FAQ #32`). A área única (Gestão de Riscos) da iteração anterior foi revertida. **Pendente (DT-10).** | 05 BC-07, §2.3, §2.4; 06 C-46, VC-26 a VC-30, DT-10 |
| [BC-08] Catálogo de temas sensíveis sem dono | **Tratado:** o 01 listava os temas dentro do BC-03; o `05` §2.5 torna o BC-03 dono explícito do catálogo. Níveis continuam em DP-08. | 05 BC-03, §2.5; 06 C-26, D-04 |
| [BC-09] Fronteira avaria × sinistro indefinida | **Tratado com conflito mantido:** o 01 registra a fronteira BC-10/BC-11 como conflito (T-37, pendência 4). Para a orientação, a regra formal da `POL-001 §3.5` prevalece sobre o `FAQ #38`, com o conflito exposto; áreas nas linhas 10 e 11 da tabela. A pendência 4 continua aberta. | 05 §2.3, §2.4, T-37; 06 VC-15 |
| [LU-01] Ambiguidade de 500 kg resolvida por definição | **Já resolvido no 01:** o T-05 já registrava a ambiguidade. A causa estava na C-06 do requirements v1, que omitia a ressalva; a C-06 foi corrigida e o T-05 ganhou nota de comportamento (exibir os dois trechos e indicar o Comercial). | 05 T-05; 06 C-06, VC-07 |
| [LU-02] T-26 definido pelo escopo da exceção de devolução | **Tratado:** o 01 define T-26 como classes 1 a 6 da `POL-001 §3.2` de propósito e já registra a lacuna. A definição foi mantida, com a regra de não classificar outras classes e exibir validação humana (coerente com VC-28 e VC-30). A separação em dois termos da iteração anterior foi desfeita. | 05 T-26, BC-07; 06 C-06, VC-28 |
| [LU-03] Glossário embutindo valores de regra | **Tratado com abordagem revista:** os valores do 01 foram mantidos, sempre com seção de origem, e foi criada a regra de uso: o glossário não é evidência, o Anexo A prevalece e o termo deve ser revisto quando o documento mudar. A remoção de valores da iteração anterior foi desfeita. | 05 §3 (regra de uso) |
| [LU-04] "Chamado" com três sentidos; data de referência do PROC-042 ambígua | **Tratado:** novo termo T-41 Chamado (polissemia), no mesmo formato do T-40; TP-04 Data de referência mantido do 01. **Pendente (DT-11, DP-12):** qual chamado fornece a data. | 05 T-41, TP-04; 06 C-11, C-14, DP-12, DT-11 |
| [LU-05] S1–S8 sem definição; estados sem situação | **Tratado:** TP-17 com S0 a S9, mantendo os nomes do 01 (completa, parcial, sem evidência, baixa evidência, conflito); tabela completa no `06` §3.9. | 05 BC-01, TP-17, §3.4; 06 §3.9, C-40, C-45, C-49 |
| [LU-06] Termos sem definição | **Tratado:** o 01 já definia interpretação permitida (TP-08) e inferência (TP-09); ambos ganharam exemplos. Novos: afirmação explícita, regra prevalente, bloco independente e dependência bloqueante (TP-19 a TP-22). "Orientação utilizável" removida do O-01. | 05 §3.3; 06 O-01, C-16, C-23 |
| [LU-07] Vocabulário divergente entre artefatos | **Tratado:** "baixa evidência" (situação, TP-07) e "baixa confiança" (nível, TP-18) são termos distintos do 01, agora diferenciados; tabela termo → rótulo de UI (`05` §3.4) exigida pela C-55. | 05 TP-07, TP-18, §3.4; 06 C-55, VC-49 |
| [LU-08] "Data de atualização da fonte" indefinida | **Tratado:** TP-27 = data do cabeçalho do documento (coluna "Versão / data" da convenção de fontes do 01), nunca a data de ingestão; C-56 padroniza a exibição. | 05 TP-27; 06 C-20, C-56 |
| [LU-09] Granularidade de "fonte com conflito registrado" | **Tratado:** registro de conflitos por seção (`05` §2.6), com as seções do 01 (§2.1 multiplicadores, §2 fator de peso, §3 prazo adicional, §4 desconto, `POL-001 §3.5`); C-39; VC-01. A fórmula do `PROC-042 §2`, idêntica nas duas versões, não recebe alerta. | 05 §2.6; 06 C-09, C-24, C-39, VC-01, VC-09 |
| [RQ-01] O-02 × C-24 (nível global mínimo) | **Tratado:** nível por parte; orientação de duas partes não exibe nível global; alertas rotulados por parte. | 06 O-02, O-06, C-24, C-53, VC-08 |
| [RQ-02] Agregação de "Não se aplica" indefinida | **Tratado:** nível por bloco; nível da parte = menor entre os blocos com nível aplicável. | 06 C-23, C-24 |
| [RQ-03] Sem situação por parte nem precedência | **Tratado:** situação por parte e por bloco; ordem S0 > S8 > S6 > S7 > S3 > S4 > S5 > S2 > S1. | 06 C-08, §3.9, C-40, VC-09, VC-14 |
| [RQ-04] Três ou mais partes, parte não classificada | **Tratado:** limite de 2 partes com excedentes declarados; parte não classificada; VC-44 e VC-45. **Pendente (DT-13):** confirmar o limite. | 06 C-41, C-42, F-08, VC-44, VC-45 |
| [RQ-05] "Fora das categorias" × "fora do domínio" | **Tratado:** domínio = contexts BC-05 a BC-11 (TP-11); categorias são agrupamento (TP-12). | 05 TP-11, TP-12; 06 §1.1, C-45 |
| [RQ-06] S7 sem segundo turno; sem "não tenho o dado" | **Tratado:** janela de 30 minutos com referência ao identificador anterior; opção "Não tenho essa informação"; VC-41, VC-50. **Pendente (DT-01, DP-29):** confirmar a janela. | 06 C-14, C-43, C-44, C-48, VC-19, VC-33, VC-41, VC-50 |
| [RQ-07] Caminho condicional da C-14 sem VC | **Tratado:** C-14 especifica blocos separados com condição e citação, sem valor único; VC-41. | 06 C-14, VC-41 |
| [RQ-08] C-11 sem chamado anterior e encerrado | **Tratado:** caso encerrado → S6 com Comercial; S7 pede também o status; VC-42. | 06 C-11, C-14, VC-42 |
| [RQ-09] Versões misturadas na mesma fórmula (VC-09) | **Tratado:** fórmula sem valores substituídos e cada parâmetro em bloco próprio com versão. | 06 C-35, VC-09, VC-20 |
| [RQ-10] VC-09 dependente de "hoje" | **Tratado:** data fixada em 24/09/2026; regra geral de datas fixas. | 06 §5, VC-09 |
| [RQ-11] VC-12 com falsos positivos | **Tratado:** restrito a números de regra no texto das partes, com lista de exceções. | 06 VC-12 |
| [RQ-12] Citação literal inviável para tabelas | **Tratado:** normalização e citação por referência de célula; ingestão que preserve tabelas (D-06). | 06 C-38, D-06, VC-02, VC-11, VC-39 |
| [RQ-13] Comparação literal × semântica indefinida | **Tratado:** "informa X" = valores exatos e elementos obrigatórios; VC-01 reescrito por elementos. | 06 §5, VC-01 |
| [RQ-14] "Sempre escalado" sugeria execução | **Tratado:** C-25 fala em indicação; C-47 proíbe linguagem de execução, coerente com TP-10 do 01. | 06 C-25, C-47, VC-55 |
| [RQ-15] DPs citadas e não listadas | **Tratado:** DP-10, DP-17 e DP-30 listadas com provisório; prefixos SEG e RNF no cabeçalho; decisões novas com prefixo DT. A DP-03 e a DP-27 constam como respondidas pelo 01 (§4.2), não como pendentes. | 06 cabeçalho, C-01, §4.2, §4.3, §4.4 |
| [RQ-16] VC-38 com "ou"; VC-26 com área incoerente | **Tratado:** área única pela tabela. VC-38 → Comercial (a Diretoria Comercial fica para pedidos acima dos percentuais da v2, linha 4); VC-26 e VC-29 → Compliance; VC-27 e VC-28 → Gestão de Riscos. **Pendente (DT-10).** | 05 §2.4; 06 C-46, VC-26, VC-29, VC-36, VC-38 |
| [RQ-17] Temas sensíveis sem asserção nos VCs | **Tratado:** alerta e área incluídos em VC-14, VC-31, VC-32 e VC-38. | 06 VC-14, VC-31, VC-32, VC-38 |
| [RQ-18] S3 × S4 quando o FAQ cobre documento ausente | **Tratado:** S4 com ausência declarada primeiro; S3 sem conteúdo no FAQ. | 06 §2.4, C-36, C-54, VC-22 |
| [RQ-19] Metadado ausente sem fallback | **Tratado:** sentinelas (TP-28); fonte formal com metadado ausente limitada a Média; VC-48. | 05 TP-28; 06 C-37, F-05, VC-48 |
| [RQ-20] Fallbacks indefinidos | **Tratado:** tabela F-01 a F-14; VC-35 e VC-51 a VC-54. | 06 C-30, §3.13, VC-35, VC-51 a VC-54 |
| [RQ-21] Pontos de medição de tempo diferentes; 100% sem carga | **Tratado:** ponto de medição único; VC-34 com 3 execuções, sequencial e concorrente. **Pendente (DT-05, DP-24):** concorrência e percentil de produção. | 06 O-01, C-29, D-01, VC-34 |
| [RQ-22] CPF gravado integralmente no log | **Tratado:** mascaramento antes de gravar; VC-40 verifica o registro. **Pendente (DT-06, DP-17):** retenção e acesso. | 06 C-50, C-51, VC-40 |
| [RQ-23] VC-40 só com CPF | **Tratado:** CPF, endereço, nome e telefone do destinatário, na orientação e no registro (lista do BC-01 no 01). | 06 C-28, VC-40 |
| [RQ-24] C-31 e C-32 sem VC; matrizes incorretas | **Tratado:** VC-46 e VC-47; §6.1 e §6.3 refeitas. | 06 VC-46, VC-47, §6 |
| [RQ-25] ADR-0002 indisponível | **Pendente (DT-09):** confirmar valores e medir tokens de VC-09 e VC-14. Provisório: C-31 com parte declarada (VC-46). | 06 C-31, D-05, §4.1, VC-46 |
| [RQ-26] P-01 sem confirmação | **Tratado:** VCs marcados [P-01] com valor alternativo. **Pendente (DT-08):** confirmação pelo BC-02. | 06 C-03, VC-01, VC-02, VC-08, VC-32 |
| [RQ-27] Contrato de saída fora do escopo | **Tratado:** contrato lógico com campos obrigatórios; serialização no `plan.md`; VC-49. | 06 §2.4, §3.12, VC-49 |
| [RQ-28] Identidade do atendente não mencionada | **Tratado:** dependência D-02; identificador do atendente no registro. | 06 D-02, C-50 |
| [RQ-29] Outcomes subjetivos | **Tratado:** O-01 e O-06 reescritos com sinais observáveis. | 06 O-01, O-06 |
| [MK-01] "Escalar ao supervisor" como ação | **Tratado:** o próprio 01 (TP-10) registra "procure seu supervisor" como interpretação incorreta de escalonamento. C-47 exige "Área indicada: <área>" sem ação de roteamento. **Pendente (DP-09, DT-15).** | 05 TP-10, TP-40, §3.4; 06 C-47, VC-55 |
| [MK-02] Área de destino só na tela 3 | **Tratado:** C-32 e C-53 exigem área visível; VC-36, VC-47. Mockup a atualizar (DT-15). | 06 C-32, C-53, VC-36, VC-47 |
| [MK-03] Título afirmando percentuais do FAQ | **Tratado:** título neutro em S3, S4 e S6 (C-54); VC-13, VC-22. Mockup a atualizar (DT-15). | 06 C-54, VC-13, VC-22 |
| [MK-04] Nível único para duas partes | **Tratado:** nível por parte (C-24); VC-08. Mockup a atualizar (DT-15). | 06 C-24, VC-08 |
| [MK-05] Estados não mockados | **Pendente (DT-15):** mockup v2; conteúdo de cada estado já especificado em §3.9, C-48 e C-49. | 06 §3.9, C-48, C-49, DT-15 |
| [MK-06] Rótulos sem mapeamento para S1–S8 | **Tratado:** tabela termo → rótulo (`05` §3.4). O rótulo "Resposta completa" do mockup foi mantido para S1, por ser o termo do 01; a troca por "Resposta fundamentada" da iteração anterior foi revertida. | 05 §3.4; 06 §3.9, C-55 |
| [MK-07] Metadados inconsistentes entre telas | **Tratado:** C-56 padroniza campos, ordem e rótulos; versão como no cabeçalho do documento. | 05 TP-27; 06 C-56 Ajustado em V-02: a data é exibida com o rótulo de cada documento. |
| [MK-08] Linha de tabela como trecho literal | **Tratado:** citação por referência de célula (C-38); VC-02. | 06 C-38, VC-02 |
| [MK-09] Análise comparativa sem citação | **Tratado com provisório:** C-34 permite "Comparação" sem números novos, recomendação ou impacto. **Pendente (DT-03).** | 06 C-21, C-34, VC-14 |
| [MK-10] Feedback fora do escopo e taxonomia incompleta | **Tratado:** o 01 atribui ao BC-01 o registro de concordância e discordância e ao BC-04 o tratamento posterior; o requirements v1 atribuía a captura ao BC-04, o que foi corrigido. O registro é uma operação do BC-01 fora do query endpoint, com os seis motivos da `JORNADA` R1 (o mockup mostrava três). **Pendente (DT-04).** | 05 BC-01, TP-38, §3.4; 06 §2.2, §2.4, C-57, DT-04 |
| [MK-11] "Acompanhar" sem fluxo; identificador indefinido | **Tratado:** texto sem promessa de acompanhamento; requisitos do identificador (C-52). **Pendente (DT-14).** | 06 C-52, C-57, VC-37, VC-55 |
| [MK-12] Truncamento ou colapso no Teams | **Tratado:** ordem de exibição e proibição de esconder alertas, nível e área; VC-47 no cliente Teams. | 06 C-53, D-01, VC-47 |
| [MK-13] Canal Teams assumido; timeout e markdown | **Tratado:** premissa P-02 e dependência D-01. **Pendente (DT-07):** ADR de canal. | 06 D-01, §4.1 |
| [MK-14] FAQ descrito como "não validado por Compliance ou Operações" | **Revertido (V-01):** o próprio cabeçalho do FAQ declara "NÃO validado por Compliance ou Operações"; o texto do mockup estava correto e o achado da revisão era improcedente. Rótulo alinhado ao cabeçalho. | 05 §3.4; 06 C-02 |
| [Rec-01] A versão anterior do `05` atribuía sentidos diferentes a IDs do 01 (T-09, T-12, T-13, T-37, T-40) e criava termos a partir de T-40, colidindo com "T-40 Reembolso e crédito" | **Tratado:** `05` regenerado a partir do 01 com os IDs originais; termos do produto com prefixo TP; referências do `06` remapeadas conforme o Apêndice. | 05 inteiro; 06 (todas as referências a termos) |
| [Rec-02] A iteração anterior registrou "T-16 com dois sentidos" como problema novo | **Tratado:** a linha estava errada. No 01, T-16 é o prazo de entrega do frete especial, que inclui o prazo adicional de +2 ou +3 dias úteis, e a C-12 do v1 (T-11, T-13, T-16) estava correta. A C-12 foi restaurada e a linha retirada. | 06 C-12 |
| [Rec-03] A reconstrução anterior do `05` omitia conteúdo do 01: 23 termos de negócio, exemplo de travessia, relações entre contexts, pendências 1 a 13 e apêndices A e B | **Tratado:** todo o conteúdo do 01 foi mantido no `05`, com as alterações da revisão aplicadas como edições pontuais. | 05 inteiro |
| [Rec-04] A versão anterior do `06` listava a DP-03 como pendente, mas o 01 a dá como respondida (a PROC-042 v2 existe); o mesmo para a DP-27 | **Tratado:** DP-03 removida da §4.3 e da origem da C-01; DP-03 e DP-27 registradas como respondidas na §4.2. | 06 C-01, §4.2, §4.3 |
| [V-01] O item MK-14 da revisão considerou incorreto o texto do mockup sobre o FAQ, mas o cabeçalho do FAQ diz "NÃO validado por Compliance ou Operações" | **Tratado:** achado revertido; rótulo da fonte informal e C-02 alinhados ao cabeçalho do FAQ. | 05 §3.4; 06 C-02 |
| [V-02] O PROC-042 declara "Data de emissão", não "Última atualização"; o "emitido em" do mockup era fiel. C-56 impunha "Atualizado em" para todos | **Tratado:** TP-27 e C-56 exibem a data com o rótulo de cada documento. | 05 TP-27; 06 C-20, C-56, VC-56 |
| [V-03] O Anexo A e os arquivos individuais têm o mesmo texto, mas o Anexo A tem negrito, itálico e ⚠️ | **Tratado:** C-38 define o arquivo individual como texto canônico e remove a marcação na normalização. | 06 C-38, VC-11 |
| [V-04] Os chunks do Anexo B são condensados (SLA-2024-B achata a tabela; SLA-2024-D omite "declarado"; PROC-042-A junta §1 e §2) | **Tratado:** C-58 proíbe citar o texto do chunk; VC-11 e VC-66 verificam. | 06 C-58, VC-11, VC-66 |
| [V-05] O VC-43 pressupunha no Anexo B uma divergência não registrada, que não existe | **Pendente (DT-16):** criar massa de teste própria para homologação; VC-43 reescrito. | 06 VC-43, DT-16 |
| [V-06] O gabarito do Anexo B espera a v2 direto para perguntas de multiplicador sem data; a C-14 exige S7 | **Tratado:** divergência registrada como decisão consciente nas regras gerais do §5; o mapa do Anexo B segue válido para retrieval. | 06 §5 (regras gerais) |
| [V-07] O guardrail (3) do exercício manda "sugerir escalar para o supervisor" e a raia 5 da JORNADA inclui "Supervisão, SAC N2" | **Tratado:** a área padrão SAC N2 passa a ter base documental; C-47 e DP-30 citam as fontes. O MK-01 continua valendo contra a ação de roteamento. | 05 §2.4; 06 C-47, DP-30 |
| [V-08] O guardrail (4) do exercício pede "português formal" | **Tratado:** C-18 exige português formal e acessível. | 06 C-18 |
| [V-09] Os requisitos simulados do Product Specialist pedem atualização em até 24h e as duas versões com indicação de data | **Tratado:** provisório de DP-05 e DP-06 = 24h (fora do escopo do endpoint); C-07 exige versão, data e status de vigência de cada fonte. | 06 C-07, §4.3 |
| [V-10] A jornada da Tarefa 1 traz uma tabela de escalonamento (devolução → Operações; sinistro → Jurídico; cargas perigosas → Operações e Compliance) | **Tratado:** `05` §2.4 revisado (linha 10 → Operações; linha 11 → Jurídico; nova linha 11a, seguro → Comercial, conforme FAQ #22 e o exemplo do F2); VC-13 → Comercial; VC-15 → Operações. **Pendente (DT-10):** carga perigosa, porque a proposta tem duas áreas. | 05 §2.4, §4.1; 06 VC-13, VC-15, DT-10 |
| [V-11] O G3 da JORNADA exige "data de vigência" visível, mas nenhum documento declara vigência | **Tratado:** novo termo TP-41 (status de vigência) com os valores possíveis; C-20 e C-56 o exigem; VC-56. | 05 TP-41; 06 C-20, C-56, VC-56 |
| [V-12] Duas numerações de guardrails (G1–G9 na Tarefa 1; G1–G6 no PDF); o G5 da Tarefa 1 contradiz a C-13 | **Tratado:** o PDF da Tarefa 2 é declarado a JORNADA de referência; a Tarefa 1 fica superada. | 05 convenção de fontes; 06 cabeçalho |
| [V-13] A Tarefa 1 diz que o assistente não classifica incidente crítico (G6) e que o atendente segue o fluxo do SLA (P1); o 06 não tratava disso | **Tratado:** nota no T-22; nova C-59; VC-57 e VC-62. | 05 T-22; 06 C-59, VC-57, VC-62 |
| [V-14] Três listas de motivos de feedback (3 no mockup, 4 na Tarefa 1, 6 na JORNADA) | **Tratado:** prevalecem os seis da JORNADA R1, já previstos no 05 (TP-38) e na C-57. | 05 TP-38; 06 C-57 |
| [V-15] A Tarefa 1 usa o número do chamado como referência na consulta e no feedback | **Tratado:** campo opcional na C-50. | 06 C-50 |
| [V-16] O 01 registra ambiguidades de negócio (T-02, T-08, T-13, T-18, T-22, T-32, T-40) que o 06 não verificava | **Tratado:** VC-58 a VC-65, redigidos com o texto dos documentos; VC-63 cobre penalidades (T-23), antes sem VC. | 06 §5.15, §6 |
| [V-17] "30 segundos" e "15% das dúvidas com duas categorias" não aparecem em nenhum arquivo disponível; o único 15% do cenário é o de casos escalados ao supervisor | **Pendente (DT-17):** confirmar a origem; valores mantidos como herdados do 02. | 06 §1.1, O-01, C-29, VC-10, VC-34, DT-17 |
| [V-18] O ADR-0002 é um entregável do Tech Lead no exercício; a análise simulada do desenvolvedor estima ~2K de system, não ~4K | **Pendente (DT-09):** confirmar ou escrever o ADR; nota acrescentada ao §4.1. | 06 §4.1, C-31 |

## Apêndice · Mapa de IDs (versão anterior do 05 → final)

| ID anterior | Termo | ID final |
| --- | --- | --- |
| T-83 | Tier | T-02 |
| T-03 | Gold | T-03 (sem mudança) |
| T-05, T-06, T-11, T-15, T-16, T-19, T-26, T-27, T-28, T-36 | Mesmo termo do 01 | Sem mudança |
| T-09 | Tabela mensal de fretes | T-09 (Valor base, que inclui a tabela mensal) |
| T-85 | Valor base | T-09 |
| T-12 | Multiplicador regional | T-10 |
| T-75 | Regra de transição do PROC-042 | T-12 (Versão aplicável do PROC-042) |
| T-40 | Desconto por volume | T-13 |
| T-13 | Prazo adicional | T-16 (Prazo de entrega do frete especial) |
| T-84 | Data de recebimento | T-29 |
| T-81, T-82 | Incidente crítico; pausa do relógio | T-22 (e T-18 para horário comercial) |
| T-79 | Avaria na devolução | T-37 |
| T-80 | Sinistro / seguro de carga | T-37 (carga danificada) e T-38 (seguro) |
| T-41 | Exceção de devolução de carga perigosa | Incorporado ao T-26 (regra para classes não listadas) |
| T-42 | Chamado de atendimento | T-41 (Chamado, polissemia) |
| T-37 | Conflito registrado | TP-31 |
| T-55, T-56, T-57 | Fonte formal; fonte informal; vigência confirmada | TP-01, TP-02, TP-03 |
| T-43 | Data de referência do PROC-042 | TP-04 (com T-12 e T-41) |
| T-64, T-51 | Tema sensível; interpretação permitida | TP-06, TP-08 |
| T-77, T-78 | Domínio; categoria de dúvida | TP-11, TP-12 |
| T-44 a T-49 | Dúvida; parte; bloco; orientação; situação; nível de confiança | TP-13 a TP-18 |
| T-50, T-52, T-53, T-54 | Afirmação explícita; regra prevalente; bloco independente; dependência bloqueante | TP-19, TP-20, TP-21, TP-22 |
| T-73, T-76 | Dado do caso; pedido de ação | TP-23, TP-24 |
| T-59, T-60, T-61, T-62, T-63 | Citação; trecho literal; data de atualização; metadado ausente; documento ausente | TP-25 a TP-29 |
| T-74, T-58 | Coexistência de versões; divergência detectada | TP-30, TP-32 |
| T-65, T-67, T-68 | Validação humana; área de destino; área padrão | TP-33, TP-34, TP-35 |
| T-69, T-70, T-71, T-72 | Registro de interação; identificador; feedback; contestação | TP-36, TP-37, TP-38, TP-39 |
| T-66 | Indicação de escalonamento | TP-40 |
