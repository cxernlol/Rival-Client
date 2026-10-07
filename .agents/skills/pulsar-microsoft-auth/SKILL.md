---
name: rival-microsoft-auth
description: >
  Skill for implementing Microsoft OAuth 2.0 PKCE authentication in Rival Client.
  Use when working on src-tauri/src/auth/: the loopback server, PKCE flow,
  Minecraft token exchange, and secure local token storage.
---

# Rival Client — Microsoft OAuth 2.0 PKCE Skill

## Security Rules (Non-Negotiable)
- **Never log** `access_token`, `refresh_token`, `mc_token`, or `user_hash`.
- Token exchange and storage happen **entirely in Rust** — the frontend never touches raw tokens.
- The loopback server MUST shut down after receiving the auth code (or on timeout).
- Store tokens in the OS keyring via `keyring` crate — never write them to plain files.

---

## Full Microsoft → Minecraft Auth Flow

```
1. Generate PKCE code_verifier (random 64-byte base64url string)
2. Derive code_challenge = BASE64URL(SHA-256(code_verifier))
3. Open browser → Microsoft OAuth URL with code_challenge
4. Spin up Rust HTTP server on 127.0.0.1:PORT
5. Microsoft redirects to 127.0.0.1:PORT?code=AUTH_CODE
6. Exchange AUTH_CODE + code_verifier → Microsoft access_token + refresh_token
7. Exchange MS access_token → Xbox Live token (XBL)
8. Exchange XBL token → XSTS token
9. Exchange XSTS → Minecraft access_token (mc_token)
10. Fetch Minecraft profile (UUID + username)
11. Store mc_token + refresh_token in OS keyring
12. Shut down loopback server
13. Emit `auth_complete` Tauri event to frontend
```

---

## Key Endpoints

| Step | URL |
|---|---|
| MS OAuth authorize | `https://login.microsoftonline.com/consumers/oauth2/v2.0/authorize` |
| MS token exchange | `https://login.microsoftonline.com/consumers/oauth2/v2.0/token` |
| Xbox Live auth | `https://user.auth.xboxlive.com/user/authenticate` |
| XSTS auth | `https://xsts.auth.xboxlive.com/xsts/authorize` |
| Minecraft auth | `https://api.minecraftservices.com/authentication/login_with_xbox` |
| MC profile | `https://api.minecraftservices.com/minecraft/profile` |

MS OAuth Scopes needed: `XboxLive.signin offline_access`

---

## oauth.rs Implementation Skeleton

```rust
use base64::{engine::general_purpose::URL_SAFE_NO_PAD, Engine};
use sha2::{Digest, Sha256};
use uuid::Uuid;

const CLIENT_ID: &str = "YOUR_AZURE_APP_CLIENT_ID";  // Register at portal.azure.com
const REDIRECT_PORT: u16 = 7878;
const REDIRECT_URI: &str = "http://localhost:7878/callback";

pub struct PkceChallenge {
    pub verifier: String,
    pub challenge: String,
}

pub fn generate_pkce() -> PkceChallenge {
    let verifier = URL_SAFE_NO_PAD.encode(Uuid::new_v4().as_bytes());
    let hash = Sha256::digest(verifier.as_bytes());
    let challenge = URL_SAFE_NO_PAD.encode(hash);
    PkceChallenge { verifier, challenge }
}

pub fn build_auth_url(challenge: &str) -> String {
    format!(
        "https://login.microsoftonline.com/consumers/oauth2/v2.0/authorize\
        ?client_id={CLIENT_ID}\
        &response_type=code\
        &redirect_uri={REDIRECT_URI}\
        &scope=XboxLive.signin+offline_access\
        &code_challenge={challenge}\
        &code_challenge_method=S256"
    )
}

/// Starts the loopback server, returns the auth code when received.
/// Times out after 5 minutes.
pub async fn wait_for_auth_code() -> Result<String, String> {
    // Bind to 127.0.0.1:7878
    // Serve one request on /callback?code=...
    // Extract `code` query param
    // Respond with a simple "You can close this window" HTML page
    // Shut down server
    // Return code
    todo!("implement loopback server")
}
```

---

## token.rs — Keyring Storage

```rust
use keyring::Entry;

const SERVICE: &str = "Rival-client";
const MC_TOKEN_KEY: &str = "mc_token";
const REFRESH_TOKEN_KEY: &str = "refresh_token";

pub fn save_token(mc_token: &str, refresh_token: &str) -> Result<(), keyring::Error> {
    Entry::new(SERVICE, MC_TOKEN_KEY)?.set_password(mc_token)?;
    Entry::new(SERVICE, REFRESH_TOKEN_KEY)?.set_password(refresh_token)?;
    Ok(())
}

pub fn load_token() -> Result<Option<String>, keyring::Error> {
    match Entry::new(SERVICE, MC_TOKEN_KEY)?.get_password() {
        Ok(token) => Ok(Some(token)),
        Err(keyring::Error::NoEntry) => Ok(None),
        Err(e) => Err(e),
    }
}

pub fn clear_tokens() -> Result<(), keyring::Error> {
    Entry::new(SERVICE, MC_TOKEN_KEY)?.delete_password()?;
    Entry::new(SERVICE, REFRESH_TOKEN_KEY)?.delete_password()?;
    Ok(())
}
```

---

## Azure App Registration Required
Before the OAuth flow works, you must register an Azure app:
1. Go to [portal.azure.com](https://portal.azure.com) → Azure Active Directory → App registrations
2. New registration → **Accounts in any organizational directory and personal Microsoft accounts**
3. Redirect URI → **Public client/native** → `http://localhost:7878/callback`
4. Copy the **Application (client) ID** → paste into `CLIENT_ID` constant
5. No client secret needed (PKCE is used instead)

---

## IPC Commands to Expose
```rust
#[tauri::command]
pub async fn start_auth(app: tauri::AppHandle) -> Result<(), String> {
    // 1. Generate PKCE
    // 2. Open browser to auth URL
    // 3. Wait for code on loopback
    // 4. Exchange all tokens
    // 5. Save to keyring
    // 6. Emit auth_complete event
    todo!()
}

#[tauri::command]
pub async fn check_auth() -> Result<bool, String> {
    Ok(token::load_token().map_err(|e| e.to_string())?.is_some())
}

#[tauri::command]
pub async fn logout() -> Result<(), String> {
    token::clear_tokens().map_err(|e| e.to_string())
}
```
