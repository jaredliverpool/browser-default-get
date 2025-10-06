# HTB - Basic Auth: Finding the Flag via GET / search.php

**Tagline:** Demonstration of using Basic HTTP auth and `curl`/DevTools/Fetch to retrieve a JSON-backed search endpoint.

---

## Outline

1. Project overview (what I’m doing and why)
2. Scope & ethics (this was done on Hack The Box with provided credentials)
3. Quick prerequisites (platforms, tools, creds)
4. Step-by-step walkthrough (what I did, commands I ran, what I observed)
5. Results (where the flag was found)
6. Notes, lessons learned, and next steps
7. Screenshots & attachments (placeholders)

---

## 1) Project overview

I was given a target on Hack The Box along with a username and password. The goal was to use the supplied credentials to authenticate to the web app and discover the flag. This write-up shows how I authenticated with Basic HTTP auth, observed the search traffic in the browser, reproduced requests with `curl`, and used the `fetch()` snippet copied from DevTools to confirm the JSON endpoint returning the results.

> **Important:** This work was performed on the Hack The Box platform using the credentials provided by the platform. Do not attempt these steps on systems you don't have permission to test.

---

## 2) Scope & Ethics

* **Scope:** HTB box, credentials provided by the lab. I only interacted with that environment.
* **Ethics:** All testing was authorized by the platform. This repo documents my methodology and findings for learning / portfolio purposes.

---

## 3) Prerequisites

* Access to the Hack The Box target and the username/password the lab gave me.
* A Linux environment (I used a terminal) with `curl` installed.
* A modern browser (Chrome or Firefox) with Developer Tools (Network tab, Console).

**Credentials:** (use the ones HTB provided; shown below as an example)

```
username: admin
password: admin
```

**Placeholder for target:** `94.237.123.119:59527`

---

## 4) Step-by-step walkthrough

> I’ll write this exactly like how I did it and how I’d explain it to someone on my team.

### a. Visit the site and log in

1. I opened the HTB URL in the browser and entered the username and password supplied by the lab. The app used **Basic HTTP Authentication**. When I first checked the page, the response headers included:

```
WWW-Authenticate: Basic realm="Access denied"
```

<img width="817" height="404" alt="Screenshot 2025-10-06 at 2 30 22 AM" src="https://github.com/user-attachments/assets/f3e80765-51f2-4d2d-ac54-b05dfa1ac3bb" />


That confirmed Basic auth was in use.

---

### b. Verify access with `curl`

2. From Linux I verified access with `curl` by supplying the credentials with `-u`:

```bash
curl -u admin:admin http://<SERVER_IP>:<PORT>/
```

<img width="965" height="503" alt="Screenshot 2025-10-06 at 2 32 00 AM" src="https://github.com/user-attachments/assets/a7883ce8-cede-4c93-af4d-417dba70e287" />


3. I also tried embedding credentials in the URL (alternate form):

```bash
curl http://admin:admin@<SERVER_IP>:<PORT>/
```

<img width="811" height="571" alt="Screenshot 2025-10-06 at 2 35 36 AM" src="https://github.com/user-attachments/assets/d4244e59-da59-4fc7-85d2-0230c777b2ba" />


Both gave me the page content.

---

### c. Inspecting the request details

4. To see more details about the HTTP exchange, I used verbose mode:

```bash
curl -v http://admin:admin@<SERVER_IP>:<PORT>/
```

<img width="804" height="481" alt="Screenshot 2025-10-06 at 2 38 25 AM" src="https://github.com/user-attachments/assets/23877050-94a9-4148-bcaf-1d7a20595d9f" />


That showed the `Authorization` header being sent:

```
Authorization: Basic YWRtaW46YWRtaW4=
```

(That `YWRtaW46YWRtaW4=` is simply the Base64 encoding of `admin:admin`.)

5. With that value in hand I confirmed I could authenticate by manually setting the header (without `-u`):

```bash
curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://<SERVER_IP>:<PORT>/
```

<img width="805" height="550" alt="Screenshot 2025-10-06 at 2 41 37 AM" src="https://github.com/user-attachments/assets/8557e23c-743e-4358-831b-0dc8f60918dd" />


This returned the same page content as supplying credentials.

---

### d. Finding the search endpoint with Browser DevTools

6. After authenticating in the browser, I navigated to the page with the search bar. To watch the network requests I opened **Developer Tools → Network** and cleared previous entries (trash icon) so I would only see new requests.

<img width="1181" height="246" alt="Screenshot 2025-10-06 at 2 43 39 AM" src="https://github.com/user-attachments/assets/56e29fb7-c56b-4f3c-971e-d2d79a14a890" />


7. I typed `ne` into the search box and pressed Enter. A new request appeared called `search.php` with the query parameter `search=ne` — indicating the search widget performs a GET to `search.php`.

<img width="1180" height="251" alt="Screenshot 2025-10-06 at 2 45 02 AM" src="https://github.com/user-attachments/assets/3abcbd64-c7b6-4c5b-8435-1469e0e551fc" />


8. I right-clicked that network row and selected **Copy → Copy as URL**. That gave me a full URL I could paste into the terminal.

---

### e. Reproducing the search request with `curl`

9. I pasted the copied URL into the terminal and re-ran it with the same Authorization header, for example:

```bash
curl 'http://<SERVER_IP>:<PORT>/search.php?search=ne' -H 'Authorization: Basic YWRtaW46YWRtaW4='
```
<img width="814" height="126" alt="Screenshot 2025-10-06 at 2 48 07 AM" src="https://github.com/user-attachments/assets/ba2abec8-29f4-4ff9-81a4-689a4d1860c2" />


The response I received matched what the browser showed. It looked like the search endpoint returned JSON results (not just HTML), which made it easier to parse programmatically.

---

### f. Replaying the request using the Fetch snippet

10. From DevTools I also used **Copy → Copy as Fetch** and pasted the snippet into the Console. I executed it there to reproduce the same request from within the browser environment.

<img width="1224" height="248" alt="Screenshot 2025-10-06 at 2 57 35 AM" src="https://github.com/user-attachments/assets/a6e80fb1-f9fe-413b-a645-fce0cd352e94" />


---

## 5) Results

* Using the above steps I was able to access the search endpoint and retrieve the JSON results. The flag appeared in the returned data.
* <img width="816" height="100" alt="Screenshot 2025-10-06 at 3 00 21 AM" src="https://github.com/user-attachments/assets/24be1cfa-4ec8-4fdc-877a-51f97c7c1319" />


---

## 6) Notes & lessons learned

* **Basic Auth is not encrypted**: Basic only Base64-encodes credentials; do not rely on it over plain HTTP. Prefer HTTPS or stronger auth for real apps.
* **DevTools + Copy features are handy**: `Copy as URL` and `Copy as Fetch` make it trivial to reproduce and debug browser requests.
* **`curl -v` shows headers**: Verbose mode is useful to inspect what the browser is sending (Authorization header, user-agent, etc.).

---

