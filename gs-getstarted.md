---
icon: address-card
---

# Get Started

## Software Registration

If you are a new user, please consult the [Illumina Connected Software Registration Guide](https://help.connected.illumina.com/account-management/rg-registration) for detailed guidance on setting up an account and registering a subscription.

## Tenant Setup

The platform requires a provisioned tenant in the[ **Illumina account management**](https://help.connected.illumina.com/account-management/admin-console) ([IAM](https://help.connected.illumina.com/account-management/admin-console)) system with access to the **Illumina Connected Analytics** (ICA) application. Once a tenant has been provisioned, a tenant administrator will be assigned. The **tenant administrator** has permission to manage account access including add users, create workgroups, and add additional tenant administrators.

Each tenant is assigned a domain name used to login to the platform. The domain name is used in the login URL to navigate to the appropriate login page in a web browser. The login URL is `https://<domain_name>.login.illumina.com`  with `<domain_name>` replaced by the actual domain name.

**New user accounts** can be created

For more details on identity and access management, please see the [Illumina Connected Software](https://help.connected.illumina.com/) help.

{% hint style="info" %}
If you have intrusion detection systems active on your infrastructure, be aware that activities performed by ICA on your behalf (such as accessing [your own S3 storage](../home/h-storage/s-awss3/) ) might trigger suspicious activity alerts. Please review the alerts and rules with your vendor to set up appropriate policies on your detection system.
{% endhint %}

* by the **tenant administrator** by logging in to their domain and navigating to **Illumina Account Management** under their profile at the top right
* or by the **user** by accessing `https://platform.login.illumina.com` and selecting the option **Don't have an account**. 

Once the account has been added to the domain, the tenant administrator may assign registered users to [workgroups](https://help.connected.illumina.com/account-management/admin-console/workgroups) which bundle users with permission to use the ICA application. Registered users can be made workgroup administrators by tenant administrators or existing workgroup administrators.

## API Keys

If you want to use the [**command-line interface**](broken-reference) (CLI) or the [**Application Programming Interface**](../reference/r-api.md) (API), you can use an [API Key ](https://help.connected.illumina.com/account-management/platform-home)as credentials when logging in. API Keys operate similar to a user name and password combination and must be **kept secure** and **rotated on a regular basis** (preferably yearly). \`

When **keys are compromised or no longer in use, they must be revoked**. This is done through the [domain login URL](https://ilmn.login.illumina.com/platform-home/#/home) by navigating to the User menu item on the left and selecting "API Keys", followed by selecting the key and using the trash icon next to it.

## Generate an API Key
## API Keys

Some text here.

{% hint style="warning" %}
For security reasons, do **not use accounts with administrator level access** to generate API keys. Create a specific CLI user with basic permissions instead. This will minimize the possible impact of compromised keys.
{% endhint %}

More text here.

API Keys are limited to 10 per user and are managed through the product dashboard after logging in through the [domain login URL](https://ilmn.login.illumina.com/platform-home/#/home).  See[ Managing API Keys](https://help.connected.illumina.com/account-management/platform-home#manage-api-keys) for more information.
{% hint style="warning" %}
For security reasons, do **not use accounts with administrator level access** to generate API keys. Create a specific CLI user with basic permissions instead. This will minimize the possible impact of compromised keys.
{% endhint %}

{% hint style="warning" %}
Once the API key generation window is closed, the key contents will not be accessible through the domain login page, so be sure to store it securely for future reference.
{% endhint %}

***

## Access via Web UI

The web application provides a visual user interface (UI) for navigating resources in the platform, managing projects, and extended features beyond the API. To access the web application, navigate to the [Illumina Connected Analytics portal](https://ica.illumina.com/ica).

* On the **left**, you have the **navigation bar** (1) which will auto-collapse on smaller screens. To  collapse it, use the **double arrow symbol** (2). When collapsed, use the >> symbol to expand it.
* The **central** **part** (3) of the display is the item on which you are **performing your actions** and the **breadcrumb** **menu** (4) to return to the projects overview or a previous level. You can also use your browser's back button to return to the level from which you came.
* At the top **right**, you have icons to **refresh contents** (5)  I**llumina product access** (6), access to the **online help** (7) and **user** **information** (8), .

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

## Access via the CLI

The command-line interface offers a developer-oriented experience for interacting with the APIs to manage resources and launch analysis workflows. Find instructions for using the command-line interface including download links for your operating system in the [CLI documentation](../command-line-interface/cli-installation.md).

## Access via the API

The HTTP-based application programming interfaces (APIs) are listed in the [API Reference](../reference/r-api.md) section of the documentation. The reference documentation provides the ability to call APIs from the browser page and shows detailed information about the API schemas. HTTP client tooling such as Postman or cURL can be used to make direct calls to the API outside of the browser.

{% hint style="info" %}
When accessing the API using the API Reference page or through REST client tools, the `Authorization` header must be provided with the value set to `Bearer <token>` where `<token>` is replaced with a valid JSON Web Token (JWT). For generating a JWT, see [JSON Web Token (JWT)](gs-getstarted.md#json-web-token-jwt).
{% endhint %}

***

## Object Identifiers

The object data models for resources that are created in the platform include a unique `id` field for identifying the resource. These fixed machine-readable IDs are used for accessing and modifying the resource through the API or CLI, even if the resource name changes.

## JSON Web Token (JWT)

Accessing the platform APIs requires authorizing calls using JSON Web Tokens (JWT). A JWT is a standardized trusted claim containing authentication context. This is a primary security mechanism to protect against unauthorized cross-account data access.

A JWT is generated by providing user credentials (API Key or username/password) to the token creation endpoint. Token creation can be performed using the API directly or the CLI.
