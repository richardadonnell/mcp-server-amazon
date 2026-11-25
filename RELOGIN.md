# Re-logging in the Amazon connector

Symptom: the connector says "You need to be logged in" (usually on order history).
Amazon kills the session server-side even though the cookies look valid.
A single ERR_CONNECTION_FAILED is Amazon bot detection, not a logout. Retry once first.

## Steps

1. In PowerShell:

```powershell
cd C:\Users\richa\projects\mcp-server-amazon
$env:AMAZON_EMAIL = "your-amazon-email"
$env:AMAZON_PASSWORD = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR((Read-Host "Amazon password" -AsSecureString)))
node login_and_save_cookies.cjs
```

2. Finish any CAPTCHA / OTP / device approval in the browser window it opens.
   Don't close the window; the script saves amazonCookies.json and closes it itself (15-minute limit).
3. Fully quit Claude Desktop from the tray icon (not just close the window) and reopen it.
   The server only reads amazonCookies.json at startup.
4. Ask Claude to check order history to confirm.
