# 5. Project implementation teams

---

## Team structure and ownership

### MVP team setup

for the mvp, i’d keep the team **very small** to reduce coordination overhead.

#### how many teams?

* **1 cross-functional team**

#### how many devs?

* **3 developers total**

#### Roles and responsibilities

* **frontend developer (1)**

  * employee app, technician dashboard, supervisor dashboard
  * basic ui flows (create request, assign, update status)
  * integrates with main api over https + json

* **backend developer (1)**

  * main api (controllers, service logic, authn/authz)
  * database schema + migrations
  * core workflows (requests, assignments, comments)

* **backend / platform developer (1)**

  * notification worker
  * email provider integration
  * basic deployment setup
  * simple logging and error handling

*(everyone has freedom to look at eachothers work and cooperate)*

---

### Why this works for the mvp

* Mafricafix MVP is a layered monolith, so splitting into many teams would slow things down
* async work (notifications) is simple enough to be handled by one dev
* UI is straightforward and doesn’t need a separate mobile or design-heavy team
* decisions stay fast and communication is easy

---

## Phase I team structure

once we move to **phase I** (sso, job queue, attachments, monitoring), the team needs a bit more structure.

#### how many teams?

* **2 teams**

#### total devs?

* **5–6 developers**

---

### Team 1: product & core api team (3–4 devs)

**responsibilities**

* frontend apps (employee, technician, supervisor)
* main api business logic
* auth integration (sso)
* request lifecycle, assignments, permissions
* attachments metadata (db side)

**why**

* this team owns everything directly visible to users
* they move based on product feedback and workflows

---

### Team 2: platform & reliability team (2 devs)

**responsibilities**

* job queue and worker reliability
* retries, failure handling
* monitoring and logs
* email delivery guarantees
* file storage integration (infrastructure side)

**why**

* phase I introduces real operational risk
* separating this work keeps product devs focused and prevents outages

---

## How team structure might evolve later

### Signals to change structure

* frequent outages or missed notifications → invest more in platform team
* many feature requests from supervisors → expand product team
* multiple facilities or tenants → split API ownership by domain

### Possible future teams

* **mobile team** (technician mobile app)
* **reporting and analytics team**
* **enterprise platform team** (multi-tenant, audit logs, compliance)
