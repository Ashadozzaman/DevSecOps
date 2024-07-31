https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```
aws

aws configure

```


### Get Access Credential

If you are already logged in as an IAM user and need your own access credentials for the AWS CLI, you can follow these steps to create or retrieve your access keys:

1. **Navigate to the IAM Dashboard:**

   Go to the [IAM Dashboard](https://console.aws.amazon.com/iam/home).

2. **Access Your User Details:**

   - In the IAM Dashboard, click on "Users" in the left-hand navigation pane.
   - Find and click on your IAM username in the list. This will take you to the details page for your IAM user.

3. **Create Access Keys:**

   - On the user details page, click on the "Security credentials" tab.
   - Scroll down to the "Access keys" section.
   - Click on the "Create access key" button.

4. **Download Access Keys:**

   - After creating the access keys, a dialog will appear with your Access Key ID and Secret Access Key.
   - **Important:** Download the .csv file or copy the keys to a secure location. This is the only time you will be able to view the Secret Access Key.

5. **Configure AWS CLI:**

   Open your Command Prompt or PowerShell and run the `aws configure` command:

   ```sh
   aws configure
   ```

   Enter the Access Key ID and Secret Access Key you obtained, along with your default region and output format.

   ```plaintext
   AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
   AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
   Default region name [None]: YOUR_DEFAULT_REGION
   Default output format [None]: json
   ```

Replace `YOUR_ACCESS_KEY_ID`, `YOUR_SECRET_ACCESS_KEY`, `YOUR_DEFAULT_REGION`, and `json` with your actual values.
