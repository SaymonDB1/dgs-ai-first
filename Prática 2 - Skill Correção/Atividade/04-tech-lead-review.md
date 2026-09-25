# 04-tech-lead-review — Revisão pré-implementação do Assistente NovaTech

| Item | Conteúdo |
| --- | --- |
| Artefato | Revisão do Tech Lead |
| Data | 24/09/2026 |
| Artefatos revisados | `01-bounded-contexts-v1.md` (via referências), linguagem ubíqua (via referências), `02-requirements-v1.md`, `03-mockup-teams.png` |
| Status | Revisão emitida. Os documentos ainda não foram reescritos. |

**Limitação da revisão:** o `01-bounded-contexts-v1.md` e a linguagem ubíqua não foram anexados. Os itens BC-xx e LU-xx foram avaliados apenas pelo que o requirements referencia (contexts BC-xx, termos T-xx e o uso que os VCs fazem deles) e devem ser confirmados com os arquivos originais.

**Escala de criticidade:**

- **Crítica:** bloqueia a implementação ou torna um VC impossível de executar.
- **Alta:** leva a comportamento errado ou a retrabalho provável.
- **Média:** gera ambiguidade contornável.
- **Baixa:** cosmética ou de rastreabilidade.

---

## 1. Mapa de bounded contexts (via referências)

| ID | Artefato | Problema | Impacto | Criticidade | Alteração sugerida |
| --- | --- | --- | --- | --- | --- |
| BC-01 | Mapa de BCs | O arquivo não foi entregue. O cabeçalho do requirements cita a rev. 1.1, mas a §4.2 cita a rev. 1. | Não é possível validar fronteiras nem termos T-xx, e não se sabe qual revisão vale. | Alta | Anexar o mapa e alinhar o número de revisão em todos os artefatos. |
| BC-02 | Mapa de BCs | Nenhum BC é dono do mapeamento tema/context → área de destino. As áreas aparecem soltas nos VCs: Comercial, Diretoria Comercial, Operações, Compliance e Gestão de Riscos (ramal 4500). | O VC-36 (área de destino obrigatória) não tem fonte. Cada dev vai escolher uma área diferente. | Crítica | Criar uma tabela tema/BC → área única, sob responsabilidade do BC-03, e referenciá-la nos VCs. |
| BC-03 | Mapa de BCs | O BC-01 "identifica conflito", mas o BC-02 mantém o registro de conflitos em modo Conformist. Não está definido o que o BC-01 faz com uma divergência detectada em runtime que não está registrada. | Duas implementações possíveis (ignorar ou tratar como S6), com resultados opostos para o atendente. | Alta | Definir que divergência não registrada vira S6 com o rótulo "possível divergência" e gera pendência para o BC-02. Alternativa: o BC-01 usa só o registro. |
| BC-04 | Mapa de BCs | O registro da interação aparece no BC-03 ("registra o caso") e no BC-04 (RAS-02). | Risco de dois logs com dados divergentes e sem um dono claro. | Alta | Manter um único registro de interação com dono definido. BC-03 e BC-04 passam a referenciá-lo pelo ID da orientação. |
| BC-05 | Mapa de BCs | "Tier" aparece no BC-08 (prazos por tier) e no BC-09 (critério de tier). | Uma mudança de critério pode ser aplicada em dois lugares, e a área de escalonamento fica ambígua. | Alta | Tornar o BC-09 dono do conceito. O BC-08 consome o tier como dado, e a relação entre eles fica registrada no mapa. |
| BC-06 | Mapa de BCs | O desconto por volume está no PROC-042 (BC-05), mas "regras de desconto" também são consultadas no BC-09. | Não fica claro qual context classifica a parte da dúvida nem qual área recebe o escalonamento (VC-14, VC-38). | Alta | Atribuir a regra de desconto a um único BC e deixar o outro apenas como referência. |
| BC-07 | Mapa de BCs | Carga perigosa aparece no BC-07, no BC-10 (POL-001 §3.2) e no BC-05 (PROC-043). O VC-26 escala para Operações e Compliance, enquanto o BC-07 indica Gestão de Riscos. | Alerta e área inconsistentes para o mesmo tema sensível. | Alta | Tornar o BC-07 dono do alerta e da área. Os demais BCs só remetem a ele. |
| BC-08 | Mapa de BCs | O catálogo de temas sensíveis está espalhado: o BC-01 identifica, a lista está na C-26, os níveis estão na DP-08 e a validação no BC-03. | A lista fica sem dono e sem processo de atualização. | Média | Definir o dono do catálogo (BC-03 ou BC-02). O BC-01 apenas consome. |
| BC-09 | Mapa de BCs | Avaria aparece na POL-001 §3.5 (BC-10) e no FAQ #38 e seguro (BC-11). O VC-15 não diz em qual context a dúvida se enquadra. | Classificação e área de destino ambíguas. | Média | Explicitar a fronteira entre avaria na devolução e avaria/sinistro. |

## 2. Linguagem ubíqua (via referências)

| ID | Artefato | Problema | Impacto | Criticidade | Alteração sugerida |
| --- | --- | --- | --- | --- | --- |
| LU-01 | Linguagem ubíqua (T-05) | O T-05 define frete especial como "acima de 500 kg" e, com isso, resolve por definição a ambiguidade que o VC-07 manda não resolver (a §1 diz "acima de 500kg", a §2 diz "de 500kg a 1.000kg"). | Contradição direta entre o glossário (via C-06) e o VC-07. Se o modelo seguir o glossário, o VC-07 falha. | Alta | O T-05 passa a registrar a ambiguidade no limite de 500 kg e a remeter à fonte, sem fixar o valor. |
| LU-02 | Linguagem ubíqua (T-26) | O T-26 define carga perigosa como "classes 1 a 6 da ANTT", mas isso é o escopo da exceção da POL-001 §3.2, não a definição de carga perigosa. Os VC-28 e VC-30 tratam a classe 8 como perigosa. | Pelo glossário, a classe 8 não seria perigosa e não dispararia a validação humana. | Alta | Separar dois termos: "carga perigosa" (declarada como tal, sem enumerar classes, por causa da C-05) e "exceção de devolução de carga perigosa" (classes 1 a 6). |
| LU-03 | Linguagem ubíqua (T-03, T-05, T-28) | Os termos embutem valores de regra, como 7 dias úteis, 500 kg e Gold §1. | O glossário vira uma segunda fonte de verdade e fica desatualizado quando o documento mudar. | Média | O glossário define o conceito e aponta para a fonte. Valores ficam só nos documentos. |
| LU-04 | Linguagem ubíqua | "Chamado" tem três sentidos: chamado de atendimento (SLA), chamado de devolução (POL-001 §3.3) e "data de abertura do chamado" como data de referência do PROC-042, que é frete. | A regra de transição C-11 depende de uma data que o atendente pode interpretar errado. | Alta | Definir qual data vale para o PROC-042 (chamado, solicitação de cotação ou embarque) e dar um nome distinto a ela. |
| LU-05 | requirements / LU | As situações S1 a S8 são usadas em constraints e VCs sem uma tabela de definição. Fora do domínio, timeout e modo indisponível não correspondem a nenhuma S. | VCs que exigem "classificada como Sx" não podem ser verificados sem essa definição. | Crítica | Incluir a tabela S1 a S8 com gatilho, confiança e escalonamento, e declarar os estados extras. |
| LU-06 | Linguagem ubíqua | Vários termos não têm definição: "interpretação permitida", "explícita", "dependência bloqueante", "parte independente", "regra prevalente", "orientação utilizável" e "tema sensível do baseline". | Confiança (C-23), S2 (C-16) e escalonamento viram julgamento subjetivo. | Alta | Definir cada termo no glossário com um exemplo positivo e um negativo. |
| LU-07 | LU × mockup | O vocabulário diverge entre os artefatos: orientação × "Resposta"; feedback × contestação; escalonamento × escalar × encaminhar × validação humana; "Baixa evidência" × "Baixa confiança". | UI, logs e testes usam termos diferentes para a mesma coisa. | Média | Fixar um termo de domínio e o rótulo de UI correspondente para cada conceito. |
| LU-08 | Linguagem ubíqua | "Data de atualização da fonte" não está definida: pode ser a publicação do documento, a emissão ("emitido em" no mockup) ou a ingestão na base (FON-10). | C-20, VC-11 e a UI podem exibir datas diferentes. | Média | Definir o termo e o campo de metadado de origem. |
| LU-09 | LU / C-24 | Na C-24, "fonte com conflito registrado nunca recebe Alta" não diz se "fonte" é o documento ou a seção. A POL-001 tem conflito registrado (§3.5 × FAQ #38). | Se "fonte" for o documento, o VC-01 (POL-001 §3.1 com confiança Alta) e o mockup 1 falham. | Crítica | Definir a granularidade como seção ou regra. |

## 3. requirements.md

| ID | Artefato | Problema | Impacto | Criticidade | Alteração sugerida |
| --- | --- | --- | --- | --- | --- |
| RQ-01 | requirements O-02 × C-24 | O O-02 diz que "alertas de uma parte não contaminam a outra", mas a C-24 exibe um único nível de confiança, o menor entre os blocos. | Em dúvidas de duas categorias, a parte forte aparece rebaixada. Os dois requisitos se contradizem. | Alta | Decidir entre confiança por parte, confiança por parte com indicador global, ou só global, e alinhar O-02, C-24 e o mockup 2. |
| RQ-02 | requirements C-24 | Não está definido como combinar "Não se aplica" (S6/S7) com Alta, Média ou Baixa numa resposta de duas partes. | Não existe um "menor valor" determinístico para esse caso. | Alta | Definir a regra de agregação ou exibir a confiança por parte. |
| RQ-03 | requirements C-07/C-08 | Falta a situação por parte e a precedência entre situações. Exemplos: o VC-14 tem conflito formal × formal (S6) e formal × FAQ (S5) na mesma dúvida; o VC-09 mistura S2, S6 e confiança Média. | A classificação e os VCs do tipo "é S6" ficam ambíguos. | Alta | Definir a S por parte e uma ordem de precedência (por exemplo, S8 > S6 > S7 > S3 > S4 > S5 > S2 > S1). |
| RQ-04 | requirements O-02 | Não há comportamento definido para dúvidas com 3 ou mais categorias, duas perguntas da mesma categoria ou categoria não identificada. | Sem fallback, a parte excedente pode sumir, o que viola o O-02. | Média | Definir o limite de partes e o fallback correspondente (declarar a parte não tratada). |
| RQ-05 | requirements §1.1 × DP-31 | Seguro (BC-11), carga perigosa e desconto são atendidos, mas não fazem parte das 4 categorias. A fronteira entre "fora das categorias" e "fora do domínio" não está clara. | O classificador pode rejeitar dúvidas válidas como fora do domínio. | Média | Definir o domínio como o conjunto de BCs consultados. As categorias passam a ser só agrupamento. |
| RQ-06 | requirements C-14 × DP-29 | S7 exige um segundo turno (data, tier), mas o provisório da DP-29 ("cada orientação cumpre C-20 por si só") não diz como a resposta do atendente se vincula à pergunta original. Também não diz como o atendente indica "não tenho o dado". | Os fluxos dos VC-19 e VC-33 não fecham, e o caminho condicional da C-14 é inalcançável. | Crítica | Decidir a DP-29 ao menos para S7: guardar contexto do turno anterior (com janela e expiração) ou exigir reenvio da pergunta. Definir também a ação "não tenho o dado". |
| RQ-07 | requirements C-14 | O caminho "sem o dado, apresentar a regra de forma condicional" não tem VC e pode colidir com o VC-19 (não apresentar 1.4 nem 1.5 como valor único). | Comportamento não testado e com fronteira ambígua. | Média | Adicionar um VC para a resposta condicional, deixando claro que ela lista as duas condições com as duas citações. |
| RQ-08 | requirements C-11 | A regra não cobre chamado aberto antes de 01/12/2023 e já encerrado, nem como o sistema sabe se ele está "ainda em processamento", já que o S7 pede só a data. | Lacuna na regra de transição que pode levar à escolha errada da versão. | Alta | Completar a regra, ou tratar a lacuna como S6 ou S7 pedindo também o status. |
| RQ-09 | requirements VC-09 × C-10/VC-20 | O VC-09 exibe o multiplicador da v2 escolhido junto com o fator de peso e o prazo da v1 e da v2 lado a lado, dentro da mesma fórmula. Não fica claro se isso é "mistura" pela C-10. | Risco de a UI montar uma memória de cálculo com versões misturadas. | Alta | Explicitar que a fórmula aparece sem valores substituídos e que cada parâmetro aparece em seu bloco com a versão correspondente. |
| RQ-10 | requirements VC-09 | O VC-09 usa "chamado aberto hoje", então o resultado depende da data em que o teste roda. | Teste não determinístico. | Média | Fixar a data no enunciado. |
| RQ-11 | requirements VC-12 | A regra "nenhum número sem aparecer em trecho citado" é violada por metadados legítimos: Ref NT-2609-0417, "§3.1", "versão 3.1", datas, "2 partes" e números ecoados da pergunta ("12 fretes", "15/11/2023"). | Falsos positivos em massa, e o VC acaba ignorado. | Alta | Restringir a regra aos números de regra no corpo da orientação e listar as exceções. |
| RQ-12 | requirements VC-11 / C-19 | A exigência "literal caractere a caractere" não prevê tabelas (SLA-2024 §2) nem a normalização de espaços, aspas, hifenização e quebras vindas do PDF. | O VC fica inexequível para SLA e frágil para os demais documentos. | Alta | Definir as regras de normalização e um formato de citação de célula de tabela (linha, coluna e cabeçalho). |
| RQ-13 | requirements VC-01/02/31 | Os VCs exigem que a orientação "informe" strings exatas, mas não dizem se o texto gerado é comparado de forma literal ou semântica. O mockup 1 já diverge ("no tracking" em vez de "no sistema de tracking"). | Critério ambíguo entre QA e dev. | Alta | Tornar valores e prazos exatos obrigatórios, manter a citação literal e avaliar o texto livre por um checklist de elementos. |
| RQ-14 | requirements C-25 | A C-25 diz "sempre escalado", mas pelo escopo 2.2 o endpoint só indica o escalonamento. | Sugere uma ação que está fora do escopo. | Média | Reescrever como "sempre indica escalonamento com motivo e área". |
| RQ-15 | requirements §4.3 | O VC-36 depende da DP-30 (área padrão de fallback), que não está listada. As DP-03, DP-10 e DP-17 são citadas e também não aparecem, e os prefixos SEG e RNF não constam do cabeçalho da ESPEC-V2. | Decisões pendentes ficam invisíveis e o comportamento provisório não está definido. | Média | Completar a tabela de DPs com o comportamento provisório de cada uma. |
| RQ-16 | requirements VC-26/VC-38 | O VC-38 aceita "Diretoria Comercial ou o Comercial", e o VC-26 aponta Operações e Compliance em vez da Gestão de Riscos do BC-07. | Critério não binário e inconsistente com o BC-07. | Média | Uma área por caso, derivada da tabela do BC-02 (desta revisão). |
| RQ-17 | requirements C-26 × VCs | A C-26 inclui incidente crítico e desconto como temas sensíveis, mas os VC-31, VC-32 e VC-38 não verificam o alerta "Requer validação humana". | Um requisito de segurança fica sem teste, ou o tema está mal classificado. | Média | Adicionar a asserção aos VCs ou retirar esses temas do baseline. |
| RQ-18 | requirements VC-22 × §2.4 | O VC-22 aceita citar o FAQ #27 ("10 dias úteis") como S4, mas a §2.4 manda tratar o prazo padrão por rota como ausência (S3). | Não está definido se vale S3 ou S4 quando o FAQ cobre um documento formal ausente. | Alta | Definir a regra: citar o FAQ com Baixa e rótulo de ausente, ou omitir o FAQ. |
| RQ-19 | requirements C-20 | A C-20 exige versão e data em toda citação, mas o FAQ não tem nenhum dos dois (o mockup mostra "não controlada" e "diversas"). O fallback para metadado ausente não existe. | Citações do FAQ nunca cumprem a C-20 e o VC-11. | Alta | Definir valores sentinela para metadado ausente e o efeito disso na confiança. |
| RQ-20 | requirements (fallbacks) | Vários fallbacks estão indefinidos: o texto das mensagens de fallback parcial e de modo indisponível, o que conta como "parcial" num timeout, a confiança de fonte com falha de atualização (FON-10), dúvida em outro idioma (DP-24), anexo ou imagem enviados no Teams, e falha ao registrar a interação (entregar sem ID?). | Cada dev implementa o seu próprio fallback, e o VC-35 não tem saída esperada. | Crítica | Criar uma tabela de fallbacks com gatilho, mensagem, S, confiança, área e VC. |
| RQ-21 | requirements O-01 × VC-34 | O O-01 mede até o atendente "ver" a orientação, o que inclui o Teams, enquanto o VC-34 mede no endpoint. Os 100% exigidos não vêm com carga concorrente nem número de repetições. | Meta sem ponto de medição definido e frágil demais para CI. | Alta | Definir o ponto de medição, a carga, o número de repetições e o percentil de produção (DP-24). |
| RQ-22 | requirements C-28 × VC-37 | A orientação não pode reproduzir o CPF, mas o registro guarda a pergunta integral. Retenção e LGPD não são tratadas (DP-17). | Dado pessoal fica persistido no log. | Alta | Mascarar os dados pessoais antes de registrar e definir retenção e acesso ao log. |
| RQ-23 | requirements VC-40 | A C-28 cita CPF, endereço e dados do destinatário, mas o VC-40 testa só o CPF. | Cobertura parcial. | Baixa | Ampliar o VC para os outros tipos de dado. |
| RQ-24 | requirements §6 | C-31 e C-32 não têm VC direto. A §6.3 mapeia C-29 a C-32 para o VC-10, que não os verifica, e o O-06 inclui VC-04, VC-06 e VC-40, que não tratam de confiança. | Rastreabilidade enganosa. | Média | Corrigir as matrizes e adicionar VCs para C-31 (parte declarada ao estourar o orçamento) e C-32. |
| RQ-25 | requirements C-31 / ADR-0002 | O ADR-0002 não está disponível, e a própria §7 admite que o VC-09 pode não caber em 8K. O conteúdo fixo (registro de conflitos, glossário, catálogo de sensíveis) disputa os 4K de system. | Uma decisão anterior está sem validação, com risco de respostas parciais sistemáticas. | Alta | Obter o ADR e medir os tokens dos VC-09 e VC-14 antes de fechar a C-31. |
| RQ-26 | requirements C-03 | A premissa P-01 sustenta a confiança Alta dos VC-01, VC-02 e dos mockups 1 e 2 sem confirmação do BC-02. | Se a premissa cair, VCs e mockups passam a Média. | Alta | Confirmar com o BC-02 antes do sprint ou parametrizar os VCs pela vigência. |
| RQ-27 | requirements §2.4 × mockup | O contrato de saída foi declarado fora do escopo, mas a UI e os VCs dependem de campos estruturados: S e confiança por parte, citações com metadados, alertas, área, motivo e ref. | Sem contrato, o adapter do Teams e os testes automáticos não têm o que validar. | Crítica | Definir no requirements um schema lógico com os campos obrigatórios e deixar o formato para o `plan.md`. |
| RQ-28 | requirements | Identidade, autenticação e autorização do atendente não são mencionadas, mas o registro e o feedback precisam saber quem perguntou. | Dependência implícita de SSO do Teams e Entra ID. | Média | Declarar a dependência e dizer quais dados do usuário vão para o registro. |
| RQ-29 | requirements O-01/O-06 | Termos como "orientação utilizável" e "sabe quanto pode confiar" são subjetivos. | Baixo, porque os sinais observáveis compensam. | Baixa | Remover os adjetivos ou apontar para os sinais observáveis. |

## 4. Mockup Teams

| ID | Artefato | Problema | Impacto | Criticidade | Alteração sugerida |
| --- | --- | --- | --- | --- | --- |
| MK-01 | Mockup (todas as telas) | O botão "Escalar ao supervisor" é uma ação executável, mas a DP-09 está aberta e o escopo diz que a execução não é do endpoint. "Supervisor" não é uma área, e a barra lateral ("Supervisão SAC N2 · Canal de escalonamento") decide a DP-09 implicitamente. | A UI assume uma decisão que ainda não foi tomada e um fluxo sem especificação. | Alta | Até a DP-09, trocar por "Área indicada: X". Se a decisão já existe, registrá-la. |
| MK-02 | Mockup 1, 2, 4, 5 | A área de destino só aparece na tela 3. A tela 4 (sensível, só FAQ) não mostra área. | Viola o VC-36 e a C-25. | Alta | Exibir motivo e área sempre que houver escalonamento. |
| MK-03 | Mockup 4 | Os percentuais do FAQ aparecem como título afirmativo ("Seguro adicional de 0,3%..."), com o mesmo peso visual de uma resposta formal. | Contraria a C-25 e o próprio alerta "Não use como regra". O atendente tende a ler só o título. | Alta | Usar um título neutro ("Não há regra formal sobre seguro") e mostrar os valores só dentro do bloco informal. |
| MK-04 | Mockup 2 | Há um único nível de confiança e alertas globais para as duas partes. | Reflete a contradição do RQ-01. | Média | Ajustar depois da decisão do RQ-01. |
| MK-05 | Mockup | Vários estados não foram mockados: S2 e documento ausente, S3, S5 (regra + FAQ, VC-15), S7, S8, fora do domínio, timeout e indisponível, carga perigosa (VC-27), coexistência com uma só versão recuperada (VC-16) e confiança Média. | A UI de metade das situações vai ser inventada na implementação. | Alta | Completar os estados antes do desenvolvimento da UI. |
| MK-06 | Mockup | Os rótulos "Resposta completa", "Resposta completa · 2 partes", "Conflito não resolvido" e "Baixa evidência" não têm mapeamento para S1 a S8. | Rótulos e lógica ficam desalinhados. | Média | Criar a tabela S → rótulo de UI. |
| MK-07 | Mockup | Os metadados variam entre telas: "Atualizado em" × "emitido em" (tela 3); "Versão 1.0/2.0" × "v1/v2"; título da POL-001 com e sem "de Mercadorias". | Citação inconsistente, o que afeta a C-20 e o LU-08. | Média | Padronizar os campos e a nomenclatura de versão. |
| MK-08 | Mockup 2 | Uma linha de tabela reconstruída do SLA-2024 aparece como se fosse trecho literal. | Contradiz o VC-11 como está escrito hoje. | Média | Ajustar depois da regra definida no RQ-12. |
| MK-09 | Mockup 3 | A tela traz uma análise gerada sem citação ("Diferença… Impacto: informar ao cliente um desconto que pode não se aplicar"). | Pode violar a C-21. O requirements não autoriza nem proíbe síntese comparativa. | Média | Decidir se a comparação é permitida e, se for, marcá-la como leitura do assistente e não como regra. |
| MK-10 | Mockup 5 | A tela implementa captura de feedback, uma taxonomia de motivos (Desatualizada/Incompleta/Ambígua) e a mensagem "encaminhada para revisão". Isso é do BC-04, fora do escopo do endpoint e sem spec. Faltam motivos como "fonte errada" e "outro", e não está definido o que acontece com "Correta" ou com "Incorreta" em S3 e S6. | Uma funcionalidade sem requisito entra por meio do mockup. | Alta | Escrever uma spec própria do BC-04 ou marcar a tela como fora do MVP. |
| MK-11 | Mockup | O texto "use esta referência para contestar ou acompanhar" pressupõe consulta de status (FBK-03), que não foi especificada. O formato NT-2609-0417 (aparentemente ano e mês mais 4 dígitos) não está definido. | Promessa sem fluxo por trás, com risco de colisão ou estouro do ID. | Média | Definir o formato do ID e retirar "acompanhar" até existir o fluxo. |
| MK-12 | Mockup × C-32 | Respostas longas, como a do VC-09 com cerca de 6 blocos, podem exceder o limite de tamanho de card do Teams ou ser colapsadas ("ver mais"), escondendo alertas. O mockup só mostra casos curtos. | Viola a C-32 justamente nos casos mais arriscados. | Alta | Definir a prioridade de exibição (alertas primeiro) e mockar e testar o VC-09 no Teams. |
| MK-13 | Mockup × DP-24 | O canal Teams ainda é decisão pendente na DP-24. Há dependências implícitas: o Bot Framework tem timeout curto para resposta síncrona (cerca de 15 s, a confirmar), o que obriga a entrega assíncrona com indicador de digitação para cumprir os 30 s; e a renderização de markdown pode alterar trechos literais (\*, \_). | O requisito de 30 s e o de citação literal dependem do canal. | Alta | Registrar a decisão de canal num ADR e acrescentar os requisitos de entrega assíncrona e de escape de texto. |
| MK-14 | Mockup 4 | A tela afirma que o FAQ "não é validado por Compliance ou Operações", mas a C-02 só diz "sem responsável". | Afirmação sem fonte na própria UI. | Baixa | Alinhar o texto com a C-02. |

---

## 5. Bloqueios e decisões pendentes

### 5.1 Bloqueios para iniciar a implementação (Crítica)

| Item | Tema |
| --- | --- |
| LU-05, RQ-03 | Definição de S1 a S8 e agregação por parte |
| RQ-27 | Contrato lógico de saída do endpoint |
| RQ-20 | Tabela de fallbacks |
| RQ-06 | Fluxo de segundo turno para S7 |
| LU-09 | Granularidade de "fonte" na regra de confiança |
| BC-02 | Mapeamento tema → área de destino |

### 5.2 Decisões que o time precisa tomar

| Decisão | Itens relacionados |
| --- | --- |
| Confiança por parte ou global | RQ-01, RQ-02, MK-04 |
| DP-09: mecanismo e canal de escalonamento, que o mockup já está assumindo | MK-01, MK-02 |
| DP-24: canal (Teams) e percentil de tempo em produção | RQ-21, MK-13 |
| DP-29: contexto entre turnos | RQ-06 |
| DP-30: área padrão de fallback | RQ-15, BC-02 |
| Se a síntese comparativa em conflitos é permitida | MK-09 |
| Se o feedback do BC-04 entra no MVP | MK-10, MK-11 |
| Regra de normalização da citação literal | RQ-12, MK-08 |
| Confirmação da premissa P-01 com o BC-02 | RQ-26 |
| Confirmação dos valores do ADR-0002 | RQ-25 |

### 5.3 Próximo passo

Reescrever os artefatos aplicando as alterações sugeridas, começando pelos bloqueios da seção 5.1.
