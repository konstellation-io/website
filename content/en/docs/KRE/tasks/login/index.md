---
title: "Login"
linkTitle: "Login"
description: >
  This tutorial guides you through the process of login.
weight: 10
---

KRE login is a passwordless process so you only need to provide your email:

{{< imgproc login_screen Resize "600x" >}}

{{< /imgproc >}}

After submitting your email address you will see the following screen:

{{< imgproc login_passwordless Resize "600x" >}}

{{< /imgproc >}}

You will receive an email with a login link:

{{< imgproc login_link Resize "600x" >}}

{{< /imgproc >}}

Clicking the login link you will navigate to the runtime list screen.

{{< imgproc user_viewer Resize "1000x" >}}

{{< /imgproc >}}

If you are a new user, your role will be `viewer` so you should ask to the admin user for privileges for edition and creation (`manager` role).
An `admin` user can change the user role using the user administration interface (`/settings/users`):

{{< imgproc user_admin Resize "1000x" >}}

{{< /imgproc >}}

### User roles

There are three roles for users in KRE. The following table shows the available actions for each role/resource:

| Role    | Description                                                 | Metrics | Runtimes  | Versions  | Audits | Logs | Settings  | Users     |
| ------- | ----------------------------------------------------------- | ------- | --------- | --------- | ------ | ---- | --------- | --------- |
| admin   | Full control                                                | view    | view/edit | view/edit | view   | view | view/edit | view/edit |
| manager | Can do anything but management of settings and users        | view    | view/edit | view/edit | view   | view | -         | -         |
| viewer  | Only can see the metrics, created runtimes and its versions | view    | view      | view      | -      | -    | -         | -         |
