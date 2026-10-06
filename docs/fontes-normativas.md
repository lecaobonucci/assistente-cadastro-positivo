# Fontes normativas do corpus

**Data da conferência de vigência:** 05/10/2026
**Fontes consultadas:** Portal da Legislação do Planalto e Busca de Normas do Banco Central.

## Normas que entram no corpus

| # | Norma | Assunto | Situação | Link oficial |
|---|---|---|---|---|
| 1 | Lei nº 12.414, de 9/6/2011 | Lei do Cadastro Positivo: formação e consulta de bancos de dados com histórico de pagamento | Em vigor, com as alterações da LC 166/2019 | [Planalto](https://www.planalto.gov.br/ccivil_03/_ato2011-2014/2011/lei/l12414.htm) |
| 2 | Lei Complementar nº 166, de 8/4/2019 | Alterou a Lei 12.414: inclusão automática (*opt-out*), compartilhamento de dados e responsabilidades | Em vigor | [Planalto](https://www.planalto.gov.br/ccivil_03/leis/lcp/lcp166.htm) |
| 3 | Decreto nº 9.936, de 24/7/2019 | Regulamenta a Lei 12.414. Revogou o Decreto 7.829/2012 | Em vigor | [Planalto](https://www.planalto.gov.br/ccivil_03/_ato2019-2022/2019/decreto/D9936.htm) |
| 4 | Resolução CMN nº 4.737, de 29/7/2019 | Envio de dados de adimplemento pelas instituições financeiras aos gestores. Condições para registro e cancelamento do registro dos gestores no BC | Em vigor, alterada pela Res. CMN 4.990/2022 | [Banco Central](https://www.bcb.gov.br/estabilidadefinanceira/exibenormativo?tipo=Resolu%C3%A7%C3%A3o&numero=4737) |
| 5 | Resolução CMN nº 4.990, de 24/3/2022 | Altera a Res. 4.737: definições de controlador e grupo de controle, e ajustes nos arts. 6 a 9 e 13 | Em vigor desde 1/4/2022 | [Banco Central](https://www.bcb.gov.br/estabilidadefinanceira/exibenormativo?tipo=Resolu%C3%A7%C3%A3o%20CMN&numero=4990) |
| 6 | Resolução BCB nº 14, de 9/9/2020 | Procedimentos de registro dos gestores, cancelamento, diretores e mudança de controle. Envio de dados por administradoras de consórcio | Em vigor desde 1/10/2020 | [Banco Central](https://www.bcb.gov.br/estabilidadefinanceira/exibenormativo?tipo=Resolu%C3%A7%C3%A3o%20BCB&numero=14) |

## Normas excluídas e por quê

| Norma | Motivo |
|---|---|
| Circular BCB nº 3.955/2019 | Revogada pela Resolução BCB nº 14/2020, a partir de 1/10/2020 |
| Decreto nº 7.829/2012 | Revogado pelo Decreto 9.936/2019 |
| Resolução CMN nº 4.172/2012 | Revogada pela Resolução CMN 4.737/2019 |
| Resoluções BCB nº 317/2023, 341/2023 e 548/2026 | Tratam só dos prazos do BC para decidir pedidos de autorização em geral. Não mudam as regras do Cadastro Positivo |

## Decisões de produto

- **Usar o texto compilado da Lei 12.414.** Ele já incorpora as alterações da LC 166/2019, o que evita que o assistente responda com a redação antiga. A LC 166 entra no corpus só para responder perguntas do tipo "o que mudou em 2019?".
- **Da Res. CMN 4.737, usar a versão já atualizada pela 4.990/2022**, quando o Banco Central disponibilizar. Se não houver texto consolidado, carregar as duas normas e marcar no metadado qual artigo foi alterado.
- **Registrar a data de coleta** em cada trecho (coluna `data_coleta`), para cumprir a US08.

## Como reconferir

1. No Planalto, abra a página da norma e procure por "Revogado" ou "Redação dada pela...".
2. No Banco Central, use a [Busca de Normas](https://www.bcb.gov.br/estabilidadefinanceira/buscanormas). A página de cada norma mostra se ela foi revogada e lista as alterações.
3. Atualize a data no topo deste arquivo.
