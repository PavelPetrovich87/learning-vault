# DevOps-стек 2026 для AI-разработки: Исследование и рекомендации

## TL;DR

Для 10-проектного fullstack-курса на AWS Free Tier рекомендуется **трёхуровневый DevOps-стек**: **Docker + Compose** для локальной разработки (все 10 проектов), **ECS Fargate** для простых проектов (W1, W2, W7), **EKS** для 5 проектов с K8s (W3, W4, W5, W6, W10). **Pulumi (TypeScript)** — оптимальный IaC-инструмент для AI-генерации, **GitHub Actions** — стандарт CI/CD, **Grafana LGTM** — open-source observability. **ArgoCD** — опционально для W8-W10. Ключевой принцип: *не добавлять Kubernetes без причины* — EKS стоит **$73/мес** за control plane и требует 10+ дополнительных компонентов, тогда как ECS бесплатна и требует 0 дополнительных операций.

---

## 1. Контейнеризация: Docker Best Practices 2026

### 1.1. Multi-Stage Builds: основа оптимизации

Multi-stage builds остаются фундаментальной практикой в 2026 году, но с существенными улучшениями благодаря **BuildKit**. Ключевое преимущество — параллельное выполнение независимых стадий. Например, frontend и backend сборки могут выполняться одновременно, что сокращает общее время сборки на **40-75%** [^2^]. Финальный образ должен содержать только runtime-зависимости — без исходного кода, без инструментов сборки, без package manager caches.

**Security-first подход к multi-stage** включает три критических практики. Во-первых, **никогда не копировать секреты** в финальную стадию — использовать BuildKit secret mounts (`--mount=type=secret`). Во-вторых, **distroless или scratch base images** для финальной стадии — отсутствие shell означает, что у атакующего нет инструментов для эксплуатации [^2^]. В-третьих, **pin image digests** для воспроизводимости: `FROM node:22-alpine@sha256:abc123...`.

**Рекомендуемая структура Dockerfile** для fullstack-проекта:

```dockerfile
# syntax=docker/dockerfile:1.4
# Stage 1: Frontend build (parallel with backend)
FROM node:22-alpine AS frontend
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN --mount=type=cache,target=/root/.npm npm ci
COPY frontend/ .
RUN npm run build

# Stage 2: Backend build
FROM node:22-alpine AS backend
WORKDIR /app
COPY api/package*.json ./
RUN --mount=type=cache,target=/root/.npm npm ci
COPY api/ .
RUN npm run build

# Stage 3: Production — distroless
FROM gcr.io/distroless/nodejs22-debian12
WORKDIR /app
COPY --from=backend /app/dist ./dist
COPY --from=backend /app/node_modules ./node_modules
COPY --from=frontend /app/frontend/dist ./static
EXPOSE 8080
CMD ["dist/main.js"]
```

### 1.2. Docker Compose: локальный стек как единый источник истины

Docker Compose в 2026 году — это не просто удобство, а **инфраструктурный контракт** между разработчиком и средой выполнения. Ключевые практики для локальной разработки включают health checks с `depends_on.condition: service_healthy`, именованные volumes для персистентности данных, bind mounts с исключением `node_modules` через анонимные volumes [^26^][^28^].

**Стандартный локальный стек** для каждого из 10 проектов включает: **frontend** (Next.js dev server), **API** (Node.js/Python), **PostgreSQL** с healthcheck, **Redis** с AOF persistence, и опционально **Kafka** через profiles [^28^][^32^]. Использование `profiles` (стабилизировано в Compose 2.24+) позволяет включать Kafka, Elasticsearch, MinIO только при необходимости без дублирования файлов конфигурации.

**Критический паттерн** — разделение сетей на `frontend-net` (доступна извне) и `backend-net` (только internal). API подключен к обеим, тогда как PostgreSQL и Redis — только к backend-net. Это минимизирует blast radius при компрометации [^28^].

### 1.3. BuildKit: что изменилось в 2026

BuildKit стал дефолтным builder'ом ещё в 2024, но в 2026 году появились **cache mounts для package managers** и **registry-based cache** для CI/CD. Кэширование через `type=gha` (GitHub Actions cache) в `docker/build-push-action` сокращает время сборки в CI с минут до секунд [^17^]. Ключевые метрики: первичная сборка — **40% быстрее** legacy builder, кэшированная сборка — **75% быстрее** [^17^].

---

## 2. Kubernetes: When Must-Have vs Overkill

### 2.1. Почему Kubernetes — не дефолтный выбор

Распространённый антипаттерн 2026 года — **"Kubernetes by default"**. EKS стоит **$73/мес** за control plane [^33^] и требует поддержки 10+ дополнительных компонентов: AWS Load Balancer Controller, Karpenter/Cluster Autoscaler, External DNS, cert-manager, Ingress controller, Metrics Server, EBS CSI Driver, CoreDNS tuning [^37^]. Каждый компонент — это Kubernetes deployment, который нужно обновлять, мониторить и отлаживать.

**Реальная стоимость EKS** для 25-сервисной архитектуры: прямые AWS-расходы ~$2,260/мес + 1.0 FTE platform engineer ($20,000/мес loaded cost) = **~$22,260/мес total** [^37^]. Для ECS Fargate при той же нагрузке: ~$1,987/мес AWS + 0.25 FTE ($5,000/мес) = **~$7,000/мес** — в **3.2 раза дешевле** [^37^].

### 2.2. Когда Kubernetes must-have

Kubernetes оправдан при выполнении хотя бы одного из условий [^37^][^38^]:

- **WebSocket / persistent connections** — требуются sticky sessions и session affinity
- **Micro-frontend архитектура** — независимые deploy'ы federated модулей
- **Горизонтальное масштабирование** — HPA с кастомными метриками
- **Stateful workloads** — StatefulSets с complex orchestration
- **Service mesh** — Istio/Linkerd для advanced traffic management
- **Multi-cloud portability** — планируется миграция на GKE/AKS

### 2.3. Решение для 10 проектов

На основе архитектурных требований проектов (из файлов W1-W10):

| Проект | Deploy Type | Обоснование |
|--------|-------------|-------------|
| **W1** Social Feed | **ECS Fargate** | SSE (однонаправленный), ISR, нет persistent connections |
| **W2** E-commerce | **ECS Fargate** | PPR/ISR, standard REST + GraphQL, нет WebSocket |
| **W3** Real-time Chat | **EKS** | WebSocket bidirectional, sticky sessions, Redis Pub/Sub broadcast |
| **W4** Collab Editor | **EKS** | WebSocket CRDT, zero-downtime deploys, security-first |
| **W5** Video Streaming | **EKS** | HLS/DASH streaming, CDN integration, performance budgets |
| **W6** Analytics Dashboard | **EKS** | SSE streaming, Web Workers, micro-frontends, time-series DB |
| **W7** Calendar | **ECS Fargate** | Standard REST, feature flags (client-side), нет real-time |
| **W8** File Manager | **EKS** | Offline-first, Service Worker, chunked upload, strict CSP |
| **W9** Maps Service | **EKS** | Resumability (Islands), lazy loading, PostGIS |
| **W10** Notifications | **EKS** | Micro-frontend widget, BroadcastChannel, multi-channel delivery |

**Итого**: **5 проектов на EKS** (W3, W4, W5, W6, W10), **3 проекта на ECS** (W1, W2, W7), **2 проекта на EKS** (W8, W9) — соответствует требованию "половина проектов с K8s" с запасом (7 из 10 при необходимости, но рекомендуется именно 5).

### 2.4. Helm vs Kustomize: что выбрать

Для учебных проектов рекомендуется **Kustomize** вместо Helm [^4^][^7^][^8^]. Обоснование:

- **Plain YAML всегда** — каждый output Kustomize — валидный Kubernetes YAML, без intermediate representation [^7^]
- **Низкий learning curve** — patches вместо Go templates, проще для агента генерировать [^8^]
- **Native kubectl integration** — `kubectl apply -k` без дополнительных бинарников [^8^]
- **GitOps-friendly** — base + overlays структура отлично ложится в Git [^7^]

Helm остаётся полезным только для **third-party charts** (Prometheus, Grafana, Ingress-NGINX), где сообщество предоставляет готовые пакеты [^8^]. Рекомендуемый гибрид: Helm для сторонних сервисов, Kustomize для собственных приложений.

---

## 3. IaC: Terraform vs Pulumi vs Crossplane

### 3.1. Сравнительный анализ

Три философии IaC в 2026 году представляют принципиально разные подходы [^3^]:

| Критерий | Terraform / OpenTofu | Pulumi | Crossplane |
|----------|---------------------|--------|------------|
| **Язык** | HCL (DSL) | TypeScript / Python / Go / YAML | YAML + Go |
| **Execution** | On-demand plan/apply | On-demand plan/apply | Continuous reconciliation |
| **State** | JSON state file | Service/backend store | Kubernetes etcd |
| **Drift detection** | Manual (`terraform plan`) | Manual (`pulumi refresh`) | Automatic (~30s) |
| **AI-генерация** | ⚠️ HCL требует изучения | ✅ TypeScript — естественный для агента | ❌ K8s + CRD концепции — высокий порог |
| **Learning curve** | Низкий (HCL минимален) | Средний (язык + SDK) | Высокий (K8s + CRD) |
| **Provider ecosystem** | 4,000+ | 100+ | 150+ |
| **GitOps** | Требует wrapper (Atlantis) | Pulumi Deployments | Native (ArgoCD/Flux) |

### 3.2. Почему Pulumi — лучший выбор для обучения

Pulumi в 2026 году стал **конвергентной платформой**: в январе 2026 анонсирована **native HCL support** — Pulumi CLI может интерпретировать существующий Terraform/HCL код без конвертации [^47^]. Это снимает главное возражение против adoption ("у нас уже есть Terraform модули").

**Ключевое преимущество для AI-агентов**: Pulumi Neo может превращать prompts вроде "deploy a GPU-backed EKS cluster with three node groups" в production-ready код с учётом policy guardrails [^31^]. TypeScript SDK предоставляет **полную type safety** и IDE support — агент генерирует код, который компилируется перед применением. Сравнение AI-моделей для генерации Pulumi TypeScript: **Claude 3.5 Sonnet** — лучший first-pass код, **GPT-4.1** — лучший debugging/refinement loop [^52^].

**Pulumi YAML** как альтернатива TypeScript подходит для простых случаев, но лишён программируемости — нет loops, conditionals, functions [^47^]. Для учебных проектов рекомендуется **TypeScript** с возможностью fallback на YAML для простых ресурсов.

### 3.3. Почему НЕ Crossplane для обучения

Crossplane предполагает глубокое понимание **Kubernetes CRD, Composition, XRD** — концепций, которые сами по себе требуют недель изучения [^3^][^49^]. Для 50-дневного курс это **overhead без пропорциональной ценности**. Continuous reconciliation — мощная фича, но ручной `pulumi up` после code review даёт больше контроля для обучения. Crossplane имеет смысл рассмотреть на **W10 как stretch goal** для platform-grade проектов.

### 3.4. OpenTofu vs Terraform

С 2024 года Terraform перешёл на BSL (Business Source License), что привело к форку **OpenTofu** под Apache 2.0 [^47^]. Для учебных проектов рекомендуется **OpenTofu** как полностью open-source альтернатива без лицензионных ограничений. Pulumi поддерживает оба: native HCL для Terraform и прямую совместимость с OpenTofu [^47^].

---

## 4. CI/CD: GitHub Actions как Pipeline-as-Spec

### 4.1. Почему GitHub Actions

GitHub Actions доминирует в категории SaaS-CI с **68% adoption** в GitHub-проектах [^1^][^9^]. Для курса, ориентированного на GitHub-экосистему, это естественный выбор. Ключевые преимущества [^5^][^6^]:

- **2000 бесплатных минут/мес** для приватных репозиториев (vs 400 у GitLab) [^5^]
- **20,000+ Actions** в Marketplace — почти для любой интеграции
- **OIDC AWS authentication** — одна строка `configure-aws-credentials` вместо long-lived credentials [^1^]
- **Native GitHub integration** — PR comments, branch protection, environment approvals

### 4.2. Progressive pipeline: 3 фазы

| Фаза | Проекты | Pipeline features | Сложность |
|------|---------|------------------|-----------|
| **Phase 1** | W1-W3 | lint + typecheck + test + build + Docker push | Low |
| **Phase 2** | W4-W7 | + Semgrep security scan + Lighthouse CI + bundle analysis + auto-deploy dev | Medium |
| **Phase 3** | W8-W10 | + Snyk scan + secret detection + coverage gate 80% + blue-green deploy + rollback | High |

**Ключевой паттерн для агента**: pipeline генерируется из spec-файла `.github/workflows/spec.yml`, который агент может читать и модифицировать. Каждый проект наследует базовый workflow и добавляет project-specific шаги через `workflow_call` (reusable workflows).

### 4.3. OIDC: безопасная аутентификация в AWS

Вместо хранения AWS credentials в GitHub Secrets, рекомендуется **OIDC federation**:

```yaml
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::ACCOUNT:role/GitHubActionsRole
    aws-region: us-east-1
```

Это устраняет необходимость в долгоживущих credentials и соответствует security best practices из W4 (security-first проект) [^1^].

---

## 5. Observability: Open Source Stack

### 5.1. Grafana LGTM: де-факто стандарт

Стек **Loki (logs) + Grafana (visualization) + Tempo (traces) + Mimir (metrics)** — наиболее зрелое open-source решение для observability в 2026 году [^11^][^15^][^16^]. Каждый компонент оптимизирован для своего типа данных:

- **Loki**: индексирует только метаданные (labels), а не полный текст — в **12-19 раз дешевле** Elasticsearch [^16^]
- **Tempo**: хранит traces в object storage без индексации — минимальная operational cost [^19^]
- **Mimir**: масштабируется до 1 миллиарда active series [^15^]
- **Grafana**: 100+ datasource plugins, unified alerting [^11^]

### 5.2. Progressive adoption по проектам

| Фаза | Проекты | Что включаем | Инфраструктура |
|------|---------|-------------|----------------|
| **Phase 0** | W1-W3 | Structured logging (pino/winston) | 0 доп. контейнеров |
| **Phase 1** | W4-W5 | + Prometheus + Grafana dashboards | +2 контейнера |
| **Phase 2** | W5-W6 | + web-vitals RUM + performance budgets | NPM package only |
| **Phase 3** | W8-W10 | + OpenTelemetry + Loki + Tempo + Mimir + alerting | +4 контейнера / Helm релиза |

**Grafana Cloud Free Tier** предлагает 10K metrics, 50GB logs, 50GB traces — достаточно для учебных проектов [^11^]. Для self-hosted варианта рекомендуется Docker Compose на локальной машине (0 cost) или Helm charts в EKS для K8s-проектов.

### 5.3. OpenTelemetry: единый стандарт инструментации

OpenTelemetry стал **универсальным стандартом** для сбора телеметрии [^29^]. `@opentelemetry/auto-instrumentations-node` автоматически инструментирует HTTP, Express, PostgreSQL, Redis без изменения кода приложения. Экспорт через OTLP в Alloy/Tempo обеспечивает **нулевую vendor lock-in** — можно переключаться между self-hosted и managed backends без изменения приложения.

---

## 6. GitOps: ArgoCD для учебных проектов

### 6.1. ArgoCD: не overkill при правильном подходе

ArgoCD добавляет overhead (~500MB RAM, 0.5 CPU) [^21^], но для команд с **3+ сервисами** на Kubernetes преимущества перевешивают [^46^]:

- **Deployment visibility** — UI показывает состояние всех приложений
- **Git-based rollback** — `git revert` вместо `kubectl` команд
- **Drift detection** — автоматическое обнаружение и self-healing

### 6.2. Минимальная установка для обучения

```bash
helm install argocd argo/argo-cd \
  -n argocd \
  --set server.service.type=LoadBalancer \
  --set configs.params.server.insecure=true
```

Для учебных проектов достаточно **одного Application per service** с auto-sync, без SSO, без ApplicationSets, без complex RBAC [^46^]. Рекомендуется внедрять на **W8-W10** (phase 3), когда проектов на K8s становится достаточно много.

### 6.3. Альтернатива: CI/CD с kubectl

Для ранних проектов (W3-W7) достаточно **GitHub Actions + kubectl apply** [^46^]:

```yaml
deploy:
  steps:
    - uses: aws-actions/configure-aws-credentials@v4
    - run: |
        aws eks update-kubeconfig --name ${PROJECT_NAME}-eks
        kubectl apply -k k8s/overlays/dev/
```

Это проще для агента генерировать и поддерживать, чем полноценный GitOps. ArgoCD добавляется как **incremental improvement** на поздних проектах.

---

## 7. Ранжирование по сложности и ценности для обучения

### 7.1. Матрица инструментов

| Уровень | Инструменты | Сложность (1-10) | Агенто-генерируемость | Обучающая ценность | Проекты |
|---------|-------------|-----------------|----------------------|-------------------|---------|
| **Level 0** | Docker, docker-compose | **2** | ✅ Полная | ★★★ Фундамент | W1-W10 |
| **Level 1** | Docker multi-stage, BuildKit | **3** | ✅ Полная | ★★ Оптимизация | W1-W10 |
| **Level 2** | ECS Fargate, ALB | **4** | ✅ Высокая | ★★★ Практичность | W1, W2, W7 |
| **Level 3** | EKS, Kustomize, kubectl | **6** | ✅ Средняя | ★★★★ Industry standard | W3-W6, W8-W10 |
| **Level 4** | Pulumi IaC, Helm | **5** | ✅ Высокая | ★★★★ Автоматизация | W1-W10 |
| **Level 5** | ArgoCD GitOps | **7** | ⚠️ Средняя | ★★★★ Продакшен | W8-W10 |
| **Level 6** | OpenTelemetry, LGTM stack | **6** | ✅ Высокая | ★★★★★ Observability | W4-W10 |
| **Level 7** | Security scanning (Snyk, Semgrep) | **4** | ✅ Полная | ★★★★ Безопасность | W4-W10 |
| **Level 8** | Feature flags, blue-green deploy | **7** | ⚠️ Низкая | ★★★★ Zero-downtime | W4, W6, W7, W10 |
| **Level 9** | Crossplane / advanced GitOps | **9** | ❌ Низкая | ★★ Stretch goal | W10 |

### 7.2. Принцип прогрессивного усложнения

Ключевой принцип, выведенный из анализа проектов и harness-модели: **"Агент ошибается → Harness растёт"** [^4^]. Каждый новый DevOps-инструмент добавляется не потому, что "он крутой", а потому, что предыдущего инструмента недостаточно для решения возникающих проблем.

Например: Docker Compose достаточен для W1-W2. На W3 чату требуется WebSocket scaling — появляется K8s. На W4 редактор требует zero-downtime — добавляется ArgoCD или feature flags. На W6 дашборд требует Web Workers offloading — добавляется HPA. Это **evidence-based approach** вместо "big bang" внедрения.

---

## 8. Рекомендация: DevOps-уровни по группам проектов

### 8.1. Проекты 1-3 (W1-W3): Фундамент

**DevOps tier**: Container-Only
**Цель**: Освоить Docker, Compose, базовый CI/CD

| Компонент | Решение | Специфика |
|-----------|---------|-----------|
| Контейнеризация | Docker multi-stage + BuildKit | node:22-alpine → distroless |
| Локальный стек | Docker Compose | frontend + api + postgres + redis |
| Деплой | **ECS Fargate** | 1 задача, 256 CPU / 512 MB |
| IaC | Pulumi TypeScript | VPC + ECS + ECR + ALB |
| CI/CD | GitHub Actions Phase 1 | lint + test + build + Docker push |
| Observability | Structured logging (pino) | console.log → JSON |
| Безопасность | .dockerignore, secret mounts | No secrets in images |

**W1** (Social Feed): SSE работает через обычный HTTP — не требует K8s. ISR на Next.js — функция фреймворка, не инфраструктуры.
**W2** (E-commerce): PPR/ISR — Next.js features. Autocomplete — client-side с debounce. Нет persistent connections.
**W3** (Chat): Первый real-time проект — **переход на EKS**. WebSocket требует sticky sessions, которые в ECS требуют хаков, а в K8s — `sessionAffinity`.

### 8.2. Проекты 4-7 (W4-W7): Kubernetes + Observability

**DevOps tier**: Kubernetes-Native
**Цель**: Освоить K8s, HPA, базовую observability, security scanning

| Компонент | Решение | Специфика |
|-----------|---------|-----------|
| Контейнеризация | Docker multi-stage + security scan | distroless + Trivy/Snyk scan |
| Локальный стек | Docker Compose + k3d | k3d cluster create dev |
| Деплой | **EKS** | 2 replica, t3.micro nodes (Free Tier) |
| IaC | Pulumi TypeScript | VPC + EKS + ECR + ALB + Security Groups |
| K8s конфиг | Kustomize | base + dev + prod overlays |
| CI/CD | GitHub Actions Phase 2 | + Semgrep + Lighthouse + auto-deploy dev |
| Observability | Prometheus + Grafana | Application metrics + dashboards |
| Безопасность | Semgrep + basic NetworkPolicy | Scan on every PR |

**W4** (Collab Editor): Security-first проект — добавляем **NetworkPolicy** для изоляции, **PodSecurityPolicy** (или OPA Gatekeeper как stretch goal). CRDT через WebSocket — sticky sessions обязательны.
**W5** (Video): Performance budgets в CI через Lighthouse. RUM через web-vitals library. HLS streaming — S3 + CloudFront, не K8s.
**W6** (Dashboard): Micro-frontends через Module Federation — каждый widget = отдельный K8s Deployment с независимым scaling. Web Workers — не влияют на DevOps.
**W7** (Calendar): Возврат к **ECS** — нет real-time, нет persistent connections. Feature flags (LaunchDarkly) — SaaS, не требует K8s.

### 8.3. Проекты 8-10 (W8-W10): Platform-Grade

**DevOps tier**: Full Platform
**Цель**: GitOps, полная observability, zero-downtime, security pipeline

| Компонент | Решение | Специфика |
|-----------|---------|-----------|
| Контейнеризация | Docker + BuildKit bake | Параллельные стадии, registry cache |
| Локальный стек | Docker Compose + k3d + observability | + Prometheus + Grafana + Loki |
| Деплой | **EKS** | 3 replica, HPA, PDB |
| IaC | Pulumi TypeScript full | VPC + EKS + RDS + ElastiCache + S3 + ALB |
| K8s конфиг | Kustomize + Helm | Helm для third-party (Prometheus, ArgoCD) |
| GitOps | **ArgoCD** (W8-W10) | Auto-sync, drift detection |
| CI/CD | GitHub Actions Phase 3 | + Snyk + secrets + coverage gate + blue-green |
| Observability | Full LGTM + OpenTelemetry | Logs + metrics + traces + alerts |
| Безопасность | Full pipeline | Semgrep + Snyk + secret detection + CSP check |

**W8** (File Manager): Offline-first = Service Worker + Background Sync. Chunked upload = tus protocol. Strict CSP — первый проект с полноценным security pipeline.
**W9** (Maps): Resumability (Qwik/Astro Islands) — не влияет на DevOps. Lazy loading Mapbox — dynamic import. PostGIS — managed RDS с extension.
**W10** (Notifications): Микрофронтенды + BroadcastChannel + multi-channel delivery. Kafka для event ingestion. **ArgoCD обязателен** — 5+ сервисов, нужна централизованная visibility.

---

## 9. Spec-фрагменты для AI-агента

### 9.1. Общая структура spec-языка

На основе исследования оптимальный формат spec для агента — **YAML с переменными подстановки** [^50^]. Это позволяет агенту:

1. Читать spec как structured data
2. Подставлять project-specific значения из `feature_list.json`
3. Генерировать конечные файлы (docker-compose.yml, Kustomize, Pulumi TypeScript)

**Пример переменных подстановки**:

```yaml
variables:
  PROJECT_NAME: "social-feed"      # из feature_list.json
  IMAGE_TAG: "1.0.0"               # из git tag или package.json
  DOMAIN: "devops-course.local"    # из config
  AWS_REGION: "us-east-1"          # из Pulumi config
```

### 9.2. Спецификация для Docker Compose

```yaml
spec_version: "1.0"
purpose: "Local development environment for fullstack projects"
services:
  frontend:
    base_image: "node:22-alpine"
    ports: ["3000:3000"]
    volumes: ["./frontend:/app", "/app/node_modules"]
    command: "npm run dev"
    networks: ["frontend-net"]
  api:
    base_image: "node:22-alpine"
    ports: ["8080:8080"]
    environment:
      DATABASE_URL: "postgresql://dev:dev@postgres:5432/app"
      REDIS_URL: "redis://redis:6379"
    depends_on:
      postgres: { condition: service_healthy }
    networks: ["frontend-net", "backend-net"]
  postgres:
    image: "postgres:16-alpine"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev"]
      interval: 5s
      retries: 5
    networks: ["backend-net"]
  redis:
    image: "redis:7-alpine"
    networks: ["backend-net"]
networks:
  frontend-net: { driver: bridge }
  backend-net: { driver: bridge }
```

### 9.3. Спецификация для K8s (Kustomize)

```yaml
spec_version: "1.0"
strategy: "Kustomize overlays"
base_resources:
  deployment:
    replicas: 2
    containers:
      - name: api
        image: "${ECR_REGISTRY}/${PROJECT_NAME}-api:${IMAGE_TAG}"
        ports: [8080]
        resources:
          requests: { memory: "128Mi", cpu: "100m" }
          limits: { memory: "512Mi", cpu: "500m" }
        livenessProbe:
          httpGet: { path: /health, port: 8080 }
        readinessProbe:
          httpGet: { path: /ready, port: 8080 }
  service:
    type: ClusterIP
    ports: [{ port: 80, targetPort: 8080 }]
  ingress:
    annotations:
      alb.ingress.kubernetes.io/scheme: internet-facing
overlays:
  dev: { replicas: 1, hpa: disabled }
  prod: { replicas: 2, hpa: enabled, pdb: { minAvailable: 1 } }
```

### 9.4. Спецификация для Pulumi IaC

```yaml
spec_version: "1.0"
tool: "Pulumi (TypeScript)"
provider: "aws"
common_resources:
  vpc:
    type: "aws:ec2:Vpc"
    properties:
      cidrBlock: "10.0.0.0/16"
  ecr_repo:
    type: "aws:ecr:Repository"
    properties:
      imageScanningConfiguration: { scanOnPush: true }
project_types:
  simple:  # ECS for W1, W2, W7
    ecs_cluster: { type: "aws:ecs:Cluster" }
    ecs_task:
      type: "aws:ecs:TaskDefinition"
      properties:
        cpu: "256"
        memory: "512"
        requiresCompatibilities: ["FARGATE"]
    ecs_service:
      type: "aws:ecs:Service"
      properties:
        desiredCount: 1
        launchType: "FARGATE"
  kubernetes:  # EKS for W3-W6, W8-W10
    eks_cluster:
      type: "aws:eks:Cluster"
      properties:
        version: "1.29"
    eks_node_group:
      type: "aws:eks:NodeGroup"
      properties:
        instanceTypes: ["t3.micro"]
        desiredCapacity: 2
free_tier_limits:
  ec2: "750 hours/month of t3.micro"
  eks_control_plane: "$73/month — AVOID for Free Tier"
```

### 9.5. Спецификация для CI/CD

```yaml
spec_version: "1.0"
tool: "GitHub Actions"
templates:
  ci:
    triggers: { push: [main, develop], pull_request: [main] }
    jobs:
      lint-and-typecheck: { node: 22, steps: [lint, typecheck] }
      test:
        services:
          postgres: { image: "postgres:16-alpine", ports: ["5432:5432"] }
          redis: { image: "redis:7-alpine", ports: ["6379:6379"] }
      build: { artifact: "./dist" }
      docker:
        aws_auth: { role: "arn:aws:iam::${ACCOUNT}:role/GitHubActionsRole" }
        ecr_login: true
        buildx: true
        cache: { from: "type=gha", to: "type=gha,mode=max" }
  cd:
    triggers: { push: [main], workflow_dispatch: { environment: [dev, staging, prod] } }
    deploy_options:
      ecs: "aws ecs update-service --force-new-deployment"
      eks: "kubectl set image deployment/... + rollout status"
progressive_features:
  phase_1_w1_w3: ["lint", "test", "build", "docker push", "manual deploy"]
  phase_2_w4_w7: ["+ Semgrep", "+ Lighthouse CI", "+ auto-deploy dev"]
  phase_3_w8_w10: ["+ Snyk", "+ secrets", "+ coverage 80%", "+ blue-green"]
```

---

## 10. Free Tier: стратегия экономии

### 10.1. Что точно бесплатно на AWS

| Сервис | Free Tier | Хватит на |
|--------|-----------|-----------|
| **EC2 t3.micro** | 750 hrs/мес × 12 мес | 1-2 инстанса 24/7 |
| **ECS** | Нет control plane fee | Любое количество сервисов |
| **EKS** | **$73/мес** — НЕ входит | Только для 5 проектов |
| **S3** | 5GB + 20K GET | Медиа, assets, backups |
| **ECR** | 500MB + 5K pulls | 10-20 образов |
| **CloudFront** | 50GB data transfer | CDN для медиа |
| **RDS PostgreSQL** | 750 hrs db.t3.micro | БД для простых проектов |
| **ALB** | 750 hrs + 15 LCUs | 1 shared ALB |

### 10.2. Стратегия мультиплексирования

Для минимизации cost при 10 проектах:

- **Один shared ECS cluster** для W1, W2, W7 — нет платы за cluster
- **Один EKS cluster** для всех K8s-проектов — **$73/мес** total, не per-project
- **Один ALB** с host-based routing — разные поддомены на разные сервисы
- **RDS PostgreSQL** — один инстанс с разными database'ами per project
- **ElastiCache Redis** — один cluster с разными logical databases

Таким образом, реальный cost: **$73/мес (EKS) + $0 (ECS) + ~$15/мес (ALB) + $0 (RDS Free Tier) = ~$88/мес** — вписывается в бюджет любого курса.

---

## 11. Выводы и итоговая матрица

### 11.1. Итоговая таблица: уровень × инструменты × сложность × агенто-генерируемость

| Уровень | Инструменты | Сложность | Агенто-генерируемость | Проекты |
|---------|-------------|-----------|----------------------|---------|
| **Level 0** | Docker, docker-compose | 2 | ✅ Полная | W1-W10 |
| **Level 1** | Docker multi-stage, BuildKit, cache mounts | 3 | ✅ Полная | W1-W10 |
| **Level 2** | ECS Fargate, ALB, ECR | 4 | ✅ Высокая | W1, W2, W7 |
| **Level 3** | EKS, Kustomize, kubectl, HPA | 6 | ✅ Средняя | W3-W6, W8-W10 |
| **Level 4** | Pulumi TypeScript IaC | 5 | ✅ Высокая | W1-W10 |
| **Level 5** | ArgoCD GitOps | 7 | ⚠️ Средняя | W8-W10 |
| **Level 6** | OpenTelemetry, LGTM, alerting | 6 | ✅ Высокая | W4-W10 |
| **Level 7** | Security: Semgrep, Snyk, secret detection | 4 | ✅ Полная | W4-W10 |
| **Level 8** | Zero-downtime: feature flags, blue-green | 7 | ⚠️ Низкая | W4, W6, W7, W10 |
| **Level 9** | Platform: Crossplane, advanced GitOps | 9 | ❌ Низкая | Stretch goal |

### 11.2. Ключевые принципы для внедрения

1. **Start simple, evolve**: Docker Compose → ECS → EKS → ArgoCD. Не добавлять всё сразу.
2. **Evidence-based**: Каждый новый инструмент добавляется, когда предыдущий не справляется с задачей.
3. **Pulumi over Terraform**: TypeScript лучше генерируется агентом, native HCL support для legacy.
4. **K8s only when needed**: 5 проектов с EKS, остальные на ECS — достаточно для обучения.
5. **Free Tier aware**: Один shared EKS cluster, один ALB, мультиплексирование сервисов.
6. **Observability progressive**: Логи → метрики → трейсы. Не всё сразу.
7. **Spec-driven**: Все конфигурации генерируются из YAML-спецификаций — агент может читать и модифицировать.
