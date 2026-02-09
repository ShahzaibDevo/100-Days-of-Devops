

# Day 76: Jenkins Project Security

**Lab:** Grant Project Permissions to Users (sam & rohan)

**Objective:**
Provide specific job-level permissions to Jenkins users while ensuring they can access the dashboard with minimum required global permissions.

---

## ✅ Prerequisites

* Jenkins installed and running (tested on Jenkins 2.492.1)
* Admin credentials:

  ```
  Username: admin
  Password: Adm!n321
  ```
* Existing users:

  * `sam` → sam@pass12345
  * `rohan` → rohan@pass12345
* Existing job: `Packages`

---

## Step 1: Login as Admin

1. Open Jenkins UI → click **Jenkins** button.
2. Login using admin credentials.

---

## Step 2: Enable Project-Based Security (if not already enabled)

1. Go to:

   ```
   Manage Jenkins → Configure Global Security
   ```
2. Under **Authorization**, select:

   ```
   Project-based Matrix Authorization Strategy
   ```
3. Scroll down → click **Save**

💡 *Tip:* If using **Matrix-based security**, ensure you refresh after enabling to avoid access issues.

---

## Step 3: Add Users

1. In **Project-based Matrix** section → click **Add User**.
2. Add users one by one:

   * `sam`
   * `rohan`

---

## Step 4: Grant Job Permissions

### For `sam`:

| Permission | Granted? |
| ---------- | -------- |
| Build      | ✅        |
| Configure  | ✅        |
| Read       | ✅        |

### For `rohan`:

| Permission | Granted? |
| ---------- | -------- |
| Build      | ✅        |
| Cancel     | ✅        |
| Configure  | ✅        |
| Read       | ✅        |
| Update     | ✅        |
| Tag        | ✅        |

💡 *Note:* Select **Inherit permissions from parent ACL**.

---

## Step 5: Grant Global Read Permission

> Without this, users see **Access Denied**.

1. Go to:

   ```
   Manage Jenkins → Configure Global Security → Matrix-based security
   ```
2. For each user (`sam` and `rohan`), grant:

   ```
   Overall → Read
   ```
3. Save changes.

---

## Step 6: Test User Access

1. Logout admin → login as `sam`.

2. Check:

   * Dashboard visible ✅
   * Packages job accessible ✅
   * Can build, configure, read ✅

3. Logout `sam` → login as `rohan`.

4. Check:

   * Dashboard visible ✅
   * Packages job accessible ✅
   * Can build, configure, cancel, update, tag, read ✅

---

## Step 7: Job Run Verification

* Trigger **Build Now** for Packages job.
* Console output should show successful build commands, e.g., installing packages via SSH.

💡 *Example Output:*

```
Started by user admin
Building in workspace /var/lib/jenkins/workspace/Packages
+ sshpass -p Ir0nM@n ssh ...
...
Total download size: 18 M
Installed size: 77 M
Transaction check succeeded.
```

---

## ✅ Lab Completion Checklist

* [x] Users `sam` and `rohan` exist
* [x] Project permissions granted correctly
* [x] Global Overall/Read granted
* [x] Both users can access dashboard and job
* [x] Packages job can be built successfully
* [x] No modifications to other jobs

---

## Optional: Screenshots / Recording Notes

* Capture **Global Security Matrix**
* Capture **Project Permission Matrix**
* Capture **Job build console output**
* Optional: Use Loom or OBS to record full steps

---

## Conclusion

Lab successfully demonstrates **Jenkins Project Security**:

* Correct separation of **global vs job permissions**
* Users can access required jobs without over-permission
* Jenkins security best practices applied

---

You can save this file as:

```
Day76_Jenkins_Project_Security.md
```

* Open in **VSCode** version 6 → copy/paste → GitHub commit
* Include screenshots in `/images` folder and link them in markdown if desired

---

If you want, I can also **prepare the GitHub-ready folder structure** for Day 76 including **screenshots, markdown, and VSCode project setup** so you just commit and push.

Do you want me to do that?
