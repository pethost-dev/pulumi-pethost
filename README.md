# Pulumi provider for Pethost: deploy Docker Compose projects and their databases as code

The [Pethost](https://pethost.dev) provider for [Pulumi](https://www.pulumi.com) lets a Pulumi
program deploy and manage Pethost's projects and databases: a Docker Compose
project from its files or from a GitHub repository, with its HTTPS address, and the Postgres and
Valkey databases its apps use. It is the package `pethost` of the Pulumi Registry, in
TypeScript, Python, Go, C#, Java and YAML.

## Install

In a Pulumi project:

```bash
pulumi package add pethost --server github://api.github.com/pethost-dev/pulumi-pethost
```

It generates the SDK for the project's language, records the package in `Pulumi.yaml` and prints
the import line. Once the package is in the Registry, `pulumi package add pethost` is
enough.

## A first program

A page of nginx's demo image, deployed from its `compose.yaml`, in Pulumi YAML
([examples/yaml/Pulumi.yaml](examples/yaml/Pulumi.yaml)):

```yaml
name: pethost-example
runtime: yaml
description: A page on Pethost, deployed from its compose.yaml.

resources:
  hello:
    type: pethost:Project
    properties:
      projectId: hello
      files:
        - path: /compose.yaml
          text: |
            services:
              web:
                image: nginxdemos/hello:plain-text
                ports: ["80"]

outputs:
  url: ${hello.url}
```

```bash
pulumi config set --secret pethost:token pth_…
pulumi up
```

The port it publishes gets the project's HTTPS address at every deploy, which `url` shows. A
change of the file deploys it again; `pulumi destroy` deletes it, after a final backup while
backups are on.

## Resources

| Resource | What it is |
|---|---|
| `pethost:index:Project` | Creates a project from its files and deploys it. |
| `pethost:index:Database` | A database project, Postgres, MariaDB (for an app that asks for MySQL or MariaDB) or Valkey: the machine writes its compose.yaml and /.env and deploys it, with no address. |

Each resource's documentation says which of its inputs a change sends as a deploy and which
replace it, deleting it first.

## Functions

| Function | What it reads |
|---|---|
| `pethost:index:getMachine` | Start here: the machine (resources, the person's domain, backups, SSH), every project with its problems and URL, deleted projects with snapshots, GitHub repositories to create projects from. |
| `pethost:index:getProject` | One project in full: services (state, health, image, the ports it listens on, volumes, environment, the SSH command), volumes, routes, hosts with certificates, recent operations, snapshots, the last day's HTTP traffic. |
| `pethost:index:listCommits` | A github project's branch commits that change its directory, newest first, 30 a page, with whether the files are at it and its last deploy. |
| `pethost:index:getOperation` | An operation, after waiting for it to end (waitSeconds): then the project as it is, and the log's end if it FAILED. |
| `pethost:index:queryHttpTraffic` | Which paths fail and which are slow (topPaths): the HTTP requests that reached the project through the proxy, since oldestKeptTime, each counted as it ends: totals, a time series, top paths and the requests, for one filter and range. |
| `pethost:index:queryContainerLogs` | The containers' stdout and stderr lines, since oldestKeptTime; by default the newest, oldest first in the page. |
| `pethost:index:readPath` | Reads a file (text, 64 KiB a call; ≤1 MiB with lengthBytes) or a directory's entries (~64 KiB a page): of the project's deployed files, or with `service` or `volume`, of a container (whole filesystem; while not running only its volumes, FAILED_PRECONDITION listing them) or a volume. |

## Configuration

- `pethost:token` (secret): The account's API token, `pth_…`: make one in the panel's Settings, under API tokens. Absent = the environment's `PETHOST_TOKEN`.
- `pethost:url`: Where the API is served. Absent = the environment's `PETHOST_URL`, else https://console.pethost.dev.

Make the token in [the panel](https://console.pethost.dev), in Settings under "API tokens".

## Generated from the API's description

Everything here is generated whole from Pethost's API description,
[panel.proto](https://github.com/pethost-dev/sdk/blob/main/proto/v1/panel.proto), as
Pethost's SDKs are: the schema, the provider, its documentation (the proto's comments) and
this page. The proto says which RPCs are a resource's steps; a field added to a create request is
a new input, with nothing written by hand. A new release, `v<version>`, follows each change of
the API; its workflow builds the plugin for Linux, macOS and Windows. Issues are welcome here; a
change is made in Pethost's own repository, which this one is copied from.

## Docker Compose hosting at one flat price

Pethost is managed hosting for Docker Compose projects. A plan has a set size in CPU, memory and
disk, at one flat price, for as many projects as fit. Pethost's software deploys them, gives
them HTTPS addresses, keeps their request logs and backs them up every night. You run it through your
agent or from a web panel: both show the same projects, logs and requests.

- **Always on:** no cold starts, nothing sleeps, on every plan. Security updates install
  themselves.
- **HTTPS addresses:** anything.yourname.pethost.app at once, or your own domain with one DNS
  record. Certificates renew themselves. A password page in front of the whole site: one
  sentence to your agent.
- **Deploys from GitHub:** every push deploys itself, and an earlier commit is one step back.
- **Request logs:** every request with its status and time, the busiest paths and the error
  rate, kept for 30 days.
- **Nightly backups:** encrypted, stored off the machine, kept for 14 days.
- **SSH and SFTP** into any container, and a terminal in the panel.
- **Docker Compose, as written:** a project is a Compose file, and the same file runs anywhere
  else Docker does. A database is one click, Postgres, MariaDB or Valkey, with a page for
  connecting your apps, at no extra cost.
- **In the EU:** your data lives on your machine, and requests to your sites go straight to
  it, never through our panel.

**Pricing.** Starter is €9.99 a month for 2 vCPU, 3 GB of memory and 30 GB of SSD. Grow is
€17.99 for 4 vCPU, 6 GB and 60 GB. Scale is €59 for 8 vCPU, 30 GB and 200 GB. Prices are without
VAT. Whatever the plan, you can try anything: as many projects, containers and databases as
fit, nothing metered, and nothing billed on top. There is no free tier. [See the plans](https://pethost.dev/#pricing).

**Who it is for.** Solo developers and early startups who host several projects, bots or small
apps and want one fixed monthly bill, and people who build with an AI agent and want it to
deploy and run what it wrote. It is not for large teams whose applications need several regions
or autoscaling: a plan's size is the limit.

## How Pethost compares

| If you use | The difference |
|---|---|
| [Heroku](https://pethost.dev/about/#heroku), [Render](https://pethost.dev/about/#render), [DigitalOcean App Platform](https://pethost.dev/about/#digitalocean) | There each app, worker and database is one more line on the bill. On Pethost one flat price covers everything you run: as many projects and databases as fit, and nothing sleeps. |
| [Railway, Fly.io](https://pethost.dev/about/#railway), [Koyeb](https://pethost.dev/about/#koyeb) | There CPU, memory and traffic are metered, so the bill moves with every busy month. On Pethost your plan's size is the limit: nothing is metered, nothing is billed on top. |
| [Vercel, Netlify](https://pethost.dev/about/#vercel) | Made for front ends and serverless functions: a bot, a long-running worker or a database is another service with its own price. Pethost runs the front end, the back end and the database side by side, at one flat price. |
| [Coolify, Dokploy](https://pethost.dev/about/#coolify), [CapRover, Dokku, Easypanel](https://pethost.dev/about/#caprover) | Self-hosted: the server is yours to rent, update, secure and back up. On Pethost the server and the panel come together, both looked after for you, and agents are first-class users, not an add-on. |
| [Sliplane](https://pethost.dev/about/#sliplane) | The closest: a flat price per server. There a Compose file is converted into its own services; Pethost runs a Compose project as written. |
| [Replit](https://pethost.dev/about/#replit) | You build and publish inside Replit, with its own agent. Pethost works with the agent you already use, and one flat price covers all your apps. |
| [PythonAnywhere](https://pethost.dev/about/#pythonanywhere) | Python only, no Docker. Pethost runs anything Docker runs, in any language. |
| [A VPS you set up yourself](https://pethost.dev/about/#vps) | The server is the cheap part. You still set up the proxy, DNS, certificates, backups and updates yourself, and fix them when they break. On Pethost all of that is done. |

[About Pethost](https://pethost.dev/about/) has each comparison in full, with prices.

## Questions

**Which AI agents does Pethost work with?**
Almost any agent that supports skills and MCP. The ones we tested ourselves: Claude Code, Codex,
Cursor, ChatGPT, Gemini CLI, OpenClaw, Hermes and Pi.

**What can I deploy?**
Anything that runs in Docker: web apps, APIs, bots, workers, databases. No Dockerfile? With the
plugin installed, your agent knows the details and writes it for you.

**Do I need an agent?**
No. The web panel does the same, and shows the same projects, logs and requests.

**Can I use my own domain?**
Yes. One DNS record, and the certificate comes by itself.

**Is there a free tier?**
No: plans start at €9.99 a month, and nothing sleeps. Cancel any time:
[refunds and cancellation](https://pethost.dev/refunds/) has the rules.

