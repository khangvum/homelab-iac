# Jellyfin-Authentik Integration Guide

A comprehensive guide to **_integrating Authentik LDAP_** with a **_Jellyfin_** instance in a homelab environment.

## 1. Authentik Configuration

### Provider Setup

- Navigate to **Applications** > **Providers** and create an **_LDAP Provider_**.
- Fill out the details:

  |      Field      | Setting                                               |
  | :-------------: | ----------------------------------------------------- |
  |    **Name**     | `Jellyfin - LDAP`                                     |
  |  **Bind Flow**  | `ldap-authentication-flow (LDAP Authentication Flow)` |
  | **Unbind Flow** | `default-invalidation-flow (Logout)`                  |
  |   **Base DN**   | `DC=khangvum,DC=lab`                                  |
  | **Certificate** | `authentik Self-signed Certificate`                   |

### Application Setup

- Navigate to **Applications** > **Applications** and create a **_New Application_**:
- Fill out the details:

  |    Field     | Setting                  |
  | :----------: | ------------------------ |
  |   **Name**   | `Jellyfin`               |
  | **Provider** | Select `Jellyfin - LDAP` |

### Outpost Setup

- Navigate to **Applications** > **Outposts** and create a **_New Outpost_**:
- Fill out the details:

  |      Field       | Setting                 |
  | :--------------: | ----------------------- |
  | **Outpost Name** | `Jellyfin LDAP Outpost` |
  |     **Type**     | `LDAP`                  |
  | **Applications** | Select `Jellyfin`       |

- After creating the outpost, navigate to **Directory** > **Tokens and App passwords**.
- Locate the newly created **_Jellyfin LDAP Outpost_** (_e.g.,_ `ak-outpost-...-api`), and **_copy_** the **_outpost token_**.
- Add the **_LDAP outpost container_** to the `docker-compose.yml` file using the outpost token retrieved from Authentik:

  ```yaml
  authentik_ldap:
    image: ghcr.io/goauthentik/ldap:2026.8.2
    restart: unless-stopped
    ports:
      - "389:3389"
      - "636:6636"
    environment:
      AUTHENTIK_HOST: https://authentik.khangvum.com
      AUTHENTIK_INSECURE: "false"
      AUTHENTIK_TOKEN: "{{ authentik_outpost_token }}"
    depends_on:
      - server
  ```

> [!TIP]
> If deployed successful navigate back to **Applications** > **Outposts** > **Health and Version** in Authentik. Check the **_Health and Version_** status; it should update to show that the outpost is actively connected, displaying a **_recent timestamp_** such as `Last seen: 5 seconds ago (12:43:11 PM)`.

## 2. Jellyfin Plugin Configuration

### Plugin Installation

- Log in to Jellyfin instance as **_Administrator_**.
- Navigate to **Dashboard** > **Plugins**.
- Search for **_LDAP Authentication_** (**_LDAP-Auth_**), and click **_Install_**.

> [!IMPORTANT]
> **_Restart Jellyfin_** to initialize the plugin.

### Plugin Settings

Once restarted, click on the **_LDAP-Auth_** plugin icon in the installed plugins list to **_configure the connection_**:

|               Field                | Value                                                    |
| :--------------------------------: | -------------------------------------------------------- |
|          **LDAP Server**           | (The **_Authentik Outpost's IP_** (_e.g.,_ `KVM-IAM01`)) |
|           **LDAP Port**            | `636`                                                    |
|          **Secure LDAP**           | `CHECKED`                                                |
|   **Skip SSL/TLS Verification**    | `CHECKED`                                                |
|         **LDAP Bind User**         | `cn=akadmin,ou=users,dc=khangvum,dc=lab`                 |
|    **LDAP Bind User Password**     | (`akadmin`'s password)                                   |
|   **LDAP Base DN for searches:**   | `dc=khangvum,dc=lab`                                     |
|       **LDAP Search Filter**       | `(&(objectClass=user))`                                  |
|     **LDAP Search Attributes**     | `sAMAccountName, cn, mail, displayName`                  |
|       **LDAP Uid Attribute**       | `sAMAccountName`                                         |
|    **LDAP Username Attribute**     | `sAMAccountName`                                         |
|      **Enable User Creation**      | `CHECKED`                                                |
| **Enable access to all libraries** | `CHECKED`                                                |

> [!IMPORTANT]
> **_Restart Jellyfin_** again after saving these settings for the changes to **_take effect_**.

### References

[Integrate with Jellyfin](https://integrations.goauthentik.io/media/jellyfin/)
