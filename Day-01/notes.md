# Day-01 – BUG-NAME

Explanation:
IDOR happens when an application exposes a reference to an internal object (like a file, user ID, order ID, etc.) and doesn’t properly check if the user is authorized to access it. 
in very simple terms: You can access or modify someone else's data just by changing a value in the request — usually a numeric ID or some predictable identifier.

For example there is a banking app which sends this request when you visit your account info
GET /accounts/56789
Authorization: Bearer YOUR_TOKEN

now you see your id is 56789, if you can change this id and see someone else account, this is IDOR , for example you change 56789 to 56790, and you see someone else account whose id is 56790. This happens is the app does not validates the ownership

Common Places Where IDOR Happens

    URLs: GET /user/1234

    Form data: POST /updateProfile with user_id=1234

    Query params: /invoice?id=9999

    JSON body: { "user_id": 4321 }

    Headers: X-User-ID: 5566
    
This happens because developers rely on client input without validating it and thinking "authentication" is enough — but forget authorization.
Authentication = "who you are"
Authorization = "what you’re allowed to do"
IDOR is a failure of authorization.



I started practicing idor in portswigger idor lab, it felt easy then practiced on owasp juice shop that was also easy because there i have to simpt change a value
in old labs there are only few simple step to find idor
1:Intercept requests using Burp Suite.
2:Find IDs or references in parameters, URLs, headers, or body.
3:Change the value to another valid or guessed ID.

After practicing on some labs i picked up a target from open bug bounty, i was just thinking where to start when i found a store on that webapp, when i added a product in the cart i observed the url there was an id , the url was like example.com/store/cart.php?id=11285&action=add, i opened burp changed the id and i saw another product was added in cart, it was kind of normal, until i saw i was able to add or delete products by changing id parameter and changing action,the site had aslo week or you can say broken authentication so i came up with another idea.

That application was allowing direct modification of the PHPSESSID cookie to impersonate another user’s session without authentication. By capturing a valid PHPSESSID and replaying it from a different browser or IP address, an attacker can fully take over another user’s account, including adding or removing items from their cart.
so what i did?
Login / initiate a session in Browser A (Burp’s built-in browser) and perform an action (e.g., add a book to the cart).
Capture the request in Burp Suite and note the PHPSESSID value.
Open Browser B (e.g., Firefox) and start a separate session (different cookies).
Perform an action in Browser B and intercept the request in Burp Suite.
Replace the PHPSESSID in Browser B’s request with the value captured from Browser A.
Forward the request.
Observe that Browser B now operates in Browser A’s session, allowing actions such as: Viewing or modifying the cart, deleting items and potentially accessing sensitive data
I also changed my ip several times  (via VPN) and repeat steps 4–6. The attack still works, confirming the vulnerability is not restricted to a specific IP.

In new and moderen apps mostly UUID-based ids are used but they can also be vulnerable , it looks secure but isn’t if UUIDs are predictable or enumerable and Apps often still fail to enforce proper access control


In the end i will close this topic by saying IDOR = “Change the ID, steal the data.”

{If the server doesn’t check permissions, you win.If you're a dev and don't validate ownership — you lose.} quote by "chatgpt"
