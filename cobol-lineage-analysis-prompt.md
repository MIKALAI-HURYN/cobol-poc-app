# COBOL Data Lineage Analysis Prompt

## Objective

Analyze COBOL programs to extract and display field-level data lineage mappings in a clear text format. Show how data flows from source tables/columns to target tables/columns, including transformation logic.

## Analysis Requirements

### 1. Identify Source and Target Tables

Analyze the COBOL code to identify:

- **Source tables**: From SELECT statements in EXEC SQL blocks
- **Target tables**: From INSERT or UPDATE statements in EXEC SQL blocks
- **Table columns**: Extract all column names from SQL statements

### 2. Trace Field Mappings

For each target field in INSERT/UPDATE statements, trace back through the PROCEDURE DIVISION to find:

- Which source field(s) populate it
- What transformations are applied (MOVE, STRING, EVALUATE, COMPUTE, etc.)
- The complete transformation logic

### 3. Code Patterns to Analyze

#### MOVE Statements - Direct Mapping

```cobol
MOVE WS-ACCT-ID TO TGT-ACCT-ID
MOVE WS-BALANCE TO TGT-AMOUNT
```

#### STRING Statements - Concatenation

```cobol
STRING 'Customer: ' DELIMITED BY SIZE
       WS-CUST-NAME DELIMITED BY SPACE
       ' - Account Inquiry' DELIMITED BY SIZE
       INTO TGT-DESC
END-STRING
```

#### EVALUATE Statements - Conditional Logic

```cobol
EVALUATE WS-ACCT-TYPE
    WHEN 'Checking'  MOVE 'CHK-Inquiry' TO TGT-SUBTYPE
    WHEN 'Savings'   MOVE 'SAV-Inquiry' TO TGT-SUBTYPE
    WHEN 'Investment' MOVE 'INV-Inquiry' TO TGT-SUBTYPE
END-EVALUATE
```

#### COMPUTE - Mathematical Operations

```cobol
COMPUTE TGT-TOTAL = WS-AMOUNT * WS-RATE
```

## Output Format

Generate a text report with clear field-to-field mappings in the following format:

```
PROGRAM: [PROGRAM-ID]
DESCRIPTION: [Program description from comments]

SOURCE → TARGET MAPPINGS:
========================

[SourceTable].[SourceColumn] → [TargetTable].[TargetColumn]
  Transformation: [Direct mapping | Transformation description]

[SourceTable].[SourceColumn] → [TargetTable].[TargetColumn]
  Transformation: [Transformation description]
```

### Example Output

```
PROGRAM: ACCTXFRM
DESCRIPTION: Data transformation from CustomerAccounts to AccountTransactions table

SOURCE → TARGET MAPPINGS:
========================

CustomerAccounts.AccountID → AccountTransactions.AccountID
  Transformation: Direct mapping

CustomerAccounts.CustomerID → AccountTransactions.CustomerID
  Transformation: Direct mapping

CustomerAccounts.AccountNumber → AccountTransactions.AccountNumber
  Transformation: Direct mapping

CustomerAccounts.CustomerName → AccountTransactions.Description
  Transformation: STRING 'Customer: ' + CustomerName + ' - Account Inquiry'

CustomerAccounts.AccountType → AccountTransactions.TransactionSubType
  Transformation: Checking→CHK-Inquiry, Savings→SAV-Inquiry, Investment→INV-Inquiry, Credit→CRD-Inquiry, Other→GEN-Inquiry

CustomerAccounts.Balance → AccountTransactions.Amount
  Transformation: Direct mapping

CustomerAccounts.Balance → AccountTransactions.RunningBalance
  Transformation: Direct mapping

CustomerAccounts.BranchCode → AccountTransactions.BranchCode
  Transformation: Direct mapping

CustomerAccounts.RiskScore → AccountTransactions.ReferenceNumber
CustomerAccounts.AccountNumber → AccountTransactions.ReferenceNumber
  Transformation: STRING 'RSK-' + RiskScore + '-' + AccountNumber
```

## Analysis Steps

1. **Extract Program Info**
   - Get PROGRAM-ID from IDENTIFICATION DIVISION
   - Read program description from comments

2. **Find Source Tables and Columns**
   - Parse all SELECT statements in EXEC SQL blocks
   - Extract table names and column lists

3. **Find Target Tables and Columns**
   - Parse all INSERT/UPDATE statements in EXEC SQL blocks
   - Extract table names and column lists

4. **Map Each Target Column to Source**
   - For each target column in INSERT/UPDATE:
     - Find the COBOL variable being inserted
     - Trace backwards through PROCEDURE DIVISION
     - Find all MOVE, STRING, EVALUATE, COMPUTE statements affecting that variable
     - Identify the source column(s) and transformation logic

5. **Generate Text Output**
   - List program information
   - Show each source→target mapping with transformation description
   - Group related mappings together

## Mapping Rules

- **One-to-one mapping**: Single source field to single target field
- **One-to-many mapping**: Same source field populates multiple target fields (show each separately)
- **Many-to-one mapping**: Multiple source fields combine into one target field (list all sources)
- **Transformation description**: Clearly explain the logic in plain text

## Additional Guidance

- Use actual table and column names from SQL statements
- Preserve case as it appears in the code
- For complex transformations, summarize the logic concisely
- If transformation logic is unclear, note it as "Complex transformation - review code"
- Include line numbers or code references if helpful for verification
