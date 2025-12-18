## 🚀 Access Token and Refresh Token — How It Actually Works

### ⏳ Token Lifetimes
- Access Token → 15 minutes  
- Refresh Token → 7 days  

### 🧠 Core Rule
Refresh is reactive, not automatic.

There is:
- No timer  
- No background job  
- No automatic refresh  

Tokens refresh only when the user makes an API request.

### 🔁 When Is Refresh Called?
The refresh endpoint is called ONLY when all conditions below are met:

1. User makes an API request  
2. Access token is expired  
3. Backend returns 401 Unauthorized  
4. Frontend interceptor catches the 401  
5. Interceptor calls the refresh endpoint  

When refresh is successful:
- A **new access token** is issued  
- A **new refresh token** is issued  
- The old refresh token becomes invalid  

If even one condition is missing, no refresh happens.

### 💤 Inactive User Flow
Day 1 – 00:00  
User logs in  
Access token expires at 00:15  
Refresh token expires at Day 7  

00:15  
Access token expires  
No action (user inactive)

Day 1 to Day 6  
User inactive  
No API calls  
No refresh  

Day 7  
Refresh token expires  
Still no action

User opens app after 7 days

API call  
↓  
401 Unauthorized (access expired)  
↓  
Interceptor calls /refresh  
↓  
Refresh token expired  
↓  
Redirect to login  

### ⚡ Active User Flow
00:00  
User logs in  

00:10  
API call  
Access token valid  

00:20  
API call  
Access token expired  
401 returned  
Refresh called  
New access token issued  
New refresh token issued  

00:35  
API call  
Access token valid  

This cycle continues as long as the refresh token is valid.

### 📌 Important Facts
- Access token expiry does not mean logout  
- Refresh token expiry means logout  
- Tokens expire silently  
- No API call means no refresh  
- Refresh always issues **both new tokens**

### 🧩 Mental Model
Access Token → Short life, sent with every request  
Refresh Token → Long life, used only to get new access and refresh tokens  

### ✨ One-Line Summary
User stays logged in until the refresh token expires (7 days) and an API request is made
