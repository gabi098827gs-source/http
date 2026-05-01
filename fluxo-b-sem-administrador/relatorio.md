# Relatório — Laboratório de Inspeção HTTP/HTTPS — Fluxo B (sem privilégio administrativo)

> **Como usar este template:** substitua os campos `[...]` pelas suas respostas,
> anexe as capturas de tela na pasta `evidencias/` e referencie-as onde indicado.
> Preserve a formatação markdown (tabelas, blocos de código) para facilitar a correção.
>
> **Observação:** toda a análise prática deste relatório é feita sobre tráfego **HTTP em texto claro**. A análise de HTTPS é teórica, baseada na fundamentação do `readme.md` do repositório.

---

## Identificação

| Campo       | Valor                  |
|-------------|------------------------|
| Nome        | [seu nome completo]    |
| RA          | [seu RA]               |
| Disciplina  | Redes de Computadores  |
| Turma       | [sua turma]            |
| Data        | [data da realização]   |
| Fluxo       | **B — Aluno sem privilégio de administrador** |
| SO utilizado | [Windows 11 / Ubuntu 22.04 / macOS ...] |
| Ferramenta de proxy | [Fiddler Classic per-user / mitmproxy / HTTP Toolkit / ...] |
| Navegador(es)       | [Chrome 124 / Firefox 125 / ...] |
| HTTPS-First Mode / HTTPS-Only desabilitado? | [sim / não] |

---

## Atividade 1 — Primeira captura (`http://example.com`)

**Captura de tela:** `evidencias/atv1_sessao.png`
<img width="1596" height="790" alt="image" src="https://github.com/user-attachments/assets/de57b7ee-7cf9-4cc7-b900-b7fa28a4301d" />


**Request-line enviada:**

```http
[colar aqui a linha inicial do request, ex: GET / HTTP/1.1]
```

**Status-line recebida:**

```http
[colar aqui, ex: HTTP/1.1 200 OK]
<img width="1596" height="790" alt="image" src="https://github.com/user-attachments/assets/7d6a3036-9ce0-4e76-904d-0760259c2cbf" />

```

### Pergunta 1.1
> Quantos cabeçalhos o navegador enviou no request? Liste-os.

**Resposta:**
[7]

Cabeçalhos:

Host: example.com

Connection: keep-alive

Upgrade-Insecure-Requests: 1

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.7

Accept-Encoding: gzip, deflate

Accept-Language: pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7
### Pergunta 1.2
> Qual foi o `Content-Length` da resposta? Se ele não apareceu, registre `Transfer-Encoding`, versão do protocolo ou outro indício observado. O corpo retornado é HTML, texto puro, JSON ou binário? Como você descobriu?

O cabeçalho Content-Length não está presente na resposta, pois foi utilizado o cabeçalho Transfer-Encoding: chunked, indicando que o corpo foi enviado em blocos de tamanho variável. O corpo retornado é um documento HTML, conforme identificado pelo cabeçalho Content-Type: text/html, mas ele se apresenta inicialmente em formato binário comprimido devido ao uso de Content-Encoding: gzip. Essa característica é confirmada visualmente pelos caracteres ilegíveis na aba "Raw" e pelo aviso do Fiddler informando que o corpo da resposta precisa ser decodificado.

## Atividade 2 — Anatomia de um GET (`http://httpbin.org/get?...`)

**Captura de tela:** `evidencias/atv2_raw.png`
<img width="1494" height="799" alt="image" src="https://github.com/user-attachments/assets/11631963-fe70-428c-b121-a42c582896e8" />


**Request-line completa:**

```http
GET /get?aluno=Maria&curso=redes HTTP/1.1
```

**Cabeçalhos-chave capturados:**

| Cabeçalho    | Valor                    |
|--------------|--------------------------|
| `Host`       | [httpbin.org]                    |
| `User-Agent` | [Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36]                    |
| `Accept`     | [text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,/;q=0.8,application/signed-exchange;v=b3;q=0.7]                    |

**Campos do JSON de resposta:**

```json
{
  "args": {
    "aluno": "Maria",
    "curso": "redes"
  },
  "headers": {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/138.0.0.0"
  },
  "origin": "179.157.196.34"
}
```

### Pergunta 2.1
> O valor do campo `origin` corresponde a qual elemento da rede? Por que normalmente não é o IP local?

**Resposta:** [O campo origin corresponde ao IP público da requisição na internet. Ele não é o IP local da máquina porque a comunicação passa por um roteador com NAT (Network Address Translation), que substitui o IP privado (ex: 192.168.x.x) por um IP público ao acessar a internet.]

### Pergunta 2.2
> Compare o `User-Agent` enviado com o que aparece no JSON da resposta. Coincidem?

**Resposta:** [Sim, coincidem. O valor do User-Agent enviado pelo navegador é o mesmo que aparece no JSON da resposta, pois o servidor apenas retorna os cabeçalhos recebidos na requisição.]

### Pergunta 2.3
> Em `http://httpbin.org/headers`, liste até três cabeçalhos que o servidor vê mas **não aparecem** no Raw do request. De onde vêm? Se não encontrar três, explique por que o resultado pode variar.

**Resposta:**

| Cabeçalho visto pelo servidor | Origem provável | Observação |
|-------------------------------|-----------------|------------|
| [X-Forwarded-For]                         | [roxy/rede (ou infraestrutura do servidor)]           | [Indica o IP original do cliente]      |
| [X-Amzn-Trace-Id]                         | [Infraestrutura do servidor (AWS)]           | [Usado para rastreamento de requisições]      |
| [Via]                         | [Proxy intermediário]           | [Indica que a requisição passou por intermediários]      |

---

## Atividade 3 — POST e envio de formulário (`http://httpbin.org/forms/post` → `/post`)

**Captura de tela:** `evidencias/atv3_post_raw.png`
<img width="1490" height="796" alt="image" src="https://github.com/user-attachments/assets/25f19fcf-2bbd-4891-b527-322a49e17972" />


**Request-line do POST:**

```http
POST /post HTTP/1.1
```

**Cabeçalhos do request:**

| Cabeçalho        | Valor |
|------------------|-------|
| `Content-Type`   | [application/x-www-form-urlencoded] |
| `Content-Length` | [112] |

**Corpo completo do request:**

```
custname=Maria&custtel=13981465780&custemail=mm6@gmail.com&size=medium&topping=onion&delivery=20%3A00&comments=
```

**Trecho do JSON de resposta (campo `form`):**

```json
"form": {
  "custname": "Maria",
  "custtel": "13981465780",
  "custemail": "mm6@gmail.com",
  "size": "medium",
  "topping": "onion",
  "delivery": "20:00",
  "comments": ""
}
```

### Pergunta 3.1
> Qual o formato do corpo? Como esse formato codifica caracteres especiais (espaço, acentos)?

**Resposta:** [O formato do corpo é application/x-www-form-urlencoded. Nesse formato, os dados são enviados como pares chave=valor separados por &. Caracteres especiais são codificados usando percent-encoding, por exemplo: espaço pode ser representado como + ou %20, e caracteres como : são codificados como %3A.]

### Pergunta 3.2
> Comparando **Request → WebForms** e **Request → Raw**: qual das duas corresponde literalmente aos bytes enviados no socket TCP?

**Resposta:** [A aba Raw corresponde literalmente aos bytes enviados no socket TCP, pois mostra a requisição exatamente como foi transmitida. Já a aba WebForms é apenas uma representação organizada dos dados para facilitar a leitura]

### Pergunta 3.3 — Composer
> Envie manualmente via Composer um `POST` para `http://httpbin.org/post` com JSON. Registre a resposta. Qual campo do JSON confirma que o servidor interpretou o JSON?

**Captura de tela:** `evidencias/atv3_composer.png`
<img width="1498" height="738" alt="image" src="https://github.com/user-attachments/assets/c8a852b7-b8f2-4d0b-8ff6-a998e390840d" />



**Response JSON (trecho relevante):**

```json
{
  "json": {
    "protocolo": "HTTP",
    "versao": "1.1"
  }
}
```

**Resposta:** [O campo json confirma que o servidor recebeu e interpretou corretamente o corpo da requisição, pois ele retorna exatamente os dados enviados no formato JSON.]

---

## Atividade 4 — Catálogo de status codes (`http://httpbin.org/...`)

**Captura de tela (lista do Fiddler com as 7 sessões):** `evidencias/atv4_lista.png`

| # | Método | URL | Status-line | `Content-Length` / `Transfer-Encoding` | Body presente? |
|---|--------|-----|-------------|-----------------------------------------|----------------|
| 1 | GET    | `http://httpbin.org/status/200` | [...] | [...] | [sim/não] |
| 2 | GET    | `http://httpbin.org/redirect-to?status_code=301&url=/get` | [...] | [...] | [sim/não] |
| 3 | GET    | `http://httpbin.org/status/404` | [...] | [...] | [sim/não] |
| 4 | GET    | `http://httpbin.org/status/418` | [...] | [...] | [sim/não] |
| 5 | GET    | `http://httpbin.org/status/500` | [...] | [...] | [sim/não] |
| 6 | GET    | `http://httpbin.org/status/503` | [...] | [...] | [sim/não] |
| 7 | GET    | `http://example.com/` com `If-Modified-Since` | [...] | [...] | [sim/não] |

### Pergunta 4.1
> Em qual dos status o corpo está ausente/tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Resposta:** [...]

### Pergunta 4.2
> No `301`, qual cabeçalho da resposta informa para onde ir? O que aconteceria se estivesse ausente?

**Resposta:** [...]

### Pergunta 4.3
> Diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

**Resposta:** [...]

---

## Atividade 5 — Identificação de cabeçalhos (`http://httpbin.org/response-headers?...` + `/gzip`)

**Captura de tela (Inspectors → Headers):** `evidencias/atv5_headers.png`

| Cabeçalho                    | Req/Resp | Valor capturado | Função em uma frase |
|------------------------------|----------|------------------|----------------------|
| `Host`                       | [...]    | [...]            | [...]                |
| `User-Agent`                 | [...]    | [...]            | [...]                |
| `Accept`                     | [...]    | [...]            | [...]                |
| `Accept-Encoding`            | [...]    | [...]            | [...]                |
| `Cookie`                     | [...]    | [...]            | [...]                |
| `Server`                     | [...]    | [...]            | [...]                |
| `Content-Type`               | [...]    | [...]            | [...]                |
| `Content-Encoding`           | [...]    | [...]            | [...]                |
| `Set-Cookie`                 | [...]    | [...]            | [...]                |
| `Cache-Control`              | [...]    | [...]            | [...]                |
| `Strict-Transport-Security`  | [...]    | [...]            | [...]                |

### Pergunta 5.1
> `Content-Encoding: gzip`/`br` apareceu? Compare `Content-Length`, quando presente, com o conteúdo visível. O que explica a diferença?

**Resposta:** [...]

### Pergunta 5.2
> Cliente envia `Accept: application/json` mas o recurso só existe em `text/html`. Qual status code esperar?

**Resposta:** [...]

### Pergunta 5.3
> `Strict-Transport-Security` apareceu nas respostas HTTP? Por que esse cabeçalho está ausente neste fluxo? (Consulte a RFC 6797.) Qual é seu papel contra downgrades para HTTP puro?

**Resposta:** [...]

---

## Atividade 6 — HTTP vs HTTPS (análise sem decriptação)

**Captura de tela HTTP (`neverssl.com`):** `evidencias/atv6_http.png`
**Captura de tela HTTPS (`https://httpbin.org/get`, apenas CONNECT):** `evidencias/atv6_https.png`

### Pergunta 6.1
> Que método HTTP aparece na sessão do `https://httpbin.org/get`? O que ele faz e por que existe?

**Resposta:** [...]

### Pergunta 6.2
> Tabela comparativa dos campos visíveis ao Fiddler em cada caso:

| Campo                          | Visível em HTTP? | Visível em HTTPS (sem decriptação)? |
|--------------------------------|------------------|-------------------------------------|
| Método                         | [...]            | [...]                               |
| URL completa (path + query)    | [...]            | [...]                               |
| Cabeçalhos de request          | [...]            | [...]                               |
| Corpo de request               | [...]            | [...]                               |
| Status code                    | [...]            | [...]                               |
| Cabeçalhos de response         | [...]            | [...]                               |
| Corpo de response              | [...]            | [...]                               |
| Host (via SNI, no `CONNECT`)   | [...]            | [...]                               |
| IP e porta de destino          | [...]            | [...]                               |

### Pergunta 6.3 (teórica)
> O que você **veria** no Fiddler se tivesse privilégio de administrador e pudesse habilitar *Decrypt HTTPS traffic*? Indique telas/abas e justifique por que essa inspeção exige a instalação de um certificado raiz.

**Resposta:** [...]

### Pergunta 6.4
> Por que a técnica de decriptação dos *debugging proxies* **não** funcionaria contra um usuário se um atacante a tentasse sem instalar o certificado?

**Resposta:** [...]

---

## Atividade 7 — Cookies e sessão (`http://httpbin.org/cookies/...`)

**Captura de tela da sequência:** `evidencias/atv7_cookies.png`

| # | URL | `Set-Cookie` recebido | `Cookie` enviado |
|---|-----|-----------------------|-------------------|
| 1 | `/cookies/set?...`       | [...] | [nenhum / ...] |
| 2 | `/cookies` (1ª visita)   | [...] | [...]          |
| 3 | `/cookies` (reload 1)    | [...] | [...]          |
| 4 | `/cookies` (reload 2)    | [...] | [...]          |

### Pergunta 7.1
> `Set-Cookie` aparece uma vez ou em toda requisição? Justifique.

**Resposta:** [...]

### Pergunta 7.2
> Que atributos o `Set-Cookie` trouxe? Explique cada um presente. Para atributos não observados, registre `não observado`.

**Resposta:**

| Atributo | Valor | Função |
|----------|-------|--------|
| [...]    | [...] | [...]  |

### Pergunta 7.3
> O atributo `Secure` pode aparecer num cookie recebido por HTTP puro? Qual seria o comportamento esperado? Relacione com o fato de que todo o tráfego desta atividade é visível em texto claro.

**Resposta:** [...]

### Pergunta 7.4
> Na aba **Inspectors → Cookies**, o cookie armazenado coincide com o campo `cookies` do JSON?

**Resposta:** [...]

---

## Atividade 8 — Manipulação com breakpoints

**Captura de tela da edição do User-Agent:** `evidencias/atv8_ua_edit.png`

**JSON de resposta após edição:**

```json
{
  "user-agent": "[valor forjado]"
}
```

### Pergunta 8.1
> O servidor pode detectar que o `User-Agent` foi forjado? Discuta.

**Resposta:** [...]

### Pergunta 8.2
> Após editar a status-line de `200 OK` para `404 Not Found`, o que o navegador exibe? Comente o papel do proxy como MITM.

**Captura de tela:** `evidencias/atv8_status_edit.png`

**Resposta:** [...]

### Pergunta 8.3
> Confirme que todos os breakpoints foram desabilitados.

- [ ] Breakpoints desabilitados ao final (Shift+F11)

---

## Atividade 9 — Redirecionamento HTTP → HTTPS

**Captura de tela:** `evidencias/atv9_redir.png`

**Status-line da resposta a `http://httpbin.org/redirect-to?status_code=301&url=https%3A%2F%2Fhttpbin.org%2Fget`:**

```http
[colar aqui, ex: HTTP/1.1 301 Moved Permanently]
```

**Cabeçalho `Location` da resposta:**

```
Location: [colar aqui]
```

### Pergunta 9.1
> Código de status e cabeçalho que direcionaram o navegador para `https://`.

**Resposta:** [...]

### Pergunta 9.2
> Além do redirecionamento 3xx, qual outro mecanismo/cabeçalho faz o navegador passar a forçar HTTPS em visitas futuras? Cite a RFC.

**Resposta:** [...]

### Pergunta 9.3
> Se esse cabeçalho fosse enviado por uma resposta servida via HTTP puro, o navegador deveria obedecer? Justifique com base na RFC.

**Resposta:** [...]

---

## Questões de Verificação

### 1. Ordem dos elementos em uma mensagem HTTP/1.1. O que separa cabeçalhos do corpo?

[resposta]

### 2. Por que `Host` é obrigatório em HTTP/1.1 mas era opcional em HTTP/1.0?

[resposta]

### 3. Diferença entre `401 Unauthorized` e `403 Forbidden`.

[resposta]

### 4. Um `POST` enviado duas vezes produz o mesmo efeito? E um `PUT`? Justifique em termos de idempotência.

[resposta]

### 5. Por que HTTPS permite ainda que um observador saiba qual site está sendo visitado? (SNI, DNS)

[resposta]

### 6. O que muda com `Content-Encoding: gzip`? Onde os dados são compactados e descompactados?

[resposta]

### 7. Impacto prático de `Cache-Control: no-store`.

[resposta]

### 8. Como um debugging proxy decifra HTTPS sem violar a criptografia, e por que isso exige cooperação do usuário (e por que, justamente, você não pôde executar essa etapa)?

[resposta]

### 9. Exemplo de cabeçalho de request que o navegador envia automaticamente, sem a página pedir.

[resposta]

### 10. Se fosse automatizar a inspeção via script, qual ferramenta alternativa escolheria? Por quê?

[resposta]

### 11. (Exclusiva do Fluxo B) Três cabeçalhos de segurança que não aparecem ou não fazem sentido em respostas HTTP puro. Para cada um, o que aconteceria se enviado por um servidor HTTP? (Cite RFC 6797 para HSTS.)

**Resposta:**

| Cabeçalho | Comportamento esperado sobre HTTP | Referência |
|-----------|-----------------------------------|-----------|
| [...]     | [...]                             | [...]     |
| [...]     | [...]                             | [...]     |
| [...]     | [...]                             | [...]     |

---

## Reflexão final (opcional, até 10 linhas)

> O que você aprendeu que não conhecia antes deste laboratório? Há algum
> cabeçalho, código de status ou comportamento que passou a olhar com
> mais atenção? Alguma dificuldade que recomendaria evitar para a próxima turma?

[reflexão]

---

## Encerramento — justificativa de segurança (Fluxo B)

**Parágrafo: por que a remoção de certificado é dispensável neste fluxo e por que seria obrigatória para o aluno administrador:**

[redigir, em até 5 linhas, com base na seção 4.6 do readme.md]

- [ ] HTTPS-First Mode / HTTPS-Only Mode reabilitado no navegador
- [ ] Fiddler / mitmproxy / HTTP Toolkit fechado (porta de proxy liberada)
- [ ] Configuração de proxy removida do navegador (se aplicável)

---

## Checklist de entrega

- [ ] Todos os campos `[...]` substituídos
- [ ] Pasta `evidencias/` com capturas nomeadas por atividade (incluindo Atv. 9)
- [ ] 11 questões de verificação respondidas
- [ ] Atividade 9 (redirecionamento HTTP→HTTPS) documentada
- [ ] Justificativa de encerramento redigida
- [ ] Arquivo compactado como `NOME_RA_LAB_HTTP_FLUXOB.zip`
- [ ] Submetido no Microsoft Teams dentro do prazo
