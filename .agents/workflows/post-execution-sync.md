---
description: Post-execution context sync — automatically analyzes changes and updates Spec docs and Skills after complex tasks
---

# Post-Execution Sync Workflow

> **Trigger:** Run this workflow after completing any **COMPLEX** or **ORCHESTRATE** task (as classified by the Socratic Gate). Do NOT run for SIMPLE FIX or CLARIFY requests.

---

## When to Trigger

Run this sync when the completed work matches ANY of:
- Created or deleted files
- Changed module structure, imports, or dependencies
- Added/removed/refactored a feature
- Changed architectural patterns
- Introduced new tech, libraries, or integrations
- Made decisions with >10% project impact

**Skip** for: typo fixes, copy changes, single-line tweaks, style-only edits.

---

## Step 1: Analyze Impact

Before updating anything, classify what changed:

```
CHECKLIST:
- [ ] Khởi tạo hoặc xóa file script/Prefab mới?
- [ ] Di chuyển thư mục, thay đổi cấu trúc Assets?
- [ ] Thay đổi giới hạn module (ví dụ: Core bắt đầu gọi UI)?
- [ ] Thêm plugin/package mới (Unity Package Manager, Asset Store, .dll)?
- [ ] Thay đổi API Public (Interface, Event mới, tham số hàm thay đổi)?
- [ ] Giới thiệu Design Pattern mới (ví dụ: bắt đầu dùng Object Pool)?
- [ ] Có logic/domain mới chưa được ghi chú luật (ví dụ: Shader, IAP)?
- [ ] Có quyết định kỹ thuật ảnh hưởng đến tương lai dự án không?
```

---

## Step 2: Update Spec Documents

Based on the impact analysis, update **only** the relevant Spec docs:

### Priority: CRITICAL (always check)

| Condition | Action |
|-----------|--------|
| Task was tracked in `tasks.md` | Mark as `[DONE]`, update date |
| Any architectural change | Update `@ARCHITECTURE` with new structure |

### Priority: HIGH (check if applicable)

| Condition | Action |
|-----------|--------|
| Tech stack changed (new Unity Package, plugin, tool) | Update `@STACK` tech stack section |
| UI Prefab, Materials, Shader changed | Update `DESIGN_SYSTEM` (nếu có) |

### Priority: LOGGED (append if applicable)

| Condition | Action |
|-----------|--------|
| Architectural/stack decision was made | Append to `@STATE` |
| New risk identified during implementation | Append to `@STATE` |
| Sprint milestone reached | Update `@ROADMAP` |

---

## Step 3: Evaluate Skills

Check if the completed work reveals new patterns or important functions.

> **[LUẬT THÉP - CẤM TẠO SKILL MỚI]**
> CẤM tuyệt đối việc tự động tạo thư mục/file SKILL mới. Khi phát hiện một mẫu code (pattern) mới hoặc một hàm cốt lõi cần ghi nhớ để tái sử dụng, hãy thực hiện theo thứ tự ưu tiên sau:
> 
> 1. Nếu hàm/pattern đó thuộc về các kỹ năng ĐÃ CÓ (ví dụ: `brain`, `testing`, `debugging`), hãy append tên hàm và cách dùng vào file `SKILL.md` của kỹ năng đó.
> 2. Nếu đó là quy tắc cấu trúc chung của dự án (như cách xử lý Unity Physics, Grid Logic...), hãy append vào file `.specs/project/CONVENTIONS.md`.
> 3. Chỉ được phép ĐỀ XUẤT tạo Skill mới khi dự án tích hợp một mảng công nghệ hoàn toàn độc lập và quy mô lớn (ví dụ: cổng thanh toán IAP, hệ thống server multiplayer), và **bắt buộc phải đợi User tạo thủ công**.

---

## Step 4: Report Summary

After syncing, provide a brief summary to the user:

```markdown
## 🔄 Post-Execution Sync Complete

**Spec Updates:**
- [list of .specs/ docs updated, or "None needed"]

**Skills:**
- [new skills created, existing skills updated, or "No changes needed"]

**Decision Log:**
- [decisions recorded, or "No new decisions"]
```

---

## Notes

- **Be surgical:** Only update docs that are actually affected. Do not rewrite entire documents.
- **Preserve history:** When updating `@STATE`, always append — never overwrite past entries.
- **Minimal diffs:** Update the specific section within a Spec doc, not the whole file.
- **User confirmation:** If unsure whether a change warrants a new Skill, ask the user before creating one.
