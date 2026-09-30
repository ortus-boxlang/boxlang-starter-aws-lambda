# ⚡ BoxLang AWS Lambda Template

```
|:------------------------------------------------------:|
| ⚡︎ B o x L a n g ⚡︎
| Dynamic : Modular : Productive
|:------------------------------------------------------:|
```

<blockquote>
	Copyright Since 2023 by Ortus Solutions, Corp
	<br>
	<a href="https://www.boxlang.io">www.boxlang.io</a> |
	<a href="https://www.ortussolutions.com">www.ortussolutions.com</a>
</blockquote>

<p>&nbsp;</p>

🚀 **Production-ready template** for building and deploying BoxLang applications to AWS Lambda. This template provides a complete Java-BoxLang hybrid runtime with Gradle build automation, comprehensive testing, and AWS SAM deployment scripts.

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   AWS Lambda    │    │  LambdaRunner    │    │  BoxLang App    │
│    Runtime      │───▶│  (Java Wrapper)  │───▶│  (Lambda.bx)    │
│   (Java 21)     │    │                  │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
  Handler Invocation    Event Processing        Business Logic
  Context Management    Response Formatting     Application Lifecycle
```

**Key Components:**

- 📦 **BoxLang AWS Lambda Runtime**: Maven dependency `io.boxlang:boxlang-aws-lambda:1.4.0`
- 🎯 **Lambda Handler**: `ortus.boxlang.runtime.aws.LambdaRunner::handleRequest`
- 📝 **BoxLang Code**: Lives in `src/main/bx/` (Lambda.bx, Application.bx)
- ⚙️ **Java Wrapper**: Gradle build system with shadow JAR packaging
- 🏗️ **Deployment**: ZIP artifact for AWS Lambda with SAM template

The main lambda code is located in `src/main/bx/Lambda.bx` and the test code is located in `src/test/java/com/myproject`.

## 🚀 Quick Start

### Prerequisites

- ☕ **Java 21+** (required for runtime)
- 🔧 **AWS CLI** (for deployment)
- 🏗️ **SAM CLI** (for local testing)

### Get Started in 2 Steps

1. **🧪 Run tests to verify setup:**

   ```bash
   ./gradlew test
   ```

2. **📦 Build deployment package:**

   ```bash
   ./gradlew build
   ```

Your deployable ZIP will be created at `build/distributions/boxlang-aws-project-<version>.zip`

### 🔥 Deploy to AWS

#### 1️⃣ **Setup Configuration**

Copy the config template and customize for your environment:

```bash
cp workbench/config.env workbench/config.local.env
# Edit config.local.env with your settings
```

Key settings:

```conf
# BoxLang Lambda Deployment Configuration
# Copy this file to config.local.env and customize for your environment

# S3 Bucket for Lambda artifacts (required)
# Must be globally unique, 3-63 characters, lowercase letters/numbers/hyphens only
AWS_LAMBDA_BUCKET=my-boxlang-lambda-artifacts

# AWS Region (optional - uses your default AWS region if not set)
AWS_REGION=us-east-1

# CloudFormation Stack Name (optional - defaults to 'boxlang-lambda-stack')
STACK_NAME=my-boxlang-lambda

# Lambda Function Name for direct invocation (optional)
# Use this to invoke by function name instead of looking up via stack
FUNCTION_NAME=my-boxlang-lambda

# Lambda Function Configuration (optional)
LAMBDA_MEMORY=128
LAMBDA_TIMEOUT=15

# Deployment Environment (optional)
ENVIRONMENT=dev
```

#### 2️⃣ **Create S3 Bucket**

```bash
./workbench/1-create-bucket.sh
```

#### 3️⃣ **Deploy Lambda**

```bash
./workbench/2-deploy.sh
```

This will:

- Build your BoxLang project
- Upload to S3
- Deploy via CloudFormation with your configured parameters
- Show function name and settings

#### 4️⃣ **Test Your Lambda**

```bash
./workbench/3-invoke.sh
```

> 💡 **Pro Tip**: The deployment scripts automatically use your configuration from `config.local.env` → `config.env` → environment variables and the workbench/sampleEvents/event-live.json

## 🤖 CI/CD Workflows

This template includes **complete GitHub Actions workflows** for automated testing, building, and deployment. All workflows are production-ready and can be used immediately.

### 📋 Available Workflows

#### 🧪 **Test Workflow** (`.github/workflows/tests.yml`)

**Purpose**: Reusable testing workflow with comprehensive validation

**Features**:

- ☕ **Java 21** setup with caching
- 🔍 **Full test suite** execution via Gradle
- 📊 **Test reports** as workflow artifacts
- 🔄 **Reusable** across multiple workflows

**Usage**: Automatically called by other workflows or can be triggered manually

#### 🚀 **Release Workflow** (`.github/workflows/release.yml`)

**Purpose**: Complete build, test, and optional AWS deployment pipeline

**Features**:

- 🏗️ **Full build** with shadow JAR and Lambda ZIP creation
- 🧪 **Test execution** via reusable test workflow
- 📦 **Build artifacts** uploaded (ZIP, JAR, test reports)
- 🔧 **AWS Lambda deployment** (commented out, ready to enable)
- 📤 **S3 distribution upload** (commented out, ready to enable)
- 🏷️ **GitHub release creation** (commented out, ready to enable)

**Triggers**:

- Push to `main` branch
- Manual workflow dispatch

**💡 AWS Deployment**: Ready-to-use Lambda function updates:

```yaml
# Uncomment these lines in .github/workflows/release.yml for AWS deployment:

- name: Update AWS Lambda Function
  uses: kazimanzurrashid/aws-lambda-update-action@v2.0.3
  with:
    zip-file: "./build/distributions/${{ env.PROJECT_NAME }}-${{ env.VERSION }}.zip"
    lambda-name: ${{ env.PROJECT_NAME }}-${{ env.DEPLOY_TIER }}
  env:
    AWS_REGION: ${{ secrets.AWS_REGION }}
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_PUBLISHER_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_PUBLISHER_KEY }}
```

> ⚠️ **Important**: You must create the Lambda function FIRST (using the workbench scripts), then uncomment the AWS deployment workflow to enable automatic function updates on releases.

#### 📸 **Snapshot Workflow** (`.github/workflows/snapshot.yml`)

**Purpose**: Development builds and testing for non-release branches

**Features**:

- 🔧 **Development builds** with snapshot versioning
- 🧪 **Test execution** for validation
- 📦 **Snapshot artifacts** for testing
- 🌿 **Branch-based** development workflow

**Triggers**:

- Push to any branch except `main`
- Pull requests

### 🛠️ Setting Up CI/CD

#### **1. GitHub Secrets (for AWS deployment)**

Add these secrets to your GitHub repository (Settings → Secrets and variables → Actions):

```bash
AWS_REGION                    # e.g., us-east-1
AWS_PUBLISHER_KEY_ID         # AWS Access Key ID
AWS_SECRET_PUBLISHER_KEY     # AWS Secret Access Key
```

#### **2. Enable AWS Deployment**

1. **Deploy function first** using workbench scripts:

   ```bash
   ./workbench/1-create-bucket.sh
   ./workbench/2-deploy.sh
   ```

2. **Uncomment AWS deployment** in `.github/workflows/release.yml`

3. **Push to main** or trigger workflow manually

#### **3. Workflow Architecture**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Push to main   │    │  Release Flow   │    │  AWS Lambda     │
│  (or manual)    │───▶│   • Build       │───▶│   • Update      │
│                 │    │   • Test        │    │   • Deploy      │
└─────────────────┘    │   • Package     │    └─────────────────┘
                       └─────────────────┘
                               │
                               ▼
                       ┌─────────────────┐
                       │   Artifacts     │
                       │   • Lambda ZIP  │
                       │   • Shadow JAR  │
                       │   • Test Reports│
                       └─────────────────┘
```

> 🎯 **Pro Tip**: The workflows are designed for zero-configuration operation. Just create your GitHub repository, add AWS secrets if needed, and push code. The CI/CD pipeline handles the rest!

## 📁 Directory Structure

Here is a comprehensive overview of the project structure:

### 🗂️ Root Files & Folders

| File/Folder | Required | In Git | Description |
|-------------|----------|--------|-------------|
| 📁 `build/` | ❌ | ❌ | Temporary build assets (generated by Gradle) |
| 📁 `gradle/` | ✅ | ✅ | Gradle wrapper and configuration |
| 📁 `src/` | ✅ | ✅ | Your module source code |
| 📁 `workbench/` | ✅ | ✅ | AWS deployment scripts and SAM template |
| 📄 `.cfformat.json` | ❌ | ✅ | CFFormat configuration (Ortus Standards) |
| 📄 `.editorconfig` | ❌ | ✅ | Editor consistency settings |
| 📄 `.gitattributes` | ❌ | ✅ | Git attributes |
| 📄 `.gitignore` | ✅ | ✅ | Git ignore patterns |
| 📄 `.markdownlint.json` | ❌ | ✅ | Markdown linting rules |
| 📄 `.ortus-java-style.xml` | ❌ | ✅ | Java style guide (IntelliJ/VSCode/Eclipse) |
| 📄 `box.json` | ✅ | ✅ | BoxLang dependencies and automations |
| 📄 `build.gradle` | ✅ | ✅ | Gradle build configuration |
| 📄 `changelog.md` | ❌ | ✅ | Version history tracking |
| 📄 `gradlew` | ✅ | ✅ | Gradle wrapper (Linux/macOS) |
| 📄 `gradlew.bat` | ✅ | ✅ | Gradle wrapper (Windows) |
| 📄 `settings.gradle` | ✅ | ✅ | Gradle project settings |

> 📝 **Legend**: ✅ = Required/Included, ❌ = Optional/Excluded

### 📂 Source Directory (`src/`)

- 🧑‍💻 **`main/`** - Production source code
  - `bx/` - BoxLang source files
    - `Application.bx` - Application lifecycle hooks and configuration
    - `Lambda.bx` - **Your Lambda entry point** (implements `run(event, context, response)`)
    - `handlers/` - **Routed handlers** - see [URI Routing with handlers/](#-uri-routing-with-handlers) below
    - `manifest.json` - **Generated** (gitignored) by `./gradlew generateManifest` from `handlers/`; do not edit by hand
    - _(Add your BoxLang classes here)_
- 🧪 **`test/`** - Test source code
  - `java/com/myproject/` - JUnit test classes
    - `LambdaIntegrationTest.java` - Comprehensive Lambda integration tests
    - `mocks/TestContext.java` - Mock AWS Lambda Context
    - `mocks/TestLogger.java` - Test logging utilities
    - `runner/LocalLambdaRunner.java` - Local Lambda test runner
- 🗂️ **`resources/`** - Runtime resources
  - ⚙️ `boxlang.json` - BoxLang runtime configuration
  - 📦 `boxlang_modules/` - Local BoxLang modules (auto-packaged)
  - 🔧 `libs/` - Any non-maven managed Jars that you want to include in your project

### 🔬 Workbench Directory (`workbench/`)

The workbench provides a complete AWS deployment and testing workflow:

#### 📋 Core Scripts

- 🛠️ `0-check-aws.sh` - **AWS credentials troubleshooter** (diagnoses configuration issues)
- 🪣 `1-create-bucket.sh` - **Creates S3 bucket** for deployment artifacts
- 🚀 `2-deploy.sh` - **Builds and deploys** Lambda via SAM/CloudFormation
- 📞 `3-invoke.sh` - **Invokes deployed Lambda** with test payloads
- 🧹 `4-cleanup.sh` - **Cleanup** deployment resources and stack

#### ⚙️ Configuration

- 📝 `config.env` - **Default configuration template** (copy to `config.local.env`)
- 🔐 `config.local.env` - **Your local settings** (gitignored)

#### 🎯 Templates & Events

- 📋 `template.yml` - **SAM template** for AWS deployment (parameterized)
- 🧪 `template-local.yml` - **SAM template** for local development with HTTP endpoints
- 📄 `event-live.json` - **Production test payload** for Lambda invocation
- 📁 `sampleEvents/` - **Sample events** (API Gateway, S3, etc.)

## 💻 Lambda Development Guide

### 🎯 Writing Your Lambda Function

Your main Lambda logic goes in `src/main/bx/Lambda.bx`. The entry point **must** be named `run`:

```java
/**
 * Lambda Entry Point
 * Convention: run(event, context, response)
 */
class {
    function run( event, context, response ) {
        response.body = {
            "error": false,
            "messages": [],
            "data": "Hello from BoxLang Lambda! Event: " & event.toString()
        }
        response.statusCode = 200
    }

    // Alternative function (call with x-bx-function header)
    function anotherFunction( event, context, response ) {
        return "Alternative lambda function!"
    }
}
```

### 📋 Function Parameters

- **`event`**: AWS Lambda event object (API Gateway, S3, etc.)
- **`context`**: AWS Lambda context (`com.amazonaws.services.lambda.runtime.Context`)
- **`response`**: Response object with standard structure:
  - `statusCode` - HTTP status (default: 200)
  - `headers` - HTTP headers map/struct
  - `body` - Response payload (your data goes here)
  - `cookies` - Array of response cookies

### 🧭 URI Routing with `handlers/`

Every request goes to `Lambda.bx` by default. To route requests to a different
class based on the URL path, add it under `src/main/bx/handlers/` instead of the
project root:

```java
// src/main/bx/handlers/Products.bx
class {
    function run( event, context, response ) {
        response.body = { "message": "Hello from Products" }
        response.statusCode = 200
    }
}
```

A request to `/products` now runs `handlers/Products.bx` instead of `Lambda.bx`.
Folders can be nested and use any case you like - only the leaf `.bx` filename
needs to be PascalCase - so `handlers/api/Test.bx` handles `/api/test`.

**Only files under `handlers/` are ever routable.** `Application.bx`, `Lambda.bx`,
and anything else at the project root can never be reached this way, no matter
what path or `x-bx-function` header a client sends - the routing table is a
build-time allowlist (`manifest.json`, generated from `handlers/` by
`./gradlew generateManifest`, wired automatically into `runLocal`, `test`, and
`buildLambdaZip`), not a live filesystem scan. A request to a route that isn't
registered just runs `Lambda.bx`, same as no routing had happened at all.

As with `Lambda.bx`, the `x-bx-function` header can call an alternative method on
a handler - only ever a method you declared, since BoxLang's public/remote scope
rules are exactly what gates it: don't make a method public if you don't want it
externally callable.

If `handlers/manifest.json` is ever missing or invalid, the runtime falls back to
scanning `handlers/` directly at cold start and logs a `WARNING` listing every
handler it registered - check your Lambda logs if routing behaves unexpectedly
after a deploy that skipped `generateManifest`. If neither `manifest.json` nor
`handlers/` exists at all, the runtime falls back further to scanning the
project root for backward compatibility with pre-`handlers/` deployments; set
`BOXLANG_ENABLE_ROOT_SCAN=false` to disable that last-resort scan entirely and
restrict routing to the default `Lambda.bx` handler only.

`manifest.json`'s `reserved` and `defaultHandler` fields are enforced by the
runtime, not just documentation - a manifest can never route to a reserved
file (`Application.bx`, `Lambda.bx`, or anything else it lists), and
`defaultHandler.file`/`method` is honored as the fallback handler for
unmatched routes when present.

### 🔧 Application Lifecycle

Use `src/main/bx/Application.bx` for initialization and request processing:

```java
class {
    this.name = "My-AWS-Lambda"

    function onApplicationStart() {
        // Initialize databases, caches, etc.
        return true;
    }

    function onRequestStart( targetPage ) {
        // Per-request initialization
        return true;
    }
}
```

## ⚙️ Configuration

### 🔧 Project Properties

**Project Name**: Defined in `settings.gradle`

```gradle
rootProject.name = 'boxlang-aws-project'
```

**Versions**: Configured in `gradle.properties` and `build.gradle`

```properties
# gradle.properties
version=1.0.0
jdkVersion=21
boxlangVersion=1.0.0
```

### 🎛️ BoxLang Runtime Configuration

Configure BoxLang behavior in `src/resources/boxlang.json`:

```jsonc
{
  "debugMode": false,          // Enable for development
  "trustedCache": true,        // Enable for production
  "requestTimeout": "0,0,15,0", // 15 minute timeout (matches AWS Lambda)
  "timezone": "${env:TZ:UTC}",  // Use TZ environment variable or UTC
  "logging": {
    "rootLevel": "WARN",       // ERROR, WARN, INFO, DEBUG, TRACE
    "loggers": {
      "runtime": { "level": "INFO", "appender": "console" }
    }
  }
}
```

**Key Settings for AWS Lambda:**

- 🔒 **`trustedCache: true`** - Enables class caching (recommended for production)
- 🕐 **`requestTimeout`** - Should match your Lambda timeout
- 🌍 **`timezone`** - Use `${env:TZ:UTC}` to respect Lambda environment
- 📝 **`logging.*.appender: "console"`** - Required for CloudWatch logs

## 📦 BoxLang Modules

### 🔌 Adding Modules

**Method 1: CommandBox CLI**

You can leverage the CommandBox CLI to install BoxLang modules easily.

```bash
# Install to src/resources/boxlang_modules/
box install {moduleName} --production --directory=src/resources/boxlang_modules
```

**Method 2: Via box.json**

CommandBox will read your `box.json` file for module dependencies.  This is a durable way to manage your dependencies.

```jsonc
{
  "dependencies": {
    "bx-pdf": "^1.0.0",
	"bx-mysql": "^1.0.0"
  },
  "installPaths": {
    "bx-pdf": "src/resources/boxlang_modules/bx-pdf",
	"bx-mysql": "src/resources/boxlang_modules/bx-mysql"
  }
}
```

Then run:

```bash
box install --production
```

Remember that these dependencies should not be added to your source control (e.g., Git).

**Method 3: BoxLang Module Installer**

The BoxLang module installer is an OS level CLI tool that is used to install modules easily without CommandBox.  Eventually, this tool and CommandBox will merge, but it is here for reference.

```bash
cd src/resources
install-bx-module <module-name> --local
```

### 📂 Module Structure

Modules are automatically packaged into your deployment ZIP:

```
build/distributions/your-lambda.zip
├── Lambda.bx
├── boxlang.json
├── lib/
│   └── (jars)
└── boxlang_modules/          # ← Your modules go here
    ├── bx-pdf/
    └── bx-mysql/
```

## 🔨 Build System & Tasks

### 🚀 Essential Commands

**Development Workflow:**

```bash
# 1. Run tests
./gradlew test

# 2. Build deployment package
./gradlew build

# 3. Deploy to AWS
./workbench/2-deploy.sh
```

### 📋 All Available Tasks

| Task | Description | Output |
|------|-------------|---------|
| 🏗️ `build` | Full build lifecycle (clean, compile, test, package) | `build/distributions/*.zip` |
| 🧹 `clean` | Delete build artifacts and temporary files | - |
| ☕ `compileJava` | Compile Java source files | `build/classes/java/main/` |
| 🧪 `compileTestJava` | Compile Java test files | `build/classes/java/test/` |
| 📊 `dependencyUpdates` | Check for newer dependency versions | Console report |
| 🔗 `shadowJar` | Create uber-JAR with all dependencies | `build/distributions/` |
| 🧭 `generateManifest` | Scan `handlers/` and (re)generate `manifest.json` | `src/main/bx/manifest.json` |
| 🎁 `buildLambdaZip` | Package Lambda deployment ZIP | `build/distributions/*.zip` |
| 📄 `jar` | Create standard JAR (without dependencies) | `build/libs/` |
| 📚 `javadoc` | Generate Java API documentation | `build/docs/javadoc/` |
| 🧪 `test` | Run JUnit tests | `build/reports/tests/` |
| 🏠 `runLocal` | **Run Lambda locally with default event** | Console output |
| 🌐 `runLocalApi` | **Run Lambda locally with API Gateway event** | Console output |
| 🔧 `runLocalLegacy` | **Run Lambda locally with legacy API event** | Console output |
| 🌍 `startSamServer` | **Start SAM local API server (foreground)** | HTTP server at :3000 |
| 🔄 `startSamServerBackground` | **Start SAM local API server (background)** | HTTP server at :3000 |
| 🛑 `stopSamServer` | **Stop background SAM server** | - |
| ✨ `spotlessApply` | Auto-format source code | - |
| 🔍 `spotlessCheck` | Check code formatting | - |
| 📋 `tasks` | List all available Gradle tasks | - |

### 🎯 Key Build Outputs

```
build/distributions/
├── boxlang-aws-project-1.0.0.zip     # 🚀 Deploy this to AWS Lambda
├── boxlang-aws-project-1.0.0-all.jar # 📦 Shadow JAR (uber-JAR)
└── boxlang-aws-project-1.0.0/        # 📁 Unpacked contents
    ├── Lambda.bx                      # Your BoxLang entry point
    ├── boxlang.json                   # Runtime configuration
    ├── lib/                          # All Java dependencies
    └── boxlang_modules/              # BoxLang modules
```

## 🧪 Testing

### 🎯 Test Structure

```
src/test/java/com/myproject/
├── LambdaIntegrationTest.java    # Comprehensive integration tests
├── mocks/
│   ├── TestContext.java         # Mock AWS Lambda Context
│   └── TestLogger.java          # Test logging utilities
└── runner/
    └── LocalLambdaRunner.java   # Local Lambda test runner
```

### ✅ Running Tests

```bash
# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests "com.myproject.LambdaIntegrationTest"

# Run with detailed output
./gradlew test --info
```

### 📊 Test Reports

After running tests, view results at:

- 📄 **HTML Report**: `build/reports/tests/test/index.html`
- 📊 **XML Results**: `build/test-results/test/`

### 🧪 Sample Test

```java
@Test
@DisplayName("Test Lambda.bx execution")
public void testValidLambda() throws IOException {
    Path validPath = Path.of("src", "main", "bx", "Lambda.bx");
    LambdaRunner runner = new LambdaRunner(validPath, true);
    Context context = new TestContext();

    var event = new HashMap<String, Object>();
    event.put("name", "Ortus Solutions");
    event.put("when", Instant.now().toString());

    IStruct response = runner.handleRequest(event, context);

    assertThat(response.getAsInteger(Key.of("statusCode"))).isEqualTo(200);
}
```

### 🏠 Local Testing

Test your Lambda locally without deploying to AWS:

#### 🚀 Quick Local Testing

```bash
# Test locally with default event
./gradlew runLocal

# Test with API Gateway event
./gradlew runLocalApi

# Test with custom event file
./gradlew runLocal -PeventFile=workbench/sampleEvents/s3-event.json
```

#### 🌐 SAM Local Development Server

If you have [SAM CLI installed](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html):

```bash
# Start local API server (HTTP endpoint testing)
./workbench/5-test-local.sh
```

This creates a local HTTP endpoint at `http://localhost:3000` where you can:

- Send HTTP requests directly to your Lambda
- Test API Gateway integration locally
- Debug with hot reload capabilities

#### 📁 Available Sample Events

```
workbench/sampleEvents/
├── api.json         # API Gateway HTTP v2.0 event
├── api-post.json    # POST request with JSON body
├── event.json       # Legacy API Gateway event
├── event-live.json  # Production test payload (used by 3-invoke.sh)
├── event-local.json # Local development test payload
└── s3-event.json    # S3 bucket notification event
```

#### 🔧 Local Test Runner Features

The built-in `LocalLambdaRunner` provides:

- ⚡ **Fast execution** - No deployment required
- 📊 **Performance metrics** - Shows execution time
- 🔍 **JSON output formatting** - Pretty-printed responses
- 📄 **Multiple event support** - Easy event switching
- ❌ **Error handling** - Clear error messages with stack traces

#### 🧪 Enhanced Integration Tests

The `LambdaIntegrationTest` suite includes:

- **Basic Lambda execution** - Validates core functionality
- **API Gateway simulation** - Tests HTTP request/response handling
- **Complex nested data** - Handles rich JSON payloads
- **Performance testing** - Large payload stress testing (5-second timeout)
- **Error handling** - Null value and edge case testing
- **Mock AWS Context** - Realistic Lambda environment simulation

**Test scenarios covered:**

```java
@Test void testBasicExecution()        // Core Lambda functionality
@Test void testApiGatewayEvent()       // HTTP API simulation
@Test void testComplexEvent()          // Nested JSON structures
@Test void testLargePayload()          // Performance validation
@Test void testNullHandling()          // Edge case testing
```

#### 🌐 SAM Local HTTP Server

For **HTTP endpoint testing**, start a local API Gateway simulation:

**Background Server (Recommended for Development):**

```bash
# Start server in background
./gradlew startSamServerBackground

# Your Lambda is now available at http://localhost:3000
curl http://localhost:3000
curl -X POST http://localhost:3000 -d '{"test":"data"}' -H 'Content-Type: application/json'

# Stop when done
./gradlew stopSamServer
```

**Foreground Server:**

```bash
# Start server in foreground (blocks terminal)
./gradlew startSamServer
# Press Ctrl+C to stop
```

**Test Endpoints:**

```bash
# Manual testing examples with curl
curl http://localhost:3000
curl -X POST http://localhost:3000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"John","email":"john@example.com"}'

# Or use your favorite HTTP client:
# - Postman
# - Insomnia
# - HTTPie: http POST localhost:3000/api/users name=John email=john@example.com
```

**Development Workflow:**

1. 🚀 `./gradlew startSamServerBackground` - Start server
2. ✏️ Edit `src/main/bx/Lambda.bx` - Make changes
3. 🔄 `./gradlew build` - Rebuild (server auto-reloads)
4. 🧪 `curl http://localhost:3000` - Test changes
5. 🔁 Repeat steps 2-4 for rapid development

**Requirements:**

- SAM CLI installed: `brew install aws-sam-cli`
- AWS credentials configured (can use dummy values for local testing)

## 🚀 AWS Deployment

### 🛠️ Deployment Scripts

The `workbench/` directory provides complete deployment automation:

| Script | Purpose | Requirements |
|--------|---------|--------------|
| 🪣 `1-create-bucket.sh` | Create S3 bucket for artifacts, skip if defined already | AWS CLI configured |
| 🚀 `2-deploy.sh` | Build & Deploy via CloudFormation | S3 bucket must exist & AWS CLI Configured |
| 📞 `3-invoke.sh` | Test deployed Lambda | Lambda deployed |
| 🧹 `4-cleanup.sh` | Remove AWS resources | - |

### 📋 SAM Template

The deployment uses `workbench/template.yml` with configurable parameters:

```yaml
Parameters:
  LambdaMemorySize:
    Type: Number
    Default: 128
    Description: Memory allocated to the Lambda function (MB)

  LambdaTimeout:
    Type: Number
    Default: 15
    Description: Lambda function timeout (seconds)

  Environment:
    Type: String
    Default: "dev"
    Description: Deployment environment (dev, staging, prod)

Resources:
  bxFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: build/distributions/boxlang-aws-project-1.0.0.zip
      Handler: ortus.boxlang.runtime.aws.LambdaRunner::handleRequest
      Runtime: java21
      Timeout: !Ref LambdaTimeout
      MemorySize: !Ref LambdaMemorySize
      Environment:
        Variables:
          ENVIRONMENT: !Ref Environment
          BOXLANG_DEBUG: "false"

Outputs:
  FunctionName:
    Description: "BoxLang Lambda Function Name"
    Value: !Ref bxFunction
```

> 💡 **Configuration**: The deploy script passes your `config.local.env` values as CloudFormation parameters

### 🎯 Testing Your Deployed Lambda

#### **Simple Invocation**
```bash
# Test with production payload (uses event-live.json)
./workbench/3-invoke.sh
```

The `3-invoke.sh` script:
- Loads your configuration from `config.local.env`
- Uses the `FUNCTION_NAME` setting (or derives it from `STACK_NAME`)
- Invokes with `event-live.json` as test payload
- Shows the response and execution details

#### **Manual AWS CLI Invocation**
```bash
# Test with custom payload
aws lambda invoke --function-name your-function \
  --payload '{"name":"test","data":"hello"}' \
  response.json && cat response.json

# List your functions
aws lambda list-functions --query 'Functions[?contains(FunctionName, `boxlang`)].FunctionName'
```

#### **Configuration Variables in Action**

Your `config.local.env` settings are used throughout the deployment:

```bash
# Example config.local.env
STACK_NAME=my-api-stack
AWS_LAMBDA_BUCKET=my-unique-bucket-name
LAMBDA_MEMORY=512          # Higher memory for performance
LAMBDA_TIMEOUT=30          # Longer timeout for complex operations
FUNCTION_NAME=my-api-stack-bxFunction-ABC123DEF456
ENVIRONMENT=production
```

When you deploy, you'll see:
```
⚙️  Lambda Memory: 512MB, Timeout: 30s, Environment: production
```

## ⚙️ Configuration Reference

### 📋 **Complete Configuration Variables**

The template uses a hierarchical configuration system: `config.local.env` → `config.env` → environment variables

#### **Required Settings**
| Variable | Description | Example | Used By |
|----------|-------------|---------|---------|
| `AWS_LAMBDA_BUCKET` | S3 bucket for artifacts (globally unique) | `my-company-lambda-artifacts` | All deployment scripts |

#### **Stack & Function Settings**
| Variable | Default | Description | Used By |
|----------|---------|-------------|---------|
| `STACK_NAME` | `boxlang-lambda-stack` | CloudFormation stack name | `2-deploy.sh`, `4-cleanup.sh` |
| `FUNCTION_NAME` | `{STACK_NAME}-bxFunction-{SUFFIX}` | Direct function name for invocation | `3-invoke.sh` |

#### **Lambda Runtime Settings**
| Variable | Default | Description | Constraints |
|----------|---------|-------------|-------------|
| `LAMBDA_MEMORY` | `128` | Memory allocation (MB) | 128-10240 MB |
| `LAMBDA_TIMEOUT` | `15` | Timeout in seconds | 1-900 seconds |
| `ENVIRONMENT` | `dev` | Deployment environment tag | dev/staging/prod |

#### **AWS Settings**
| Variable | Default | Description | Used By |
|----------|---------|-------------|---------|
| `AWS_REGION` | *your default* | AWS region for deployment | All AWS CLI operations |

### 🔧 **Configuration Setup**

#### **1. Copy Template**
```bash
cp workbench/config.env workbench/config.local.env
```

#### **2. Edit Your Settings**
```bash
# config.local.env (this file is gitignored)
AWS_LAMBDA_BUCKET=my-unique-bucket-2024
STACK_NAME=my-api-prod
LAMBDA_MEMORY=256
LAMBDA_TIMEOUT=30
ENVIRONMENT=production
```

#### **3. Verify Configuration**
```bash
# Check if AWS is configured
./workbench/0-check-aws.sh

# Deploy with your config
./workbench/2-deploy.sh
```

### 🎯 **Configuration Best Practices**

- ✅ **Use `config.local.env`** - Keep your settings out of git
- ✅ **Unique bucket names** - Include company/project prefix
- ✅ **Environment-specific stacks** - `my-app-dev`, `my-app-prod`
- ✅ **Resource sizing** - Start small, scale up based on needs
- ✅ **Timeout alignment** - Match Lambda timeout with `boxlang.json` settings

## 💫 Complete Development Workflow

### 📋 **From Zero to Deployed**

#### **1. Initial Setup**

```bash
# Clone or download the template
git clone <your-repo>
cd boxlang-aws-lambda-template

# Configure your deployment settings
cp workbench/config.env workbench/config.local.env
# Edit config.local.env with your AWS settings
```

#### **2. Development Cycle**

```bash
# Test locally during development
./gradlew runLocal                    # Quick local test
./gradlew test                       # Run full test suite

# Test with HTTP endpoints (if SAM CLI installed)
./gradlew startSamServerBackground   # Start local API server
curl http://localhost:3000           # Test your endpoints
./gradlew stopSamServer             # Stop when done
```

#### **3. Pre-Deployment Validation**

```bash
# Verify AWS configuration
./workbench/0-check-aws.sh

# Full build and test
./gradlew clean build test

# Verify your config
cat workbench/config.local.env
```

#### **4. AWS Deployment**

```bash
# Create S3 bucket (one-time setup)
./workbench/1-create-bucket.sh

# Deploy to AWS Lambda
./workbench/2-deploy.sh

# Test deployed function
./workbench/3-invoke.sh
```

#### **5. Iteration & Updates**

```bash
# Make code changes in src/main/bx/Lambda.bx
# Run tests
./gradlew test

# Redeploy
./gradlew build && ./workbench/2-deploy.sh

# Test updated function
./workbench/3-invoke.sh
```

#### **6. Cleanup (when done)**

```bash
# Remove all AWS resources
./workbench/4-cleanup.sh
```

### ⚡ **Quick Commands Reference**

#### **Daily Development**

```bash
./gradlew runLocal                   # Test locally
./gradlew test                      # Run tests
./gradlew build                     # Build package
```

#### **AWS Operations**

```bash
./workbench/0-check-aws.sh          # Check AWS setup
./workbench/1-create-bucket.sh      # Create S3 bucket
./workbench/2-deploy.sh             # Deploy to AWS
./workbench/3-invoke.sh             # Test deployed function
./workbench/4-cleanup.sh            # Remove AWS resources
```

#### **Local Testing Options**

```bash
./gradlew runLocal                  # Default event
./gradlew runLocalApi              # API Gateway event
./gradlew runLocalLegacy           # Legacy API event
./gradlew startSamServerBackground # HTTP server
./gradlew stopSamServer           # Stop server
```

## ✅ Development Tips

### 💡 Best Practices

- 🧪 **Test locally before deploying** - Use `./gradlew test` to catch issues early
- 🚀 **Use the wrapper** - Always use `./gradlew` (not `gradle`) for consistency
- 📦 **Check your ZIP** - Verify `build/distributions/*.zip` contains expected files
- 🔧 **Configure for production** - Set `trustedCache: true` in `boxlang.json`

### 🎨 Code Formatting Standards

**Spacing around symbols**: Always add spaces around parentheses `( )`, brackets `[ ]`, braces `{ }`, and operators for readability

**Examples:**

- ✅ `function run( event, context, response )`
- ❌ `function run(event,context,response)`
- ✅ `var results = [ 1, 2, 3 ]`
- ❌ `var results = [1,2,3]`
- ✅ `if ( condition ) { doSomething(); }`
- ❌ `if(condition){doSomething();}`

Apply this spacing standard to all BoxLang, Java, and configuration code in the project.

### 🐛 Troubleshooting

#### 🔧 **Build & Test Issues**

| Problem | Solution |
|---------|----------|
| ❌ Tests fail with ClassNotFoundException | Check dependency resolution, run `./gradlew clean build` |
| ❌ Gradle build fails | Ensure Java 21+ installed, run `./gradlew --version` |
| ❌ BoxLang class not found | Check module paths in `src/resources/boxlang_modules/` |
| ❌ Large deployment package | Review dependencies in `build.gradle`, exclude unnecessary JARs |

#### ☁️ **AWS Deployment Issues**

| Problem | Solution |
|---------|----------|
| ❌ AWS credentials not configured | Run `./workbench/0-check-aws.sh` for diagnosis |
| ❌ S3 bucket already exists | Choose a globally unique bucket name in `config.local.env` |
| ❌ CloudFormation deployment fails | Check AWS permissions, review stack events in AWS Console |
| ❌ Lambda function not found | Verify `FUNCTION_NAME` in config matches deployed function |

#### 🚀 **Lambda Runtime Issues**

| Problem | Solution |
|---------|----------|
| ❌ Lambda timeout in AWS | Increase `LAMBDA_TIMEOUT` in config and redeploy |
| ❌ Lambda memory errors | Increase `LAMBDA_MEMORY` in config and redeploy |
| ❌ BoxLang initialization errors | Check `boxlang.json` configuration, enable `debugMode: true` |
| ❌ Module loading failures | Verify modules in `src/resources/boxlang_modules/` directory |

#### 🛠️ **Quick Diagnosis Commands**

```bash
# Check AWS configuration
./workbench/0-check-aws.sh

# Verify project build
./gradlew clean build test

# Test locally before deployment
./gradlew runLocal

# Check deployed function
aws lambda list-functions --query 'Functions[?contains(FunctionName, `boxlang`)].FunctionName'
```

> 📚 **Additional Help**: See `workbench/DEPLOYMENT.md` for detailed deployment troubleshooting

## Ortus Sponsors

BoxLang is a professional open-source project and it is completely funded by the [community](https://patreon.com/ortussolutions) and [Ortus Solutions, Corp](https://www.ortussolutions.com).  Ortus Patreons get many benefits like a cfcasts account, a FORGEBOX Pro account and so much more.  If you are interested in becoming a sponsor, please visit our patronage page: [https://patreon.com/ortussolutions](https://patreon.com/ortussolutions)

### THE DAILY BREAD

 > "I am the way, and the truth, and the life; no one comes to the Father, but by me (JESUS)" Jn 14:1-12
