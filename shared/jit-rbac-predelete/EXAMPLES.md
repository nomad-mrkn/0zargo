# JIT RBAC PreDelete Examples

## Примеры интеграции для различных типов приложений

### 1. cert-manager

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: k8s-spb2-backend1-certmanager
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  syncPolicy:
    automated:
      prune: true
    syncOptions:
    - CreateNamespace=true
    - SkipDryRunOnMissingResource=true
    - RespectIgnoreDifferences=true
  project: default
  destination:
    name: kind-test-1
    namespace: cert-manager
  sources:
    # JIT RBAC for deployment
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-roles
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-certmanager
        patches:
          - target:
              kind: ClusterRole
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-certmanager
          - target:
              kind: ClusterRoleBinding
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-certmanager
              - op: replace
                path: /roleRef/name
                value: argocd-manager-certmanager
    
    # PreDelete hook for safe deletion
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-predelete
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-certmanager
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-create-deletion-rbac
            patch: |-
              - op: replace
                path: /metadata/name
                value: k8s-spb2-backend1-certmanager-create-deletion-rbac
              - op: replace
                path: /spec/template/spec/containers/0/env/1/value
                value: argocd-manager-certmanager-deletion
              - op: replace
                path: /spec/template/spec/containers/0/env/2/value
                value: argocd-manager-certmanager-deletion
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: jit-rbac-predelete-k8s-spb2-backend1-certmanager
    
    # Mark completed job
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-certmanager
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-mark-completed
            patch: |-
              - op: replace
                path: /metadata/name
                value: k8s-spb2-backend1-certmanager-mark-completed
    
    # Helm chart
    - repoURL: 'https://github.com/nomad-mrkn/charts.git'
      targetRevision: HEAD
      path: cert-manager
      helm:
        releaseName: k8s-spb2-backend2-certmanager
        values: |
          installCRDs: true
          startupapicheck:
           enabled: false
```

### 2. fluent-bit

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: k8s-spb2-backend1-fluent-bit
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "3"
spec:
  syncPolicy:
    automated:
      prune: true
    syncOptions:
    - CreateNamespace=true
  project: default
  destination:
    name: kind-test-1
    namespace: logging
  sources:
    # JIT RBAC for deployment
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-roles
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-fluent-bit
        patches:
          - target:
              kind: ClusterRole
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-fluent-bit
          - target:
              kind: ClusterRoleBinding
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-fluent-bit
              - op: replace
                path: /roleRef/name
                value: argocd-manager-fluent-bit
    
    # PreDelete hook
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-predelete
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-fluent-bit
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-create-deletion-rbac
            patch: |-
              - op: replace
                path: /metadata/name
                value: k8s-spb2-backend1-fluent-bit-create-deletion-rbac
              - op: replace
                path: /spec/template/spec/containers/0/env/1/value
                value: argocd-manager-fluent-bit-deletion
              - op: replace
                path: /spec/template/spec/containers/0/env/2/value
                value: argocd-manager-fluent-bit-deletion
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: jit-rbac-predelete-k8s-spb2-backend1-fluent-bit
          - target:
              kind: ServiceAccount
              name: jit-rbac-predelete
            patch: |-
              - op: replace
                path: /metadata/namespace
                value: logging
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /subjects/0/namespace
                value: logging
    
    # Mark completed
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-fluent-bit
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-mark-completed
            patch: |-
              - op: replace
                path: /metadata/name
                value: k8s-spb2-backend1-fluent-bit-mark-completed
          - target:
              kind: ServiceAccount
              name: jit-rbac-marker
            patch: |-
              - op: replace
                path: /metadata/namespace
                value: logging
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-marker-PLACEHOLDER
            patch: |-
              - op: replace
                path: /subjects/0/namespace
                value: logging
    
    # Helm chart
    - repoURL: 'https://github.com/nomad-mrkn/charts.git'
      targetRevision: HEAD
      path: fluent-bit
      helm:
        releaseName: fluent-bit
        values: |
          # fluent-bit values
```

### 3. ingress-nginx

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: k8s-spb2-backend1-ingress-nginx
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  syncPolicy:
    automated:
      prune: true
    syncOptions:
    - CreateNamespace=true
  project: default
  destination:
    name: kind-test-1
    namespace: ingress-nginx
  sources:
    # JIT RBAC
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-roles
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-ingress-nginx
        patches:
          - target:
              kind: ClusterRole
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-ingress-nginx
          - target:
              kind: ClusterRoleBinding
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-ingress-nginx
              - op: replace
                path: /roleRef/name
                value: argocd-manager-ingress-nginx
    
    # PreDelete hook
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-predelete
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-ingress-nginx
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-create-deletion-rbac
            patch: |-
              - op: replace
                path: /metadata/name
                value: k8s-spb2-backend1-ingress-nginx-create-deletion-rbac
              - op: replace
                path: /spec/template/spec/containers/0/env/1/value
                value: argocd-manager-ingress-nginx-deletion
              - op: replace
                path: /spec/template/spec/containers/0/env/2/value
                value: argocd-manager-ingress-nginx-deletion
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: jit-rbac-predelete-k8s-spb2-backend1-ingress-nginx
          - target:
              kind: ServiceAccount
              name: jit-rbac-predelete
            patch: |-
              - op: replace
                path: /metadata/namespace
                value: ingress-nginx
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /subjects/0/namespace
                value: ingress-nginx
    
    # Mark completed
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac
      kustomize:
        commonLabels:
          jit-rbac/app: k8s-spb2-backend1-ingress-nginx
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-mark-completed
            patch: |-
              - op: replace
                path: /metadata/name
                value: k8s-spb2-backend1-ingress-nginx-mark-completed
          - target:
              kind: ServiceAccount
              name: jit-rbac-marker
            patch: |-
              - op: replace
                path: /metadata/namespace
                value: ingress-nginx
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-marker-PLACEHOLDER
            patch: |-
              - op: replace
                path: /subjects/0/namespace
                value: ingress-nginx
    
    # Helm chart
    - repoURL: 'https://github.com/nomad-mrkn/charts.git'
      targetRevision: HEAD
      path: ingress-nginx
      helm:
        releaseName: ingress-nginx
```

## Шаблон для новых приложений

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <cluster>-<app-name>
  namespace: argocd
  annotations:
    argocd.argoproj.io/sync-wave: "<wave-number>"
spec:
  syncPolicy:
    automated:
      prune: true
    syncOptions:
    - CreateNamespace=true
  project: default
  destination:
    name: <cluster-name>
    namespace: <app-namespace>
  sources:
    # Source 1: JIT RBAC for deployment
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-roles
      kustomize:
        commonLabels:
          jit-rbac/app: <cluster>-<app-name>
        patches:
          - target:
              kind: ClusterRole
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-<app-name>
          - target:
              kind: ClusterRoleBinding
              name: PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: argocd-manager-<app-name>
              - op: replace
                path: /roleRef/name
                value: argocd-manager-<app-name>
    
    # Source 2: PreDelete hook for safe deletion
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac-predelete
      kustomize:
        commonLabels:
          jit-rbac/app: <cluster>-<app-name>
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-create-deletion-rbac
            patch: |-
              - op: replace
                path: /metadata/name
                value: <cluster>-<app-name>-create-deletion-rbac
              - op: replace
                path: /spec/template/spec/containers/0/env/1/value
                value: argocd-manager-<app-name>-deletion
              - op: replace
                path: /spec/template/spec/containers/0/env/2/value
                value: argocd-manager-<app-name>-deletion
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /metadata/name
                value: jit-rbac-predelete-<cluster>-<app-name>
          # If app namespace is NOT cert-manager, patch namespace
          - target:
              kind: ServiceAccount
              name: jit-rbac-predelete
            patch: |-
              - op: replace
                path: /metadata/namespace
                value: <app-namespace>
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-predelete-PLACEHOLDER
            patch: |-
              - op: replace
                path: /subjects/0/namespace
                value: <app-namespace>
    
    # Source 3: Mark completed job
    - repoURL: 'https://github.com/nomad-mrkn/0zargo.git'
      targetRevision: HEAD
      path: shared/jit-rbac
      kustomize:
        commonLabels:
          jit-rbac/app: <cluster>-<app-name>
        patches:
          - target:
              kind: Job
              name: PLACEHOLDER-mark-completed
            patch: |-
              - op: replace
                path: /metadata/name
                value: <cluster>-<app-name>-mark-completed
          # If app namespace is NOT cert-manager, patch namespace
          - target:
              kind: ServiceAccount
              name: jit-rbac-marker
            patch: |-
              - op: replace
                path: /metadata/namespace
                value: <app-namespace>
          - target:
              kind: ClusterRoleBinding
              name: jit-rbac-marker-PLACEHOLDER
            patch: |-
              - op: replace
                path: /subjects/0/namespace
                value: <app-namespace>
    
    # Source 4: Application resources (Helm/Kustomize/Plain)
    - repoURL: '<your-repo>'
      targetRevision: HEAD
      path: <path-to-app>
      # Choose one:
      # helm:
      #   releaseName: <release-name>
      #   values: |
      #     # values here
      # OR
      # kustomize:
      #   # kustomize options
      # OR
      # directory:
      #   recurse: true
```

## Команды для тестирования

### Развертывание

```bash
# Apply application
kubectl apply -f <app>.yaml

# Watch sync status
argocd app wait <app-name> --sync

# Check JIT RBAC
kubectl get clusterrole,clusterrolebinding -l jit-rbac/app=<app-name>

# Check sync status
kubectl get clusterrole -l jit-rbac/app=<app-name> -o jsonpath='{.items[0].metadata.labels.jit-rbac/sync-status}'
```

### Удаление

```bash
# Delete application
kubectl delete application <app-name> -n argocd

# Watch PreDelete job
kubectl get job -n <namespace> -w

# Check deletion RBAC created
kubectl get clusterrole,clusterrolebinding -l jit-rbac/phase=deletion

# Check PreDelete job logs
kubectl logs -n <namespace> job/<app-name>-create-deletion-rbac

# After deletion, cleanup
./shared/jit-rbac-predelete/cleanup-deletion-rbac.sh argocd-manager-<app-name>
```

## Частые ошибки и решения

### Ошибка 1: PreDelete Job не запускается

**Проблема**: Job не создается при удалении Application

**Решение**:
```bash
# Проверьте наличие PreDelete source
kubectl get application <app-name> -n argocd -o yaml | grep -A 10 "jit-rbac-predelete"

# Если отсутствует, добавьте source
```

### Ошибка 2: Неправильный namespace для ServiceAccount

**Проблема**: ServiceAccount создается в cert-manager, а приложение в другом namespace

**Решение**: Добавьте патчи для namespace:
```yaml
patches:
  - target:
      kind: ServiceAccount
      name: jit-rbac-predelete
    patch: |-
      - op: replace
        path: /metadata/namespace
        value: <your-namespace>
  - target:
      kind: ClusterRoleBinding
      name: jit-rbac-predelete-PLACEHOLDER
    patch: |-
      - op: replace
        path: /subjects/0/namespace
        value: <your-namespace>
```

### Ошибка 3: Deletion RBAC не удаляется автоматически

**Проблема**: После удаления приложения deletion RBAC остается

**Решение**: Это ожидаемое поведение. Удалите вручную:
```bash
./shared/jit-rbac-predelete/cleanup-deletion-rbac.sh argocd-manager-<app-name>
```

