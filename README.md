# Unraid Templates – Docling Serve

This repo hosts the Community Apps template for **Docling Serve**.

- Upstream: https://github.com/docling-project/docling-serve
- Support thread: https://forums.unraid.net/topic/193982-support-docling-serve-convert-documents-to-markdownjson/

## Notes

- `DOCLING_SERVE_ARTIFACTS_PATH` should match the mounted models path (`/opt/app-root/src/.cache/docling/models`).
- If logs show `artifacts_path is set to an invalid directory`, the mapped models directory is usually missing or mismatched.
- Keep `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true` during initial setup so model download issues appear clearly at container startup.
- For NVIDIA systems where CUDA images still run on CPU, ensure `--gpus all` is set and try `cu126` if your driver stack is older.

## Troubleshooting: models not downloading on fresh install

If startup logs show model-not-found errors, work through these checks:

1. **Verify artifacts directory mapping and write access**
   - Container path must be `/opt/app-root/src/.cache/docling/models`.
   - Host path should exist (for example `/mnt/user/appdata/docling/models`) and be writable by the container user.
   - `DOCLING_SERVE_ARTIFACTS_PATH` must match the container path above exactly.

2. **Verify cache locations are writable**
   - Hugging Face cache: `/opt/app-root/src/.cache/huggingface`
   - EasyOCR cache: `/opt/app-root/.EasyOCR`

3. **Verify outbound network from the container**
   - First run needs DNS + HTTPS access to download model artifacts from upstream model hosts.

4. **Expect first-run warmup delay**
   - Initial startup may take longer while models are downloaded.
   - With `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true`, failures surface early in logs instead of you finding out on the first OCR request.
