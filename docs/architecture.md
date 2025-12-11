# Project architecture

- Create an architecture diagram for your project.
- Include only the elements that are necessary for your MVP.
- Show your users and how they interface with the system.
- Where important, note what communication and auth strategies will be used.
- Identify what is available directly to the outside internet vs only available internally.
- Follow a clear set of conventions and include a legend.

---
![Architecture Diagram](./architecture.png)

## Legend (MVP) 
### Zones 
- public: internet-facing (frontends + main api)
- internal: backend components not exposed to users (DB + worker)
- external: third-party service (email provider) 

### How users interact
users access their apps via HTTPS, and each app sends requests to the main API using HTTPS + JSON REST. 

### Communication + auth
- main API handles all authN (login/session/token) and authZ (role checks)
- main API → DB uses SQL CRUD
- main API → worker enqueues async jobs
- worker → email provider sends notification emails

### simple for MVP:
my diagram includes: 3 user types, 3 frontends, main api, database, notification worker, email provider, maybe adding SSO further on.