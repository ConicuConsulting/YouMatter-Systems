## YouMatter Identity and Access Management Technical Document

### Overview
This document outlines the identity and access management (IAM) processes for the YouMatter project. It includes a detailed flowchart depicting the interactions between various components and an overview of access control methods, roles, and policies.

### Components

- **YouMatter Systems**: Includes the various subsystems and applications that comprise the YouMatter environment.
- **Browser**: The client-side interface used by users to access YouMatter.
- **Client Application**: The application interface through which users interact with YouMatter systems.
- **Azure Active Directory**: Provides identity and access management services.
- **REST/App Service**: Handles RESTful API and application services.
- **Azure Communication Services**: Manages communication between services.
- **Azure SQL DB**: Database service storing patient records.
- **Azure AI Search**: Provides search capabilities within the YouMatter system.

### Flowchart Details

![IAM - Flowchart](https://github.com/user-attachments/assets/f2166f7b-ce38-4f33-91d6-02cabf65c732)

#### Authentication Process

1. **User Authentication via Azure AD**
   - User opens the YouMatter system through a browser.
   - The client application sends a request for authorization.
   - Azure AD handles the authentication of the user.
   - Upon successful authentication, Azure AD issues an access token for the YouMatter application.
   - The user is redirected to the YouMatter application with the access token.

2. **Get Access Token for User**
   - The YouMatter application requests an access token from the backend services.
   - The request is forwarded to the control plane, which interacts with Azure AD.
   - Azure AD returns the access token to the backend service, which is then forwarded to the client application.

#### Patient Record Access

1. **Open YouMatter Record**
   - The user requests to open a patient record within the YouMatter system.
   - The system verifies the user’s access token and permissions.

2. **Search and Retrieve Patient Record**
   - The user searches for a patient record.
   - The request is sent to Azure SQL DB.
   - The patient record is retrieved and displayed to the user.

### Identity and Access Management Overview

#### Access Control Methods

1. **ABAC (Attribute-Based Access Control)**
   - Uses attributes such as department, role, location, and time to control access.
   - Provides fine-grained access control based on dynamic conditions.

2. **RBAC (Role-Based Access Control)**
   - Manages access based on predefined roles.
   - Roles have specific permissions assigned to them.

3. **Policy-Based Access Control**
   - Uses policies to define access control rules.
   - Policies specify which actions are allowed or denied under specific conditions.

4. **Role Hierarchy**
   - Defines roles in a hierarchical structure.
   - Higher-level roles inherit permissions from lower-level roles.

5. **Separation of Duties**
   - Ensures no single individual can complete critical tasks alone.
   - Reduces the risk of fraud and error.

6. **Least Privilege Principle**
   - Users are granted the minimum access necessary to perform their duties.
   - Limits potential damage from compromised accounts.

7. **Audit and Monitoring**
   - Regularly audits access controls and user activities.
   - Monitors for suspicious activities and enforces compliance.

8. **Data Encryption**
   - Encrypts data at rest and in transit.
   - Ensures data confidentiality and integrity.

9. **Exceptions**
   - Manages exceptional access cases with additional approvals.
   - Provides flexibility for unique situations.

10. **Sensitive Fields**
    - Defines sensitive data fields with higher access control requirements.
    - Protects critical information from unauthorized access.

### Access Levels and Roles

#### Access Levels

- **YouMatter_RBAC_AccessLevel1**
  - Basic access, mostly read-only.
  - Suitable for roles requiring minimal interaction with patient data.

- **YouMatter_RBAC_AccessLevel2**
  - Intermediate access with some write permissions.
  - Suitable for nurses and allied health professionals.

- **YouMatter_RBAC_AccessLevel3**
  - Advanced access with more write and modify permissions.
  - Suitable for doctors and senior medical staff.

- **YouMatter_RBAC_AccessLevel4**
  - Full administrative access.
  - Suitable for administrators who need to manage system configurations and high-level operations.

#### RBAC Roles

- **YouMatter_RBAC_Doctor**
- **YouMatter_RBAC_Nurse**
- **YouMatter_RBAC_Admin**
- **YouMatter_RBAC_Staff**
- **YouMatter_RBAC_Researcher**

#### User Roles

- **YouMatter_RBAC_DoctorRole**
  - Access to patient records with detailed medical information.
  - Can update patient records and add notes.

- **YouMatter_RBAC_NurseRole**
  - Access to patient records with nursing notes.
  - Can update nursing notes and patient statuses.

- **YouMatter_RBAC_AdminRole**
  - Full access to all system functionalities.
  - Manages user roles and permissions.

- **YouMatter_RBAC_StaffRole**
  - Access to general patient information.
  - Can view but not update patient records.

- **YouMatter_RBAC_ResearcherRole**
  - Access to anonymized patient data for research purposes.
  - Can request access to specific data sets.

### VM_patient_Portal Schema

| Column Name       | Data Type   | Description                               |
|-------------------|-------------|-------------------------------------------|
| patient_id        | INT (Primary Key) | Unique identifier for the patient         |
| first_name        | VARCHAR(50) | Patient's first name                      |
| last_name         | VARCHAR(50) | Patient's last name                       |
| dob               | DATE        | Patient's date of birth                   |
| gender            | VARCHAR(10) | Patient's gender                          |
| address           | VARCHAR(255)| Residential address                       |
| contact_number    | VARCHAR(20) | Patient's contact number                  |
| email             | VARCHAR(100)| Patient's email address                   |
| medical_history   | TEXT        | Patient's medical history                 |
| current_conditions| VARCHAR(100)| Current medical conditions                |
| medications       | TEXT        | Current medications                       |
| allergies         | TEXT        | Known allergies                           |
| emergency_contact | VARCHAR(100)| Emergency contact information             |
| admission_date    | DATE        | Date of admission                         |
| discharge_date    | DATE        | Date of discharge (if applicable)         |
| ym_rbac           | JSON ARRAY  | Array of roles that have access to the record |
| created_by        | VARCHAR(100)| User who created the record               |
| created_at        | DATETIME    | Timestamp when the record was created     |
| modified_by       | VARCHAR(100)| User who last modified the record         |
| modified_at       | DATETIME    | Timestamp when the record was last modified|

### Example Policy for ABAC/PBAC

#### ABAC Policy Example

```python
# Check access based on patient record attributes
def check_access(user, patient_record):
    if user.role == "Doctor" and user.department == patient_record.department:
        return True
    if user.role == "Nurse" and user.shift == "Day" and patient_record.ward == "General":
        return True
    return False
```

#### PBAC Policy Example

```python
# Define policy for accessing patient records
def access_policy(user, patient_record):
    policy = {
        "allow": [
            {"role": "Doctor", "conditions": {"department": user.department}},
            {"role": "Nurse", "conditions": {"shift": "Day", "ward": "General"}}
        ],
        "deny": [
            {"role": "Admin", "conditions": {"action": "delete"}}
        ]
    }
    return policy
```

---

This document provides a comprehensive technical overview of the identity and access management processes for the YouMatter project.
