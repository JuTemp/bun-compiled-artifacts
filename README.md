# bun-compiled-artifacts

This project periodically checks for new `Bun` releases; if a new version is detected, it builds and publishes a release.

This project executes `bun build ./index.ts --compile` to generate the corresponding `amd64-glibc` and `amd64-musl` build artifacts.

The goal of this project is to address the issue of excessively large build artifacts during upload. Since Bun's build artifacts consist of the Bun binary itself and the corresponding code, uploading the massive binary is unnecessary if a placeholder for the Bun binary already exists in the cloud. Furthermore, `rsync` determines binary file duplication by comparing chunks; in my project, provided the Bun binary is already present in the cloud, only a few kilobytes of code need to be uploaded, making the process extremely fast.

## Note

When compiling locally, you **must** use the same Bun version and execute `bun build ./index.ts` **within the `/app` directory**. **Do not** pass the `--outfile` parameter; it will generate an `index` binary by default, which you can then rename and upload.

You can check whether the local and cloud environments are using the same image by comparing their `index digests`.

```bash
$ docker image ls oven/bun:latest
REPOSITORY   TAG       IMAGE ID       CREATED        CREATED AT                      SIZE      REPOSITORY:TAG
oven/bun     latest    e10577f0db68   2 months ago   2026-05-13 11:50:50 +0800 CST   310MB     oven/bun:latest

$ docker image inspect oven/bun:latest
...
        "Id": "sha256:e10577f0db68676a7024391c6e5cb4b879ebd17188ab750cf10024a6d700e5c4",
...
```

You can compare the local `index digest` and the release message; if the versions are identical, executing the following commands should generate completely identical files.

```bash
cd /app
echo 'console.log("Hello via Bun!");' > ./index.ts
bun build ./index.ts --compile
```

