---
title: "Connect Snowflake"
id: connect-snowflake
description: "Configure Snowflake connection."
sidebar_label: "Connect Snowflake"
---

The following fields are required when creating a Snowflake connection

| Field | Description | Examples |
| ----- | ----------- | -------- |
| Account | The Snowflake account to connect to. Take a look [here](/docs/core/connect-data-platform/snowflake-setup#account) to determine what the account field should look like based on your region.| <Snippet path="snowflake-acct-name" /> |
| Role | A mandatory field indicating what role should be assumed after connecting to Snowflake | `transformer` |
| Database | The logical database to connect to and run queries against. | `analytics` |
| Warehouse | The virtual warehouse to use for running queries. | `transforming` |


**Note:** A crucial part of working with dbt atop Snowflake is ensuring that users (in development environments) and/or service accounts (in deployment to production environments) have the correct permissions to take actions on Snowflake! Here is documentation of some [example permissions to configure Snowflake access](/reference/database-permissions/snowflake-permissions).

### Username / Password

**Available in:** Development environments, Deployment environments

The `Username / Password` auth method is the simplest way to authenticate
Development or Deployment credentials in a dbt project. Simply enter your Snowflake
username (specifically, the `login_name`) and the corresponding user's Snowflake `password`
to authenticate dbt Cloud to run queries against Snowflake on behalf of a Snowflake user.

**Note**: The schema field in the **Developer Credentials** section is a required field.
<Lightbox src="/img/docs/dbt-cloud/snowflake-userpass-auth.png" title="Snowflake username/password authentication"/>

### Key pair

**Available in:** Development environments,  Deployment environments

The `Keypair` auth method uses Snowflake's [Key Pair Authentication](https://docs.snowflake.com/en/user-guide/key-pair-auth) to authenticate Development or Deployment credentials for a dbt Cloud project.

1. After [generating an encrypted key pair](https://docs.snowflake.com/en/user-guide/key-pair-auth.html#configuring-key-pair-authentication), be sure to set the `rsa_public_key` for the Snowflake user to authenticate in dbt Cloud:

   ```sql
   alter user jsmith set rsa_public_key='MIIBIjANBgkqh...';   
   ```

2. Finally, set the **Private Key** and **Private Key Passphrase** fields in the **Credentials** page to finish configuring dbt Cloud to authenticate with Snowflake using a key pair.
   - **Note:** Unencrypted private keys are permitted. Use a passphrase only if needed. Starting from [dbt version 1.7](/docs/dbt-versions/core-upgrade/upgrading-to-v1.7), dbt introduced the ability to specify a `private_key` directly as a string instead of a `private_key_path`. This `private_key` string can be in either Base64-encoded DER format, representing the key bytes, or in plain-text PEM format. Refer to [Snowflake documentation](https://docs.snowflake.com/en/user-guide/key-pair-auth) for more info on how they generate the key.

3. To successfully fill in the Private Key field, you _must_ include commented lines. If you receive a `Could not deserialize key data` or `JWT token` error, refer to [Troubleshooting](#troubleshooting) for more info. 

**Example:**

```sql
-----BEGIN ENCRYPTED PRIVATE KEY-----
< encrypted private key contents here - line 1 >
< encrypted private key contents here - line 2 >
< ... >
-----END ENCRYPTED PRIVATE KEY-----
```

   <Lightbox src="/img/docs/dbt-cloud/snowflake-keypair-auth.png" width="60%" title="Snowflake keypair authentication"/>

### Snowflake OAuth

**Available in:** Development environments, Enterprise plans only

The OAuth auth method permits dbt Cloud to run development queries on behalf of
a Snowflake user without the configuration of Snowflake password in dbt Cloud. For
more information on configuring a Snowflake OAuth connection in dbt Cloud, please see [the docs on setting up Snowflake OAuth](/docs/cloud/manage-access/set-up-snowflake-oauth).
<Lightbox src="/img/docs/dbt-cloud/dbt-cloud-enterprise/database-connection-snowflake-oauth.png" title="Configuring Snowflake OAuth connection"/>

## Configuration

To learn how to optimize performance with data platform-specific configurations in dbt Cloud, refer to [Snowflake-specific configuration](/reference/resource-configs/snowflake-configs).

## Troubleshooting
<!--might need to turn this into details toggle if more troubleshooting items arise -->

If you're receiving a `Could not deserialize key data` or `JWT token` error, refer to the following causes and solutions:

<details>

<summary>Error: <code>Could not deserialize key data</code></summary>

  - <b>Possible cause</b>
 
    - This could be because of mistakes like not copying correctly, missing dashes, or leaving out commented lines.
  - <b>Solution</b> 
    
    - You can copy the key from its source and paste it into a text editor to verify it before using it in dbt Cloud. 
  
</details>

<details>
<summary>Error: <code>JWT token</code></summary>

  - <b>Possible causes</b>
  
    - This could be a transient issue between Snowflake and dbt Cloud. When connecting to Snowflake, dbt gets a JWT token valid for only 60 seconds. If there's no response from Snowflake within this time, you might see a `JWT token is invalid` error in dbt Cloud.
    -  The public key was not entered correctly in Snowflake.
  
  - <b>Solutions</b>

    - dbt needs to retry connections to Snowflake.
    -  Confirm and enter Snowflake's public key correctly. Additionally, you can reach out to Snowflake for help or refer to this Snowflake doc for more info: [Key-Based Authentication Failed with JWT token is invalid Error](https://community.snowflake.com/s/article/Key-Based-Authentication-Failed-with-JWT-token-is-invalid-Error).

</details>
