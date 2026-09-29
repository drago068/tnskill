# Project Documentation: ServiceNow Import Sets & Transform Maps

---

## 1. Executive Summary

This project documents the end-to-end implementation of bulk data migration into the ServiceNow platform using **Import Sets**, **Transform Maps**, **Coalesce Configurations**, **Reporting**, and **Platform Analytics Dashboards**.

The objective is to simulate an enterprise data onboarding lifecycle:
1. Ingesting raw external employee data from an Excel spreadsheet.
2. Staging data within a temporary Import Set table (`u_employee_import`).
3. Mapping fields and transforming data into a production-ready custom table (`u_employee_test`).
4. Preventing record duplication through primary key coalescing on `Employee ID`.
5. Executing delta/update imports and verifying idempotency.
6. Generating real-time visual reports and consolidating them into an **Employee Analytics Dashboard**.

---

## 2. Technical Architecture & Data Flow

```mermaid
flowchart TD
    subgraph Data Sources
        S1["Source Spreadsheet<br/>(sample-employee spread sheet .xlsx)"]
        S2["Delta Spreadsheet<br/>(updated sample employee spread sheet.xlsx)"]
    end

    subgraph ServiceNow Staging Area
        IS["Import Set Staging Table<br/>(u_employee_import)"]
    end

    subgraph Transformation Engine
        TM["Transform Map<br/>(Sample Spreadsheet Import)"]
        CO{"Coalesce on<br/>Employee ID"}
    end

    subgraph ServiceNow Target System
        TT[("Target Table<br/>u_employee_test")]
    end

    subgraph Analytics & Visualization
        R1["Report: Employees by Department (Pie Chart)"]
        R2["Report: Employees by Location"]
        R3["Report: Employee List (Data Grid)"]
        DB["Dashboard:<br/>Employee Analytics Dashboards"]
    end

    S1 -->|Load Data| IS
    S2 -->|Load Data (Delta)| IS
    IS --> TM
    TM --> CO
    CO -->|No Match (Insert)| TT
    CO -->|Match Found (Update)| TT
    TT --> R1 & R2 & R3
    R1 & R2 & R3 --> DB
```

---

## 3. Project Assets & Repository Artifacts

### 3.1 Data Files (`/data`)
- **`sample-employee spread sheet .xlsx`**: Initial dataset containing employee records across departments (`Servicenow`, `Salesforce`) with fields: `Employee ID`, `Name`, `Email`, `Department`, `Location`.
- **`updated sample employee spread sheet.xlsx`**: Delta dataset containing updated attributes (name/email changes) and newly hired employee records to validate coalesce and upsert behavior.
- **`Employee Analytics Dashboard.ppt`**: Presentation deck showcasing dashboard visualizations, architecture, and executive project deliverables.

### 3.2 Visual Evidence (`/screenshots`)
- **`1 - spread creation.png`**: Google Spreadsheet with source employee data.
- **`2 - table creation.png`**: Custom target table (`u_employee_test`) and form layout setup.
- **`3 - Import table .png`**: ServiceNow Load Data interface creating staging table `u_employee_import`.
- **`4 - transform creation.png`**: Transform Map `Sample Spreadsheet Import` and auto field mapping.
- **`5 - transform data and validate.png`**: Successful transform execution and validated records in `u_employee_test`.
- **`6 - Enable Coalesce to Avoid Duplicate Records.png`**: Field map with `Coalesce = true` configured on `Employee ID`.
- **`7 - inset new data.png`**: Incremental import showing insert vs update counts in transform history.
- **`8 - report.png`**: Interactive report (`Employees by Department`) grouped by department with count aggregation.
- **`9  - dashboard.png`**: Multi-visualization **Employee Analytics Dashboards** displaying department breakdown, location metrics, and data list.

---

## 4. Phase-by-Phase Implementation Guide

### Phase 1: Source Data Preparation
1. Create a structured spreadsheet in Google Sheets with the following schema:
   - `Employee ID` (e.g., `SB001`, `SB002`, `SB003`, `SB004`, `SB005`)
   - `Name` (e.g., `user1`, `user2`, `user3`, `user4`, `user5`)
   - `Email` (e.g., `user1@gmail.com`, `user2@gmail.com`, etc.)
   - `Department` (e.g., `Servicenow`, `Salesforce`)
   - `Location` (e.g., `Tamil Nadu`)
2. Export and save the file locally as `sample-employee spread sheet .xlsx`.

### Phase 2: Target Custom Table Creation
1. **Navigation:** **All > System Definition > Tables** (or `Tables > Create New`).
2. Set configuration:
   - **Label:** `Employee Test`
   - **Name:** `u_employee_test`
3. Click **Submit** or **Save**.
4. Scroll down to **Related Links** and click **Show Form**.
5. Click the **Form Context Menu (Additional Actions / Hamburger icon)** in the top-left header.
6. Navigate to **Configure > Form Layout**.
7. In the Form Layout view, under *Create new field*, add:
   - `Employee ID` — Type: `String`
   - `Employee Name` — Type: `String`
   - `Email` — Type: `String`
   - `Department` — Type: `String`
   - `Location` — Type: `String`
8. Click **Add** for each field, position them appropriately, and click **Save**.
9. Verify that all fields display accurately on the `Employee Test` form.

### Phase 3: Staging via Import Set Table
1. **Navigation:** **All > System Import Sets > Load Data**.
2. Configure import parameters:
   - **Import Set table:** Select **Create table**.
   - **Label:** `Employee Import`
   - **Name:** `u_employee_import` (auto-populated).
   - **Source of Import:** Select **File**.
   - **Choose File:** Attach `sample-employee spread sheet .xlsx`.
   - **Sheet number:** `1`
   - **Header row:** `1`
3. Click **Submit**.
4. Review the confirmation page showing state `Completed` and rows imported.
5. Under *Next Steps*, click **Create Transform Map**.

### Phase 4: Transform Map Configuration & Field Mapping
1. Configure Transform Map header:
   - **Name:** `Sample Spreadsheet Import`
   - **Source table:** `Employee Import [u_employee_import]` (pre-selected)
   - **Target table:** `Employee Test [u_employee_test]`
2. Click **Auto Map Matching fields** to pair matching column names.
3. Verify the field map alignments under the **Field Maps** related list:
   - `u_employee_id` ➔ `u_employee_id`
   - `u_name` ➔ `u_employee_name`
   - `u_email` ➔ `u_email`
   - `u_department` ➔ `u_department`
   - `u_location` ➔ `u_location`
4. Click **Save**.
5. Under Related Links, click **Transform**.
6. On the Specify Import Set and Transform Map screen, select `Sample Spreadsheet Import` and click **Transform**.
7. Confirm that the transform completes with status `Success` and all rows inserted.

### Phase 5: Transform Verification & Data Validation
1. **Navigation:** Open **Employee Test** (`u_employee_test.list`) via Application Navigator.
2. Confirm all rows from the spreadsheet have been successfully created.
3. Click the gear icon (**Personalize List Columns**) on the list header to configure display columns:
   - Add: `Employee ID`, `Employee Name`, `Email`, `Department`, `Location`.
4. Click **OK** to verify complete data fidelity.

### Phase 6: Coalesce Configuration for Deduplication
In production integrations, spreadsheets are frequently re-imported with additions or modifications. Without coalesce keys, re-imports cause duplicate records.

1. **Navigation:** **All > System Import Sets > Administration > Transform Maps**.
2. Open **`Sample Spreadsheet Import`**.
3. Scroll to the **Field Maps** related list.
4. Locate the row where **Target field** is `u_employee_id` (or `Employee ID`).
5. Double-click the **Coalesce** column for that field and toggle it from `false` to `true`.
6. Click the green checkmark / **Save** the record.
7. *Behavior:* ServiceNow will now query the target table using `Employee ID` before inserting. If a record matches, it performs an **update**; if no record matches, it performs an **insert**.

### Phase 7: Delta Load & Upsert Validation
1. Prepare `updated sample employee spread sheet.xlsx` containing:
   - Existing records with modified details (e.g., updated email addresses or names).
   - Additional new employee records with distinct IDs.
2. **Navigation:** **All > System Import Sets > Load Data**.
3. Select **Existing table**: `Employee Import [u_employee_import]`.
4. Choose `updated sample employee spread sheet.xlsx` and click **Submit**.
5. Under Next Steps, click **Run transform**.
6. Select `Sample Spreadsheet Import` and click **Transform**.
7. Open **Transform History**:
   - Check the **Inserted** count (new employee IDs).
   - Check the **Updated** count (modified existing employee IDs).
   - Verify **Errors** = 0.
8. Re-running the transform with identical data verifies idempotency:
   - State: `Completed`
   - **Inserted:** 0, **Updated:** 0, **Ignored:** Total rows.

### Phase 8: ServiceNow Reports
1. **Navigation:** **All > Reports > View / Run** (or click **Create New**).
2. Click **Create a report**.
3. **Report Configuration (Employees by Department):**
   - **Report name:** `Employees by Department`
   - **Source type:** `Table`
   - **Table:** `Employee Test [u_employee_test]`
   - Click **Next**.
   - **Type:** `Pie` chart.
   - Click **Next**.
   - **Configure:**
     - **Group by:** `Department`
     - **Aggregation:** `Count`
   - Click **Next**.
   - **Style:** Default palette.
   - Click **Run** to preview and **Save** to publish.
4. Additional complementary reports created:
   - `Employees by Location`
   - `Employee List`

### Phase 9: Consolidated Platform Analytics Dashboard
1. **Navigation:** **All > Platform Analytics > Dashboards**.
2. Click **Create dashboard**.
3. Select **In-line editor**.
4. Set name: **`Employee Analytics Dashboards`**.
5. Click **Create dashboard** to open canvas in editing mode.
6. Click **Add new element > Data visualization**.
7. Add the three target visualizations:
   - **Employees by Department** (Pie / Donut Chart)
   - **Employees by Location** (Bar / Distribution Chart)
   - **Employee List** (Interactive Records Table)
8. Position and resize widgets across the grid for optimal executive viewing.
9. Click **Save**, then click **Exit editing mode**.
10. Confirm dynamic drill-down capabilities from dashboard widgets directly into target records.

---

## 5. Summary of Results

| Metric / Checkpoint | Expected Result | Actual Result | Status |
|---------------------|-----------------|---------------|--------|
| Target Table Creation | `u_employee_test` with 5 string fields | Created with Form Layout configured | Passed |
| Import Set Load | Staging table `u_employee_import` created | Data loaded with 0 parsing errors | Passed |
| Field Mapping | 5 source columns mapped to target | Automated & verified | Passed |
| Initial Transformation | All rows inserted | Target table populated accurately | Passed |
| Coalesce on Employee ID | Prevent duplicate rows on re-run | Coalesce set to `true` | Passed |
| Delta Import | New records inserted, modified updated | Transform History validates upsert | Passed |
| Reporting | Visual breakdown of employee metrics | Pie chart created & saved | Passed |
| Executive Dashboard | Unified view of department & location data | Dashboard live in Platform Analytics | Passed |
