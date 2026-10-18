---
name: liora-sku-schema-validator
description: Connects to the Liora ERP & Website Admin Panel to validate product records, detect duplicate SKUs, verify image assets, and enforce the strict manual LKR discount rule.
---

# Verification & Execution Procedure

1. Open the Liora ERP Product Catalog table and Website Admin Panel product list.
2. For each product record, verify the following mandatory fields are non-null:
   - `product_name` (Title Case, e.g., "Liora Grand Aurelia Block Heel Sandal - Black")
   - `master_sku` (`LG-FW-[4_DIGIT_CODE]` for Footwear, `LG-HB-[4_DIGIT_CODE]` for Handbags)
   - `variant_skus` (Appended with `-[COLOR_CODE]-[EU_SIZE]`, e.g., `LG-FW-1042-BLK-38`)
   - `category` (`Footwear` or `Handbags`)
   - `unit_cost_lkr` (Numeric $> 0$)
   - `retail_price_lkr` (Numeric $> \text{unit\_cost\_lkr}$)
   - `manual_discount_lkr` (Numeric $\ge 0$; verify `discount_type` is strictly `FIXED_AMOUNT`, never `PERCENTAGE`)
   - `final_selling_price_lkr` (Must equal $\text{retail\_price\_lkr} - \text{manual\_discount\_lkr}$)
   - `image_urls` (Minimum 3 resolving HTTPS URLs returning HTTP 200)
3. Check for duplicate `master_sku`, `variant_skus`, or fuzzy-matched `product_name` strings ($>90\%$ similarity).
4. Output a structured Markdown Exception Table listing any failing SKUs, exact missing/corrupted fields, and the prepared correction payload waiting for Owner approval.

---

## Output Template: Exception Table

| Master / Variant SKU | Failing Field(s) | Issue Description | Prepared Correction Payload (JSON) | Owner Approval Status |
| :--- | :--- | :--- | :--- | :--- |
| `LG-FW-1042-BLK-38` | `discount_type`, `final_selling_price_lkr` | `discount_type` set to `PERCENTAGE`; final price mismatch. | `{"discount_type": "FIXED_AMOUNT", "manual_discount_lkr": 1500, "final_selling_price_lkr": 8000}` | Pending |
| `LG-HB-2015-TAN-OS` | `image_urls` | Only 2 image URLs provided; 3rd URL returns HTTP 404. | `{"image_urls": ["https://cdn.liora.lk/hb2015-1.jpg", "https://cdn.liora.lk/hb2015-2.jpg", "https://cdn.liora.lk/hb2015-3.jpg"]}` | Pending |