# Mvumi Website

Minimal static website ready for GitHub Pages with a custom domain.

## Files

- `index.html`: Page structure and content
- `styles.css`: Visual style
- `CNAME`: Custom domain (`mvumi.website`)

## Publish with your GitHub account

1. Create a new GitHub repository (for example, `mvumi-website`).
2. Upload these files to the repository root.
3. In GitHub: **Settings -> Pages**
4. Under **Build and deployment**, set:
   - **Source**: Deploy from a branch
   - **Branch**: `main` and folder `/ (root)`
5. Ensure your domain DNS points to GitHub Pages:
   - For apex domain (`mvumi.website`), use A records to GitHub Pages IPs.
   - For `www.mvumi.website`, use a CNAME record to `<your-github-username>.github.io`.
6. In Pages settings, set **Custom domain** to `mvumi.website` and enable HTTPS.

After DNS propagation, your site will be live on `https://mvumi.website`.
