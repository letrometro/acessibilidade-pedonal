# Acessibilidade — PWAs

Este repositório tem **duas PWAs independentes**, publicadas a partir da mesma origem/alojamento:

| App | Pasta | Avalia |
|---|---|---|
| **Selo de Acessibilidade** | `index.html` (raiz) | Comércio, serviços e restauração (DL 163/2006) |
| **Microacessibilidades — Lisboa** | `pedonal/` | Barreiras no espaço público pedonal (piso, rampas, obstáculos, travessias) |

Ambas são **instaláveis e offline**, com **sincronização entre dispositivos** via Supabase, e
partilham a mesma base técnica — mas têm o seu próprio formulário, manifesto, service worker e
tabela/bucket no Supabase, para não misturar dados entre si.

---

## 1. Selo de Acessibilidade

App para avaliar a acessibilidade de comércio, serviços e restauração (âmbito do DL 163/2006),
com fotografia do estabelecimento, regra de selo com critérios eliminatórios, e exportação CSV.

### Conteúdo

```
index.html              A app (HTML/CSS/JS, tudo num ficheiro)
sync.js                 Sincronização com Supabase (pull/push/fila offline)
schema.sql               Esquema da base de dados + Storage (executar no Supabase)
manifest.webmanifest     Manifesto da PWA (nome, ícones, cores)
sw.js                    Service worker (offline)
vendor/supabase.js       Cliente Supabase JS, vendorizado para funcionar offline
icons/                   Ícones 180/192/512 + maskable
```

### Estado atual

- **Instalável**: manifesto + ícones + meta tags de ecrã inteiro.
- **Offline garantido**: o `sw.js` guarda a app (incluindo o cliente Supabase) em cache;
  depois da 1ª abertura funciona sem rede. A UI nunca espera pela rede — os dados ficam
  primeiro em `localStorage` e sincronizam em segundo plano.
- **Sincronização entre dispositivos**: avaliações e fotografias partilhadas por
  "código de equipa", com fila offline e resolução de conflitos por *last-write-wins*.

### Configurar a sincronização com Supabase

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

4. **Publicar** — depois de preencher as chaves, publica a app (ver secção "Como publicar" abaixo).
   Sem chaves configuradas, a app continua a funcionar normalmente em modo local/offline; a
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

### Dados & Exportação

- Cada avaliação tem `id` único, gerado no cliente, e um campo `atualizado` (timestamp),
  usados como ponto único de integração com a sincronização.
- **Exportar CSV**: botão "Exportar" no formulário de uma avaliação (um registo), ou
  duplo-toque no título "Avaliações" na lista para exportar tudo.

### Critério de pronto

- Criar uma avaliação num dispositivo e vê-la aparecer noutro após o sync.
- Editar offline e confirmar que reconcilia ao voltar a ligar (sem perder dados).
- Fotografias a carregar a partir do Storage, não embebidas nas linhas.
- App continua a instalar e a abrir offline.

---

## 2. Microacessibilidades — Lisboa

App para registar **barreiras de microacessibilidade pedonal** no espaço público de Lisboa —
piso degradado, rampas e lancis em falta ou inadequados, obstáculos que reduzem a largura livre
de circulação, e falhas no piso podotátil ou nas travessias. Cada registo tem fotografia,
localização (GPS), categoria/subcategoria e nível de gravidade.

As categorias baseiam-se nas normas técnicas de acessibilidade para vias e espaços exteriores
do DL 163/2006 (largura livre mínima do passeio, inclinação máxima de rampas, lancis
rebaixados nas passadeiras, piso podotátil de alerta e direcional).

### Conteúdo

```
pedonal/index.html            A app (HTML/CSS/JS, tudo num ficheiro)
pedonal/sync.js               Sincronização com Supabase (pull/push/fila offline)
pedonal/manifest.webmanifest  Manifesto da PWA (nome, ícones, cores)
pedonal/sw.js                 Service worker (offline), âmbito /pedonal/
schema-pedonal.sql             Esquema da base de dados + Storage (executar no Supabase)
```

Reaproveita `vendor/supabase.js`, `vendor/leaflet.js`/`leaflet.css` e `icons/` da raiz do
repositório (referenciados com `../`) — não há ficheiros duplicados.

### Estado atual

- **Instalável em separado do Selo de Acessibilidade**: manifesto e service worker próprios,
  com âmbito `/pedonal/`, para que as duas apps se instalem e atualizem de forma independente
  no mesmo telemóvel.
- **Offline garantido**: mesmo modelo do Selo de Acessibilidade — cache-first via `pedonal/sw.js`,
  dados sempre em `localStorage` primeiro.
- **Sincronização entre dispositivos**: por "código de equipa", como no Selo de Acessibilidade,
  mas com a sua própria tabela (`barreiras_pedonais`) e bucket (`fotos-pedonal`) no Supabase —
  os registos das duas apps não se misturam, mesmo usando o mesmo código de equipa.

### Categorias de barreira

| Categoria | Exemplos de subcategoria |
|---|---|
| Piso e pavimento | Piso irregular, buracos, desnível sem rampa, piso escorregadio |
| Rampas e lancis | Rampa inexistente ou com inclinação excessiva, lancil demasiado alto |
| Obstáculos e largura livre | Mobiliário urbano, esplanadas, veículos mal estacionados, largura < 1,20 m |
| Piso podotátil e travessias | Piso podotátil em falta/danificado, passadeira sem rebaixamento, sem sinal sonoro |

Cada registo tem ainda uma **freguesia** (as 24 de Lisboa) e uma **gravidade** (Baixa/Média/Alta).

### Configurar a sincronização com Supabase

Pode usar-se **o mesmo projeto Supabase** já criado para o Selo de Acessibilidade — só é preciso
criar a tabela e o bucket próprios desta app.

1. No SQL Editor do projeto Supabase, corre o conteúdo de `schema-pedonal.sql`. Isto cria a
   tabela `barreiras_pedonais`, as políticas de acesso (RLS) e o bucket de Storage `fotos-pedonal`.
2. Em *Project Settings → API*, confirma/preenche o objeto `CONFIG` no topo de `pedonal/sync.js`
   (mesmo `url`/`anonKey` do projeto, `table: "barreiras_pedonais"`, `bucket: "fotos-pedonal"`).
3. Publica a app (ver secção "Como publicar" abaixo). Sem chaves configuradas, a app funciona
   normalmente em modo local/offline; a sincronização fica apenas desligada.

O funcionamento (código de equipa, pull/push, fotografias, estado de sincronização) é idêntico
ao descrito na secção "Como funciona" do Selo de Acessibilidade, acima.

### Dados & Exportação

- Cada registo tem `id` único, gerado no cliente, e um campo `atualizado` (timestamp).
- **Exportar CSV**: botão "Exportar" no formulário de um registo, ou duplo-toque no título
  "Registos" na lista para exportar tudo.

### Critério de pronto

- Criar um registo num dispositivo e vê-lo aparecer noutro após o sync.
- Editar offline e confirmar que reconcilia ao voltar a ligar (sem perder dados).
- Fotografias a carregar a partir do Storage, não embebidas nas linhas.
- App instala e abre offline, em separado do Selo de Acessibilidade.

---

## Como publicar (necessário para PWA e service worker)

O service worker só funciona sobre **HTTPS** (ou `localhost`) — não a partir de `file://`.
Publica a pasta inteira (raiz do repositório, incluindo `pedonal/`) num alojamento estático
gratuito. Qualquer um destes serve:

- **Cloudflare Pages** ou **Netlify**: liga a um repositório Git, ou arrasta a pasta. Sem build.
- **GitHub Pages**: coloca os ficheiros num repositório e ativa Pages.

Depois de publicada:
- **Selo de Acessibilidade**: abre `/index.html` (ou a raiz) no telemóvel.
- **Microacessibilidades**: abre `/pedonal/index.html` no telemóvel.

Em ambos, usa **Adicionar ao ecrã principal** (iPhone/Safari) ou **Instalar aplicação**
(Android/Chrome) — instalam-se como duas apps distintas.

### Testar localmente

```bash
# a partir da pasta do projeto
python3 -m http.server 8080
# Selo de Acessibilidade:   http://localhost:8080
# Microacessibilidades:     http://localhost:8080/pedonal/
# (o service worker funciona em localhost)
```

### Atualizações

Depois de mudares `index.html`, `sync.js` ou os ícones, incrementa a versão do cache em
`sw.js` (`acessibilidade-v17` → `acessibilidade-v18`). Depois de mudares `pedonal/index.html`
ou `pedonal/sync.js`, incrementa a versão do cache em `pedonal/sw.js`
(`acessibilidade-pedonal-v1` → `acessibilidade-pedonal-v2`). Os dispositivos instalados
atualizam sozinhos.

---

## Usar as Aplicações

### No Smartphone

1. Abre o browser (Chrome, Safari, etc.)
2. Cola o link da app publicada (Selo de Acessibilidade ou Microacessibilidades)
3. Instala com **Adicionar ao ecrã principal** / **Instalar aplicação**
4. Funciona offline (dados guardados no telemóvel) e sincroniza quando há rede

### No Computador

1. Qualquer browser funciona
2. Usa o mesmo código de equipa (dentro de cada app) para ver os registos partilhados

---

## Notas

- Manter as apps **local-first**: a UI nunca deve bloquear à espera da rede.
- A política de acesso em `schema.sql` / `schema-pedonal.sql` (`acesso por equipa`) é
  intencionalmente simples — qualquer cliente com a anon key lê/escreve qualquer linha, e o
  "código de equipa" é apenas um filtro lógico, não uma barreira de segurança. Para restringir
  por utilizador, trocar por Supabase Auth (magic link) e políticas RLS filtradas por
  utilizador — ver comentários nos ficheiros de esquema.
- As duas apps partilham `vendor/` e `icons/`, mas têm `localStorage` totalmente separado
  (chaves de armazenamento e de sincronização prefixadas de forma diferente) e correm service
  workers com âmbitos distintos (`/` vs `/pedonal/`) — podem coexistir no mesmo dispositivo
  sem interferir uma com a outra.
