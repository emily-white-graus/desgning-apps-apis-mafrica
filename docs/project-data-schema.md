## Project data schema (mvp)

```mermaid
flowchart LR

  %% boundary = one database in MVP
  subgraph SQLDB["Relational database (SQL)"]
    U[Users\n- id (pk)\n- email (unique)\n- role (employee|technician|supervisor)\n- created_at]
    L[Locations\n- id (pk)\n- name\n- building\n- floor]
    R[ServiceRequests\n- id (pk)\n- title\n- description\n- status (open|in_progress|resolved)\n- priority\n- created_by_user_id (fk)\n- location_id (fk)\n- created_at]
    A[Assignments\n- id (pk)\n- request_id (fk)\n- technician_user_id (fk)\n- assigned_by_user_id (fk)\n- assigned_at]
    C[Comments\n- id (pk)\n- request_id (fk)\n- author_user_id (fk)\n- body\n- created_at]
  end

  %% 1-n / n-1 (single-ended arrows with labels)
  U -->|1..n creates| R
  L -->|1..n has| R

  R -->|1..n has| C
  U -->|1..n writes| C

  %% n-n (double-ended arrow via join table)
  U <--> |n..n via Assignments| R

  %% also show the join table explicitly (each side is 1..n)
  R -->|1..n has| A
  U -->|1..n assigned to (technician)| A
  U -->|1..n assigned by (supervisor)| A
```

### Notes

* all MVP data is in one boundary: so a single **relational (sql) database**.
* **1..n** relationships use single-ended arrows
* **n..n** is shown as a double ended arrow between
