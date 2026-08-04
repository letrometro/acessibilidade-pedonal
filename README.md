# Selo de Acessibilidade — PWA

App para avaliar a acessibilidade de comércio, serviços e restauração (âmbito do DL 163/2006),
com fotografia do estabelecimento, regra de selo com critérios eliminatórios, e exportação CSV.

É uma **PWA instalável e offline**, com **sincronização entre dispositivos** via Supabase.

## Conteúdo

```
index.html              A app (HTML/CSS/JS, tudo num ficheiro)
sync.js                 Sincronização com Supabase (pull/push/fila offline)
schema.sql               Esquema da base de dados + Storage (executar no Supabase)
manifest.webmanifest     Manifesto da PWA (nome, ícones, cores)
sw.js                    Service worker (offline)
vendor/supabase.js       Cliente Supabase JS, vendorizado para funcionar offline
icons/                   Ícones 180/192/512 + maskable
README.md                Este ficheiro
```

## Estado atual

- **Instalável**: manifesto + ícones + meta tags de ecrã inteiro.
- **Offline garantido**: o `sw.js` guarda a app (incluindo o cliente Supabase) em cache;
  depois da 1ª abertura funciona sem rede. A UI nunca espera pela rede — os dados ficam
  primeiro em `localStorage` e sincronizam em segundo plano.
- **Sincronização entre dispositivos**: avaliações e fotografias partilhadas por
  "código de equipa", com fila offline e resolução de conflitos por *last-write-wins*.

## Como publicar (necessário para PWA e service worker)

O service worker só funciona sobre **HTTPS** (ou `localhost`) — não a partir de `file://`.
Publica a pasta inteira num alojamento estático gratuito. Qualquer um destes serve:

- **Cloudflare Pages** ou **Netlify**: liga a um repositório Git, ou arrasta a pasta. Sem build.
- **GitHub Pages**: coloca os ficheiros num repositório e ativa Pages.

Depois de publicada, abre o endereço no telemóvel e usa **Adicionar ao ecrã principal**
(iPhone/Safari) ou **Instalar aplicação** (Android/Chrome).

### Testar localmente

```bash
# a partir da pasta do projeto
python3 -m http.server 8080
# abre http://localhost:8080  (o service worker funciona em localhost)
```

### Atualizações

Depois de mudares o `index.html`, `sync.js` ou os ícones, incrementa a versão do cache no
`sw.js` (`acessibilidade-v1` → `acessibilidade-v2`). Os dispositivos instalados atualizam
sozinhos.

---

## Configurar a sincronização com Supabase

1. **Criar o projeto** em [supabase.com](https://supabase.com) (tem plano gratuito).
2. **Base de dados**: no SQL Editor do projeto, corre o conteúdo de `schema.sql`. Isto cria
   a tabela `avaliacoes`, as políticas de acesso (RLS) e o bucket de Storage `fotos` para as
   fotografias.
3. **Chaves**: em *Project Settings → API*, copia o **Project URL** e a **anon public key** e
   preenche o objeto `CONFIG` no topo de `sync.js`:

   ```js
   const CONFIG = {
     url: "https://xxxxxxxxxxxx.supabase.co",
     anonKey: "eyJhbGciOi...",
     table: "avaliacoes",
     bucket: "fotos",
   };
   ```

   A anon key é pública por design — a segurança fica a cargo das políticas RLS em
   `schema.sql`, não da chave estar "escondida".
4. **Publicar** — depois de preencher as chaves, publica a app (ver secção acima). Sem
   chaves configuradas, a app continua a funcionar normalmente em modo local/offline; a
   sincronização fica apenas desligada (indicado no botão de estado, ver abaixo).

### Como funciona

- **Código de equipa**: toca no indicador junto ao contador de avaliações (canto superior
  da lista) para definir/mudar o código de equipa partilhado por todos os dispositivos que
  devem ver os mesmos registos. Sem código definido, usa-se `default`.
- **Pull**: ao arrancar e sempre que a ligação volta (evento `online`), a app vai buscar as
  linhas alteradas desde o último sync dessa equipa e faz merge por `id`, com regra
  *last-write-wins* pelo campo `atualizado`. Registos eliminados noutro dispositivo
  (tombstone `eliminado=true`) são removidos localmente.
- **Push**: `Store.save()` grava sempre primeiro em local (a UI nunca espera pela rede) e só
  depois envia (`upsert`) para o Supabase. Sem rede, o `id` fica numa fila em `localStorage`
  e é reenviado automaticamente quando a ligação volta.
- **Fotografias**: ao guardar uma avaliação com foto nova, a imagem (já redimensionada,
  canvas → JPEG 0,7, máx. 1024px) é enviada para o bucket `fotos` do Supabase Storage em
  segundo plano; o registo passa a ter `foto_url` e essa é a única versão da foto enviada
  para a base de dados — nunca em base64 dentro das linhas.
- **Estado de sincronização**: o botão junto ao contador mostra `sincronizado`,
  `a sincronizar…`, `offline`, `erro de sync` ou `sync desligado` (sem chaves configuradas).

---

## Usar a Aplicação

### No Smartphone

1. Abre o browser (Chrome, Safari, etc.)
2. Cola o link da app publicada
3. Instala com **Adicionar ao ecrã principal** / **Instalar aplicação**
4. Funciona offline (dados guardados no telemóvel) e sincroniza quando há rede

### No Computador

1. Qualquer browser funciona
2. Usa o mesmo código de equipa para ver os registos partilhados

---

## Dados & Exportação

- Cada avaliação tem `id` único, gerado no cliente, e um campo `atualizado` (timestamp),
  usados como ponto único de integração com a sincronização.
- **Exportar CSV**: botão "Exportar" no formulário de uma avaliação (um registo), ou
  duplo-toque no título "Avaliações" na lista para exportar tudo.

---

## Critério de pronto

- Criar uma avaliação num dispositivo e vê-la aparecer noutro após o sync.
- Editar offline e confirmar que reconcilia ao voltar a ligar (sem perder dados).
- Fotografias a carregar a partir do Storage, não embebidas nas linhas.
- App continua a instalar e a abrir offline.

## Notas

- Manter a app **local-first**: a UI nunca deve bloquear à espera da rede.
- A política de acesso em `schema.sql` (`acesso por equipa`) é intencionalmente simples —
  qualquer cliente com a anon key lê/escreve qualquer linha, e o "código de equipa" é apenas
  um filtro lógico, não uma barreira de segurança. Para restringir por utilizador, trocar por
  Supabase Auth (magic link) e políticas RLS filtradas por utilizador — ver comentários em
  `schema.sql`.
