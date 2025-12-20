# Web App único (clientes + briefings + roadmap)

Um único Apps Script atende:

- Formulário de cliente: `web/cadastro-de-cliente.html` → aba `cliente_db_master` (ou `clientes_db_master`).
- Formulário de briefing: `web/cadastro-briefing.html` → aba `jobs_db`.
- Log público de jobs: `?action=jobs`.
- Roadmap interno/cliente (`admin_roadmap.html` / `roadmap_public.html`): abas `roadpmaps` e `roadpmaps_posts`.

## Passo a passo

1. Abra a planilha e vá em **Extensões → Apps Script**.
2. Substitua todo o código pelo script abaixo.
3. Implante: **Implantar → Nova implantação → Web App** → Executar como: você · Acesso: qualquer pessoa com o link.
4. Copie a URL e configure nos HTMLs (o log e as novas páginas já usam a mesma URL).
5. Reimplante sempre que editar o script.

## Código único (cole tudo)

```javascript
// Web App único: clientes, jobs, roadmaps
const CLIENT_SHEET = "cliente_db_master";
const CLIENT_SHEET_ALT = "clientes_db_master"; // fallback
const JOB_SHEET = "jobs_db";
const ROADMAP_SHEET = "roadpmaps";
const ROADMAP_POST_SHEET = "roadpmaps_posts";

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
  // opcionais
  "slug",
  "logo_url",
  "default_notes_client",
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

const JOB_FIELDS_PUBLIC = [
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
  "drive_job_folder_link",
  "trello_pasta_link_ok",
];

const ROADMAP_HEADERS = [
  "id",
  "client_id",
  "month",
  "title",
  "notes_client",
  "notes_internal",
  "created_at",
  "updated_at",
];

const ROADMAP_POST_HEADERS = [
  "id",
  "roadmap_id",
  "date",
  "channel",
  "caption",
  "drive_asset_url",
  "status",
  "client_comment",
  "order",
  "created_at",
  "updated_at",
];

// Pasta raiz no Drive para armazenar assets dos roadmaps
const ROADMAP_ASSETS_ROOT_ID = "1pjnkpdJfD6lQLMUMBrrAq9y0Q6p_9mAy"; // roadmaps_assets

function doGet(e) {
  const action = e?.parameter?.action;
  if (action === "clientes" || action === "clients") return listClientes();
  if (action === "jobs") return listJobs(e?.parameter);
  if (action === "roadmap") {
    const clientSlug = (e.parameter.client_slug || "").trim();
    const month = (e.parameter.month || "").trim();
    return jsonResponse(getOrCreateRoadmap(clientSlug, month));
  }
  return jsonResponse({ success: true, message: "OK" });
}

function doPost(e) {
  try {
    const body = parseBody(e);
    const action = (body.action || "").trim();

    if (action === "roadmap_save") return jsonResponse(saveRoadmap(body));
    if (action === "roadmap_feedback") return jsonResponse(saveFeedback(body));
    if (action === "roadmap_upload_asset") return jsonResponse(uploadRoadmapAsset(body));

    const kind = detectKind(body);
    if (kind === "cliente") return handleCliente(body);
    if (kind === "job") return handleJob(body);

    throw new Error(
      "Tipo de payload não identificado (cliente, job ou roadmap)."
    );
  } catch (err) {
    console.error(err);
    return jsonResponse({
      success: false,
      message: err.message || "Erro inesperado",
    });
  }
}

// Clientes
function handleCliente(body) {
  validateCliente(body);
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = getClientSheet(ss);
  ensureHeaders(sheet, CLIENT_HEADERS);

  const clienteId = `CLI-${Date.now()}`;
  const row = CLIENT_HEADERS.map((key) =>
    mapClienteValue(key, body, clienteId)
  );
  sheet.appendRow(row);

  return jsonResponse({
    success: true,
    cliente_id: clienteId,
    message: "Cliente registrado com sucesso",
  });
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

// Jobs
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

// Roadmap
function getOrCreateRoadmap(clientSlug, month) {
  if (!clientSlug || !month)
    throw new Error("client_slug e month são obrigatórios.");
  const normalizedMonth = normalizeMonth(month);
  const client = findClientBySlug(clientSlug);
  if (!client) throw new Error("Cliente não encontrado.");

  let { headers: rh, rows: rr, sheet: rSheet } = readTable(ROADMAP_SHEET, ROADMAP_HEADERS);
  const { headers: ph, rows: pr } = readTable(ROADMAP_POST_SHEET, ROADMAP_POST_HEADERS);

  const candidates = rr.filter(
    (r) => String(r.client_id) == String(client.id) && normalizeMonth(r.month) == normalizedMonth
  );

  let roadmap = candidates.length ? candidates[0] : null;
  if (!roadmap) {
    roadmap = {
      id: Utilities.getUuid(),
      client_id: client.id,
      month: normalizedMonth,
      title: `Roadmap Social – ${month}`,
      notes_client: client.default_notes_client || "",
      notes_internal: "",
      created_at: new Date().toISOString(),
      updated_at: new Date().toISOString(),
    };
    ensureHeaders(rSheet, ROADMAP_HEADERS);
    appendRow(rSheet, ROADMAP_HEADERS, roadmap);
    ({ headers: rh, rows: rr, sheet: rSheet } = readTable(ROADMAP_SHEET, ROADMAP_HEADERS));
  }

  const candidateIds = (candidates.length ? candidates : [roadmap]).map((r) => r.id);
  const posts = pr.filter((p) => candidateIds.includes(p.roadmap_id));
  return { client, roadmap, posts };
}

function saveRoadmap(body) {
  const { client_slug, month } = body;
  if (!client_slug || !month)
    throw new Error("client_slug e month são obrigatórios.");
  const normalizedMonth = normalizeMonth(month);
  const client = findClientBySlug(client_slug);
  if (!client) throw new Error("Cliente não encontrado.");

  let { headers: rh, rows: rr, sheet: rSheet } = readTable(ROADMAP_SHEET, ROADMAP_HEADERS);
  let {
    headers: ph,
    rows: pr,
    sheet: pSheet,
  } = readTable(ROADMAP_POST_SHEET, ROADMAP_POST_HEADERS);

  const candidates = rr.filter(
    (r) => String(r.client_id) == String(client.id) && normalizeMonth(r.month) == normalizedMonth
  );
  let roadmap = candidates.length ? candidates[0] : null;
  if (!roadmap) {
    roadmap = {
      id: Utilities.getUuid(),
      client_id: client.id,
      month: normalizedMonth,
      created_at: new Date().toISOString(),
    };
    ensureHeaders(rSheet, ROADMAP_HEADERS);
    appendRow(rSheet, ROADMAP_HEADERS, roadmap);
    ({ headers: rh, rows: rr, sheet: rSheet } = readTable(ROADMAP_SHEET, ROADMAP_HEADERS));
  }

  roadmap.month = normalizedMonth;
  roadmap.title = body.title || roadmap.title || `Roadmap Social – ${normalizedMonth}`;
  roadmap.notes_client = body.notes_client ?? roadmap.notes_client ?? "";
  roadmap.notes_internal = body.notes_internal ?? roadmap.notes_internal ?? "";
  roadmap.updated_at = new Date().toISOString();

  const rIdx = rr.findIndex((r) => r.id == roadmap.id);
  if (rIdx >= 0) updateRow(rSheet, rIdx + 2, rh, roadmap);

  const toDelete = Array.isArray(body.posts_to_delete)
    ? body.posts_to_delete
    : [];
  let postsRows = pr;
  if (toDelete.length) {
    postsRows = pr.filter(
      (p) => !(p.roadmap_id == roadmap.id && toDelete.includes(p.id))
    );
    pSheet
      .getRange(2, 1, pSheet.getLastRow() - 1, pSheet.getLastColumn())
      .clearContent();
    ensureHeaders(pSheet, ROADMAP_POST_HEADERS);
    postsRows.forEach((post) => appendRow(pSheet, ROADMAP_POST_HEADERS, post));
  }

  const payloadPosts = Array.isArray(body.posts) ? body.posts : [];
  payloadPosts.forEach((post) => {
    const now = new Date().toISOString();
    if (post.id) {
      const idx = postsRows.findIndex((p) => p.id == post.id);
      if (idx >= 0) {
        const merged = {
          ...postsRows[idx],
          ...post,
          roadmap_id: roadmap.id,
          updated_at: now,
        };
        updateRow(pSheet, idx + 2, ROADMAP_POST_HEADERS, merged);
        postsRows[idx] = merged;
      }
    } else {
      const newPost = {
        ...post,
        id: Utilities.getUuid(),
        roadmap_id: roadmap.id,
        status: post.status || "sent_to_client",
        created_at: now,
        updated_at: now,
      };
      appendRow(pSheet, ROADMAP_POST_HEADERS, newPost);
      postsRows.push(newPost);
    }
  });

  return getOrCreateRoadmap(client_slug, month);
}

function saveFeedback(body) {
  const { roadmap_id, post_id, status, client_comment = "" } = body;
  if (!roadmap_id || !post_id)
    throw new Error("roadmap_id e post_id são obrigatórios.");
  if (status === "changes_requested" && !String(client_comment || "").trim()) {
    throw new Error("Comentário obrigatório ao pedir ajustes.");
  }
  const {
    headers: ph,
    rows: pr,
    sheet: pSheet,
  } = readTable(ROADMAP_POST_SHEET, ROADMAP_POST_HEADERS);
  const idx = pr.findIndex(
    (p) => p.id == post_id && p.roadmap_id == roadmap_id
  );
  if (idx === -1) throw new Error("Post não encontrado.");
  const updated = {
    ...pr[idx],
    status: status || pr[idx].status || "sent_to_client",
    client_comment,
    updated_at: new Date().toISOString(),
  };
  updateRow(pSheet, idx + 2, ph, updated);
  return { success: true };
}

// Upload de assets para Drive (cria pasta /client_slug/AAAA-MM)
function uploadRoadmapAsset(body) {
  const { client_slug, month, filename, mimeType, base64 } = body || {};
  if (!client_slug || !month || !filename || !base64) {
    throw new Error("client_slug, month, filename e base64 são obrigatórios.");
  }
  const normalizedMonth = normalizeMonth(month);
  const client = findClientBySlug(client_slug);
  if (!client) throw new Error("Cliente não encontrado.");

  const folder = getOrCreateAssetFolder(client_slug, normalizedMonth);
  const blob = Utilities.newBlob(Utilities.base64Decode(base64), mimeType || "application/octet-stream", filename);
  const file = folder.createFile(blob);
  file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
  const url = `https://drive.google.com/uc?export=view&id=${file.getId()}`;
  return { success: true, url, fileId: file.getId() };
}

function getOrCreateAssetFolder(clientSlug, month) {
  if (!ROADMAP_ASSETS_ROOT_ID) throw new Error("ROADMAP_ASSETS_ROOT_ID não configurado.");
  const root = DriveApp.getFolderById(ROADMAP_ASSETS_ROOT_ID);
  const clientName = slugify(clientSlug);
  let clientFolder = findOrCreateSubfolder(root, clientName);
  let monthFolder = findOrCreateSubfolder(clientFolder, month);
  return monthFolder;
}

function findOrCreateSubfolder(parent, name) {
  const it = parent.getFoldersByName(name);
  if (it.hasNext()) return it.next();
  return parent.createFolder(name);
}

// Trigger opcional de limpeza: remove arquivos com mais de 30 dias
function cleanupOldAssets() {
  if (!ROADMAP_ASSETS_ROOT_ID) return;
  const root = DriveApp.getFolderById(ROADMAP_ASSETS_ROOT_ID);
  const now = new Date();
  const cutoff = new Date(now.getTime() - 30 * 24 * 60 * 60 * 1000);
  const folders = root.getFolders();
  while (folders.hasNext()) {
    const clientFolder = folders.next();
    const months = clientFolder.getFolders();
    while (months.hasNext()) {
      const monthFolder = months.next();
      const files = monthFolder.getFiles();
      while (files.hasNext()) {
        const file = files.next();
        const created = file.getDateCreated();
        if (created < cutoff) {
          try {
            file.setTrashed(true);
          } catch (err) {
            // ignora erro de permissão
          }
        }
      }
    }
  }
}

// Utilidades
function detectKind(body) {
  if (!body || typeof body !== "object") return "";
  const hasJob = body.job_titulo || body.job_tipo;
  const hasCliente = body.nome_fantasia || body.cnpj_ein || body.segmento;
  if (hasJob) return "job";
  if (hasCliente) return "cliente";
  return "";
}

function getClientSheet(ss) {
  return (
    ss.getSheetByName(CLIENT_SHEET) ||
    ss.getSheetByName(CLIENT_SHEET_ALT) ||
    ss.insertSheet(CLIENT_SHEET)
  );
}

function ensureHeaders(sheet, headers) {
  const existing = sheet.getRange(1, 1, 1, headers.length).getValues()[0];
  const hasAny = existing.some((cell) => cell);
  const sameLength = sheet.getLastColumn() === headers.length;
  const matches =
    hasAny &&
    sameLength &&
    existing.every((cell, idx) => String(cell || "").trim() === String(headers[idx] || "").trim());
  if (!matches) {
    sheet.getRange(1, 1, 1, headers.length).setValues([headers]);
  }
}

function safeValue(value) {
  if (value === null || value === undefined) return "";
  if (typeof value !== "string") return value;
  const trimmed = value.trim();
  if (/^[=+\-@]/.test(trimmed)) return "'" + trimmed;
  return trimmed;
}

function normalizeId(value) {
  if (value === null || value === undefined) return "";
  return String(value).trim().toLowerCase();
}

function toTimestamp(value) {
  if (value instanceof Date) return value.getTime();
  const parsed = Date.parse(value);
  return isNaN(parsed) ? 0 : parsed;
}

function normalizeMonth(value) {
  if (value instanceof Date) {
    const y = value.getFullYear();
    const m = String(value.getMonth() + 1).padStart(2, "0");
    return `${y}-${m}`;
  }
  const s = String(value || "").trim();
  const match = s.match(/^(\d{4})[-/](\d{2})/);
  if (match) return `${match[1]}-${match[2]}`;
  // se vier em formato ISO com dia, corta os 7 primeiros
  if (/^\d{4}-\d{2}-\d{2}/.test(s)) return s.slice(0, 7);
  return s;
}

function mapClienteValue(key, body, clienteId) {
  if (key === "cliente_id") return clienteId;
  if (key === "data_cadastro") return new Date();
  if (key === "origem_cadastro")
    return body[key] ? safeValue(body[key]) : "form_cadastro_cliente";
  return safeValue(body[key]);
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
  const sheet = getClientSheet(SpreadsheetApp.getActiveSpreadsheet());
  if (!sheet) return jsonResponse({ success: true, clients: [] });

  ensureHeaders(sheet, CLIENT_HEADERS);
  const lastRow = sheet.getLastRow();
  if (lastRow < 2) return jsonResponse({ success: true, clients: [] });

  const values = sheet
    .getRange(2, 1, lastRow - 1, CLIENT_HEADERS.length)
    .getValues();
  const idx = CLIENT_HEADERS.reduce((acc, key, i) => {
    acc[key] = i;
    return acc;
  }, {});

  const clients = values
    .map((row) => {
      const name = safeValue(row[idx["nome_fantasia"]]);
      const slug = safeValue(row[idx["slug"]]) || slugify(name);
      return {
        id: safeValue(row[idx["cliente_id"]]),
        name,
        slug,
        logo_url: safeValue(row[idx["logo_url"]]),
        default_notes_client: safeValue(row[idx["default_notes_client"]]),
        created_at: safeValue(row[idx["data_cadastro"]]),
        updated_at: safeValue(row[idx["data_cadastro"]]),
      };
    })
    .filter((c) => c.id);

  return jsonResponse({ success: true, clients });
}

function findClientBySlug(slug) {
  const resp = listClientes().getContent();
  const parsed = JSON.parse(resp);
  const clients = parsed.clients || [];
  return clients.find(
    (c) => c.slug === slug || slugify(c.name) === slug || c.id === slug
  );
}

function listJobs(params) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(JOB_SHEET);
  if (!sheet) return jsonResponse({ success: true, jobs: [] });

  ensureHeaders(sheet, JOB_HEADERS);
  const lastRow = sheet.getLastRow();
  if (lastRow < 2) return jsonResponse({ success: true, jobs: [] });

  const values = sheet
    .getRange(2, 1, lastRow - 1, JOB_HEADERS.length)
    .getValues();
  const idx = JOB_HEADERS.reduce((acc, key, index) => {
    acc[key] = index;
    return acc;
  }, {});

  let jobs = values.map((row) => {
    const job = {};
    JOB_FIELDS_PUBLIC.forEach((field) => {
      job[field] = safeValue(row[idx[field]]);
    });
    return job;
  });

  const rawCliente = params?.cliente_id || params?.cliente || "";
  if (rawCliente) {
    const target = normalizeId(rawCliente);
    jobs = jobs.filter((job) => normalizeId(job.cliente_id) === target);
  }

  jobs.sort(
    (a, b) => toTimestamp(b.data_cadastro) - toTimestamp(a.data_cadastro)
  );

  const limitParam = parseInt(params?.limit, 10);
  const limit = isNaN(limitParam)
    ? 150
    : Math.max(1, Math.min(limitParam, 500));
  if (jobs.length > limit) jobs = jobs.slice(0, limit);

  return jsonResponse({ success: true, jobs });
}

function readTable(name, headersToEnsure) {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(name);
  if (!sheet) throw new Error(`Aba não encontrada: ${name}`);
  if (headersToEnsure && headersToEnsure.length) ensureHeaders(sheet, headersToEnsure);
  const values = sheet.getDataRange().getValues();
  if (!values.length || values[0].every((c) => c === "")) {
    return { headers: headersToEnsure || [], rows: [], sheet };
  }
  const headers = headersToEnsure?.length ? headersToEnsure : values[0].map(String);
  const rows = values.slice(1).map((r) => {
    const obj = {};
    headers.forEach((h, i) => (obj[h] = r[i]));
    return obj;
  });
  return { headers, rows, sheet };
}

function appendRow(sheet, headers, obj) {
  const row = headers.map((h) => obj[h] ?? "");
  sheet.appendRow(row);
}

function updateRow(sheet, rowIndex, headers, obj) {
  headers.forEach((h, i) =>
    sheet.getRange(rowIndex, i + 1).setValue(obj[h] ?? "")
  );
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

function slugify(text) {
  return (
    String(text || "")
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, "")
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, "-")
      .replace(/^-+|-+$/g, "") || "cliente"
  );
}
```

## Colunas esperadas (linha 1)

- `cliente_db_master` / `clientes_db_master`  
  `cliente_id,nome_fantasia,razao_social,cnpj_ein,segmento,descricao_resumida_negocio,cidade,estado,pais,fuso_horario,idioma_principal,idioma_secundario,responsavel_principal_nome,responsavel_principal_cargo,responsavel_principal_email,responsavel_principal_whatsapp,outros_contatos,publico_principal,publico_secundario,ticket_medio_aproximado,principais_diferenciais,principais_concorrentes,tom_voz_marca,tipo_plano_social,qtd_posts_mes,qtd_reels_mes,qtd_stories_semanais,outros_entregaveis_fixos,observacoes_escopo,tipo_de_servico,gestao_trafego_ativa,plataformas_trafego,orcamento_midia_mensal,links_redes_sociais,link_identidade_visual,link_site,observacoes_gerais,data_cadastro,origem_cadastro,trello_card_id,trello_card_url,drive_client_root_id,drive_jobs_root_id,drive_conteudos_root_id,drive_admin_root_id,trello_board_id,trello_list_inbox_id,slug,logo_url,default_notes_client`

- `jobs_db`  
  `job_id,cliente_id,cliente_nome,job_titulo,job_tipo,tipo_demanda_social,canal_midias,prazo_entrega_desejado,prioridade,origem_job,responsavel_job_nome,responsavel_job_email,objetivo_principal,contexto_job,publico_job,links_referencias,sm_rede_principal,sm_formato,sm_qtd_pecas,sm_legenda_completa,sm_tom_voz,sm_cta,sm_camp_objetivo_principal,sm_camp_periodo_inicio,sm_camp_periodo_fim,sm_camp_plataformas,sm_camp_meta_resultado,sm_camp_pagina_destino,sm_camp_segmentacoes,sm_camp_localizacao,sm_camp_orcamento_total,sm_camp_orcamento_diario,sm_camp_observacoes_orcamento,sm_camp_formatos_necessarios,sm_camp_mensagem_principal,sm_camp_ofertas_beneficios,sm_camp_cta_sugerido,sm_camp_entregaveis,sm_camp_observacoes_gerais,sm_camp_referencias,ads_objetivo,ads_plataformas,ads_orcamento,ads_periodo,ads_oferta_principal,ads_publico_segmentacao,video_tipo,video_duracao,video_formato,video_tipo_trabalho,video_materiais_disponiveis,video_local,site_tipo,site_dominio,site_objetivo,site_paginas,site_responsavel_conteudo,site_funcionalidades,site_identidade_visual_link,pack_produto,pack_tipo,pack_dimensoes,pack_info_obrigatoria,pack_restricoes_legais,pack_referencias,brand_situacao_atual,brand_entregaveis,brand_palavras_chave,job_descricao_livre,links_arquivos,observacoes_gerais_job,mes_ano,data_cadastro,origem_cadastro,trello_card_id,trello_card_url,drive_job_folder_id,drive_job_folder_link,trello_pasta_link_ok`

- `roadpmaps`  
  `id,client_id,month,title,notes_client,notes_internal,created_at,updated_at`

- `roadpmaps_posts`  
  `id,roadmap_id,date,channel,theme,caption,cta,drive_asset_url,status,client_comment,order,created_at,updated_at`

## Endpoints principais

- Clientes: `GET ?action=clients`
- Jobs (logs): `GET ?action=jobs&cliente_id=CLI-123&limit=150`
- Roadmap (admin/público): `GET ?action=roadmap&client_slug=slug-ou-nome&month=YYYY-MM`
- Salvar roadmap: `POST JSON { action:"roadmap_save", client_slug, month, title, notes_client, notes_internal, posts, posts_to_delete }`
- Feedback cliente: `POST JSON { action:"roadmap_feedback", roadmap_id, post_id, status, client_comment }`

## Observações

- Slug: se a aba de clientes não tiver `slug`, ele gera a partir de `nome_fantasia` (minúsculas + hífens). Pode usar `cliente_id` no lugar do slug, se preferir.
- O roadmap é criado automaticamente se não existir (combinação client_id + month).
- Para o cliente: `roadmap_public.html?client_slug=<slug-ou-nome>&month=YYYY-MM`.
- As demais páginas (cadastro/log) continuam usando o mesmo endpoint. Reimplante o Web App após colar o script.
