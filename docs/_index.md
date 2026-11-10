---
title: Pethost
meta_desc: Use the Pulumi Pethost provider to deploy Docker Compose projects and their Postgres, MariaDB or Valkey databases on Pethost.
layout: package
---

The Pethost provider for Pulumi lets you manage [Pethost](https://pethost.dev/) resources (projects and databases) as part of your Pulumi programs: a Docker Compose project deployed from its files or from a GitHub repository, with its HTTPS address, and the Postgres, MariaDB or Valkey databases its apps use. Pethost is managed hosting for Docker Compose projects: a plan has a set size in CPU, memory and disk, at one flat price, for as many projects as fit.

## Installation

{{< chooser language "typescript,python,go,csharp,java,yaml,hcl" >}}
{{% choosable language typescript %}}

```bash
pulumi package add pethost
```

{{% /choosable %}}
{{% choosable language python %}}

```bash
pulumi package add pethost
```

{{% /choosable %}}
{{% choosable language go %}}

```bash
pulumi package add pethost
```

{{% /choosable %}}
{{% choosable language csharp %}}

```bash
pulumi package add pethost
```

{{% /choosable %}}
{{% choosable language java %}}

```bash
pulumi package add pethost
```

{{% /choosable %}}
{{% choosable language yaml %}}

```bash
pulumi package add pethost
```

{{% /choosable %}}
{{% choosable language hcl %}}

Declare the provider in your program, then run `pulumi install`:

```hcl
terraform {
  required_providers {
    pethost = {
      source  = "pulumi/pethost"
      version = "0.2.0"
    }
  }
}
```

```bash
pulumi install
```

{{% /choosable %}}
{{< /chooser >}}

`pulumi package add` generates the SDK for the language of your project, records the package in `Pulumi.yaml` and prints the import line. The provider's plugin comes from the GitHub releases of [github.com/pethost-dev/pulumi-pethost](https://github.com/pethost-dev/pulumi-pethost).

## Example Usage

A page of nginx's demo image, deployed from its `compose.yaml`. The port it publishes gets the project's HTTPS address at every deploy, and the program exports it.

{{< chooser language "typescript,python,go,csharp,java,yaml" >}}
{{% choosable language typescript %}}

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as pethost from "@pethost-dev/pulumi-pethost";

const hello = new pethost.Project("hello", {
    projectId: "hello",
    files: [{
        path: "/compose.yaml",
        text: `services:
  web:
    image: nginxdemos/hello:plain-text
    ports: ["80"]
`,
    }],
});

export const url = hello.url;
```

{{% /choosable %}}
{{% choosable language python %}}

```python
import pulumi
import pulumi_pethost as pethost

hello = pethost.Project("hello",
    project_id="hello",
    files=[pethost.FileChangeArgs(
        path="/compose.yaml",
        text="""services:
  web:
    image: nginxdemos/hello:plain-text
    ports: ["80"]
""",
    )])

pulumi.export("url", hello.url)
```

{{% /choosable %}}
{{% choosable language go %}}

```go
package main

import (
	"github.com/pethost-dev/pulumi-pethost/sdk/go/pethost"
	"github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
	pulumi.Run(func(ctx *pulumi.Context) error {
		hello, err := pethost.NewProject(ctx, "hello", &pethost.ProjectArgs{
			ProjectId: pulumi.String("hello"),
			Files: pethost.FileChangeArray{
				pethost.FileChangeArgs{
					Path: pulumi.String("/compose.yaml"),
					Text: pulumi.String("services:\n  web:\n    image: nginxdemos/hello:plain-text\n    ports: [\"80\"]\n"),
				},
			},
		})
		if err != nil {
			return err
		}
		ctx.Export("url", hello.Url)
		return nil
	})
}
```

{{% /choosable %}}
{{% choosable language csharp %}}

```csharp
using System.Collections.Generic;
using Pulumi;
using Pethost = PethostDev.Pethost;

return await Deployment.RunAsync(() =>
{
    var hello = new Pethost.Project("hello", new()
    {
        ProjectId = "hello",
        Files =
        {
            new Pethost.Inputs.FileChangeArgs
            {
                Path = "/compose.yaml",
                Text = "services:\n  web:\n    image: nginxdemos/hello:plain-text\n    ports: [\"80\"]\n",
            },
        },
    });

    return new Dictionary<string, object?>
    {
        ["url"] = hello.Url,
    };
});
```

{{% /choosable %}}
{{% choosable language java %}}

```java
package myproject;

import com.pulumi.Pulumi;
import dev.pethost.pethost.Project;
import dev.pethost.pethost.ProjectArgs;
import dev.pethost.pethost.inputs.FileChangeArgs;

public class App {
    public static void main(String[] args) {
        Pulumi.run(ctx -> {
            var hello = new Project("hello", ProjectArgs.builder()
                .projectId("hello")
                .files(FileChangeArgs.builder()
                    .path("/compose.yaml")
                    .text("services:\n  web:\n    image: nginxdemos/hello:plain-text\n    ports: [\"80\"]\n")
                    .build())
                .build());

            ctx.export("url", hello.url());
        });
    }
}
```

{{% /choosable %}}
{{% choosable language yaml %}}

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

{{% /choosable %}}
{{< /chooser >}}

Run it with your account's API token:

```bash
pulumi config set --secret pethost:token pth_…
pulumi up
```

## Configuration

- `token` (Required, Secret) — The account's API token, `pth_…`: make one in the panel's Settings, under API tokens. Absent = the environment's `PETHOST_TOKEN`.
- `workspace` (Optional) — The workspace a resource is made in when its own `workspace` is absent, and that a function acts on when its request names none: its id or name. Absent = the environment's `PETHOST_WORKSPACE`, else the only workspace the token reaches.
- `url` (Optional) — Where the API is served. Absent = the environment's `PETHOST_URL`, else https://console.pethost.dev.

Make the token in [the panel](https://console.pethost.dev), in Settings under "API tokens". The provider calls Pethost's API, whose [reference](https://pethost.dev/docs/api/) describes every call; a resource's own page says which it makes.

## Resources

- `Project`: Creates a project from its files and deploys it.
- `Database`: A database project, Postgres, MariaDB (for an app that asks for MySQL or MariaDB) or Valkey: the machine writes its compose.yaml and /.env and deploys it, with no address.

## Functions

- `listWorkspaces`: The workspaces the caller may reach, by name, 20 a page: each one's id, name, size and what is free, location, hostname and apps domain.
- `getMachine`: Start here: the machine (resources, the person's domain, backups, SSH), every project with its problems and URL, deleted projects with snapshots, GitHub repositories to create projects from, and the workspace it serves.
- `getProject`: One project in full: services (state, health, image, the ports it listens on, volumes, environment, the SSH command), volumes, routes, hosts with certificates, recent operations, snapshots, the last day's HTTP traffic.
- `listCommits`: A github project's branch commits that change its directory, newest first, 30 a page, with whether the files are at it and its last deploy.
- `getOperation`: An operation, after waiting for it to end (waitSeconds): then the project as it is, and the log's end if it FAILED.
- `queryHttpTraffic`: Which paths fail and which are slow (topPaths): the HTTP requests that reached the project through the proxy, since oldestKeptTime, each counted as it ends: totals, a time series, top paths and the requests, for one filter and range.
- `queryContainerLogs`: The containers' stdout and stderr lines, since oldestKeptTime; by default the newest, oldest first in the page.
- `readPath`: Reads a file (text, 64 KiB a call; ≤1 MiB with lengthBytes) or a directory's entries (~64 KiB a page): of the project's deployed files, or with `service` or `volume`, of a container (whole filesystem; while not running only its volumes, FAILED_PRECONDITION listing them) or a volume.
