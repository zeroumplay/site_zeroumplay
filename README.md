# Zero Um Play — site

Site estático hospedado no GitHub Pages, domínio `zeroumplay.shop` (DNS na
Hostinger). Postar um filme ou série é só abrir uma **Issue** no repositório —
nada fora do GitHub, sem CMS separado, sem Cloudflare. Tudo pelo navegador.

## Estrutura de pastas

```
.
├── CNAME                          → domínio customizado (zeroumplay.shop)
├── index.html                      → a página do site inteira (HTML/CSS/JS)
├── content/
│   ├── filmes.json                   → lista de filmes exibida em "Novidades"
│   └── series.json                    → lista de séries exibida em "Novidades"
├── assets/capas/                     → (sem uso neste modo; as capas vêm do TMDB)
└── .github/
    ├── ISSUE_TEMPLATE/
    │   ├── novo-filme.yml                → formulário "Novo filme"
    │   ├── nova-serie.yml                 → formulário "Nova série"
    │   └── config.yml                      → desativa issue em branco
    └── workflows/
        └── novidades.yml                   → automação: lê a issue, busca no
                                               TMDB e publica no site sozinha
```

Como funciona: você abre uma issue pelo modelo "Novo filme"/"Nova série" dizendo só
o nome → a automação (GitHub Actions, já incluso e grátis) busca os dados no TMDB,
atualiza o `content/filmes.json` ou `content/series.json`, publica e fecha a issue
com um comentário confirmando. O `index.html` lê esses dois arquivos com `fetch()` —
por isso não existe passo de build nem backend.

---

## Configuração (uma vez só)

### 1. Criar o repositório e subir os arquivos
Em [github.com/new](https://github.com/new), crie o repositório (ex.:
`zeroumplay-site`), marcado como **Public**. Extraia o `.zip` no seu computador e,
na página do repositório, clique **uploading an existing file** (ou **Add file →
Upload files**). Abra a pasta extraída e arraste **todo o conteúdo dela** — inclusive
a pasta `.github` (ative "mostrar arquivos ocultos" no seu sistema se ela não
aparecer) — para a área de upload. Role até o fim e **Commit changes**.

### 2. Ativar o GitHub Pages
**Settings → Pages** → Source: **Deploy from a branch**, branch `main`, pasta
**/ (root)**. Em **Custom domain**, digite `zeroumplay.shop` e salve.

### 3. Apontar o domínio na Hostinger
No hPanel: **Domínios → zeroumplay.shop → DNS/Nameservers → aba "Registros de
DNS"** (Editor de Zona DNS). Os nameservers continuam os da própria Hostinger —
não mexa neles, só nos registros abaixo:

| Tipo  | Nome | Conteúdo                | TTL   |
|-------|------|--------------------------|-------|
| A     | @    | 185.199.108.153           | 14400 |
| A     | @    | 185.199.109.153           | 14400 |
| A     | @    | 185.199.110.153           | 14400 |
| A     | @    | 185.199.111.153           | 14400 |
| CNAME | www  | zeroumplay.github.io      | 14400 |

Apague qualquer outro registro A ou CNAME que já exista nesses mesmos hosts (`@` ou
`www`) antes de adicionar — não pode haver dois apontamentos para o mesmo nome.
Depois de propagar (minutos a poucas horas), confira em **Settings → Pages** que
`zeroumplay.shop` e `www.zeroumplay.shop` aparecem validados, sem aviso vermelho, e
marque **Enforce HTTPS**.

### 4. Permitir que a automação publique no repositório
**Settings → Actions → General**, role até **Workflow permissions**, selecione
**Read and write permissions**, clique **Save**. Sem isso a automação roda mas não
consegue salvar as alterações.

### 5. Guardar a chave do TMDB
Crie uma conta grátis em [themoviedb.org](https://www.themoviedb.org) → **Configurações
→ API** → copie a **"Chave da API (v3 auth)"**.

No repositório: **Settings → Secrets and variables → Actions → New repository
secret**. Nome: `TMDB_API_KEY`. Valor: cole a chave. **Add secret**.

Pronto — a configuração acabou aqui.

---

## Como postar um filme ou série (dia a dia)

1. No repositório, aba **Issues → New issue**.
2. Escolha **🎬 Novo filme** ou **📺 Nova série**.
3. Digite o nome exatamente como no TMDB. Se o nome for comum e puder confundir,
   inclua o ano: `Duna (2021)`. Para série, informe também o número da temporada.
4. Clique **Submit new issue**.
5. Em alguns segundos a issue recebe um comentário confirmando o que foi publicado
   (com a capa) e é fechada automaticamente. Em 1-2 minutos aparece em
   `zeroumplay.shop`.

Se o TMDB não encontrar ou pegar o título errado, a issue fica aberta com um
comentário explicando — é só abrir uma nova issue com o nome ajustado (ano entre
parênteses ajuda bastante).

### Editar ou remover um item já publicado
Isso não tem formulário — edite direto o arquivo: abra `content/filmes.json` (ou
`series.json`) no GitHub, clique no lápis, ajuste ou apague o bloco do item dentro
de `"itens": [...]`, e **Commit changes**. É um JSON simples, cada item é um
bloco `{ "capa": ..., "nome": ..., "sinopse": ... }` separado por vírgula.

### Por que só você consegue postar
A automação só processa issues abertas pela sua própria conta do GitHub (isso está
fixado no `novidades.yml`) — mesmo o repositório sendo público, outras pessoas podem
abrir issues, mas elas simplesmente não disparam a publicação.

---

## Mudanças futuras no código (fora das issues)

Para ajustes de layout, textos fixos, cores etc., edite o arquivo direto no GitHub:
abra o arquivo → ícone de lápis → edite → **Commit changes**. Para mexer em vários
arquivos de forma mais confortável, aperte `.` (ponto) na página do repositório para
abrir o editor completo no navegador.

O GitHub Pages republica sozinho a cada alteração salva na branch `main`, geralmente
em menos de 2 minutos — sem nenhum passo extra.
