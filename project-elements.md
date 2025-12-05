## What are the elements of your project?

They are

**Users**

* **employees** – create maintenance or service requests and track progress
* **technicians** – view assigned work, update status, and resolve tasks
* **supervisors** – oversee the full workflow, assign technicians, and monitor performance

**Frontends**

* **employee webapp** (simple UI for submitting and tracking requests)
* **technician dashboard** (list of jobs, filters, status updates)
* **supervisor dashboard** (overview, assignments, light reporting)

**and Backends**

* **main API**

  * handles all requests from all frontends
  * manages core logic: creating requests, updating status, adding comments, assigning technicians
  * exposes a simple, rest-style HTTP API

* **notification worker**

  * runs in the background to send emails/alerts when something changes
  * helps keep the main API responsive (so things feel fast)

**Data store**

* **relational database** with a few main entities:

  * users
  * service_requests
  * assignments
  * comments / activity_log
  * assets or locations (maybe optional)

**External integrations**

* **email/notification provider** (simple API for sending updates)
* (optional) **SSO or identity provider** for login if needed later

---

## Which architectural patterns will apply?

* **layered monolith for the backend**

  * controllers (HTTP layer)
  * service layer (business logic)
  * data layer (DB access)
  * *(keeps things organized but still simple and deployable as one app)*

* **modular frontends**

  * separate views for each user type
  * all talk to the same backend API

* **async worker pattern**

  * notification worker sends emails without blocking user requests
  * *(basically: slow stuff happens in the background)*

---

## How will the pieces communicate with each other?

* **frontends → main api**

  * HTTPS + JSON (rest)
  * examples:

    * `POST /requests`
    * `GET /requests/{id}`
    * `PATCH /requests/{id}`
    * ...

* **main API → database**

  * standard CRUD operations

* **main API → notification worker**

  * pushes a message/job whenever something important happens
  * e.g. “request_created”, “assigned_to_tech”, “status_changed”

* **notification worker → email provider**

  * sends the actual email through an external API

so basically everything flows through the main api, the worker handles the slow tasks and the email provider handles messages

---

## What kind of authN and authZ will be needed where?

**authN (who you are)**

* simple login (email + password or optional SSO)
* users receive a session or token after signing in

**authZ (what you can do)**
role-based permissions:

* **employee**

  * create requests
  * view and comment on their own requsts

* **technician**

  * view assigned work
  * update status and add notes

* **supervisor**

  * view all requests
  * assign work
  * close or reopen requests
  * acces light reporting

**here auth is enforced**

* checked on every request at the **main API**
* API verifies role + resource ownership (ex: employees can’t access other employees’ requests)

