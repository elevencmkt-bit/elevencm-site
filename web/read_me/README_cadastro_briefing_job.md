# Briefing de Job → Google Sheets (jobs_db)

Formulário `web/documentation/cadastro-briefing.html` envia JSON para um Web App do Apps Script que grava na aba `jobs_db`.

## Como publicar o Web App

1. Na planilha onde já está/estará a aba `jobs_db`, abra **Extensões → Apps Script**.
2. Cole o script abaixo (substitua qualquer código existente).
3. Clique em **Implantar → Nova implantação → Tipo: Web App**.
4. Configuração: **Executar como:** você (owner) · **Quem tem acesso:** qualquer pessoa com o link.
5. Salve, autorize e copie a URL gerada.
6. No `web/documentation/cadastro-briefing.html`, troque a constante `ENDPOINT_URL` por essa URL.

## Script do Apps Script

```javascript
// Web App para gravar briefings na aba jobs_db
const SHEET_NAME = "jobs_db";
const HEADERS = [
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

function doPost(e) {
  try {
    const body = parseBody(e);
    validate(body);

    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAME) || ss.insertSheet(SHEET_NAME);
    ensureHeaders(sheet);

    const jobId = `JOB-${Date.now()}`;
    const now = new Date();
    const timezone = ss.getSpreadsheetTimeZone?.() || Session.getScriptTimeZone();
    const mesAno = Utilities.formatDate(now, timezone || "GMT-3", "yyyy-MM");
    const row = HEADERS.map((key) => mapValue(key, body, jobId, now, mesAno));
    sheet.appendRow(row);

    return jsonResponse({
      success: true,
      job_id: jobId,
      message: "Briefing registrado com sucesso",
    });
  } catch (err) {
    console.error(err);
    return jsonResponse({
      success: false,
      message: err.message || "Erro inesperado",
    });
  }
}

function validate(body) {
  const missing = [];
  if (!body?.cliente_id) missing.push("cliente_id");
  if (!body?.job_titulo) missing.push("job_titulo");
  if (!body?.job_tipo) missing.push("job_tipo");
  if (missing.length) {
    throw new Error(`Campos obrigatórios faltando: ${missing.join(", ")}`);
  }
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

function ensureHeaders(sheet) {
  const firstRow = sheet.getRange(1, 1, 1, HEADERS.length).getValues()[0];
  const hasHeaders = firstRow.some((cell) => cell);
  if (!hasHeaders) {
    sheet.getRange(1, 1, 1, HEADERS.length).setValues([HEADERS]);
  }
}

function safeValue(value) {
  if (value === null || value === undefined) return "";
  if (typeof value !== "string") return value;
  const trimmed = value.trim();
  // Evita que o Sheets interprete como fórmula
  if (/^[=+\-@]/.test(trimmed)) return "'" + trimmed;
  return trimmed;
}

function mapValue(key, body, jobId, now, mesAno) {
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

function jsonResponse(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj)).setMimeType(
    ContentService.MimeType.JSON
  );
}
```

### Colunas da aba `jobs_db`

Ordem exata na linha 1 (copie/cole):

```
job_id,cliente_id,cliente_nome,job_titulo,job_tipo,tipo_demanda_social,canal_midias,prazo_entrega_desejado,prioridade,origem_job,responsavel_job_nome,responsavel_job_email,objetivo_principal,contexto_job,publico_job,links_referencias,sm_rede_principal,sm_formato,sm_qtd_pecas,sm_legenda_completa,sm_tom_voz,sm_cta,sm_camp_objetivo_principal,sm_camp_periodo_inicio,sm_camp_periodo_fim,sm_camp_plataformas,sm_camp_meta_resultado,sm_camp_pagina_destino,sm_camp_segmentacoes,sm_camp_localizacao,sm_camp_orcamento_total,sm_camp_orcamento_diario,sm_camp_observacoes_orcamento,sm_camp_formatos_necessarios,sm_camp_mensagem_principal,sm_camp_ofertas_beneficios,sm_camp_cta_sugerido,sm_camp_entregaveis,sm_camp_observacoes_gerais,sm_camp_referencias,ads_objetivo,ads_plataformas,ads_orcamento,ads_periodo,ads_oferta_principal,ads_publico_segmentacao,video_tipo,video_duracao,video_formato,video_tipo_trabalho,video_materiais_disponiveis,video_local,site_tipo,site_dominio,site_objetivo,site_paginas,site_responsavel_conteudo,site_funcionalidades,site_identidade_visual_link,pack_produto,pack_tipo,pack_dimensoes,pack_info_obrigatoria,pack_restricoes_legais,pack_referencias,brand_situacao_atual,brand_entregaveis,brand_palavras_chave,job_descricao_livre,links_arquivos,observacoes_gerais_job,mes_ano,data_cadastro,origem_cadastro,trello_card_id,trello_card_url,drive_job_folder_id,drive_job_folder_link,trello_pasta_link_ok
```

### Observações rápidas

- Campos de checkbox chegam como string separada por vírgula (conforme o HTML).
- `data_cadastro`, `mes_ano`, `origem_cadastro` e campos de Trello/Drive são preenchidos pelo script/integrações posteriores.
- Sempre reimplante o Web App após editar o código para atualizar a URL/versão ativa.
