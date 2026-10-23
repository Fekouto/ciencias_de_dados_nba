# Dicionário de dados — NBA / CP1

Este documento descreve os dados usados na coleta e validação. Não é apenas um glossário de basquete: inclui granularidade, tipos lógicos, chaves, unidades e campos criados pelo projeto. Base: cabeçalhos dos CSVs do repositório consultados em 23/10/2026 e versão revisada do notebook com nove módulos. Mudanças de esquema devem ser registradas aqui.

## 1. Convenções

- Tipos abaixo são lógicos; a leitura automática de CSV pode produzir outros tipos.
- IDs são identificadores, não medidas. `GAME_ID` deve ser lido como texto para preservar zeros iniciais.
- `SEASON` é texto no formato `AAAA-AA`, por exemplo `2024-25`.
- Ausente não é sinônimo de zero. Campos sem valor devem ser investigados antes de preenchimento ou exclusão.
- Proporções como `FG_PCT` são armazenadas em escala 0–1: 0,45 corresponde a 45%. Não dividir novamente por 100.
- Campos cadastrais e métricas auxiliares são preservados; sua presença não implica uso na análise.

## 2. Tabelas e relacionamentos

| Conjunto | Caminho | Uma linha representa | Chave esperada |
|---|---|---|---|
| Estatísticas | dados/jogadores_por_temporada/<temporada>.csv | Jogador na temporada regular | PLAYER_ID + SEASON |
| Eventos | dados/arremessos/<temporada>/<PLAYER_ID>.csv | Tentativa de quadra | GAME_ID + GAME_EVENT_ID |
| Perfis | dados/perfis_por_temporada/<temporada>.csv | Perfil retornado para jogador e temporada | PLAYER_ID + SEASON |

Para associar eventos a perfis ou totais, utilizar `PLAYER_ID` e `SEASON`, com relação muitos-para-um. Nomes não são chaves. Validar unicidade e correspondência antes da integração. A auditoria atual verifica duplicatas de eventos dentro de cada arquivo, não globalmente em todos os arquivos.

## 3. Campos compartilhados

| Campo | Tipo / unidade | Significado e observação |
|---|---|---|
| PLAYER_ID | Inteiro identificador positivo | Código do jogador usado nas consultas e junções |
| PLAYER_NAME | Texto | Nome retornado pela API; apresentação, não chave |
| SEASON | Texto | Temporada adicionada pelo projeto conforme a consulta |
| TEAM_ID | Identificador | Equipe informada pelo endpoint; não usar para inferir toda a trajetória sazonal |
| TEAM_ABBREVIATION | Texto | Abreviação da equipe nos conjuntos agregados |
| AGE | Numérico / anos | Idade informada pelo endpoint para o recorte |

## 4. Estatísticas de referência — LeagueDashPlayerStats

Consulta em modo `Totals`. Contagens são acumuladas no recorte; percentuais continuam sendo razões. FGA e FGM se referem a arremessos de quadra, excluindo lances livres.

| Campo | Tipo / unidade | Uso ou definição operacional |
|---|---|---|
| FGA | Inteiro / tentativas | Total de tentativas de quadra; referência para o número de eventos |
| FGM | Inteiro / acertos | Total convertido; referência para a soma da flag de acerto |
| FG3A | Inteiro / tentativas | Referência para eventos classificados como três pontos |
| FG3M | Inteiro / acertos | Referência para conversões entre os eventos de três pontos |
| FG_PCT | Decimal / proporção | FGM dividido por FGA quando o denominador é positivo |
| FG3_PCT | Decimal / proporção | FG3M dividido por FG3A quando o denominador é positivo |
| GP | Inteiro / jogos | Participações do jogador no recorte |
| W, L | Inteiros / jogos | Vitórias e derrotas nos jogos contabilizados |
| W_PCT | Decimal / proporção | Participação de vitórias no total de jogos |
| MIN | Decimal / minutos | Tempo acumulado em quadra; não interpretar como formato minutos:segundos |
| FTM, FTA | Inteiros | Conversões e tentativas de lance livre |
| FT_PCT | Decimal / proporção | FTM dividido por FTA quando o denominador é positivo |
| OREB, DREB, REB | Inteiros | Rebotes ofensivos, defensivos e totais |
| AST | Inteiro | Assistências |
| TOV | Inteiro | Perdas de posse atribuídas ao jogador |
| STL | Inteiro | Roubos de bola |
| BLK, BLKA | Inteiros | Tocos realizados e tocos sofridos |
| PF, PFD | Inteiros | Faltas pessoais cometidas e faltas sofridas |
| PTS | Inteiro / pontos | Pontuação acumulada, incluindo lances livres |
| PLUS_MINUS | Numérico / pontos | Saldo da equipe com o jogador em quadra, acumulado no recorte |
| DD2, TD3 | Inteiros / jogos | Ocorrências de double-double e triple-double |
| NICKNAME | Texto | Nome curto retornado pelo serviço |
| TEAM_COUNT | Inteiro | Contagem de equipes informada pelo endpoint |
| NBA_FANTASY_PTS, WNBA_FANTASY_PTS | Numéricos | Pontuações de fantasy retornadas; fórmulas não usadas nem validadas pelo projeto |
| FP_HIGH_SCORE | Numérico | Campo auxiliar de fantasy preservado; sem interpretação analítica validada neste projeto |
| Campos terminados em _RANK | Inteiros ordinais | Posição no ranking da métrica indicada pelo prefixo, no universo consultado; não assumir que número maior é melhor |

A regra `_RANK` cobre todos os campos com esse sufixo presentes no CSV, inclusive métricas de fantasy. Zero tentativas exige cuidado na interpretação do percentual retornado: não representa evidência de desempenho nulo.

## 5. Eventos — ShotChartDetail

Consulta com `context_measure_simple="FGA"`, temporada regular e `team_id=0`. O projeto utiliza o conjunto `shot_chart_detail`; tabelas auxiliares da resposta não são exportadas por esse módulo.

| Campo | Tipo / unidade | Significado ou domínio |
|---|---|---|
| GRID_TYPE | Texto | Identificação do conjunto, usualmente Shot Chart Detail |
| GAME_ID | Texto identificador | Código do jogo, mantendo zeros à esquerda |
| GAME_EVENT_ID | Inteiro identificador | Código do evento dentro do jogo; isoladamente não é único |
| PLAYER_ID, PLAYER_NAME | Identificador / texto | Jogador responsável pelo evento |
| TEAM_ID, TEAM_NAME | Identificador / texto | Equipe vinculada ao evento |
| PERIOD | Inteiro | Período; 1–4 regulamentares e valores posteriores para prorrogações |
| MINUTES_REMAINING | Inteiro / minutos | Componente de minutos restantes no período |
| SECONDS_REMAINING | Inteiro / segundos | Componente de segundos restantes, normalmente 0–59 |
| EVENT_TYPE | Categoria | Descrição do resultado, como Made Shot ou Missed Shot |
| ACTION_TYPE | Categoria | Descrição do movimento/técnica, por exemplo Jump Shot |
| SHOT_TYPE | Categoria | 2PT Field Goal ou 3PT Field Goal |
| SHOT_ZONE_BASIC | Categoria | Zona geral fornecida pela NBA |
| SHOT_ZONE_AREA | Categoria | Área direcional da zona |
| SHOT_ZONE_RANGE | Categoria | Faixa de distância fornecida pela fonte |
| SHOT_DISTANCE | Numérico / pés | Distância registrada para o evento; conservar a precisão da fonte |
| LOC_X, LOC_Y | Numéricos / coordenadas da fonte | Posição lateral e longitudinal. A convenção usada no mapa do projeto é dividir por 10 para obter pés; validar escala e orientação antes da visualização |
| SHOT_ATTEMPTED_FLAG | Binário | Deve ser 1 para os eventos de tentativa selecionados |
| SHOT_MADE_FLAG | Binário | 1 indica conversão; 0 indica erro |
| GAME_DATE | Texto de data | Formato AAAAMMDD; converter explicitamente antes de análises temporais |
| HTM, VTM | Texto | Abreviações da equipe mandante e visitante |
| SEASON | Texto | Temporada adicionada pelo projeto |

Métricas deriváveis: tentativas = número de eventos; acertos = soma de SHOT_MADE_FLAG; erros = tentativas menos acertos. Aproveitamento = acertos/tentativas apenas quando há tentativas. Não há uma coluna obrigatória de percentual de erro.

## 6. Perfis — LeagueDashPlayerBioStats

| Campo | Tipo / unidade | Significado ou cuidado |
|---|---|---|
| PLAYER_HEIGHT | Texto / pés-polegadas | Representação cadastral, por exemplo 6-6; não é número decimal |
| PLAYER_HEIGHT_INCHES | Numérico / polegadas | Altura numérica usada na validação |
| PLAYER_WEIGHT | Numérico lógico / libras | Peso; pode ser lido como texto ou ausente |
| COLLEGE | Texto | Instituição informada na origem |
| COUNTRY | Texto | País informado no cadastro; não inferir local de nascimento sem confirmação |
| DRAFT_YEAR | Texto | Ano do draft ou categoria como Undrafted |
| DRAFT_ROUND | Texto | Rodada do draft ou categoria não numérica |
| DRAFT_NUMBER | Texto | Escolha do draft ou categoria não numérica |
| GP, PTS, REB, AST | Numéricos | Jogos, pontos, rebotes e assistências no modo retornado; conferir parâmetros antes de combinar com outras tabelas |
| NET_RATING | Numérico | Indicador de saldo por 100 posses retornado pela NBA; não equivale ao PLUS_MINUS bruto |
| OREB_PCT, DREB_PCT | Decimais / proporções | Indicadores relativos de rebotes ofensivos e defensivos |
| USG_PCT | Decimal / proporção | Indicador de uso das posses |
| TS_PCT | Decimal / razão | Eficiência de pontuação que considera lances livres; não aplicar automaticamente limite máximo de 1 |
| AST_PCT | Decimal / proporção | Indicador relativo de assistências |
| BIO_COLLECTED_AT | Texto / data-hora ISO 8601 UTC | Momento da consulta de perfil, adicionado pelo projeto; não é data da medição física |

Identificadores, nome, equipe, idade e temporada seguem a seção 3. Métricas avançadas são auxiliares e não participam da auditoria; suas fórmulas devem ser documentadas antes de uso analítico. Conversões possíveis na CP2: polegadas × 2,54 para centímetros e libras × 0,45359237 para quilogramas. Preservar as colunas originais.

Não existe coluna POSITION nos cabeçalhos de perfis conferidos. O recorte sazonal do endpoint não garante a historicidade exata de altura e peso.

## 7. Logs de coleta

| Campo | Tipo | Significado |
|---|---|---|
| PLAYER_ID, SEASON | Identificador / texto | Par consultado |
| TENTATIVA | Inteiro | Número da tentativa, de 1 a 3 |
| STATUS | Categoria | falha, salvo ou resposta_vazia |
| LINHAS | Inteiro ou ausente | Quantidade recebida; ausente em falha |
| ERRO | Texto | Mensagem de falha; vazio quando não registrada |
| DATA_UTC | Data-hora ISO 8601 | Horário de registro da ocorrência |

Um par pode ter várias linhas de log. Resposta vazia não equivale automaticamente a falha. Arquivos reutilizados não aparecem necessariamente no log da execução atual.

## 8. Auditorias e resumo

| Campo ou família | Significado |
|---|---|
| FGA_ESPERADO, FGM_ESPERADO, FG3A_ESPERADO, FG3M_ESPERADO | Valores da base de referência |
| FGA_OBSERVADO, FGM_OBSERVADO, FG3A_OBSERVADO, FG3M_OBSERVADO | Valores calculados nos eventos lidos |
| LINHAS_SALVAS | Número de eventos lidos; ausente quando a leitura não foi concluída |
| STATUS dos eventos | confere, arquivo_ausente, verificar ou erro_leitura_ou_estrutura |
| DETALHE | Problemas encontrados, separados por ponto e vírgula |
| ESPERADOS, ENCONTRADOS | Quantidade de jogadores previstos e quantidade prevista com perfil encontrado |
| FALTANTES, EXTRAS | Jogadores esperados sem perfil e perfis fora da lista esperada |
| STATUS dos perfis / STATUS_PERFIS | confere, verificar ou erro_leitura_ou_estrutura |
| PROBLEMA | Descrição de uma ocorrência de perfil; um jogador pode ter várias |
| ARQUIVO | Caminho do CSV extra encontrado no inventário |
| PARES_ESPERADOS, PARES_APROVADOS | Quantidade planejada e quantidade com status confere |
| ARQUIVOS_AUSENTES | Pares sem arquivo esperado |
| PARES_DIVERGENTES | Pares com status verificar, inclusive inconsistências de estrutura lógica ou totais |
| ERROS_LEITURA_OU_ESTRUTURA | Quantidade de pares com falha capturada durante leitura ou processamento |
| ARQUIVOS_EXTRAS | Número de caminhos encontrados fora do planejamento |
| TEMPORADAS_PERFIS_COM_PENDENCIAS | Temporadas cuja auditoria de perfis não recebeu confere |
| OCORRENCIAS_PENDENCIAS_PERFIS | Número de ocorrências, não necessariamente de jogadores distintos |
| AUDITADO_EM_UTC | Momento de conclusão da auditoria correspondente |
| CONSOLIDADO_EM_UTC | Momento de exportação do conjunto de relatórios |
| RESULTADO | Síntese das verificações executadas; não certifica qualidade absoluta |

As contagens são inteiras não negativas quando conhecidas. Campos não calculados permanecem ausentes, não devem ser substituídos por zero para ocultar falhas.

## 9. Regras de uso e limites

Um evento é aprovado apenas se passar nas verificações implementadas e seus totais concordarem com a referência. Um CSV vazio pode ser coerente quando todos os totais correspondentes são zero. Arquivos ausentes, ilegíveis ou divergentes devem ser investigados; não corrigir os totais apenas para produzir concordância.

As auditorias não demonstram precisão absoluta da fonte, plausibilidade de todas as medidas físicas, qualidade de cada coordenada ou ausência de viés. Identificam problemas definidos pelo código. Ausências da própria fonte podem ser documentadas e tratadas posteriormente na CP2.

## 10. Referências e origem das definições

As descrições operacionais são próprias do projeto e dos campos efetivamente utilizados. O glossário da NBA é referência para a nomenclatura esportiva; não define os campos locais de auditoria.

- [Glossário oficial NBA Stats](https://www.nba.com/stats/help/glossary).
- [Código e documentação do nba_api](https://github.com/swar/nba_api).
- [Esquema LeagueDashPlayerStats](https://github.com/swar/nba_api/blob/master/docs/nba_api/stats/endpoints/leaguedashplayerstats.md).
- [Esquema LeagueDashPlayerBioStats](https://github.com/swar/nba_api/blob/master/docs/nba_api/stats/endpoints/leaguedashplayerbiostats.md).
- [Esquema ShotChartDetail](https://github.com/swar/nba_api/blob/master/docs/nba_api/stats/endpoints/shotchartdetail.md).
- [Repositório do projeto](https://github.com/Fekouto/ciencias_de_dados_nba): cabeçalhos consultados em 23/10/2026.
