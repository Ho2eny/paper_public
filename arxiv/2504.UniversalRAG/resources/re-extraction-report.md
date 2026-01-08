# Figure Re-Extraction Report

**Paper**: 2504.20734 (UniversalRAG)
**Date**: 2026-02-06
**Method**: PDF Structure-Based (PyMuPDF)
**Task**: Re-extract 3 problematic figures with tighter bounding boxes

---

## Issues Addressed

1. **Figure 1**: Previous extraction included Abstract body text from left column
2. **Figure 3**: Previous extraction included Table 1 data above the bar charts
3. **Figure 7a**: Previous extraction included surrounding tables and headers

---

## PDF Structure Analysis

| Figure | Page | Total Drawings | Region Analysis | Solution |
|--------|------|----------------|-----------------|----------|
| 1 | 1 | 71 | Two-column layout: left=Abstract, right=Figure | Extract right column only (x > 305) |
| 3 | 5 | 888 | Table 1 ends at y=280, bar charts start at y=282 | Start from y=282 (bar charts only) |
| 7a | 19 | 1245 | Table 9 data in left column (y=497-603) | Extract left column table region only |

---

## Extracted Figures

### Figure 1

- **Page**: 1
- **Description**: Conceptual illustration comparing RAG strategies
- **Old bbox**: [65.87, 214.9, 529.42, 406.71]
- **New bbox**: [301.14, 214.9, 529.42, 406.81]
- **Fix applied**: Changed x_min from 65.87 to 301.14 (right column only)
- **File**: figure_1.png
- **Size**: 952x801 pixels (300 DPI)

### Figure 3

- **Page**: 5
- **Description**: Bar chart comparison across different RAG methods
- **Old bbox**: [52.15, 115.55, 527.95, 380.28]
- **New bbox**: [67.33, 277.02, 527.95, 380.28]
- **Fix applied**: Changed y_min from 115.55 to 277.02 (exclude Table 1)
- **File**: figure_3.png
- **Size**: 1920x431 pixels (300 DPI)

### Figure 7a

- **Page**: 19
- **Description**: Table 9: Effect of granularity on performance
- **Old bbox**: [65.87, 495.49, 527.98, 664.63]
- **New bbox**: [65.87, 491.98, 291.95, 608.19]
- **Fix applied**: Changed x_max from 527.98 to 291.95, y_max from 664.63 to 608.19 (left column table only)
- **File**: figure_7a.png
- **Size**: 943x486 pixels (300 DPI)

---

## Verification Results

| Figure | Unwanted Content Excluded | Expected Content Present | Status |
|--------|--------------------------|--------------------------|--------|
| 1 | ✓ Abstract text excluded | ✓ All diagram labels present | PASS |
| 3 | ✓ Table 1 excluded | ✓ Bar charts and legend present | PASS |
| 7a | ✓ Other tables/headers excluded | ✓ Table 9 data complete | PASS |

---

## Extraction Details

### Method

1. Analyzed PDF structure using PyMuPDF 
2. Identified caption positions for boundary anchoring
3. Filtered drawings by region (column, y-coordinate)
4. Calculated precise bounding boxes with 5pt margins
5. Extracted at 300 DPI using 
6. Verified text content within each bbox

### Key Learnings

- **Two-column layouts**: Must separate left/right content by x-coordinate
- **Stacked content**: Use y-coordinate filtering to exclude adjacent tables/figures
- **Caption anchoring**: Figure boundaries end where captions begin
- **Drawing clustering**: Group drawings by region to identify figure extent
