# Unraid Templates – Docling Serve

This repo hosts the Community Apps template for **Docling Serve**.

- Upstream: https://github.com/docling-project/docling-serve
- Support thread: https://forums.unraid.net/topic/193982-support-docling-serve-convert-documents-to-markdownjson/

## Quick Tips

- `DOCLING_SERVE_ARTIFACTS_PATH` should match the mounted models path (`/opt/app-root/src/.cache/docling/models`).
- If logs show `artifacts_path is set to an invalid directory`, the mapped models directory is usually missing or mismatched.
- Keep `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true` during initial setup so model download issues appear clearly at container startup.
- **CPU vs GPU image:** `DOCLING_DEVICE=cpu` is a *runtime hint* — it does **not** replace choosing the `cpu` image branch. For CPU-only servers, select the `cpu` branch in the template.

## Troubleshooting: models not downloading on fresh install

If startup logs show model-not-found errors (e.g. `RapidOcr ... .onnx is not found`), work through these checks:

1. **Enable remote services for first run**
   - Set `DOCLING_SERVE_ENABLE_REMOTE_SERVICES=true` so RapidOCR artifacts can download from upstream hosts such as `modelscope.cn`.
   - After the first successful start and model cache is populated, you may set this back to `false`.

2. **Verify artifacts directory mapping and write access**
   - Container path must be `/opt/app-root/src/.cache/docling/models`.
   - Host path should exist (for example `/mnt/user/appdata/docling/models`) and be writable by the container user.
   - `DOCLING_SERVE_ARTIFACTS_PATH` must match the container path above exactly.

3. **Verify cache locations are writable**
   - Hugging Face cache: `/opt/app-root/src/.cache/huggingface`
   - EasyOCR cache: `/opt/app-root/.EasyOCR`

4. **Verify outbound network from the container**
   - First run needs DNS + HTTPS access to download model artifacts from upstream model hosts.

5. **Expect first-run warmup delay**
   - Initial startup may take longer while models are downloaded.
   - With `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true`, failures surface early in logs instead of the first OCR request.
