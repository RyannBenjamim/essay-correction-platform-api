<div align="center">

# ✍️ Plataforma de Redação — API

**API REST para gestão de um curso preparatório de redação no modelo do ENEM:**
envio e correção de redações por competência, propostas de tema, simulados, turmas, frequência, pagamentos e videoaulas.

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express_5-000000?style=for-the-badge&logo=express&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma_6-2D3748?style=for-the-badge&logo=prisma&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-3E67B1?style=for-the-badge&logo=zod&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

</div>

---

## 📌 Sobre o projeto

Esta API é o backend de uma plataforma para cursos de redação. Ela centraliza o fluxo entre alunos e a equipe pedagógica:

- o **aluno** envia sua redação em arquivo e acompanha as correções, as notas e a própria posição no ranking;
- a **equipe/administração** publica propostas de tema, corrige as redações pelas **5 competências do ENEM**, lança notas de simulados, controla a frequência e os pagamentos dos alunos e organiza módulos de videoaulas a partir de playlists do YouTube.

O projeto expõe apenas a API; a interface web que a consome não faz parte deste repositório.

---

## ✨ Funcionalidades

| Módulo | O que a API oferece |
| --- | --- |
| 👤 **Usuários** | Cadastro (vinculado a uma turma), login com JWT, busca por nome, edição, exclusão, troca de senha e foto de perfil (JPEG, PNG, GIF ou WebP, até 5 MB) |
| 🏫 **Turmas** | CRUD completo, com nome único |
| 📝 **Redações** | Upload do arquivo da redação, listagem com filtros (por aluno, corrigidas, pendentes) e download |
| ✅ **Correções** | Nota de 0 a 200 em cada uma das 5 competências, nota final calculada automaticamente (0 a 1000), feedback escrito e arquivo da correção; a redação passa de `PENDENTE` para `CORRIGIDA` |
| 📄 **Propostas** | Upload de propostas de tema e download da proposta mais recente |
| 🏆 **Ranking** | Classificação dos alunos pela média das notas das redações corrigidas, com a turma de cada um |
| 🧪 **Simulados** | Simulados por turma e lançamento de notas por competência para cada aluno (uma nota por aluno em cada simulado) |
| 📅 **Frequência** | Registro de presença (`PRESENTE`, `FALTOU`, `JUSTIFICADO`) com justificativa opcional e consulta por aluno |
| 💳 **Pagamentos** | Registro de pagamentos com valor e vencimento, e consulta do histórico de um aluno |
| 🎬 **Módulos e vídeos** | Ao criar um módulo com a URL de uma playlist do YouTube, a API importa automaticamente todos os vídeos (título, link, miniatura e ordem) |

### Regras de negócio que vale destacar

- **Nota da correção:** o campo `nota` é a soma das cinco competências e é calculado pela própria API.
- **Retenção de arquivos:** cada aluno mantém até **20 redações** e **20 correções**, e a plataforma guarda até **8 propostas**. Ao atingir o limite, o registro mais antigo é removido junto com seu arquivo.
- **Senha inicial:** ao cadastrar um usuário, a senha inicial é gerada a partir do e-mail (a parte antes de `@gmail.com`) e armazenada com hash `bcrypt`. O aluno pode alterá-la depois pela rota de troca de senha.
- **Validação:** os corpos das requisições são validados com **Zod**; dados inválidos retornam `400`.
- **Exclusões em cascata:** remover um usuário, redação, módulo ou simulado remove também os registros dependentes. Ao remover uma turma, seus simulados e frequências são apagados, e os alunos ficam sem turma vinculada.

---

## 🛠️ Tecnologias

| Camada | Ferramentas |
| --- | --- |
| Runtime e servidor | Node.js, Express 5, CORS |
| Banco de dados | PostgreSQL, Prisma ORM (schema + migrations) |
| Validação | Zod |
| Autenticação e segurança | JSON Web Token, bcrypt |
| Upload de arquivos | Multer |
| Integrações | `ytpl` (leitura de playlists do YouTube) |
| Infraestrutura | Docker Compose (PostgreSQL local), configuração para deploy na Vercel |

---

## 🧭 Arquitetura

O código segue uma arquitetura em camadas, em que cada recurso tem seu próprio controller, model, repository, schema e entidade:

```text
Requisição HTTP
   │
   ▼
routes.js ──► middlewares (auth, upload com Multer)
   │
   ▼
controllers/   → lê req/res e devolve a resposta HTTP
   │
   ▼
models/        → regras de negócio + validação com Zod
   │
   ▼
repositories/  → acesso ao banco com Prisma Client
   │
   ▼
PostgreSQL
```

- **`entities/`** montam os objetos antes da persistência (geração de UUID, cálculo da nota da correção).
- **`error/http-error.js`** define um erro com status HTTP, que o **`error-middleware`** converte em uma resposta JSON `{ "error": "mensagem" }`.

---

## 📁 Estrutura do projeto

```text
.
├── prisma/
│   ├── schema.prisma        # modelos, enums e relações
│   └── migrations/          # histórico de migrations do PostgreSQL
├── src/
│   ├── server.js            # configuração do Express e inicialização
│   ├── routes.js            # definição de todas as rotas
│   ├── controllers/         # camada HTTP
│   ├── models/              # regras de negócio
│   ├── repositories/        # consultas com Prisma
│   ├── schemas/             # validações com Zod
│   ├── entities/            # construção dos objetos persistidos
│   ├── middlewares/         # autenticação, uploads e tratamento de erros
│   ├── database/db.js       # instância do Prisma Client
│   ├── error/               # classe HttpError
│   └── utils/               # remoção de arquivos e leitura de playlists
├── docker-compose.yml       # PostgreSQL para desenvolvimento
├── vercel.json              # configuração de deploy na Vercel
└── package.json
```

---

## 🚀 Como executar localmente

### Pré-requisitos

- [Node.js](https://nodejs.org/) 18.18 ou superior (exigência do Prisma 6)
- npm
- [Docker](https://www.docker.com/) com Docker Compose **ou** uma instância própria do PostgreSQL

### 1. Clone o repositório e instale as dependências

```bash
git clone https://github.com/RyannBenjamim/essay-correction-platform-api.git
cd essay-correction-platform-api
npm install
```

### 2. Suba o banco de dados

O `docker-compose.yml` cria um PostgreSQL exposto na porta **5433**:

```bash
docker compose up -d
```

### 3. Configure as variáveis de ambiente

Crie um arquivo `.env` na raiz do projeto:

```env
DATABASE_URL="postgresql://postgres:docker@localhost:5433/BDPlataformaRedacao?schema=public"
PORT=3000
```

| Variável | Obrigatória | Descrição |
| --- | --- | --- |
| `DATABASE_URL` | Sim | String de conexão do PostgreSQL usada pelo Prisma. O exemplo acima corresponde às credenciais de desenvolvimento do `docker-compose.yml`. |
| `PORT` | Não | Porta do servidor HTTP. Padrão: `3000`. |

### 4. Aplique as migrations e gere o Prisma Client

```bash
npx prisma migrate deploy
npx prisma generate
```

### 5. Inicie o servidor

```bash
npm run dev
```

O script usa `node --watch`, que reinicia o servidor a cada alteração. Para verificar se a API está no ar:

```bash
curl http://localhost:3000/teste
```

```json
{ "message": "Se você está lendo essa mensagem, é porque a api está funcionando." }
```

### Scripts disponíveis

| Script | Comando | Uso |
| --- | --- | --- |
| `dev` | `node --watch src/server.js` | Desenvolvimento local |
| `vercel-build` | `npx prisma generate && npx prisma migrate deploy` | Build executado no deploy da Vercel |

---

## 📡 Endpoints

Todas as respostas de sucesso seguem o formato `{ "message"?: string, "data": ... }`. Rotas de upload recebem `multipart/form-data` com o arquivo no campo **`file`**.

<details>
<summary><b>👤 Usuários</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `POST` | `/usuarios/login` | Autentica e retorna o token JWT |
| `GET` | `/usuarios` | Lista usuários (`?filter=nome` busca por nome) |
| `GET` | `/usuarios/:id` | Detalha um usuário 🔒 |
| `POST` | `/usuarios` | Cadastra um usuário |
| `PUT` | `/usuarios/:id` | Atualiza um usuário |
| `DELETE` | `/usuarios/:id` | Remove um usuário |
| `POST` | `/usuarios/:id/trocar-senha` | Troca a senha (`senhaAtual`, `novaSenha`) |
| `POST` | `/usuarios/:id` | Envia a foto de perfil (upload) |
| `GET` | `/usuarios/:id/profile-image` | Retorna a foto de perfil |

🔒 Requer o cabeçalho `Authorization: Bearer <token>`.

</details>

<details>
<summary><b>🏫 Turmas</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/turmas` | Lista as turmas |
| `GET` | `/turmas/:id` | Detalha uma turma |
| `POST` | `/turmas` | Cria uma turma |
| `PUT` | `/turmas/:id` | Atualiza uma turma |
| `DELETE` | `/turmas/:id` | Remove uma turma |

</details>

<details>
<summary><b>📝 Redações e correções</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/redacoes` | Lista redações (`?usuarioId=`, `?corrigidas=true`, `?pendentes=true`) |
| `GET` | `/redacoes/:id` | Detalha uma redação com aluno e correção |
| `POST` | `/redacoes/:usuarioId/upload` | Envia uma redação (`titulo`, `usuarioId`, `file`) |
| `GET` | `/redacoes/download/:id` | Baixa o arquivo da redação |
| `GET` | `/correcoes` | Lista as correções com redação, aluno e turma |
| `POST` | `/correcoes/:usuarioId/upload` | Registra uma correção (`competencia01`…`competencia05`, `feedback`, `redacaoId`, `file`) |
| `GET` | `/correcoes/download/:id` | Baixa o arquivo da correção |

</details>

<details>
<summary><b>📄 Propostas e 🏆 Ranking</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/propostas` | Lista as propostas |
| `GET` | `/propostas/:id` | Detalha uma proposta |
| `POST` | `/propostas` | Envia uma proposta (`tema`, `file`) |
| `DELETE` | `/propostas/:id` | Remove uma proposta e seu arquivo |
| `GET` | `/propostas/download` | Baixa a proposta mais recente |
| `GET` | `/ranking` | Ranking dos alunos pela média das redações corrigidas |

</details>

<details>
<summary><b>🧪 Simulados</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/simulados` | Lista os simulados |
| `GET` | `/simulados/:id` | Detalha um simulado |
| `GET` | `/simulados/turmaId/:id` | Lista os simulados de uma turma |
| `POST` | `/simulados` | Cria um simulado (`titulo`, `turmaId`) |
| `DELETE` | `/simulados/:id` | Remove um simulado |
| `GET` | `/notaSimulado` | Lista todas as notas de simulado |
| `GET` | `/notaSimulado/:id` | Detalha uma nota |
| `GET` | `/notaSimulado/simuladoId/:id` | Lista as notas de um simulado |
| `POST` | `/notaSimulado` | Lança a nota de um aluno |
| `DELETE` | `/notaSimulado/:id` | Remove uma nota |

</details>

<details>
<summary><b>📅 Frequência e 💳 Pagamentos</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/frequencias` | Lista as frequências |
| `GET` | `/frequencias/:id` | Detalha uma frequência |
| `GET` | `/frequencias/aluno/:id` | Lista as frequências de um aluno |
| `POST` | `/frequencias` | Registra uma frequência |
| `PUT` | `/frequencias/:id` | Atualiza uma frequência |
| `DELETE` | `/frequencias/:id` | Remove uma frequência |
| `GET` | `/pagamentos` | Lista os pagamentos |
| `GET` | `/pagamentos/:id` | Lista os pagamentos de um aluno (id do usuário) |
| `POST` | `/pagamentos` | Registra um pagamento |
| `PUT` | `/pagamentos/:id` | Atualiza um pagamento |
| `DELETE` | `/pagamentos/:id` | Remove um pagamento |

</details>

<details>
<summary><b>🎬 Módulos e vídeos</b></summary>

| Método | Rota | Descrição |
| --- | --- | --- |
| `GET` | `/modulos` | Lista os módulos com seus vídeos |
| `GET` | `/modulos/:id` | Detalha um módulo com seus vídeos |
| `POST` | `/modulos` | Cria um módulo e importa os vídeos da playlist |
| `PUT` | `/modulos/:id` | Atualiza um módulo |
| `DELETE` | `/modulos/:id` | Remove um módulo e seus vídeos |
| `GET` | `/videos/:id` | Detalha um vídeo |

</details>

### Exemplos de requisição

**Login**

```bash
curl -X POST http://localhost:3000/usuarios/login \
  -H "Content-Type: application/json" \
  -d '{ "email": "aluno@gmail.com", "password": "aluno" }'
```

```json
{
  "message": "Login efetuado com sucesso",
  "data": { "token": "<jwt>", "id": "<uuid>", "role": "STANDARD" }
}
```

**Envio de uma redação**

```bash
curl -X POST http://localhost:3000/redacoes/<usuarioId>/upload \
  -F "titulo=Desafios da educação digital no Brasil" \
  -F "usuarioId=<usuarioId>" \
  -F "file=@redacao.pdf"
```

**Correção por competências**

```bash
curl -X POST http://localhost:3000/correcoes/<usuarioId>/upload \
  -F "redacaoId=<redacaoId>" \
  -F "competencia01=160" \
  -F "competencia02=180" \
  -F "competencia03=160" \
  -F "competencia04=200" \
  -F "competencia05=140" \
  -F "feedback=Boa argumentação; desenvolva melhor a proposta de intervenção." \
  -F "file=@correcao.pdf"
```

A correção é salva com `nota: 840` e a redação passa a ter o status `CORRIGIDA`.

**Criação de um módulo a partir de uma playlist**

```bash
curl -X POST http://localhost:3000/modulos \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Competência 5",
    "descricao": "Proposta de intervenção",
    "playlistUrl": "https://www.youtube.com/playlist?list=<id-da-playlist>"
  }'
```
