```markdown
# SOP: Cloudflare Family DNS (1.1.1.3 / 1.0.0.3)  
**Blocks: Porn, Malware, Phishing**  
**Scope**: System-wide + Brave + Edge  
**OS**: Windows 10/11  
**Date**: November 14, 2025  

---

## 1. CLI IMPLEMENTATION (System-Wide DNS)

> **Run all commands in CMD as Administrator**  
> (Right-click Start → "Command Prompt (Admin)" or "Windows Terminal (Admin)")

### 1.1 Apply DNS to All Interfaces
```cmd
for /f "skip=3 tokens=3*" %A in ('netsh interface show interface') do @if not "%A"=="Disconnected" if not "%A"=="Disabled" if not "%B"=="" netsh interface ipv4 set dnsservers "%B" static 1.1.1.3 primary >nul 2>&1 & netsh interface ipv4 add dnsservers "%B" 1.0.0.3 >nul 2>&1 & netsh interface ipv6 set dnsservers "%B" static 2606:4700:4700::1113 primary >nul 2>&1 & netsh interface ipv6 add dnsservers "%B" 2606:4700:4700::1003 >nul 2>&1
```

### 1.2 Flush DNS Cache
```cmd
ipconfig /flushdns
```

### 1.3 Verify System DNS
```cmd
ipconfig /all | findstr "DNS"
```
**Expected**:
```
   DNS Servers . . . . . . . . . . . : 1.1.1.3
                                       1.0.0.3
                                       2606:4700:4700::1113
                                       2606:4700:4700::1003
```

### 1.4 Test Blocking
```cmd
nslookup pornhub.com
```
**Expected (BLOCKED)**:
```
Server:  UnKnown
Address:  2606:4700:4700::1113

*** UnKnown can't find pornhub.com: No response from server

```

---

## 2. GUI IMPLEMENTATION: BRAVE BROWSER

### 2.1 Disable Brave Secure DNS
1. Open **Brave**
2. Go to: `brave://settings/security?search=dns`
3. Find **"Use secure DNS"**
4. **Select DNS Provider**  
   → CleanBrowser (Family filter)**

### 2.2 Clear Brave DNS Cache
1. Go to: `brave://net-internals/#dns`
2. Click **"Clear host cache"**

### 2.3 Test in Brave (InPrivate)
1. Press `Ctrl+Shift+N`
2. Go to: `pornhub.com`
3. **Expected**: "This site can’t be reached" / `DNS_PROBE_FINISHED_NXDOMAIN`

---

## 3. GUI IMPLEMENTATION: EDGE BROWSER

### 3.1 Disable Edge Secure DNS
1. Open **Edge**
2. Go to: `edge://settings/privacy`
3. Scroll to **Security**
4. Find **"Use secure DNS"**
5. **Turn OFF**  
   → Edge now uses **system DNS (1.1.1.3)**

### 3.2 Clear Edge DNS Cache
1. Go to: `edge://net-internals/#dns`
2. Click **"Clear host cache"**

### 3.3 Test in Edge (InPrivate)
1. Press `Ctrl+Shift+N`
2. Go to: `pornhub.com`
3. **Expected**: "This site can’t be reached" / `DNS_PROBE_FINISHED_NXDOMAIN`

---

## 4. FINAL VERIFICATION (ALL BROWSERS)

| Test | Command / Action | Expected |
|------|------------------|----------|
| System DNS | `ipconfig /all \| findstr DNS` | Shows `1.1.1.3` |
| Porn Block | `nslookup pornhub.com` | No response |
| Safe Site | `nslookup google.com` | Returns IP |
| Brave | InPrivate → `pornhub.com` | Blocked |
| Edge | InPrivate → `pornhub.com` | Blocked |

---

## 5. AUTO-RUN ON BOOT (Optional)

1. Press `Win + R` → `shell:startup` → Enter
2. Create `lock_dns.cmd` with:
```cmd
@echo off
for /f "skip=3 tokens=3*" %A in ('netsh interface show interface') do @if not "%A"=="Disconnected" if not "%A"=="Disabled" if not "%B"=="" netsh interface ipv4 set dnsservers "%B" static 1.1.1.3 primary >nul 2>&1 & netsh interface ipv4 add dnsservers "%B" 1.0.0.3 >nul 2>&1
ipconfig /flushdns >nul
```

---

## 6. REVERT TO ISP DNS (Undo)

```cmd
for /f "skip=3 tokens=3*" %A in ('netsh interface show interface') do @if not "%B"=="" netsh interface ipv4 set dnsservers "%B" source=dhcp >nul 2>&1 & netsh interface ipv6 set dnsservers "%B" source=dhcp >nul 2>&1
ipconfig /flushdns
```

---

**STATUS: ACTIVE**  
**All browsers & apps now use Cloudflare Family DNS**  
**Porn & malware blocked at the root**
```
