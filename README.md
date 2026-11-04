<p align="center">
  <img src="Images/Header.png" alt="Okta Authentication Policies and Network Zones">
</p>

# Okta Authentication Policies and Network Zones

This lab builds a layered authentication setup for the fictional company Kennon Technologies. It covers authenticator settings, password and enrollment policies, a trusted network zone, a global session policy, and an app level authentication policy, then tests each layer by signing in as test users from inside and outside the trusted network.

---

## Skills Demonstrated

- Configuring Okta Verify verification options and push number challenge
- Creating group scoped password policies with complexity and lockout settings
- Building authenticator enrollment policies with required and optional authenticators
- Creating IP zones and blocklist zones
- Configuring a global session policy with zone based MFA and session lifetimes
- Creating app level authentication policies with group and zone conditions
- Understanding policy evaluation order and how the stricter policy wins
- Troubleshooting sign in behavior and verifying results in the System Log

---

## Prerequisites

- An Okta Integrator Free Plan org (Identity Engine) with Super Administrator access
- Okta Verify installed on a phone
- A phone with cellular data for testing sign ins from outside the trusted zone
- An email inbox you control for the test users' Email authenticator
- A second browser session signed in as your admin account while editing policies

---

## Environment

| Component | Details |
|---|---|
| Identity Platform | Okta Integrator Free Plan (Identity Engine) |
| Company | Kennon Technologies (kennon.tech) |
| Test Users | Jordan Ellis (Standard), Morgan Reyes (HighSecurity) |
| Test Groups | AuthPolicy Standard, AuthPolicy HighSecurity |
| Test App | Kennon Intranet (Bookmark App) |
| MFA | Okta Verify (push and TOTP), Email |
| Inside Zone Device | Computer on home network |
| Outside Zone Device | Phone on cellular data |

> **Note:** Real IP addresses are blurred in screenshots. Kennon Intranet appears as **Bookmark App** in the System Log.

---

## Table of Contents

1. [Prepare Test Users and Groups](#1-prepare-test-users-and-groups)
2. [Configure Authenticators and Password Policy](#2-configure-authenticators-and-password-policy)
3. [Authenticator Enrollment Policies](#3-authenticator-enrollment-policies)
4. [Network Zones](#4-network-zones)
5. [Global Session Policy](#5-global-session-policy)
6. [App Level Authentication Policies](#6-app-level-authentication-policies)
7. [Test Matrix Summary](#7-test-matrix-summary)
8. [Cleanup](#8-cleanup)
9. [Closing Note](#closing-note)

---

## 1) Prepare Test Users and Groups

### Create Test Groups

1. In the **Okta Administrator Console** open the **Directory** dropdown then **Groups**
2. Press **Add Group** then create a group with the following information then **Save:**
   - **Name:** AuthPolicy Standard
   - **Description:** Test users for standard authentication policies
  
<p align="center">
  <img src="Images/Image%201.PNG" alt="Image 1">
</p>

3. Create a second group with the following information then **Save:**
   - **Name:** AuthPolicy HighSecurity
   - **Description:** Test users for strict authentication policies
  
<p align="center">
  <img src="Images/Image%202.PNG" alt="Image 2">
  <img src="Images/Image%203.PNG" alt="Image 3">
</p>

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
  
<p align="center">
  <img src="Images/Image%204.png" alt="Image 4">
</p>

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
  
<p align="center">
  <img src="Images/Image%205.png" alt="Image 5">
  <img src="Images/Image%206.png" alt="Image 6">
</p>

### Create Test App

1. Open the **Applications and Resources** dropdown then select **Applications** then **Browse App Catalog**
2. Search for the **Bookmark App** then **Add Integration**
3. On the **General Settings** page enter the following information then press **Done:**
   - **Application Label:** Kennon Intranet
   - **URL:** https://example.com

<p align="center">
  <img src="Images/Image%207.PNG" alt="Image 7">
</p>

4. Open the **Sign On** tab then next to **User Authentication** select **Edit**
5. For the **Authentication Policy** select **Any Two Factors** then **Save**

<p align="center">
  <img src="Images/Image%208.PNG" alt="Image 8">
</p>

6. Open the **Assignments** tab then click the **Assign** dropdown then **Assign to Groups**
7. Select **Assign** for both **AuthPolicy Standard** and **AuthPolicy HighSecurity** then **Done**

<p align="center">
  <img src="Images/Image%209.PNG" alt="Image 9">
  <img src="Images/Image%2010.PNG" alt="Image 10">
</p>

### Confirm Sign In

1. Attempt to log in to **Okta** as **Jordan Ellis** in an **incognito** browser
2. Confirm that the **Kennon Intranet** app appears on the dashboard then **Sign Out**

<p align="center">
  <img src="Images/Image%2011.PNG" alt="Image 11">
</p>

3. Attempt to log in to **Okta** as **Morgan Reyes**
4. Confirm that the **Kennon Intranet** app appears on the dashboard then **Sign Out**

<p align="center">
  <img src="Images/Image%2012.PNG" alt="Image 12">
</p>

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown then **System Log**
2. Search for user creation events filtered to Jordan Ellis to confirm the account was created:
   ```text
   eventType eq "user.lifecycle.create" and target.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The **Create okta user** event appears with your admin account as the actor and **Jordan Ellis** as the target

<p align="center">
  <img src="Images/Image%2013.PNG" alt="Image 13">
  <img src="Images/Image%2014.png" alt="Image 14">
</p>

3. Search for group membership changes filtered to Jordan Ellis to confirm the group assignment:
   ```text
   eventType eq "group.user_membership.add" and target.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The **Add user to group membership** event appears with **AuthPolicy Standard** as a target

<p align="center">
  <img src="Images/Image%2015.PNG" alt="Image 15">
  <img src="Images/Image%2016.png" alt="Image 16">
</p>

4. Search for app assignments filtered to Jordan Ellis to confirm Kennon Intranet was assigned through the group:
   ```text
   eventType eq "application.user_membership.add" and target.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The **Add user to application membership** event appears with **Kennon Intranet** as a target

<p align="center">
  <img src="Images/Image%2017.PNG" alt="Image 17">
  <img src="Images/Image%2018.png" alt="Image 18">
</p>

5. Search for sign in sessions started by Jordan Ellis to confirm the baseline sign in worked:
   ```text
   eventType eq "user.session.start" and actor.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The **User login to Okta** event appears with a **SUCCESS** outcome

<p align="center">
  <img src="Images/Image%2019.PNG" alt="Image 19">
  <img src="Images/Image%2020.png" alt="Image 20">
</p>

6. Search for sign on policy evaluations for Jordan Ellis to see which policy Okta applied:
   ```text
   eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The **Evaluation of sign on policy** event appears with the matched policy and rule listed under **Targets**

<p align="center">
  <img src="Images/Image%2021.PNG" alt="Image 21">
  <img src="Images/Image%2022.png" alt="Image 22">
</p>

7. Search for app sign ons by Jordan Ellis to confirm access to Kennon Intranet:
   ```text
   eventType eq "user.authentication.sso" and actor.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The **User single sign on to app** event appears with **Kennon Intranet** as the target

<p align="center">
  <img src="Images/Image%2023.PNG" alt="Image 23">
  <img src="Images/Image%2024.png" alt="Image 24">
</p>

---

## 2) Configure Authenticators and Password Policy

### Configure Okta Verify

1. In the **Okta Administrator Console** open the **Security** dropdown then **Authenticators**
2. To the right of **Okta Verify** open the **Actions** dropdown then **Edit**
3. Under **Verification Options** make sure **TOTP** and **Push Notifications** are checked and **Okta FastPass** is unchecked

<p align="center">
  <img src="Images/Image%2025.PNG" alt="Image 25">
</p>

4. Under **Push Notification: Number Challenge** make sure **All Push Challenges** is selected then **Save**

<p align="center">
  <img src="Images/Image%2026.PNG" alt="Image 26">
</p>

### Create HighSecurity Password Policy

1. In the **Okta Administrator Console** open the **Security** dropdown then **Authenticators**
2. To the right of **Password** open the **Actions** dropdown then **Edit**
3. Click **Add New Password Policy** then create a policy with the following information then **Create Policy** then **Create Rule:**

   - **General Information**
      - **Policy Name:** HighSecurity Password Policy
      - **Policy Description:** Stricter password requirements for high security users
      - **Add Group:** AuthPolicy HighSecurity
      - **Authentication Providers:** Okta
   
<p align="center">
  <img src="Images/Image%2027.PNG" alt="Image 27">
</p>

   - **Password Requirements**
      - **Minimum Length:** 14
      - **Complexity Requirements:** check lower case letter, upper case letter, number, and symbol
      - **Restrict Use of Common Passwords:** Check
      - **Does not Contain Part of Username / First Name / Last Name:** Check
    
<p align="center">
  <img src="Images/Image%2028.PNG" alt="Image 28">
</p>

   - **Password Security**
      - **Enforce Password History for Last:** 5 passwords
      - **Minimum Password Age:** Uncheck
      - **Password Expires After:** 90 days

<p align="center">
  <img src="Images/Image%2029.PNG" alt="Image 29">
</p>

   - **Rule Settings:** HighSecurity Password Rule

<p align="center">
  <img src="Images/Image%2030.PNG" alt="Image 30">
</p>

### Test HighSecurity Password Policy

1. Log in to **Okta** as **Morgan Reyes** in an **incognito** browser
2. When asked choose **Get a Push Notification** to confirm it’s active then finish logging in

<p align="center">
  <img src="Images/Image%2031.PNG" alt="Image 31">
</p>

3. Click **Morgan's Name** in the top right of the screen then **Account Settings**
4. Under **Security Methods** find **Password** then **Reset**

<p align="center">
  <img src="Images/Image%2032.PNG" alt="Image 32">
</p>

5. Enter `Password1` as the password and confirm that it has been rejected

<p align="center">
  <img src="Images/Image%2033.PNG" alt="Image 33">
</p>

6. Create a password that meets all requirements then reset password and sign out

### Test Account Lockout

1. Attempt to log in to **Okta** as **Morgan Reyes** with an **Incorrect Password** in an **incognito** browser until it no longer allows a login using a password
2. In the **Okta Administrator Console** open the **Directory** dropdown then **People**
3. Open **Morgan Reyes's** profile then confirm that **Morgan Reyes** has the **Locked Out** status

<p align="center">
  <img src="Images/Image%2034.png" alt="Image 34">
</p>

4. Press **Unlock Account** and confirm that **Morgan Reyes** has the **Active** status

<p align="center">
  <img src="Images/Image%2035.png" alt="Image 35">
</p>

5. Back in the **incognito** browser confirm you can log in to **Okta** as **Morgan Reyes** using a password then sign out

### Confirm User not in Group is Unaffected

1. Log in to **Okta** as **Jordan Ellis** in an **incognito** browser
2. Click **Jordan's Name** in the top right of the screen then **Account Settings**
3. Under **Security Methods** find **Password** then **Reset**
4. Confirm fewer password requirements appear

<p align="center">
  <img src="Images/Image%2036.PNG" alt="Image 36">
</p>

5. Sign out of Jordan's account

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown then **System Log**
2. Search for policy creation events to confirm the password policy was created:
   ```text
   eventType eq "policy.lifecycle.create"
   ```
   The **Create policy** event appears with your admin account as the actor and **HighSecurity Password Policy** as the target

<p align="center">
  <img src="Images/Image%2037.PNG" alt="Image 37">
  <img src="Images/Image%2038.png" alt="Image 38">
</p>

3. Search for policy rule additions to confirm the password rule was created:
   ```text
   eventType eq "policy.rule.add"
   ```
   The **Add policy rule** event appears with **HighSecurity Password Policy** and **HighSecurity Password Rule** as targets

<p align="center">
  <img src="Images/Image%2039.PNG" alt="Image 39">
  <img src="Images/Image%2040.png" alt="Image 40">
</p>

4. Search for password updates filtered to Morgan Reyes to confirm the password change:
   ```text
   eventType eq "user.account.update_password" and target.alternateId eq "morgan.reyes@kennon.tech"
   ```
   The password update event appears with **Morgan Reyes** as both the actor and the target and a **SUCCESS** outcome

<p align="center">
  <img src="Images/Image%2041.PNG" alt="Image 41">
  <img src="Images/Image%2042.png" alt="Image 42">
</p>

5. Search for account lockouts filtered to Morgan Reyes to confirm the lockout policy worked:
   ```text
   eventType eq "user.account.lock" and target.alternateId eq "morgan.reyes@kennon.tech"
   ```
   The lockout event appears right after Morgan's third failed sign in attempt

<p align="center">
  <img src="Images/Image%2043.PNG" alt="Image 43">
  <img src="Images/Image%2044.png" alt="Image 44">
</p>

6. Search for admin unlocks filtered to Morgan Reyes to confirm the account was unlocked:
   ```text
   eventType eq "user.account.unlock_by_admin" and target.alternateId eq "morgan.reyes@kennon.tech"
   ```
   The unlock event appears with your admin account as the actor and **Morgan Reyes** as the target

<p align="center">
  <img src="Images/Image%2045.PNG" alt="Image 45">
  <img src="Images/Image%2046.png" alt="Image 46">
</p>

### Reset Okta Verify for Both Users

1. In the **Okta Administrator Console** open the **Directory** dropdown then **People**
2. Select **Morgan Reyes** then open the **More Actions** dropdown then select **Reset Authenticators**

<p align="center">
  <img src="Images/Image%2047.png" alt="Image 47">
</p>

3. Select **Okta Verify** then **Reset Selected Authenticators**
4. Repeat with **Jordan Ellis**

---

## 3) Authenticator Enrollment Policies

### Create Standard Enrollment Policy

1. In the **Okta Administrator Console** open the **Security** dropdown then **Authenticators**
2. Open the **Enrollment** tab then **Add a Policy**
3. Create a policy with the following information then **Create Policy:**

   - **Policy Name:** Standard Enrollment Policy
   - **Policy Description:** Okta Verify optional for standard users
   - **Assign to Groups:** AuthPolicy Standard
   - **Email Enrollment:** Optional
   - **Okta Verify Enrollment:** Optional
   - **Password Enrollment:** Required

<p align="center">
  <img src="Images/Image%2049.PNG" alt="Image 49">
</p>

4. On the **Add Rule** page enter the following information then **Create Rule:**

   - **Rule Name:** Standard Employee Rule
   - **If User's IP is:** Anywhere
   - **And User is Accessing:** Okta or Any App
   - **Then Enroll in Authenticators:** Allowed for All Authenticators

<p align="center">
  <img src="Images/Image%2050.PNG" alt="Image 50">
</p>

### Create HighSecurity Enrollment Policy

1. Create a second policy with the following information then **Create Policy:**

   - **Policy Name:** HighSecurity Enrollment Policy
   - **Policy Description:** Okta Verify required for high security users
   - **Assign to Groups:** AuthPolicy HighSecurity
   - **Email Enrollment:** Optional
   - **Okta Verify Enrollment:** Required
   - **Password Enrollment:** Required

<p align="center">
  <img src="Images/Image%2051.PNG" alt="Image 51">
</p>

2. On the **Add Rule** page enter the following information then **Create Rule:**

   - **Rule Name:** HighSecurity Enrollment Rule
   - **If User's IP is:** Anywhere
   - **And User is Accessing:** Okta or Any App
   - **Then Enroll in Authenticators:** Allowed for All Authenticators

<p align="center">
  <img src="Images/Image%2052.PNG" alt="Image 52">
</p>

### Add a Standard Rule to the Okta Dashboard Policy

1. Open the **Security** dropdown then **Authentication Policies** and select **Okta Dashboard**
2. Click **Add rule** and configure the following:

   - **Rule Name:** Standard Dashboard Rule
   - **And User's group membership includes:** At least one of the following groups, **AuthPolicy Standard**
   - **Then Access is:** Allowed after successful authentication
   - **User must authenticate with:** Any 2 factor types
   - **Authentication methods:** Allow any method that can be used to meet the requirement

<p align="center">
  <img src="Images/Image%2052.1.PNG" alt="Image 52.1">
</p>

3. Click **Save**
4. Confirm **Standard Dashboard Rule** sits at priority 1 above the **Catch-all Rule**

<p align="center">
  <img src="Images/Image%2052.2.PNG" alt="Image 52.2">
</p>

### Test Standard Policy

1. Log in to **Okta** as **Jordan Ellis** in an **incognito** browser
2. Confirm the sign in succeeds using only a password and email verification

<p align="center">
  <img src="Images/Image%2053.PNG" alt="Image 53">
</p>

### Test HighSecurity Policy

1. Log in to **Okta** as **Morgan Reyes** in an **incognito** browser
2. Confirm that setting up Okta Verify is required to log in

<p align="center">
  <img src="Images/Image%2054.PNG" alt="Image 54">
</p>

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown then **System Log**
2. Search for policy creation events to confirm both enrollment policies were created:
   ```text
   eventType eq "policy.lifecycle.create"
   ```
   **Create policy** events appear for **Standard Enrollment Policy** and **HighSecurity Enrollment Policy** with your admin account as the actor

<p align="center">
  <img src="Images/Image%2055.PNG" alt="Image 55">
  <img src="Images/Image%2056.png" alt="Image 56">
</p>

3. Search for policy rule additions to confirm the enrollment and dashboard rules were created:
   ```text
   eventType eq "policy.rule.add"
   ```
   **Add policy rule** events appear for **Standard Employee Rule**, **HighSecurity Enrollment Rule**, **Standard Dashboard Rule** and **Test Groups Enrollment Rule**

<p align="center">
  <img src="Images/Image%2057.PNG" alt="Image 57">
  <img src="Images/Image%2058.png" alt="Image 58">
</p>

4. Search for authenticator activations filtered to Morgan Reyes to confirm Okta Verify enrollment:
   ```text
   eventType eq "user.mfa.factor.activate" and actor.alternateId eq "morgan.reyes@kennon.tech"
   ```
   Three **Activate factor for user** events appear at the same time with **Okta Verify** as the target, one for each verification method enrolled with the app

<p align="center">
  <img src="Images/Image%2059.PNG" alt="Image 59">
  <img src="Images/Image%2060.png" alt="Image 60">
</p>

---

## 4) Network Zones

### Create IP Zone

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Networks**
2. Open the **Add Zone** dropdown then **IP Zone**
3. Create a zone with the following information then **Save:**

   - **Zone Name:** Kennon HQ
   - **Gateway IPs:** Your Current IP Address

<p align="center">
  <img src="Images/Image%2061.png" alt="Image 61">
</p>

### Confirm the Zone Detects Sign Ins

1. Log in to **Okta** as **Jordan Ellis** in an **incognito** browser then **Sign Out**
2. On your phone turn off WiFi so you’re on cellular data then log in to **Okta** as **Jordan Ellis** in an **incognito** browser then sign out
3. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
4. Search for sign in sessions started by Jordan Ellis to compare the two sign ins:
   ```text
   eventType eq "user.session.start" and actor.alternateId eq "jordan.ellis@kennon.tech"
   ```
   Two **User login to Okta** events appear, one from the computer and one from the phone, each with a different IP address

<p align="center">
  <img src="Images/Image%2062.PNG" alt="Image 62">
</p>

5. Expand the computer sign in and open **Client**. The IP address matches your public IP inside Kennon HQ

<p align="center">
  <img src="Images/Image%2063.png" alt="Image 63">
</p>

6. Expand the phone sign in and open **Client**. The IP address is a different cellular IP outside Kennon HQ

<p align="center">
  <img src="Images/Image%2064.png" alt="Image 64">
</p>

### Create a Blocklist Zone

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Networks**
2. Open the **Add Zone** dropdown then **IP Zone**
3. Create a zone with the following information then **Save:**

   - **Zone Name:** Kennon Blocked IPs
   - **Gateway IPs:** Enter your phone's IP address
   - **Block Access from IPs Matching Conditions Listed in this Zone:** Check

<p align="center">
  <img src="Images/Image%2065.png" alt="Image 65">
</p>

4. On your phone with WiFi off attempt to log in to **Okta** as **Jordan Ellis** in an **incognito** browser. The sign in is refused with a permission error

<p align="center">
  <img src="Images/Image%2066.webp" alt="Image 66">
</p>

5. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
6. Search for requests stopped by a blocklist zone to confirm the phone was blocked:
   ```text
   eventType eq "security.request.blocked"
   ```
   The **Blocked request from IP** event appears with **Okta System** as the actor, the phone's IP, and **NETWORK_ZONE_BLACKLIST** as the reason under **Outcome**

<p align="center">
  <img src="Images/Image%2067.PNG" alt="Image 67">
  <img src="Images/Image%2068.webp" alt="Image 68">
</p>

---

## 5) Global Session Policy

### Lower the Standard Dashboard Rule to Password Only

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Authentication Policies** 
2. Select **App Sign-In** then open **Okta Dashboard**
3. Find the **Standard Dashboard Rule** then open the **Actions** dropdown then **Edit**
4. Change **User Must Authenticate with:** Password then **Save**

<p align="center">
  <img src="Images/Image%2069.PNG" alt="Image 69">
</p>

### Create Session Policy

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Global Session Policy** 
2. Press **Add Policy** then create a policy with the following information then **Create Policy and Add Rule:**

   - **Policy Name:** Kennon Test Session Policy
   - **Policy Description:** Zone based MFA and session lengths for test users
   - **Assign to Groups:** AuthPolicy Standard and AuthPolicy HighSecurity

<p align="center">
  <img src="Images/Image%2070.PNG" alt="Image 70">
</p>

3. For the rule enter the following information then **Create Rule:**

   - **General Information**
   - **Rule Name:** Inside Kennon HQ Rule
  
<p align="center">
  <img src="Images/Image%2071.PNG" alt="Image 71">
</p>

   - **Policy Settings**
   - **If User's IP is:** In zone Kennon HQ
   - **Then Access is:** Allowed
   - **Establish the User Session with:** Any factor used to meet the Authentication Policy requirements
   - **Multifactor Authentication (MFA) is:** Not Required

<p align="center">
  <img src="Images/Image%2072.PNG" alt="Image 72">
</p>

   - **Okta Global Session Management**
   - **Maximum Okta Global Session Lifetime:** 12 hours
   - **Maximum Okta Global Session Idle Time:** 2 hours

<p align="center">
  <img src="Images/Image%2073.PNG" alt="Image 73">
</p>

### Add the Outside Zone Rule

1. On the **Kennon Test Session Policy** press **Add Rule**
2. Enter the following information then **Create Rule:**

   - **General Information**
   - **Rule Name:** Outside Zone Rule
  
<p align="center">
  <img src="Images/Image%2074.PNG" alt="Image 74">
</p>

   - **Policy Settings**
   - **If User's IP is:** Not in zone Kennon HQ
   - **Then Access is:** Allowed
   - **Establish the User Session with:** A password
   - **Multifactor Authentication (MFA) is:** Required
   - **Users will be Prompted for MFA:** At every sign in
  
<p align="center">
  <img src="Images/Image%2075.PNG" alt="Image 75">
</p>

   - **Okta Global Session Management**
   - **Maximum Okta Global Session Lifetime:** 2 hours
   - **Maximum Okta Global Session Idle Time:** 30 minutes

<p align="center">
  <img src="Images/Image%2076.PNG" alt="Image 76">
</p>

3. Confirm that the **Inside Kennon HQ Rule** is priority 1 and **Outside Zone Rule** is priority 2

<p align="center">
  <img src="Images/Image%2077.PNG" alt="Image 77">
</p>

### Test Inside the Zone

1. Log in to **Okta** as **Jordan Ellis** in an **incognito** browser
2. Confirm that **Password** is the only verification type that it asks for then **Sign Out**

<p align="center">
  <img src="Images/Image%2078.PNG" alt="Image 78">
</p>

### Test Outside the Zone

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Networks**
2. Find **Kennon Blocked IPs** then open the **Active** dropdown then select **Inactive** then **Delete**

<p align="center">
  <img src="Images/Image%2079.png" alt="Image 79">
</p>

3. On your phone with WiFi off attempt to log in to **Okta** as **Jordan Ellis** in an **incognito** browser
4. Confirm that both the **Password** and **Email Verification** are required when logging in then **Sign Out**

<p align="center">
  <img src="Images/Image%2080.webp" alt="Image 80">
</p>

### Test Morgan Inside the Zone

1. Log in to **Okta** as **Morgan Reyes** in an **incognito** browser
2. Confirm that both **Password** and **Okta Verify** are required when logging in then **Sign Out**

<p align="center">
  <img src="Images/Image%2081.PNG" alt="Image 81">
</p>

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
2. Search for policy creation events to confirm the session policy was created:
   ```text
   eventType eq "policy.lifecycle.create"
   ```
   The **Create policy** event appears with your admin account as the actor and **Kennon Test Session Policy** as the target

<p align="center">
  <img src="Images/Image%2082.PNG" alt="Image 82">
  <img src="Images/Image%2083.png" alt="Image 83">
</p>

3. Search for policy rule additions to confirm both session rules were created:
   ```text
   eventType eq "policy.rule.add"
   ```
   **Add policy rule** events appear for **Inside Kennon HQ Rule** and **Outside Zone Rule**, both under **Kennon Test Session Policy**

<p align="center">
  <img src="Images/Image%2084.PNG" alt="Image 84">
  <img src="Images/Image%2085.png" alt="Image 85">
</p>

4. Search for sign on policy evaluations for Jordan Ellis to see which session rule matched in each location:
   ```text
   eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "jordan.ellis@kennon.tech"
   ```
   The computer sign in from your public IP matches **Inside Kennon HQ Rule**, and the phone sign in from the cellular IP matches **Outside Zone Rule**, confirming the Kennon HQ zone controls which rule applies

<p align="center">
  <img src="Images/Image%2086.PNG" alt="Image 86">
  <img src="Images/Image%2087.png" alt="Image 87">
</p>

---

## 6) App Level Authentication Policies

### Create Intranet Policy

1. In the **Okta Administrator Console** open the **Security** dropdown and then **Authentication Policies**
2. Select **App Sign-In** then **Create Policy**
3. Create a policy with the following information:

   - **Name:** Kennon Intranet Policy
   - **Description:** Zone and group based access for Kennon Intranet

<p align="center">
  <img src="Images/Image%2088.PNG" alt="Image 88">
</p>

### Add HighSecurity Intranet Rule

1. With the **Kennon Intranet Policy** open select **Add Rule**
2. Fill out the following information then **Save:**

   - **General Information**
   - **Rule Name:** HighSecurity Intranet Rule
  
<p align="center">
  <img src="Images/Image%2089.PNG" alt="Image 89">
</p>

   - **IF**
   - **And User's Group Membership Includes:** At least one of the following groups AuthPolicy HighSecurity

<p align="center">
  <img src="Images/Image%2090.PNG" alt="Image 90">
</p>

   - **Then**
   - **Then Access is:** Allowed after successful authentication
   - **User Must Authenticate with:** Any 2 factor types
   - **Authentication Methods:** Disallow specific authentication methods: Email

<p align="center">
  <img src="Images/Image%2091.PNG" alt="Image 91">
</p>

   - **When to Prompt for Authentication**
   - **Re-Authentication Frequency is:** Every sign in attempt

<p align="center">
  <img src="Images/Image%2092.PNG" alt="Image 92">
</p>

### Add Standard Outside HQ Rule

1. With the **Kennon Intranet Policy** open select **Add Rule**
2. Fill out the following information then **Save:**

   - **General Information**
   - **Rule Name:** Standard Outside HQ Rule
  
<p align="center">
  <img src="Images/Image%2093.PNG" alt="Image 93">
</p>

   - **IF**
   - **And User's Group Membership Includes:** At least one of the following groups AuthPolicy Standard
   - **And User's IP is:** Not in any of the following zones Kennon HQ

<p align="center">
  <img src="Images/Image%2094.PNG" alt="Image 94">
</p>

   - **Then**
   - **Then Access is:** Allowed after successful authentication
   - **User Must Authenticate with:** Any 2 factor types
   - **Authentication Methods:** Allow any method that can be used to meet the requirement

<p align="center">
  <img src="Images/Image%2095.PNG" alt="Image 95">
</p>

### Add Standard Inside HQ Rule

1. With the **Kennon Intranet Policy** open select **Add Rule**
2. Fill out the following information then **Save:**

   - **General Information**
   - **Rule Name:** Standard Inside HQ Rule
  
<p align="center">
  <img src="Images/Image%2096.PNG" alt="Image 96">
</p>

   - **IF**
   - **And User's Group Membership Includes:** At least one of the following groups AuthPolicy Standard
   - **And User's IP is:** In zone Kennon HQ

<p align="center">
  <img src="Images/Image%2097.PNG" alt="Image 97">
</p>

   - **Then**
   - **Then Access is:** Allowed after successful authentication
   - **User Must Authenticate with:** Password

<p align="center">
  <img src="Images/Image%2098.PNG" alt="Image 98">
</p>

3. Confirm that **HighSecurity Intranet Rule** is priority 1, **Standard Inside HQ Rule** is priority 2, **Standard Outside HQ Rule** is priority 3, and the **Catch-all Rule** is priority 4

<p align="center">
  <img src="Images/Image%2099.PNG" alt="Image 99"> <br>
  <img src="Images/Image%20100.PNG" alt="Image 100"> <br>
  <img src="Images/Image%20101.PNG" alt="Image 101">
</p>

### Assign Policy to Kennon Intranet

1. In the **Kennon Intranet Policy** open the **Apps** tab then **Add App**
2. Find **Kennon Intranet** then press **Add** then **Done**

<p align="center">
  <img src="Images/Image%20102.PNG" alt="Image 102">
</p>

3. Open the **Applications and Resources** dropdown then **Applications**
4. Select **Kennon Intranet** then open the **Sign On** tab
5. Confirm **User Authentication** shows **Kennon Intranet Policy**

<p align="center">
  <img src="Images/Image%20103.PNG" alt="Image 103">
</p>

### Test Morgan Reyes

1. Log in to **Okta** as **Morgan Reyes** in an **incognito** browser
2. On **Morgan's Dashboard** select **Kennon Intranet**
3. Confirm **Okta Verify** is prompted before being taken to example.com

<p align="center">
  <img src="Images/Image%20104.PNG" alt="Image 104">
</p>

4. Close the **example.com** tab then on **Morgan's Dashboard** open **Kennon Intranet** again
5. Confirm **Okta Verify** is prompted again

<p align="center">
  <img src="Images/Image%20105.PNG" alt="Image 105">
</p>

6. **Sign Out** of Morgan Reyes's account

### Test Jordan Inside and Outside the Zone

1. Log in to **Okta** as **Jordan Ellis** in an **incognito** browser
2. On **Jordan's Dashboard** open **Kennon Intranet**
3. Confirm it opens without any additional verification necessary
4. **Sign Out** of Jordan Ellis's account
5. On your phone turn off WiFi then log into **Jordan Ellis's** account in an **incognito** browser
6. On **Jordan's Dashboard** open **Kennon Intranet**
7. Confirm it opens without any additional verification necessary

<p align="center">
  <img src="Images/Image%20106.webp" alt="Image 106">
</p>

### Verify in System Log

1. In the **Okta Administrator Console** open the **Reports** dropdown and then **System Log**
2. Search for policy creation events filtered to Kennon Intranet Policy to confirm the app policy was created:
   ```text
   eventType eq "policy.lifecycle.create" and target.displayName eq "Kennon Intranet Policy"
   ```
   The **Create policy** event appears with your admin account as the actor and **Kennon Intranet Policy** as the target

<p align="center">
  <img src="Images/Image%20107.PNG" alt="Image 107">
  <img src="Images/Image%20108.PNG" alt="Image 108">
</p>

3. Search for policy rule additions to confirm all three app rules were created:
   ```text
   eventType eq "policy.rule.add"
   ```
   **Add policy rule** events appear for **HighSecurity Intranet Rule**, **Standard Outside HQ Rule** and **Standard Inside HQ Rule**, all under **Kennon Intranet Policy**

<p align="center">
  <img src="Images/Image%20109.PNG" alt="Image 109">
  <img src="Images/Image%20110.png" alt="Image 110">
</p>

4. Search for sign on policy evaluations for Morgan Reyes to confirm the app challenged on every launch:
   ```text
   eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "morgan.reyes@kennon.tech"
   ```
   Two **Bookmark App** events match **HighSecurity Intranet Rule** with a **CHALLENGE** outcome, one for each time Kennon Intranet was opened, while Morgan's sign in just before matched **Inside Kennon HQ Rule**

<p align="center">
  <img src="Images/Image%20111.PNG" alt="Image 111">
  <img src="Images/Image%20112.png" alt="Image 112">
</p>

5. Search for allowed sign on policy evaluations for Jordan Ellis to see which app rule matched in each location:
   ```text
   eventType eq "policy.evaluate_sign_on" and actor.alternateId eq "jordan.ellis@kennon.tech" and outcome.result eq "ALLOW"
   ```
   The computer launch matches **Standard Inside HQ Rule** and the phone launch matches **Standard Outside HQ Rule**, both with an **ALLOW** outcome and no extra prompt

<p align="center">
  <img src="Images/Image%20113.PNG" alt="Image 113">
  <img src="Images/Image%20114.png" alt="Image 114">
</p>

6. Search for app sign ons by Morgan Reyes to confirm access to Kennon Intranet after verifying:
   ```text
   eventType eq "user.authentication.sso" and actor.alternateId eq "morgan.reyes@kennon.tech"
   ```
   The **User single sign on to app** event appears with **Bookmark App** as the target and a **SUCCESS** outcome

<p align="center">
  <img src="Images/Image%20115.PNG" alt="Image 115">
  <img src="Images/Image%20116.png" alt="Image 116">
</p>

7. Search for app sign ons by Jordan Ellis filtered to the Bookmark App to confirm access from both locations:
   ```text
   eventType eq "user.authentication.sso" and actor.alternateId eq "jordan.ellis@kennon.tech" and target.displayName eq "Bookmark App"
   ```
   Two **User single sign on to app** events appear with a **SUCCESS** outcome, one from your public IP and one from the cellular IP

<p align="center">
  <img src="Images/Image%20117.PNG" alt="Image 117">
  <img src="Images/Image%20118.png" alt="Image 118">
</p>

---

## 7) Test Matrix Summary

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

---

## 8) Cleanup

### Remove the Kennon Intranet Policy

1. In the **Okta Administrator Console** open the **Applications and Resources** dropdown and then **Applications**
2. Select the **Kennon Intranet** app then open the **Sign On** tab
3. Under **User Authentication** press **Edit**
4. Switch the policy to **Any Two Factors** then **Save**

<p align="center">
  <img src="Images/Image%20119.PNG" alt="Image 119">
</p>

5. Open the **Security** dropdown and then **Authentication Policies**
6. Press the **Three Dots** to the right of **Kennon Intranet Policy** then **Delete**

<p align="center">
  <img src="Images/Image%20120.PNG" alt="Image 120">
</p>

### Remove the Session Policy

1. Open the **Security** dropdown and then **Global Session Policy**
2. Next to the **Kennon Test Session Policy** open the **Active** dropdown and select **Deactivate**
3. Press **Delete**

<p align="center">
  <img src="Images/Image%20121.PNG" alt="Image 121">
</p>

### Remove the Kennon HQ Zone

1. Open the **Security** dropdown and then **Networks**
2. Next to **Kennon HQ** open the **Active** dropdown then select **Inactive**
3. Press the **X**

<p align="center">
  <img src="Images/Image%20122.png" alt="Image 122">
</p>

### Remove the Added Policy Rules

1. Open the **Security** dropdown and then **Authentication Policies** then **Okta Account Management**
2. Find **Test Groups Enrollment Rule** then open the **Actions** dropdown then **Deactivate**
3. Open the **Actions** dropdown again then **Delete**

<p align="center">
  <img src="Images/Image%20123.PNG" alt="Image 123">
</p>

4. At the top of the page press **Back to All Authentication Policies** then press **App Sign-In**
5. Open **Okta Dashboard** then find the **Standard Dashboard Rule**
6. Open the **Actions** dropdown then **Deactivate**
7. Open the **Actions** dropdown again then press **Delete**

<p align="center">
  <img src="Images/Image%20124.PNG" alt="Image 124">
</p>

### Remove Enrollment Policies

1. Open the **Security** dropdown then **Authenticators** then open the **Enrollment** tab
2. Find the **Standard Enrollment Policy** then open the **Actions** dropdown and press **Delete**

<p align="center">
  <img src="Images/Image%20125.PNG" alt="Image 125">
</p>

3. Find the **HighSecurity Enrollment Policy** then open the **Actions** dropdown and press **Delete**

<p align="center">
  <img src="Images/Image%20126.PNG" alt="Image 126">
</p>

### Remove Password Policy

1. Open the **Security** dropdown then **Authenticators**
2. Find **Password** then open the **Actions** dropdown and select **Edit**
3. Select **HighSecurity Password Policy** and press **Delete**

<p align="center">
  <img src="Images/Image%20127.PNG" alt="Image 127">
</p>

### Revert Okta Verify

1. Open the **Security** dropdown then **Authenticators**
2. Find **Okta Verify** then open the **Actions** dropdown and select **Edit**
3. Set **Number Challenge for Okta Verify Push** to **Never** then **Save**

<p align="center">
  <img src="Images/Image%20128.PNG" alt="Image 128">
</p>

### Remove the Kennon Intranet App

1. Open the **Applications and Resources** dropdown then **Applications** then open **Kennon Intranet**
2. Open the **Active** dropdown then **Deactivate**
3. Open the **Inactive** dropdown then **Delete**

<p align="center">
  <img src="Images/Image%20129.PNG" alt="Image 129">
</p>

### Remove Test Users

1. Open the **Directory** dropdown then **People** then open **Morgan Reyes's** profile
2. Open the **More Actions** dropdown then **Deactivate**
3. Then press **Delete**

<p align="center">
  <img src="Images/Image%20130.png" alt="Image 130">
</p>

4. Open **Jordan Ellis's** profile then open the **More Actions** dropdown then **Deactivate**
5. Then press **Delete**

<p align="center">
  <img src="Images/Image%20131.png" alt="Image 131">
</p>

### Remove Groups

1. Open the **Directory** dropdown then **Groups** then open **AuthPolicy HighSecurity**
2. Open the **Actions** dropdown then **Delete**

<p align="center">
  <img src="Images/Image%20132.PNG" alt="Image 132">
</p>

3. Open the **AuthPolicy Standard** group then open the **Actions** dropdown then **Delete**

<p align="center">
  <img src="Images/Image%20133.PNG" alt="Image 133">
</p>

### Verify in System Log

1. Open the **Reports** dropdown then **System Log**
2. Search for policy deletions to confirm every policy from this lab was removed:
   ```text
   eventType eq "policy.lifecycle.delete"
   ```
   **Delete policy** events appear for **Kennon Intranet Policy**, **Kennon Test Session Policy**, **Standard Enrollment Policy**, **HighSecurity Enrollment Policy** and **HighSecurity Password Policy**, in reverse order of creation

<p align="center">
  <img src="Images/Image%20134.PNG" alt="Image 134">
  <img src="Images/Image%20135.png" alt="Image 135">
</p>

3. Search for policy rule deletions to confirm the rules added to built in policies were removed:
   ```text
   eventType eq "policy.rule.delete"
   ```
   **Delete policy rule** events appear for **Test Groups Enrollment Rule** and **Standard Dashboard Rule**, along with the rules Okta removed automatically when their policies were deleted

<p align="center">
  <img src="Images/Image%20136.PNG" alt="Image 136">
  <img src="Images/Image%20137.png" alt="Image 137">
</p>

4. Search for network zone deletions to confirm Kennon HQ was removed:
   ```text
   eventType eq "zone.delete"
   ```
   The **Network zone delete** event appears with your admin account as the actor and **Kennon HQ** as the target

<p align="center">
  <img src="Images/Image%20138.PNG" alt="Image 138">
  <img src="Images/Image%20139.png" alt="Image 139">
</p>

5. Search for application deletions to confirm Kennon Intranet was removed:
   ```text
   eventType eq "application.lifecycle.delete"
   ```
   The application delete event appears with your admin account as the actor and **Kennon Intranet** (shown as **Bookmark App**) as the target

<p align="center">
  <img src="Images/Image%20140.PNG" alt="Image 140">
  <img src="Images/Image%20141.png" alt="Image 141">
</p>

6. Search for completed user deletions to confirm both test users were removed:
   ```text
   eventType eq "user.lifecycle.delete.completed"
   ```
   Delete events appear for **Morgan Reyes** and **Jordan Ellis** with your admin account as the actor

<p align="center">
  <img src="Images/Image%20142.PNG" alt="Image 142">
  <img src="Images/Image%20143.png" alt="Image 143">
</p>

7. Search for group deletions to confirm both test groups were removed:
   ```text
   eventType eq "group.lifecycle.delete"
   ```
   Delete events appear for **AuthPolicy HighSecurity** and **AuthPolicy Standard** with your admin account as the actor

<p align="center">
  <img src="Images/Image%20144.PNG" alt="Image 144">
  <img src="Images/Image%20145.png" alt="Image 145">
</p>

---

## Closing Note

This lab built a layered authentication setup for Kennon Technologies using authenticators, enrollment policies, a network zone, a global session policy, and an app level authentication policy. Sign ins inside Kennon HQ had less friction, while sign ins outside the zone required MFA. HighSecurity users were held to Okta Verify by the app policy even when the session policy required nothing more than a password.

Troubleshooting also showed a few behaviors worth remembering. An optional enrollment policy doesn't help if the authentication policy only accepts factors the user hasn't enrolled. Newer enrollment policies depend on the Okta Account Management Policy. Okta evaluates rules from the top down and applies the first match. Together, these show why policy order and scope matter as much as the settings themselves.
