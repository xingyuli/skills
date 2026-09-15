# Lanhu sources

Drive every list/download through the **lanhu MCP** (`GetDynamicTools` / `CallDynamicTool` on namespace `lanhu`). Cookie `curl` is only the unique-file write in Download below.

## URL kinds

| URL | MCP tool |
|---|---|
| `.../link/#/invite?sid=` | `lanhu_resolve_invite_link` first |
| `.../stage?tid=&pid=` (no `docId`) | `lanhu_get_designs`: UI frames, grouped by **sector** |
| `.../product?tid=&pid=&docId=` | `lanhu_get_pages`: Axure pages |

`lanhu_list_product_documents` returns empty on a design-only project. A missing `docId` is a stage board.

## Download

Frames in one sector are often all named `页面`. `lanhu_get_ai_analyze_design_result` writes one `页面.png` and clobbers the rest.

For each design in the named sector:

1. Take `index`, `id`, and `url` from `lanhu_get_designs`.
2. GET `url` with the Lanhu cookie (`Referer: https://lanhuapp.com/web/`).
3. Write `{index}_{id[:8]}.png`.
4. Read the image, then rename to a screen title.

**Done when:** file count matches the sector's `image_count`, and names describe the screen.
