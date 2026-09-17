# UE5-Semantic-Versioning

This GitHub Action fetches Git tags and calculates the version and build number for Unreal Engine projects by reading the project version from an INI file. It supports both regular development builds and release builds by optionally using the release tag for versioning.

## How it Works

The action performs the following steps:

1.  Fetches all Git tags available in the repository.
2.  Reads the specified INI file to extract the project's version.
3.  For release builds, it uses the GitHub release tag as the `BUILD_ID` and sets `VERSION` to major.minor.patch format.
4.  For non-release builds, it identifies the latest Git tag that matches a Semantic Versioning pattern and calculates the number of commits since the latest identified version tag.
5.  Outputs the new version and build ID, considering the provided build prefix, commit count, and whether it's a release build.

## Requirements

-   The repository must have an initial semantic tag of at least `0.0.0`.
-   The INI file referenced should contain the project version in the correct format, typically found in `DefaultGame.ini`.
-   For non-release builds, build prefixes are added as part of your CI process.

## Inputs

-   `BUILD_PREFIX`: The prefix to apply to the build number (e.g., `dev`, `alpha`, `beta`, `rc`).
-   `CONFIG_DIR_PATH`: The full path to the `MyProject/Config` folder, including the path to the GitHub workspace. For a project. Mutually exclusive with `UPLUGIN_PATH`.
-   `UPLUGIN_PATH`: The full path to a `.uplugin`. For a plugin. Mutually exclusive with `CONFIG_DIR_PATH`.
-   `ADD_BUILD_INFO`: Adds a `BuildInfo.ini` file containing the Build ID to the `MyProject/Config` folder. Ignored for a plugin.
-   `USE_RELEASE_BUILD`: Specifies whether to use the release build logic (`true` or `false`). If `true`, the action uses the GitHub release tag as the build version.

## Plugins

A plugin has no `Config/DefaultGame.ini`. Its version lives in the descriptor, so pass
`UPLUGIN_PATH` instead and the action reads `VersionName` in place of `ProjectVersion`.

For a plugin the version is also **written back** to the descriptor. That is the point: `RunUAT
BuildPlugin` packages whatever the descriptor says, and the packaged zip is conventionally named
from `VersionName`, so a tag can only reach a release if it lands in the file that ships. A
project's version is reported and not written, and that is unchanged.

Both fields move together, because UE compares the integer `Version`, not the string:

| Field | Written as |
| --- | --- |
| `VersionName` | the computed semver string, e.g. `2.3.1` |
| `Version` | `major * 10000 + minor * 100 + patch`, e.g. `20301`, and never lower than what was there |

The two fields are replaced in place rather than by re-serialising the JSON, so the diff is the two
lines that changed and the rest of the descriptor keeps its formatting and key order.

``` yaml
- name: Set Plugin Version
  uses: Sector9Ltd/UE5-Semantic-Versioning@latest
  with:
    BUILD_PREFIX: dev
    UPLUGIN_PATH: ${{ github.workspace }}/MyPlugin/MyPlugin.uplugin
    USE_RELEASE_BUILD: false

- uses: Sector9Ltd/UE5-Build-Plugin@v1
  with:
    RUNUAT_PATH: ${{ env.RUNUAT }}
    UPLUGIN_PATH: ${{ github.workspace }}/MyPlugin/MyPlugin.uplugin
    PACKAGE_PATH: ${{ runner.temp }}/PluginBuild
    ARCHIVE: 'true'
```

## Using the Action

Add the following step to your workflow to use this action:

``` yaml
- name: Set Project Version and Build Number
  uses: OrchidIsle/UE5-Semantic-Versioning@latest
  with:
    BUILD_PREFIX: dev
    CONFIG_DIR_PATH: ${{ github.workspace }}/MyGameFolder/Config
    ADD_BUILD_INFO: true
    USE_RELEASE_BUILD: false` 
```

## Outputs

-   `BUILD_ID`: Identifier for the build. For release builds, it's the release tag. For non-release builds, it includes the version number, build prefix, and commit count.
-   `VERSION`: The release version in Major.Minor.Patch format based on Semantic Versioning.

Example of using outputs in your workflow:

``` yaml
- name: Use Build ID
  run: echo "The build ID is ${{ env.BUILD_ID }}"` 
```

```yaml
- name: Use Version
  run: echo "The next version is ${{ env.VERSION }}"` 
```
## Typical Usage

Set the project settings version to the version you plan to release next. The action will output the appropriate build and version numbers based on your workflow configuration.

## Additional Information

-   Use the actions/checkout step before this action to clone the repository.
-   Update the version in the INI file as part of your development process.
-   Visit SemVer.org for more information on Semantic Versioning.

----------
