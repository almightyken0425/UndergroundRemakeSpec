# CLAUDE.md

本 repo 為 UndergroundRemake 產品的 **Module Spec git**，module_id 為 `no1_concept`，承載概念階段遊戲的 module 規格。

## 三層 git 配對

- **頂層 Product git：** `../../`
- **本 Module Spec git：** 即本 repo，位於 Product git 的 `no3_product_specs/no1_concept/`
- **對側 Module Impl git：** 尚無，本 module 屬概念階段，auditor 回 `CONCEPT_ONLY`

完整配對表由 `decision_framework_router` skill 的 `products_registry.md` 維護。

---

## 當前 Spec 內容

- Logic 層
    - `no3_logics/no1_vehicle_physics_engine.md` — 載具物理引擎規格
- Model 層與 View 層待補

---

## 撰寫規範

所有規格文件依循 `spec_writer` 分層政策與 `universal_writing_linter` 通用政策。
