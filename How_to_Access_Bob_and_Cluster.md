# How to Access Bob and the OpenShift Cluster
## LAB-1512 | Manufacturing Defect Inspection

Use this page together with the Student Lab Guide. Replace `studentXX` with your own student ID (for example `student04`).

---

## 1. Open Bob on your lab desktop

Bob runs on the remote desktop of your lab machine.

1. Open the TechZone event page:
   https://techzone.ibm.com/my/events/6abf282026700b992039ef25/6abf295798af3b77cce233d1/6abf296198af3b77cce233e4
2. Find the reservation named **Student N - LAB-1512** that matches your student ID. Its status should be **Ready**.
3. Open the **Guacamole remote desktop** link on that reservation:
   https://vdi.cloud.techzone.ibm.com/guacamole
4. Log in with the desktop username and password shown on your reservation (the username is `itzuser`).
5. Open Bob from the desktop and keep it open next to your browser.

If you cannot find your reservation or Bob, ask your instructor before continuing.

---

## 2. OpenShift cluster details

| Item | Value |
|------|-------|
| API URL | `https://api.itz-t53413.hub01-lb.techzone.ibm.com:6443` |
| OpenShift console | https://console-openshift-console.apps.itz-t53413.hub01-lb.techzone.ibm.com |
| RHOAI Dashboard | https://rh-ai.apps.itz-t53413.hub01-lb.techzone.ibm.com |
| Username | `kubeadmin` |
| Password | `pwXIy-fVwDX-25Vp8-MCeG3` |
| OpenShift version | 4.19 |

These are the shared lab credentials used in **Step 1** of the lab guide (**Log in with OpenShift**).

---

## 3. Your own resources

Your project, S3 bucket and workbench all use your student ID:

| Resource | URL |
|----------|-----|
| Your workbench | `https://rh-ai.apps.itz-t53413.hub01-lb.techzone.ibm.com/notebook/studentXX/my-workbench/lab` |
| Inference endpoint | `https://defect-classifier-ext-studentXX.apps.itz-t53413.hub01-lb.techzone.ibm.com` |
| Web app | `https://image-rec-app-studentXX.apps.itz-t53413.hub01-lb.techzone.ibm.com` |

---

## 4. Please be careful

The login above is a **shared cluster-admin account**. It can see and change every student's project.

- Only work in your own `studentXX` project.
- Do not delete, edit or restart anything that belongs to another student.
- Do not change cluster-wide settings.
- Do not share these credentials outside this lab.
