# Redis

The `redis-crossplane` ClusterResourceType gives developers a Redis cache. It creates a
`RedisInstance` in the cell namespace, and the backing installed on that data plane builds it.

For installing the module, see the [module README](../../README.md).

## Backings

| Backing | Builds a `RedisInstance` as | Guide |
| :------ | :-------------------------- | :---- |
| Azure | Azure Managed Redis | [azure/README.md](../../azure/README.md#redis) |
| AWS | Amazon ElastiCache for Valkey | [aws/README.md](../../aws/README.md#redis) |

## Per-environment settings

Set on the `ResourceReleaseBinding`, under `resourceTypeEnvironmentConfigs`. The ResourceType has
no developer parameters.

| Setting | Default | Description |
| :------ | :------ | :---------- |
| `size` | `small` | `small`, `medium` or `large`. Each backing maps it to its own compute and memory |

```yaml
spec:
  resourceTypeEnvironmentConfigs:
    size: medium
```

## Outputs

| Output | Kind | Comes from |
| :----- | :--- | :--------- |
| `host` | `value` | `RedisInstance` `status.address` |
| `port` | `value` | `RedisInstance` `status.port` |
| `password` | `secretKeyRef` | Secret `<RedisInstance name>-conn`, key `password` |

## Example

A Resource, from [`samples/resource.yaml`](samples/resource.yaml):

```yaml
apiVersion: openchoreo.dev/v1alpha1
kind: Resource
metadata:
  name: orders-cache
spec:
  owner:
    projectName: shop
  type:
    kind: ClusterResourceType
    name: redis-crossplane
```

A Workload that uses it, from [`samples/workload.yaml`](samples/workload.yaml):

```yaml
spec:
  dependencies:
    resources:
      - ref: orders-cache
        envBindings:
          host: REDIS_HOST
          port: REDIS_PORT
          password: REDIS_PASSWORD
```

## Implementing a backing

A Composition for `RedisInstance` ([`definition.yaml`](definition.yaml)) must, once the cache is
reachable:

- set `status.address` and `status.port`
- write a Secret named `<RedisInstance name>-conn` in the same namespace, with the password under
  the key `password`
- map `spec.size` to the platform's compute and memory

The Azure Composition, [`azure/redis-composition.yaml`](../../azure/redis-composition.yaml), and the
AWS one, [`aws/redis-composition.yaml`](../../aws/redis-composition.yaml), are working examples. The
Azure one composes that Secret, since its provider publishes the password under a different key; the
AWS one has the provider generate the password straight into it.
