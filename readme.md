# ⚡︎ BoxLang AWS Lambda Starter Template

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

## 🚀 Welcome

This is the starter template for building **BoxLang serverless applications on AWS Lambda**. It bootstraps everything you need: the [BoxLang AWS Lambda runtime](https://github.com/ortus-boxlang/boxlang-aws-lambda), the convention-based `handlers/` routing setup, a Gradle build with SAM CLI integration, and a ready-to-run test suite.

> 💡 This template is intentionally structured the same way as our [Google Cloud Functions](https://github.com/ortus-boxlang/boxlang-starter-google-functions) and [Azure Functions](https://github.com/ortus-boxlang/boxlang-starter-azure-functions) starter templates. Your `.bx` handler code can move between all three providers unmodified - only the deployment step differs.

## 📋 Prerequisites

- **Java 21+**
- **AWS CLI**, configured (`aws configure`) - for deployment
- **[SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)** - for local HTTP testing (optional)

## 🏗️ Project Structure

```
.
├── build.gradle                    # Gradle build, generateManifest task, shadowJar/buildLambdaZip
├── src/
│   ├── main/
│   │   └── bx/
│   │       ├── Application.bx      # Application lifecycle hooks
│   │       ├── Lambda.bx           # Default handler (fallback for unmatched routes)
│   │       └── handlers/
│   │           ├── Products.bx     # Routed handler -> /products
│   │           └── api/
│   │               └── Test.bx     # Nested routed handler -> /api/test
│   ├── resources/
│   │   ├── boxlang.json            # BoxLang runtime configuration
│   │   └── boxlang_modules/        # Local BoxLang modules (auto-packaged)
│   └── test/
│       └── java/com/myproject/     # JUnit integration tests + mocks
└── workbench/
    ├── config.env                  # Default deployment config (copy to config.local.env)
    ├── template.yml                # SAM template used by 2-deploy.sh
    ├── sampleEvents/                # Sample Lambda event payloads for local testing
    └── *.sh                        # Deployment scripts (see AWS Deployment below)
```

## 🧭 URI Routing with `handlers/`

Only files under `src/main/bx/handlers/` (or listed in the build-time-generated `manifest.json`) are ever reachable by URI. `Application.bx` and the default `Lambda.bx` are never routable, no matter what's on disk.

| Incoming URI | Handler File |
|---|---|
| `/products` | `handlers/Products.bx` |
| `/api/test` | `handlers/api/Test.bx` |
| `/user-profiles` | `handlers/UserProfiles.bx` (hyphens map to PascalCase) |
| `/` or anything unmatched | `Lambda.bx` (the default handler) |

Add a new route by creating a `.bx` file under `handlers/`:

```boxlang
// src/main/bx/handlers/Orders.bx
class {
    function run( event, context, response ) {
        return { "message": "Fetching orders" };
    }
}
```

`./gradlew generateManifest` scans `handlers/` and writes `src/main/bx/manifest.json` - it's wired via `dependsOn` into `test`, `runLocal`, and `buildLambdaZip`, so it's always regenerated fresh and can never silently drift. `manifest.json` is gitignored, never hand-edited or committed.

If `manifest.json` is ever missing or invalid, the runtime falls back to scanning `handlers/` directly, and if that directory doesn't exist either, to scanning the project root for backward compatibility with pre-`handlers/` deployments; set `BOXLANG_ENABLE_ROOT_SCAN=false` to disable that last-resort scan entirely and restrict routing to the default `Lambda.bx` handler only.

`manifest.json`'s `reserved` and `defaultHandler` fields are enforced by the runtime, not just documentation - a manifest can never route to a reserved file (`Application.bx`, `Lambda.bx`, or anything else it lists), and `defaultHandler.file`/`method` is honored as the fallback handler for unmatched routes when present.

As with `Lambda.bx`, the `x-bx-function` header can call an alternative method on a handler - only ever a method you declared, since BoxLang's public/remote scope rules are exactly what gates it: don't make a method public if you don't want it externally callable.

## 🔧 Application Lifecycle

Use `src/main/bx/Application.bx` for initialization and per-request hooks:

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

## 🛠️ Local Development

```bash
# Run the tests
./gradlew test

# Test locally without deploying (default event)
./gradlew runLocal

# Test with an API Gateway event
./gradlew runLocalApi
```

For HTTP endpoint testing with [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) installed:

```bash
./gradlew startSamServerBackground   # start a local API server at :3000
curl http://localhost:3000
curl -H "x-bx-function: anotherLambda" http://localhost:3000
./gradlew stopSamServer              # stop it when done
```

Sample event payloads live in `workbench/sampleEvents/` (`api.json`, `api-post.json`, `s3-event.json`, etc.) - pass one with `-PeventFile=workbench/sampleEvents/s3-event.json`.

Set `BOXLANG_LAMBDA_DEBUGMODE=true` to enable verbose logging and disable class caching, so `.bx` changes are picked up immediately.

## ☁️ AWS Deployment

The `workbench/` directory automates the full deploy cycle via CloudFormation:

| Script | Purpose |
|---|---|
| `0-check-aws.sh` | Diagnose AWS credential/config issues |
| `1-create-bucket.sh` | Create the S3 bucket for deployment artifacts |
| `2-deploy.sh` | Build and deploy via SAM/CloudFormation |
| `3-invoke.sh` | Invoke the deployed Lambda with a test payload |
| `4-cleanup.sh` | Tear down all deployed AWS resources |

```bash
# One-time setup
cp workbench/config.env workbench/config.local.env
# edit config.local.env: AWS_LAMBDA_BUCKET, STACK_NAME, AWS_REGION, etc.

./workbench/1-create-bucket.sh
./workbench/2-deploy.sh
./workbench/3-invoke.sh
```

`config.local.env` is gitignored and layers over `config.env` → environment variables. Key settings: `AWS_LAMBDA_BUCKET` (required, globally unique), `STACK_NAME`, `LAMBDA_MEMORY`, `LAMBDA_TIMEOUT`, `AWS_REGION`.

The template also ships GitHub Actions workflows (`.github/workflows/`) for test, snapshot, and release builds - the AWS deployment step is commented out by default; uncomment it once your function exists and your `AWS_*` secrets are configured.

## 🧪 Testing

```bash
./gradlew test
```

Tests in `src/test/java/com/myproject/` exercise the full request pipeline using `LambdaRunner` directly with mock AWS Context objects - no live AWS environment required, so they run fast in CI. Test report: `build/reports/tests/test/index.html`.

## ⚙️ Configuration

### `boxlang.json`

`src/resources/boxlang.json` controls BoxLang runtime behavior: class-generation caching (`trustedCache`), debug mode, logging, request timeouts, and more. Set `trustedCache: false` and `debugMode: true` for local development; flip both for production.

### Environment Variables

| Variable | Description | Default |
|---|---|---|
| `BOXLANG_LAMBDA_CLASS` | Absolute path to the default handler | `/var/task/Lambda.bx` |
| `BOXLANG_LAMBDA_DEBUGMODE` | Verbose logging, disables class caching | `false` |
| `BOXLANG_LAMBDA_CONFIG` | Path to a custom `boxlang.json` | `/var/task/boxlang.json` |
| `BOXLANG_LAMBDA_CONNECTION_POOL_SIZE` | Database connection pool size | `2` |
| `BOXLANG_ENABLE_ROOT_SCAN` | Allow the legacy root-directory scan fallback | `true` |
| `LAMBDA_TASK_ROOT` | Lambda deployment root directory | `/var/task` |

## 📦 Adding BoxLang Modules

```bash
box install {moduleName} --production --directory=src/resources/boxlang_modules
```

Or declare them in `box.json` under `dependencies`/`installPaths` and run `box install --production`. Modules are automatically packaged into your deployment ZIP under `boxlang_modules/`.

## 📚 Additional Resources

- **BoxLang AWS Lambda Runtime** - [boxlang-aws-lambda](https://github.com/ortus-boxlang/boxlang-aws-lambda)
- **BoxLang Documentation** - [boxlang.ortusbooks.com](https://boxlang.ortusbooks.com)
- **Google Cloud Functions Starter** - [boxlang-starter-google-functions](https://github.com/ortus-boxlang/boxlang-starter-google-functions)
- **Azure Functions Starter** - [boxlang-starter-azure-functions](https://github.com/ortus-boxlang/boxlang-starter-azure-functions)

## 🐛 Troubleshooting

| Problem | Solution |
|---|---|
| Tests fail with `ClassNotFoundException` | `./gradlew clean build` to refresh dependency resolution |
| AWS credentials not configured | `./workbench/0-check-aws.sh` for diagnosis |
| S3 bucket already exists | Choose a globally unique name in `config.local.env` |
| Lambda timeout/memory errors in AWS | Increase `LAMBDA_TIMEOUT`/`LAMBDA_MEMORY` in `config.local.env` and redeploy |
| Routing looks off after a deploy | Check your Lambda logs for a manifest `WARNING`; confirm `generateManifest` ran |

## License

Apache License, Version 2.0.

## Open-Source & Professional Support

This project is a professional open source project and is available as FREE and open source to use.  Ortus Solutions, Corp provides commercial support, training and commercial subscriptions which include the following:

- Professional Support and Priority Queuing
- Remote Assistance and Troubleshooting
- New Feature Requests and Custom Development
- Custom SLAs
- Application Modernization and Migration Services
- Performance Audits
- Enterprise Modules and Integrations
- Much More

https://www.boxlang.io/plans

<p>&nbsp;</p>

<blockquote>
"We ❤️ Open Source and BoxLang" - Luis Majano
</blockquote>

### THE DAILY BREAD

> "I am the way, and the truth, and the life; no one comes to the Father, but by me (JESUS)" Jn 14:1-12
