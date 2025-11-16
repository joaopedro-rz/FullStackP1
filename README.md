# NewsFinder – Projeto 1 (Frontend)

Aplicação SPA em React + Vite que busca notícias da NewsAPI com filtros, validação e estado global via Context API + useReducer.

Tecnologias principais
- API: NewsAPI (https://newsapi.org/)
- Hook React: useReducer (gestão de estado global e transições de requisição)
- Biblioteca externa: React Hook Form (validação de formulário)

Funcionalidades
- Busca com parâmetros: termo (q), idioma (language), data inicial (from), ordenação (sortBy) e tamanho da página (pageSize)
- Validação de campos obrigatórios e regras: termo mínimo 2 caracteres, idioma obrigatório, pageSize entre 5 e 100
- Mensagens de erro antes da requisição (validação cliente) e depois (erros retornados pela API tratados) 
- Paginação (próxima/anterior) com cálculo de total de páginas
- Estado global com Context API + useReducer (status, erro, artigos, filtros, paginação)
- Skeleton loading e indicadores de status (idle, loading, failed, succeeded)
- Tratamento de erros específicos da NewsAPI (ex.: bloqueio de requisições pelo browser)
- Interface responsiva e foco em legibilidade
- Acessibilidade básica (aria-live em mensagens de erro)
- SEO básico com meta tags em index.html

Configuração da chave da NewsAPI (Somente no Projeto 1 – frontend direto)
A aplicação inclui uma chave padrão hard-coded em `src/contexts/NewsContext.jsx` (API_KEY_HARD_CODED). Para usar sua própria chave sem expor no código:
1. Crie arquivo .env na raiz
2. Defina: `VITE_NEWSAPI_KEY="SUA_CHAVE"`
3. Reinicie o servidor de desenvolvimento

> Nota: Na versão Fullstack a chave da NewsAPI NÃO fica no frontend; ela é lida pelo backend via variável de ambiente `NEWSAPI_KEY`.

Estrutura
- src/components: App.jsx, main.jsx, SearchForm.jsx, NewsList.jsx, styles.css
- src/contexts: NewsContext.jsx

Requisitos de build/execução
- Node 18+ (recomendado)

Scripts
- Desenvolvimento: npm run dev
- Build produção: npm run build
- Preview do build: npm run preview

Notas
- Uso acadêmico. Chaves expostas no frontend podem ser copiadas. Considere proxy/backend se necessário.
- Caso a API retorne bloqueio a requisições do navegador, utilize plano adequado ou um servidor intermediário.

---
# NewsFinder – Projeto Fullstack (Entrega 2)

Aplicação SPA (Frontend React) + Backend Express + Banco SQLite (Knex). Implementa Login, Busca, Inserção e Gestão (listar/remover) de artigos salvos.

Tecnologias principais
- Frontend: React, Vite, useReducer, React Hook Form
- Backend: Express, SQLite (Knex), JWT, bcrypt, helmet, rate-limit, express-validator, sanitize-html, apicache, compression, morgan

Funcionalidades Frontend
- Login (email/senha) e manutenção de sessão JWT
- Busca autenticada de notícias (q, language, from, sortBy, page, pageSize)
- Salvar artigo autenticado
- Visualizar artigos salvos (toggle de modo: busca vs salvos) na mesma página
- Remover artigo salvo (DELETE)
- Validação client + exibição de erros da API
- Paginação e loading skeleton (somente na busca)
- Estado global via Context + useReducer (inclui campo `mode: 'search' | 'saved'`)
- Feedback de estado ao salvar (normal / salvando / salvo)

Backend (REST)
- POST /api/auth/login: autenticação (JWT)
- POST /api/auth/logout: invalida token (blacklist via jti)
- POST /api/auth/seed: semear usuário (opcional)
- GET /api/news: busca proxy NewsAPI (autenticado, cache 5 min)
- GET /api/articles: lista artigos salvos (autenticado, suporta paginação `?page=&pageSize=`)
- POST /api/articles: cria artigo salvo
- DELETE /api/articles/:id: remove artigo salvo pertencente ao usuário

Segurança
- Senhas com bcrypt (salt rounds = 10)
- JWT com expiração (2h) e jti para blacklist de logout
- helmet (headers), rate limiting (mitiga brute force), CORS controlado
- Validação e sanitização: express-validator + sanitize-html (mitiga XSS / Injection)
- Knex parametriza queries (mitiga SQL Injection)
- Blacklist de tokens in-memory (logout seguro)

Performance
- Cache de respostas de notícias (apicache 5 min)
- Compressão gzip (compression)
- Pool de conexões SQLite (Knex pool min 1 max 5)
- Redução de payload JSON (apenas campos necessários de artigo)

Variáveis de ambiente (backend .env)
- PORT=3000
- JWT_SECRET=chave_segura
- NEWSAPI_KEY=sua_chave_newsapi
- SEED_DEFAULT_USER=true (opcional para criar admin@example.com)

Execução
Frontend:
1. npm install
2. npm run dev

Backend:
1. cd backend
2. npm install
3. Criar .env conforme acima
4. npm run dev

Estrutura
- frontend/src/components: App.jsx, LoginForm.jsx, SearchForm.jsx, NewsList.jsx
- frontend/src/contexts: AuthContext.jsx, NewsContext.jsx
- backend/src/config: db.js, tokens.js
- backend/src/models: User.js, Article.js
- backend/src/routes: auth.js, news.js, articles.js
- backend/src/server.js

Notas
- Apenas usuários autenticados podem buscar, salvar, listar e remover artigos.
- Validações de campos no frontend e backend com retorno de mensagens JSON estruturadas.
- Modo de visualização `saved` não paginado (carrega todos os artigos do usuário conforme implementação atual). Para grandes volumes, considerar paginação no frontend.

## Diferenças entre Projeto 1 e Fullstack
| Aspecto | Projeto 1 (Somente Frontend) | Projeto Fullstack |
|---------|------------------------------|-------------------|
| Chave da NewsAPI | Exposta no frontend (.env VITE_) | Mantida no backend (.env) |
| Endpoint de busca | Direct NewsAPI | Proxy `/api/news` (seguro + cache) |
| Artigos salvos | Não existe | CRUD parcial (create/list/delete) |
| Remoção de artigo | — | DELETE `/api/articles/:id` |
| Logout | — | POST `/api/auth/logout` (blacklist jti) |
| Segurança | Apenas validação client | JWT, bcrypt, headers, rate-limit, sanitização |

## Tabela de Endpoints Atualizada
| Método | Endpoint | Descrição | Autenticação | Principais Status |
|--------|----------|-----------|--------------|------------------|
| POST | /api/auth/login | Login usuário | ❌ | 200, 400, 401 |
| POST | /api/auth/logout | Logout (blacklist) | ✅ | 200, 400 |
| POST | /api/auth/seed | Criar usuário inicial | ❌ | 201, 200, 400 |
| GET | /api/news | Buscar notícias | ✅ | 200, 400, 401, 500, 502 |
| GET | /api/articles | Listar artigos salvos | ✅ | 200, 400, 401 |
| POST | /api/articles | Salvar artigo | ✅ | 201, 400, 401, 500 |
| DELETE | /api/articles/:id | Remover artigo salvo | ✅ | 200, 400, 401, 404, 500 |
| GET | /api/health | Health check | ❌ | 200 |

## Pontos de Validação e Erros
- Todos os inputs principais validados server-side (login, busca, salvar, remover).
- Erros retornados como JSON estruturado `{ errors: [...] }` ou `{ error: 'mensagem' }`.
- Frontend exibe mensagens vindas do backend além das validações locais (React Hook Form).

## Estado Global (NewsContext)
- Reducer controla filtros, artigos, totalResults, status e `mode`.
- Ação extra: `SET_MODE` e `REMOVE_ARTICLE` refletem funcionalidades de visualização / remoção.

## Possíveis Melhorias Futuras
- Paginação dos artigos salvos no frontend (já há suporte query no backend).
- Persistência de blacklist de tokens (hoje in-memory, reinício limpa). Para produção: Redis ou DB.
- Refresh token e rotação de chaves JWT.
- Testes automatizados (unitários e integração) para rotas críticas.

## Checklist Rápido de Coerência (README vs Código)
| Item | Status | Observação |
|------|--------|-----------|
| Tecnologias listadas (backend) | OK | Todas presentes em package.json |
| Tecnologias listadas (frontend) | OK | Dependências presentes |
| Endpoints login/news/articles (GET/POST) | OK | Implementados |
| Endpoint logout | ADICIONADO | Agora documentado |
| Endpoint DELETE artigos | ADICIONADO | Agora documentado |
| Visualizar artigos salvos | ADICIONADO | Modo `saved` no NewsContext |
| Remover artigo salvo | ADICIONADO | Botão e rota DELETE |
| Chave NewsAPI no frontend fullstack | CORRIGIDO | Uso somente backend |
| Blacklist de tokens | ADICIONADO | Explicação incluída |
| Pool SQLite | OK | min 1 max 5 conforme db.js |
| Cache notícias | OK | apicache 5 min server.js |
| Compressão | OK | compression no server |
| Sanitização XSS | OK | sanitize-html em articles POST |
| Rate limiting | OK | express-rate-limit no server |
| Mensagens de validação | OK | express-validator em rotas |

## Como Rodar (Resumo)
```bash
# Backend
cd backend
npm install
cp .env.example .env  # (se houver exemplo) ou criar manualmente
# Editar .env: PORT=3000 / JWT_SECRET=... / NEWSAPI_KEY=...
npm run dev

# Frontend (em outro terminal)
cd frontend
npm install
npm run dev
```

Pronto: README sincronizado com o estado atual do projeto.
