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

