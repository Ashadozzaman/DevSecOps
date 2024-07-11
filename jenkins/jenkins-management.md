Let's break down the steps to achieve the setup you need using the Role-Based Authorization Strategy Plugin in Jenkins.

### Step 1: Install the Role-Based Authorization Strategy Plugin

1. **Manage Jenkins** > **Manage Plugins**.
2. Click on the **Available** tab.
3. Search for **Role-based Authorization Strategy**.
4. Check the box next to it and click **Install without restart** or **Download now and install after restart**.

### Step 2: Create Users

1. **Manage Jenkins** > **Manage Users**.
2. Click on **Create User**.
3. Create three users: `admin`, `intern`, and `reader` with appropriate usernames and passwords.

### Step 3: Configure Role-Based Authorization Strategy

1. **Manage Jenkins** > **Configure Global Security**.
2. Under **Authorization**, select **Role-Based Strategy**.
3. Click on **Save**.

### Step 4: Manage and Assign Roles

1. **Manage Jenkins** > **Manage and Assign Roles** > **Manage Roles**.

#### Create Global Roles
1. **Global roles** section:
    - **admin**: Check all permissions.
    - **reader**: Check **Overall Read**, **Job Read**, and **Job Build**.
    - **intern**: Check **Overall Read**.

#### Create Item Role
1. **Item roles** section:
    - **intern**: Add role with pattern `intern.*` and check all permissions.

### Step 5: Assign Roles

1. **Manage Jenkins** > **Manage and Assign Roles** > **Assign Roles**.

#### Assign Global Roles
1. **Global roles** section:
    - Assign `admin` role to the `admin` user.
    - Assign `reader` role to the `reader` user.
    - Assign `intern` role to the `intern` user.

#### Assign Item Roles
1. **Item roles** section:
    - Assign `intern` role to the `intern` user with the pattern `intern.*`.

### Summary

- **Global Roles**:
  - **admin**: Full permissions.
  - **reader**: Overall Read, Job Read, Job Build.
  - **intern**: Overall Read.

- **Item Roles**:
  - **intern**: Full permissions for items matching `intern.*`.

This configuration will give admins full access, interns access to pipelines starting with "intern", and readers read and build access.

### Example Walkthrough

#### Step 2: Create Users
- Username: `admin`
  - Password: `admin_password`
  - Full Name: `Admin User`
  - Email: `admin@example.com`
  
- Username: `intern`
  - Password: `intern_password`
  - Full Name: `Intern User`
  - Email: `intern@example.com`
  
- Username: `reader`
  - Password: `reader_password`
  - Full Name: `Reader User`
  - Email: `reader@example.com`

#### Step 4: Manage Roles
- **Global Roles**:
  - **admin**: All permissions checked.
  - **reader**: 
    - Overall: Read
    - Job: Read, Build
  - **intern**:
    - Overall: Read

- **Item Role**:
  - **intern**:
    - Pattern: `intern.*`
    - All permissions checked

#### Step 5: Assign Roles
- **Global Roles**:
  - **admin**: Assign to `admin`
  - **reader**: Assign to `reader`
  - **intern**: Assign to `intern`

- **Item Roles**:
  - **intern**: Assign to `intern` with pattern `intern.*`

This setup will ensure the correct permissions for the admin, intern, and reader users.