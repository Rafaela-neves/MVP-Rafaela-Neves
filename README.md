MVP – Engenharia de Dados

Análise do Desempenho Operacional de Entregas de Alimentos

Rafaela Neves da Silva

Pós-graduação em Ciência de Dados

Disciplina: Engenharia de Dados


1. Objetivo
   
1.1 Problema a ser resolvido
Este trabalho tem como propósito entender os principais padrões e fatores associados aos acidentes de trânsito nas rodovias federais brasileiras, de modo a apoiar a identificação de pontos críticos e situações de maior risco, contribuindo para o direcionamento de políticas de segurança viária.

1.2 Perguntas de negócio
	1. Quais são os trechos/BRs com maior número de acidentes e maior gravidade (mortos/feridos)?
	2. Existe relação entre o horário do dia (período: madrugada/amanhecer/dia/noite) e a gravidade dos acidentes?
	3. Quais são as principais causas de acidentes e como elas se relacionam com o número de vítimas?
	4. Há diferença nos padrões de acidentes entre dias de semana e finais de semana?
	5. Existe relação entre a quantidade de veículos envolvidos em um acidente e a sua gravidade (mortos/feridos)?
	6. Existe correlação entre condições da via (pista, traçado, clima) e a gravidade dos acidentes?

Observação: a pergunta 5 original ("quais tipos de veículo estão mais associados a acidentes fatais") foi ajustada durante o desenvolvimento, pois o arquivo "por pessoa/veículo" da PRF não estava disponível para download no momento da coleta (ver Seção 2.2 e Autoavaliação, Seção 7).

2. Fonte de Dados e Coleta
   
2.1 Fonte
Os dados utilizados são públicos e provenientes do Portal de Dados Abertos da Polícia Rodoviária Federal (PRF), gerados pelo sistema BR-Brasil, em operação nacional desde 2007.
●	Portal: https://portal.prf.gov.br/dados-abertos-acidentes
●	Dicionário de variáveis: https://portal.prf.gov.br/dados-abetos-dicionario-acidentes
●	Licença: dado governamental aberto, sujeito à Lei de Acesso à Informação (Lei nº 12.527/2011) e ao Decreto nº 8.777/2016 (Política de Dados Abertos do Executivo Federal).

2.2 Conjunto de dados utilizado
Foram utilizados os arquivos "Acidentes agrupados por ocorrência" (grão: 1 linha = 1 acidente) dos anos de 2023, 2024 e 2025, nomeados datatran2023.csv, datatran2024.csv e datatran2025.csv.
O conjunto "agrupado por pessoa" (que permitiria granularidade de vítima/veículo individual) estava listado no portal, porém sem link de download ativo no momento da coleta. Por esse motivo, o escopo do trabalho foi ajustado para utilizar exclusivamente o grão de acidente, conforme decisão registrada e justificada na Seção 7 (Autoavaliação).

2.3 Coleta
Os 3 arquivos CSV foram baixados manualmente do portal oficial da PRF e carregados via upload direto para um Volume do Unity Catalog no Databricks (prf_acidentes.bronze.raw_files), preservando o arquivo original em sua forma bruta.
Total de registros coletados: 213.451 acidentes.

3. Arquitetura e Pipeline (Arquitetura Medalhão)
O pipeline foi construído na plataforma Databricks Free Edition, utilizando Delta Lake como formato de tabela e Unity Catalog para organização dos dados. Foi adotada a Arquitetura Medalhão (Bronze / Silver / Gold), com um catálogo único (prf_acidentes) e três schemas correspondentes a cada camada.

3.1 Camada Bronze — dado bruto
Os 3 arquivos CSV foram lidos sem inferência de schema (todas as colunas como string) e unidos (union) em uma única tabela, preservando o dado exatamente como recebido da fonte. Foram adicionadas colunas de controle: _ano_referencia, _arquivo_origem e _data_ingestao, para fins de linhagem.
●	Tabela: prf_acidentes.bronze.acidentes_raw
●	Total de linhas: 213.451

3.2 Camada Silver — dado limpo e padronizado
A partir da Bronze, foram aplicadas as seguintes transformações de limpeza e padronização (detalhadas na Seção 5 — Qualidade de Dados):
●	Conversão de tipos: id (long), br/pessoas/mortos/feridos/ilesos/veículos (integer), data_inversa (date).
●	Correção de separador decimal (vírgula → ponto) em km, latitude e longitude, com conversão para double.
●	Extração da hora (inteiro) a partir do campo horario.
●	Padronização de texto (remoção de espaços extras, uf em maiúsculas) nas colunas categóricas.
●	Criação da flag booleana flag_fim_de_semana a partir de dia_semana.
●	Decomposição do atributo multivalorado tracado_via em 12 colunas booleanas (flag_curva, flag_aclive, flag_declive, flag_ponte, flag_viaduto, flag_rotatoria, flag_em_obras, flag_intersecao_vias, flag_desvio_temporario, flag_tunel, flag_retorno_regulamentado, flag_reta).
●	Tabela: prf_acidentes.silver.acidentes_limpo — 213.451 linhas (nenhuma linha perdida no processo).

3.3 Camada Gold — Esquema Estrela
Na camada Gold, os dados foram modelados segundo um Esquema Estrela, composto por uma tabela fato e cinco tabelas de dimensão, detalhado na Seção 4.

4. Modelagem de Dados — Esquema Estrela
A tabela fato_acidente possui grão de 1 linha por acidente e se relaciona com cinco dimensões, cada uma representando um conjunto coeso de atributos descritivos.
Tabela Fato: fato_acidente (213.451 linhas)
Coluna	Tipo	Descrição
id_acidente	long	Identificador único do acidente (chave natural da PRF)
id_tempo, id_local, id_causa, id_condicao, id_classificacao	long	Chaves estrangeiras para as dimensões
pessoas	int	Total de pessoas envolvidas
mortos	int	Total de mortos
feridos_leves / feridos_graves	int	Total de feridos por gravidade
ilesos / ignorados	int	Total de ilesos / estado físico não identificado
veiculos	int	Quantidade de veículos envolvidos
_ano_referencia	int	Ano de referência do arquivo de origem (linhagem)

Dimensão: dim_tempo (152.007 linhas)
Atributos: data_inversa, ano, mes, dia, dia_semana, horario, hora, fase_dia, flag_fim_de_semana.
Dimensão: dim_local (157.683 linhas)
Atributos: uf, br, km, municipio, regional, delegacia, uop, latitude, longitude.
Dimensão: dim_causa (947 linhas)
Atributos: causa_acidente, tipo_acidente.
Dimensão: dim_condicao (4.738 linhas)
Atributos: condicao_metereologica, tipo_pista, uso_solo, sentido_via, e as 12 flags booleanas derivadas de tracado_via (ver Seção 5.3).
Dimensão: dim_classificacao (4 linhas)
Atributo: classificacao_acidente (Sem Vítimas / Com Vítimas Feridas / Com Vítimas Fatais / Ignorado).

4.1 Catálogo de Dados (Metadados)
Item	Descrição
Nome do conjunto	Acidentes em Rodovias Federais Brasileiras — PRF
Versão	v1 — período 2023-2025, atualização anual pela fonte
Descrição	Registros de acidentes de trânsito em rodovias federais, coletados via sistema BR-Brasil pela Polícia Rodoviária Federal
Estrutura	1 tabela fato (fato_acidente) + 5 dimensões (dim_tempo, dim_local, dim_causa, dim_condicao, dim_classificacao)
Chaves identificadoras	id_acidente (fato); id_tempo, id_local, id_causa, id_condicao, id_classificacao (dimensões)
Chaves de ligação	FKs da fato_acidente apontam para as respectivas dimensões (relacionamento N:1)
Granularidade	1 linha da fato = 1 acidente (ocorrência), consolidado por dia/hora/local
Temporalidade	Acidentes ocorridos entre 01/01/2023 e 31/12/2025
Origem	Coleta primária governamental via sistema BR-Brasil (boletim de ocorrência da PRF)
Licença de uso	Dado aberto governamental — Lei nº 12.527/2011 (LAI) e Decreto nº 8.777/2016
Transformações realizadas	Ver Seção 3.2 (Camada Silver) e Seção 5 (Qualidade de Dados)
Comentários	Dataset não contém dados pessoais identificáveis (sem CPF, nome ou outro identificador de indivíduos); portanto, não há incidência direta da LGPD sobre os registros utilizados.

5. Análise de Qualidade de Dados

5.1 Duplicidade e valores nulos
Foi verificada a unicidade da chave id na camada Bronze: 213.451 linhas e 213.451 IDs distintos — nenhuma duplicata encontrada. Também não foram encontrados valores nulos ou vazios em nenhuma das colunas do dataset bruto.

5.2 Problemas de formato identificados e corrigidos
●	Notação científica no campo id (ex.: "6e+05"): corrigido via conversão double → long com try_cast, sem perda de registros.
●	Separador decimal em vírgula nos campos km, latitude e longitude (ex.: "-23,48586772"): corrigido via substituição de vírgula por ponto antes da conversão para double.

5.3 Atributo multivalorado: tracado_via
Identificou-se que o campo tracado_via não representa uma categoria única, mas sim uma concatenação de múltiplas características do trecho da via separadas por ponto e vírgula (ex.: "Aclive;Curva;Ponte"), totalizando 12 valores elementares possíveis. O tratamento inicial (manter como texto único) gerava uma explosão combinatória de 7.438 categorias espúrias na dimensão de condição. A correção aplicada foi a decomposição em 12 colunas booleanas independentes (uma para cada característica), reduzindo a dimensão para 4.738 combinações genuínas e permitindo análises corretas por característica isolada (ex.: "acidentes em curva", independentemente de outras características do trecho).

5.4 Consistência entre pessoas e a soma dos estados físicos
Verificou-se se o campo pessoas corresponde à soma de mortos + feridos_leves + feridos_graves + ilesos + ignorados:
Grupo	Quantidade	% do total	Observação
Consistente (diferença = 0)	201.977	94,64%	Nenhuma ação necessária
Padrão sistemático (diferença = 1 − ignorados)	10.368	4,86%	Sugere que "ignorados" pode incluir um envolvido não classificável como pessoa (ex.: condutor evadido); mantido sem alteração
Divergência irregular, sem padrão	1.106	0,52%	Erro de preenchimento na fonte primária; volume imaterial para as análises agregadas

Decisão de tratamento: os registros não foram alterados ou removidos (preservação da fonte original), e as análises de negócio utilizam diretamente os campos oficiais da PRF (mortos, feridos_leves, feridos_graves), não a soma calculada.

5.5 Validação de domínio: UF e BR
Todas as 27 siglas de UF encontradas são válidas, sem valores nulos ou inconsistentes. Na coluna br, foram identificados 513 registros com valor "0" (não identificado, sempre acompanhados de km = 0.0) e 1 registro com valor "498" (provável erro de digitação, sem correspondência a uma rodovia federal conhecida). Esses 514 registros (0,24% do total) foram excluídos apenas das análises agregadas "por BR" (Pergunta de negócio 1), sendo mantidos intactos nas tabelas.

5.6 Integridade referencial na camada Gold
Após a criação da tabela fato_acidente via junção com as 5 dimensões, foi confirmado que o total de linhas (213.451) e o total de id_acidente distintos (213.451) permanecem idênticos, comprovando que nenhum dos joins gerou duplicação ou perda de registros.

6. Análise e Respostas às Perguntas de Negócio

6.1 Pergunta 1 — BRs com maior número de acidentes e maior gravidade
 
Figura 1 — Top 15 BRs por quantidade de acidentes (2023-2025)
A BR-101 (37.426 acidentes) e a BR-116 (33.303) concentram o maior volume, muito à frente das demais, o que é coerente com sua extensão e alto tráfego. Entretanto, ao calcular o índice de gravidade (mortos por acidente), destacam-se a BR-316 (0,1685), BR-230 (0,1082) e BR-153 (0,0982) — todas com gravidade proporcional muito superior à BR-101 (0,0576), apesar do volume bem menor. Conclui-se que volume e letalidade por ocorrência são fenômenos distintos: BR-101 e BR-116 demandam atenção por escala, enquanto BR-316, BR-230 e BR-153 demandam atenção por risco por evento.
6.2 Pergunta 2 — Horário do dia e gravidade dos acidentes
 
Figura 2 — Gravidade dos acidentes por fase do dia
Existe relação clara entre horário e gravidade. O período "Pleno dia" concentra 55% dos acidentes, mas apresenta a menor gravidade proporcional (0,0594). Já o "Amanhecer" apresenta o maior índice de gravidade (0,1363) — mais que o dobro do período diurno — mesmo com volume bem menor. O período "Plena Noite" também apresenta gravidade elevada (0,116). Esse padrão é coerente com fatores de risco já documentados na literatura de segurança viária, como sonolência, velocidades mais altas (menor tráfego) e menor visibilidade.

6.3 Pergunta 3 — Causas de acidentes e relação com vítimas
 
Figura 3 — Causas mais letais entre as mais frequentes
As causas mais frequentes (reação tardia, ausência de reação, acessar via sem observar) respondem por cerca de 40% dos acidentes, com gravidade moderada (0,058 a 0,073). Já causas menos frequentes apresentam gravidade desproporcionalmente maior: "Transitar na contramão" tem índice de 0,3758 (mais de 6 vezes a média das causas mais comuns), e "Ultrapassagem Indevida" tem 0,2259. Isso sugere que ações de fiscalização voltadas a essas duas causas específicas têm potencial de reduzir mortes de forma desproporcional ao número de ocorrências evitadas.

6.4 Pergunta 4 — Dias de semana vs. Finais de semana
 
Figura 4 — Comparação normalizada por dia: dias de semana vs. fim de semana
Após normalização pelo número de dias em cada categoria (783 dias úteis vs. 313 dias de fim de semana no período), constatou-se que os finais de semana apresentam, em média, 19% mais acidentes por dia (219,73 vs. 184,77) e 31% maior gravidade por acidente (índice 0,0994 vs. 0,076). O padrão é coerente com maior consumo de álcool, viagens de lazer e maior cansaço em deslocamentos de fim de semana.

6.5 Pergunta 5 — Quantidade de veículos e gravidade do acidente
 
Figura 5 — Gravidade em função do número de veículos envolvidos
Observa-se uma relação quase monotônica: quanto maior o número de veículos envolvidos, maior a gravidade do acidente. Acidentes com 1 veículo têm índice de 0,0391, subindo para 0,083 com 2 veículos, e ultrapassando 0,20 a partir de 5 veículos envolvidos — mais de 5 vezes a gravidade de um acidente isolado. Esse resultado reforça a relevância de causas como "condutor deixou de manter distância do veículo da frente" (identificada na Pergunta 3), associada a colisões múltiplas.

6.6 Pergunta 6 — Condições da via e gravidade
Esta pergunta é respondida de forma qualitativa a partir da dimensão dim_condicao, que reúne condição meteorológica, tipo de pista, traçado (flags booleanas) e sentido da via. As flags de traçado (curva, aclive, declive, cruzamento) permitem cruzar a gravidade dos acidentes com características físicas específicas do trecho de forma independente, sem o viés de combinações espúrias que existiria caso o atributo multivalorado original não tivesse sido tratado (ver Seção 5.3). Essa análise está disponível para exploração adicional na tabela dim_condicao e pode ser aprofundada em trabalhos futuros (ver Seção 7).

7. Autoavaliação

7.1 Objetivos atingidos
Das 6 perguntas de negócio originalmente propostas, 5 foram respondidas de forma quantitativa e conclusiva (Perguntas 1 a 5). A Pergunta 6 foi respondida de forma qualitativa/estrutural, já que a análise quantitativa completa (cruzando cada uma das 12 flags de traçado com gravidade) não foi aprofundada por limitação de tempo, mas a estrutura de dados já suporta essa análise.

7.2 Ajuste de escopo
A pergunta de negócio 5 original (relação entre tipo de veículo e vítimas fatais) dependia do conjunto de dados "por pessoa" da PRF, que não estava disponível para download no portal oficial no momento da coleta. Diante disso, avaliou-se a alternativa de usar uma fonte espelho de terceiros, mas optou-se por manter o uso exclusivo da fonte primária oficial, ajustando a pergunta para uma métrica equivalente e disponível no conjunto "por ocorrência" (quantidade de veículos envolvidos), preservando o rigor de linhagem de dados.

7.3 Dificuldades encontradas
●	Inconsistências de formatação típicas de dados governamentais legados (notação científica, separador decimal em vírgula).
●	Identificação e tratamento de um atributo multivalorado (tracado_via) que, se não tratado, distorceria significativamente a dimensão de condição da via.
●	Indisponibilidade temporária de parte da fonte de dados (arquivo "por pessoa"), que exigiu replanejamento do escopo analítico.

7.4 Trabalhos futuros
●	Incorporar o conjunto "por pessoa" assim que disponibilizado pela PRF, permitindo análise por tipo de veículo e perfil demográfico dos envolvidos.
●	Aprofundar a Pergunta 6 com análise quantitativa cruzando cada flag de traçado da via com os índices de gravidade.
●	Enriquecer a dim_local com dados abertos complementares (ex.: extensão duplicada/simples de cada BR via DNIT), permitindo normalizar os rankings por quilometragem.
●	Investigar mais a fundo a divergência sistemática identificada na Seção 5.4 (grupo de 4,86%), possivelmente por meio de consulta ao dicionário de dados histórico da PRF.

8. Referências
●	BRASIL. Polícia Rodoviária Federal. Dados Abertos — Acidentes. Disponível em: https://portal.prf.gov.br/dados-abertos-acidentes.
●	DAMA INTERNATIONAL. DAMA-DMBOK: Data Management Body of Knowledge. 2. ed. Technics Publications, 2017.
●	BARBIERI, C. Governança de dados: práticas, conceitos e novos caminhos. Rio de Janeiro: Alta Books, 2020.
●	Databricks Documentation — Medallion Architecture. Disponível em: https://docs.databricks.com.
●	BRASIL. Lei nº 13.709/2018 (LGPD). Lei nº 12.527/2011 (LAI). Decreto nº 8.777/2016 (Política de Dados Abertos do Executivo Federal).
