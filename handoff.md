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

WSO2 Integrator 5.0.0 is installed at:

```text
/Applications/WSO2 Integrator.app
```

The installer checksum was verified against the checksum published with the
official WSO2 GitHub release. The application also passed macOS notarization
verification.

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
