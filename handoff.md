# WSO2 AI Integration Handoff

Last updated: 2026-07-26

## Objective

Use WSO2 Integrator 5 primarily to create and run AI agents, RAG pipelines,
MCP services, and related AI integrations. Keep the initial footprint minimal
and avoid installing unrelated WSO2 products.

## Infrastructure

- Server: `212.108.82.158`
- SSH user: `root`
- Hostname: `nl-vmv3-heavy`
- OS: Ubuntu 24.04, x86_64
- CPU: 8 vCPUs
- RAM: 31 GiB total, approximately 26 GiB available during inspection
- Swap: not configured
- Root disk: 236 GiB total, 92 GiB available during inspection
- Docker Engine: 29.4.3
- Docker Compose: 5.1.3

SSH access uses an existing key. No credentials or private keys are stored in
this repository.

## Existing Services

The existing `mass-ai` Docker stack remains unchanged:

- Nginx
- Reranker
- Surya OCR
- Model adapter
- Ollama
- PostgreSQL 16
- Qdrant

PostgreSQL is used by other services and must not be replaced, removed, or
reconfigured as part of the WSO2 setup.

## Installed Components

### Developer workstation

WSO2 Integrator 5.0.0.1 (Intel/x64 build) is installed at:

```text
/Applications/WSO2 Integrator.app
```

The installer checksum was verified against the checksum published with the
official WSO2 GitHub release. The application also passed macOS notarization
verification.

#### macOS repair kit

The stock WSO2 Integrator 5.0.0.1 installation repeatedly left the HTTP
Service visualizer loading. This was not an authentication or general network
failure. The bundled Ballerina visualizer had desktop compatibility and
packaging errors, including:

- invocation of the unavailable `getContext` command;
- invocation of the unavailable `BI.project-explorer.notify` command;
- an incorrect local path for the WSO2 font stylesheet; and
- a reference to an unbundled legacy theme stylesheet.

The tested one-click repair package is stored at:

```text
WSO2-Integrator-macOS-Repair-Kit.zip
```

SHA-256:

```text
88f5b6fc3d9a1fab63a8a8ae67f092e7cb422596dbe5df3da41b898e1eef17f2
```

On another Mac:

1. Install WSO2 Integrator 5.0.0.1 in `/Applications`.
2. Choose the Apple Silicon DMG for an M-series Mac or the x64 DMG for an
   Intel Mac.
3. Start WSO2 Integrator once, then quit it.
4. Extract the repair ZIP and double-click
   `WSO2-Integrator-macOS-Repair.command`.
5. If Gatekeeper blocks it, Control-click the command, select **Open**, and
   confirm **Open**.
6. Wait for the success message, reopen the project, and select its HTTP
   Service.

The repair detects the Mac architecture, downloads the official WSO2 Ballerina
extension 5.12.3 from Visual Studio Marketplace, applies and validates the
compatibility fixes, creates a rollback backup, clears only disposable WSO2
webview caches, disables automatic extension updates so the fix is retained,
and restarts WSO2 Integrator. It does not delete or modify integration
projects.

The package was tested end to end on the developer workstation. The HTTP
Service visualizer rendered its listener, base path, resources, and **Add
Resource** controls with no loader remaining. A subsequent normal launch did
not reproduce the command, stylesheet, local-resource, or internal extension
errors.

#### Intel macOS project TLS repair

The Intel/x64 distribution contains a malformed native Netty classifier at:

```text
Contents/components/ballerina/repo/bala/ballerina/http/2.16.3/java21/
platform/java21/
netty-tcnative-boringssl-static-2.0.77.Final-osx-x86_64.jar
```

The installed JAR is only 28,458 bytes and its `META-INF/native` directory is
empty. A project can therefore compile successfully but fail when the WSO2
model provider creates its HTTPS client:

```text
java.lang.UnsatisfiedLinkError:
Failed to load any of the given libraries:
netty_tcnative_osx_x86_64
```

This is an environment packaging defect, not a Ballerina source-code error.
The corresponding Apple Silicon classifier contains its native binary and was
not affected during inspection.

The repair ZIP now includes:

```text
WSO2-Ballerina-Project-TLS-Repair.command
assets/netty-tcnative-boringssl-static-2.0.77.Final-osx-x86_64.jar
```

The included binary is the official Netty 2.0.77.Final macOS Intel artifact
from Maven Central. Its published SHA-1 checksum is:

```text
3b3dfbf6136c0b278205f042f449ef8f20813ce6
```

Run the project repair once for each affected Intel project and select the
folder containing `Ballerina.toml`. The tool:

- verifies the bundled native artifact;
- saves `Ballerina.toml.before-wso2-tls-repair`;
- copies the artifact into the project's `libs` directory;
- adds it as a Java 21 platform dependency;
- builds the project; and
- confirms that the generated executable contains
  `libnetty_tcnative_osx_x86_64.jnilib`.

It does not change `.bal` source files or modify the signed WSO2 application
bundle. On Apple Silicon it exits without changing the project.

The repair was validated against:

```text
/Users/mac/WSO2Integrator/prj-ai-integrator-training/int_sentimentanalyzer
```

Before repair, this project failed during WSO2 model-provider initialization.
After repair, it compiled without warnings, started successfully, and listened
on port `9090`. The verification process was then stopped so the IDE can start
the service normally without a port conflict.

### Server

No permanent WSO2 runtime, Integration Control Plane, Kubernetes cluster, or
additional database has been installed. The existing Docker installation is
the runtime prerequisite.

The following deployment directories have been prepared with mode `0750`:

```text
/opt/wso2-integrator/
├── apps/
├── config/
└── secrets/
```

## Deployment Model

Create and test AI integrations in the WSO2 Integrator IDE. Build each agent,
RAG pipeline, or related group of integrations as an independent Docker image,
then deploy that image to the server with Docker Compose.

Use one directory under `/opt/wso2-integrator/apps/` per deployable application.
Keep non-secret configuration under `/opt/wso2-integrator/config/` and runtime
secrets under `/opt/wso2-integrator/secrets/`.

Do not commit model-provider keys, database passwords, access tokens, or
generated secret files. Pin container image versions and apply explicit CPU and
memory limits.

## Next Steps

1. In WSO2 Integrator, create an integration using the **AI Chat Agent**
   artifact.
2. Configure the required model provider, tools, memory, and API listener.
3. Add a RAG flow only when a concrete knowledge source and embedding model
   have been selected.
4. Generate and validate the Docker image locally.
5. Create an application-specific Compose file on the server.
6. Connect the container to existing PostgreSQL, Qdrant, or Ollama services
   only when the integration requires them.
7. Expose only the agent API through the existing reverse proxy; keep
   management and internal ports private.

## Operational Notes

- The host has no swap. Enforce container memory limits to reduce the risk of
  one AI or JVM workload exhausting host memory.
- Qdrant and the reranker are the largest existing memory consumers observed
  during inspection.
- Back up persistent application data before upgrades.
- Do not use the `latest` tag for production images.
