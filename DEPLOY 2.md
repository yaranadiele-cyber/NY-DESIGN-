# 🚀 Guia Completo — VSCode + Vercel + Supabase

## O que você vai precisar instalar

- **VSCode** → code.visualstudio.com
- **Node.js** → nodejs.org (versão LTS)
- **Git** → git-scm.com

---

## PARTE 1 — Configurar o VSCode

### 1. Instale as extensões recomendadas
Abra o VSCode → aba Extensions (Ctrl+Shift+X) e instale:
- **Live Server** (ver o site localmente)
- **Prettier** (formatar código)
- **GitLens** (controle de versão visual)

### 2. Abra a pasta do projeto
1. Crie uma pasta no seu computador chamada `ndesignweb`
2. Coloque os arquivos `index.html` e `admin.html` dentro
3. No VSCode: File → Open Folder → selecione `ndesignweb`

### 3. Ver o site localmente
- Clique com botão direito no `index.html`
- Clique em **"Open with Live Server"**
- Abre no navegador em `http://localhost:5500`

---

## PARTE 2 — Configurar o Git e GitHub

### 1. Configure o Git (primeira vez)
Abra o terminal no VSCode (Ctrl+`) e rode:

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seu@email.com"
```

### 2. Inicie o repositório
```bash
git init
git add .
git commit -m "primeiro commit - N Design Web"
```

### 3. Crie o repositório no GitHub
1. Acesse github.com → clique em **"New"**
2. Nome: `ndesignweb`
3. Deixe como **Public**
4. Clique em **"Create repository"**

### 4. Conecte e suba o código
```bash
git remote add origin https://github.com/SEU-USUARIO/ndesignweb.git
git branch -M main
git push -u origin main
```

> Substitua SEU-USUARIO pelo seu usuário do GitHub

---

## PARTE 3 — Deploy na Vercel

### 1. Instale a Vercel CLI
```bash
npm install -g vercel
```

### 2. Faça login
```bash
vercel login
```
Vai abrir o navegador para autenticar com sua conta Vercel.

### 3. Deploy com um comando
Dentro da pasta do projeto:
```bash
vercel --prod
```

Responda as perguntas:
- **Set up and deploy?** → Y
- **Which scope?** → seu usuário
- **Link to existing project?** → N
- **Project name?** → ndesignweb
- **In which directory?** → ./  (só aperta Enter)

✅ Pronto! Você recebe o link:
```
https://ndesignweb.vercel.app
```

### 4. Para atualizar o site depois
Sempre que editar os arquivos, basta rodar:
```bash
git add .
git commit -m "atualização"
git push
```
A Vercel atualiza automaticamente! ✨

---

## PARTE 4 — Configurar o Supabase

### 1. Criar projeto
1. Acesse **supabase.com** → "Start for free"
2. Login com GitHub
3. "New Project" → nome: `ndesignweb`
4. Crie uma senha forte para o banco
5. Região: **South America (São Paulo)**
6. Aguarde ~2 minutos

### 2. Pegar as credenciais
Vá em **Settings → API** e copie:
- **Project URL** → `https://xxxx.supabase.co`
- **anon public key** → chave longa

### 3. Criar as tabelas
Vá em **SQL Editor → New Query**, cole e execute:

```sql
-- PROJETOS
create table projects (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  category text,
  url text,
  description text,
  icon text default '🌐',
  status text default 'publicado',
  created_at timestamptz default now()
);

-- SERVIÇOS
create table services (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  description text,
  icon text default '⚡',
  price text,
  prazo text,
  active boolean default true,
  order_num int default 0,
  created_at timestamptz default now()
);

-- DEPOIMENTOS
create table reviews (
  id uuid default gen_random_uuid() primary key,
  client_name text not null,
  company text,
  review_text text not null,
  stars int check (stars between 1 and 5),
  status text default 'pendente',
  created_at timestamptz default now()
);

-- CONFIGURAÇÕES
create table settings (
  key text primary key,
  value text
);

-- Dados iniciais
insert into settings (key, value) values
  ('name', 'N Design Web'),
  ('whatsapp', '5582999999999'),
  ('email', 'contato@ndesignweb.com'),
  ('pix_key', 'contato@ndesignweb.com'),
  ('projects_count', '50'),
  ('clients_count', '40'),
  ('years_count', '5');

-- Segurança RLS
alter table projects enable row level security;
alter table services enable row level security;
alter table reviews enable row level security;
alter table settings enable row level security;

create policy "leitura publica projetos" on projects
  for select using (status = 'publicado');

create policy "leitura publica servicos" on services
  for select using (active = true);

create policy "leitura publica reviews" on reviews
  for select using (status = 'publicado');

create policy "leitura publica settings" on settings
  for select using (true);

create policy "envio reviews" on reviews
  for insert with check (status = 'pendente');
```

### 4. Conectar Supabase no index.html
No VSCode, abra o `index.html` e adicione antes do `</body>`:

```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
const { createClient } = supabase;
const db = createClient(
  'https://SEU-PROJETO.supabase.co',
  'SUA-ANON-KEY'
);

async function loadProjects() {
  const { data } = await db.from('projects').select('*')
    .eq('status','publicado').order('created_at',{ascending:false});
  if(data && data.length > 0) renderProjects(data);
}

async function loadServices() {
  const { data } = await db.from('services').select('*')
    .eq('active',true).order('order_num');
  if(data && data.length > 0) renderServices(data);
}

loadProjects();
loadServices();
</script>
```

> Substitua `SEU-PROJETO` e `SUA-ANON-KEY` pelos valores copiados

### 5. Criar usuário admin no Supabase
1. Vá em **Authentication → Users**
2. Clique em **"Add user"**
3. Digite seu e-mail e uma senha forte
4. Este será o login do painel admin em produção

---

## PARTE 5 — Domínio personalizado (opcional)

Na Vercel → Settings → Domains:
1. Adicione `ndesignweb.com.br`
2. No seu provedor de domínio (Registro.br, Hostinger...):
   - Tipo A → `76.76.19.19`
   - ou CNAME → `cname.vercel-dns.com`

---

## ✅ Checklist Final

- [ ] VSCode instalado com extensões
- [ ] Node.js instalado
- [ ] Git configurado
- [ ] Repositório no GitHub criado
- [ ] Deploy na Vercel feito
- [ ] Supabase criado com tabelas
- [ ] Credenciais Supabase inseridas no código
- [ ] Número de WhatsApp atualizado
- [ ] Chave PIX atualizada
- [ ] Usuário admin criado no Supabase Auth

---

## 🆘 Comandos úteis no terminal VSCode

```bash
# Ver status dos arquivos alterados
git status

# Subir alterações (use sempre os 3 juntos)
git add . && git commit -m "update" && git push

# Ver no navegador sem Live Server
npx serve .

# Ver logs da Vercel
vercel logs
```

---

## 📞 Links para baixar tudo

- VSCode → https://code.visualstudio.com
- Node.js → https://nodejs.org
- Git → https://git-scm.com
- GitHub → https://github.com
- Vercel → https://vercel.com
- Supabase → https://supabase.com
