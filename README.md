# `cargo-cache` Action

This GitHub Action caches Rust Cargo build files to speed up your CI workflows.

## Example workflow

```yaml
on:
  pull_request:

jobs:
  cargo-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      # Install Rust and Cargo
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
          override: true

      # Cache the Cargo build files
      # This action must be used *after* `actions-rs/toolchain`!
      - uses: Leafwing-Studios/cargo-cache@v1

      # Do stuff with Cargo
      - name: Run cargo check
        run: cargo check
```

## Inputs

| Name          | Description                                                                                                                  | Type     | Default             |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------- |
| `cache-group` | The group of the cache, defaults to the job ID. If you want two jobs to share the same cache, give them the same group name. | `string` | `${{ github.job }}` |

## Outputs

| Name        | Description                                                          | Type      | Note                                            |
| ----------- | -------------------------------------------------------------------- | --------- | ----------------------------------------------- |
| `cache-hit` | A boolean value to indicate if an exact match was found for the key. | `boolean` | Passed through from the `actions/cache` action. |

## How It Works

This action caches the `target` directory, as well the Cargo registry.
Under the hood, this action uses the `actions/cache` action to do the actual caching.
The cache key is generated from the `Cargo.toml` and `Cargo.lock` files, to ensure that the cache is always up-to-date.
When the `Cargo.toml` or `Cargo.lock` files change, the cache falls back to previously cached versions to not start from scratch.

When the `Cargo.lock` file was not commited to the repository (this is the case for libraries), it is first generated via `cargo update`.
This is why the `actions-rs/toolchain` action must be used _before_ this action.
For more information on why the `Cargo.toml` files don't specify the dependencies completely, please consult [the Cargo Book](https://doc.rust-lang.org/cargo/guide/cargo-toml-vs-cargo-lock.html).
