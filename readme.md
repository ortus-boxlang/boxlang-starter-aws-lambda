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

## 📦 What This Starter Includes

- The BoxLang AWS Lambda runtime, pre-wired as your Lambda handler
- Convention-based `handlers/` routing, backed by a build-time `manifest.json`
- A Gradle build with a shaded JAR, `buildLambdaZip` packaging, and SAM CLI integration for local HTTP testing
- JUnit integration tests that exercise the full request pipeline with mock AWS Context objects
- Ready-to-use GitHub Actions workflows for test, snapshot, and release builds
- Deployment automation via `workbench/*.sh` and a parameterized SAM template

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
├── workbench/
│   ├── config.env                  # Default deployment config (copy to config.local.env)
│   ├── template.yml                # SAM template used by 2-deploy.sh
│   ├── sampleEvents/               # Sample Lambda event payloads for local testing
│   └── *.sh                        # Deployment scripts (see AWS Deployment below)
├── .github/workflows/              # Test, snapshot, and release CI/CD pipelines
├── box.json                        # BoxLang module dependencies
└── gradle.properties               # version, jdkVersion, boxlangVersion
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

`./gradlew generateManifest` scans `handlers/` and writes `src/main/bx/manifest.json` - it's wired via `dependsOn` into `test`, `runLocal`, `runLocalApi`, `runLocalLegacy`, and `buildLambdaZip`, so it's always regenerated fresh and can never silently drift. `manifest.json` is gitignored, never hand-edited or committed.

If `manifest.json` is ever missing or invalid, the runtime falls back to scanning `handlers/` directly, and if that directory doesn't exist either, to scanning the project root for backward compatibility with pre-`handlers/` deployments; set `BOXLANG_ENABLE_ROOT_SCAN=false` to disable that last-resort scan entirely and restrict routing to the default `Lambda.bx` handler only.

`manifest.json`'s `reserved` and `defaultHandler` fields are enforced by the runtime, not just documentation - a manifest can never route to a reserved file (`Application.bx`, `Lambda.bx`, or anything else it lists), and `defaultHandler.file`/`method` is honored as the fallback handler for unmatched routes when present.

As with `Lambda.bx`, the `x-bx-function` header can call an alternative method on a handler - only ever a method you declared, since BoxLang's public/remote scope rules are exactly what gates it: don't make a method public if you don't want it externally callable.

## 🔧 Application Lifecycle

Use `src/main/bx/Application.bx` for initialization and per-request hooks. It fires for every request, whether served by `Lambda.bx` or by a routed handler under `handlers/`:

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

`run()` and every request lifecycle hook (`onRequestStart`, `onRequestEnd`, `onError`, `onAbort`) receive the same `response` struct as their last argument. A returned value is stored in `response.body` before `onRequestEnd` runs, so a hook can wrap it, and a handled error defaults to status `500` unless `onError` sets one:

```js
class {

    function onRequestEnd( target, event, context, response ) {
        response.body = { ok: true, data: response.body }
    }

    function onError( exception, eventName, event, context, response ) {
        response.body = { ok: false, error: exception.message }
    }

}
```

Set `BOXLANG_RESPONSE_MODE=raw` to return only `response.body`, unwrapped, instead of the default `statusCode`/`headers`/`body`/`cookies` envelope. Use it for direct invocation or an API Gateway REST API without a proxy integration. If `Application.bx` defines `onError`, the error counts as handled; rethrow from the hook to fail the invocation.

## 📋 Handler Contract

Every handler - `Lambda.bx` or anything under `handlers/` - implements `run( event, context, response )` (or an alternate method called via the `x-bx-function` header):

```boxlang
class{
    function run( event, context, response ){
        response.body = {
            "error": false,
            "messages": [],
            "data": "Incoming event: " & event.toString()
        }
        response.statusCode = 200
    }

    // Call with header: x-bx-function: anotherLambda
    function anotherLambda( event, context, response ){
        return "Hola!!"
    }
}
```

- **`event`** - the event struct/map that triggered the Lambda (API Gateway, Function URL, ALB, or direct invocation)
- **`context`** - the AWS Lambda context object (`com.amazonaws.services.lambda.runtime.Context`)
- **`response`** - the struct returned to the caller, with a standard shape: `statusCode` (default `200`), `headers`, `body`, `cookies` (array), plus any other property you add

You can either populate `response` or simply `return` a value - both are auto-serialized to JSON.

## 🛠️ Local Development

```bash
./gradlew test           # run the test suite
./gradlew runLocal        # test locally with the default event
./gradlew runLocalApi     # test locally with an API Gateway event
./gradlew runLocalLegacy  # test locally with a legacy API Gateway event
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

## 🧪 Testing

```bash
./gradlew test
```

Tests in `src/test/java/com/myproject/` exercise the full request pipeline using `LambdaRunner` directly with mock AWS Context objects - no live AWS environment required, so they run fast in CI. Test report: `build/reports/tests/test/index.html`.

```java
@Test
@DisplayName( "Test Lambda.bx execution" )
public void testValidLambda() throws IOException {
    Path validPath = Path.of( "src", "main", "bx", "Lambda.bx" );
    LambdaRunner runner = new LambdaRunner( validPath, true );
    Context context = new TestContext();

    var event = new HashMap<String, Object>();
    event.put( "name", "Ortus Solutions" );

    IStruct response = ( IStruct ) runner.handleRequest( event, context );

    assertThat( response.getAsInteger( Key.of( "statusCode" ) ) ).isEqualTo( 200 );
}
```

## 🔨 Build Tasks

| Task | Description |
|---|---|
| `build` | Full build lifecycle (clean, compile, test, package) |
| `test` | Run the JUnit test suite |
| `generateManifest` | Scan `handlers/` and (re)generate `manifest.json` |
| `shadowJar` | Create the uber-JAR with all dependencies |
| `buildLambdaZip` | Package the Lambda deployment ZIP (`build/distributions/*.zip`) |
| `runLocal` / `runLocalApi` / `runLocalLegacy` | Run the Lambda locally with a default / API Gateway / legacy API event |
| `startSamServer` / `startSamServerBackground` | Start a local SAM HTTP API server (foreground / background) |
| `stopSamServer` | Stop the background SAM server |
| `spotlessApply` / `spotlessCheck` | Auto-format / check Java source formatting |

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

`config.local.env` is gitignored and layers over `config.env` → environment variables.

| Variable | Default | Description |
|---|---|---|
| `AWS_LAMBDA_BUCKET` | *(required)* | S3 bucket for artifacts - must be globally unique |
| `STACK_NAME` | `boxlang-lambda-stack` | CloudFormation stack name |
| `FUNCTION_NAME` | `{STACK_NAME}-bxFunction-{SUFFIX}` | Direct function name for `3-invoke.sh` |
| `LAMBDA_MEMORY` | `128` | Memory allocation in MB (128-10240) |
| `LAMBDA_TIMEOUT` | `15` | Timeout in seconds (1-900) |
| `AWS_REGION` | *your default* | AWS region for deployment |
| `ENVIRONMENT` | `dev` | Deployment environment tag |

## 🤖 CI/CD Workflows

`.github/workflows/` ships three ready-to-use pipelines:

| Workflow | Trigger | Purpose |
|---|---|---|
| `tests.yml` | Called by the other workflows | Reusable Java 21 test run with report artifacts |
| `snapshot.yml` | Push to any non-`main` branch, PRs | Development builds with snapshot versioning |
| `release.yml` | Push to `main`, manual dispatch | Full build, test, and package; optional AWS deployment (commented out by default) |

To enable automatic AWS deployment on release: deploy your function once via the `workbench/` scripts, add `AWS_REGION`/`AWS_PUBLISHER_KEY_ID`/`AWS_SECRET_PUBLISHER_KEY` as GitHub Secrets, then uncomment the deployment step in `release.yml`.

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
| `BOXLANG_ENABLE_ROOT_SCAN` | Allow the legacy root-directory routing fallback (see URI Routing above) | `true`. Shared across every BoxLang serverless runtime (AWS/GCP/Azure). |
| `BOXLANG_RESPONSE_MODE` | What the Lambda returns: `http` returns the `statusCode`/`headers`/`body`/`cookies` envelope, `raw` returns only `response.body`, unwrapped. Any other value aborts cold start | `http` |
| `LAMBDA_TASK_ROOT` | Lambda deployment root directory | `/var/task` |

## 📦 Adding BoxLang Modules

```bash
box install {moduleName} --production --directory=src/resources/boxlang_modules
```

Or declare them in `box.json` under `dependencies`/`installPaths` and run `box install --production`. Modules are automatically packaged into your deployment ZIP under `boxlang_modules/`.

## 🐛 Troubleshooting

| Problem | Solution |
|---|---|
| Tests fail with `ClassNotFoundException` | `./gradlew clean build` to refresh dependency resolution |
| AWS credentials not configured | `./workbench/0-check-aws.sh` for diagnosis |
| S3 bucket already exists | Choose a globally unique name in `config.local.env` |
| Lambda timeout/memory errors in AWS | Increase `LAMBDA_TIMEOUT`/`LAMBDA_MEMORY` in `config.local.env` and redeploy |
| Routing looks off after a deploy | Check your Lambda logs for a manifest `WARNING`; confirm `generateManifest` ran |
| Large deployment package | Review dependencies in `build.gradle`; exclude unnecessary JARs |

## 📚 Additional Resources

- **BoxLang AWS Lambda Runtime** - [boxlang-aws-lambda](https://github.com/ortus-boxlang/boxlang-aws-lambda)
- **BoxLang Documentation** - [boxlang.ortusbooks.com](https://boxlang.ortusbooks.com)
- **Google Cloud Functions Starter** - [boxlang-starter-google-functions](https://github.com/ortus-boxlang/boxlang-starter-google-functions)
- **Azure Functions Starter** - [boxlang-starter-azure-functions](https://github.com/ortus-boxlang/boxlang-starter-azure-functions)

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
