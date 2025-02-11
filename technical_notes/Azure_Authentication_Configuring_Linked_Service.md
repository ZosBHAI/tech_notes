---
date: 2025-02-02
---
# Authentication in Azure
#Azure #Concept 
## Authentication Methods
- **Account Key**: Similar to Access Key in AWS.
- **Azure Key Vault**: Securely store and manage secrets, keys, and certificates.
- **Service Principal**: 
  - Acts as an identity created for an application to access resources.
  - Uses **Client ID** and **Client Secret** for authentication.
- **Managed Identity**: Built-in identity for Azure services to authenticate without storing secrets.
- **RBAC (Role-Based Access Control)**:
  - Example Role: **Contributor**

---

## Linked Service Authentication Methods

### **System Assigned Managed Identity**
*(Once the Data Factory is deleted, the managed identity is also deleted.)*
1. Navigate to **Managed Identities** in Data Factory/Storage Account.
2. Select the **Scope** (e.g., Resource Group or Storage).
3. Assign **Blob Storage Contributor** role (important for access).

---

### **User Assigned Managed Identity (UAMI)**
1. **Create a User Assigned Managed Identity** (search for it in Azure Portal).
2. Go to **Data Factory** and **assign the UAMI** to the Data Factory service.
3. Assign **Storage Blob Contributor** role to UAMI in **Storage Account**.
4. **Create Credentials** in ADF to store UAMI.
5. **Publish** the changes.
6. Once successfully published, select the **created credentials** in **Linked Service**.

---

### **Service Principal (For Connecting to Resources Outside Azure)**
1. Navigate to **App Registrations** in Azure AD and create an **App**.
2. **Create a Key and Secret** for the app and **save the VALUE** (as it will be masked once refreshed).
3. Assign **Storage Blob Contributor Role** to the **Service Principal** in the **Storage Account**.
4. In **ADF Linked Service**:
   - Choose **Inline Authentication**.
   - Enter **Application ID → Client ID**.
   - Select **Service Principal Key** and paste the **saved secret value**.