# Passou Concursos — Landing Page (lista de espera)

Landing page estática (HTML + CSS + JS vanilla) para captura de e-mails da lista de espera do beta focado no **Banco do Brasil**. Hospedada na Hostinger, com deploy automático via GitHub Actions.

## Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | A landing page (CSS e JS inline — arquivo único) |
| `privacidade.html` | Política de privacidade (LGPD) |
| `.htaccess` | HTTPS, gzip e cache na Hostinger |
| `.github/workflows/deploy.yml` | Deploy automático por FTP a cada push na `main` |

## 1. Configurar o destino dos e-mails (Supabase — sem n8n)

No painel do Supabase, rode este SQL (**SQL Editor → New query**):

```sql
create table public.waitlist (
  id uuid primary key default gen_random_uuid(),
  email text not null unique,
  source text,
  created_at timestamptz not null default now()
);

alter table public.waitlist enable row level security;

-- A anon key só consegue INSERIR. Ninguém lê a lista pelo navegador.
create policy "anon pode entrar na lista"
  on public.waitlist for insert
  to anon
  with check (true);
```

Depois, em `index.html`, procure `CONFIG` (dentro do `<script>`) e preencha:

```js
var CONFIG = {
  SUPABASE_URL: "https://SEU-PROJETO.supabase.co",  // Settings > API > Project URL
  SUPABASE_ANON_KEY: "eyJ...",                       // Settings > API > anon public
  WEBHOOK_URL: ""                                     // deixe vazio
};
```

> Alternativa: se preferir n8n, preencha só o `WEBHOOK_URL` — ele tem prioridade.

## 2. Configurar o deploy automático (GitHub → Hostinger)

No repositório do GitHub: **Settings → Secrets and variables → Actions → New repository secret**. Crie os 3 segredos:

| Secret | Valor (pegue no hPanel da Hostinger em **Arquivos → Contas FTP**) |
|---|---|
| `FTP_SERVER` | Host FTP (ex.: `ftp.seudominio.com.br` ou o IP mostrado) |
| `FTP_USERNAME` | Usuário FTP |
| `FTP_PASSWORD` | Senha FTP |

A cada `git push` na branch `main`, o site é publicado sozinho. Também dá pra disparar manualmente em **Actions → Deploy para Hostinger → Run workflow**.

⚠️ Se o deploy conectar mas os arquivos caírem no lugar errado, ajuste `server-dir` no workflow:
- Usuário FTP cai na raiz da conta → `server-dir: ./public_html/` (padrão atual)
- Usuário FTP já cai dentro de `public_html` → `server-dir: ./`

## 3. Antes de publicar

- [ ] Trocar `https://passouconcursos.com.br` pelo domínio real (canonical, OG e JSON-LD no `index.html`)
- [ ] Preencher `SUPABASE_URL` e `SUPABASE_ANON_KEY` no `CONFIG`
- [ ] Subir uma `og-image.png` de 1200x630 na raiz (preview de redes sociais)
- [ ] Testar o formulário no site publicado
- [ ] Rodar o PageSpeed: https://pagespeed.web.dev
