## Redis Service

Declarative Redis 7.2 service definition powered by the shared infrastructure roles. The defaults capture networking, storage, and runtime-agnostic settings so the service can be rendered for Proxmox LXC, Docker Compose, Podman Quadlet, Kubernetes, or bare-metal systemd using the same variables.

### Runtime Coverage
- Proxmox LXC managed through `common.render_runtime`/`common.apply_runtime`
- Docker Compose v2
- Podman Quadlet units
- Kubernetes Deployment + Service + Secret + PVC
- Bare-metal systemd service

### Exports
```
CACHE_HOST={{ service_ip }}
CACHE_PORT={{ redis_service_port }}
CACHE_PASSWORD={{ redis_password }}
```

### Secrets
- `REDIS_PASSWORD` -> forwarded to Bitnami Redis for the `requirepass` configuration. Defaults to `REDIS_PASSWORD` environment lookup with a placeholder fallback; override via inventory or vault.

### Health Check
Python socket probe authenticates (if a password is set) and expects `+PONG` from Redis. The command is reused across runtimes for Compose healthchecks, Quadlet probes, Kubernetes readiness/liveness, and the post-deploy gate.

### Key Overrides
| Variable | Default | Purpose |
| --- | --- | --- |
| `redis_service_port` | `6379` | Published TCP port |
| `redis_data_volume` | `redis-data` | Persistent data volume name |
| `redis_container_vmid` | `201` | Proxmox VMID |
| `redis_container_ip` | `192.168.100.11` | LXC container address |
| `redis_container_storage_gb` | `10` | Storage allocation for data |
| `redis_container_memory_mb` | `1024` | Memory allocation across runtimes |
| `redis_container_cpu_cores` | `1` | CPU allocation across runtimes |
| `redis_kubernetes_namespace` | `caches` | Namespace for the Deployment/Service/PVC |

Tune these in inventory or play vars to fit your environment. Additional runtime-specific knobs can be added by extending the defaults with the keys consumed by the shared templates.

### Usage
```yaml
- hosts: cache_hosts
  roles:
    - role: svc-redis
      vars:
        runtime: podman
        redis_service_port: 16379
        redis_password: "{{ vault_redis_password }}"
```
