# Passou Concursos — Landing Page (lista de espera)

Landing page estática (HTML + CSS + JS vanilla) para captura de e-mails da lista de espera do beta focado no **Banco do Brasil**. Hospedada na Hostinger, com deploy automático via GitHub Actions. E-mails salvos em Google Sheets via webhook do n8n.

## Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | A landing page (CSS e JS inline, arquivo único) |
| `privacidade.html` | Política de privacidade (LGPD) |
| `.htaccess` | HTTPS, gzip e cache na Hostinger |
| `.github/workflows/deploy.yml` | Deploy automático por FTP a cada push |
| `n8n-workflow-lista-espera.json` | Fluxo do n8n: webhook → Google Sheets (importar no n8n) |

## 1. Destino dos e-mails: n8n → Google Sheets

O formulário já aponta para o webhook de produção:
`https://webhook.vektoria.cloud/webhook/4eb5e8b9-c013-47ed-aca8-162bd681935d`

Setup no n8n:

1. Crie uma planilha no Google Sheets com a primeira linha: `email | origem | data_hora`
2. No n8n: **Workflows → Import from File** → selecione `n8n-workflow-lista-espera.json`
3. Abra o nó **Salvar no Google Sheets** → conecte sua credencial Google → selecione a planilha e a aba
4. **Ative o workflow** (toggle no topo). Sem ativar, a URL de produção responde 404
5. O nó Webhook já vem com CORS liberado (`Allowed Origins: *`). Depois de publicar, restrinja para o domínio do site

O fluxo usa `appendOrUpdate` casando pela coluna `email`: cadastro repetido atualiza a linha em vez de duplicar.

> Alternativa sem n8n: preencher `SUPABASE_URL`/`SUPABASE_ANON_KEY` no `CONFIG` do `index.html` e esvaziar `WEBHOOK_URL` (POST direto numa tabela com RLS insert-only).

## 2. Deploy automático (GitHub → Hostinger)

No repositório: **Settings → Secrets and variables → Actions → New repository secret**. Crie os 3 segredos:

| Secret | Valor (hPanel da Hostinger em **Arquivos → Contas FTP**) |
|---|---|
| `FTP_SERVER` | Host FTP (ex.: `ftp.seudominio.com.br`) |
| `FTP_USERNAME` | Usuário FTP |
| `FTP_PASSWORD` | Senha FTP |

A cada `git push` na `main`, o site publica sozinho. Disparo manual: **Actions → Deploy para Hostinger → Run workflow**.

⚠️ Se os arquivos caírem no lugar errado, ajuste `server-dir` no workflow:
- Usuário FTP cai na raiz da conta → `server-dir: ./public_html/` (padrão atual)
- Usuário FTP já cai dentro de `public_html` → `server-dir: ./`

Se a conexão falhar, troque `protocol: ftps` por `ftp`.

## 3. Antes de publicar

- [x] Domínio configurado: `https://beta.passouconcursos.com` (canonical, OG e JSON-LD no `index.html`)
- [ ] Importar e ativar o workflow no n8n (passo 1)
- [ ] Testar o formulário no site publicado e conferir a linha na planilha
- [ ] Restringir o CORS do webhook ao domínio do site
- [ ] Rodar o PageSpeed: https://pagespeed.web.dev
