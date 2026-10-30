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

### Configure Okta Verify

1. In the **Okta Administrator Console** open the **Security** dropdown then **Authenticators**
2. To the right of **Okta Verify** open the **Actions** dropdown then **Edit**
3. Under **Verification Options** make sure **TOTP** and **Push Notifications** are checked and **Okta FastPass** is unchecked

# Image 25

4. Under **Push Notification: Number Challenge** make sure **All Push Challenges** is selected then **Save**

# Image 26

### Create HighSecurity Password Policy

1. In the **Okta Administrator Console** open the **Security** dropdown then **Authenticators**
2. To the right of **Password** open the **Actions** dropdown then **Edit**
3. Click **Add New Password Policy** then create a policy with the following information then **Create Policy** then **Create Rule:**

   - **General Information**
      - **Policy Name:** HighSecurity Password Policy
      - **Policy Description:** Stricter password requirements for high security users
      - **Add Group:** AuthPolicy HighSecurity
      - **Authentication Providers:** Okta
    
# Image 27

   - **Password Requirements**
      - **Minimum Length:** 14
      - **Complexity Requirements:** check lower case, upper case letters, number, and symbol
      - **Restrict Use of Common Passwords:** checked
      - **Does not Contain Part of Username / First Name / Last Name:** Checked
    
# Image 28

   - **Password Security**
      - **Enforce Password History for Last:** 5 passwords
      - **Minimum Password Age:** Uncheck
      - **Password Expires After:** 90 days

# Image 29

   - **Rule Settings:** HighSecurity Password Rule

# Image 30

### Test HighSecurity Password Policy

1. Log in to **Okta** as **Morgan.Reyes** in an **Incgonito** browser
2. When asked choose **Get a Push Notification** to confirm its active then finish logging in

# Image 31

3. Click **Morgan's Name** in the top right of the screen then **Account Settings**
4. Under **Security Methods** find **Password** then **Reset**

# Image 32

5. Enter `Password1` as the password and confirm that it has been rejected

# Image 33

6. Create a password that meets all requirements then reset password and sign out

### Test Account Lockout

1. Attempt to log into **Okta** as **Morgan.Reyes** with an **Incorrect Password** in an **Incgonito** browser until it no longer allows a login using a password
2. In the **Okta Administrator Console** open the **Directory** dropdown then **People**
3. Open **Morgan Reyes** profile then confirm that **Morgan Reyes** has the **Locked Out** status

# Image 34

4. Press **Unlock Account** and confirm that **Morgan Reyes** has the **Active** status

# Image 35

5. Back in the **Incognito** browser confirm successfully log into **Okta** as **Morgan Reyes** using a password then sign out

### Confirm User not in Group is Unaffected

1. Log in to **Okta** as **Jordan.Ellis** in an **Incgonito** browser
2. Click **Jordan's Name** in the top right of the screen then **Account Settings**
3. Under **Security Methods** find **Password** then **Reset**
4. Confirm less password requirements appear

# Image 36

5. Sign out of Jordan's account

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown then **System Log**
2. Use the following search prompt then confirm **Password Policy Creation** event appears:
   `eventType eq "policy.lifecycle.create"`

# Image 37
# Image 38

3. Use the following search prompt then confirm **Add Policy Rule** event appears:
   `eventType eq "policy.rule.add"`

# Image 39
# Image 40

4. Use the following search prompt then confirm **Morgan's Password Change** event appears:
   `eventType eq "user.account.update_password" and target.alternateId eq "morgan.reyes@kennon.tech"`

# Image 41
# Image 42

5. Use the following search prompt then confirm **Morgan's Account Lock Out** event appears:
   `eventType eq "user.account.lock" and target.alternateId eq "morgan.reyes@kennon.tech"`

# Image 43
# Image 44

6. Use the following search prompt then confirm **Morgan's Account Unlock** event appears:
   `eventType eq "user.account.unlock_by_admin" and target.alternateId eq "morgan.reyes@kennon.tech"`

# Image 45
# Image 46

### Reset Okta Verify for Both Users

1. In the **Okta Administrator Console** open the **Directory** dropdown then **People**
2. Select **Morgan Reyes** open the **More Actions** dropdown then select **Reset Authenticators**

# Image 47

3. Select **Okta Verify** then **Reset Selected Authenticators**
4. Repeat with **Jordan Ellis**

---

## 3) Authenticator Enrollment Policies

### Create Standard Enrollment Policy

1. In the **Okta Administration Console** open the **Security** dropdown then **Authenticators**
2. Open the **Enrollment** tab then **Add a Policy**
3. Create a Policy with the following information then **Create Policy:**

   - **Policy Name:** Standard Enrollment Policy
   - **Policy Description:** Okta Verify optional for standard users
   - **Assign to Groups:** AuthPolicy Standard
   - **Email Enrollment:** Optional
   - **Okta Verify Enrollment:** Optional
   - **Password Enrollment:** Required

# Image 49

4. On the **Add Rule** Page enter the following information then **Create Rule:**

   - **Rule Name:** Standard Employee Rule
   - **If User's IP is:** Anywhere
   - **And User is Accessing:** Okta or Any App
   - **Then Enroll in Authenticators:** Allowed for All Authenticators

# Image 50

### Create HighSecurity Enrollment Policy

1. Create a second Policy with the following information then **Create Policy:**

   - **Policy Name:** HighSecurity Enrollment Policy
   - **Policy Description:** Okta Verify required for high security users
   - **Assign to Groups:** AuthPolicy HighSecurity
   - **Email Enrollment:** Optional
   - **Okta Verify Enrollment:** Required
   - **Password Enrollment:** Required

# Image 51

2. On the **Add Rule** Page enter the following information then **Create Rule:**

   - **Rule Name:** HighSecurity Enrollment Rule
   - **If User's IP is:** Anywhere
   - **And User is Accessing:** Okta or Any App
   - **Then Enroll in Authenticators:** Allowed for All Authenticators

# Image 52

### Add a Standard Rule to the Okta Dashboard Policy

1. Open the **Security** dropdown then **Authentication Policies** and select **Okta Dashboard**
2. Click **Add rule** and configure the following:

   - **Rule name:** `Standard Dashboard Rule`
   - **AND User's group membership includes:** At least one of the following groups, **AuthPolicy Standard**
   - **THEN Access is:** Allowed after successful authentication
   - **User must authenticate with:** Any 2 factor types
   - **Authentication methods:** Allow any method that can be used to meet the requirement

# Image 52.1

3. Click **Save**
4. Confirm **Standard Dashboard Rule** sits at priority 1 above the **Catch-all Rule**

# Image 52.2

### Test Standard Policy

1. Log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser
2. Confirm log in success only using a password and email verification

# Image 53

### Test HighSecurity Policy

1. Log in to **Okta** as **Morgan Reyes** in an **Incgonito** browser
2. Confirm that setting up Okta Verify is required to log in

# Image 54

### Verify in System Logs

1. In the **Okta Administration Console** open the **Reports** dropdown then **System Log**
2. Use the following search prompt then confirm **Create Policy** events appear:
   `eventType eq "policy.lifecycle.create"`

# Image 55
# Image 56

3. Use the following search prompt then confirm **Add Policy Rule** event appears:
   `eventType eq "policy.rule.add"`

# Image 57
# Image 58

4. Use the following search prompt then confirm **Activate Factor** event appears:
   `eventType eq "user.mfa.factor.activate" and actor.alternateId eq "morgan.reyes@kennon.tech"`

# Image 59
# Image 60

---

## 4) Network Zones




















