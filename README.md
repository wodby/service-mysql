# MySQL service for Kubernetes on Wodby

Run MySQL 8 as a reusable, persistent database service with Wodby.

This repository defines the Wodby service manifest and operational contract for MySQL.

- [MySQL service on Wodby](https://wodby.com/services/mysql)
- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Ghost application stack](https://github.com/wodby/stack-ghost)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `mysql` |
| Type | Database |
| Versions | MySQL 8 |
| Workloads | `main` (StatefulSet), primary, one replica |
| Containers | `mysql` using the Wodby image built from the official MySQL image |
| Endpoint | MySQL TCP 3306 |
| Volume | Data, 10 GB by default |
| Operations | Database and user lifecycle, import, and backup |
| Helm | `oci://registry-1.docker.io/wodby/stateful`, version `0.2.0` |

## Data operations

Wodby creates application databases and users through the operations provided by `wodby/mysql`. Database imports accept
SQL files and common compressed archive formats through `/wodby/import`. Backups use consistent snapshots and are
uploaded or streamed as compressed SQL files.

## Maintain a custom version

1. Fork this repository.
2. Edit `service.yml`.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference `mysql` from a stack manifest.

Keep service, workload, container, endpoint, volume, and database-operation names stable unless consuming stacks and
app-level overrides are updated at the same time.

Validate the manifest with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [managed services index](https://github.com/wodby/services) for other Wodby services.
