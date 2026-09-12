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

- 

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
- Navigate to **Dashboard** > **Plugins** > **Manage Repositories**.
- Click **New Repository**, and fill out the details:

  |        Field        | Value                                                                                      |
  | :-----------------: | ------------------------------------------------------------------------------------------ |
  | **Repository Name** | `SSO-Auth`                                                                                 |
  | **Repository URL**  | `https://raw.githubusercontent.com/9p4/jellyfin-plugin-sso/manifest-release/manifest.json` |

- Go back to **Plugins**, search for **_SSO-Auth_**, and click **_Install_**.

> [!IMPORTANT]
> **_Restart Jellyfin_** to initialize the plugin.

### Plugin Settings

Once restarted, click on the **_SSO-Auth_** plugin icon in the installed plugins list to **_configure the connection_**:

|                     Field                     | Value                                                                                   |
| :-------------------------------------------: | --------------------------------------------------------------------------------------- |
|          **Name of OpenID Provider**          | `authentik`                                                                             |
|              **OpenID Endpoint**              | `http://authentik.khangvum.lab/application/o/jellyfin/.well-known/openid-configuration` |
|             **OpenID Client ID**              | (Paste the **_Client ID_** from Authentik)                                              |
|           **OpenID Client Secret**            | (Paste the **_Client Secret_** from Authentik)                                          |
|                  **Enabled**                  | `CHECKED`                                                                               |
|      **Enable Authorization by Plugin**       | `CHECKED`                                                                               |
| **Disable OpenID HTTPS Discovery (Insecure)** | `CHECKED`                                                                               |

> [!IMPORTANT]
> **_Restart Jellyfin_** again after saving these settings for the changes to **_take effect_**.

## 3. Login Branding

To display the **_"Sign in with SSO"_** button, inject this HTML into the **_Login disclaimer_** (found in **Dashboard** > **Branding**):

```html
<form action="https://jellyfin.khangvum.com/sso/OID/start/authentik">
  <button class="raised block emby-button button-submit">
    Sign in with SSO
  </button>
</form>
```

### References

[Integrate with Jellyfin](https://integrations.goauthentik.io/media/jellyfin/)
