# HTB - Basic Auth: Finding the Flag via GET / search.php

**Tagline:** Demonstration of using Basic HTTP auth and `curl`/DevTools/Fetch to retrieve a JSON-backed search endpoint (screenshots included — placeholders below).

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

**Placeholder for target:** `http://<SERVER_IP>:<PORT>/`

---

## 4) Step-by-step walkthrough

> I’ll write this exactly like how I did it and how I’d explain it to someone on my team.

### a. Visit the site and log in

1. I opened the HTB URL in the browser and entered the username and password supplied by the lab. The app used **Basic HTTP Authentication**. When I first checked the page, the response headers included:

```
WWW-Authenticate: Basic realm="Access denied"
```

That confirmed Basic auth was in use.

---

### b. Verify access with `curl`

2. From Linux I verified access with `curl` by supplying the credentials with `-u`:

```bash
curl -u admin:admin http://<SERVER_IP>:<PORT>/
```

3. I also tried embedding credentials in the URL (alternate form):

```bash
curl http://admin:admin@<SERVER_IP>:<PORT>/
```

Both gave me the page content.

---

### c. Inspecting the request details

4. To see more details about the HTTP exchange, I used verbose mode:

```bash
curl -v http://admin:admin@<SERVER_IP>:<PORT>/
```

That showed the `Authorization` header being sent:

```
Authorization: Basic YWRtaW46YWRtaW4=
```

(That `YWRtaW46YWRtaW4=` is simply the Base64 encoding of `admin:admin`.)

5. With that value in hand I confirmed I could authenticate by manually setting the header (without `-u`):

```bash
curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://<SERVER_IP>:<PORT>/
```

This returned the same page content as supplying credentials.

---

### d. Finding the search endpoint with Browser DevTools

6. After authenticating in the browser, I navigated to the page with the search bar. To watch the network requests I opened **Developer Tools → Network** and cleared previous entries (trash icon) so I would only see new requests.

7. I typed `ne` into the search box and pressed Enter. A new request appeared called `search.php` with the query parameter `search=ne` — indicating the search widget performs a GET to `search.php`.

8. I right-clicked that network row and selected **Copy → Copy as URL**. That gave me a full URL I could paste into the terminal.

---

### e. Reproducing the search request with `curl`

9. I pasted the copied URL into the terminal and re-ran it with the same Authorization header, for example:

```bash
curl 'http://<SERVER_IP>:<PORT>/search.php?search=ne' -H 'Authorization: Basic YWRtaW46YWRtaW4='
```

The response I received matched what the browser showed. It looked like the search endpoint returned JSON results (not just HTML), which made it easier to parse programmatically.

---

### f. Replaying the request using the Fetch snippet

10. From DevTools I also used **Copy → Copy as Fetch** and pasted the snippet into the Console. I executed it there to reproduce the same request from within the browser environment.

*(Keep screenshots here showing: Network entry for `search.php`, the Copy as URL action, the copied Fetch snippet in the console, and the JSON response.)*

---

## 5) Results

* Using the above steps I was able to access the search endpoint and retrieve the JSON results. The flag appeared in the returned data (redacted here for the repo — include a screenshot of the flag in the `screenshots/` folder when you commit).

---

## 6) Notes & lessons learned

* **Basic Auth is not encrypted**: Basic only Base64-encodes credentials; do not rely on it over plain HTTP. Prefer HTTPS or stronger auth for real apps.
* **DevTools + Copy features are handy**: `Copy as URL` and `Copy as Fetch` make it trivial to reproduce and debug browser requests.
* **`curl -v` shows headers**: Verbose mode is useful to inspect what the browser is sending (Authorization header, user-agent, etc.).

---

## 7) Screenshots (placeholders)

* `screenshots/01-login.png` — login prompt and `WWW-Authenticate` header visible in devtools
* `screenshots/02-curl-v.png` — `curl -v` output showing `Authorization` header
* `screenshots/03-network-search.png` — Network tab showing `search.php?search=ne`
* `screenshots/04-copy-fetch-console.png` — pasted Fetch snippet in Console and response
* `screenshots/05-flag.png` — redacted/uncropped screenshot showing the flag (or blurred if you prefer)

---

## Suggested repo name and README tagline

* **Repo name (kebab-case):** `htb-basic-auth-search`
* **Short README line:** "Documenting how I used Basic HTTP auth, `curl`, and browser DevTools to reproduce a JSON search endpoint and recover the flag on an HTB machine."

---

If you want, I can also:

* Create a ready-to-go `README.md` file (this is already in the canvas).
* Add example screenshots to the repo structure and a `.gitignore` for any sensitive images.
* Generate a short `report.md` version suitable for sharing with teammates.

---

*End of document.*
