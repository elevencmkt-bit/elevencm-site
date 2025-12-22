# Cadastro de Cliente + Briefing de Job → Google Sheets (Web App único)

Um único Apps Script recebe os envios dos formulários `web/documentation/cadastro-de-cliente.html` e `web/documentation/cadastro-briefing.html` e grava tudo na mesma planilha, em abas separadas: `cliente_db_master` e `jobs_db`.

## Como publicar o Web App

1. Abra a planilha que terá as abas `cliente_db_master` e `jobs_db` e vá em **Extensões → Apps Script**.
2. Cole o script abaixo (substitua qualquer código existente).
3. Em **Implantar → Nova implantação → Tipo: Web App**:
   - **Executar como:** você (owner)
   - **Quem tem acesso:** qualquer pessoa com o link
4. Salve, autorize e copie a URL gerada.
5. Atualize o `ENDPOINT_URL` em `web/documentation/cadastro-de-cliente.html` e `web/documentation/cadastro-briefing.html` com essa mesma URL.
6. Reimplante sempre que editar o código.

## Script (um endpoint para os dois formulários)

```javascript
// Web App único para clientes e jobs
const CLIENT_SHEET = "cliente_db_master";
const JOB_SHEET = "jobs_db";

const CLIENT_HEADERS = [
  "cliente_id",
  "nome_fantasia",
  "razao_social",
  "cnpj_ein",
  "segmento",
  "descricao_resumida_negocio",
  "cidade",
  "estado",
  "pais",
  "fuso_horario",
  "idioma_principal",
  "idioma_secundario",
  "responsavel_principal_nome",
  "responsavel_principal_cargo",
  "responsavel_principal_email",
  "responsavel_principal_whatsapp",
  "outros_contatos",
  "publico_principal",
  "publico_secundario",
  "ticket_medio_aproximado",
  "principais_diferenciais",
  "principais_concorrentes",
  "tom_voz_marca",
  "tipo_plano_social",
  "qtd_posts_mes",
  "qtd_reels_mes",
  "qtd_stories_semanais",
  "outros_entregaveis_fixos",
  "observacoes_escopo",
  "tipo_de_servico",
  "gestao_trafego_ativa",
  "plataformas_trafego",
  "orcamento_midia_mensal",
  "links_redes_sociais",
  "link_identidade_visual",
  "link_site",
  "observacoes_gerais",
  "data_cadastro",
  "origem_cadastro",
  "trello_card_id",
  "trello_card_url",
  "drive_client_root_id",
  "drive_jobs_root_id",
  "drive_conteudos_root_id",
  "drive_admin_root_id",
  "trello_board_id",
  "trello_list_inbox_id",
];

const JOB_HEADERS = [
  "job_id",
  "cliente_id",
  "cliente_nome",
  "job_titulo",
  "job_tipo",
  "tipo_demanda_social",
  "canal_midias",
  "prazo_entrega_desejado",
  "prioridade",
  "origem_job",
  "responsavel_job_nome",
  "responsavel_job_email",
  "objetivo_principal",
  "contexto_job",
  "publico_job",
  "links_referencias",
  "sm_rede_principal",
  "sm_formato",
  "sm_qtd_pecas",
  "sm_legenda_completa",
  "sm_tom_voz",
  "sm_cta",
  "sm_camp_objetivo_principal",
  "sm_camp_periodo_inicio",
  "sm_camp_periodo_fim",
  "sm_camp_plataformas",
  "sm_camp_meta_resultado",
  "sm_camp_pagina_destino",
  "sm_camp_segmentacoes",
  "sm_camp_localizacao",
  "sm_camp_orcamento_total",
  "sm_camp_orcamento_diario",
  "sm_camp_observacoes_orcamento",
  "sm_camp_formatos_necessarios",
  "sm_camp_mensagem_principal",
  "sm_camp_ofertas_beneficios",
  "sm_camp_cta_sugerido",
  "sm_camp_entregaveis",
  "sm_camp_observacoes_gerais",
  "sm_camp_referencias",
  "ads_objetivo",
  "ads_plataformas",
  "ads_orcamento",
  "ads_periodo",
  "ads_oferta_principal",
  "ads_publico_segmentacao",
  "video_tipo",
  "video_duracao",
  "video_formato",
  "video_tipo_trabalho",
  "video_materiais_disponiveis",
  "video_local",
  "site_tipo",
  "site_dominio",
  "site_objetivo",
  "site_paginas",
  "site_responsavel_conteudo",
  "site_funcionalidades",
  "site_identidade_visual_link",
  "pack_produto",
  "pack_tipo",
  "pack_dimensoes",
  "pack_info_obrigatoria",
  "pack_restricoes_legais",
  "pack_referencias",
  "brand_situacao_atual",
  "brand_entregaveis",
  "brand_palavras_chave",
  "job_descricao_livre",
  "links_arquivos",
  "observacoes_gerais_job",
  "mes_ano",
  "data_cadastro",
  "origem_cadastro",
  "trello_card_id",
  "trello_card_url",
  "drive_job_folder_id",
  "drive_job_folder_link",
  "trello_pasta_link_ok",
];

function doGet(e) {
  const action = e?.parameter?.action;
  if (action === "clientes") return listClientes();
  return jsonResponse({ success: true, message: "OK" });
}

function doPost(e) {
  try {
    const body = parseBody(e);
    const kind = detectKind(body);

    if (kind === "cliente") {
      return handleCliente(body);
    }
    if (kind === "job") {
      return handleJob(body);
    }

    throw new Error("Tipo de payload não identificado (cliente ou job).");
  } catch (err) {
    console.error(err);
    return jsonResponse({
      success: false,
      message: err.message || "Erro inesperado",
    });
  }
}

// --- Clientes ---
function handleCliente(body) {
  validateCliente(body);
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(CLIENT_SHEET) || ss.insertSheet(CLIENT_SHEET);
  ensureHeaders(sheet, CLIENT_HEADERS);

  const clienteId = `CLI-${Date.now()}`;
  const row = CLIENT_HEADERS.map((key) => mapClienteValue(key, body, clienteId));
  sheet.appendRow(row);

  return jsonResponse({
    success: true,
    cliente_id: clienteId,
    message: "Cliente registrado com sucesso",
  });
}

function mapClienteValue(key, body, clienteId) {
  if (key === "cliente_id") return clienteId;
  if (key === "data_cadastro") return new Date();
  if (key === "origem_cadastro")
    return body[key] ? safeValue(body[key]) : "form_cadastro_cliente";
  if (key === "trello_card_id" || key === "trello_card_url")
    return safeValue(body[key]);
  return safeValue(body[key]);
}

function validateCliente(body) {
  const required = [
    "nome_fantasia",
    "razao_social",
    "cnpj_ein",
    "segmento",
    "descricao_resumida_negocio",
    "cidade",
    "estado",
    "pais",
    "fuso_horario",
    "idioma_principal",
    "responsavel_principal_nome",
    "responsavel_principal_cargo",
    "responsavel_principal_email",
    "responsavel_principal_whatsapp",
    "publico_principal",
    "principais_diferenciais",
    "tom_voz_marca",
    "tipo_plano_social",
    "qtd_posts_mes",
    "qtd_reels_mes",
    "qtd_stories_semanais",
    "tipo_de_servico",
    "gestao_trafego_ativa",
  ];
  const missing = required.filter((field) => !body?.[field]);
  if (missing.length)
    throw new Error(
      `Campos obrigatórios faltando (cliente): ${missing.join(", ")}`
    );
}

// --- Jobs ---
function handleJob(body) {
  validateJob(body);
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(JOB_SHEET) || ss.insertSheet(JOB_SHEET);
  ensureHeaders(sheet, JOB_HEADERS);

  const jobId = `JOB-${Date.now()}`;
  const now = new Date();
  const timezone = ss.getSpreadsheetTimeZone?.() || Session.getScriptTimeZone();
  const mesAno = Utilities.formatDate(now, timezone || "GMT-3", "yyyy-MM");
  const row = JOB_HEADERS.map((key) =>
    mapJobValue(key, body, jobId, now, mesAno)
  );
  sheet.appendRow(row);

  return jsonResponse({
    success: true,
    job_id: jobId,
    message: "Briefing registrado com sucesso",
  });
}

function validateJob(body) {
  const required = [
    "cliente_id",
    "job_titulo",
    "job_tipo",
    "prazo_entrega_desejado",
  ];
  const missing = required.filter((field) => !body?.[field]);
  if (missing.length)
    throw new Error(
      `Campos obrigatórios faltando (job): ${missing.join(", ")}`
    );
}

// --- Utilidades ---
function detectKind(body) {
  if (!body || typeof body !== "object") return "";
  const hasJob = body.job_titulo || body.job_tipo;
  const hasCliente = body.nome_fantasia || body.cnpj_ein || body.segmento;
  if (hasJob) return "job";
  if (hasCliente) return "cliente";
  return "";
}

function ensureHeaders(sheet, headers) {
  const firstRow = sheet.getRange(1, 1, 1, headers.length).getValues()[0];
  const hasHeaders = firstRow.some((cell) => cell);
  if (!hasHeaders) {
    sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
  }
}

function safeValue(value) {
  if (value === null || value === undefined) return "";
  if (typeof value !== "string") return value;
  const trimmed = value.trim();
  // Evita que Sheets interprete como fórmula (+1..., =, -, @, etc.)
  if (/^[=+\-@]/.test(trimmed)) return "'" + trimmed;
  return trimmed;
}

function mapJobValue(key, body, jobId, now, mesAno) {
  if (key === "job_id") return jobId;
  if (key === "data_cadastro") return now || new Date();
  if (key === "mes_ano") return safeValue(body[key]) || mesAno || "";
  if (key === "origem_cadastro")
    return body[key] ? safeValue(body[key]) : "form_cadastro_briefing";
  if (
    key === "trello_card_id" ||
    key === "trello_card_url" ||
    key === "drive_job_folder_id" ||
    key === "drive_job_folder_link" ||
    key === "trello_pasta_link_ok"
  )
    return safeValue(body[key]);
  if (key === "responsavel_job_nome") {
    const raw = typeof body[key] === "string" ? body[key].trim() : "";
    return raw || "Levi";
  }
  return safeValue(body[key]);
}

function listClientes() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(CLIENT_SHEET);
  if (!sheet) return jsonResponse({ success: true, clientes: [] });

  ensureHeaders(sheet, CLIENT_HEADERS);
  const lastRow = sheet.getLastRow();
  if (lastRow < 2) return jsonResponse({ success: true, clientes: [] });

  const values = sheet
    .getRange(2, 1, lastRow - 1, CLIENT_HEADERS.length)
    .getValues();
  const idxId = CLIENT_HEADERS.indexOf("cliente_id");
  const idxNome = CLIENT_HEADERS.indexOf("nome_fantasia");
  const clientes = values
    .map((row) => ({
      cliente_id: safeValue(row[idxId]),
      nome_fantasia: safeValue(row[idxNome]),
    }))
    .filter((c) => c.cliente_id);

  return jsonResponse({ success: true, clientes });
}

function parseBody(e) {
  if (e?.postData?.contents) {
    try {
      return JSON.parse(e.postData.contents);
    } catch (err) {
      throw new Error("Payload inválido (JSON esperado).");
    }
  }
  return {};
}

function jsonResponse(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(
    ContentService.MimeType.JSON
  );
}
```

## Abas e colunas (linha 1)

- `cliente_db_master`

  ```
  cliente_id,nome_fantasia,razao_social,cnpj_ein,segmento,descricao_resumida_negocio,cidade,estado,pais,fuso_horario,idioma_principal,idioma_secundario,responsavel_principal_nome,responsavel_principal_cargo,responsavel_principal_email,responsavel_principal_whatsapp,outros_contatos,publico_principal,publico_secundario,ticket_medio_aproximado,principais_diferenciais,principais_concorrentes,tom_voz_marca,tipo_plano_social,qtd_posts_mes,qtd_reels_mes,qtd_stories_semanais,outros_entregaveis_fixos,observacoes_escopo,tipo_de_servico,gestao_trafego_ativa,plataformas_trafego,orcamento_midia_mensal,links_redes_sociais,link_identidade_visual,link_site,observacoes_gerais,data_cadastro,origem_cadastro,trello_card_id,trello_card_url,drive_client_root_id,drive_jobs_root_id,drive_conteudos_root_id,drive_admin_root_id,trello_board_id,trello_list_inbox_id
  ```

- `jobs_db`
  ```
  job_id,cliente_id,cliente_nome,job_titulo,job_tipo,tipo_demanda_social,canal_midias,prazo_entrega_desejado,prioridade,origem_job,responsavel_job_nome,responsavel_job_email,objetivo_principal,contexto_job,publico_job,links_referencias,sm_rede_principal,sm_formato,sm_qtd_pecas,sm_legenda_completa,sm_tom_voz,sm_cta,sm_camp_objetivo_principal,sm_camp_periodo_inicio,sm_camp_periodo_fim,sm_camp_plataformas,sm_camp_meta_resultado,sm_camp_pagina_destino,sm_camp_segmentacoes,sm_camp_localizacao,sm_camp_orcamento_total,sm_camp_orcamento_diario,sm_camp_observacoes_orcamento,sm_camp_formatos_necessarios,sm_camp_mensagem_principal,sm_camp_ofertas_beneficios,sm_camp_cta_sugerido,sm_camp_entregaveis,sm_camp_observacoes_gerais,sm_camp_referencias,ads_objetivo,ads_plataformas,ads_orcamento,ads_periodo,ads_oferta_principal,ads_publico_segmentacao,video_tipo,video_duracao,video_formato,video_tipo_trabalho,video_materiais_disponiveis,video_local,site_tipo,site_dominio,site_objetivo,site_paginas,site_responsavel_conteudo,site_funcionalidades,site_identidade_visual_link,pack_produto,pack_tipo,pack_dimensoes,pack_info_obrigatoria,pack_restricoes_legais,pack_referencias,brand_situacao_atual,brand_entregaveis,brand_palavras_chave,job_descricao_livre,links_arquivos,observacoes_gerais_job,mes_ano,data_cadastro,origem_cadastro,trello_card_id,trello_card_url,drive_job_folder_id,drive_job_folder_link,trello_pasta_link_ok
  ```

## Observações rápidas

- Endpoint único para os dois formulários; detecção é automática pelos campos recebidos.
- `tom_voz_marca` agora é obrigatório no cadastro de cliente; crie a coluna na aba se a planilha já existir.
- A listagem de clientes (GET `?action=clientes`) retorna `cliente_id` e `nome_fantasia` para popular o dropdown do briefing; não gera novos IDs.
- Campos de checkbox chegam como string separada por vírgula (conforme os HTMLs).
- `data_cadastro`, `mes_ano`, `origem_cadastro`, campos de Trello/Drive são preenchidos pelo script/integrações ou por automações posteriores.
- Reimplante o Web App sempre que editar o código.

## (Opcional) Normalizar cliente_id nos jobs antigos

Cole e execute manualmente no Apps Script para ajustar a aba `jobs_db` com base no `nome_fantasia` da `cliente_db_master`:

```javascript
function normalizarClienteIdsPorNome() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const clients = buildClienteMapByNome(ss);
  const sheetJobs = ss.getSheetByName(JOB_SHEET);
  if (!sheetJobs) return;
  const lastRow = sheetJobs.getLastRow();
  if (lastRow < 2) return;

  const values = sheetJobs
    .getRange(2, 1, lastRow - 1, JOB_HEADERS.length)
    .getValues();
  const idxClienteId = JOB_HEADERS.indexOf("cliente_id") + 1;
  const idxClienteNome = JOB_HEADERS.indexOf("cliente_nome") + 1;

  let updated = 0;
  values.forEach((row, i) => {
    const nome = safeValue(row[idxClienteNome - 1]);
    const idAtual = safeValue(row[idxClienteId - 1]);
    const idCorreto = clients[nome];
    if (nome && idCorreto && idAtual !== idCorreto) {
      sheetJobs.getRange(i + 2, idxClienteId).setValue(idCorreto);
      updated++;
    }
  });
  Logger.log(`Atualizadas ${updated} linhas em jobs_db.`);
}

function buildClienteMapByNome(ss) {
  const sheetCli = ss.getSheetByName(CLIENT_SHEET);
  if (!sheetCli) return {};
  const lastRow = sheetCli.getLastRow();
  if (lastRow < 2) return {};
  const values = sheetCli
    .getRange(2, 1, lastRow - 1, CLIENT_HEADERS.length)
    .getValues();
  const idxId = CLIENT_HEADERS.indexOf("cliente_id");
  const idxNome = CLIENT_HEADERS.indexOf("nome_fantasia");
  return values.reduce((map, row) => {
    const nome = safeValue(row[idxNome]);
    const id = safeValue(row[idxId]);
    if (nome && id) map[nome] = id;
    return map;
  }, {});
}
```
