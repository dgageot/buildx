# GitHub Actions cache storage

> **Warning**
>
> The GitHub Actions cache is a beta feature. You can use it today, in current
> releases of Buildx and BuildKit. However, the interface and behavior are
> unstable and may change in future releases.

The GitHub Actions cache utilizes the
[GitHub-provided Action's cache](https://github.com/actions/cache) available
from within your CI execution environment. This is the recommended cache to use
inside your GitHub action pipelines, as long as your use case falls within the
[size and usage limits set by GitHub](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy).

> **Note**
>
> This cache storage backend requires using a different driver than the default
> `docker` driver - see more information on selecting a driver
> [here](../drivers/index.md). To create a new driver (which can act as a simple
> drop-in replacement):
>
> ```console
> docker buildx create --use --driver=docker-container
> ```

## Synopsis

```console
$ docker buildx build . --push -t <registry>/<image> \
  --cache-to type=gha[,parameters...] \
  --cache-from type=gha[,parameters...]
```

Common parameters:

- `url`: cache server URL (default `$ACTIONS_CACHE_URL`), see
  [authentication](#authentication)
- `token`: access token (default `$ACTIONS_RUNTIME_TOKEN`), see
  [authentication](#authentication)
- `scope`: cache scope (defaults to the name of the current Git branch).

Parameters for `--cache-to`:

- `mode`: specify cache layers to export (default: `min`), see
  [cache mode](./index.md#cache-mode)

## Authentication

If the `url` or `token` parameters are left unspecified, the `gha` cache backend
will fall back to using environment variables. If you invoke the `docker buildx`
command manually from an inline step, then the variables must be manually
exposed (using
[`crazy-max/ghaction-github-runtime`](https://github.com/crazy-max/ghaction-github-runtime),
for example).

## Scope

By default, cache is scoped per Git branch. This ensures a separate cache
environment for the main branch and each feature branch. If you build multiple
images on the same branch, each build will overwrite the cache of the previous,
leaving only the final cache.

To preserve the cache for multiple builds on the same branch, you can manually
specify a cache scope name using the `scope` parameter. In the following
example, the cache is set to a combination of the branch name and the image
name, to ensure each branch gets its own cache):

```console
$ docker buildx build . --push -t <registry>/<image> \
  --cache-to type=gha,url=...,token=...,scope=$GITHUB_REF_NAME-image
  --cache-from type=gha,url=...,token=...,scope=$GITHUB_REF_NAME-image
$ docker buildx build . --push -t <registry>/<image2> \
  --cache-to type=gha,url=...,token=...,scope=$GITHUB_REF_NAME-image2
  --cache-from type=gha,url=...,token=...,scope=$GITHUB_REF_NAME-image2
```

GitHub's
[cache access restrictions](https://docs.github.com/en/actions/advanced-guides/caching-dependencies-to-speed-up-workflows#restrictions-for-accessing-a-cache),
still apply. Only the cache for the current branch, the base branch and the
default branch is accessible by a workflow.

### Using `docker/build-push-action`

When using the
[`docker/build-push-action`](https://github.com/docker/build-push-action), the
`url` and `token` parameters are automatically populated. No need to manually
specify them, or include any additional workarounds.

For example:

```yaml
- name: Build and push
  uses: docker/build-push-action@v3
  with:
    context: .
    push: true
    tags: "<registry>/<image>:latest"
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

<!-- FIXME: cross-link to ci docs once docs.docker.com has them -->

## Further reading

For an introduction to caching see
[Optimizing builds with cache management](https://docs.docker.com/build/building/cache).

For more information on the `gha` cache backend, see the
[BuildKit README](https://github.com/moby/buildkit#github-actions-cache-experimental).
