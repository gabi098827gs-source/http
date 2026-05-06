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

Oooo cabeçalho Content-Length não está presente na resposta, pois foi utilizado o cabeçalho Transfer-Encoding: chunked, indicando que o corpo foi enviado em blocos de tamanho variável. O corpo retornado é um documento HTML, conforme identificado pelo cabeçalho Content-Type: text/html, mas ele se apresenta inicialmente em formato binário comprimido devido ao uso de Content-Encoding: gzip. Essa característica é confirmada visualmente pelos caracteres ilegíveis na aba "Raw" e pelo aviso do Fiddler informando que o corpo da resposta precisa ser decodificado.

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

<img width="1426" height="634" alt="image" src="https://github.com/user-attachments/assets/9ae2e3b4-96d2-4b1c-9e54-e27cc6f28f8e" />


| # | Método | URL | Status-line | `Content-Length` / `Transfer-Encoding` | Body presente? |
|---|--------|-----|-------------|-----------------------------------------|----------------|
| 1 | GET    | [http://httpbin.org/status/200](http://httpbin.org/status/200) | [HTTP/1.1 200 OK] | [0] | [não] |
| 2 | GET    | [http://httpbin.org/redirect-to](http://httpbin.org/redirect-to)... | [HTTP/1.1 301 Moved Permanently] | [0] | [não] |
| 3 | GET    | `[http://httpbin.org/status/404](http://httpbin.org/status/404)` | [HTTP/1.1 418 I'M A TEAPOT] | [135] | [sim] |
| 4 | GET    | `[http://httpbin.org/status/418](http://httpbin.org/status/418)` | [HTTP/1.1 500 INTERNAL SERVER ERROR] | [0] | [não] |
| 5 | GET    | `[http://httpbin.org/status/500](http://httpbin.org/status/500)` | [HTTP/1.1 503 SERVICE UNAVAILABLE] | [0] | [não] |
| 6 | GET    | `[http://httpbin.org/status/503](http://httpbin.org/status/503)` | [HTTP/1.1 304 Not Modified] | [0 | [não] |
| 7 | GET    | `http://example.com/` com `If-Modified-Since` | [...] | [...] | [sim/não] |

### Pergunta 4.1
> Em qual dos status o corpo está ausente/tamanho zero? Isso é obrigatório pela especificação ou depende do servidor?

**Resposta:** [o teste realizado, o corpo está ausente nos status 200, 301, 404, 500, 503 (especificamente nesta ferramenta httpbin) e obrigatoriamente no 304.

De acordo com a especificação HTTP (RFC 9110), a ausência de corpo é obrigatória para o status 304 (Not Modified), bem como para as respostas 1xx e 204. Para os outros status (como 404 ou 500), a presença de um corpo (uma página de erro, por exemplo) depende da implementação do servidor, embora o httpbin escolha retornar vazio para simplificação.]

### Pergunta 4.2
> No `301`, qual cabeçalho da resposta informa para onde ir? O que aconteceria se estivesse ausente?

**Resposta:** [Cabeçalho: O cabeçalho é o Location.

O que aconteceria: Se o cabeçalho Location estivesse ausente em um status 301 (Moved Permanently), o navegador ficaria "perdido". Como o código 301 indica um redirecionamento, sem a URL de destino, o navegador não saberia para onde encaminhar o usuário, resultando geralmente em uma página de erro do próprio browser ou em uma requisição interrompida.]

### Pergunta 4.3
> Diferença semântica entre `200`, `304` e `404` do ponto de vista do cache do navegador.

**Resposta:** [a diferença fundamental reside em como o navegador valida a necessidade de baixar o conteúdo novamente:

200 OK (Carga Total): O navegador entende que o recurso foi obtido com sucesso do servidor. Se o cache estiver vazio ou expirado, o servidor envia o corpo completo do arquivo. O navegador então armazena essa nova versão para uso futuro.

304 Not Modified (Validação de Cache): É o status mais importante para a eficiência. O navegador pergunta ao servidor: "Eu tenho uma versão guardada aqui, ela ainda vale?". Se o servidor responder 304, ele não envia o corpo do arquivo (tamanho zero), economizando banda. O navegador, então, usa a cópia que já possui no cache local.

404 Not Found (Invalidação/Erro): Semanticamente, informa que o recurso não existe mais naquela URL. Para o cache, isso significa que qualquer versão anteriormente armazenada para esse endereço deve ser considerada inválida ou descartada, e o navegador não exibirá conteúdo antigo (obsoleto) para o usuário.]

---

## Atividade 5 — Identificação de cabeçalhos (`http://httpbin.org/response-headers?...` + `/gzip`)

**Captura de tela (Inspectors → Headers):** `evidencias/atv5_headers.png`
<img width="1537" height="771" alt="image" src="https://github.com/user-attachments/assets/a1336851-a4ec-4bf2-9000-584cd1c64d1c" />


| Cabeçalho                    | Req/Resp | Valor capturado | Função em uma frase |
|------------------------------|----------|------------------|----------------------|
| `Host`                       | [Req]    | [...httpbin.org]            | [Indica o nome de domínio do servidor para o qual a requisição está a ser enviada.]                |
| `User-Agent`                 | [Req]    | [Mozilla/5.0 (Windows NT 10.0; ]            | [Identifica o navegador e o sistema operativo do cliente que faz a requisição]                |
| `Accept`                     | [Req]    | [...text/html, application/xhtml+xml...]            | [ndica quais formatos de conteúdo o navegador está preparado para receber.]                |
| `Accept-Encoding`            | [Req]    | [gzip, deflate]            | [Comunica ao servidor quais métodos de compressão o cliente suporta para diminuir o tráfego.]                |
| `Cookie`                     | [Req]    | []            | [Envia informações de estado ou sessão previamente armazenadas para o servid]                |
| `Server`                     | [Resp]    | [gunicorn/19.9.0]            | [dentifica o software de servidor web que gerou a resposta.]                |
| `Content-Type`               | [Resp]    | [application/json]            | [Especifica que o formato do corpo da resposta enviada é um objeto JSON]                |
| `Content-Encoding`           | [Resp]    | []            | [Mostra o tipo de compressão (como gzip) que foi aplicado ao corpo da mensagem.]                |
| `Set-Cookie`                 | [Resp]    | []            | [Utilizado pelo servidor para solicitar que o navegador armazene um novo cookie.]                |
| `Cache-Control`              | [Resp]    | [no-cache]            | [Indica que o navegador deve validar com o servidor se o conteúdo mudou antes de usar uma cópia em cache.]                |
| `Strict-Transport-Security`  | [Resp]    | []            | [Força o uso exclusivo de conexões seguras (HTTPS) para o domínio em visitas futuras]                |

### Pergunta 5.1
> `Content-Encoding: gzip`/`br` apareceu? Compare `Content-Length`, quando presente, com o conteúdo visível. O que explica a diferença?

**Resposta:** [Não, o cabeçalho Content-Encoding não apareceu na resposta analisada. O valor capturado para o Content-Length foi de 123 bytes. Como não houve compressão, o Content-Length corresponde exatamente ao tamanho do corpo da mensagem entregue. Caso o Content-Encoding: gzip estivesse presente, o Content-Length indicaria o tamanho do arquivo compactado (menor), enquanto o conteúdo visível no navegador (após a descompressão) seria maior. O que explica essa diferença é o mecanismo de compressão do HTTP, que visa reduzir o tráfego de dados na rede.]

### Pergunta 5.2
> Cliente envia `Accept: application/json` mas o recurso só existe em `text/html`. Qual status code esperar?

**Resposta:** [O status code esperado é o 406 Not Acceptable. Este código indica que o servidor não consegue gerar uma resposta que atenda aos critérios de formato (MIME types) especificados pelo cliente no cabeçalho Accept da requisição]

### Pergunta 5.3
> `Strict-Transport-Security` apareceu nas respostas HTTP? Por que esse cabeçalho está ausente neste fluxo? (Consulte a RFC 6797.) Qual é seu papel contra downgrades para HTTP puro?

**Resposta:** [Não, o cabeçalho Strict-Transport-Security (HSTS) não apareceu. De acordo com a RFC 6797, este cabeçalho está ausente porque a conexão foi realizada via HTTP puro (porta 80). A especificação determina que os navegadores devem ignorar o HSTS se ele não for enviado através de uma conexão segura (HTTPS/TLS) para evitar que atacantes injetem cabeçalhos falsos. O papel do HSTS contra downgrades é instruir o navegador a converter automaticamente qualquer tentativa de acesso inseguro (http://) em seguro (https://) antes mesmo da requisição sair do computador do usuário, impedindo ataques que tentam forçar a comunicação em texto claro.]

---

## Atividade 6 — HTTP vs HTTPS (análise sem decriptação)

**Captura de tela HTTP (`neverssl.com`):** `evidencias/atv6_http.png`
<img width="1573" height="803" alt="image" src="https://github.com/user-attachments/assets/f0873009-69e5-4876-859a-03bda49f5c37" />

**Captura de tela HTTPS (`https://httpbin.org/get`, apenas CONNECT):** `evidencias/atv6_https.png`
<img width="469" height="339" alt="image" src="https://github.com/user-attachments/assets/080e06cc-a221-436e-8d95-09c4d22d64bf" />


### Pergunta 6.1
> Que método HTTP aparece na sessão do `https://httpbin.org/get`? O que ele faz e por que existe?

**Resposta:** [O método exibido corresponde ao CONNECT, apresentado no Fiddler como “Tunnel to”. Ele é utilizado para estabelecer um túnel TCP entre o cliente e o servidor, permitindo a comunicação segura via HTTPS. Esse método existe para que o proxy apenas intermedie a conexão criptografada, sem acessar o conteúdo da comunicação.]

### Pergunta 6.2
> Tabela comparativa dos campos visíveis ao Fiddler em cada caso:

| Campo                          | Visível em HTTP? | Visível em HTTPS (sem decriptação)? |
|--------------------------------|------------------|-------------------------------------|
| Método                         | [sim]            | [nao]                               |
| URL completa (path + query)    | [...[sim]            | [...[nao]                               |
| Cabeçalhos de request          | [...[sim]            | [...[nao]                               |
| Corpo de request               | [...[sim]            | [...[nao]                               |
| Status code                    | [...[sim]            | [...[nao]                               |
| Cabeçalhos de response         | [...[sim]            | [...[nao]                               |
| Corpo de response              | [...[sim]            | [...[nao]                               |
| Host (via SNI, no `CONNECT`)   | [...[sim]            | [...[sim]                               |
| IP e porta de destino          | [...[sim]            | [...[sim]                               |

### Pergunta 6.3 (teórica)
> O que você **veria** no Fiddler se tivesse privilégio de administrador e pudesse habilitar *Decrypt HTTPS traffic*? Indique telas/abas e justifique por que essa inspeção exige a instalação de um certificado raiz.

**Resposta:** [e a opção Decrypt HTTPS traffic estivesse habilitada, o Fiddler passaria a exibir todas as informações da comunicação HTTPS, assim como ocorre no HTTP. Seria possível visualizar o método (GET/POST), a URL completa (com path e query), os cabeçalhos e o corpo da requisição e da resposta. Essas informações apareceriam nas abas Inspectors → Raw, Headers e JSON.
Essa inspeção exige a instalação de um certificado raiz porque o proxy atua como um intermediário confiável (man-in-the-middle controlado), descriptografando e recriptografando os dados. Para que o navegador aceite essa interceptação sem erro de segurança, ele precisa confiar no certificado emitido pelo proxy.]

### Pergunta 6.4
> Por que a técnica de decriptação dos *debugging proxies* **não** funcionaria contra um usuário se um atacante a tentasse sem instalar o certificado?

**Resposta:** [A técnica não funcionaria sem a instalação do certificado porque os navegadores validam a autenticidade dos certificados digitais durante a conexão HTTPS. Se um atacante tentasse interceptar a comunicação sem que o certificado dele estivesse instalado como confiável, o navegador exibiria um aviso de segurança e bloquearia a conexão. Isso impede que terceiros consigam descriptografar o conteúdo sem o consentimento do usuário.]

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
<img width="1417" height="788" alt="image" src="https://github.com/user-attachments/assets/729c2183-2b80-451a-859f-c3274eb277a6" />




**JSON de resposta após edição:**

```json
{
  "user-agent": "[valor forjado]"
}
```

### Pergunta 8.1
> O servidor pode detectar que o `User-Agent` foi forjado? Discuta.

**Resposta:** [Sim, o servidor pode detectar a falsificação, embora não seja algo imediato apenas pela leitura do cabeçalho User-Agent. A detecção geralmente ocorre através de uma técnica chamada Fingerprinting]

### Pergunta 8.2
> Após editar a status-line de `200 OK` para `404 Not Found`, o que o navegador exibe? Comente o papel do proxy como MITM.

**Captura de tela:** `evidencias/atv8_status_edit.png`
<img width="1520" height="738" alt="image" src="https://github.com/user-attachments/assets/f6480f78-b87b-4c04-8e6e-db357ece2bab" />


**Resposta:** [pós alterar a status-line de 200 OK para 404 Not Found, o navegador passa a exibir uma página de erro indicando que o recurso não foi encontrado, mesmo que o servidor tenha retornado originalmente sucesso. Isso ocorre porque o proxy intercepta e modifica a resposta antes que ela chegue ao navegador. Nesse contexto, o proxy atua como um intermediário do tipo man-in-the-middle (MITM), sendo capaz de visualizar e alterar tanto requisições quanto respostas.
]

### Pergunta 8.3
> Confirme que todos os breakpoints foram desabilitados.

- [ -] Breakpoints desabilitados ao final (Shift+F11)

---

## Atividade 9 — Redirecionamento HTTP → HTTPS

**Captura de tela:** `evidencias/atv9_redir.png`
<img width="1349" height="757" alt="image" src="https://github.com/user-attachments/assets/2e1339ea-643e-452e-b02e-ea71f6853874" />




### Pergunta 9.1
> Código de status e cabeçalho que direcionaram o navegador para `https://`.

**Resposta:** [Status-line: HTTP/1.1 301 Moved Permanently  Cabeçalho Location: https://httpbin.org/get]

### Pergunta 9.2
> Além do redirecionamento 3xx, qual outro mecanismo/cabeçalho faz o navegador passar a forçar HTTPS em visitas futuras? Cite a RFC.

**Resposta:** [O cabeçalho é Strict-Transport-Security (HSTS).
Ele informa ao navegador que todas as futuras conexões com aquele domínio devem usar HTTPS, mesmo que o usuário digite “http://”.]

### Pergunta 9.3
> Se esse cabeçalho fosse enviado por uma resposta servida via HTTP puro, o navegador deveria obedecer? Justifique com base na RFC.

**Resposta:** [Não.
Segundo a RFC 6797, o cabeçalho Strict-Transport-Security só é aceito quando recebido por uma conexão HTTPS válida.
Se for enviado por HTTP puro, o navegador ignora o cabeçalho, justamente para evitar ataques de downgrade ou falsificação de políticas de segurança.]

---

## Questões de Verificação

### 1. Ordem dos elementos em uma mensagem HTTP/1.1. O que separa cabeçalhos do corpo?

[ mensagem HTTP segue esta ordem:

Linha inicial (Status-Line ou Request-Line)

Cabeçalhos (Headers)

Corpo (Body)

Os cabeçalhos são separados do corpo por uma linha em branco — ou seja, um par de caracteres CRLF (\r\n\r\n).]

### 2. Por que `Host` é obrigatório em HTTP/1.1 mas era opcional em HTTP/1.0?

[Em HTTP/1.0 cada servidor geralmente atendia apenas um site por IP, então o Host era opcional.
Já em HTTP/1.1, com o uso de virtual hosts (vários domínios no mesmo IP), o cabeçalho Host tornou-se obrigatório para o servidor saber qual site o cliente quer acessar.]

### 3. Diferença entre `401 Unauthorized` e `403 Forbidden`.

[401 Unauthorized: o cliente não está autenticado ou forneceu credenciais inválidas; o servidor pede autenticação.

403 Forbidden: o cliente está autenticado, mas não tem permissão para acessar o recurso.]

### 4. Um `POST` enviado duas vezes produz o mesmo efeito? E um `PUT`? Justifique em termos de idempotência.

[POST: não é idempotente — enviar duas vezes pode criar dois recursos ou duplicar ações.

PUT: é idempotente — enviar o mesmo conteúdo várias vezes produz o mesmo resultado (substitui o recurso pelo mesmo estado).]

### 5. Por que HTTPS permite ainda que um observador saiba qual site está sendo visitado? (SNI, DNS)

[Mesmo com HTTPS, o nome do host pode ser revelado por:

SNI (Server Name Indication): o nome do domínio é enviado em texto claro durante o handshake TLS.

DNS: as consultas de resolução de nome geralmente não são criptografadas (a menos que use DNS over HTTPS ou DNS over TLS).
Por isso, um observador ainda pode saber qual site está sendo acessado.]

### 6. O que muda com `Content-Encoding: gzip`? Onde os dados são compactados e descompactados?

[resposta]

### 7. Impacto prático de `Cache-Control: no-store`.

[Esse cabeçalho instrui o navegador e proxies a não armazenar nenhuma parte da resposta em cache.
Impacto prático: cada vez que o recurso for acessado, o cliente terá que buscar novamente no servidor. É usado em dados sensíveis (ex.: páginas de login, informações bancárias) para evitar que fiquem guardados em disco ou memória.]

### 8. Como um debugging proxy decifra HTTPS sem violar a criptografia, e por que isso exige cooperação do usuário (e por que, justamente, você não pôde executar essa etapa)?

Um proxy de debugging (como Fiddler ou Burp) atua como “man-in-the-middle” autorizado:

Ele gera um certificado raiz falso e o usuário instala esse certificado no sistema.

Assim, o navegador confia no proxy como se fosse o servidor real.

O proxy estabelece duas conexões TLS: uma com o cliente e outra com o servidor, descriptografando e recriptografando no meio.[]

### 9. Exemplo de cabeçalho de request que o navegador envia automaticamente, sem a página pedir.

[Um exemplo é o User-Agent, que identifica o navegador e sistema operacional.
Outro exemplo comum: Accept-Language, indicando o idioma preferido do usuário.]

### 10. Se fosse automatizar a inspeção via script, qual ferramenta alternativa escolheria? Por quê?

[Uma alternativa seria o cURL ou o Python com a biblioteca requests.

São ferramentas leves, automatizáveis e permitem capturar headers e corpo de respostas facilmente.

Diferente de proxies gráficos, podem ser integradas em scripts para testes repetitivos ou pipelines.]

### 11. (Exclusiva do Fluxo B) Três cabeçalhos de segurança que não aparecem ou não fazem sentido em respostas HTTP puro. Para cada um, o que aconteceria se enviado por um servidor HTTP? (Cite RFC 6797 para HSTS.)

**Resposta:**

| Cabeçalho | Comportamento esperado sobre HTTP | Referência |
|-----------|-----------------------------------|-----------|
| [...Strict-Transport-Security (HSTS)]     | [...gnorado pelo navegador se enviado via HTTP, pois só é aceito em conexões HTTPS válidas. Evita ataques de downgrade.]                             | [...RFC 6797]     |
| [...Set-Cookie: ...; Secure]     | [...O navegador não armazena cookies com a flag Secure se recebidos por HTTP, já que só devem ser enviados em conexões HTTPS.]                             | [...RFC 6265]     |
| [...Upgrade-Insecure-Requests]     | [...Não faz sentido em HTTP puro, pois instrui o navegador a converter recursos inseguros em HTTPS. Se não houver HTTPS, não há efeito prático.]                             | [...****]     |

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
A remoção do certificado não é necessária neste fluxo porque o navegador já voltou ao modo seguro (HTTPS-First/HTTPS-Only) e não há proxy ativo interceptando conexões. Como o Fiddler/mitmproxy/HTTP Toolkit foi fechado e a configuração de proxy desativada, não existe risco de manter certificados falsos confiados. Para um aluno administrador, porém, seria obrigatório remover o certificado raiz instalado, pois esse permanece válido no sistema e poderia ser explorado por terceiros se não fosse revogado.

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
