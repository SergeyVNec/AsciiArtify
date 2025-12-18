# AsciiArtify GitOps PoC з Argo CD

## Мета PoC

Мета цього Proof of Concept (PoC) – показати, що команда **AsciiArtify** може:

- розгорнути локальний Kubernetes-кластер;
- встановити та сконфігурувати систему GitOps **Argo CD**;
- надати розробникам доступ до веб-інтерфейсу Argo CD;
- підготувати середовище для подальшого розгортання MVP додатку AsciiArtify.

PoC виконується локально на машині розробника і не претендує на production-рівень, але відтворює базову GitOps-архітектуру.

---

## Архітектура PoC

- **Kubernetes-кластер**: локальний кластер на базі `k3d` (K3s у Docker).
- **Git репозиторій**: `https://github.com/<username>/AsciiArtify`  
  (цей репозиторій, гілка `main`).
- **GitOps-система**: `Argo CD`, встановлена в namespace `argocd`.
- **Доступ до UI Argo CD**: через `kubectl port-forward` або `https://localhost:8080`.

---

## 1. Передумови

На робочій машині мають бути встановлені:

- **Docker** (або сумісний runtime)  
- **k3d**  
- **kubectl**
- (опціонально) **Argo CD CLI** `argocd`

Перевірка:

```bash
docker version
k3d version
kubectl version --client
