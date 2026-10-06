# Assistente de Normas do Cadastro Positivo (RAG)

> Assistente de IA que responde perguntas sobre a regulamentação do Cadastro Positivo, sempre citando o artigo da norma de onde tirou a resposta.
> Feito com Python, Gemini (Vertex AI), BigQuery e Streamlit.

**Status:** 🟡 Semana 1 de 6: discovery concluído, desenvolvimento ainda não começou.

---

## 1. Problema

Analistas de crédito, compliance e produto precisam consultar com frequência as regras do Cadastro Positivo. Essas regras estão espalhadas em vários normativos: uma lei, uma lei complementar que a alterou, um decreto e resoluções e circulares do CMN e do Banco Central.

Hoje a consulta acontece assim:

- o analista abre PDFs longos e procura por palavra-chave (Ctrl+F);
- precisa descobrir qual norma está em vigor e qual trecho foi alterado depois;
- quando não acha, pergunta a um colega ou ao jurídico, e espera a resposta.

**Resultado:** perda de tempo, respostas inconsistentes entre analistas e risco de aplicar uma regra desatualizada.

> **Declaração do problema:** analistas perdem tempo e confiança ao buscar regras em normativos longos e fragmentados, e não têm um jeito rápido de chegar à resposta com a fonte correta.

## 2. Usuário e hipótese

### Persona principal: Ana, analista de crédito pleno

- Trabalha em uma instituição financeira de médio porte.
- Consulta normativos algumas vezes por semana, em geral para tirar dúvidas operacionais ("posso usar esse dado?", "em quanto tempo preciso corrigir?").
- Não é advogada. Precisa de uma resposta clara **e** da referência exata para conferir ou repassar.

### Personas secundárias

- **Analista de compliance:** precisa da citação exata para registrar em pareceres.
- **PO ou analista de produto de crédito:** precisa entender as restrições regulatórias antes de desenhar uma funcionalidade.

### Hipótese de valor

> **Acreditamos que** um assistente que responde em linguagem natural e **sempre cita o artigo da norma de origem**
> **para** analistas de crédito e compliance
> **vai reduzir** o tempo para chegar a uma resposta confiável de vários minutos para menos de 1 minuto,
> **sem perder precisão**, porque toda resposta pode ser conferida na fonte.
>
> **Saberemos que é verdade quando** o assistente acertar pelo menos 80% de um conjunto de validação com 30 perguntas e citar a fonte correta em pelo menos 90% das respostas.

### Fora do escopo do MVP

- Dar parecer jurídico ou substituir o jurídico. O assistente informa o que a norma diz, sem interpretar casos concretos.
- Normas de outros temas, como provisão, risco de crédito ou open finance.
- Acompanhar automaticamente novas publicações do Banco Central.
- Usar dados internos ou confidenciais. O corpus tem **apenas documentos públicos**.

## 3. Métricas de sucesso

| Tipo | Métrica | Como medir | Meta do MVP |
|---|---|---|---|
| **North Star** | % de respostas corretas | Conjunto de 30 perguntas com resposta esperada, avaliado manualmente (certo / parcial / errado) | ≥ 80% corretas |
| Confiabilidade | % de respostas com a citação correta | A resposta cita a norma e o artigo que contêm a informação? | ≥ 90% |
| Segurança | % de recusas corretas | Para perguntas fora do escopo ou sem resposta na base, o assistente diz "não encontrei" em vez de inventar | 100% das perguntas "pegadinha" |
| Performance | Tempo de resposta (p95) | Log no BigQuery: momento da pergunta até a resposta | < 10 s |
| Custo | Custo por pergunta | Tokens consumidos × preço do modelo | < R$ 0,05 |
| Uso (pós-MVP) | Avaliação do usuário | Botões 👍/👎 na interface, gravados no BigQuery | ≥ 75% positivas |

> A meta de 80% é o ponto de partida. Se a primeira avaliação ficar abaixo, isso vira um item do backlog: ajustar o tamanho dos trechos, o prompt ou o número de trechos recuperados, e medir de novo.

## 4. Corpus inicial (documentos públicos)

| Norma | Assunto |
|---|---|
| Lei nº 12.414/2011 | Lei do Cadastro Positivo (texto compilado, já com as alterações da LC 166/2019) |
| Lei Complementar nº 166/2019 | Alterou a Lei 12.414: inclusão automática no cadastro (o titular pode pedir para sair) |
| Decreto nº 9.936/2019 | Regulamenta a Lei 12.414 |
| Resolução CMN nº 4.737/2019 (alterada pela Res. CMN nº 4.990/2022) | Envio de dados pelas instituições financeiras aos gestores e requisitos para registro dos gestores no Banco Central |
| Resolução BCB nº 14/2020 | Procedimentos de registro dos gestores no Banco Central (substituiu a Circular 3.955/2019) |

> ✅ Vigência conferida em 05/10/2026. Detalhes, links oficiais e normas excluídas em [`docs/fontes-normativas.md`](docs/fontes-normativas.md).

## 5. Solução proposta

```
Pergunta (Streamlit)
      │
      ▼
Embedding da pergunta (Vertex AI)
      │
      ▼
Busca vetorial no BigQuery (VECTOR_SEARCH) ──► top-k trechos + metadados (norma, artigo)
      │
      ▼
Gemini gera a resposta usando SÓ os trechos recebidos e citando a fonte
      │
      ▼
Resposta + fontes na tela  ──►  log da pergunta, resposta, latência e feedback no BigQuery
```

**Tabelas no BigQuery**

- `trechos`: id, norma, artigo, texto, embedding, url_fonte, data_coleta
- `log_perguntas`: id, timestamp, pergunta, resposta, ids_trechos, latência_ms, tokens, feedback
- `avaliacao`: id_pergunta, pergunta, resposta_esperada, resposta_obtida, nota, citação_correta, rodada

## 6. Backlog (User Stories)

Priorização: **MoSCoW**. Must = essencial para o MVP. Should = importante, mas o MVP funciona sem. Could = desejável se sobrar tempo.

### US01: Fazer uma pergunta em linguagem natural · *Must*

**Como** analista de crédito, **quero** digitar uma dúvida em português comum **para** não precisar saber em qual norma está a resposta.

```gherkin
Cenário: Pergunta dentro do escopo
  Dado que estou na tela inicial do assistente
  Quando eu pergunto "Qual o prazo para o gestor corrigir uma informação contestada?"
  Então recebo uma resposta em português em até 10 segundos
  E a resposta tem no máximo 5 parágrafos
```

### US02: Ver a fonte de cada resposta · *Must*

**Como** analista de compliance, **quero** ver a norma e o artigo que fundamentam a resposta **para** conferir e citar em pareceres.

```gherkin
Cenário: Resposta com citação
  Dado que fiz uma pergunta dentro do escopo
  Quando a resposta é exibida
  Então ela mostra ao menos uma referência no formato "Norma, art. X"
  E consigo expandir o trecho original usado na resposta
  E há um link para o documento oficial
```

### US03: Não inventar respostas · *Must*

**Como** analista, **quero** que o assistente diga quando não sabe **para** não tomar decisões com base em informação inventada.

```gherkin
Cenário: Pergunta sem resposta na base
  Dado que a base contém apenas normas do Cadastro Positivo
  Quando eu pergunto "Qual o percentual de provisão para operações em atraso de 90 dias?"
  Então o assistente responde que não encontrou essa informação nas normas disponíveis
  E não apresenta nenhum número ou regra
```

### US04: Carregar e atualizar os normativos · *Must*

**Como** PO do produto, **quero** um processo repetível para carregar os normativos no BigQuery **para** atualizar a base quando uma norma mudar.

```gherkin
Cenário: Carga de um novo normativo
  Dado um PDF ou HTML de norma pública
  Quando executo o script de ingestão
  Então o texto é dividido em trechos, com a norma e o artigo identificados
  E os trechos são gravados na tabela "trechos" com embedding e data de coleta
  E executar o script de novo com o mesmo arquivo não duplica os trechos
```

### US05: Medir a qualidade com um conjunto de validação · *Must*

**Como** PO, **quero** rodar as 30 perguntas de teste e obter a taxa de acerto **para** saber, com dados, se o assistente está pronto.

```gherkin
Cenário: Rodada de avaliação
  Dado o arquivo com 30 perguntas e respostas esperadas
  Quando executo o script de avaliação
  Então cada resposta é gravada na tabela "avaliacao" com o número da rodada
  E é gerado um resumo com % de acerto, % de citações corretas e latência p95
```

### US06: Registrar o uso e o feedback · *Should*

**Como** PO, **quero** que cada pergunta e cada avaliação 👍/👎 fiquem registradas **para** identificar lacunas da base e priorizar melhorias.

```gherkin
Cenário: Feedback negativo
  Dado que recebi uma resposta
  Quando eu clico em 👎
  Então o feedback é gravado em "log_perguntas" junto com a pergunta e a resposta
  E nenhum dado pessoal do usuário é gravado
```

### US07: Ver perguntas de exemplo · *Should*

**Como** novo usuário, **quero** ver exemplos de perguntas **para** entender o que o assistente sabe responder.

```gherkin
Cenário: Primeiro acesso
  Dado que abri o assistente sem histórico
  Então vejo de 3 a 5 perguntas de exemplo clicáveis
  E um aviso de que as respostas não substituem um parecer jurídico
```

### US08: Saber a data da base · *Could*

**Como** analista, **quero** saber até que data os normativos foram atualizados **para** avaliar se preciso conferir alterações recentes.

```gherkin
Cenário: Data da base visível
  Dado que estou em qualquer tela do assistente
  Então vejo a data da última coleta dos normativos
```

## 7. Plano de validação

Mesmo princípio usado na minha rotina de trabalho: **validar com dados antes de confirmar**.

- **30 perguntas**, cada uma com resposta esperada e fonte (norma e artigo), escritas **antes** de rodar o modelo, para evitar viés.
- Distribuição:
  - 15 perguntas **factuais diretas** (ex.: prazos, direitos do titular);
  - 8 perguntas **que exigem combinar trechos** (ex.: regra original + alteração da LC 166);
  - 4 perguntas **operacionais** escritas como um analista falaria;
  - 3 perguntas **"pegadinha"**, fora do escopo ou sem resposta, para testar a US03.
- **Critério de nota:** certo (responde corretamente e cita a fonte certa) / parcial (resposta incompleta ou citação imprecisa) / errado.
- **Rodadas:** linha de base → ajustes → nova rodada. Cada rodada fica registrada na tabela `avaliacao`, para mostrar a evolução.

## 8. Roadmap

| Semana | Entrega | Status |
|---|---|---|
| 1 | Discovery e README de produto | ✅ |
| 2 | Coleta dos normativos, conferência da vigência ✅ e carga no BigQuery (US04) | 🟡 |
| 3–4 | RAG em Python com Gemini (US01, US02, US03) | ⬜ |
| 5 | 30 perguntas, avaliação e ajustes (US05) | ⬜ |
| 6 | Interface em Streamlit (US06, US07), README final e vídeo de demonstração | ⬜ |

## 9. Resultados

*Esta seção será preenchida na semana 5.*

| Rodada | % acerto | % citação correta | Recusas corretas | Latência p95 | Principal ajuste |
|---|---|---|---|---|---|
| 1 (linha de base) | – | – | – | – | – |
| 2 | – | – | – | – | – |

## 10. Próximos passos (pós-MVP)

- Ampliar o corpus para outros temas de crédito, como gestão de risco de crédito e provisão.
- Monitorar novas publicações do Banco Central e alertar quando uma norma da base for alterada.
- Responder considerando a versão da norma vigente em uma data específica.
- Avaliação automática com um segundo modelo como juiz, sempre comparada com a nota manual.

## 11. Stack

Python · Gemini (Vertex AI) · Vertex AI Embeddings · BigQuery (armazenamento, busca vetorial e logs) · Streamlit

---

**Aviso:** projeto pessoal e educacional. Usa apenas normas públicas e não constitui orientação jurídica.
