# hc-gitops-config — Repositorio de Configuración (GitOps Repo / "Repo B")

**Única Fuente de Verdad** del estado deseado del sistema, según la separación de
intereses (SoC) del estándar **IEEE 828-2012**. Este repositorio contiene únicamente
los manifiestos de Kubernetes (YAML); el código fuente, tests y Dockerfile viven en el
**App Repo** (`hc-backend`).

## Estructura
```
k8s/
  desarrollo/
    01-configmap.yaml   # configuración externalizada (sync-wave -1)
    02-database.yaml    # adaptador de salida BD (sync-wave 0, arranca primero)
    03-deployment.yaml  # servicio de dominio / adaptador REST (sync-wave 1)
    04-service.yaml     # exposición NodePort 30100 (sync-wave 1)
```

## Flujo GitOps
1. El pipeline de CI (`hc-backend`, GitHub Actions) construye la imagen Docker y la
   etiqueta con una versión **SemVer 2.0.0** (p. ej. `v1.1.0`), evitando `latest`.
2. El pipeline hace **commit automático** en este repo actualizando el `image:` en
   `03-deployment.yaml`.
3. **Argo CD** detecta el cambio en este repo (única fuente de verdad) y sincroniza el
   clúster (rolling update), sin intervención manual (pull, no push).

`prune: true` + `selfHeal: true` garantizan inmutabilidad y autorrecuperación: cualquier
cambio manual en el clúster (drift) se revierte hacia el estado declarado aquí.
