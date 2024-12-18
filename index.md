Challenge Name: Armaxis

Challenge Description: In the depths of the Frontier, Armaxis powers the enemy’s dominance, dispatching weapons to crush rebellion. Fortified and hidden, it controls vital supply chains. Yet, a flaw whispers of opportunity, a crack to expose its secrets and disrupt their plans. Can you breach Armaxis and turn its power against tyranny?

Category: Web

Difficulty: Very easy

# Preface

To solve this challenge, you will need to perform Authentication Bypass and Local File Inclusion vulnerabilities. The challenge page provided the option to download files for analysis, which you can obtain via the link: https://github.com/StepQuest/htb-uni-ctf-web-writeup-2024/blob/main/web_armaxis.zip

## Skills Required

* Basic understanding of JavaScript
* HTTP requests interception
* Vulnerability research skills

## Skills Learned

* Analyzing JavaScript code
* Login Authentication Bypass
* Markdown Local File Inclusion

# Challenge Overview

The challenge begins with us being provided two addresses: one hosting a web application called Armaxis with a login form, and another simulating a mailbox.

Here, we see that account registration, login, and password recovery are available.

![](assets/armaxis.png)

Here, we see a mailbox belonging to the address `test@email.htb`, which currently has no messages. There's also a button that deletes all messages from the mailbox.

![](assets/mailinbox.png)

# Curiosities in Code

At an earlier stage, by analyzing the relatively simple JavaScript code, we can identify the existence of the `admin@armaxis.htb` email, which is mentioned in the `/web_armaxis/challenge/views/database.js` file.

![](assets/admin_mail.png)

# Walkthrough

Perform a password reset for `test@email.htb`

![](assets/reset_password.png)

As a result, we will receive a random token in the mailbox, which will need to be used to reset the password.

![](assets/password_token.png)

On the password recovery page, enter the token and a new password. Then, open Burp Suite, click the button to confirm the password change, and intercept the request.

In the intercepted packet, replace `test@mail.htb` with `admin@armaxis.htb`  and forward the package. As a result, we will have successfully changed the password for the admin account.

![](assets/chane_testmail_to_admimail.png)

After logging into the admin account, we are redirected to the `http://site.com/weapons` page, which appears empty. However, we can see that the Dispatch Weapon page is available.

![](assets/admin_panel.png)

On the Dispatch Weapon page, we see a form with multiple fields. After filling in all the fields with test values, let's check the result.

![](assets/filled_test_form.png)

We receive a table displaying the previously entered values.

![](assets/test_table_row.png)

By analyzing `/web_armaxis/challenge/views/markdown.js`, we can see that it allows us to insert images. This could potentially be a vulnerability. 

![](assets/markdown_url.png)

Let's perform a test to disclose the `/etc/passwd` file. First, we need to find the request in Burp Suite's history where we submitted the form with test values.

![](assets/test_request_preedit.png)

Replace the value of the `note` field with `![url](file:///etc/passwd)` and send the request. In the response, we can see that our "weapon" has been successfully dispatched.

![](assets/url_etc_passwd.png)

On the weapons table page, let's download our image.

![](assets/img_etc_passwd.png)

Finally, we open the downloaded image file. As a result, we see the successful exploitation of an LFI vulnerability, displaying the contents of the `/etc/passwd` file. Knowing this, we can easily retrieve our flag.

You can take an alternative approach by using the browser's developer tools. Copy the base64-encoded value of the image and decode it to retrieve the data. Overall, it doesn't matter which method you choose to go with.

![](assets/etc_passwd.png)


# Solution

Repeat the step with modifying the `note` field value, but this time use `![url](file:///flag.txt)`.

![](assets/url_flag.png)


After downloading the new image from the table and opening the file, we will obtain our flag.

![](assets/flag.png)
