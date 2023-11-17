# Publish Action Package

This action packages your action repository as OCI artifacts and publishes it to [GHCR](ghcr.io).

This allows your action to be consumed as an immutable package to make the actions ecosystem more secure.

The whole action repository is packaged by default. Set `path` input to specify which path you want to package.

Make sure you use the [Starter Workflow](https://github.com/actions-on-packages/.github) (TODO) to run the action.
Please also ensure you have the release trigger in the workflow where you use this action.

## Usage

<!-- start usage -->
```yaml
on:
  release:

- uses: immutable-actions/publish-action-package@1.0.1
  with:

    # Personal access token (PAT) or GITHUB_TOKEN with write:package scope used to upload the package to GHCR. The GITHUB_TOKEN is taken by default.
    #
    # We recommend using a service account with the least permissions necessary. Also
    # when generating a new PAT, select the least scopes necessary.
    #
    # [Learn more about creating and using encrypted secrets](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets)
    #
    # Default: ${{ github.token }}
    token: ''

    # Relative path of the working directory of the repository to be tar archived
    # and uploaded as OCI Artifact layer. You can mention multiple files/folders
    # by mentioning relative paths as space separated values.
    #
    # This defaults to the entire action repository contents if not explicitly defined.
    # Default: '.'
    path: 'src/ action.yml dist/'


```
<!-- end usage -->

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

## [Internal] Differences from previous implementation

This is a new implementation of an Action which publishes a given release to ghcr.io (GitHub Packages).

It will eventually be moved to our `actions`` org.

The key differences are:

- This Action goes directly to GitHub Packages rather than using an API endpoint to pass a bundle to.
- This Action uses Node.js libraries to create both a `zip` and `tar.gz` of the content as layers.
- This Action creates and publishes the OCI manifest which houses those archives, which was previously done on the backend.
- This Action has the goal of generating provenance attestations for any release that is created.
- This Action parses and validates that the release tag which triggered it is in a valid SemVer format, either `1.0.3-prerelease` or `v1.0.0-prerelease`.

## [Internal] Features to consider adding to this Action

These are some features which should likely be part of this implementation.

- [ ] Parsing SemVer more strictly, based on what formats we decide to support (e.g. we might not support prerelease tags)
- [ ] Checking that there is a valid `action.yml` file in the repository running the action, failing if it is not an Action repository
  - Could be as simple as just checking for the file, or could be validating it is yml with expected actions components
- [ ] Sanitising and validating relative paths passed into the `path` arg
  - For example, checking that they're inside the repository of the action and not other places on the filsystem
- [ ] Integration with Attestations API (speak to `phillmv` about this)
- [ ] A way to dynamically work out the Container Registry's URL rather than relying on the user to provide it
  - Options seem to be an API endpoint or an environment variable in the runner, but this might be difficult for backwards compatibility
