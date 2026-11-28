# Deploy no Google Cloud (Cloud Run + Vertex AI)

Este guia publica a API e o front-end no Cloud Run, com o Gemini acessado pelo Vertex AI. Os workflows `deploy.yml` dos repositórios `back-end` e `front-end` fazem o build e o deploy depois de cada CI verde na `main`. Eles ficam inativos até as variáveis abaixo serem configuradas.

> Esta versão é um **piloto acadêmico**: dados em memória e contas fictícias (`HEALTHIA_MODO_PILOTO=true`). Não use dados reais de pacientes. Um uso real exige banco persistente, provedor de identidade, validação jurídica do termo e definição de controlador e encarregado (LGPD).

## 1. Preparar o projeto (uma vez)

Execute no Cloud Shell ou com o `gcloud` autenticado. Troque os valores de exemplo.

```bash
PROJETO=meu-projeto-healthia
REGIAO=southamerica-east1
ORG_REPO=FMU-Tech-Inovacao-2026-2
gcloud config set project "$PROJETO"

gcloud services enable run.googleapis.com artifactregistry.googleapis.com \
  aiplatform.googleapis.com secretmanager.googleapis.com iamcredentials.googleapis.com

gcloud artifacts repositories create healthia --repository-format=docker --location="$REGIAO"
```

## 2. Contas de serviço

```bash
# Quem roda a API: só pode chamar o Vertex AI e ler o segredo do JWT.
gcloud iam service-accounts create healthia-run --display-name="HealthIA Cloud Run"
gcloud projects add-iam-policy-binding "$PROJETO" \
  --member="serviceAccount:healthia-run@$PROJETO.iam.gserviceaccount.com" --role=roles/aiplatform.user

# Quem faz o deploy pelo GitHub Actions.
gcloud iam service-accounts create healthia-deploy --display-name="HealthIA deploy (GitHub)"
for PAPEL in roles/run.admin roles/artifactregistry.writer; do
  gcloud projects add-iam-policy-binding "$PROJETO" \
    --member="serviceAccount:healthia-deploy@$PROJETO.iam.gserviceaccount.com" --role="$PAPEL"
done
gcloud iam service-accounts add-iam-policy-binding "healthia-run@$PROJETO.iam.gserviceaccount.com" \
  --member="serviceAccount:healthia-deploy@$PROJETO.iam.gserviceaccount.com" --role=roles/iam.serviceAccountUser
```

## 3. Segredo do JWT

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))" | \
  gcloud secrets create healthia-jwt-secret --data-file=-
gcloud secrets add-iam-policy-binding healthia-jwt-secret \
  --member="serviceAccount:healthia-run@$PROJETO.iam.gserviceaccount.com" --role=roles/secretmanager.secretAccessor
```

## 4. Workload Identity Federation (sem chave JSON no GitHub)

```bash
gcloud iam workload-identity-pools create github --location=global --display-name="GitHub"
gcloud iam workload-identity-pools providers create-oidc github \
  --location=global --workload-identity-pool=github \
  --issuer-uri="https://token.actions.githubusercontent.com" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --attribute-condition="assertion.repository_owner == '$ORG_REPO'"

NUMERO=$(gcloud projects describe "$PROJETO" --format='value(projectNumber)')
for REPO in back-end front-end; do
  gcloud iam service-accounts add-iam-policy-binding "healthia-deploy@$PROJETO.iam.gserviceaccount.com" \
    --role=roles/iam.workloadIdentityUser \
    --member="principalSet://iam.googleapis.com/projects/$NUMERO/locations/global/workloadIdentityPools/github/attribute.repository/$ORG_REPO/$REPO"
done
echo "GCP_WIF_PROVIDER=projects/$NUMERO/locations/global/workloadIdentityPools/github/providers/github"
```

## 5. Variáveis no GitHub

Em cada repositório: **Settings > Secrets and variables > Actions > Variables**. Nenhum valor abaixo é secreto.

| Variável | back-end | front-end | Exemplo |
| --- | --- | --- | --- |
| `GCP_PROJECT_ID` | sim | sim | `meu-projeto-healthia` |
| `GCP_REGION` | opcional | opcional | `southamerica-east1` (padrão) |
| `GCP_WIF_PROVIDER` | sim | sim | saída do passo 4 |
| `GCP_DEPLOY_SA` | sim | sim | `healthia-deploy@<projeto>.iam.gserviceaccount.com` |
| `GCP_RUNTIME_SA` | sim | sim | `healthia-run@<projeto>.iam.gserviceaccount.com` |
| `CORS_ORIGIN` | sim | — | URL do front-end no Cloud Run |
| `HEALTHIA_API_URL` | — | sim | URL da API no Cloud Run |
| `GEMINI_PROVEDOR` | opcional | — | `vertex` (padrão) ou `demonstracao` |
| `GEMINI_MODEL` | opcional | — | `gemini-3.5-flash` (padrão) |
| `GEMINI_MODELOS_RESERVA` | opcional | — | `gemini-3.8-flash;gemini-3.1-flash-lite` (separados por `;`) |
| `VERTEX_LOCATION` | opcional | — | `us-central1` (padrão) |
| `HEALTHIA_MODO` | — | opcional | `backend` (padrão) ou `demonstracao` |

Os workflows usam o ambiente `producao`. Para exigir aprovação manual antes do deploy, adicione revisores em **Settings > Environments > producao**.

## 6. Primeira publicação

A API precisa da URL do front-end (CORS), e o front-end precisa da URL da API. Na primeira vez:

1. Configure as variáveis do back-end com `CORS_ORIGIN=https://exemplo.invalid` e rode **Actions > Deploy Cloud Run > Run workflow**. Anote a URL da API no resumo da execução.
2. Configure `HEALTHIA_API_URL` no front-end e rode o deploy. Anote a URL do front-end.
3. Atualize `CORS_ORIGIN` no back-end com a URL do front-end e rode o deploy de novo.

Depois disso, cada push na `main` com CI verde publica automaticamente.

## Verificação

```bash
curl https://<url-da-api>/health   # {"status":"ok","extracao":"vertex","piloto":true,...}
curl https://<url-do-front>/config.js
```

Entre no front-end com as contas fictícias do README do back-end e percorra o fluxo: nova alta, revisão, aceite e plano do paciente.

## Auditoria em produção

No Cloud Run, deixe `AUDITORIA_ARQUIVO` vazio: os eventos de auditoria já vão ao Cloud Logging pelo log estruturado (campo `auditoria`). O sistema de arquivos do contêiner é temporário, e um JSONL local se perderia a cada reinício. Para retenção, crie um coletor (sink) do Cloud Logging filtrando `jsonPayload.auditoria.evento`.

## Custos e limites

- O Cloud Run cobra por uso e escala a zero (`min-instances 0`). O primeiro acesso após inatividade é mais lento.
- A API roda com `max-instances 1` porque guarda dados em memória. Os dados somem quando a instância reinicia.
- Cada extração e cada transcrição de PDF chamam o Gemini e geram custo por token. A API limita 20 chamadas a cada 10 minutos por profissional.
- A disponibilidade de modelos muda com o tempo e pode variar entre a Gemini API e o Vertex AI. Confirme no console quais modelos o projeto acessa antes de definir `GEMINI_MODEL`.
- Confira os preços vigentes do Cloud Run e do Vertex AI no console antes de publicar.
