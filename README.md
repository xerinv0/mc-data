# data_generator

This repository automatically runs the [Data Generator](https://minecraft.wiki/w/Tutorial:Running_the_data_generator) for every available Minecraft server version and publishes the results as a static JSON API with interactive OpenAPI docs.

A GitHub Actions workflow runs on a schedule, uses the official Mojang server JAR for each version, and commits the output to a single `data` branch. GitHub Pages serves that branch.

> Forked from [MinecraftDataGenerator/data_generator](https://github.com/MinecraftDataGenerator/data_generator). The original publishes one branch per version; this fork keeps everything on one branch with a stable, versioned folder layout so it can be used as a web API.

---

## API and Docs

Once GitHub Pages is enabled for the `data` branch, the API lives at:

```
https://xerinv0.github.io/mc-data/
```

Opening that URL shows the interactive docs (Swagger UI), generated from `openapi.json`.

| Endpoint | Description |
| --- | --- |
| `/versions.json` | All available versions, newest first |
| `/latest.json` | Latest release and snapshot ids |
| `/openapi.json` | OpenAPI 3 specification |
| `/versions/{type}/{id}/reports/{report}.json` | A generated report for one version |

Examples:

```
/versions/release/1.21.10/reports/blocks.json
/versions/release/1.21.10/reports/items.json
/versions/snapshot/<snapshot-id>/reports/registries.json
```

All responses are static files with open CORS headers, so they can be fetched directly from a browser or any HTTP client.

---

## Branch and Folder Structure

| Branch | Purpose |
| --- | --- |
| `main` | Workflow file and documentation |
| `data` | Generated data and the GitHub Pages site |

Layout of the `data` branch:

```
index.html          Swagger UI docs page
openapi.json        OpenAPI spec (rebuilt on every run)
versions.json       Index of all versions
latest.json         Latest release / snapshot ids
versions/
  release/<version>/
    meta.json
    reports/
  snapshot/<version>/
    meta.json
    reports/
```

`<type>` is the version type from Mojang's manifest (`release` or `snapshot`). Each version keeps its own folder, so nothing is overwritten when a new version is added.

You can also read files without Pages through raw URLs:

```
https://raw.githubusercontent.com/xerinv0/mc-data/data/versions/release/1.21.10/reports/blocks.json
```

---

## How It Works

1. The workflow runs daily (`cron`) or manually via the Actions tab.
2. It loads Mojang's version manifest and walks every version from `START_VERSION` up to the latest, oldest first.
3. For each version that has not been generated yet:
   - It downloads the official Mojang server JAR (into a temporary folder, never committed).
   - It runs the Data Generator bundled in that server (`--all`).
   - It copies the output into `versions/<type>/<version>/` on the `data` branch and commits it.
4. It rebuilds `versions.json`, `latest.json`, `openapi.json` and `index.html`.

Versions that already exist are skipped, so a normal run only processes what is new.

---

## Configuration

Set these in the `env` block of the workflow:

| Variable | Default | Description |
| --- | --- | --- |
| `DATA_BRANCH` | `data` | Branch that stores the data and the Pages site |
| `START_VERSION` | `1.21.10` | First version to process |
| `INCLUDE_DATA` | `false` | Also publish the large `data/` folder (loot tables, recipes, and so on), not just `reports/` |
| `SITE_URL` | empty | Custom domain used in the OpenAPI spec, for example `https://api.example.com` |

Notes:

- The Data Generator only exists in Minecraft 1.13 and later, so `START_VERSION` cannot be older than that.
- GitHub Pages sites are limited to about 1 GB. Publishing only `reports/` (the default) keeps each version small.

---

## Setup (for your own fork)

1. Add a `GH_PAT` repository secret (a PAT with Contents read and write). It is optional, but it makes sure pushes trigger the Pages build.
2. Run the workflow once from the Actions tab to create the `data` branch.
3. Go to **Settings → Pages**, choose "Deploy from a branch", select the `data` branch and the `/ (root)` folder.
4. Open `https://xerinv0.github.io/mc-data/`.

---

## 📝 Notes on Licensing

The generated data originates from the official Mojang server JAR.
While the generator output is not explicitly licensed by Mojang, it is derivative of Minecraft's game data.

Because of that, **it is unclear whether a custom open-source license should be applied**.
For now, the repository **does not apply an additional license** to avoid implying permissions that may not exist.

If you intend to use the data, please ensure you comply with Mojang's guidelines and EULA.
