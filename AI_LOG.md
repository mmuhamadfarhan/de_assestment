# Major Prompts and Tools Used
## 1. Tool Used: Gemini 
### Key Prompts Ran:
- "What happen if the transaction_time is more than due_date? "
- "my data dont have expected_amounts. So, we need to rely base on data we had.Can you find other solution?"
- "Can you ellaborate how standard deviation formula can find incossistent payment?"
- "One of the column my dataframe using json structure.Please advice how to manipulate that column using struct type?"
- "What happen if rate pct moves higher or lower? Can you explain??"
## 2. Examples of AI Errors, Gaps, or Inaccuracies
You are required to show at least two instances where the AI fell short and how you, as the engineer, caught it.
### Example 1: The loan_product Blindspot (Hallucinated Schema)
- The AI Error: The AI provided an advanced aggregation script structured around a column named loan_product.
- How I trace It: When executing the code against the raw data, Spark threw an UNRESOLVED_COLUMN runtime exception. I audited the raw transaction schema provided by the source data and verified that loan_product did not exist.
- The Resolution: I corrected the AI by pointing out the missing column. We cross-referenced the active data, discovered the column was actually named product_type, and updated the code accordingly.
### Example 2: Datatype Mismatch on Semi-Structured JSON (Struct vs. String)
- The AI Error: To find data completeness anomalies, the AI suggested checking for missing values using when(col("metadata") == "null", 1).
- How I Caught It: Spark threw a DATATYPE_MISMATCH compiler error. Because metadata is a complex STRUCT (JSON object) in our schema, it cannot be directly compared to a flat string literal ("null").
- The Resolution: I corrected the coded and forced the validation to strictly utilize the .isNull() method, which safely evaluates distributed structural objects.
## 3. Architectural / Design Decisions Overridden
- The AI Suggestion: When asked to save the final computed delinquency report to my local workspace folder path (/Workspace/Users/...), the AI recommended using the local file system protocol prefix (file:/Workspace/...).
- My Override and Why: The AI's approach ignored the operational realities of a enterprise cloud cluster environment. Running that code on a cluster with Shared Access Mode / Unity Catalog security policies triggered a severe java.io.IOException because worker nodes are restricted from making local directories on the workspace control plane.
- The Action Taken: I rewrite the path layout and redirected the storage architecture to utilize native cloud object storage via DBFS (dbfs:/FileStore/tables/...) and registered it as a managed workspace Delta table. This ensured the pipeline complied with modern data lakehouse best practices.
### 4. Engineering Reflection

In this assignment, I trusted the AI to support data manipulation, mathematical definitions, and high-level syntax structures. For instance, translating business logic into a chain of PySpark functions like datediff() and handling standard deviation calculations for behavioral profiles to saved significant implementation time.
However, I strictly refused to trust the AI when it came to understanding my underlying data schema, data types and cluster infrastructure security rules. AI tools easily to generate code based on assumptions, frequently inventing columns or misinterpreting structural data types like JSON Structs. 
At the end, this exercise reinforced that while an AI tools is an incredibly efficient execution companion for writing structural code blocks quickly and advisor,but it cannot replace an engineer's and domain expert role in understanding data,schema validation, debugging system-level stack traces, and enforcing cloud infrastructure realities. AI collabration are require to encourage our mindset to become code reviewer.