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

### Create IP Zone

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Networks**
2. Open the **Add Zone** dropdown then **IP Zone**
3. Create a zone with the following information then **Save:**

   - **Zone Name:** Kennon HQ
   - **Gateway IPs:** Your Current IP Address

# Image 61

### Confirm the Zone Detects Sign Ins

1. Log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser then **Sign Out**
2. On your phone turn off wifi so your on cellular date then log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser then sign out
3. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
4. In the **Search** bar enter the following command:
   `eventType eq "user.session.start" and actor.alternateId eq "jordan.ellis@kennon.tech"`

# Image 62

5. Expand the computer sign in and open **Client**. The IP address matches your public IP inside Kennon HQ

# Image 63

6. Expand the phone sign in and open **Client**. The IP address is a different cellular IP outside Kennon HQ

# Image 64

### Create a Blocklist Zone

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Networks**
2. Open the **Add Zone** dropdown then **IP Zone**
3. Create a zone with the following information then **Save:**

   - **Zone Name:** Kennon Blocked IPs
   - **Gateway IPs:** Enter your phone's IP address
   - **Block Access from IPs Matching Conditions Listed in this Zone:** Check

# Image 65

4. On your phone with wifi off attempt to log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser

# Image 66 on phone

5. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
6. In the **Search** bar enter the following command:
   `eventType eq "security.request.blocked"`

# Image 67
# Image 68

---

## 5) Global Session Policy

### Lower the Standard Dashboard Rule to One Factor

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Authentication Policies** 
2. Select **App Sign-In** then open **Okta Dashboard**
3. Find the **Standard Dashboard Rule** then open the **Actions** dropdown then **Edit**
4. Change **User Must Authenticate with:** Password then **Save**

# Image 69

### Create Session Policy

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Global Session Policy** 
2. Press **Add Policy** then create a policy with the following information the **Create Policy and Add Rule:**

   - **Policy Name:** Kennon Test Session Policy
   - **Policy Description:** Zone based MFA and session lengths for test users
   - **Assign to Groups:** AuthPolicy Standard and AuthPolicy HighSecurity

# Image 70

3. For the rule enter the following information then **Create Rule:**

   - **General Information**
   - **Rule Name:** Inside Kennon HQ Rule
  
# Image 71

   - **Policy Settings**
   - **If User's IP is:** In zone Kennon HQ
   - **Then Access is:** Allowed
   - **Establish the User Session with:** Any factor used to meet the Authentication Policy requirements
   - **Multifactor Authentication (MFA) is:** Not Required

# Image 72

   - **Okta Global Session Management**
   - **Maximum Okta Global Session Lifetime:** 12 hours
   - **Maximum Okta Global Session Idle Time:** 2 hours

# Image 73

### Add the Outside Zone Rule

1. On the **Kennon Test Session Policy** press **Add Rule**
2. Enter the following information then **Create Rule:**

   - **General Information**
   - **Rule Name:** Outside Zone Rule
  
# Image 74

   - **Policy Settings**
   - **If User's IP is:** Not in zone Kennon HQ
   - **Then Access is:** Allowed
   - **Establish the User Session with:** A password
   - **Multifactor Authentication (MFA) is:** Required
   - **Users will be Prompted for MFA:** At every sign in
  
# Image 75

   - **Okta Global Session Management**
   - **Maximum Okta Global Session Lifetime:** 2 hours
   - **Maximum Okta Global Session Idle Time:** 30 minutes

# Image 76

3. Confirm that the **Inside Kennon HQ Rule** is priority 1 and **Outside Zone Rule** is priority 2

# Image 77

### Test Inside the Zone

1. Log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser
2. Confirm that **Password** is the only verification type that it asks for then **Sign Out**

# Image 78

### Test Outside the Zone

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Networks**
2. Find **Kennon Blocked IPs** then open the **Active** dropdown then select **Inactive** then **Delete**

# Image 79

3. On your phone with wifi off attempt to log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser
4. Confirm that both the **Password** and **Email Verification** are required when logging in then **Sign Out**

# Image 80

### Another Test

1. Log in to **Okta** as **Morgan Reyes** in an **Incgonito** browser
2. Confirm that both **Password** and **Okta Verify** are required when logging in then **Sign Out**

# Image 81

### Verify In System Logs

1. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
2. In the **Search** bar enter the following command:
   `eventType eq "policy.lifecycle.create"`

# Image 82
# Image 83

3. In the **Search** bar enter the following command:
   `eventType eq "policy.rule.add"`

# Image 84
# Image 85

4. In the **Search** bar enter the following command:
   `eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "jordan.ellis@kennon.tech"`

# Image 86
# Image 87

---

## 6) App Level Authentication Policies

### Create Intranet Policy

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Authentication Policies**
2. Select **App Sign-In** then **Create Policy**
3. Create a policy with the following information:

   - **Name:** Kennon Intranet Policy
   - **Description:** Zone and group based access for Kennon Intranet

# Image 88

### Add HighSecurity Intranet Rule

1. With the **Kennon Intranet Policy** open select **Add Rule**
2. Fill out the following information then **Save:**

   - **General Information**
   - **Rule Name:** HighSecurity Intranet Rule
  
# Image 89

   - **IF**
   - **And User's Group Membership Includes:** At least one of the following groups AuthPolicy HighSecurity

# Image 90

   - **Then**
   - **Then Access is:** Allowed after successful authenticaion
   - **User Must Authenticate with:** Any 2 factor types
   - **Authentication Methods:** Disallow specific authentication Email

# Image 91

   - **When to Prompt for Authentication**
   - **Re-Authentication Frequency is:** Every sign in attempt

# Image 92

### Add Standard Outside HQ Rule

1. With the **Kennon Intranet Policy** open select **Add Rule**
2. Fill out the following information then **Save:**

   - **General Information**
   - **Rule Name:** Standard Outside HQ Rule
  
# Image 93

   - **IF**
   - **And User's Group Membership Includes:** At least one of the following groups AuthPolicy Standard
   - **And User's IP is:** Not in any of the following zones Kennon HQ

# Image 94

   - **Then**
   - **Then Access is:** Allowed after successful authenticaion
   - **User Must Authenticate with:** Any 2 factor types
   - **Authentication Methods:** Allow any method that can be used to meet the requirement

# Image 95

### Add Standard Inside HQ Rule

1. With the **Kennon Intranet Policy** open select **Add Rule**
2. Fill out the following information then **Save:**

   - **General Information**
   - **Rule Name:** Standard Inside HQ Rule
  
# Image 96

   - **IF**
   - **And User's Group Membership Includes:** At least one of the following groups AuthPolicy Standard
   - **And User's IP is:** In zone Kennon HQ

# Image 97

   - **Then Access is:** Allowed after successful authentication
   - **User Must Authenticate with:** Password

# Image 98

3. Confirm that **HighSecurity Intranet Rule** is priority 1, **Standard Inside HQ Rule** is priority 2, **Standard Outsude HQ Rule** is priority 3, and the **Catch-All Rule** is priority 4

# Image 99
# Image 100
# Image 101

### Assign Policy to Kennon Intranet

1. In the **Kennon Intranet Policy** open the **Apps** tab then **Add App**
2. Find **Kennon Intranet** then press *Add** then **Done**

# Image 102

3. Open the **Applications and Resources** dropdown the **Applications**
4. Select **Kennon Intranet** then open the **Sign On** tab
5. Confirm **User Authentication** shows **Kennon Intranet Policy**

# Image 103

### Test Morgan Reyes

1. Log in to **Okta** as **Morgan Reyes** in an **Incgonito** browser
2. On **Morgan's Dashboard** select **Kennon Intranet**
3. Confirm **Okta Verify** is prompted before being taken to example.com

# Image 104

4. Close the **exmaple.com** tab then on **Morgan's Dashboard** open **Kennon Intranet** again
5. Confirm **Okta Verify** is prompted again

# Image 105

6. **Sign Out** of Morgan Reyes's account

### Test Jordan Inside and Outside the Zone

1. Log in to **Okta** as **Jordan Ellis** in an **Incgonito** browser
2. On **Jordan's Dashboard** open **Kennon Intranet**
3. Confirm it opens without any additional verification necessary
4. **Sign Out** of Jordan Ellis's account
5. On your phone disconnect from Wi-Fi then log into **Jordan Ellis's** account in an **Incognito** browser
6. On **Jordan's Dashboard** open **Kennon Intranet**
7. Confirm it opens without any additional verification necessary

# Image 106

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
2. In the **Search** bar enter the following command:
   `eventType eq "policy.lifecycle.create" and target.displayName eq "Kennon Intranet Policy"`

# Image 107
# Image 108 hide ip address

3. In the **Search** bar enter the following command:
   `eventType eq "policy.rule.add"`

# Image 109
# Image 110

4. In the **Search** bar enter the following command:
   `eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "morgan.reyes@kennon.tech"`

# Image 111
# Image 112

5. In the **Search** bar enter the following command:
   `eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "jordan.ellis@kennon.tech" and outcome.result eq "ALLOW"`

# Image 113
# Image 114

6. In the **Search** bar enter the following command:
   `eventType eq "user.authentication.sso" and actor.alternateId eq "morgan.reyes@kennon.tech"`

# Image 115
# Image 116

7. In the **Search** bar enter the following command:
   `eventType eq "user.authentication.sso" and actor.alternateId eq "jordan.ellis@kennon.tech" and target.displayName eq "Bookmark App"`

# Image 117
# Image 118

---

## Phase 7: Test Matrix Summary

The table below combines every sign in tested in this lab, showing what each user was asked for and which rules Okta matched.

| User | Location | Resource | Required | Session Rule | App Rule |
|---|---|---|---|---|---|
| Jordan Ellis | Inside Kennon HQ | Okta Dashboard | Password | Inside Kennon HQ Rule | Standard Dashboard Rule |
| Jordan Ellis | Outside Kennon HQ | Okta Dashboard | Password and Email | Outside Zone Rule | Standard Dashboard Rule |
| Jordan Ellis | Inside Kennon HQ | Kennon Intranet | No additional prompt | N/A | Standard Inside HQ Rule |
| Jordan Ellis | Outside Kennon HQ | Kennon Intranet | No additional prompt | N/A | Standard Outside HQ Rule |
| Morgan Reyes | Inside Kennon HQ | Okta Dashboard | Password and Okta Verify | Inside Kennon HQ Rule | Catch-all Rule |
| Morgan Reyes | Inside Kennon HQ | Kennon Intranet | Okta Verify on every launch | N/A | HighSecurity Intranet Rule |

Morgan required Okta Verify inside Kennon HQ even though the session rule required no MFA, showing that the stricter policy always wins.



