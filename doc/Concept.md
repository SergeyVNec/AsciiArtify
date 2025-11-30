# AsciiArtify – Concept локального Kubernetes-середовища для PoC

## 1. Вступ

**AsciiArtify** — стартап, який розробляє сервіс перетворення зображень у ASCII-art за допомогою ML-моделей. Для PoC команда планує:

- використовувати **GitHub** як систему контролю версій (гілка `main`, pull-request-флоу);
- розгортати сервіс у **Kubernetes** навіть на етапі PoC, щоб одразу наблизити середовище розробки до майбутнього production.

Для локального Kubernetes-кластеру розглядаються три інструменти:

- **minikube** — “локальний Kubernetes в один клік” :contentReference[oaicite:0]{index=0}  
- **kind** (Kubernetes IN Docker) — локальні кластери, де вузли — це Docker/Podman-контейнери :contentReference[oaicite:1]{index=1}  
- **k3d** — тонка оболонка над **k3s** (легка дистрибуція Kubernetes), що запускається всередині Docker :contentReference[oaicite:2]{index=2}  

Також важливо врахувати:

- ліцензійні обмеження **Docker Desktop**;
- можливість використати **Podman** як альтернативу Docker.

---

## 2. Характеристики інструментів

### 2.1 minikube

- **Тип**: локальний одно- або багатовузловий Kubernetes-кластер.
- **ОС**: Linux, macOS, Windows. :contentReference[oaicite:3]{index=3}  
- **Архітектури**: `amd64`, `arm64` (включно з Apple Silicon). :contentReference[oaicite:4]{index=4}  
- **Драйвери**:
  - контейнерні: Docker, Podman (експериментально), containerd;
  - VM-драйвери: VirtualBox, KVM, Hyper-V тощо. :contentReference[oaicite:5]{index=5}  
- **Автоматизація**: CLI-команди легко вбудовуються у Makefile/GitHub Actions.
- **Додаткові можливості**:
  - `minikube addons` (Dashboard, Ingress, Metrics Server тощо);
  - вбудована підтримка локального registry;
  - профілі — кілька кластерів на одній машині.

### 2.2 kind

- **Тип**: Kubernetes-cluster “у контейнерах”; кожен вузол — це контейнер. :contentReference[oaicite:6]{index=6}  
- **ОС**: Linux, macOS, Windows (через Docker/Podman).
- **Архітектури**: `amd64`, `arm64` (за умови підтримки з боку контейнер-рушія).
- **Container-engine**:
  - Docker, Podman, nerdctl — автоматичне визначення, або вибір через `KIND_EXPERIMENTAL_PROVIDER`. :contentReference[oaicite:7]{index=7}  
- **Автоматизація**:
  - добре підходить для CI (GitHub Actions, GitLab CI);
  - конфіг кластеру описується YAML-файлом.
- **Додаткові можливості**:
  - кілька worker-нод;
  - вбудований локальний registry через extra-config;
  - інтеграції з Podman Desktop. :contentReference[oaicite:8]{index=8}  

### 2.3 k3d

- **Тип**: обгортка для запуску **k3s** (легкого Kubernetes) у Docker-контейнерах. :contentReference[oaicite:9]{index=9}  
- **ОС**: Linux, macOS, Windows (через Docker Desktop / інші Docker-аналогі).
- **Архітектури**: `amd64`, `arm64` (залежить від Docker-зображення).
- **Особливості k3s**:
  - мінімалістичний Kubernetes (менша споживаність ресурсів, один бінарник <512MB RAM) :contentReference[oaicite:10]{index=10}  
- **Автоматизація**:
  - `k3d cluster create` та YAML-конфіги для кластерів;
  - проста інтеграція в CI.
- **Додаткові можливості**:
  - дуже швидкий старт/видалення кластерів (секунди) :contentReference[oaicite:11]{index=11}  
  - вбудований локальний registry;
  - легке створення multi-node-кластерів.

---

## 3. Порівняльна таблиця

| Критерій                               | minikube                                                                 | kind                                                                    | k3d                                                                                   |
|----------------------------------------|--------------------------------------------------------------------------|-------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| Тип                                    | Локальний Kubernetes, один або кілька вузлів                             | Kubernetes-вузли як контейнери                                         | Легкий k3s у Docker-контейнерах                                                       |
| Container-engine                       | Docker, Podman (експеримент.), інші драйвери                             | Docker, Podman, nerdctl                                                | Docker (або сумісний engine)                                                         |
| ОС                                     | Linux, macOS, Windows                                                    | Linux, macOS, Windows                                                  | Linux, macOS, Windows                                                                 |
| Архітектури                            | amd64, arm64                                                             | amd64, arm64                                                           | amd64, arm64                                                                          |
| Підтримка Podman                       | Так (драйвер `--driver=podman`) :contentReference[oaicite:12]{index=12}                     | Так (через provider podman / Podman Desktop) :contentReference[oaicite:13]{index=13} | Обмежена / непряма (k3d орієнтований на Docker-API)                                  |
| Швидкість створення кластера           | Середня                                                                  | Швидка                                                                  | Дуже швидка (секунди) :contentReference[oaicite:14]{index=14}                            |
| Ресурси                                | Повноцінний Kubernetes, відносно важчий                                  | Легкий, але використовує “повний” Kubernetes                           | Найлегший, орієнтований на low-resources                                              |
| Аддони / extras                        | Багато аддонів (Dashboard, Ingress тощо)                                 | Немає аддонів “з коробки”, все як у звичайному Kubernetes              | Вбудований локальний registry, прості multi-node                                      |
| Зручність для початківців              | Дуже висока (офіційний “навчальний” варіант Kubernetes)                  | Вища для тих, хто вже працює з Docker/Podman                           | Зручний, але потребує базового розуміння Kubernetes/k3s                               |
| Типові сценарії                        | Навчання, локальна розробка, MVP                                         | Тестування маніфестів, CI, e2e-тести                                   | Локальна розробка мікросервісів, edge/IoT, PoC та швидкі кластери                    |

---

## 4. Переваги та недоліки

### 4.1 minikube

**Переваги**

- Офіційний інструмент з чудовою документацією та великою спільнотою. :contentReference[oaicite:15]{index=15}  
- Підтримує як VM, так і контейнерні драйвери (Docker/Podman).
- Зручні аддони (Dashboard, Ingress, Metrics Server) — швидкий старт.
- Добре підходить для новачків і для демонстрацій.

**Недоліки**

- Важчий за kind/k3d за споживанням ресурсів.
- Деякі фічі (rootless Podman) ще можуть бути експериментальними.
- Multi-node сценарії менш “нативні”, ніж у kind/k3d.

### 4.2 kind

**Переваги**

- Дуже легкий, вузли — звичайні контейнери. :contentReference[oaicite:16]{index=16}  
- Чудово інтегрується у CI (GitHub Actions, GitLab CI).
- Працює з Docker і Podman (через відповідний provider).
- Добрий варіант для тестування самих Kubernetes-маніфестів та операторів.

**Недоліки**

- Потребує попередньо встановленого Docker/Podman з повними правами.
- Немає “з коробки” GUI-dashboard’ів, треба ставити самостійно.
- Орієнтований більше на тестування, ніж на “зручну” локальну розробку.

### 4.3 k3d

**Переваги**

- Надзвичайно швидкий запуск/видалення кластерів. :contentReference[oaicite:17]{index=17}  
- Базується на k3s — легкий Kubernetes, менше споживання ресурсів.
- Простий multi-node, вбудований локальний registry.
- Хороший баланс між “реальним” Kubernetes-досвідом і ресурсами ноутбука.

**Недоліки**

- Менш розповсюджений, ніж minikube/kind → дещо менше прикладів в інтернеті. :contentReference[oaicite:18]{index=18}  
- Жорстка залежність від Docker-сумісного engine (з Podman є обхідні шляхи, але не так прямо, як у kind/minikube).
- K3s — спрощена дистрибуція: іноді поведінка може трохи відрізнятися від “vanilla” Kubernetes.

---

## 5. Ліцензування Docker та Podman як альтернатива

**Docker Desktop** безкоштовний для:

- індивідуальних користувачів;
- освіти, навчальних цілей;
- open-source проектів;
- малих компаній: **< 250 співробітників і < $10 млн річного доходу**. :contentReference[oaicite:19]{index=19}  

Для організацій, які перевищують ці пороги, **комерційне використання Docker Desktop вимагає платної підписки** (Pro/Team/Business). :contentReference[oaicite:20]{index=20}  

Оскільки AsciiArtify — молодий стартап з невеликою командою, на етапі PoC Docker Desktop формально може використовуватися безкоштовно. Але:

- на майбутнє краще **завчасно** мати план, як перейти на альтернативи;
- з точки зору ліцензійного ризику та vendor-lock-in доцільно одразу розглянути **Podman**.

### Podman

- Open-source, rootless-by-default container engine.
- CLI максимально сумісний з Docker (`podman run`, `podman build` тощо). :contentReference[oaicite:21]{index=21}  
- **minikube** має окремий драйвер `--driver=podman`. :contentReference[oaicite:22]{index=22}  
- **kind** може працювати поверх Podman (через provider podman, у тому числі через Podman Desktop). :contentReference[oaicite:23]{index=23}  

Для **k3d** прямої повноцінної підтримки Podman немає: інструмент орієнтований на Docker-API, хоча в деяких сценаріях Podman може емітувати Docker-socket.

**Висновок щодо ліцензії**:  
на етапі PoC AsciiArtify може спокійно використовувати Docker Desktop / Docker Engine, але варто:

- прописати в Concept, що у разі росту компанії планується перехід на **Podman + minikube/kind** чи інший безкоштовний runtime;
- уникати глибокої зав’язки саме на Desktop-функціонал.

---

## 6. Демо (вбудоване) – Hello World на k3d

У цьому розділі показано мінімальне демо з використанням **k3d** — інструменту, який рекомендується як основний для PoC AsciiArtify.

### 6.1 Передумови

- Встановлені:
  - `docker` (або сумісний engine);
  - `kubectl`;
  - `k3d` (через пакетний менеджер або `curl`-інсталер). :contentReference[oaicite:24]{index=24}  

### 6.2 Створення кластера

```bash
# створюємо двовузловий кластер для розробки
k3d cluster create asciiartify-dev \
  --agents 2 \
  --port 8080:80@loadbalancer

# перевіряємо вузли
kubectl get nodes
