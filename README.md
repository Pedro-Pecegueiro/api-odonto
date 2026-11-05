# API de Agendamento Odontológico

API para organizar o cadastro e a consulta de dentistas. Neste projeto, trabalhei com autenticação por token, banco de dados MySQL e migrations para preparar a aplicação para execução local e publicação em nuvem.

## O que a API oferece

- Cadastro, consulta, atualização e inativação de dentistas.
- Login com geração de token JWT.
- Proteção das rotas da API, com exceção do login e da verificação de saúde.
- Criação e atualização das tabelas pelo Flyway.
- Verificação de saúde em `/actuator/health`.
- Empacotamento em Docker e validação automatizada pelo GitHub Actions.

## Tecnologias

Java 21 · Spring Boot · Spring Security · JWT · Spring Data JPA · MySQL · Flyway · Maven · Docker · GitHub Actions

## Como executar localmente

Você vai precisar do JDK 21 e de acesso a um banco MySQL. A pasta `api-odonto/` contém o projeto Maven.

Configure estas variáveis no terminal ou na configuração de execução da sua IDE:

| Variável | Para que serve |
| --- | --- |
| `DB_URL` | URL JDBC do banco MySQL |
| `DB_USERNAME` | Usuário do banco |
| `DB_PASSWORD` | Senha do banco |
| `JWT_SECRET` | Segredo aleatório, com pelo menos 32 bytes, usado nos tokens |
| `PORT` | Porta da aplicação; localmente, o padrão é `8080` |

O arquivo `api-odonto/.env.example` mostra o formato esperado, sem conter credenciais reais. O Spring Boot não carrega esse arquivo automaticamente. Não coloque senhas ou segredos nesse exemplo nem em arquivos enviados ao GitHub.

No PowerShell, configure seus valores e inicie a aplicação:

```powershell
$env:DB_URL = "jdbc:mysql://HOST:PORTA/BANCO?sslMode=REQUIRED"
$env:DB_USERNAME = "USUARIO"
$env:DB_PASSWORD = "SENHA"
$env:JWT_SECRET = "SEGREDO_ALEATORIO"

Set-Location api-odonto
.\mvnw.cmd spring-boot:run
```

Para validar o projeto e executar os testes:

```powershell
Set-Location api-odonto
.\mvnw.cmd clean verify
```

O teste atual inicializa o contexto Spring e o Flyway, por isso precisa alcançar o MySQL configurado. Recomendo usar um banco de laboratório, sem dados reais.

## Docker

O `Dockerfile` usa duas etapas: uma para compilar o projeto com Maven e outra, menor, para executar o JAR. Para montar a imagem, execute o comando dentro de `api-odonto/`:

```powershell
docker build -t api-odonto:local .
```

Para executar a imagem localmente, entre em `api-odonto/`, configure as variáveis no arquivo `.env` local (ele é ignorado pelo Git) e rode:

```powershell
docker run --rm --env-file .env -e PORT=8080 -p 8080:8080 api-odonto:local
```

## Banco de dados no Aiven

Preparei a conexão para que o MySQL seja hospedado no Aiven. Confira o plano e os custos antes de criar o serviço. Depois, consulte os dados de conexão no painel e monte `DB_URL` neste formato:

```text
jdbc:mysql://HOST:PORTA/BANCO?sslMode=REQUIRED
```

As migrations em `api-odonto/src/main/resources/db/migration` criam e atualizam as tabelas automaticamente quando a aplicação inicia. Não é necessário criar as tabelas manualmente.

## Integração contínua e publicação

O workflow `.github/workflows/ci.yml` prepara o Java 21, executa `clean verify` e constrói a imagem Docker quando há atualização na branch `main`. Também pode ser iniciado manualmente pela aba **Actions** do GitHub.

Antes de executar o workflow, cadastre estes Repository Secrets no GitHub:

- `DB_URL`
- `DB_USERNAME`
- `DB_PASSWORD`
- `JWT_SECRET`

O Render precisa receber as mesmas quatro variáveis separadamente no ambiente do serviço; os secrets do GitHub não são transferidos para lá. Confira o plano antes de criar o serviço. Para publicar este repositório, use:

- **Runtime:** Docker
- **Branch:** `main`
- **Root Directory:** `api-odonto`
- **Dockerfile Path:** `./Dockerfile`
- **Health Check Path:** `/actuator/health`
- **Auto-Deploy:** `After CI Checks Pass`, quando essa opção estiver disponível

O Render fornece a variável `PORT`; a aplicação escuta nessa porta e no endereço `0.0.0.0`.

**URL pública da API:** [https://api-odonto-eice.onrender.com](https://api-odonto-eice.onrender.com)

**Health check:** [https://api-odonto-eice.onrender.com/actuator/health](https://api-odonto-eice.onrender.com/actuator/health)

## Rotas principais

| Método | Rota | Acesso |
| --- | --- | --- |
| `POST` | `/login` | Público; retorna o token JWT |
| `GET` | `/actuator/health` | Público; informa o estado da aplicação sem expor detalhes internos |
| `GET` | `/dentista` | Requer token JWT |
| `GET` | `/dentista/{id}` | Requer token JWT |
| `POST` | `/dentista` | Requer token JWT |
| `PUT` | `/dentista` | Requer token JWT |
| `DELETE` | `/dentista/{id}` | Requer token JWT |

Nas rotas protegidas, envie o token no cabeçalho `Authorization`:

```text
Authorization: Bearer SEU_TOKEN
```
