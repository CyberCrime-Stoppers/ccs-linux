| Directory | Purpose | Users |
|---|---|---|
| `/bin` | Essential user commands needed even in single-user/no-network boot | All users |
| `/sbin` | System administration binaries (often need root) | Root/admins |
| `/usr/bin` | The bulk of user-facing programs | All users |
| `/usr/sbin` | Non-essential admin tools | Root/admins |
| `/usr/local/bin` | Locally compiled/installed software — untouched by the package manager | Site admin |
| `/opt/<vendor>/bin` | Self-contained third-party packages (Chrome, some commercial apps) | Varies |
| `/snap/bin` | Snap package shims | All users |

**What lives where, concretely if you were to have CCS Install Apps:**

- **`/bin`** (traditional) may containe: `pdua`, `proton-drive`, 
- **`/bin/usr`** (traditional): `pdua`,`proton-drive`,
