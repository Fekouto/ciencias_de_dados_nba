# NBA — coleta e validação de dados | IMD1151

Projeto de Ciência de Dados da disciplina IMD1151, 2026.2. O objetivo é preparar dados para investigar relações entre características físicas dos jogadores e seus padrões de arremesso.

## Escopo desta entrega

CP1: coleta e validação. Recorte: dez temporadas regulares, de 2015–16 a 2024–25. Playoffs e pré-temporada não fazem parte das consultas. Limpeza analítica, visualizações e respostas às perguntas de pesquisa serão desenvolvidas na CP2.

O projeto utiliza a alternativa de uma fonte complexa via API prevista no enunciado. Os três endpoints pertencem à mesma fonte, NBA Stats; o pacote comunitário `nba_api` é o meio de acesso, não uma segunda fonte.

Esta documentação descreve a versão revisada do notebook organizada em nove módulos. O nome sugerido para esse notebook é `CP1_coleta_revisada.ipynb`; caso tenha sido salvo com outro nome, abra o arquivo correspondente. Não confundir com versões anteriores de experimentação.

## Dados e finalidade

| Conjunto | Endpoint | Unidade de observação | Finalidade |
|---|---|---|---|
| Estatísticas | LeagueDashPlayerStats | Jogador–temporada | Definir os pares a coletar e fornecer totais de referência |
| Arremessos | ShotChartDetail | Tentativa de arremesso de quadra | Registrar localização, tipo e resultado de cada tentativa |
| Perfis físicos | LeagueDashPlayerBioStats | Jogador–temporada | Obter altura, peso e atributos cadastrais |

O notebook preserva as colunas recebidas e adiciona metadados. Não aplica imputação nem remoção de outliers. As tabelas são relacionáveis por `PLAYER_ID` e `SEASON`; a integração analítica será realizada posteriormente. O [dicionário de dados](DICIONARIO_DADOS.md) explica campos, unidades e regras.

## Organização dos arquivos

```text
CP1_coleta_revisada.ipynb
README.md
DICIONARIO_DADOS.md
requirements.txt
dados/
  jogadores_por_temporada/<temporada>.csv
  arremessos/<temporada>/<PLAYER_ID>.csv
  arremessos/log_<data_hora_UTC>.csv
  perfis_por_temporada/<temporada>.csv
  auditorias/<data_hora_UTC>/
```

Os caminhos são relativos à pasta de trabalho do notebook. Para uma coleta independente, execute em outra pasta ou altere as quatro pastas de dados no módulo 1. Apenas criar outro notebook na mesma pasta não separa os dados.

## Ambiente e execução

É necessário Python 3, acesso à internet para novas consultas e um editor de notebooks, como VS Code com as extensões Python e Jupyter da Microsoft.

No terminal PowerShell, aberto na pasta do projeto:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Abra o notebook e selecione `.venv` como kernel. Confira a pasta de trabalho com `Path.cwd()` e execute as células de cima para baixo. Não é necessário ativar o ambiente no terminal ao usar diretamente o executável acima.

As dependências fornecidas não têm versões fixadas: ainda não foi registrado um ambiente testado para esta versão. Depois de executar e validar o notebook no ambiente dedicado, registre as versões utilizadas:

```powershell
.\.venv\Scripts\python.exe -m pip freeze > requirements-lock.txt
```

## Módulos

1. Preparação do ambiente, temporadas e pastas.
2. Coleta ou leitura das estatísticas sazonais; conferência de chaves e totais.
3. Planejamento dos pares jogador–temporada.
4. Coleta incremental dos arremessos, novas tentativas e logs.
5. Coleta dos perfis físicos por temporada.
6. Inventário dos arquivos de eventos e identificação de caminhos extras.
7. Auditoria dos arremessos e dos arquivos vazios.
8. Auditoria da cobertura e consistência básica dos perfis.
9. Consolidação e exportação das evidências.

## Retomada e falhas

Arquivos existentes são reutilizados. Isso evita repetir consultas, mas não comprova validade: as auditorias devem ser executadas também para arquivos antigos. Os módulos de coleta tentam novamente após falhas de requisição ou interpretação da resposta. A coleta de eventos pausa por 15 minutos após três pares consecutivos com falhas.

Os dados novos são gravados em arquivo temporário antes da substituição pelo CSV final. O log de eventos é atualizado durante a execução. Nem todas as falhas possíveis, como problemas de disco ou mudanças inesperadas de esquema, são recuperáveis automaticamente.

Os logs registram novas consultas; arquivos ignorados por já existirem não ganham registros de sucesso nessa execução. Se não houver consultas de eventos, não será criado um novo log.

## Validações e interpretação dos resultados

- Estatísticas de referência: chaves presentes e únicas, temporada correta e totais numéricos, inteiros, não negativos e coerentes entre si.
- Eventos: arquivo esperado, colunas necessárias, jogador e temporada corretos, identificadores presentes, ausência de eventos duplicados dentro de cada arquivo e valores válidos de flags e tipo de arremesso.
- Reconciliação: linhas versus FGA, acertos versus FGM, tentativas de três versus FG3A e acertos de três versus FG3M.
- Arquivos vazios: aprovação depende da concordância dos totais com a referência, inclusive FGA igual a zero.
- Perfis: cobertura de jogadores, identificadores, temporada, duplicatas e altura/peso numéricos, finitos e positivos.

As auditorias registram problemas sem alterar os dados originais. Após recuperar ou modificar arquivos, execute novamente os módulos 6 a 9. O resumo final não refaz as auditorias e não detecta sozinho alterações posteriores nos arquivos.

Resultados do módulo 9:

| Arquivo | Conteúdo |
|---|---|
| resumo_geral.csv | Contagens consolidadas e conclusão limitada às verificações executadas |
| resumo_por_temporada.csv | Situação dos eventos e perfis por temporada |
| auditoria_arremessos.csv | Resultado para cada par esperado |
| pendencias_arremessos.csv | Pares não aprovados |
| auditoria_perfis.csv | Cobertura e situação dos perfis por temporada |
| pendencias_perfis.csv | Ocorrências de problemas, podendo haver várias para um jogador |
| arquivos_extras.csv | CSVs de eventos encontrados fora dos caminhos previstos |

`AUDITADO_EM_UTC` registra o término da auditoria correspondente; `CONSOLIDADO_EM_UTC`, o momento da exportação. Não são datas de coleta dos eventos.

## Limitações e estado da validação

Esta documentação não atesta execução bem-sucedida nem ausência de pendências. A evidência deve ser consultada nas saídas do notebook e nos relatórios gerados pela execução do grupo. Não foram fixadas aqui quantidades de jogadores ou eventos de execuções anteriores.

A concordância entre endpoints é um controle de consistência dentro da NBA Stats, não uma validação por fonte independente. Alterações da API, bloqueios, timeouts e revisões dos dados podem afetar novas execuções.

O parâmetro de temporada dos perfis não garante, por si só, que cada medida física represente a medição exata naquele ano. Valores positivos também podem ser implausíveis. Esses limites e a qualidade de distâncias e coordenadas devem ser investigados na CP2. Os perfis coletados não incluem necessariamente posição em quadra; não pressupor essa coluna.

O CSV não preserva tipos: leia `GAME_ID` como texto para manter zeros iniciais. Não interprete ausência como zero sem justificativa.

## Uso de IA e autoria

Foi utilizado ChatGPT para apoiar a revisão da estrutura, a elaboração dos módulos e a documentação. O grupo deve registrar no relatório as ferramentas efetivamente usadas, as etapas e as verificações realizadas, além das contribuições individuais. A revisão e a execução dos resultados permanecem sob responsabilidade dos integrantes.

## Referências

- [NBA Stats](https://www.nba.com/stats/).
- [Glossário oficial da NBA](https://www.nba.com/stats/help/glossary).
- [nba_api — código e documentação](https://github.com/swar/nba_api).
- [LeagueDashPlayerStats](https://github.com/swar/nba_api/blob/master/docs/nba_api/stats/endpoints/leaguedashplayerstats.md).
- [LeagueDashPlayerBioStats](https://github.com/swar/nba_api/blob/master/docs/nba_api/stats/endpoints/leaguedashplayerbiostats.md).
- [ShotChartDetail](https://github.com/swar/nba_api/blob/master/docs/nba_api/stats/endpoints/shotchartdetail.md).
- Enunciado do projeto da segunda unidade de IMD1151, 2026.2, páginas 2–3: reprodutibilidade e checkpoints.

Documentação preparada em 23/10/2026.
