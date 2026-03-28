# Unraid Templates – Docling Serve

This repo hosts the Community Apps template for **Docling Serve**.

- Upstream: https://github.com/docling-project/docling-serve
- Support thread: https://forums.unraid.net/topic/193982-support-docling-serve-convert-documents-to-markdownjson/

## Notes

- `DOCLING_SERVE_ARTIFACTS_PATH` should match the mounted models path (`/opt/app-root/src/.cache/docling/models`).
- If logs show `artifacts_path is set to an invalid directory`, the mapped models directory is usually missing or mismatched.
- For NVIDIA systems where CUDA images still run on CPU, ensure `--gpus all` is set and try `cu126` if your driver stack is older.
