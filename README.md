# Unraid Templates – Docling Serve

This repo hosts the Community Apps template for **Docling Serve**.

- Upstream: https://github.com/docling-project/docling-serve
- Support thread: https://forums.unraid.net/topic/193982-support-docling-serve-convert-documents-to-markdownjson/

## Notes

- `DOCLING_SERVE_ARTIFACTS_PATH` should match the mounted models path (`/opt/app-root/src/.cache/docling/models`).
- If logs show `artifacts_path is set to an invalid directory`, the mapped models directory is usually missing or mismatched.
- Keep `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true` during initial setup so model download issues appear clearly at container startup.
- For NVIDIA systems where CUDA images still run on CPU, ensure `--gpus all` is set and try `cu126` if your driver stack is older.
- If you want CPU-only operation, select the **`cpu` image branch** (`quay.io/docling-project/docling-serve-cpu`). `DOCLING_DEVICE=cpu` controls runtime device selection but does not replace choosing the CPU image.

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
   - With `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true`, failures surface early in logs instead of the first OCR request.

## Troubleshooting: RapidOCR `file not found` warnings

If you see warnings like:

- `docling.models.stages.ocr.rapid_ocr_model ... RapidOcr ... .onnx is not found`
- `... ppocr_keys_v1.txt is not found`
- and no obvious model-download activity

Use this targeted checklist:

1. **Confirm CPU/GPU image choice first**
   - For CPU-only setups, use template branch **`cpu`** (`quay.io/docling-project/docling-serve-cpu`).
   - Do not rely only on `DOCLING_DEVICE=cpu` if your intent is to switch image variants.

2. **Confirm artifacts path mapping is exact**
   - Container path: `/opt/app-root/src/.cache/docling/models`
   - Host path example: `/mnt/user/appdata/docling/models`
   - `DOCLING_SERVE_ARTIFACTS_PATH` must equal `/opt/app-root/src/.cache/docling/models`

3. **Confirm write permissions**
   - The container user must be able to write into `/mnt/user/appdata/docling/models` (and related cache paths).
   - If using `PUID/PGID`, ensure ownership/permissions align with those IDs.

4. **Confirm startup model preloading is enabled**
   - Set `DOCLING_SERVE_LOAD_MODELS_AT_BOOT=true` to force first-run model download/validation at startup.

5. **Confirm custom Docker network has outbound DNS + HTTPS**
   - First run requires network access to fetch model artifacts.
   - If no download lines appear in logs, test egress from that network before retrying.
