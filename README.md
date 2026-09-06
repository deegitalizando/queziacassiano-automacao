# Automação de Carrosséis — @queziacassiano

Projeto separado do `crewolada-ingressos`. Automação de geração, aprovação e (futuramente) publicação de conteúdo no Instagram para a Quézia Cassiano, espelhando a arquitetura já usada para @consorcioparaleigos (Método Seta), com identidade visual e tom próprios.

## Estrutura

- `aprovacao/index.html` — página de aprovação (sem senha). Publicar em `queziacassiano.com/aprovacao`. Lê e grava na fila via webhooks públicos do n8n (ver abaixo).
- `carrosseis/carrosselN/` — slides exportados (PNG 1080x1350, prontos pra upload no Instagram) + legenda sugerida de cada carrossel já gerado.

## Identidade visual

Paleta igual à do Método Seta (mesma usada em @consorcioparaleigos):
- Preto `#0A0A0A`, branco `#FFFFFF`, cinza escuro `#222222`, cinza `#6B6B6B`, cinza claro `#E5E5E5`, off-white `#F5F5F5`
- Vermelho de destaque `#D23025`
- Estilo minimalista, tipografia bold sem serifa, bastante espaço em branco
- Tom: humor + reflexão (diferente do tom mais sério/autoridade do Método Seta)

## Infraestrutura (n8n)

Tudo em `https://n8n-n8n.wmbyj1.easypanel.host`, projeto pessoal de Diego Carreiro Moura:

- **Data table** `Queziacassiano Fila Carrosseis` (id `lapHECl6ssiLCqFq`) — colunas: title, keyword, caption, image_urls, status, scheduled_slot, media_id, posted_at, review_note, slides_json, content_type (`carrossel` | `imagem_unica`)
- **Data table** `Queziacassiano Sugestoes Tema` (id `BuXrvgYRuu7q4c1d`)
- **Webhook GET** `/webhook/queziacassiano-fila` — lê a fila (usado pela página de aprovação)
- **Webhook POST** `/webhook/queziacassiano-aprovacao` — recebe decisão (approved/rejected/changes) e atualiza a linha
- **Webhook POST** `/webhook/queziacassiano-sugestao-tema` — recebe sugestão de tema
- **Cron (10 min)** "Auto-promover Aprovados para Pending" — promove `approved_awaiting_export` → `pending` quando `image_urls` já preenchido
- **Agendado (8h/12h/18h, America/Sao_Paulo)** "Publicar Post Agendado" — 8h posta imagem única (`content_type=imagem_unica`), 12h e 18h postam carrossel (`content_type=carrossel`). **Inativo** até a conta @queziacassiano ser conectada no Business Manager da Meta (ver TODO abaixo).

## TODO antes de publicar de verdade no Instagram

1. Conectar a conta comercial @queziacassiano no Business Manager da Meta usado por Diego Carreiro Moura.
2. Confirmar se a credencial existente "Instagram Graph API - Metodo Seta" já enxerga a nova conta (mesmo token, Business Manager compartilhado) ou se precisa criar uma credencial nova.
3. Substituir o placeholder `PREENCHER_APOS_CONECTAR_CONTA` (IG business account id) nos 4 nós do workflow de publicação.
4. Ativar o workflow.

## Fluxo de conteúdo

1. Claude gera o carrossel/imagem (texto + slides) e insere na fila com `status=review`.
2. Quézia revisa em `queziacassiano.com/aprovacao`: aprova, reprova ou pede ajuste.
3. Aprovado → `approved_awaiting_export` → Claude exporta os PNGs finais e hospeda (Hostinger `/midia/` ou GitHub raw) → grava `image_urls` → cron promove pra `pending`.
4. Workflow agendado publica no horário certo e marca `posted`.
