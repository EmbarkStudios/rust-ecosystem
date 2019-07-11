# Open Sourcing

TODO: More sections and beautiful prose

## Release Steps

TODO: More steps here

### Setup New Repo

1. Pick a name that isn't used on crates.io, also an **emoji** :100:
1. Do the usual repo creation under the [Embark](https://github.com/EmbarkStudios) org with the same name as the crate(s)
1. Clone down the repo or else set it as the remote for an existing repo
1. `cargo new` or `init` your repo if needed
1. Do a `cargo package` and fixup any issues, normally this is stuff like missing the repository/docs/homepage etc metadata fields
1. Do an initial publish of the crate with `cargo publish --token <TOKEN>` where `TOKEN` is
and API token for the [`embark-studios`](https://crates.io/users/embark-studios) user. This shared
bot account allows us to publish all crates under the same user and not have to worry about managing owners.

### Add CI

We mainly use Travis CI for all of our public code, but the general steps here would apply to
whatever CI a repo decides to use.

1. Create .travis.yml file, or copy one of our existing .travis.yml files, if a bin project, try [cargo-deny](https://github.com/EmbarkStudios/cargo-deny/blob/master/.travis.yml), if a lib project, try [tame-oauth](https://github.com/EmbarkStudios/tame-oauth/blob/master/.travis.yml), as well as the associated .ci/ scripts used.
1. Add/remove steps as makes sense for the project
    * As noted in our guidelines, we expect all our code to follow default rustfmt, and not have clippy
    warnings, so have at least 1 lint step that checks these on the `stable` channel, and it's recommended
    to have one that does the same thing on the `beta` channel as well as new clippy lints can trigger failures
    in the next `stable` release
    * Have a `stable` test for all the platforms we support, which is usually `windows`, `linux`, and `osx`
    * Probably want to have at least 1 test that runs using the `nightly` channel, it's recommended to use
    `linux` for this as the VM spinup, rust install, compilation, and test runs are generally all faster on
    linux machines than the other platforms, particularly Windows
1. Add a deploy step
    * Add a separate deploy stage that only runs on semver tags
    ```yml
    stages:
    - test
    - name: deploy
      if: tag =~ /^\d+\.\d+\.\d+.*$/
    ```
    * If a lib/bin project, add a deploy step for publishing to crates.io
    ```yml
    - stage: deploy
      rust: stable
      os: linux
      script: echo "deploying $TRAVIS_TAG to crates.io"
      deploy:
        provider: cargo
        token:
          secure: <SEKRETZ>
        on:
          repo: EmbarkStudios/your-repo-name
          tags: true
    ```
    You should get an crates.io API token from Johan for the `embark-studios` Github bot we have, but
    Once you have the API token, you need to encrypt it, which you can do with [travis-cli](https://github.com/travis-ci/travis.rb#installation) tool `travis encrypt <token>`. Then just copy and paste the token
    into the `<SEKRETZ>` placeholder.
    * If a bin project, copy what cargo-deny does, including all of the scripts. This will generate .tar.gz archives
    for all target platforms and upload them to Github releases so that people can install the binary without having
    to install Rust if that is their desire.
1. Push your travis config and see how it goes!

### Publishing New Versions

Given the above setup is followed correctly, publishing turns into a fairly trivial exercise.

1. Change the `version` in Cargo.toml and commit
1. Tag the commit `git tag -a <version> -m "Release <version>"`. Example: `git tag -a 0.1.0 -m "Release 0.1.0"
1. Push the commit(s) and tag so that the release will be published automatically `git push --follow-tags`
