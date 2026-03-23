# `tests/test_utils.py`

## High-Level Purpose

Unit tests for the content block conversion functions in `deepagents_acp.utils`. Verifies that ACP protocol block types are correctly converted to LangChain-compatible dict representations.

## Test Functions

| Test | Input | Expected Output |
|------|-------|-----------------|
| `test_convert_text_block_to_content_blocks` | `TextContentBlock(type="text", text="hi")` | `[{"type": "text", "text": "hi"}]` |
| `test_convert_image_block_to_content_blocks_with_data` | `ImageContentBlock(mime_type="image/png", data="AAAA")` | `[{"type": "image_url", "image_url": {"url": "data:image/png;base64,AAAA"}}]` |
| `test_convert_image_block_to_content_blocks_without_data_falls_back_to_text` | `ImageContentBlock(mime_type="image/png", data="")` | `[{"type": "text", "text": "[Image: no data available]"}]` |
| `test_convert_resource_block_to_content_blocks_truncates_root_dir` | `ResourceContentBlock(uri="file:///root/subdir/file.txt")`, `root_dir="/root"` | URI in output is truncated to `file://subdir/file.txt` |
| `test_convert_embedded_resource_block_to_content_blocks_text` | `EmbeddedResourceContentBlock` with `TextResourceContents(text="hello", mime_type="text/plain")` | `[{"type": "text", "text": "[Embedded text/plain resource: hello"}]` |

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| ACP schema types | `acp.schema` | Input block types |
| Conversion functions | `deepagents_acp.utils` | Functions under test |
