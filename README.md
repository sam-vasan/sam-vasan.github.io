# Private GitHub Pages Clipboard

Static GitHub Pages frontend; clipboard data is stored as text files in a separate **private GitHub repository** using GitHub's Contents REST API.

## Privacy model

- The website itself can be public, but the **data repository must be private**.
- Share links contain only a random clip ID. They do not grant access. Every reader needs a GitHub token with access to the private repository.
- The browser never saves the token to localStorage, cookies, or the URL; it keeps it in JavaScript memory until the page is refreshed or the token is cleared. This is safer than persisting it, but not equivalent to server-side secret storage.
- Use only on a trusted device. Any script executing in the page could potentially access a token while it is in memory. For stronger authentication, use a backend or a GitHub App OAuth flow; GitHub Pages alone cannot securely hold a client secret.
- Git history retains prior versions of clips. Deleting a file does not erase old versions from repository history.

## Deploy

1. Create a private repository for data, e.g. `private-clipboard`. Use a branch named `main`.
2. Create a separate repository for the website, e.g. `YOUR_USERNAME.github.io`. GitHub Pages generally requires a public repository on GitHub Free; check your plan and organization settings if you want a private Pages source repository. The site may be public while the data stays private.
3. Upload `index.html` from this package to the website repository's root and commit it.
4. Open website repository **Settings → Pages**. Publish from branch `main`, folder `/ (root)`. Open the generated HTTPS URL.
5. Create a fine-grained personal access token at GitHub **Settings → Developer settings → Personal access tokens → Fine-grained tokens**:
   - Resource owner: your account.
   - Repository access: **Only select repositories** → `private-clipboard`.
   - Repository permissions: **Contents: Read and write**.
   - Choose an expiry date; do not grant unrelated permissions.
6. On the website, enter owner, private data repository, branch `main`, and the token. Click **Authenticate & load link**. Paste text and let autosave run or click **Save now**.
7. Click **Copy share link** after saving. The recipient must have access to the private repository and authenticate with their own token. A clip ID is not an access credential.

## Features

- Autosave after 1.5 seconds of inactivity; manual Save now and Ctrl/Cmd+S.
- Copy text, download UTF-8 `.txt`, create new clips, share links.
- Files are stored under `clips/<24-hex-id>.txt`.
- Raw authenticated Contents API reads support files above 1 MiB; default app limit is 5 MiB per clip.
- No build step or third-party dependencies.

## Realistic limits

- The UI imposes a conservative **5 MiB (5 × 1024 × 1024 bytes) per clip** limit. This can be raised by changing `MAX_BYTES` in `index.html`, but browser memory, Base64 request expansion, and API latency grow with it.
- GitHub's Contents API supports retrieval of files up to 100 MB; files larger than 1 MB must be retrieved using a raw/object media type, and files over 100 MB are unsupported by that endpoint. See official documentation.
- GitHub is not unlimited object storage. Repository size, Git history, rate limits, and GitHub's acceptable-use policies apply. Each save creates a commit; repeatedly rewriting large logs grows repository history. For large, frequently updated logs, use object storage or a database instead.
- This app does not list/search all clips, enforce expiry, revoke individual links, or encrypt content. Access is controlled by GitHub repository permissions and tokens.
- GitHub API rate limits apply. Intended for personal/light use, not a public multi-user paste service.

## Troubleshooting

- `401/403`: token expired or missing `Contents: Read and write`; confirm selected repository access.
- `404`: private repository permissions may be insufficient, or owner/repo/branch/clip ID may be wrong.
- Share link loads nothing: enter owner, repository, branch and a token with access, then click **Authenticate & load link**.
- `409`: a concurrent edit conflicted; retry Save now.
- Network/CORS failure: use the deployed HTTPS site and check corporate proxy/firewall policies.

## Official documentation

- [GitHub REST API: Repository contents](https://docs.github.com/en/rest/repos/contents)
- [GitHub Pages](https://docs.github.com/en/pages)
- [Fine-grained personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

