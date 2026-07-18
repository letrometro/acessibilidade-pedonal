# 📍 Levantamento de Acessibilidade Pedonal - GitHub Pages

Aplicação web para levantamento de dados de acessibilidade pedonal com GPS, fotografias e mapa interativo.

**Status**: ✅ Pronto para publicar online  
**Autor**: Câmara Municipal de Lisboa - PAPL  
**Versão**: 2.0

---

## 🚀 Como Colocar Online em 5 Passos

### **Passo 1: Criar Conta no GitHub** (se não tiver)

1. Aceda a [github.com](https://github.com)
2. Clique em **"Sign up"** (canto superior direito)
3. Preencha:
   - Email (pode usar email da CML)
   - Password forte
   - Username (ex: `papl-cml` ou `acessibilidade-lisboa`)
4. Confirme no email

---

### **Passo 2: Criar Novo Repositório**

1. Depois de logged in, clique no ➕ (canto superior direito) → **"New repository"**
2. Preencha:
   - **Repository name**: `acessibilidade-pedonal` (ou nome à sua escolha)
   - **Description**: "Levantamento de acessibilidade pedonal - CML"
   - **Public** (marque a opção)
   - ✅ Marque **"Add a README file"**
3. Clique em **"Create repository"**

---

### **Passo 3: Upload dos Ficheiros**

1. Na página do repositório, clique em **"Add file"** → **"Upload files"**
2. Arraste ou selecione:
   - **index.html** (ficheiro principal da aplicação)
3. Escreva no campo "Commit message": `Initial commit - App de acessibilidade pedonal`
4. Clique em **"Commit changes"**

> **Nota importante**: O ficheiro **tem de chamar-se `index.html`** para GitHub Pages funcionar automaticamente.

---

### **Passo 4: Ativar GitHub Pages**

1. Na página do repositório, vá a **Settings** (aba junto a "Code")
2. Na barra lateral esquerda, clique em **"Pages"** (secção "Code and automation")
3. Em **"Source"**, mude para:
   - **Deploy from a branch**
   - **Branch**: `main`
   - **Folder**: `/ (root)`
4. Clique em **"Save"**

> GitHub vai processar alguns segundos. A página vai recarregar com uma mensagem verde:
> ✅ "Your site is live at `https://seuusername.github.io/acessibilidade-pedonal`"

---

### **Passo 5: Partilhar o Link**

O seu link está pronto! 🎉

Exemplo: `https://seuusername.github.io/acessibilidade-pedonal`

---

## 🔗 Usar a Aplicação

### **No Smartphone**

1. Abre o browser (Chrome, Safari, etc.)
2. Cola o link: `https://seuusername.github.io/acessibilidade-pedonal`
3. Pronto! Funciona offline (dados guardados no telemóvel)

### **No Computador**

1. Qualquer browser funciona
2. Partilha o link com a equipa
3. Cada pessoa tem os seus próprios registos locais

---

## 💾 Dados & Exportação

### **Armazenamento**
- ✅ Dados guardados **localmente no dispositivo** (localStorage)
- ✅ Funciona **offline** (sem internet)
- ✅ Cada dispositivo tem registos separados

### **Exportação**

Na aba **"Dados"**:
- **📊 CSV**: Para Excel/análise
- **📄 JSON**: Para importar noutros sistemas

---

## 🎯 Funcionalidades

### **Novo Registo** (Fase 1 → 2 → 3)

**Fase 1 - Categoria**
- 8 bolas redondas com categorias
- Clica para selecionar

**Fase 2 - Tipo de Situação**
- ✅ Bom Exemplo (verde)
- ⚠️ A Resolver (amarelo)
- 🚨 Perigo Urgente (vermelho)

**Fase 3 - Confirmação**
- Resumo das opções
- GPS automático
- Captura de foto (câmara ou galeria)
- Guardar registo

### **Mapa**
- 🗺️ Visualização de todos os registos
- 📍 Marcadores coloridos por tipo
- 🔍 Legenda com contagem

### **Dados**
- 📊 Estatísticas resumidas
- 📋 Lista completa de registos
- 📥 Exportação em CSV/JSON

---

## 🛠️ Personalização

### **Alterar Título / Logo**

Edite o `index.html` na linha:
```html
<h1>📍 Acessibilidade Pedonal</h1>
```

### **Alterar Categorias**

No `index.html`, localize `CATEGORIAS` (por volta da linha 600):
```javascript
const CATEGORIAS = {
    'pavimento': { nome: 'Problema de Pavimento', emoji: '🕳️' },
    'desnivel': { nome: 'Desnível / Degrau', emoji: '📏' },
    // ... adicione mais
};
```

### **Alterar Cores**

As cores principais estão no topo do `<style>`:
- Azul principal: `#2c5aa0` → mude para cor municipal
- Verde: `#27ae60`
- Amarelo: `#f39c12`
- Vermelho: `#e74c3c`

---

## 📱 Compatibilidade

✅ **Smartphones** (Android, iOS)  
✅ **Tablets**  
✅ **Computadores** (Chrome, Firefox, Safari, Edge)  
✅ **Offline-first** (funciona sem internet)  

---

## 🔒 Segurança & Privacidade

- ✅ **Sem servidor**: todos os dados ficam no dispositivo
- ✅ **Sem conta**: não precisa registar-se
- ✅ **Sem rastreamento**: nenhuma informação é enviada para a internet
- ✅ **Aberto**: pode auditar o código (está em HTML público)

---

## 🐛 Problemas Comuns

### "GitHub Pages não aparece ativa"
- Verifique em **Settings → Pages** se está configurado como indicado acima
- Aguarde 1-2 minutos (GitHub processa)

### "Ficheiro não abre no GitHub Pages"
- Confirme que se chama `index.html` (exatamente)
- Verifique se foi feito o upload para o branch `main`

### "GPS não funciona"
- Permita localização no browser
- Aguarde alguns segundos (requer sinal de satélite)

### "Foto não aparece"
- Confirme permissões de câmara no telemóvel
- Tente novamente

---

## 📞 Suporte

Para questões técnicas, consulte:
- [Documentação GitHub Pages](https://docs.github.com/en/pages)
- [MDN Web Docs](https://developer.mozilla.org/)

---

## 📄 Licença

Desenvolvido para Câmara Municipal de Lisboa - PAPL  
Domínio público - use e adapte livremente

---

**Última atualização**: Julho 2026  
**Versão**: 2.0 - Fluxo visual em 3 fases
