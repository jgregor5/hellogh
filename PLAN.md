# hellogh — la lliçó «GitHub way», dins la infraestructura que ja tens

> **Com fer servir aquest fitxer.** És el pla complet del projecte `hellogh`, pensat per continuar
> en una sessió nova de Claude sense context previ.
>
> - **«Què toca ara?»** → el **primer `[ ]` sense marcar** del *Full de ruta* d'aquí sota. Res més.
> - En acabar una tasca, **marca-la `[x]`** amb la data i fes-ne commit en el mateix commit que la
>   feina. El full de ruta és la font de veritat de l'estat del projecte.
> - Abans de fer res, llegeix la **secció 0** (estat, fets verificats, descartats) i **no tornis a
>   derivar el que ja hi és**. El detall de cada fase és a *Passos d'execució*, al final.

---

## Full de ruta — fet / pendent

Sis fases en ordre de dependència. **No es comença una fase sense haver superat la porta de
l'anterior.** El detall de cada una és a [Passos d'execució](#passos-dexecució).

### Fase 0 — Escriure el projecte · *jo* · **← ETS AQUÍ**

- [x] Repositori a GitHub creat, públic, `origin` configurat *(2026-08-06)*
- [x] `gh` autenticat com a `jgregor5` amb scope `workflow` *(2026-08-06)*
- [x] Branca local reanomenada a `master` *(2026-08-06)*
- [x] `PLAN.md` sanejat i publicable; dades reals a `INFRA.local.md` gitignorat *(2026-08-06)*
- [x] Seqüència d'execució redissenyada en 6 fases amb portes *(2026-08-06)*
- [x] Especificats el contracte d'execució (3 invariants) i l'exercici B *(2026-08-06)*
- [ ] Obtenir els **SHA complets** de les 4 accions (pendent 0.6.4)
- [ ] `app/` + `tests/`
- [ ] `Dockerfile` · `compose.yaml` · `Makefile` · `requirements.txt` · `.dockerignore` · `.env.example`
- [ ] `.github/workflows/{ci,deploy,lab-b}.yml` + `.github/dependabot.yml`
- [ ] `README.md` · `COMPARISON.md` · `LAB.md`  *(`SETUP.md` va a la Fase 3)*
- [ ] **Porta:** `make docker-test` verd i `make docker-up && make health` respon `version: "dev"`

> **En escriure `compose.yaml` i `deploy.yml`, respecta el «Contracte d'execució» de la Part 1**:
> `PORT` mai al `.env` del contenidor, `.env` s'escriu amb `>` i no `>>`, i verifica `id cicd`.
> Són els tres errors que donen un 502 sense cap missatge d'error.

### Fase 1 — Publicar · *jo, amb la teva autorització*

- [ ] `git push -u origin master` *(demano confirmació abans)*
- [ ] **Porta:** `ci.yml` verd — i el check `validate` ja existeix per a la Fase 4

### Fase 2 — Contenidor i xarxa · *tu*

- [ ] `incus storage info` — confirmar ~10GB lliures (pendent 0.6.1)
- [ ] Les **dues** entrades a `lockdebian/inventories/remote.yaml` (Part 2)
- [ ] Executar `lxcsetup-containers.yaml` i `frpc.yaml`
- [ ] **Porta:** el contenidor va, té Docker, i `<domini-app>` resol (502 encara)

### Fase 3 — Runner self-hosted · *tu*

- [ ] Instal·lar i registrar el runner al contenidor (5 passos)
- [ ] `incus exec hellogh -- id cicd` → confirmar que és **uid 1000** (invariant 3 de la Part 1)
- [ ] Escriure `SETUP.md` amb la versió real que s'hagi fet servir
- [ ] **Porta:** el runner surt **Idle** a *Settings → Actions → Runners*

### Fase 4 — Configuració de GitHub · *tu, per web*

- [ ] `gh variable set APP_DOMAIN`
- [ ] Environment `production`: required reviewers + només tags `v*`
- [ ] Ruleset a `master`: exigir PR + check `validate`
- [ ] **Porta:** push directe a `master` rebutjat; tag `v*` s'atura esperant aprovació

### Fase 5 — Primer desplegament real · *tots dos*

- [ ] `git tag -a v1.0.0` + push
- [ ] **Comprovar la visibilitat del paquet a GHCR** i fer-lo públic si cal — **irreversible**
- [ ] Aprovar el desplegament
- [ ] **Porta:** `curl https://<domini-app>/health` → `{"status":"healthy","version":"v1.0.0"}`

### Fase 6 — Exercicis de classe · *tu*

- [ ] A (PR bloquejat) · B (estat persistent) · C (imatge pública) · D (aprovació) · E (atestació)

---

## 0. Punt de partida (llegir primer)

### 0.1 Estat actual — res construït encara

| Element | Estat (verificat el 2026-08-06) |
|---|---|
| `hellogh/` (local) | Repo git a **`master`** (com lockdebian, no `main`). Conté `.gitignore`, `PLAN.md` i `INFRA.local.md` (gitignorat). **Cap fitxer del projecte escrit encara** |
| Repositori a GitHub | ✅ **Creat**: https://github.com/jgregor5/hellogh (públic, `origin` configurat). **Encara no s'hi ha pujat res** |
| Usuari de GitHub | **`jgregor5`** — tot en minúscules, o sigui que `ghcr.io/jgregor5/hellogh` és vàlid i `${{ github.repository }}` es pot fer servir sense problema |
| `gh` autenticat | ✅ com a `jgregor5`, amb scopes `repo`, `workflow`, `read:org`, `gist` (el `workflow` cal per pujar `.github/workflows/`) |
| Contenidor Incus `hellogh` | **No existeix** encara |
| `lockdebian` i `gitodebian` | Repositoris git, **sense cap canvi fet per aquest projecte** |

### 0.2 Entorn

| Fet | Valor |
|---|---|
| Directori de treball | `<arrel>/ansible` — **no és cap repo git**, és un contenidor de projectes |
| Projectes germans | `lockdebian/` i `gitodebian/`, **repos git independents**. Atenció al nom: és **`gitodebian`**, no «gitdebian» |
| Projectes d'alumne de referència | `<arrel>/2627/iad/` → `mlops`, `llmml`, `impl`, `prob`… (**fora** de l'arbre `ansible/`) |
| Màquina local | Ubuntu 26.04 LTS. `docker` ✅ `git` ✅ `make` ✅ `curl` ✅ `jq` ✅ `python3` ✅ (pyenv) `gh` ✅ (2.97.0, a `/usr/bin/gh`) |
| Host lockdebian | Àlies SSH **`cicdies`**, a la LAN del centre. Els playbooks hi apunten amb `hosts: ciserver`, definit a `inventories/common.yaml` |
| Altres àlies SSH | `gities` i `ops` → `<host-gitolite>` (gitolite / verbs d'ops). **No hi ha àlies per al VPS** |
| VPS | `<host-vps>`, Debian 13. Caddy (certificat wildcard `<domini-comodí>`) + frps. **No el gestiona Ansible** i **aquest projecte no el toca** |

> **Marcadors.** `<arrel>`, `<host-gitolite>` i `<host-vps>` són marcadors: els valors reals
> (rutes, IPs, usuaris i ports) són a `INFRA.local.md`, que està al `.gitignore` perquè aquest
> repositori és públic. Aquest `PLAN.md` sí que es publica.

### 0.3 Mapa de fitxers de referència

Rutes **relatives a `<arrel>/`**, que és l'arrel comuna de tot el que se cita aquí.
Aquest fitxer és `ansible/hellogh/PLAN.md`.

| Fitxer | Per a què serveix aquí |
|---|---|
| `ansible/lockdebian/GITHUB.md` | **El document al qual respon tot això.** 13 seccions; les «§N» que se citen al pla són d'aquí |
| `ansible/lockdebian/README.md` | El pipeline actual. §7 contracte de hooks, §11 contenidors autònoms, §12 proxy de ports |
| `ansible/lockdebian/FRPS.md` | Muntatge del VPS (Caddy + frps). Context, no s'hi toca |
| `ansible/lockdebian/inventories/remote.yaml` | **L'únic fitxer a modificar fora de `hellogh/`** (Part 2) |
| `ansible/lockdebian/inventories/common.yaml` | Defineix el host `ciserver` |
| `ansible/lockdebian/playbooks/lxcsetup-containers.yaml` | Crea contenidors i hi instal·la Docker (línies 38-81) |
| `ansible/lockdebian/playbooks/frpc.yaml` | Instal·la i configura frpc. **Línia 50: el parany** (vegeu 0.4.3) |
| `ansible/lockdebian/conf/frpc.j2.toml` | Plantilla frpc, `type = "http"` fix. Es llegeix, **no** es modifica |
| `ansible/lockdebian/conf/cicd-cleanup.j2.sh` | La neteja de les 3 AM. Es llegeix, **no** es modifica |
| `ansible/lockdebian/conf/cicd-worker.j2.sh` | El worker que a hellogh **no s'executarà mai** |
| `ansible/lockdebian/docker-hello/` | App trivial que ja existeix (`Dockerfile` + `server.py`); útil com a punt de comparació |
| `ansible/gitodebian/inventories/remote.yaml` | `proj_groups` i `cicd_repos`. **No es toca res** (Part 2) |
| `2627/iad/mlops/{Dockerfile,compose.yml,Makefile}` | **Les convencions a copiar** |
| `2627/iad/mlops/.cicd/hooks/*.sh` | Els 4 hooks que `COMPARISON.md` ha de mapar |

### 0.4 Fets ja verificats — no cal tornar-ho a comprovar

1. **GitHub Free + repo privat perd el que volem ensenyar.** *Branch protection* i *environments
   amb required reviewers* només existeixen en repos **públics** al pla Free; en privat calen
   Pro/Team. A més: minuts d'Actions il·limitats i GHCR il·limitat en públic, contra 2.000 min/mes
   i 500 MB en privat. **Per això el repo és públic.**
2. **Un contenidor SENSE `user:` a l'inventari rep Docker.** `lxcsetup-containers.yaml:38-81` només
   instal·la Docker quan `(item.value.user | default(lxc_cicd_user)) == lxc_cicd_user`. Els
   contenidors «autònoms» (`user:` definit) **no reben Docker** — no ens serveixen.
3. **⚠️ `frpc.yaml:50` peta si falta l'entrada frpc.** Fa
   `frp_proxies: "{{ lxc_frpc_configs[item.key] }}"` iterant sobre **totes** les entrades de
   `lxc_machines`, **sense `default()`**. Afegir `hellogh` només a `lxc_machines` trencaria la
   següent execució de `frpc.yaml` **per a tots els contenidors**. (El README §11 diu que
   `lxc_frpc_configs` és opcional: és una divergència doc/codi.)
4. **Els perfils Incus es comparteixen.** A `local.yaml`, `group01` i `group02` fan servir tots dos
   `devops-project`. Per tant `hellogh` pot fer servir `prjref-project` (2 CPU / 4GB / 10GB) i
   **no cal declarar cap `lxc_profiles` nou** ni tocar `lxc_pool_size` (`25GB`, amb perfils que ja
   sumen 15+10=25GB).
   - **`ssh_port: 2203` està lliure**: els ports ja assignats a `remote.yaml` són a `INFRA.local.md`.
   - **`local_port: 8080` no xoca amb res**, encara que `mlops` (8080, 9080) i `prjref`
     (8080, 8081) ja el facin servir: **frpc corre dins de cada contenidor**, i cada contenidor té
     el seu propi loopback. El que ha de ser únic és el `domain`, no el port.
5. **`cicd-cleanup.sh` és compatible.** Itera sobre *tots* els contenidors en marxa i fa
   `docker image prune -a -f`, però les imatges d'un contenidor **en marxa** queden exemptes. Les
   `:v1.0.0` antigues es recullen després d'un desplegament nou, que és el que volem.
6. **El VPS ja ho té tot per exposar `<domini-app>`.** El registre A comodí i el
   certificat wildcard `<domini-comodí>` de Caddy ja cobreixen qualsevol subdomini nou. **Zero canvis
   al VPS.**
7. **Convencions exactes de `mlops`** (comprovades): `FROM python:3.13-slim AS base`; capçaleres
   `# ==================== BASE STAGE ====================`; etapes `base` → `test` → `production`;
   `CMD ["pytest", "tests/", "-v"]` a `test`; `EXPOSE 8000`; `HEALTHCHECK --interval=30s
   --timeout=5s --start-period=10s --retries=3`; `CMD ["uvicorn", "app.main:app", "--host",
   "0.0.0.0", "--port", "8000"]`. Al `compose.yml`: `name: mlops` a dalt de tot,
   `ports: "${PORT:-8080}:8000"`, `env_file: [.env]`, `user: "${UID:-1000}:${GID:-1000}"`,
   `restart: unless-stopped`. Makefile amb `help setup test docker-build docker-test docker-up
   docker-down health clean`.

### 0.5 Ja descartat — no ho tornis a proposar

| Descartat | Motiu |
|---|---|
| Desplegar al VPS `<host-vps>` per SSH | Obligaria a obrir SSH entrant a la màquina que termina el TLS de tots els alumnes, amb un usuari al grup `docker` (= root efectiu) i una clau en mans de GitHub. El contenidor Incus ho evita tot |
| Un playbook nou (`ghrunner.yaml`) | Restricció explícita de l'usuari: **cap script nou ni modificat** a lockdebian ni gitodebian. El runner s'instal·la a mà, un cop |
| Exposar el SSH del contenidor amb un proxy TCP d'frp | Caldria modificar `conf/frpc.j2.toml` (`type = "http"` fix), que renderitza la config de **tots** els contenidors d'alumne |
| Repositori privat | Perd *branch protection* i *environments* al pla Free (vegeu 0.3.1) |
| Un `undeploy.yml` | L'usuari el va treure de l'abast. `COMPARISON.md` el llista com a «sense equivalent a GitHub» |
| Incus **dins del VPS** | Replicaria la capa d'aïllament, però cal instal·lar Incus en un VPS que només té Caddy+frps, per una app d'un sol propietari |
| Un `lxc_profiles` nou per a hellogh | Innecessari: els perfils es comparteixen (0.4.4) |

### 0.6 Pendents oberts

**Res bloquejant.** El nom d'usuari (`jgregor5`) i el repositori ja estan resolts — vegeu 0.1.
Substitueix `jgregor5` per `jgregor5` a tot el document quan escriguis els fitxers; la ruta
`ghcr.io/jgregor5/hellogh` és vàlida (tot minúscules).

**A comprovar durant l'execució, no abans** (cada un té una fase assignada als passos d'execució):

1. **Espai real al pool d'Incus** — *Fase 2*: `incus storage info` des de `cicdies`. No cal ampliar
   `lxc_pool_size`, però convé veure que hi hagi ~10GB lliures.
2. **Visibilitat del paquet a GHCR** — *Fase 5*, després del primer build. Les fonts es contradiuen
   (vegeu la nota a «Com viatja la imatge»); s'ha de resoldre empíricament. L'exercici C necessita
   que sigui públic, i el canvi de visibilitat és **irreversible**.
3. **Versió del runner d'Actions** — *Fase 3*: consultar
   https://github.com/actions/runner/releases. El pla **no la fixa a propòsit**, perquè quedaria
   obsoleta.
4. **Versions i SHA de les accions** — *Fase 0*, just abans d'escriure els workflows. Les majors
   vigents el 2026-08-06 són `actions/checkout` v7, `docker/build-push-action` v7 i
   `docker/login-action` v4 (vegeu «Conformitat amb la pràctica del sector»). **Torna-ho a
   comprovar el dia que escriguis els workflows** i agafa el SHA complet de cada release — canvien
   sovint, i les del `GITHUB.md` (v4/v6/v3) ja són obsoletes.
5. **Com entra `<domini-app>` als workflows** — *Fase 0* (decisió) i *Fase 4* (aplicació). El domini
   real no s'escriu literalment al repositori públic: `deploy.yml` el llegeix de la variable de
   repositori `APP_DOMAIN`. Vegeu la nota a `INFRA.local.md`.

### 0.7 Convencions de documentació (per als `.md` en català a escriure)

Copiades de `GITHUB.md` i `README.md` de lockdebian: títol `# hellogh — Guia d'ús`; seccions
`## N. Títol`; procediments `### Pas N — …`; taules de markdown per a tota la referència;
diagrames **ASCII** amb `│ ├─ └─ ▼` dins de blocs ` ``` ` o ` ```text ` — **mai mermaid**; `> `
per als advertiments; fences ` ```shell `, ` ```yaml `, ` ```bash `.

---

## Context

`lockdebian/GITHUB.md` explica com seria el pipeline amb GitHub Actions, però és teoria: no hi ha
cap projecte que ho executi. Els alumnes ja tenen un model mental sòlid de lockdebian, així que la
lliçó **no és la sintaxi YAML** (se la miraran quan la necessitin). El que cal ensenyar són les
quatre idees que a lockdebian no existeixen:

1. **La porta és abans del merge, no després del push.** A lockdebian la validació corre sobre codi
   que **ja és** a `master`. A GitHub, el PR executa el CI i la protecció de branca deixa el botó de
   merge en gris. Mateixos tests, significat oposat.
2. **Construir una vegada, desplegar aquell artefacte exacte.** Un tag produeix una imatge immutable
   a GHCR i producció corre **aquella**. lockdebian reconstrueix al destí.
3. **L'entorn d'execució és d'un sol ús.** Cada job comença de zero — no és una limitació, és el
   mecanisme que fa els builds reproduïbles.
4. **L'aprovació i els secrets són configuració, no infraestructura.** El `promote` es converteix en
   una casella i un revisor amb nom.

### La decisió que ho ordena tot

hellogh corre en un **contenidor Incus al host lockdebian**, provisionat exactament com el d'un
grup d'alumnes i exposat pel **mateix frpc + frps + Caddy**. Conseqüències:

- **El VPS `<host-vps>` no es toca gens.** Cap usuari `deploy`, cap Docker, cap bloc al Caddyfile,
  cap SSH entrant. La màquina que termina el TLS de tots els alumnes no s'exposa més.
- **La comparació té una sola variable.** Mateix contenidor, mateix Docker, mateix frpc, mateix
  Caddy, mateix `<domini-comodí>`, mateix aïllament. L'**única** diferència és qui porta el codi.
- **`GITHUB.md` §12 passa de ser una afirmació a ser observable**: FRP i Caddy es queden igual;
  Actions només substitueix el transport del codi.

### Abast a lockdebian: només inventari, cap script

Restricció acceptada: **no es crea ni es modifica cap playbook, cap plantilla ni cap script** ni a
lockdebian ni a gitodebian. L'únic canvi és **configuració a l'inventari**, i després s'executen
els playbooks que ja existeixen.

**Clau: NO definir `user` a l'entrada de l'inventari.** Segons el
[README §11](../lockdebian/README.md), un contenidor amb `user` definit és
*autònom* i **no rep Docker** — no ens serveix. Amb `user` sense definir es provisiona com a
contenidor CI/CD i rep, via
[`lxcsetup-containers.yaml:38-81`](../lockdebian/playbooks/lxcsetup-containers.yaml):
Docker + `docker-compose-plugin`, l'usuari `cicd` al grup `docker`, `/opt/cicd/cicd-worker.sh` i
`/home/cicd/repos`.

> **I aquí hi ha el millor moment pedagògic del projecte.** Com que cap repositori `hellogh` no
> estarà registrat a gitolite, `cicd-router.sh` no hi entrarà mai i **`/opt/cicd/cicd-worker.sh`
> es quedarà allà sense executar-se ni una sola vegada**. El contenidor és idèntic al d'un grup;
> l'únic que canvia és qui porta el codi. Es pot assenyalar aquell fitxer i dir: *això és
> exactament la peça que GitHub substitueix.*

### Com hi entra GitHub: runner self-hosted dins del contenidor

El contenidor és darrere del NAT de l'institut, així que un runner d'Azure no hi pot fer SSH. S'hi
instal·la l'**agent de runner de GitHub**, que manté una connexió **sortint** cap a GitHub i rep
els jobs per aquell canal. El desplegament és **local**: `docker compose up -d`, sense SSH, sense
clau de desplegament, **sense cap secret**.

Sense playbook nou, la instal·lació són ~8 comandes executades **una vegada**, documentades a
`hellogh/SETUP.md`. Per a un sol contenidor és el canvi correcte — i és més llegible per als
alumnes que un playbook: veuen què és, de debò, «instal·lar un runner».

#### Com funciona l'agent per dins

Aquesta subsecció ha d'anar tant a `SETUP.md` (per instal·lar-lo amb criteri) com a
`COMPARISON.md` (perquè conté la comparació més precisa de tot el projecte).

**Registre (un sol cop).** `config.sh` canvia un **token de registre efímer** (caduca en 1 hora)
per una **credencial permanent**:

```text
config.sh --url … --token <efímer> --labels hellogh
      │
      ├─▶ crida l'API de GitHub i registra el runner al repositori
      │
      └─▶ escriu a ~cicd/actions-runner/
            .runner                  # id, nom, etiquetes, URL  (JSON llegible)
            .credentials             # esquema d'autenticació
            .credentials_rsaparams   # ← CLAU PRIVADA: la identitat del runner
```

El token de registre només serveix per a aquest intercanvi. A partir d'aquí el runner s'autentica
amb aquella clau RSA — per això `.credentials_rsaparams` és sensible: **qui la pugui llegir pot
suplantar el runner i rebre'n els jobs**.

**Funcionament normal.** Dos processos i un bucle de *long polling*:

```text
┌─ contenidor hellogh ──────────────────────────────────────┐
│                                                            │
│  Runner.Listener   ← el servei systemd, sempre viu         │
│      │                                                     │
│      │  HTTPS 443 SORTINT, long polling:                   │
│      │  obre una petició que GitHub manté oberta ~50 s     │
│      │  fins que hi ha feina o expira; llavors la reobre   │
│      ▼                                                     │
│   … espera …                                               │
│      │                                                     │
│      │  arriba un missatge de job (xifrat)                 │
│      │  → el desxifra amb la seva clau                     │
│      ▼                                                     │
│  fork ──▶ Runner.Worker   ← UN procés nou per cada job     │
│                │                                           │
│                ├─ executa els steps en ordre               │
│                ├─ puja els logs en directe (mateixa via)   │
│                └─ reporta l'estat final i MOR              │
│                                                            │
│  el Listener torna a fer poll                              │
└────────────────────────────────────────────────────────────┘
```

La separació és el que importa: `Runner.Listener` és el dimoni de llarga vida que només **espera**;
`Runner.Worker` s'engendra per job, fa la feina i mor. **`Runner.Worker` és l'equivalent exacte de
`cicd-worker.sh`** — aquesta és la frase per a `COMPARISON.md`.

**Què queda al disc entre execucions:**

```text
~cicd/actions-runner/_work/
├── hellogh/hellogh/     ← $GITHUB_WORKSPACE, on fa el checkout
├── _actions/            ← accions descarregades (actions/checkout…) — CAU
├── _tool/               ← toolchains instal·lades
└── _temp/
```

**Això és tot l'exercici B.** A `ubuntu-24.04` tot això es destrueix amb la VM; aquí sobreviu,
juntament amb la cau d'imatges de Docker. Un job que passa perquè l'execució anterior va deixar
alguna cosa, passa **aquí** i peta **allà**.

**El contrast amb lockdebian** (nucli de `COMPARISON.md`):

| | `cicd-worker.sh` | runner de GitHub |
|---|---|---|
| Qui inicia | el **router**, entrant per SSH | el **runner**, sortint per HTTPS |
| Què és | un script que s'executa i acaba | un **dimoni** (Listener) que engendra un procés per job (Worker) |
| Credencial | clau SSH amb forced command, al router | clau RSA pròpia, dins del contenidor |
| Ports oberts | 22 al contenidor | **cap** |
| Estat entre execucions | directori efímer `<repo>-<sha8>` | `_work` persistent |

> **Dos detalls operatius.** El runner **s'auto-actualitza** per defecte quan GitHub en publica una
> versió nova (`--disableupdate` ho evita, si vols que la classe en vegi una de fixa). I els
> **secrets arriben dins del missatge de job xifrat**, es desxifren en memòria al Worker i
> s'emmascaren als logs: no s'escriuen mai al disc si cap step no ho fa explícitament.

#### Per què no per SSH

> Alternativa descartada: exposar el SSH del contenidor amb un proxy TCP d'frp (`frps.toml` ja
> té un `allowPorts` configurat). Requeriria modificar
> [`conf/frpc.j2.toml:14`](../lockdebian/conf/frpc.j2.toml), que té
> `type = "http"` fix i renderitza la config de **tots** els contenidors d'alumne — i posaria el
> SSH d'un contenidor d'aula a internet.

**Bonus de la mateixa decisió.** El blockquote de
[`GITHUB.md:512`](../lockdebian/GITHUB.md) diu que el runner self-hosted és
*el mateix truc que FRP*. Al contenidor n'hi haurà **dos, de costat**:

```shell
incus exec hellogh -- systemctl status frpc              # sortint: tràfic dels usuaris
incus exec hellogh -- systemctl status actions.runner.*  # sortint: jobs de CI
```

Cap dels dos obre cap port. Travessia de NAT com a patró general, no com a funcionalitat de GitHub.

### Seguretat: repo públic + runner self-hosted

GitHub ho desaconsella perquè el PR d'un fork pot executar codi arbitrari a la teva màquina. Aquí
la mitigació és **estructural**, no una casella:

| Workflow | `runs-on` | Disparadors | Pot executar codi d'un fork? |
|---|---|---|---|
| `ci.yml` | `ubuntu-24.04` | `pull_request`, `push: master` | Sí — però és una **VM d'un sol ús de GitHub**, no teva |
| `deploy.yml` | `[self-hosted, hellogh]` | `push: tags: v*`, `workflow_dispatch` | **No** — un fork no pot empènyer un tag |

El codi no confiable no arriba mai al runner, perquè l'únic workflow que l'usa no es pot disparar
des d'un fork. Amb l'`environment: production` a sobre, també cal un humà.

**Risc residual, dit clar:** qui tingui **accés d'escriptura** al repo pot executar codi en aquell
contenidor — si hi afegeixes alumnes com a col·laboradors, són ells. Que forquin i obrin PRs.
Incus limita el dany a un contenidor d'un sol ús, que és exactament per a això que hi és.

### La restricció que decideix l'abast de la classe

| Meitat de la lliçó | Cost | Qui la fa |
|---|---|---|
| **CI / pull request** (idees 1 i 3) | **Zero.** Repo públic = minuts il·limitats, cap servidor | **Cada alumne al seu repo** |
| **Desplegament a producció** (idees 2 i 4) | Un contenidor, un subdomini, un runner | **Només `hellogh`, el condueixes tu** |

### Decisions ja preses

| Decisió | Tria | Motiu |
|---|---|---|
| Visibilitat | **Públic** | A Free, *branch protection* (§5 Pas 4) i *environments amb required reviewers* (§8) només existeixen en repos públics — justament les idees 1 i 4. A més: minuts i GHCR il·limitats |
| On corre | **Contenidor Incus CI/CD (`user` sense definir)** | Reutilitza frpc+frps+Caddy; el VPS no es toca; comparació d'una sola variable |
| Com hi entra GitHub | **Runner self-hosted, instal·lat a mà** | Sortint com frpc. Sense SSH, sense clau, sense secrets. I sense playbooks nous |
| `ci.yml` | **`ubuntu-24.04`** | Els PRs no toquen mai la teva màquina; i cal una VM neta per a l'exercici B. Imatge fixada, no `-latest` |
| Idioma | **Català** | Coherent amb `README.md`/`GITHUB.md`: `# hellogh — Guia d'ús`, seccions numerades, diagrames ASCII, sense mermaid |
| App | **Trivial a propòsit** | Cada minut en lògica d'aplicació és un minut no gastat en el concepte |

---

## Part 1 — El repositori `hellogh/`

```
hellogh/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml            # ubuntu-24.04 — PR + push a master
│   │   ├── deploy.yml        # build a ubuntu-24.04, deploy al runner self-hosted
│   │   └── lab-b.yml         # workflow_dispatch — l'exercici B (vegeu LAB.md)
│   └── dependabot.yml        # manté al dia els SHA de les accions
├── app/{__init__.py,main.py} # FastAPI: GET / i GET /health → {"version": APP_VERSION}
├── tests/{__init__.py,test_main.py}
├── Dockerfile                # multi-stage base → test → production
├── compose.yaml              # name: hellogh, ${PORT:-8080}:8000, image: + build:
├── Makefile                  # docker-build docker-test docker-up docker-down health
├── requirements.txt          # fastapi, uvicorn[standard], pytest, httpx
├── .dockerignore .gitignore .env.example
├── README.md                 # Guia d'ús
├── SETUP.md                  # inventari + instal·lació del runner (5 passos)
├── COMPARISON.md             # lockdebian ↔ GitHub, peça per peça
└── LAB.md                    # els 5 exercicis (A–E) — el document que fa la classe
```

`app/main.py` trivial: `GET /` retorna un missatge, i `GET /health` retorna
`{"status":"healthy","version": os.getenv("APP_VERSION", "dev")}` — perquè el smoke test i
l'exercici C puguin comprovar que producció corre exactament el tag construït. D'on surt
`APP_VERSION` s'explica a «Com viatja la imatge».

**Convencions a copiar de [`2627/iad/mlops`](../../2627/iad/mlops)**
(la forma, perquè l'única diferència visible amb un projecte d'alumne sigui la capa CI/CD):
`FROM python:3.13-slim AS base`, capçaleres `# ===== BASE STAGE =====`, `requirements.txt` abans
del codi, etapa `test` amb `CMD ["pytest","tests/","-v"]`, etapa `production` amb `EXPOSE 8000` i
`HEALTHCHECK curl --fail .../health`; `compose.yaml` amb `user: "${UID:-1000}:${GID:-1000}"`,
`restart: unless-stopped`, i **`image:` a més de `build:`** perquè el desplegament faci `pull` i no
`build` — que és precisament la idea 2; `Makefile` amb `-include .env`, `.DEFAULT_GOAL := help`.

> `name: hellogh` fix al `compose.yaml` no és cosmètic: `cicd-cleanup.sh` es queixa (al seu propi
> comentari) que compose deriva el nom de projecte del directori amb el sha i va deixant xarxes
> `<repo>-<sha>_default` que esgoten el pool de subxarxes. Amb el nom fix, hellogh no en deixa cap.

### Contracte d'execució — tres invariants que si es trenquen no es veuen

Aquestes tres coses **han de ser certes** perquè el camí frpc → frps → Caddy funcioni. Cap de les
tres dona un error clar quan falla: donen un 502 amb tots els components dient que estan bé.

**1. La cadena de ports ha de quedar quadrada en 8080.**

```text
Caddy (<domini-comodí>)  ──▶  frps  ──▶  frpc dins del contenidor
                                              │  local_port: 8080   ← inventari, Part 2
                                              ▼
                                   compose:  "${PORT:-8080}:8000"
                                              │
                                              ▼
                                        uvicorn :8000  ← EXPOSE del Dockerfile
```

> ⚠️ **`PORT` no s'ha de definir MAI al `.env` del contenidor.** El valor per defecte (8080) és el
> que espera l'entrada `local_port` de l'inventari. Si algú afegeix `PORT=` a `.env.example` i
> aquell fitxer arriba al contenidor, frpc continuarà enviant al 8080, ningú no hi escoltarà i el
> resultat és un **502 sense cap error a cap log**. `PORT` només té sentit en local, per no xocar
> amb un altre projecte.

**2. El job `deploy` **sobreescriu** `.env`, no hi afegeix res.**

```shell
printf 'IMAGE_TAG=%s\n' "$TAG" > .env      # `>`, mai `>>`
```

Amb `>>` el fitxer aniria acumulant un `IMAGE_TAG` per desplegament. Compose es queda amb l'últim,
o sigui que *sembla* que funciona fins que algú llegeix el fitxer i no entén res. `.env` al
contenidor conté **només** `IMAGE_TAG`: tota la resta són els defaults del `compose.yaml`.

**3. `user: "${UID:-1000}:${GID:-1000}"` cau al default sota el runner.**

`UID` i `GID` són variables de *shell*, no d'entorn exportades: compose no les veu quan el job les
invoca de forma no interactiva, i agafa `1000:1000`. Funciona perquè l'usuari `cicd` del contenidor
**és** uid 1000 — però **verifica-ho** a la Fase 3 amb `incus exec hellogh -- id cicd`. Si no ho
fos, els fitxers del volum quedarien d'un altre propietari. En local, on `make` sí que exporta
`UID`/`GID`, el comportament és el correcte.

### `ci.yml`

- `on: pull_request` + `push: branches: [master]`; job `validate`, **`runs-on: ubuntu-24.04`**
  (fixat, no `ubuntu-latest` — vegeu «Conformitat»), `timeout-minutes: 10` (explícit, §7)
- `permissions: {contents: read}` explícit al capdamunt
- `actions/checkout` **fixat per SHA** → `docker build --target test -t hellogh-test .` →
  `docker run --rm hellogh-test`

Equivalent de `validate.sh` + `pre-deploy.sh` del contracte de hooks.

### Com viatja la imatge (el mecanisme central)

#### Abans de res: tres màquines, tres sistemes operatius

És la confusió més fàcil de tenir en tot el projecte. **Que el contenidor Incus sigui Debian 13 no
té res a veure amb el `runs-on: ubuntu-24.04`**: són màquines diferents.

| Màquina | SO | Qui la gestiona | Què hi corre |
|---|---|---|---|
| Runner de GitHub | **`ubuntu-24.04`** | GitHub (VM efímera a Azure) | `ci.yml` sencer + el job `build`: `docker build` i `docker push` |
| Contenidor Incus `hellogh` | **Debian 13** (`image: images:debian/13`) | Tu, via Ansible | L'agent del runner + el job `deploy`: `docker compose pull` i `up -d` |
| Contenidor Docker de l'app | **`python:3.13-slim`** (base Debian) | El `Dockerfile` | `uvicorn` |

Els dos `runs-on:` hi encaixen així:

- `runs-on: ubuntu-24.04` → «dona'm una VM d'un sol ús **de GitHub**». Sí que es tria el SO, però
  **del catàleg que publica GitHub**: Ubuntu (`ubuntu-slim`, `-22.04`, `-24.04`, `-26.04` en
  preview, i variants `-arm`), Windows i macOS. **Debian no hi és**, ni cap altra distribució. Tota
  la discussió de fixar la imatge (secció «Conformitat») **només afecta aquesta fila**.
- `runs-on: [self-hosted, hellogh]` → «dona-ho a **la meva** màquina etiquetada `hellogh`».
  **No hi ha cap etiqueta de SO**, perquè és el que sigui aquella màquina: el teu Debian 13.

**Si algun dia calgués Debian al runner de GitHub**, es fa amb `container:`, no amb `runs-on:`:

```yaml
runs-on: ubuntu-24.04     # la VM amfitriona, la posa GitHub
container: debian:13      # els steps corren AQUÍ DINS
```

**A hellogh seria redundant, i no s'ha de fer.** El `ci.yml` fa `docker build --target test` i
`docker run`: els tests ja corren dins de `python:3.13-slim`, que **és Debian 13 (trixie)** —
comprovat amb `docker run --rm python:3.13-slim cat /etc/os-release`. Posar-hi `container: debian:13`
donaria Debian 13 → Docker → Debian 13. A més, la imatge `debian:13` **no porta el CLI de Docker**,
i tot el job són comandes `docker`: caldria instal·lar-lo a cada execució i dependre que el socket
estigui muntat al contenidor del job. Més peces, cap benefici, i una capa més per als alumnes.

`container:` s'ho val quan els steps corren **directament al runner** i no dins de Docker: compilar
una extensió C contra la glibc de Debian, o fer `pip install && pytest` a la màquina, on la libc i
els paquets de sistema han de coincidir amb producció. Aquí no passa res d'això.

El SO amfitrió només ha de ser *una màquina amb Docker* — cap step no toca l'userland d'Ubuntu.
Fixar `ubuntu-24.04` és per no endur-se una migració a mig curs, no perquè calgui Ubuntu. És la
mateixa lliçó que lockdebian ensenya des de l'altre costat: **quan el build és dins de Docker,
l'amfitrió deixa d'importar.** De fet, de les tres màquines, **dues són Debian 13**; l'única que no
ho és, és justament aquella on res no corre fora d'un contenidor.

> El tarball de l'agent és `actions-runner-linux-x64`, independent de la distribució; el que aporta
> les dependències pròpies de Debian és `installdependencies.sh` (pas 2 de la instal·lació).

#### El recorregut

**La imatge no passa mai pel túnel del runner.** Les dues màquines no es connecten entre elles:
totes dues obren una connexió **sortint** cap a `ghcr.io`, que és l'únic punt de trobada.

```
Runner ubuntu-24.04 ──docker push──▶ ghcr.io ◀──docker compose pull── contenidor hellogh
   (construeix)                    (immutable)                          (executa)
```

Pel túnel del runner només hi baixa la definició del job, el clon del repo (només cal per al
`compose.yaml`), el `GITHUB_TOKEN` de l'execució, i pugen els logs. Les capes de la imatge, no.

| # | On | Què passa |
|---|---|---|
| 1 | GitHub | veu el tag `v1.0.0` i arrenca `deploy.yml` |
| 2 | runner `ubuntu-24.04` | `docker build --target production` |
| 3 | runner → ghcr.io | `docker push` de capes + manifest |
| 4 | GitHub | encua `deploy` per a l'etiqueta `hellogh`; s'atura a l'`environment` |
| 5 | — | **aprovació humana** |
| 6 | runner del contenidor | rep el job pel túnel que ja tenia obert |
| 7 | contenidor → ghcr.io | `docker compose pull` |
| 8 | contenidor | `docker compose up -d` |

Perquè el pas 7 funcioni, `compose.yaml` ha de **nomenar** la imatge, no només construir-la:

```yaml
services:
  app:
    image: ghcr.io/jgregor5/hellogh:${IMAGE_TAG:-latest}
    build: { context: ., target: production }   # només per a `make` en local
    environment:
      - APP_VERSION=${IMAGE_TAG:-dev}           # el que retorna GET /health
    ports:
      - "${PORT:-8080}:8000"
    restart: unless-stopped
```

`build:` hi és perquè `make docker-up` segueixi funcionant al portàtil; en producció només s'usa
`image:`, perquè `pull` + `up -d` no invoca mai cap build.

**El circuit de la versió**, que és el que fa verificables el smoke test i l'exercici C:

```
tag v1.0.0 ──▶ el job deploy escriu  IMAGE_TAG=v1.0.0  al .env
                          │
                          ├──▶ image: …/hellogh:v1.0.0     (quina imatge baixa)
                          └──▶ APP_VERSION=v1.0.0          (què respon /health)
                                        │
                    GET /health ──▶ {"status":"healthy","version":"v1.0.0"}
```

Les dues cares surten de la **mateixa** variable, així que si `/health` respon el tag esperat, és
que corre exactament la imatge que s'ha construït en aquell tag. En local, sense `.env`,
`IMAGE_TAG` no està definit i els defaults donen `:latest` i `version: "dev"`.

**El contrast que fa que això sigui la idea 2:**

| | lockdebian | hellogh |
|---|---|---|
| Què viatja | el **codi font** | la **imatge ja construïda** |
| On es construeix | **dins del contenidor destí** (`make docker-build`) | al runner, una sola vegada |
| Com hi arriba | `cicd-router.sh` fa SSH i empeny el clon | cap connexió directa; totes dues surten cap a ghcr.io |
| El que corre és el que s'ha provat? | **probablement** — es reconstrueix | **garantit** — mateix digest |

A lockdebian el build passa dues vegades (a `validate.sh` i altre cop a `deploy.sh`), en una màquina
que mentrestant ha derivat. Aquí, l'artefacte que ha passat el CI és **exactament** el que corre.

> **Dos detalls que fan mal si no es preveuen.**
> 1. **Visibilitat del paquet a GHCR.** Les fonts es contradiuen sobre si un paquet publicat amb
>    `GITHUB_TOKEN` des d'un repo públic surt públic o privat (la doc diu que per defecte és
>    privat, i alhora que hereta del repositori). Cal **comprovar-ho després del primer push** i,
>    si cal, canviar-ho a *Package settings → Change visibility* — **operació irreversible**.
>    L'exercici C necessita que sigui públic. El workflow ha de fer `docker login ghcr.io` al job
>    de deploy igualment (amb `packages: read`), perquè funcioni en tots dos casos.
> 2. **Minúscules.** GHCR rebutja les majúscules a la ruta i `${{ github.repository }}` porta el
>    teu usuari tal qual. Millor escriure la ruta en minúscules a mà, al workflow i al `compose.yaml`.

### `deploy.yml`

- `on: push: tags: ["v*"]` + `workflow_dispatch` (amb `inputs.tag`)
- `concurrency: {group: deploy, cancel-in-progress: false}`
- **job `build`** — `runs-on: ubuntu-24.04`:
  - `permissions: {contents: read, packages: write, id-token: write, attestations: write}`
    (els dos últims, per a les atestacions)
  - `docker/login-action` → `docker/metadata-action` (calcula tags i labels OCI) →
    `docker/build-push-action` amb `target: production`, `cache-from`/`cache-to: type=gha` (§10)
  - **`actions/attest-build-provenance`** amb `subject-name: ghcr.io/jgregor5/hellogh` (**sense
    tag**) i `subject-digest: ${{ steps.build.outputs.digest }}`
- **job `deploy`** — `needs: build`, **`runs-on: [self-hosted, hellogh]`**,
  `permissions: {contents: read, packages: read}`,
  `environment: {name: production, url: https://<domini-app>}`. Passos **locals**:
  `docker login ghcr.io` amb `GITHUB_TOKEN` → escriure `IMAGE_TAG` a `.env` → `docker compose pull`
  → `docker compose up -d --remove-orphans`. Cap SSH, cap secret de desplegament.
- **smoke test** final: 6 reintents × 5 s sobre `/health` verificant que `version` és el tag. És el
  `post-deploy.sh` de lockdebian, mateixa lògica.

> **Si el smoke test falla, el contenidor es queda amb la versió nova en marxa.** No hi ha rollback
> automàtic, i és **a posta**: `undeploy` està fora d'abast (0.5) i un rollback automàtic amagaria
> precisament el que la classe ha de veure. El remei és manual i és una demostració per si sol —
> tornar a desplegar el tag bo:
> ```shell
> gh workflow run deploy.yml -f tag=v1.0.0     # el `redeploy` de lockdebian
> ```
> Val la pena dir-ho a classe: **el pipeline detecta la fallada, no la repara.** Distingir «el CI
> t'avisa» de «el CI t'arregla» és part de la lliçó.

> **L'atestació és la idea 2, però criptogràfica.** `gh attestation verify` demostra que aquella
> imatge exacta (per digest) la va construir aquell repositori, en aquell workflow, en aquell
> commit. A lockdebian això no es pot ni formular: la imatge la construeix el contenidor destí i no
> hi ha cap manera de lligar-la a res. **Val la pena convertir-ho en un cinquè exercici del
> `LAB.md`**, amb una sola comanda:
> `gh attestation verify oci://ghcr.io/jgregor5/hellogh:v1.0.0 --owner jgregor5`

### Conformitat amb la pràctica del sector (comprovat el 2026-08-06)

#### Versions de les accions — **les del `GITHUB.md` estan obsoletes**

`GITHUB.md` es va escriure amb les versions d'abans. Aquestes són les majors vigents:

| Acció | `GITHUB.md` diu | Vigent (ago. 2026) |
|---|---|---|
| `actions/checkout` | `@v4` | **v7** (v7.0.1) |
| `docker/build-push-action` | `@v6` | **v7** (v7.3.0) |
| `docker/login-action` | `@v3` | **v4** (v4.6.0) |

> ⚠️ **Des del 2 de juny de 2026 les accions s'executen amb Node.js 24 per defecte.** Les majors
> antigues que encara van amb Node 20 poden fallar. Un motiu més per no copiar el YAML de
> `GITHUB.md` tal qual. **Comproveu les versions el dia que ho escriviu**: canvien sovint.

#### Fixar les accions per SHA

És **la** pràctica del sector des dels incidents de la cadena de subministrament del Q1 2026:
`tj-actions/changed-files` (23.000+ repositoris compromesos), Trivy i Nx, tots pel mateix vector —
una etiqueta mutable que l'atacant reescriu. Un `@v7` d'avui pot ser codi diferent demà.

```yaml
- uses: actions/checkout@<sha-complet-40-hex>   # v7.0.1
```

L'etiqueta va al comentari: es manté llegible per als alumnes i el que s'executa és immutable.
Afegir `.github/dependabot.yml` amb `package-ecosystem: github-actions` perquè els SHA es
mantinguin sols.

> **Aquesta és una bona història de classe**, no només una casella: explica per què `@v4` semblava
> segur i no ho era, i per què GitHub ja permet forçar el SHA-pinning per política d'organització i
> està construint una secció `dependencies:` a l'estil `go.sum`.

#### Fixar la imatge del runner

`runs-on: ubuntu-latest` migra sol: ara apunta a Ubuntu 24.04 i **26.04 ja està en public preview**,
o sigui que la migració arribarà. La pràctica és fixar-la: **`runs-on: ubuntu-24.04`**. (No afecta
l'exercici B: el que hi importa és que la VM sigui **nova**, no quina versió és.)

#### Atestacions de procedència

Publicar una imatge sense procedència ja no es considera acceptable. `actions/attest-build-provenance`
firma amb Sigstore (instància pública per a repos públics) i lliga imatge↔repositori↔workflow↔commit.
Vegeu el job `build` a `deploy.yml`.

> `docker/build-push-action` també sap generar provenance i SBOM cap al **registre** (`provenance:`,
> `sbom:`). És complementari, no el mateix: allò viu a GHCR, l'atestació de GitHub viu a l'API
> d'atestacions i es verifica amb `gh attestation verify`.

#### On aquest disseny **se separa** de la pràctica, a posta

La guia estàndard diu dues coses que aquest projecte no compleix. Cal dir-ho a classe, no amagar-ho:

| Guia del sector | Què fem aquí | Per què |
|---|---|---|
| «**Mai** runners self-hosted en repositoris públics» | Un runner self-hosted en un repo públic | Mitigat **estructuralment**: `deploy.yml` és l'únic workflow amb `runs-on: self-hosted` i **no es pot disparar des d'un fork** (només `push: tags` i `workflow_dispatch`). El `ci.yml`, que sí que corre en PRs, va a `ubuntu-24.04`. Vegeu «Seguretat» |
| «Els runners han de ser **efímers**» (ARC, JIT) | Un runner **persistent** | És **el material didàctic**: sense estat persistent no hi ha exercici B. Un runner efímer real necessita ARC o tokens JIT, i tots dos exigeixen automatització nova — exclosa per la restricció «cap script» |

> Formulació honesta per a `COMPARISON.md`: *«En producció, això aniria amb runners efímers (ARC) i
> el repositori seria privat. Aquí el runner és persistent i el repositori públic perquè volem
> **veure** l'estat que s'acumula — és exactament el que l'exercici B demostra. Sabeu quin és el
> compromís i per què l'hem triat.»*

#### El que aquest disseny ja fa bé

- **Cap secret de desplegament.** No hi ha `DEPLOY_SSH_KEY` ni clau de llarga durada: el runner ja
  és a dins. Molts equips reals encara guarden una clau SSH com a secret — això és millor.
  > **Matís honest, per no vendre-ho més net del que és.** «Cap secret» vol dir *cap credencial de
  > llarga durada que tu hagis de custodiar*. Al contenidor hi queden igualment dues coses al disc:
  > `.credentials_rsaparams` (la identitat del runner) i, després del primer desplegament,
  > `~cicd/.docker/config.json` amb el token de `docker login`. La diferència real amb una clau SSH
  > de desplegament és que **totes dues són efímeres i revocables des de la interfície de GitHub**,
  > i cap de les dues obre res cap endins. Però dir «zero credencials al disc» seria fals.
- `permissions:` explícit i mínim a cada job.
- `concurrency` al desplegament, perquè dos tags no se solapin.
- `timeout-minutes` explícit.
- Environment amb revisor obligatori i restricció a tags `v*`.

### `LAB.md` — el deliverable pedagògic

Cinc exercicis, cadascun una fallada preparada que **un sistema atrapa i l'altre no**:

| # | Exercici | Què passa | Idea |
|---|---|---|---|
| **A** | PR amb un test trencat | El botó de merge es queda **gris**. A lockdebian aquell codi ja seria a `master` i el pipeline només ho *informaria* | 1 |
| **B** | Un job que depèn d'una cosa que va quedar a la màquina | **Passa al runner self-hosted i peta a `ubuntu-24.04`** — mateix workflow, dos runners, resultat oposat | 3 |
| **C** | `git tag -a v1.0.0` → `docker pull ghcr.io/jgregor5/hellogh:v1.0.0` des d'una altra màquina | Corre **idèntic** a producció; lockdebian reconstrueix al destí | 2 |
| **D** | Un alumne empeny el tag, **un altre** l'aprova | El job es queda **en pausa** | 4 |
| **E** | `gh attestation verify oci://ghcr.io/jgregor5/hellogh:v1.0.0 --owner jgregor5` | Prova **criptogràfica** de qui va construir la imatge, des d'on i amb quin commit. A lockdebian ni tan sols es pot formular la pregunta | 2 |

L'exercici B és el que ningú ensenya i tothom aprèn a cops; aquí es demostra amb una sola
diferència de `runs-on`. A i B els fa **cada alumne al seu repo** (cost zero); C i D es demostren
**una vegada**, en directe, sobre una URL real amb certificat real.

#### L'exercici B, concretat

La resta d'exercicis són una comanda; aquest necessita un workflow propi, perquè el que ha de
demostrar és que **l'estat sobreviu entre execucions diferents** — no dins d'una sola execució.
Aquesta és la part que es fa malament quan s'improvisa: si les dues passes van al mateix job,
passa als dos runners i no es demostra res.

`.github/workflows/lab-b.yml`, només `workflow_dispatch`:

```yaml
on:
  workflow_dispatch:
    inputs:
      runner:   { type: choice, options: [ubuntu-24.04, self-hosted], default: ubuntu-24.04 }
      sembrar:  { type: boolean, default: false }   # deixa l'estat a la màquina
```

Un sol job amb `runs-on: ${{ inputs.runner }}` i dos passos:

```yaml
- name: Sembrar estat (només si es demana)
  if: inputs.sembrar
  run: mkdir -p ~/.hellogh-lab && date -Is > ~/.hellogh-lab/estat.txt

- name: Fer servir l'estat (sempre)
  run: cat ~/.hellogh-lab/estat.txt      # ← aquí peta si la màquina és nova
```

**El guió de classe, tres execucions:**

| # | `runner` | `sembrar` | Resultat | Per què |
|---|---|---|---|---|
| 1 | `self-hosted` | ✅ | ✅ passa | acaba de crear el fitxer |
| 2 | `self-hosted` | ❌ | ✅ **passa** | el `$HOME` de `cicd` continua sent el mateix |
| 3 | `ubuntu-24.04` | ❌ | ❌ **peta** | VM nova: `cat: No such file or directory` |

L'execució **2 és la lliçó**: un job que passa i no hauria de passar. El workflow és idèntic; el
que canvia és si la màquina recorda. La 3 és el mateix workflow dient la veritat.

> **El mateix passa amb la cau de Docker**, i és més realista que un fitxer a `$HOME`: al contenidor
> `docker build` reaprofita capes d'execucions anteriors, i a `ubuntu-24.04` no. Un `Dockerfile` que
> depèn d'una capa antiga construeix aquí i falla allà. El fitxer és millor per a la demostració
> perquè és determinista; la cau de Docker és millor per explicar per què això passa **de veritat**.

Per netejar entre classes: `incus exec hellogh -- rm -rf /home/cicd/.hellogh-lab`.

### `COMPARISON.md`
`GITHUB.md` va de GitHub cap a lockdebian; aquest fa el camí invers, des del projecte real.

1. **Correspondències**: `cicd-trigger.sh` ↔ bloc `on:`; `cicd-router.sh` ↔ GitHub;
   **`cicd-worker.sh` ↔ `Runner.Worker`**; `validate.sh` ↔ `ci.yml`; `deploy.sh`+`post-deploy.sh` ↔
   `deploy.yml`; branca `production` ↔ tag + aprovació; `$CICD_SECRET_FILE` ↔ secrets
   d'environment; verb `redeploy` ↔ `workflow_dispatch`; verbs `logs`/`follow` ↔ pestanya Actions.
   > La correspondència `cicd-worker.sh` ↔ `Runner.Worker` és **la més precisa del projecte** i
   > mereix desenvolupar-se, no despatxar-se en una fila de taula: reprodueix aquí la subsecció
   > **«Com funciona l'agent per dins»** (diagrama Listener/Worker, `_work` persistent i la taula
   > de contrast), de la Part 1 d'aquest pla.
2. **Contracte**: 180 s per hook ↔ 360 min per job; sense python3/pip/sudo ↔ tot disponible; hooks
   seqüencials ↔ jobs paral·lels amb `needs:`.
3. **Sense equivalent a GitHub**: `undeploy`, quota de disc, `usage`.
4. **Sense equivalent a lockdebian**: pull requests com a porta, marketplace, registre d'imatges.
5. **El que NO canvia**: Incus, frpc, frps i Caddy, idèntics. Els **dos túnels sortints** de costat.
   I `/opt/cicd/cicd-worker.sh`, present i mai executat.

`README.md` curt: què és, el cicle en 4 comandes, i com córrer-ho en local amb `make`.

---

## Part 2 — Configuració a lockdebian (només inventari)

### `inventories/remote.yaml` — dues entrades, res més

```yaml
lxc_machines:
  hellogh:                     # SENSE `user:` → contenidor CI/CD → rep Docker
    profile: prjref-project    # reutilitza un perfil existent: cap `lxc_profiles` nou
    image: images:debian/13
    ssh_port: 2203

lxc_frpc_configs:
  hellogh:
    - { name: hellogh, local_port: 8080, domain: <domini-app> }
```

**Cap perfil nou.** Els perfils es comparteixen entre màquines (a `local.yaml`, `group01` i
`group02` fan servir tots dos `devops-project`), així que `prjref-project` (2 CPU / 4GB / 10GB)
serveix tal qual. Com que no es declara cap quota nova, l'única comprovació prèvia és que hi hagi
~10GB lliures al pool (`incus storage info`) — no cal tocar `lxc_pool_size`.

> ⚠️ **`lxc_frpc_configs` NO és opcional aquí, encara que el README §11 digui que sí.**
> [`frpc.yaml:50`](../lockdebian/playbooks/frpc.yaml) fa
> `frp_proxies: "{{ lxc_frpc_configs[item.key] }}"` iterant sobre **totes** les entrades de
> `lxc_machines`, sense `default()`. Una màquina sense entrada frpc no se salta: **peta la tasca**.
> Com que `frpc.yaml` s'ha d'executar igualment per als altres contenidors, afegir `hellogh` només
> a `lxc_machines` trencaria la següent execució per a tothom. (Divergència doc/codi que potser val
> la pena corregir al README, però fora de l'abast d'aquest projecte.)

### `gitodebian`: **res**

`proj_groups` i `cicd_repos`
([`remote.yaml:45,55`](../gitodebian/inventories/remote.yaml)) són el que
registra un repositori al pipeline gitolite→lockdebian. El codi de hellogh viu a GitHub, així que
no hi ha res a registrar. **Aquesta absència és el disseny**: és exactament per això que
`cicd-worker.sh` es quedarà al contenidor sense executar-se mai.

### Playbooks a executar (existents, sense modificar)
```shell
ansible-playbook playbooks/lxcsetup-containers.yaml -i inventories/remote.yaml -i inventories/common.yaml --ask-become-pass
ansible-playbook playbooks/frpc.yaml -i inventories/remote.yaml -i inventories/common.yaml -i inventories/secrets.yaml --vault-password-file <fitxer-de-vault> --ask-become-pass
```

### El que NO es toca
Cap fitxer de `playbooks/` ni de `conf/`, cap `lxc_profiles`, el VPS, gitolite, tot gitodebian, i
tots els scripts `cicd-*`. `lxcsetup-cicd.yaml` es pot executar sense problema (els `when` filtren).

---

## Passos d'execució

Sis fases. Cada una té una **porta de sortida** verificable: no es passa a la següent sense
haver-la superat. La columna «Qui» importa perquè hi ha passos que **no puc fer jo**: els playbooks
demanen contrasenya de `become` i el vault, i la configuració de GitHub és per web.

| Fase | Què | Qui | Porta de sortida |
|---|---|---|---|
| **0** | Escriure el projecte i verificar-lo en local | jo | `make docker-test` verd |
| **1** | Publicar a GitHub | jo (amb la teva autorització) | `ci.yml` verd a `master` |
| **2** | Contenidor i xarxa | **tu** | el contenidor respon i el domini resol |
| **3** | Runner self-hosted | **tu** (jo preparo les comandes) | el runner surt **Idle** |
| **4** | Configuració de GitHub | **tu** (web) | environment i regla de branca actius |
| **5** | Primer desplegament real | tots dos | `/health` retorna `v1.0.0` |
| **6** | Exercicis de classe | tu | els 5 exercicis reproduïbles |

> **La dependència que no és òbvia i que ordena tot això:** la regla de branca de la Fase 4 exigeix
> el check `validate`, i **GitHub només ofereix checks que ja ha vist córrer**. Per tant la Fase 1
> (que dispara `ci.yml` per primer cop) ha d'anar **abans** de la Fase 4. Configurar la protecció
> abans de publicar no només és inútil: bloqueja el push inicial.

### Fase 0 — Escriure el projecte (jo)

1. **Abans dels workflows**, resoldre els pendents 0.6.4 i 0.6.5: agafar el **SHA complet** de
   `actions/checkout` v7, `docker/build-push-action` v7, `docker/login-action` v4 i
   `actions/attest-build-provenance`; i confirmar que `deploy.yml` llegeix el domini de
   `${{ vars.APP_DOMAIN }}` i no d'un literal.
2. **Escriure `hellogh/`** sencer, en aquest ordre (cada bloc depèn de l'anterior):
   `app/` + `tests/` → `Dockerfile` + `compose.yaml` + `Makefile` + `requirements.txt` →
   `.github/workflows/` + `dependabot.yml` → `README.md`, `COMPARISON.md`, `LAB.md`.
   > **`SETUP.md` es deixa per a la Fase 3.** Documenta la instal·lació del runner, i la versió
   > real no es coneix fins que s'instal·la. Escriure'l ara obliga a corregir-lo després.
3. **Cap dada d'infraestructura al repositori.** Ni IPs, ni hosts interns, ni usuaris de sistema,
   ni ports, ni el domini literal. Els valors reals viuen a `INFRA.local.md` (gitignorat).
4. **Verificar en local**: `make docker-test` · `make docker-up && make health && make docker-down`.

**Porta:** els tests passen dins de Docker i `/health` respon `version: "dev"` en local.

### Fase 1 — Publicar (jo, amb la teva autorització)

```shell
git push -u origin master
```

> - Acció cap enfora: **demano confirmació explícita** abans d'executar-la.
> - El repositori a GitHub és **buit** i encara no té branca per defecte: la primera branca que
>   s'hi pugi ho serà. Pujant `master` primer, ja queda bé. Per forçar-ho:
>   `gh repo edit jgregor5/hellogh --default-branch master`.
> - Aquest push **dispara `ci.yml`** (`push: branches: [master]`). És la primera execució, i és la
>   que fa que el check `validate` existeixi per a la Fase 4.

**Porta:** `gh run watch` acaba en verd. Sense això, no continuïs: la Fase 4 no es pot configurar.

### Fase 2 — Contenidor i xarxa (tu)

1. `incus storage info` des de `cicdies` — confirmar ~10GB lliures (pendent 0.6.1).
2. Afegir les **dues entrades** a `lockdebian/inventories/remote.yaml` (vegeu Part 2). Recorda que
   **les dues**: només `lxc_machines` trencaria `frpc.yaml` per a tothom (0.4.3).
3. Executar els dos playbooks de la Part 2 — demanen `--ask-become-pass` i el fitxer del vault.

**Porta:** `incus list hellogh` el mostra en marxa, `incus exec hellogh -- docker version` funciona,
i `<domini-app>` resol i dona 502 (encara no hi ha app: és el resultat correcte en aquest punt).

### Fase 3 — Runner self-hosted (tu; jo preparo les comandes)

Instal·lació al contenidor, **executada un sol cop**:
```shell
# 0) Consultar l'última versió a https://github.com/actions/runner/releases
RUNNER_VERSION=2.XXX.Y            # substituir per la versió real, SENSE la "v"
TOKEN=$(gh api -X POST repos/jgregor5/hellogh/actions/runners/registration-token -q .token)

# 1) Descarregar i extreure dins del contenidor, com a usuari cicd
incus exec hellogh -- su - cicd -c "
  mkdir -p ~/actions-runner && cd ~/actions-runner &&
  curl -fsSL -o runner.tar.gz \
    https://github.com/actions/runner/releases/download/v\$RUNNER_VERSION/actions-runner-linux-x64-\$RUNNER_VERSION.tar.gz &&
  tar xzf runner.tar.gz && rm runner.tar.gz"

# 2) Dependències natives del runner (cal root)
incus exec hellogh -- /home/cicd/actions-runner/bin/installdependencies.sh

# 3) Registrar-lo amb l'etiqueta que fa servir deploy.yml
incus exec hellogh -- su - cicd -c "cd ~/actions-runner && ./config.sh \
    --url https://github.com/jgregor5/hellogh --token $TOKEN \
    --labels hellogh --unattended"

# 4) Instal·lar-lo com a servei systemd (cal root; s'executarà com a cicd)
incus exec hellogh -- bash -c "cd /home/cicd/actions-runner && ./svc.sh install cicd && ./svc.sh start"
```
> - El **token de registre caduca en 1 hora**; si el `config.sh` falla per això, torna a generar-lo.
> - `RUNNER_VERSION` s'ha d'expandir dins del `su -c`, per això va escapat (`\$`).
> - L'usuari `cicd` **ja és al grup `docker`** (el crea `lxcsetup-containers.yaml`), així que el
>   runner pot fer `docker compose` sense `sudo`.
> - Els passos 2 i 4 van com a **root**, per això no passen per `su - cicd`.
> - Idempotència: si `~cicd/actions-runner/.runner` ja existeix, el runner ja està registrat i
>   `config.sh` fallarà. Per tornar-lo a registrar cal `./config.sh remove --token <nou>` primer.

Un cop instal·lat, **escriure `SETUP.md`** amb la versió real que s'ha fet servir (vegeu Fase 0,
on s'ajorna a posta) i fer-ne commit.

**Porta:** a *Settings → Actions → Runners* el runner surt **Idle**.

### Fase 4 — Configuració de GitHub (tu, per web)

La CLI no ho fa tot; aquests tres passos són de la interfície web.

1. **Variable del domini** — això sí que es pot per CLI:
   ```shell
   gh variable set APP_DOMAIN --body '<domini-app>'   # el valor real, de INFRA.local.md
   ```
2. **Environment** — *Settings → Environments → New environment → `production`*:
   *Required reviewers* (tu mateix serveix) i *Deployment branches and tags* = només `v*`.
3. **Regla de branca** — *Settings → Rules → New ruleset* sobre `master`: exigir pull request i
   el check `validate`.
   > ⚠️ **El check `validate` només apareix a la llista si ja ha corregut alguna vegada.** Per això
   > aquesta fase va després de la Fase 1. Si no el trobes, és que encara no has publicat.

**Porta:** un push directe a `master` és rebutjat, i un tag `v*` s'atura esperant aprovació.

### Fase 5 — Primer desplegament real (tots dos)

| # | Acció | Resultat esperat |
|---|---|---|
| 1 | `git tag -a v1.0.0 -m "…"` · `git push origin v1.0.0` | `deploy.yml` arrenca; el job `build` puja la imatge a GHCR |
| 2 | **Comprovar la visibilitat del paquet a GHCR** (pendent 0.6.2) | si surt privat, *Package settings → Change visibility → Public*. **Irreversible** — i l'exercici C el necessita públic |
| 3 | Aprovar el desplegament des de la web | el job `deploy` s'engega al runner del contenidor |
| 4 | `curl https://<domini-app>/health` | `{"status":"healthy","version":"v1.0.0"}`, **per frpc → frps → Caddy, sense tocar el VPS** |

**Porta:** `/health` retorna exactament el tag publicat. Aquí el pipeline ja és real de punta a punta.

### Fase 6 — Exercicis de classe (tu)

Els cinc exercicis del `LAB.md`, ara que tota la maquinària funciona:

| # | Exercici | Comprovació |
|---|---|---|
| **A** | PR amb un test trencat | el check falla i **bloqueja el botó de merge** |
| **B** | Job que depèn d'estat deixat a la màquina | **passa a `[self-hosted, hellogh]` i peta a `ubuntu-24.04`** — mateix workflow, dos runners |
| **C** | `docker pull ghcr.io/jgregor5/hellogh:v1.0.0` des d'una altra màquina, sense login | imatge pública idèntica a producció |
| **D** | Un empeny el tag, **un altre** l'aprova | el job es queda **en pausa** |
| **E** | `gh attestation verify oci://ghcr.io/jgregor5/hellogh:v1.0.0 --owner jgregor5` | procedència signada: repositori, workflow i commit |

I les tres comprovacions que fan visible el disseny:

| Comanda | Demostra |
|---|---|
| `incus exec hellogh -- systemctl status frpc actions.runner.*` | **els dos túnels sortints de costat**, cap port obert |
| `incus exec hellogh -- ls -l /opt/cicd/cicd-worker.sh` + `cat /var/log/cicd-worker.log` | hi és i **no s'ha executat mai**: la peça exacta que GitHub substitueix |
| `gh workflow run deploy.yml -f tag=v1.0.0` · `gh run watch` · `gh run view --log` | `redeploy`, `follow` i `logs` de lockdebian (§6, §13) |
