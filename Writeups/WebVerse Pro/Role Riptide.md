# Role Riptide

### Lab Link
[WebVerse - Role Riptide](https://dashboard.webverselabs-pro.com/academy/mass-assignment/practice/lab/role-riptide?back=%2Facademy%2Fmass-assignment%3Ftab%3Doverview)

---
### Skills & Knowledge
- Mass Assignment
- API Docs/Schema 

---

### Executive Summary:
Successfully exploited a Mass Assignment vulnerability combined with improper access controls on the application, escalating privileges from a standard user to an administrator and uncovering the hidden `WEBVERSE{...} flag`.

### Initial Access & Reconnaissance: 
Registered a standard user account on the target platform (roleriptide.local) and navigated to the user profile settings page to examine how data parameters are handled.

### Privilege Escalation via Mass Assignment: 
Intercepted the account update request using Caido proxy and observed a hidden or unrendered `role: user` parameter within the JSON request body.

```
{
    "name":"Unknown",
    "email":"unknown@email.com",
    "phone":"",
    "password":"",
    "role":"user"
}
```

### Parameter Manipulation: 
Modified the role parameter value from `user` to `admin` in transit, which the backend trustingly bound to the database model, granting elevated administrative privileges and exposing the restricted `Classified Projects` dashboard.

```
{
    "name":"Unknown",
    "email":"unknown@email.com",
    "phone":"",
    "password":"",
    "role":"admin"
}
```

### API Schema Discovery: 
Realized standard dashboard views only displayed limited records, and systematically tested and checked standard auto-generated API documentation endpoints—including `/swagger.json,` `/openapi.json`, `/api/docs`, and `/v1/api-docs` to map the backend schema and uncover hidden routes when standard dashboard views returned only limited records.

```
/swagger.json
/openapi.json
/api/docs
/v1/api-docs
```
#### Alternatively
- `SecLists API Endpoints:` `SecLists/Discovery/Web-Content/api/api-endpoints.txt` contains a comprehensive catalog of common REST and API paths, including various iterations of Swagger UI routes, JSON schemas, and documentation endpoints.

- `SecLists Web Content:` `SecLists/Discovery/Web-Content/common.txt and raft-medium-directories.txt` feature common root-level administrative files, folders, and documentation directories like /swagger.json, /openapi.json, and /docs.

### Flag Retrieval: 
Navigated to `/openapi.json`, which exposed the API schema and revealed hidden administrative routes, specifically accessing `/api/v1/admin/config`, which returned the `flag: WEBVERSE{...}`.