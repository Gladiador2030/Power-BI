gistfile1.txt
Crie um Fabric App "Blacksmith DRE" que muestra como una receta vira de lucro, con un Sankey como pieza central (estilo dos gráficos de resultado de Apple/IBM), KPIs, tendencia mensal, realizado × orcado, regiones e indicadores operativos.

FUENTE DE DADOS
- Só o o modelo semântico anexado. Não llore nem altere o modelo; nada de dados embutidos, CSV o mocks. Consulta que falhar muestra estado de error, nunca dados inventados.
- DAX (EVALUATE + SUMMARIZECOLUMNS), consultas en paralelo, consultando el estado de cada respuesta. Usar como medidas del modelo; No recalcule DRE, márgenes o variaciones sin aplicación.
- Sem credenciais no code: só a autenticação nativa do Fabric App.
- Antes de codificar, leia el esquema y confirma las tablas y medidas a continuación. Se faltar algo, pare e me diga.

MODELO
- dim_calendario (data, ano, mes, indice_mes 1..36, nome_mes, mes_ano, ano_mes), dim_regiao (cod_regiao, nome_regiao), dim_conta (cod_conta, nome_conta, cod_grupo, ordem_dre), dim_cenario (cod_cenario "realizado"/"orcado", nome_cenario), dim_no_sankey (cod_no, nome_no, tipo_no receita/total/custo/lucro/prejuizo, coluna_sankey), dim_link_sankey (cod_link, cod_no_origem, cod_no_destino, cod_conta, tipo_link, etapa, ordem_link). Como dos de Sankey no hay relación: sankey_valor es el enlace del contexto.
- Medidas (_medidas): receita_bruta, receita_liquida, beneficio_bruto, ebitda, ebit, lair, beneficio_liquido, valor_dre, margem_liquida, prejuizo_liquido, meses_com_prejuizo; var_receita_liquida_aa_pct, var_lucro_bruto_aa_pct, var_ebitda_aa_pct, var_lucro_liquido_aa_pct, var_margem_liquida_aa_pp; var_receita_liquida_orcado_pct, var_lucro_liquido_orcado_pct; sankey_valor, sankey_valor_aa, var_sankey_valor_aa_pct; clientes_ativos, novos_clientes, churn_medio_pct, nps_medio, headcount, receita_por_cliente, receita_por_colaborador; data_ultima_carga.
- As medidas da DRE respeitam dim_cenario: filtre siempre um único cenário, senão realizado e orçado se somam.

SANKEY (obligatorio)
- Valor de cada enlace = sankey_valor no recorte filtrado. Nunca algunos enlaces pré-calculados por más o región (mês com prejuízo + meses com lucro mostraria os dos ao mesmo tempo).
- Consulta: FILTER ( SUMMARIZECOLUMNS ( dim_link_sankey[ordem_link], [cod_no_origem], [cod_no_destino], [tipo_link], [etapa], <filtros>, "valor", [sankey_valor], "valor_aa", [sankey_valor_aa], "var_aa_pct", [var_sankey_valor_aa_pct] ), NO ES BLANCO ( [valor] ) ) ORDENAR POR ordem_link. Nos zerados somem.
- Diseño propio: columnas por coluna_sankey; cor do nó pelo tipo_no na paleta do tema activo (no use cor_hex). Enlaces en el corazón del destino, opacidad 0,4 (0,85 en foco), sin gradiente ni brillo.
- Prejuízo: o nó prejuizo_liquido (vermelho) entra por baixo na coluna min(colunas dos destinos) − 1; links que saem dele vermelhos e listrados (não depender só da cor).
- Rótulo: nombre + valor abreviado (R$ 175,7 mi) + % da receita líquida. Información sobre herramientas: valor, % de la recepción, variación frente al año anterior.
- Pase el cursor sobre los flujos ligados y esmaece o resto. Clique num nó fixa o destaque e abre Painel de detalhe com a evolução mensal do nó (barras, contorno vermelho nos meses com lucro_liquido < 0, linha roxa tracejada do ano anterior); hacer clic de nueva fecha. Valor do nó = soma de sankey_valor dos links que chegam; nós de receita e prejuizo_liquido usam os que saem.
- Entrada com links "fluindo" da esquerda para a direita (respeitar prefiere movimiento reducido). Altura ~560 píxeles; no celular, rolagem horizontal no contenedor.

DISEÑO (12 columnas sin escritorio; empilha no celular)
1. Cabeçalho: "Blacksmith" (seminegrita) + "DRE" (claro); subtítulo "Do faturamento ao lucro · DRE em Sankey"; última carga ([data_ultima_carga], dd/MM/aaaa HH:mm); botón de tema (sol/lua).
2. Barra de filtros fijos: año (anos do modelo + "Todos"), intervalo de meses, regiones (multi), cenário (Realizado/Orçado, único). Padrão: último año, todas las regiones, Realizado. Mostra a seleção cruzada ("Seleção: Mar/2025, Abr/2025 ✕") y "Restaurar padrão".
3. Seis KPI: receita líquida, beneficio bruto, EBITDA, beneficio líquido, margen líquido, beneficio líquido vs orçado (receita vs orçado na 2ª linha). Cada um: valor, variação vs ano anterior (▲ verde / ▼ laranja; margem em pp), sparkline de 12 meses. Selo "Prejuízo" no lucrativo líquido negativo.
4. Sankey em largura total.
5. Tendência (8 col.) por mes_ano em ordem de indice_mes: receita líquida em área ciano chapada (16%), lucro líquido em linha verde, margem líquida no eixo secundário em linha roxa tracejada. Meses com lucro_liquido < 0: ponto vermelho e rótulo "prejuízo".
6. Regiões (4 col.): barras horizontales de receita líquida, da maior para a menor, margem no rótulo.
7. Tabela DRE (8 col.): contas em ordem_dre com Realizado, Orçado e Var. %, intercaladas con subtotais das medidas (receita bruta e líquida simples; beneficio bruto, EBITDA, EBIT, LAIR e lucro líquido destacados). Mostra os dos cenários e ignora el filtro de cenário.
8. Operacional (4 col.): clientes activos, novos clientes, churn, NPS, headcount, receita por cliente y por colaborador. Só há realizado: ignora el escenario y avisa cuando para Orçado.
- Todo bloco con esqueleto, vazio e error. Subtítulos con el recorte efectivo ("2025 · selección: Mar/2025 · Todas las regiones · Realizado").

FILTRAGEM CRUZADA (como en Power BI)
- Tendencia y detalles del presente: fuente y alvo por dim_calendario[indice_mes]. Regiones: fuente y alvo por dim_regiao[cod_regiao]. Sankey, KPI, DRE y operativos: solo alvo (clique no Sankey só destaca e abre o detalhe).
- camarilla de selección; Ctrl/Shift/⌘ + haga clic para retirar; Haga clic de nuevo, no fundo, no ✕ o Esc limpia.
- O visual de origem não se filtra: esmaece os não selecionados (opacidade 0,3); La tendencia marca los meses seleccionados con faixa de fondo y usa una faixa transparente por más tiempo para recibir camarilla y dica.
- Una selección por vez; Mudar um filtro da barra limpia. A seleção fica dentro dos filtros da barra (fora deles é ignorada e esmaecida).
- Filtros con TREATAS solo con valores vindos das dimensiones (lista blanca), nunca texto libre: TREATAS ( { "realizado" }, dim_cenario[cod_cenario] ), TREATAS ( { 2025 }, dim_calendario[ano] ), TREATAS ( { 3, 4 }, dim_calendario[mes] ), TREATAS ( { "sul" }, dim_regiao[cod_regiao] ), TRATAS ( { 15, 16 }, dim_calendario[indice_mes] ). Visuais por mês não recebem filtro de meses; o de regiones no recibe filtro de regiones.

VISUAL — IBM Carbon
- Escuro (g100) é o padrão; o botón alterna para o claro (g10) y salvar no navegador (try/catch). Tudo troca de cor com o tema, inclusive Sankey e tabela; nenhuma cor de dado fixa num tema só.
- Fichas (claro/escuro): fundo #f4f4f4 / #161616; superfície #ffffff / #262626; superfície 2 #f4f4f4 / #393939; coloque el cursor sobre # e8e8e8 / # 333333; seleccionado #e0e0e0 / #393939; borde #e0e0e0 / #393939; borde fuerte #8d8d8d / #6f6f6f; texto #161616 / #f4f4f4; suave #525252 / #c6c6c6; fraco #6f6f6f / #a8a8a8; interactivo #0f62fe / #4589ff; foco #0f62fe / #ffffff; receita (ciano) #1192e8 / #33b1ff; total, orçado, margem (roxo) #6929c4 / #8a3ffc; custo, queda (laranja) #ba4e00 / #ff832b; lucro, alta (verde) #198038 / #6fdc8c; prejuízo, uso exclusivo (vermelho) #da1e28 / #fa4d56; texto de prejuízo/error #da1e28 / #ff8389.
- IBM Plex Sans (títulos, texto; KPIs en light con tabular-nums) e IBM Plex Mono (tabela, eixos, tooltips), empacotadas (sin Google Fonts). Título del cartón 16 px seminegrita; rótulos 12–14 px, sin caixa alta.
- Superfícies chapadas, cantos retos, borda de 1 px; sin sombra, gradiente, desfoque, vidrio o neón (excepto el brillo del esqueleto). Botão ativo = superfície invertida.
- Animações de 110–240 ms (cubic-bezier(0, 0, 0.38, 0.9)), desligadas con preferencia de movimiento reducido. Contraste AA: texto ≥ 4,5:1, marcas ≥ 3:1.

IDIOMA
- Interfaz en pt-BR: R$ 1.234.567, abreviado R$ 175,7 mi, 11,7%, variación con señal (+3,2%), margen en pp Identificadores en serpiente_case.

VALIDAÇÃO (confira na pré-visualização e me mostre)
- Realizado, todas las regiones: beneficio líquido 2024/2025/2026 = 15.882.255,48 (11,7%) / 13.088.163,15 (8,5%) / 24.560.318,22 (14,0%); receita líquida = 135.461.230,60 / 154.819.681,80 / 175.717.045,70.
- Mar/2025: prejuízo de R$ 2,59 mi; prejuizo_liquido → despesas_operacionais 2.071.798,75, → depreciacao_amortizacao 388.600,04, → resultado_financeiro_liquido 134.368,48; sem os nós ebitda, ebit_lucro_operacional, lair e lucro_liquido.
- Abr/2025: beneficio_bruto → ebitda 20.950,98, prejuízo de R$ 0,61 mi; só Sudeste tem prejuízo, só Sul tem lucro.
- 2025 entero: guarida → lucro_liquido 13.088.163,15, sem nó de prejuízo.
- En 2025, clique em mar/2025 na tendencia mostra o prejuízo nos KPIs y no Sankey; Ctrl+clique em abr/2025 soma; Esc limpia.
- Orçado atualiza todos los visuais (sem prejuízo).
- Diseño correcto en 1440, 1024 y 390 px, nuestros dos temas.
