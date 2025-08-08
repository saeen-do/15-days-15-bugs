# Day-01 – IDOR (Insecure Direct Object Reference)

## Theory
**Definition**:  
IDOR occurs when an application exposes a reference to an internal object (e.g., user ID, file ID, order ID) without properly verifying that the requesting user is authorized to access it.  
In simple terms:  
Change an ID or reference in the request → Access or modify someone else’s data.

**Example Scenario:**  
A banking app sends this request to fetch your account info:
GET /accounts/56789
Authorization: Bearer YOUR_TOKEN

If you can change `56789` to `56790` and see another person’s account details, the app is vulnerable. This happens when the server does not validate resource ownership.

**Common Places Where IDOR Occurs:**
**URLs:** `GET /user/1234`
**Form Data:** `POST /updateProfile` with `user_id=1234`
**Query Params:** `/invoice?id=9999`
**JSON Body:** `{ "user_id": 4321 }`
**Headers:** `X-User-ID: 5566`

**Root Cause:**  
Developers rely on client-supplied identifiers and think authentication is enough, forgetting proper **authorization** checks.

**Authentication** = Who you are
**Authorization** = What you are allowed to do  
IDOR is a failure of authorization.

##Lab Practice

**Labs Completed:**
1. **PortSwigger IDOR Lab** – Basic parameter manipulation  
2. **PortSwigger IDOR Lab #2** – Horizontal privilege escalation  
3. **OWASP Juice Shop** – Modified a value in the cart request

**Steps Followed:**
1. Intercept requests using Burp Suite.
2. Identify IDs or references in parameters, URLs, headers, or request bodies.
3. Modify the value to another valid or guessed ID.
4. Observe the response for unauthorized access.


## Real Target Testing

**Platform:** Open Bug Bounty  
**Target Type:** Web application  

**Discovery:**  
While exploring a store feature, I added a product to the cart. The request looked like:


GET /store/cart.php?id=11285&action=add HTTP/1.1
Host: target.com
Cookie: PHPSESSID=abcdef123456


**Manipulation:**
Changing `id` value → Added a different product to the cart  
Changing `action` → Added or removed products at will  
No server-side validation of whether the authenticated user owned that cart entry  

**Result:**  
An attacker could:
Add or remove any product from any user’s cart
Potentially manipulate other order-related actions



##Impact
Unauthorized modification of user carts
Violation of data integrity
Potential chaining into higher-impact attacks (e.g., order manipulation, price tampering)


##Recommendation
Implement **server-side authorization checks** to ensure the requesting user owns the resource.
Avoid using predictable IDs; use securely generated identifiers.
Enforce input validation and reject unauthorized modifications.


##Closing Note
IDOR in one line:  
“If the server doesn’t check permissions, you win. If you’re a dev and don’t validate ownership you lose.” (quote by chat-gpt)



