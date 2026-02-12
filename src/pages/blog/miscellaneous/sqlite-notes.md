---
title: "SQLite Notes"
date: "2026-01-28"
description: "Quick reference for SQLite commands in the terminal"
---

run sqlite in terminal:

```jsx
sqlite3 mydatabase.db
```

replace `mydatabase.db`  with whatever your own db file is

example:

```jsx
➜  directory_name git:(new-schema) sqlite3 main.db
SQLite version 3.43.2 2023-10-10 13:08:14
Enter ".help" for usage hints.
sqlite>
sqlite>
sqlite>
```

---

show tables in the database

```jsx
.tables
```

show schema of a table

```jsx
.schema peptides
```

---

SQL queries

```jsx
SELECT * FROM functions;
```

```jsx
SELECT * FROM functions LIMIT 10;
```

```jsx
SELECT * FROM functions WHERE name like '%ACE%';
```

> 2|ACE Inhibitor
52|ACE Inhibitor pepitde
128|ACE Inhibitor and Antioxidant
129|ACE Inhibitor and Antihypertensive
130|ACE Inhibitor and Opioid
131|ACE Inhibitor and Anxiolytic-like
132|ACE Inhibitor and DPP IV-inhibitory
133|ACE Inhibitor and Immunomodulatory
134|ACE Inhibitor and Antithrombotic
135|ACE Inhibitor and Anticancer
136|ACE Inhibitor and PEP-inhibitory
137|ACE Inhibitor and Antimicrobial
138|ACE Inhibitor and Other bioactivity
139|ACE Inhibitor and Renin-inhibitory
140|ACE Inhibitor and Hypocholesterolemic
141|ACE Inhibitor and Neuropeptide
142|ACE Inhibitor and Vasorelaxation
143|ACE Inhibitor and Vasoconstriction
144|ACE Inhibitor and α-Amylase inhibitory
145|ACE Inhibitor and α-Glucosidase inhibitory
188|ACE-inhibitor
> 

```jsx
SELECT * FROM peptide_function WHERE function_id = 52;
```

> 1395|52
> 

delete table

```jsx
DROP TABLE table_name;
```

---

exit

```jsx
.exit
```