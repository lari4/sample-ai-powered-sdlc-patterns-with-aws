# Agent Pipelines Documentation

Этот документ описывает все возможные пайплайны (workflows) работы AI-агента в приложении, какие промты вызываются, в каком порядке и какие данные передаются между ними.

## Содержание

1. [SDLC Phase-Specific Pipeline](#sdlc-phase-specific-pipeline)
2. [Epic Creation Pipeline](#epic-creation-pipeline)
3. [Domain Analysis and OpenAPI Generation Pipeline](#domain-analysis-and-openapi-generation-pipeline)
4. [Architecture Design Pipeline](#architecture-design-pipeline)
5. [DevOps Infrastructure Generation Pipeline](#devops-infrastructure-generation-pipeline)
6. [Performance Testing Pipeline](#performance-testing-pipeline)
7. [Incident Management Pipeline](#incident-management-pipeline)

---

## SDLC Phase-Specific Pipeline

**Назначение:** Основной пайплайн для работы в конкретной фазе SDLC.

**Компоненты:** `claude_service.py`, `enhanced_claude_service.py`

### ASCII Диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│                     User Request                             │
│                   (with phase context)                       │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Determine Current Phase                         │
│   (Requirements/Design/Development/Testing/etc.)             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│          Load Phase-Specific System Prompt                   │
│   • Requirements → Business Analyst prompt                   │
│   • Design → Software Architect prompt                       │
│   • Development → Technical Lead prompt                      │
│   • Testing → QA Engineer prompt                             │
│   • Deployment → DevOps Engineer prompt                      │
│   • Maintenance → System Administrator prompt                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│      Add Formatting Guidelines to System Prompt              │
│   • Human-readable JSON formatting                           │
│   • Structured information presentation                      │
│   • No raw JSON dumps                                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Add Available Tools Information                        │
│   • MCP server capabilities                                  │
│   • Tool descriptions                                        │
│   • Phase-specific tool recommendations                      │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Build Conversation Context (if available)                 │
│   • Previous conversation summary                            │
│   • Recent messages (last 5)                                 │
│   • Project KB context                                       │
│   • Relevant code examples                                   │
│   • Related architecture diagrams                            │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Send to Claude via AWS Bedrock                       │
│   Model: Claude 3 Sonnet                                     │
│   Max Tokens: 4000                                           │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Claude Generates Response                       │
│   • Contextual advice based on phase                         │
│   • Formatted, human-readable output                         │
│   • Tool usage recommendations                               │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                Return to User                                │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные

**Input:**
- `messages`: История сообщений пользователя
- `phase`: Текущая фаза SDLC (requirements, design, development, testing, deployment, maintenance)
- `conversation_context`: Контекст разговора с KB данными (опционально)

**Output:**
- Текстовый ответ с рекомендациями и советами, специфичными для фазы

---

## Epic Creation Pipeline

**Назначение:** Извлечение требований из Confluence и форматирование их в виде epics.

**Компоненты:** `enhanced_claude_service.py`, Amazon Q Business MCP

### ASCII Диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│       User: "Create epics from requirements"                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│     Enhanced Claude Service - Intent Analysis                │
│   Keywords: "create epics", "extract epics"                  │
│   → Identified: Epic Creation Request                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│      Build Intelligent System Prompt                         │
│   • Base phase prompt                                        │
│   • Strict tool selection rules (MAX 1 TOOL)                 │
│   • Explicit tool mapping                                    │
│   • Epic formatting requirements                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Add CRITICAL Epic Formatting Instructions                 │
│   Format: ## [Epic Name] Epic                                │
│   • Feature 1                                                │
│   • Feature 2                                                │
│   • Feature 3                                                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Claude decides to use Amazon Q Business              │
│   Tool: mcp_amazon_q_business_retrieve                       │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│   Execute Amazon Q Business with Exact Epic Prompt           │
│   "Extract requirements from AnyCompanyReads project         │
│    documentation and organize them into epics.               │
│    You MUST format the response EXACTLY like this:           │
│                                                               │
│    ## User Management Epic                                   │
│    • User registration and authentication system             │
│    • Role-based access control (Customer, Admin)             │
│    • Profile management with reading preferences             │
│                                                               │
│    CRITICAL: Use ## [Epic Name] Epic headers and             │
│    • bullet points ONLY."                                    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Amazon Q Business - Confluence Retrieval                  │
│   • Searches Confluence space                                │
│   • Extracts requirements from documents                     │
│   • Formats as epics with exact formatting                   │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│            Return Formatted Epics                            │
│   ## Epic 1 Epic                                             │
│   • Feature A                                                │
│   • Feature B                                                │
│                                                               │
│   ## Epic 2 Epic                                             │
│   • Feature C                                                │
│   • Feature D                                                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│        Enhanced Claude Service validates format              │
│   • Checks for "## [Name] Epic" headers                      │
│   • Checks for "•" bullet points                             │
│   • Returns to user if valid                                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Present Epics to User                           │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные

**Step 1 → Step 2:**
- User message: "create epics from requirements"

**Step 2 → Step 3:**
- Intent: epic_creation
- Suggested tool: mcp_amazon_q_business_retrieve

**Step 3 → Step 4:**
- Epic extraction prompt with EXACT formatting requirements

**Step 4 → Step 5:**
- Formatted epics with ## headers and • bullets

**Step 5 → User:**
- Validated epic list ready for use

---

## Domain Analysis and OpenAPI Generation Pipeline

**Назначение:** Анализ бизнес-домена и генерация OpenAPI спецификации.

**Компоненты:** `enhanced_claude_service.py`, Domain Analysis MCP, OpenAPI Generator MCP

### ASCII Диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│    User: "Analyze domain and generate OpenAPI spec"         │
└───────────────────────────┬─────────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
┌───────────────────────────┐   ┌──────────────────────────┐
│ Step 1: Domain Analysis   │   │ User: "Just generate     │
│                           │   │  OpenAPI spec"           │
│ User asks for "domain     │   │                          │
│ analysis"                 │   │ (Skip domain analysis)   │
└───────────┬───────────────┘   └─────────┬────────────────┘
            │                              │
            ▼                              │
┌─────────────────────────────────────────┼────────────────┐
│  Execute Domain Analysis Tool           │                │
│  Input: Requirements from Amazon Q      │                │
│  or Confluence data                     │                │
└───────────────────────────┬─────────────┘                │
                            │                              │
                            ▼                              │
┌─────────────────────────────────────────────────────────┐│
│       Domain Analysis Result (JSON)                     ││
│   • Business context                                    ││
│   • Domain entities and relationships                   ││
│   • Technical requirements                              ││
│   • API requirements                                    ││
└───────────────────────────┬─────────────────────────────┘│
                            │                              │
                            ├──────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│     Step 2: OpenAPI Spec Generation                          │
│  Tool: generate_openapi_spec                                 │
│  Input: Domain analysis result (or direct requirements)      │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Generate OpenAPI 3.1 Specification                        │
│   • info (title, version, description)                       │
│   • servers                                                  │
│   • paths (endpoints with operations)                        │
│   • components (schemas, securitySchemes)                    │
│   • security                                                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│      OpenAPI Spec Post-Processing                            │
│   • Extract JSON from response                               │
│   • Validate JSON structure                                  │
│   • Fix truncated JSON if needed                             │
│   • Create minimal spec if extraction fails                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Save OpenAPI Spec to S3                              │
│   Filename: openapi_spec_{timestamp}.json                    │
│   Location: s3://bucket/project-name/                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Return OpenAPI Spec to User                               │
│   • Formatted JSON                                           │
│   • S3 location                                              │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные

**Step 1 Input:**
- Requirements text or Confluence data
- Business context

**Step 1 → Step 2:**
```json
{
  "business_context": "E-commerce bookstore...",
  "entities": [
    {"name": "User", "attributes": [...], "relationships": [...]},
    {"name": "Book", "attributes": [...], "relationships": [...]}
  ],
  "technical_requirements": [...],
  "api_requirements": [...]
}
```

**Step 2 → Step 3:**
- Domain analysis JSON
- API style (REST/GraphQL/RPC)
- Authentication requirements

**Step 3 Output:**
```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "Bookstore API",
    "version": "1.0.0",
    "description": "API for online bookstore"
  },
  "servers": [...],
  "paths": {
    "/books": {
      "get": {...},
      "post": {...}
    }
  },
  "components": {
    "schemas": {...},
    "securitySchemes": {...}
  }
}
```

---

## Architecture Design Pipeline

**Назначение:** Создание архитектурных диаграмм и оценка стоимости инфраструктуры.

**Компоненты:** `enhanced_claude_service.py`, Architecture Diagram MCP, Cost Estimation MCP

### ASCII Диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│  User: "Create architecture diagram for my application"      │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Enhanced Claude Service - Intent Analysis              │
│   Keywords: "diagram", "architecture", "visualize"           │
│   → Identified: Architecture Visualization Request           │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Execute create_architecture_diagram Tool                  │
│   Input:                                                     │
│   • System requirements                                      │
│   • Component descriptions                                   │
│   • AWS services to use                                      │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Generate Architecture Diagram                          │
│   • Parse requirements                                       │
│   • Identify components and relationships                    │
│   • Create visual diagram (draw.io/mxgraph format)           │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│           Return Diagram to User                             │
│   • Diagram XML/image                                        │
│   • Component list                                           │
│   • Relationship descriptions                                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Optional: User asks "What's the cost?"                    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│      Execute estimate_architecture_cost Tool                 │
│   Input: Architecture diagram components                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Calculate AWS Resource Costs                         │
│   • Identify AWS services in architecture                    │
│   • Apply pricing for each service                           │
│   • Calculate monthly/yearly estimates                       │
│   • Provide cost breakdown by service                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│          Return Cost Estimate to User                        │
│   • Total monthly cost                                       │
│   • Cost breakdown by service                                │
│   • Cost optimization suggestions                            │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные

**Input:**
- System requirements text
- Desired AWS services
- Scale/load requirements

**Diagram Generation Output:**
```json
{
  "diagram_url": "s3://bucket/diagrams/arch-123.xml",
  "components": [
    {"name": "ALB", "type": "load_balancer"},
    {"name": "ECS Fargate", "type": "compute"},
    {"name": "RDS PostgreSQL", "type": "database"}
  ],
  "relationships": [
    {"from": "ALB", "to": "ECS Fargate", "type": "routes_to"},
    {"from": "ECS Fargate", "to": "RDS", "type": "reads_writes"}
  ]
}
```

**Cost Estimation Output:**
```json
{
  "monthly_cost": 847.50,
  "breakdown": {
    "ALB": 22.50,
    "ECS_Fargate": 350.00,
    "RDS_PostgreSQL": 475.00
  },
  "optimization_suggestions": [
    "Consider Reserved Instances for 30% savings",
    "Use Fargate Spot for non-critical workloads"
  ]
}
```

---

## DevOps Infrastructure Generation Pipeline

**Назначение:** Автоматическая генерация Dockerfile, Terraform конфигураций и buildspec.yaml.

**Компоненты:** Docker Generator, Terraform Generator (ECS/EKS), Buildspec Generator

### ASCII Диаграмма - Полный DevOps Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│         User: "Deploy my Java application to AWS"            │
│         Repository URL provided                              │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Step 1: Project Identification                  │
│   • Clone repository                                         │
│   • Detect project type (Java/Go/Node/Python/Rust)          │
│   • Find dependency files (pom.xml, go.mod, package.json)    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│      Step 2: Extract Project Information Prompt              │
│   Prompt: get_info_for_docker_file_prompt                    │
│   Input:                                                     │
│   • project_type: "java"                                     │
│   • dependency_object_content: {pom.xml content}             │
│   • project_files_list: [src/, pom.xml, ...]                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Extract Build Information                            │
│   Output:                                                    │
│   base_image: openjdk:11-jdk-slim                            │
│   app_name: bookstore-api                                    │
│   binary_name: bookstore-api-1.0.0-SNAPSHOT.jar              │
│   entry_point: java -jar /app/bookstore-api-1.0.0...jar     │
│   expose_port: EXPOSE 8080                                   │
│   build_artifact: target/bookstore-api-1.0.0-SNAPSHOT.jar    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Step 3: Generate Dockerfile                            │
│   Prompt: docker_file_generation_prompt_template             │
│   Input: Extracted build information                         │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Dockerfile Generated                                 │
│   # Build stage                                              │
│   FROM openjdk:11-jdk-slim AS build                          │
│   RUN apt-get update && apt-get install -y maven            │
│   WORKDIR /app                                               │
│   COPY . .                                                   │
│   RUN mvn clean package                                      │
│                                                               │
│   # Runtime stage                                            │
│   FROM openjdk:11-jre-slim                                   │
│   WORKDIR /app                                               │
│   COPY --from=build /app/target/bookstore-api-*.jar app.jar │
│   RUN useradd appuser && chown -R appuser:appuser /app      │
│   USER appuser                                               │
│   EXPOSE 8080                                                │
│   ENTRYPOINT ["java", "-jar", "app.jar"]                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Step 4: Build and Test Docker Image                    │
│   docker build -t app:test .                                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
    ┌──────────────────┐    ┌──────────────────────┐
    │  Build Success   │    │   Build Failed       │
    └────────┬─────────┘    └──────────┬───────────┘
             │                         │
             │                         ▼
             │              ┌─────────────────────────────────┐
             │              │ Step 4b: Fix Dockerfile         │
             │              │ Prompt: fix_dockerfile_...      │
             │              │ Input:                          │
             │              │ • docker_build_error            │
             │              │ • dockerfile_content            │
             │              └──────────┬──────────────────────┘
             │                         │
             │                         ▼
             │              ┌─────────────────────────────────┐
             │              │ Apply Package Manager Fix       │
             │              │ Retry Build                     │
             │              └──────────┬──────────────────────┘
             │                         │
             └─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│      Step 5: Choose Deployment Target                        │
│   User selects: ECS Fargate / ECS EC2 / EKS Fargate / EKS EC2│
└───────────────────────────┬─────────────────────────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
    ┌───────────┐    ┌───────────┐    ┌───────────┐
    │    ECS    │    │    EKS    │    │ Other...  │
    └─────┬─────┘    └─────┬─────┘    └───────────┘
          │                │
          ▼                ▼
┌──────────────────────────────────────────────────────────────┐
│   Step 6a: ECS Deployment Path                               │
│   Supervisor Prompt → Classify: "fargate" or "ec2-autoscaling"│
└───────────────────────────┬──────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Generate Task Definition from Dockerfile                  │
│   Prompt: task_definition_template                           │
│   • Extract FROM image                                       │
│   • Extract EXPOSE port                                      │
│   • Extract ENV variables                                    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Generate Complete Terraform Code                          │
│   Prompt: terraform_generation_fargate_template              │
│   Resources:                                                 │
│   • VPC, Subnets, IGW                                        │
│   • ECS Cluster                                              │
│   • Task Definition                                          │
│   • ECS Service                                              │
│   • Security Groups                                          │
│   • IAM Roles                                                │
│   • CloudWatch Logs                                          │
│   • (Optional) ALB                                           │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│   Step 6b: EKS Deployment Path                               │
│   Similar flow but generates:                                │
│   • EKS Cluster Terraform                                    │
│   • Kubernetes Manifests (Deployment, Service, Ingress)      │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Step 7: Generate Buildspec.yaml                        │
│   Prompt: buildspec_template                                 │
│   Input:                                                     │
│   • dockerfile_content                                       │
│   • ecr_repository_name                                      │
│   • ecr_repository_uri                                       │
│   • runtime_version (extracted from Dockerfile)              │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│          All Files Generated                                 │
│   ✓ Dockerfile                                               │
│   ✓ Terraform files (main.tf, variables.tf, outputs.tf)     │
│   ✓ buildspec.yaml                                           │
│   ✓ Kubernetes manifests (if EKS)                            │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│       Return Generated Code to User                          │
│   • Download as ZIP                                          │
│   • Instructions for deployment                              │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные между этапами

**Step 1 → Step 2:**
```json
{
  "project_type": "java",
  "dependency_files": {
    "pom.xml": "<content>",
    "src/": "..."
  },
  "project_structure": ["src/", "target/", "pom.xml"]
}
```

**Step 2 → Step 3:**
```
base_image: openjdk:11-jdk-slim
app_name: bookstore-api
binary_name: bookstore-api-1.0.0-SNAPSHOT.jar
entry_point: java -jar /app/bookstore-api-1.0.0-SNAPSHOT.jar
expose_port: EXPOSE 8080
build_artifact: target/bookstore-api-1.0.0-SNAPSHOT.jar
```

**Step 3 → Step 4:**
- Complete Dockerfile content

**Step 6a Task Definition Output:**
```hcl
container_definitions = jsonencode([
  {
    name      = "bookstore-api"
    image     = "bookstore-api:latest"
    cpu       = 512
    memory    = 1024
    essential = true
    portMappings = [
      {
        containerPort = 8080
        hostPort      = 8080
        protocol      = "tcp"
      }
    ]
    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"         = "/ecs/bookstore-api"
        "awslogs-region"        = "us-east-1"
        "awslogs-stream-prefix" = "ecs"
      }
    }
  }
])
```

---

## Performance Testing Pipeline

**Назначение:** Автоматическая генерация и выполнение JMeter performance тестов на основе архитектурной документации.

**Компоненты:** Architecture Analyzer, Scenario Generator, Test Executor, Results Analyzer

### ASCII Диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│   User: "Create performance tests for my API"                │
│   Provides: Architecture docs (Confluence/S3)                │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 1: Architecture Analysis                             │
│    Prompt: Architecture Analysis Prompt                      │
│    Input: Architecture documents (markdown/JSON/YAML)        │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Extract System Information                                │
│    Claude analyzes docs and extracts:                        │
│    • Components (services, databases, APIs)                  │
│    • API Endpoints (REST/GraphQL)                            │
│      - Path: /api/books                                      │
│      - Method: GET/POST/PUT/DELETE                           │
│      - Request/Response format                               │
│    • Data Flows (how components interact)                    │
│    • User Workflows (step-by-step business processes)        │
│    • NFRs (response time, concurrent users, throughput)      │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Analysis Result Saved to S3                               │
│    s3://bucket/perf-pipeline/{session_id}/analysis.json      │
│                                                               │
│    {                                                          │
│      "components": [...],                                    │
│      "endpoints": [                                          │
│        {                                                      │
│          "path": "/api/books",                               │
│          "method": "GET",                                    │
│          "description": "List all books"                     │
│        }                                                      │
│      ],                                                       │
│      "workflows": [                                          │
│        {                                                      │
│          "name": "User Registration Workflow",               │
│          "steps": [                                          │
│            {"api": "POST /api/users", "description": "..."}  │
│          ]                                                    │
│        }                                                      │
│      ],                                                       │
│      "nfrs": {                                               │
│        "max_concurrent_users": 1000,                         │
│        "max_test_duration": "5 minutes",                     │
│        "max_loops_per_user": 10,                             │
│        "target_response_time": "< 200ms"                     │
│      }                                                        │
│    }                                                          │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 2: Scenario Generation                               │
│    Prompt: Scenario Generation Prompt                        │
│    Input: Analysis result (workflows + NFRs)                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Generate Test Scenarios                                   │
│    Claude creates 3 types of scenarios:                      │
│                                                               │
│    1. Load Testing (normal load)                             │
│       • concurrent_users: 100 (from NFRs)                    │
│       • duration: 300 seconds (from NFRs)                    │
│       • loops_per_user: 10 (from NFRs)                       │
│       • api_sequence: [Step 1, Step 2, ...]                  │
│                                                               │
│    2. Stress Testing (beyond capacity)                       │
│       • concurrent_users: 150 (1.5x load)                    │
│       • duration: 300 seconds                                │
│       • ramp_up: faster                                      │
│                                                               │
│    3. Endurance Testing (sustained load)                     │
│       • concurrent_users: 100                                │
│       • duration: 1800 seconds (longer)                      │
│       • loops: unlimited                                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Scenarios Saved to S3                                     │
│    s3://bucket/perf-pipeline/{session_id}/scenarios.json     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 3: Generate JMeter Test Plans                        │
│    • Convert scenarios to Java DSL                           │
│    • Create Thread Groups                                    │
│    • Add HTTP Samplers for each API                          │
│    • Configure assertions and listeners                      │
│    • Compile to JMeter .jmx file                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 4: Execute Tests                                     │
│    • Run JMeter test plans                                   │
│    • Collect metrics (response times, throughput, errors)    │
│    • Generate raw results (CSV/JTL)                          │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Test Results Generated                                    │
│    {                                                          │
│      "summary": {                                            │
│        "total_requests": 10000,                              │
│        "success_rate": 99.5,                                 │
│        "avg_response_time": 145,                             │
│        "p95_response_time": 280,                             │
│        "throughput": 50.2                                    │
│      },                                                       │
│      "per_endpoint": {...},                                  │
│      "errors": [...]                                         │
│    }                                                          │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 5: AI Analysis of Results                            │
│    Prompt: Test Results Analysis Prompt                      │
│    Input: Test results JSON                                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Generate Comprehensive Analysis                           │
│    {                                                          │
│      "executive_summary": "System performed well...",        │
│      "performance_grade": "B",                               │
│      "key_findings": [                                       │
│        "Response time meets SLA (< 200ms target)",           │
│        "Error rate is acceptable (0.5%)",                    │
│        "/api/books endpoint shows latency spike"             │
│      ],                                                       │
│      "recommendations": [                                    │
│        "Add caching for /api/books endpoint",                │
│        "Consider database query optimization",               │
│        "Increase connection pool size"                       │
│      ],                                                       │
│      "risk_assessment": "Low risk - system stable",          │
│      "bottleneck_analysis": [                                │
│        "Database queries taking 80ms avg",                   │
│        "Network latency contributing 20ms"                   │
│      ],                                                       │
│      "scalability_assessment": "Can handle 2x load..."       │
│    }                                                          │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Return Analysis to User                                   │
│    • Performance grade                                       │
│    • Detailed findings                                       │
│    • Actionable recommendations                              │
│    • Full report in S3                                       │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные

**Step 1 Output (Architecture Analysis):**
```json
{
  "components": ["API Gateway", "ECS Service", "RDS"],
  "endpoints": [
    {"path": "/api/books", "method": "GET"},
    {"path": "/api/books", "method": "POST"}
  ],
  "workflows": [
    {
      "name": "Browse Books",
      "steps": [
        {"api": "GET /api/books", "order": 1},
        {"api": "GET /api/books/{id}", "order": 2}
      ]
    }
  ],
  "nfrs": {
    "max_concurrent_users": 1000,
    "max_test_duration": "5 minutes",
    "max_loops_per_user": 10
  }
}
```

**Step 2 Output (Scenarios):**
```json
{
  "scenarios": [
    {
      "name": "Load Test - Browse Books",
      "type": "load",
      "concurrent_users": 100,
      "duration_seconds": 300,
      "ramp_up_seconds": 60,
      "loops_per_user": 10,
      "api_sequence": [
        {"path": "/api/books", "method": "GET"},
        {"path": "/api/books/{id}", "method": "GET"}
      ]
    }
  ]
}
```

---

## Incident Management Pipeline

**Назначение:** Автоматический анализ и обработка инцидентов из Slack/PagerDuty.

**Компоненты:** Incident Analyzer (Claude Haiku), Slack Bot, PagerDuty Integration

### ASCII Диаграмма

```
┌─────────────────────────────────────────────────────────────┐
│         Incident Triggered                                   │
│  Source: Slack alert OR PagerDuty notification               │
│                                                               │
│  Example:                                                    │
│  "🚨 High CPU usage on prod-api-server-1                     │
│   Current: 95% | Threshold: 80%"                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Incident Data Collection                                  │
│    • Incident message                                        │
│    • Timestamp                                               │
│    • Source system (Slack/PagerDuty)                         │
│    • Severity (if available)                                 │
│    • Affected services                                       │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 1: AI Analysis (Claude Haiku)                        │
│    Multiple analyses run in parallel:                        │
│                                                               │
│    1. Root Cause Analysis                                    │
│    2. Severity Classification                                │
│    3. Remediation Suggestions                                │
│    4. Similar Incident Search                                │
│    5. Risk Level Assessment                                  │
│    6. Resolution Time Estimation                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
    ┌───────────────┐ ┌──────────┐ ┌────────────┐
    │ Root Cause    │ │ Severity │ │ Similar    │
    │ Analysis      │ │ Check    │ │ Incidents  │
    └───────┬───────┘ └────┬─────┘ └─────┬──────┘
            │              │              │
            └──────────────┼──────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│    AI Analysis Results                                       │
│    {                                                          │
│      "root_cause": {                                         │
│        "primary": "Memory leak in API service",              │
│        "contributing_factors": [                             │
│          "High traffic load",                                │
│          "Inefficient caching"                               │
│        ],                                                     │
│        "confidence": 0.85                                    │
│      },                                                       │
│      "severity": {                                           │
│        "level": "HIGH",                                      │
│        "impact": "Service degradation",                      │
│        "affected_users": "~1000 users"                       │
│      },                                                       │
│      "remediation": [                                        │
│        {                                                      │
│          "action": "Restart affected containers",            │
│          "priority": 1,                                      │
│          "command": "kubectl rollout restart deploy/api"     │
│        },                                                     │
│        {                                                      │
│          "action": "Increase memory limits",                 │
│          "priority": 2,                                      │
│          "details": "Update limits to 2Gi"                   │
│        }                                                      │
│      ],                                                       │
│      "similar_incidents": [                                  │
│        {                                                      │
│          "id": "INC-2024-001",                               │
│          "similarity": 0.92,                                 │
│          "resolution": "Memory limit increase"               │
│        }                                                      │
│      ],                                                       │
│      "risk_level": "MEDIUM",                                 │
│      "estimated_resolution_time": "30 minutes"               │
│    }                                                          │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 2: Generate Structured Response                      │
│    Format analysis results for:                              │
│    • Slack notification (markdown)                           │
│    • PagerDuty incident notes                                │
│    • Incident database entry                                 │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Post Response to Slack                                    │
│                                                               │
│    🔍 **Incident Analysis Results**                          │
│                                                               │
│    **Root Cause:** Memory leak in API service (85% confident)│
│                                                               │
│    **Severity:** HIGH - Service degradation affecting ~1000  │
│    users                                                      │
│                                                               │
│    **Recommended Actions:**                                  │
│    1️⃣ Restart affected containers (Priority 1)              │
│       `kubectl rollout restart deploy/api`                   │
│    2️⃣ Increase memory limits to 2Gi (Priority 2)            │
│                                                               │
│    **Similar Incident:** INC-2024-001 (92% match)            │
│    Resolution: Memory limit increase                         │
│                                                               │
│    **Estimated Resolution Time:** 30 minutes                 │
│                                                               │
│    **Risk Level:** MEDIUM                                    │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 3: Execute Auto-Remediation (if enabled)             │
│    • Check if auto-remediation is allowed                    │
│    • Execute safe remediation steps automatically            │
│    • Log all actions                                         │
│    • Notify team of actions taken                            │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 4: Update Incident Tracking                          │
│    • Store incident in database                              │
│    • Link to similar incidents                               │
│    • Track resolution progress                               │
│    • Update PagerDuty with analysis                          │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Step 5: Post-Incident Learning                            │
│    • Add to incident knowledge base                          │
│    • Update similar incident index                           │
│    • Generate incident report                                │
│    • Suggest preventive measures                             │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│    Incident Resolution Confirmed                             │
│    • Notify team in Slack                                    │
│    • Close PagerDuty incident                                │
│    • Archive analysis and logs                               │
└─────────────────────────────────────────────────────────────┘
```

### Передаваемые данные

**Input (Incident Alert):**
```json
{
  "source": "slack",
  "channel": "#alerts-prod",
  "message": "🚨 High CPU usage on prod-api-server-1\nCurrent: 95% | Threshold: 80%",
  "timestamp": "2025-01-15T14:30:00Z",
  "severity": "high"
}
```

**AI Analysis Output:**
```json
{
  "incident_id": "INC-2025-042",
  "root_cause": {
    "primary": "Memory leak in API service",
    "contributing_factors": ["High traffic", "Inefficient caching"],
    "confidence": 0.85
  },
  "severity": {
    "level": "HIGH",
    "impact": "Service degradation",
    "affected_users": "~1000"
  },
  "remediation": [
    {
      "action": "Restart affected containers",
      "priority": 1,
      "command": "kubectl rollout restart deploy/api",
      "safe_to_automate": true
    }
  ],
  "similar_incidents": [
    {"id": "INC-2024-001", "similarity": 0.92}
  ],
  "risk_level": "MEDIUM",
  "estimated_resolution_time": "30 minutes"
}
```

---

## Summary

Эти пайплайны демонстрируют, как AI-агент обрабатывает различные задачи в процессе разработки ПО:

1. **SDLC Phase-Specific** - базовый пайплайн для контекстной помощи
2. **Epic Creation** - извлечение и структурирование требований
3. **Domain Analysis + OpenAPI** - от бизнес-анализа до API спецификации
4. **Architecture Design** - создание диаграмм и оценка стоимости
5. **DevOps Infrastructure** - полная автоматизация создания инфраструктуры
6. **Performance Testing** - от документов до тестов и анализа результатов
7. **Incident Management** - автоматический анализ и разрешение проблем

Каждый пайплайн использует цепочку промтов, где выходные данные одного промта становятся входными для следующего, создавая полный автоматизированный workflow.

