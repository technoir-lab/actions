# setup-kotlin-multiplatform

Sets up a job for building a Kotlin Multiplatform project. Currently, it caches the Kotlin/Native toolchains that the Kotlin Gradle plugin downloads to `~/.konan`.

## Usage

Add the action after checking out the repository and before running Gradle:

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v7.0.1

  - name: Set up JDK
    uses: actions/setup-java@v6.0.1
    with:
      java-version: 25
      distribution: corretto

  - name: Setup Gradle
    uses: gradle/actions/setup-gradle@v6.4.0
    with:
      cache-provider: basic

  - name: Set up KMP
    uses: technoir-lab/actions/setup-kotlin-multiplatform@v1.0.0

  - name: Build with Gradle
    run: ./gradlew build
```

The action doesn't install a JDK or configure Gradle; use `actions/setup-java` and `gradle/actions/setup-gradle` for that.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `cache-read-only` | `true` everywhere except the default branch | Restore the Kotlin/Native toolchain cache without saving it. |

## Outputs

| Output | Description |
| --- | --- |
| `cache-hit` | Whether the Kotlin/Native toolchain cache was restored with an exact key match. |

## Caching

The cache key combines the runner OS, the runner architecture and a hash of `settings.gradle.kts`, where Technoir Lab projects declare the Kotlin Gradle plugin version through convention plugins:

```
konan-<runner.os>-<runner.arch>-<hash of settings.gradle.kts>
```

- **Saved only from the default branch.** Pull requests, merge queues, tags and other refs restore the default branch's cache but never save their own. Kotlin/Native toolchains take 1.5–2 GB per OS, so caches saved from other refs would quickly exceed the repository's cache storage limit.
- **Exact key match only.** The action doesn't fall back to an older cache entry when the key changes. Each entry holds only the toolchains its build used, so outdated Kotlin/Native versions don't build up in the cache. The first build after a key change downloads the toolchains again.
- **Saved at the end of the job.** On the default branch, the cache is saved after all later steps succeed.
