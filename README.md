# TimeTagger on Pergola

> [!TIP]
> ### ❤️ Support the Developer
> If you find TimeTagger useful, please consider donating to the original creator, Almar Klein, to support his open-source work!
> You can contribute here: **[Donate to Almar Klein](https://almarklein.org/donate.html)**

---

## About TimeTagger

[TimeTagger](https://timetagger.app) is an elegant, open-source, and privacy-focused time tracker designed to help you understand where your time goes. It features:
- A clean, modern, and fluid user interface.
- Rich interactive charts and reports.
- Full keyboard-driven operation for high productivity.
- An open API to integrate with your existing workflows.

Unlike other time tracking software, TimeTagger is lightweight and respects your data privacy. By hosting your own instance, you retain full ownership and control over your time-tracking history.

---

## Hosting on Pergola

This repository is optimized for deploying your own self-hosted TimeTagger instance on the [Pergola platform](https://pergola.cloud), a container-based cloud designed for high-availability deployments.

### Prerequisites

1. A Pergola account with the CLI installed and configured (`pergola login`).
2. An existing project created in Pergola.

### Configuration (`pergola.yaml`)

We use a declarative `pergola.yaml` file to define our stack structure:

```yaml
version: v1
components:
  - name: timetagger
    docker:
      image: ghcr.io/almarklein/timetagger
    ports:
      - 80
    ingresses:
      - host: timetagger
        port: 80
        path: /timetagger/app/
    storage:
      - name: timetagger-data
        path: /root/_timetagger
        size: 2Gi
    env:
      - name: TIMETAGGER_BIND
        value: 0.0.0.0:80
      - name: TIMETAGGER_DATADIR
        value: /root/_timetagger
      - name: TIMETAGGER_LOG_LEVEL
        value: info
      - name: TIMETAGGER_CREDENTIALS
        config-ref: TIMETAGGER_CREDENTIALS
```

### Steps to Deploy

#### 1. Set Up Stage Configuration (Secrets)

Before releasing, you need to add your user credentials to the stage configuration so they are injected safely via `config-ref`.

Use a bcrypt password hashing tool to generate your hashed credentials (e.g. at [timetagger.app/cred](https://timetagger.app/cred)). Once you have your credentials format `username:hash`, run the following command to register them securely:

```bash
pergola add config-data default --env TIMETAGGER_CREDENTIALS="your_username:your_bcrypt_hash" -p pergola-timetagger -s dev
```

#### 2. Push Build

To compile and package the application build, push the repository to Pergola:

```bash
pergola push build -p pergola-timetagger
```

This registers a new build (e.g., `main_b5`).

#### 3. Trigger Deployment

To deploy your build and configuration to your desired stage:

```bash
pergola push release -p pergola-timetagger -s dev --build main_b5 --config default
```

#### 4. Access the Application

Once deployed, the ingress makes TimeTagger available at your public domain prefix on the path `/timetagger/app/`. You can view deployment status and public URLs with:

```bash
pergola list components -p pergola-timetagger -s dev
```
