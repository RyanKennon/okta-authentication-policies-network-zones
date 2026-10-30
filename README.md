# okta-authentication-policies-network-zones

## 1) Prepare Test Users and Groups

### Create Test Groups

1. In the **Okta Administrator Console** open the **Directory** dropdown then **Groups**
2. Press **Add Group** then create a group with the following information then **Save:**
   - **Name:** AuthPolicy Standard
   - **Description:** Test users for standard authentication policies
  
# Image 1

3. Create a second group with the following information:
   - **Name:** AuthPolicy HighSecurity
   - **Description:** Test users for strict authentication policies
  
# Image 2
# Image 3

### Create Test Users

1. Open the **Directory** dropdown then **People**
2. Press **Add Person** then create a user with the following information then **Save and Add Another:**
   - **User Type:** User
   - **First Name:** Jordan
   - **Last Name:** Ellis
   - **Username:** jordan.ellis@kennon.tech
   - **Primary Email:** use an email that you actually control
   - **Groups:** AuthPolicy Standard
   - **Activation:** Activate Now
   - **Password:** create password
   - **User Must Change Password on First Login:** Uncheck
  
# Image 4 Hide email

3. Create a second user with the following information:
   - **User Type:** User
   - **First Name:** Morgan
   - **Last Name:** Reyes
   - **Username:** morgan.reyes@kennon.tech
   - **Primary Email:** use an email that you actually control
   - **Groups:** AuthPolicy HighSecurity
   - **Activation:** Activate Now
   - **Password:** create password
   - **User Must Change Password on First Login:** Uncheck
  
# Image 5 hide email
# Image 6

### Create Test App

1. Open the **Applications and Resources** dropdown then select **Applications** then **Browse App Catalog**
2. Search for the **Bookmark App** then **Add Integration**
3. On the **General Settings** page enter the following infomration then press **Done:**
   - **Application Label:** Kennon Intranet
   - **URL:** https://example.com

# Image 7

4. Open the **Sign On** tab next to **User Authentication** select **Edit**
5. For the **Authentication Policy** select **Any Two Factors** then **Save**

# Image 8

6. Open the **Assignments** tab then click the **Assign** dropdown then **Assign to Groups**
7. Select **Assign for both **AuthPolicy Standard** and **AuthPolicy HighSecurity** then **Done**

# Image 9
# Image 10

### Confirm Sign In

1. Attempt to log into **Okta** as **Jordan Ellis** in an **Incgonito** browser
2. Confirm that the **Kennon Intranet** app appears on the dashboard then **Sign Out**

# Image 11

3. Attempt to log into **Okta** as **Moragn Reyes**
4. Confirm that the **Kennon Intranet** app appears on the dashboard then **Sign Out**

# Image 12

### Verify in the System Log

1. Open the **Okta Administrator Console** open the **Reports** dropdown then **System Log**
2. Use the following search prompt then confirm **Create Users** event appears:
   `eventType eq "user.lifecycle.create" and target.alternateId eq "jordan.ellis@kennon.tech"`

# Image 13
# Image 14

3. Use the following search prompt then confirm **Add to Group** event appears:
   `eventType eq "group.user_membership.add" and target.alternateId eq "jordan.ellis@kennon.tech"`

# Image 15
# Image 16

4. Use the following search prompt then confirm **Add to Application** event appears:
   `eventType eq "application.user_membership.add" and target.alternateId eq "jordan.ellis@kennon.tech"`

# Image 17
# Image 18

5. Use the following search prompt then confirm **Login Success** event appears:
   `eventType eq "user.session.start" and actor.alternateId eq "jordan.ellis@kennon.tech"`

# Image 19
# Image 20

6. Use the following search prompt then confirm **Evaluation of Sign On Policy** event appears:
   `eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "jordan.ellis@kennon.tech"`

# Image 21
# Image 22

7. Use the following search prompt then confirm **User Sign On to App** event appears:
   `eventType eq "user.authentication.sso" and actor.alternateId eq "jordan.ellis@kennon.tech"`

# Image 23
# Image 24

---

## 2) Configure Authenticators and Password Policy
























