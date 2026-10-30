# setup-gcp-gradle-build-cache

Gives a job read and write access to the Technoir Lab remote Gradle build cache in Google Cloud Storage.

## Usage

Grant the job the `id-token: write` permission, then add the action after checking out the repository and before running Gradle:

```yaml
permissions:
  contents: read
  id-token: write

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

  - name: Set up GCP Gradle build cache
    uses: technoir-lab/actions/setup-gcp-gradle-build-cache@v1.2.1

  - name: Build with Gradle
    run: ./gradlew build
```

## Outputs

| Output | Description |
| --- | --- |
| `enabled` | Whether the job authenticated to the remote build cache. |

## Environment variables

After authenticating, the action sets these variables for the following steps:

| Variable | Value |
| --- | --- |
| `GCP_BUILD_CACHE_PROJECT_ID` | The Google Cloud project that owns the cache bucket. |
| `GCP_BUILD_CACHE_BUCKET_NAME` | The bucket that stores cache entries. |
| `GCP_BUILD_CACHE_CREDENTIALS` | The path to the credentials file. |
| `GCP_BUILD_CACHE_PUSH` | `true`, so builds store entries in the cache. |

When the action skips authentication, it sets none of them.

## Authentication

The action exchanges the job's GitHub OIDC token for credentials of the `gradle-build-cache` service account through Workload Identity Federation, so no secret is needed. It moves the credentials file into the runner's temporary directory, which the runner empties when the job finishes.

- **Outside the workspace.** The credentials file lets its holder request OIDC tokens as the job, so it stays out of the workspace, where artifact uploads and commits could pick it up.
- **Separate from Application Default Credentials.** The action doesn't set `GOOGLE_APPLICATION_CREDENTIALS` or Google Cloud project variables such as `GOOGLE_CLOUD_PROJECT`, so other Google Cloud tools in the job keep their own identity.
- **Organisation repositories only.** In repositories outside the `technoir-lab` organisation, such as forks running their own workflows, the action skips authentication with a notice, and Gradle builds run with only the local build cache.
- **Trusted events only.** Federation accepts jobs in `technoir-lab` repositories triggered by `push`, `pull_request`, `merge_group`, `workflow_dispatch` or `schedule` events. Authentication fails for other events, such as `pull_request_target` and `issue_comment`.
- **No OIDC token, no cache.** Pull requests from forks, and jobs without the `id-token: write` permission, can't request an OIDC token. The action then skips authentication with a notice, and Gradle builds run with only the local build cache.
