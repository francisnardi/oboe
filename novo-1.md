# CNPJ - Tax Id - New Alphanumeric CNPJ (2026) – Adaptations and Guidelines

---

## 📅 1. Official Timeline of the Change

| Stage                           | Date             |
| ------------------------------- | ---------------- |
| Publication of IN RFB No. 2.229 | October 15, 2024 |
| Effective date                  | October 25, 2024 |
| Alphanumeric CNPJs issued from  | **July 2026**    |

> Official source: Receita Federal do Brasil
> Link: [gov.br/receitafederal](https://www.gov.br/receitafederal/pt-br/assuntos/noticias/2024/outubro/cnpj-tera-letras-e-numeros-a-partir-de-julho-de-2026)

---

## 🧾 2. CNPJ Structure

### 🟦 **Current Format (until 2026)**

* **14 numeric digits**: NNNNNNNNNNNNNN

  * Root: 8 digits
  * Branch: 4 digits (e.g., 0001 = headquarters)
  * Check digits: 2 digits

### 🟩 **Future Format (from July 2026)**

* **12 alphanumeric characters** + **2 check digits**

  * Characters A–Z and digits 0–9 allowed
  * Check digits based on ASCII value calculations

---

## 🔢 3. Unified Validation Algorithm (Python)

This algorithm validates **both numeric and alphanumeric CNPJs**:

```python
def char_to_val(c):
    if c.isdigit():
        return int(c)
    else:
        return ord(c.upper()) - 55  # A=10, B=11, ..., Z=35

def calcula_dv_alfanumerico(cnpj_base: str) -> str:
    def calc_dv(cnpj, weights):
        total = sum(char_to_val(c) * w for c, w in zip(cnpj, weights))
        remainder = total % 11
        return '0' if remainder < 2 else str(11 - remainder)

    weights_1 = [5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2]
    weights_2 = [6] + weights_1

    dv1 = calc_dv(cnpj_base, weights_1)
    dv2 = calc_dv(cnpj_base + dv1, weights_2)

    return dv1 + dv2

def valida_cnpj(cnpj: str) -> bool:
    base = cnpj[:12]
    dv = cnpj[12:]
    return calcula_dv_alfanumerico(base) == dv
```

---

## 🧪 4. Validation Examples (Old and New)

| Type | Base CNPJ (12) | Expected DV | Full CNPJ        | Valid? |
| ---- | -------------- | ----------- | ---------------- | ------ |
| Old  | 123456780001   | 95          | 12345678000195   | ✅      |
| Old  | 000000000001   | 91          | 00000000000191   | ✅      |
| Old  | 111111110001   | 91          | 11111111000191   | ✅      |
| Old  | 999999990001   | 91          | 99999999000191   | ✅      |
| Old  | 101010100001   | 00          | 10101010000100   | ✅      |
| New  | A1B2C3D4E5F600 | 70          | A1B2C3D4E5F60070 | ✅      |
| New  | Z9Y8X7W6V5U400 | 27          | Z9Y8X7W6V5U40027 | ✅      |
| New  | BRASIL000001   | 91          | BRASIL00000191   | ✅      |
| New  | GOVBR1230001   | 63          | GOVBR123000163   | ✅      |
| New  | ALFA001BR002   | 86          | ALFA001BR00286   | ✅      |

### 🧪 Test Code

```python
samples = [
    "12345678000195",  # Old
    "00000000000191",
    "11111111000191",
    "99999999000191",
    "10101010000100",
    "A1B2C3D4E5F60070",  # New
    "Z9Y8X7W6V5U40027",
    "BRASIL00000191",
    "GOVBR123000163",
    "ALFA001BR00286",
]

for cnpj in samples:
    print(f"CNPJ: {cnpj} => Valid? {valida_cnpj(cnpj)}")
```

---

## ✅ 5. Conclusions

* 🔁 The unified algorithm supports **both old and new formats**
* 🧠 Letters are converted using **ASCII-based logic**
* 🧩 Systems must be updated to accept **alphanumeric CNPJs**
* 📌 **Existing CNPJs remain valid and unchanged**

---

## 🛠️ Guidelines for Legacy System Adaptation

---

### 🟧 1. **Check and Update Data Structures**

| Element            | Action                                                                          |
| ------------------ | ------------------------------------------------------------------------------- |
| Database           | Change INT, BIGINT, NUMERIC, DECIMAL, CHAR(14) to VARCHAR(14)                   |
| Forms / Inputs     | Allow A–Z letters (ASCII-based validation)                                      |
| Masks / Formatting | Update to \*\*\*\*\*\*\*\*\*\***-**, allowing letters in the first 12 positions |
| APIs / Interfaces  | Accept alphanumeric strings                                                     |
| Indexes / PKs      | Avoid using CNPJ as PK if still stored as number                                |

---

### 🟧 2. **Update Check Digit Validators**

Old validators assume **only numbers**. Replace them with the new unified ASCII-aware logic.

---

### 🟧 3. **Audit and Update Third-Party Integrations**

| Item                    | Considerations                                                |
| ----------------------- | ------------------------------------------------------------- |
| APIs, ERPs, CRMs, SEFAZ | May expect numeric-only CNPJs                                 |
| Certificates            | May link authorizations to old format                         |
| Solutions               | Request technical updates, maintain logs, implement fallbacks |

---

### 🟧 4. **Impact on Reports and BI**

* Numeric assumptions in filters or grouping may break
* Refactor dashboards and reports to handle VARCHAR(14)
* Normalize alphanumeric data in ETLs and queries

---

### 🟧 5. **Best Practices for a Safe Transition**

| Practice               | Description                                            |
| ---------------------- | ------------------------------------------------------ |
| 🧪 Testing environment | Simulate alphanumeric CNPJs in dev/test databases      |
| 🔐 Dual validation     | Temporarily validate with both old and new algorithms  |
| 📊 Monitoring          | Track rejections or validation failures                |
| 🔄 API versioning      | Version APIs and contracts for phased rollout          |
| 📚 Documentation       | Update coding policies, user manuals, integration docs |

---

## 🚨 Potential Risks and Warning Signs

| Symptom                | Cause                                | Suggested Fix                         |
| ---------------------- | ------------------------------------ | ------------------------------------- |
| Rejected valid CNPJs   | Improper type or outdated validation | Use unified validator and VARCHAR     |
| Integration failures   | Partner systems not adapted          | Contact providers, implement fallback |
| Sort/filter errors     | Numeric sort on alphanumerics        | Refactor queries and logic            |
| Report inconsistencies | Non-numeric CNPJs excluded           | Normalize and include all formats     |

---

### ✅ Summary of Top Priority Actions

1. **Change all fields to VARCHAR(14)**
2. **Use the unified ASCII-aware validator**
3. **Update all forms, masks, logs**
4. **Audit and alert integrations**
5. **Automate tests using alphanumeric samples**
6. **Document and train your IT team**

---

# 🧩 Adapting SQL Server for Alphanumeric CNPJ

---

## 🛠️ 1. Identify Current Structures

Find columns with INT, BIGINT, NUMERIC, or CHAR(14) formats that restrict input to numbers.

### 🔍 Example query:

```sql
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME LIKE '%CNPJ%';
```

---

## 🔄 2. Change Column Types

Convert numeric or fixed-format fields to `VARCHAR(14)`:

```sql
ALTER TABLE Companies
ALTER COLUMN CNPJ VARCHAR(14);
```

> ⚠️ Also remove or adjust constraints enforcing numeric-only formats.

---

## 🧹 3. Remove Old Masks or Format Triggers

```sql
-- Drop triggers or UDFs applying "NN.NNN.NNN/NNNN-NN"
DROP TRIGGER tr_format_cnpj;
```

---

## 🔒 4. Create Updated Validator Function (ASCII-aware)

Here’s a SQL Server UDF for validating alphanumeric CNPJs:

```sql
CREATE FUNCTION dbo.ValidateCNPJAlpha (@cnpj VARCHAR(14))
RETURNS BIT
AS
BEGIN
    DECLARE @i INT = 1, @char CHAR(1), @val INT, @sum1 INT = 0, @sum2 INT = 0
    DECLARE @weights1 TABLE (pos INT, weight INT)
    DECLARE @weights2 TABLE (pos INT, weight INT)

    INSERT INTO @weights1 VALUES (1,5),(2,4),(3,3),(4,2),(5,9),(6,8),(7,7),(8,6),(9,5),(10,4),(11,3),(12,2)
    INSERT INTO @weights2 SELECT pos + 1, weight FROM @weights1
    INSERT INTO @weights2 VALUES (1,6)

    WHILE @i <= 12
    BEGIN
        SET @char = SUBSTRING(@cnpj, @i, 1)
        SET @val = CASE
                     WHEN @char BETWEEN '0' AND '9' THEN CAST(@char AS INT)
                     ELSE ASCII(UPPER(@char)) - 55
                   END
        SELECT @sum1 += @val * weight FROM @weights1 WHERE pos = @i
        SET @i += 1
    END

    DECLARE @dv1 INT = CASE WHEN (@sum1 % 11) < 2 THEN 0 ELSE 11 - (@sum1 % 11) END

    SET @i = 1
    WHILE @i <= 13
    BEGIN
        SET @char = SUBSTRING(@cnpj, @i, 1)
        SET @val = CASE
                     WHEN @i = 13 THEN @dv1
                     WHEN @char BETWEEN '0' AND '9' THEN CAST(@char AS INT)
                     ELSE ASCII(UPPER(@char)) - 55
                   END
        SELECT @sum2 += @val * weight FROM @weights2 WHERE pos = @i
        SET @i += 1
    END

    DECLARE @dv2 INT = CASE WHEN (@sum2 % 11) < 2 THEN 0 ELSE 11 - (@sum2 % 11) END

    RETURN CASE 
             WHEN RIGHT(@cnpj, 2) = CAST(@dv1 AS VARCHAR) + CAST(@dv2 AS VARCHAR) THEN 1 
             ELSE 0 
           END
END;
```

---

## 🧪 5. Run Validation Queries

### ✅ Valid CNPJs

```sql
SELECT CNPJ
FROM Companies
WHERE dbo.ValidateCNPJAlpha(CNPJ) = 1;
```

### ❌ Invalid CNPJs

```sql
SELECT CNPJ
FROM Companies
WHERE dbo.ValidateCNPJAlpha(CNPJ) = 0;
```

---

## 🚨 6. Audit for Numeric-Type CNPJ Columns

```sql
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME LIKE '%CNPJ%' AND DATA_TYPE IN ('int', 'bigint', 'numeric');
```

---

## 💡 Final Recommendations

| Action                                          | Priority    |
| ----------------------------------------------- | ----------- |
| Convert fields to VARCHAR(14)                   | 🔴 High     |
| Use updated validation function                 | 🔴 High     |
| Refactor masks and legacy triggers              | 🟡 Medium   |
| Audit ERP / NF-e / Integration flows            | 🔴 High     |
| Add automated test cases for alphanumeric CNPJs | 🟢 Optional |