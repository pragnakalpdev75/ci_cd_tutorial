# Chapter 7: Artifacts

Artifacts are one of the most useful features in GitHub Actions.

Think of them as **files that a workflow saves so you can download or use them later**. They are *not* part of your repository — they exist only for that workflow run (or until they expire).

## Why are artifacts useful?

Suppose your test job generates an `htmlcov/` directory containing an HTML coverage report. As covered in Chapter 5, the GitHub runner is destroyed the moment the job finishes — so normally, those files would simply be lost. Uploading them as an artifact preserves them and makes them downloadable from the workflow run's summary page.

## Sharing files between jobs

Remember: every job runs on a fresh virtual machine, so artifacts are the official way to move files between jobs.

```yaml
# Job 1
- run: echo "Hello" > report.txt

- uses: actions/upload-artifact@v4
  with:
    name: report
    path: report.txt
```

```yaml
# Job 2
- uses: actions/download-artifact@v4
  with:
    name: report

- run: cat report.txt
```

The `name:` is what ties the upload in one job to the download in another — as long as both use the same artifact name, `download-artifact` will find it, even though the two jobs never ran on the same machine.

## Common things teams upload as artifacts

- Test reports
- Coverage reports (`htmlcov/`)
- Build logs
- Docker images (`.tar`)
- Compiled binaries
- Frontend build output (`dist/`)
- APK/IPA files for mobile apps
- PDFs or generated documentation

As a rule of thumb: if a job produces a file that a *later job* needs, or that a *human* might want to download after the run finishes, it belongs in an artifact.
