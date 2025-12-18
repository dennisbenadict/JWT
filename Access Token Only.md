## 1. Access token only

### What it means:
System has:  
✅ Access Token (JWT, 24h expiry)  
❌ NO Refresh Token  
❌ NO Refresh Endpoint  

### Flow:
Login → Get Token → Use Token → Token Expires → 401 → No Refresh → Logout  

---

## 2. Why access token only logs out

### Timeline:
```
Time: 0h          24h
│            │
├─ Login ────┤
│            │
│ Token:     │ Token:
│ ABC123     │ ABC123
│ (valid)    │ ❌ EXPIRED
│            │
│ API Call
│ → 401 Error
│
│ ❌ No Refresh
│ ❌ No New Token
│
│ → Logout
```

### Why:
Token Expires  
→ 401 Error  
→ No Refresh Endpoint  
→ Cannot Get New Token  
→ Must Login  

---

## 3. Complete flow: access token only

### Login:
User → Login Request → Backend → Access Token → Frontend Stores → Authenticated ✅  

### API call (valid):
Request → Token → Backend Validates ✅ → Success  

### API call (expired):
Request → Expired Token → Backend Rejects ❌ → 401 → No Refresh → Logout  

---

## 4. Why access token can't authorize like refresh token

### Access token (self-contained):
```
┌─────────────────────────┐
│ JWT Token               │
├─────────────────────────┤
│ userId: 1               │
│ email: "user@ex.com"    │
│ exp: 1234567890 ← EXPIRED!
└─────────────────────────┘
```

Backend checks:  
exp < now  
❌ EXPIRED → Cannot use  

### Refresh token (database-linked):
```
┌─────────────────────────┐
│ Random String           │
│ "a1b2c3d4e5f6..."       │
│                         │
│ ❌ No expiry in token   │
└─────────────────────────┘
```

Backend looks up in database  
Database: ExpiresAt = Day 8  
Backend checks: Day 8 > now  
✅ VALID → Can use  

---

## Comparison:

Access Token:  
- Expiry in token ❌  
- Self-validating  
- Can't use expired  

Refresh Token:  
- Expiry in database ✅  
- Database-validating  
- Can use (checks DB)  

---

## 5. How refresh token gets authorized

### Authorization flow:
Step 1: Frontend sends refresh token  
↓  
Step 2: Backend hashes token  
↓  
Step 3: Backend queries database  

```
┌────┬─────────┬────────────┬─────────────┐
│ Id │ UserId  │ ExpiresAt  │ RevokedAt   │
├────┼─────────┼────────────┼─────────────┤
│ 1  │ 1       │ Day 8      │ NULL        │
└────┴─────────┴────────────┴─────────────┘
```

↓  
Step 4: Backend validates  
- Exists? ✅  
- Not revoked? ✅  
- Not expired? ✅  

↓  
Step 5: Backend gets UserId (1)  
↓  
Step 6: Backend queries user  

```
┌────┬──────────────┬──────────┐
│ Id │ Email        │ IsAdmin  │
├────┼──────────────┼──────────┤
│ 1  │ user@ex.com  │ false    │
└────┴──────────────┴──────────┘
```

↓  
Step 7: Backend generates new tokens  
↓  
Step 8: Backend sets cookies  
↓  
✅ Authorized  

---

## Visual flow:
Refresh Token  
→ Hash  
→ Database Lookup  
→ Validate  
→ Get UserId  
→ Get User  
→ Generate Tokens  
→ Success  

---

## Summary

Access token only:  
Login → Token (24h) → Expires → 401 → No Refresh → Logout  

Access + refresh tokens:  
Login → Access (15min) + Refresh (7 days)  
↓  
Access Expires → 401 → Refresh Called → Database Lookup → New Tokens → Success  

---

## Key differences:

Access Token:  
- Expiry in token ❌  
- Can't use expired  
- Self-contained  

Refresh Token:  
- Expiry in database ✅  
- Can use (DB check)  
- Database-linked  

---

## Authorization:

Access Token:  
JWT Validation  
(Signature + Expiry)  

Refresh Token:  
Database Lookup  
(Exists + Active + UserId)  

---

## Final answer

Access token only:  
One token, no refresh mechanism.  

Why logout:  
Token expires → 401 → No refresh → Must login.  

Flow:  
Login → Token → Use → Expires → 401 → Logout.  

Why access token can't authorize refresh:  
Expiry is in the token; expired tokens are rejected.  

How refresh token gets authorized:  
Database lookup validates token, checks expiry/revocation, gets UserId, generates new tokens.  

Difference:  
Access tokens are self-contained (expiry in token),  
refresh tokens are database-validated (expiry in database).
