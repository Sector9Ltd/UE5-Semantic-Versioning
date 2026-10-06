# UE5-Semantic-Versioning

Work out the version and build number from Git tags and the project or plugin version.

By [Sector 9](https://sector9.ltd). [Tool page](https://sector9.ltd/ue5-tools/semantic-versioning) | [Documentation](https://sector9.ltd/docs/ue5-tools/semantic-versioning/)

## Requirements

- A Windows runner. The steps use `shell: powershell`.
- `git` available to that shell. The action runs `git fetch --tags`, `git tag` and `git rev-list`.
- The repository checked out with its full history and tags. Use `actions/checkout@v4` with `fetch-depth: 0`: the action counts commits since the last tag, and a shallow checkout does not hold them.
- A `DefaultGame.ini` (project) or a `.uplugin` (plugin) at the path you give.

Unreal Engine is not needed.

## Usage

`BUILD_PREFIX` is required, and you must give exactly one of `CONFIG_DIR_PATH` (a project) or `UPLUGIN_PATH` (a plugin).

```yaml
jobs:
  version:
    runs-on: [self-hosted, Windows]
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: Sector9Ltd/UE5-Semantic-Versioning@0.4.0
        with:
          BUILD_PREFIX: dev
          CONFIG_DIR_PATH: ${{ github.workspace }}\MyProject\Config

      - run: echo "Building ${{ env.BUILD_ID }}"
        shell: powershell
```

## Inputs

Inputs are set under `with:`. The action compares `USE_RELEASE_BUILD` and `ADD_BUILD_INFO` to the text `true`. Values are pasted into a PowerShell script between double quotes, so do not put a `"` in one.

| Name                | Required | Default | Description                                                                                                                                                |
| ------------------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `BUILD_PREFIX`      | Yes      | —       | Text placed before the commit count in `BUILD_ID`, such as `dev`, `alpha`, `beta` or `rc`. Used only on non-release runs.                                  |
| `CONFIG_DIR_PATH`   | No       | —       | Full path to the project's `Config` directory. Use it for a project, and leave it unset for a plugin. Give this or `UPLUGIN_PATH`, never both.             |
| `UPLUGIN_PATH`      | No       | —       | Full path to a `.uplugin`. Use it for a plugin. The version is read from `VersionName` and written back to it. Give this or `CONFIG_DIR_PATH`, never both. |
| `ADD_BUILD_INFO`    | No       | `true`  | `true` writes `BuildInfo.ini` with the build ID into `CONFIG_DIR_PATH`. Ignored for a plugin, which has no `Config` directory.                             |
| `USE_RELEASE_BUILD` | Yes      | `false` | `true` takes the version and build ID from the ref name, such as a release tag, instead of counting commits. See the documentation link below the table.                                    |

Commits are counted against the newest version tag as written, so `v1.2.3` tags count correctly. The step fails when both or neither of `CONFIG_DIR_PATH` and `UPLUGIN_PATH` are set. For a plugin, the version is written back to the `.uplugin` (`VersionName` and `Version`). For how the version is chosen, see the [documentation](https://sector9.ltd/docs/ue5-tools/semantic-versioning/inputs-and-outputs/).

## Outputs

The action declares no outputs. It sets two environment variables with `GITHUB_ENV`. Later steps in the same job read them as `${{ env.VERSION }}` and `${{ env.BUILD_ID }}`. They are not visible to the step itself or to other jobs.

| Variable   | Value                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------- |
| `VERSION`  | The computed version, such as `1.2.4`. On a release run, the ref name without its `v` and suffix. |
| `BUILD_ID` | `<VERSION>-<BUILD_PREFIX><commit count>`, or the ref name as it is on a release run.              |

## Other UE5 Tools

- [UE5-Build-Project](https://github.com/Sector9Ltd/UE5-Build-Project): Build, cook, stage, package and archive an Unreal project with RunUAT.
- [UE5-Build-Plugin](https://github.com/Sector9Ltd/UE5-Build-Plugin): Build and package an Unreal plugin with RunUAT BuildPlugin.
- [UE5-EOS-Config](https://github.com/Sector9Ltd/UE5-EOS-Config): Write Epic Online Services settings into DefaultEngine.ini, with an optional dedicated-server config.

## License

See [LICENSE](LICENSE).
