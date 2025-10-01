<html lang="pt-BR">
<body>

<h1 id="nexta">NEXTA</h1>
<hr />

<p>API do Blog CRUD construída no n8n, com Supabase como banco de dados e JWT para autenticação.
<br />Rotas públicas via Webhook do n8n.</p>

<p><strong>Base URL (produção):</strong> <code>https://primary-production-e91c.up.railway.app/webhook</code></p>

<p><strong>OpenAPI (YAML):</strong> <a href="docs/openapi-n8n-onboarding.yml" target="_blank">openapi-n8n-onboarding.yml</a> <br/>
<strong>Coleção Postman:</strong> <a href="docs/postman-n8n-onboarding.json" target="_blank">postman-n8n-onboarding.json</a>

<p><em>O claim <code>sub</code> do JWT é usado como <code>user_id</code> nos registros.</em></p>

<h2 id="sumario">📑 Sumário</h2>
<ul>
  <li><a href="#arq">Arquitetura (visão geral)</a></li>
  <li><a href="#auth">Autenticação</a></li>
  <li><a href="#cors">CORS</a></li>
  <li><a href="#endpoints">Endpoints</a>
    <ul>
      <li><a href="#get-posts">GET /posts — Listar posts</a></li>
      <li><a href="#post-posts">POST /posts — Criar post</a></li>
    </ul>
  </li>
  <li><a href="#schemas">Modelos (Schemas)</a></li>
  <li><a href="#paginacao">Paginação</a></li>
  <li><a href="#codigos">Códigos de resposta</a></li>
  <li><a href="#como-testar">Como testar</a>
    <ul>
      <li><a href="#via-postman">Via Postman</a></li>
      <li><a href="#via-curl">Via cURL</a></li>
      <li><a href="#via-swagger">Via OpenAPI/Swagger</a></li>
    </ul>
  </li>
  <li><a href="#seguranca">Notas sobre segurança e RLS</a></li>
  <li><a href="#faq">FAQ rápido</a></li>
  <li><a href="#licenca">Licença</a></li>
</ul>

<h2 id="arq">Arquitetura (visão geral)</h2>
<p>Abaixo está o diagrama de sequência que mostra o fluxo completo entre usuários, frontend, n8n, Supabase e infraestrutura:</p>

<p><img src="https://i.ibb.co/JjprGGS0/diagrama.png" alt="Diagrama de Sequência API n8n" /></p>

<ul>
  <li><strong>n8n</strong> expõe os webhooks <code>/posts</code> e orquestra:
    <ul>
      <li>Validação e decodificação do <strong>JWT</strong>.</li>
      <li>Escrita/leitura no <strong>Supabase</strong>.</li>
      <li>Respostas com <strong>CORS</strong> adequado.</li>
    </ul>
  </li>
  <li><strong>Supabase</strong> armazena os dados (<code>posts</code>) e pode aplicar <strong>RLS</strong>.</li>
  <li><strong>Frontend</strong> consome a API via <code>fetch</code>/<code>axios</code> com header <code>Authorization: Bearer &lt;JWT&gt;</code>.</li>
</ul>

<h2 id="auth">Autenticação</h2>
<p>A API usa <strong>Bearer Token (JWT)</strong>:</p>
<pre><code>Authorization: Bearer &lt;seu-jwt&gt;</code></pre>
<p>O backend decodifica o token e usa o claim <code>sub</code> como <code>user_id</code>.</p>
<p><strong>Erros comuns</strong></p>
<ul>
  <li>401 — JWT ausente ou inválido</li>
  <li>401 — JWT expirado</li>
</ul>

<h2 id="cors">CORS</h2>
<p>Pré-flight aceito na mesma rota:</p>
<p><code>OPTIONS /posts</code> → <strong>204 No Content</strong></p>
<p><strong>Headers enviados (resumo)</strong></p>
<ul>
  <li>Access-Control-Allow-Origin: *</li>
  <li>Access-Control-Allow-Methods: GET, POST, PATCH, DELETE, OPTIONS</li>
  <li>Access-Control-Allow-Headers: Content-Type, Authorization, apikey</li>
  <li>Access-Control-Max-Age: 600</li>
  <li>Vary: Origin</li>
</ul>
<p><em>Em produção, ajuste <code>Allow-Origin</code> para o(s) domínio(s) do seu frontend.</em></p>

<h2 id="endpoints">Endpoints</h2>

<h3 id="get-posts">GET <code>/posts</code> — Listar posts</h3>
<p>Lista os posts do <strong>usuário autenticado</strong> (filtrados por <code>user_id</code>).</p>

<h3 id="post-posts">POST <code>/posts</code> — Criar post</h3>
<p>Cria um novo post associado ao <code>user_id</code> do JWT.</p>

<h2 id="schemas">Modelos (Schemas)</h2>
<pre><code>{
  id: number;
  title: string;     // ≤ 140
  content: string;   // ≤ 5000
  user_id: string;   // claim `sub` do JWT
  created_at: string; // ISO date-time
}</code></pre>

<h2 id="paginacao">Paginação</h2>
<ul>
  <li><code>page</code> inicia em 1.</li>
  <li><code>limit</code> entre 1 e 100.</li>
</ul>

<h2 id="codigos">Códigos de resposta</h2>
<ul>
  <li>200 — Sucesso (GET)</li>
  <li>201 — Criado (POST)</li>
  <li>204 — Preflight CORS</li>
  <li>400 — Requisição inválida</li>
  <li>401 — Não autorizado (JWT)</li>
  <li>500 — Erro interno</li>
</ul>

<h2 id="como-testar">Como testar</h2>

<h3 id="via-postman">Via Postman</h3>
<p>Importe a coleção e configure <code>jwt</code>, <code>page</code>, <code>limit</code>.</p>

<h3 id="via-curl">Via cURL</h3>
<pre><code>curl -X GET 'https://primary-production-e91c.up.railway.app/webhook/posts?page=1&amp;limit=10' \
  -H 'Authorization: Bearer &lt;JWT&gt;'</code></pre>

<h3 id="via-swagger">Via OpenAPI/Swagger</h3>
<p>Abra <a href="sandbox:/mnt/data/openapi-n8n-onboarding.yml">openapi-n8n-onboarding.yml</a> em Swagger UI.</p>

<h2 id="seguranca">Notas sobre segurança e RLS</h2>
<ul>
  <li>Habilite <strong>RLS</strong> no Supabase em <code>posts</code>.</li>
  <li>Prefira whitelist de origens no CORS.</li>
</ul>

<h2 id="faq">FAQ rápido</h2>
<p><strong>1) Por que recebo 401?</strong> — Token inválido/expirado.</p>
<p><strong>2) O <code>user_id</code> vem de onde?</strong> — Do claim <code>sub</code> do JWT.</p>

<h2 id="licenca">Licença</h2>
<p>Este projeto segue a licença definida pelo repositório.</p>

</body>
</html>
