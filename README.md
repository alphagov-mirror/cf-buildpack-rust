# Cloudfoundry buildpack for Rust

> This is a fork of https://github.com/emk/heroku-buildpack-rust, tweaked to work on cloudformation (or at least, our cloudfoundry deployment). 

This is a Heroku buildpack for Rust with support for [cargo][] and [rustup][].  Features include:

- Caching of builds between deployments.
- Automatic updates to the latest stable Rust by default.
- Optional pinning of Rust to a specific version.
- Support for `export` so that other buildpacks can access the Rust toolchain.
- Support for compiling Rust-based extensions for projects written in other languages.

[cargo]: http://crates.io/
[rustup]: https://www.rustup.rs/

## Using this buildpack

You will first need to set your buildpack in your manifest.yml:

``` yaml
---
applications:
- name: my-project-hello
  memory: 64M
  buildpacks:
  - https://github.com/alphagov/cf-buildpack-rust
```

You will also need to create a `Procfile` pointing to the release version of your application, and commit it to `git`:

```Procfile
web: ./target/release/hello
```

...where `hello` is the name of your binary.

### Running Diesel migrations during the release phase

This will install the diesel CLI at build time and make it available in your app. Migrations will run whenever a new version of your app is released. Add the following line to your `RustConfig`

```sh
RUST_INSTALL_DIESEL=1
```

and this one to your `Procfile`

```Procfile
release: ./target/release/diesel migration run
```

## Specifying which version of Rust to use

By default, your application will be built using the latest stable Rust. Normally, this is pretty safe: New stable Rust releases have excellent backwards compatibility.

But you may wish to use `nightly` Rust or to lock your Rust version to a known-good configuration for more reproducible builds. To specify a specific version of the toolchain, use a [`rust-toolchain`](https://github.com/rust-lang-nursery/rustup.rs#the-toolchain-file) file in the format rustup uses.

Note: if you previously specified a `VERSION` variable in `RustConfig`, that will continue to work, and will override a `rust-toolchain` file.

## Customizing build flags

If you want to change the cargo build command, you can set the `RUST_CARGO_BUILD_FLAGS` variable inside the `RustConfig` file.

```sh
RUST_CARGO_BUILD_FLAGS="--release -p some_package --bin some_exe --bin some_bin_2"
```

The default value of `RUST_CARGO_BUILD_FLAGS` is `--release`.
If the variable is not set in `RustConfig`, the default value will be used to build the project.

## Development notes

If you need to tweak this buildpack, the following information may help.

### Testing with Docker

To test changes to the buildpack using the included `docker-compose-test.yml`, run:

```sh
./test_buildpack
```

Then make sure there are no Rust-related *.so files getting linked:

```sh
ldd heroku-rust-cargo-hello/target/release/hello
```

This uses the Docker image `heroku/cedar`, which allows us to test in an official Cedar-like environment.

