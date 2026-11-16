# AulaoSecurity

Laboratório de estudos de **Spring Security** com **OAuth2** e **JWT**, usando o Google como provedor de identidade.

A ideia é entender, na prática, os dois papéis que uma aplicação pode ter no OAuth2 e ver os dois convivendo no mesmo projeto:

| Papel | O que faz | Como aparece aqui |
|---|---|---|
| **OAuth2 Client** (login) | Redireciona o usuário para o Google, recebe o código de autorização e cria uma sessão com cookie. | `oauth2Login()` |
| **Resource Server** (API) | Recebe `Authorization: Bearer <token>` e valida o JWT (assinatura, emissor, expiração) antes de liberar o endpoint. | `oauth2ResourceServer().jwt()` |

## Stack

- Java 21
- Spring Boot 4.1 (Spring Security 7)
- `spring-boot-starter-security-oauth2-client`
- `spring-boot-starter-security-oauth2-resource-server`
- Maven

## Endpoints

| Rota | Acesso | O que mostra |
|---|---|---|
| `GET /public` | livre | Uma página pública, sem autenticação. |
| `GET /private` | autenticado | Uma página que só abre depois do login. |
| `GET /cookie` | autenticado (login via navegador) | Dados do usuário logado: principal, e-mail, authorities e o **ID Token (JWT)** emitido pelo Google. |
| `GET /jwt` | autenticado | Rota para testar o acesso com Bearer token. |

## Configuração

As credenciais do Google **não ficam no código**. O `application.yaml` lê `GOOGLE_CLIENT_ID` e `GOOGLE_CLIENT_SECRET`, que podem vir de:

1. **Arquivo `.env` na raiz do projeto** (mais prático no dia a dia). Ele é carregado com `spring.config.import` e está no `.gitignore`.
   ```bash
   cp .env.example .env
   # depois preencha os dois valores
   ```
2. **Variáveis de ambiente do sistema** ou da *Run Configuration* do IntelliJ. Elas têm prioridade sobre o `.env`.

Para obter as credenciais, crie um *OAuth client ID* do tipo **Web application** no [Google Cloud Console](https://console.cloud.google.com/apis/credentials) e cadastre esta URI de redirecionamento:

```
http://localhost:8080/login/oauth2/code/google
```

## Como rodar

```bash
./mvnw spring-boot:run
```

Depois abra `http://localhost:8080/public`.

## Roteiro de testes

1. **Rota pública:** abra `/public`. Deve carregar sem pedir login.
2. **Login com Google:** abra `/cookie`. Você é redirecionado para o Google e, depois do login, volta para a página com os dados do usuário e o ID Token.
3. **Bearer token:** copie o JWT exibido em `/cookie` e chame a API sem cookie de sessão:
   ```bash
   curl -i http://localhost:8080/jwt -H "Authorization: Bearer <cole-o-token-aqui>"
   ```
4. **Sem token:** repita a chamada sem o header `Authorization` e observe o status HTTP que volta e o motivo.

Dica: cole o token em [jwt.io](https://jwt.io) para ver as claims (`iss`, `aud`, `exp`, `email`...).

## O que estudar a partir daqui

- Diferença entre ID Token (OIDC) e Access Token.
- Por que o resource server só precisa do `issuer-uri` para validar a assinatura (JWKS).
- Que principal o Spring cria em cada caso: `OidcUser` no login e `Jwt` no Bearer token.
- Como converter claims do JWT em authorities (`JwtAuthenticationConverter`).
