# infra-k8s
Infraestrutura K3s/Kubernetes e configurações de deploy para projetos.

## Ingestão de telemetria (OTLP)

Endpoint externo: `otel.justgui.dev` (DNS A → IP do LoadBalancer/MetalLB).

| Protocolo | Endpoint | Porta |
|---|---|---|
| OTLP/HTTP | `https://otel.justgui.dev` | 443 |
| OTLP/gRPC | `https://otel.justgui.dev` | 4317 |

**Todo envio exige token** (validado pelo `bearertokenauth` do otel-collector,
valor no SealedSecret `telemetry/otel-auth`, chave `token`):

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=https://otel.justgui.dev
OTEL_EXPORTER_OTLP_HEADERS="authorization=Bearer <token>"
```

Recebedores internos ao cluster também precisam do header. Pipeline:
OTLP → otel-collector → Tempo (traces) / Prometheus remote-write (metrics) / Loki (logs),
visualização no Grafana (`grafana.justgui.dev`, datasource Tempo usa `tempo:3200`).

## Domínios de acesso (1 por ferramenta, login unificado)

Login único em todo acesso administrativo: **szguisantos@gmail.com** / senha
padrão da infra. Onde a ferramenta não tem auth nativa (Prometheus,
Alertmanager), o Traefik cobra basicAuth (Middleware `telemetry/infra-basicauth`,
htpasswd bcrypt no SealedSecret `telemetry/infra-basicauth`).

| Domínio (DNS A → IP do LoadBalancer) | Ferramenta | Login |
|---|---|---|
| `grafana.justgui.dev` | Grafana | usuário/senha próprio do Grafana |
| `argocd.justgui.dev` | ArgoCD | usuário/senha próprio do ArgoCD |
| `minio-console.justgui.dev` | MinIO Console | usuário/senha próprio do MinIO (policy `consoleAdmin`) |
| `prometheus.justgui.dev` | Prometheus | basicAuth (Traefik) |
| `alertmanager.justgui.dev` | Alertmanager | basicAuth (Traefik) |
| `otel.justgui.dev` | Ingestão OTLP (não é UI) | Bearer token, ver seção abaixo |

Tempo e Loki não têm UI própria — são consultados via datasource do Grafana,
não precisam de domínio.
