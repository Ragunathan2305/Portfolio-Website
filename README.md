# Portfolio Website — AWS CI/CD Pipeline

A personal portfolio website hosted on AWS, deployed and updated automatically through a CI/CD pipeline built with GitHub Actions. Every push to the `main` branch syncs the latest files directly to S3 and refreshes the CloudFront cache — no manual deployment steps required.

**Live Site:** [https://d2az3sc5f38bmc.cloudfront.net/](https://d2az3sc5f38bmc.cloudfront.net/)

---

## Architecture

```
Developer (local machine)
      │
      │ git push (main branch)
      ▼
GitHub Repository
      │
      │ triggers
      ▼
GitHub Actions Workflow
      │
      ├── Checkout repository code
      ├── Configure AWS credentials (IAM user)
      ├── Sync files to S3 bucket
      └── Invalidate CloudFront cache
      ▼
Amazon S3 (Static Website Hosting)
      │
      ▼
Amazon CloudFront (HTTPS CDN)
      │
      ▼
End User
```

---

## Tech Stack

- **Hosting:** Amazon S3 (Static Website Hosting)
- **CDN / HTTPS:** Amazon CloudFront
- **CI/CD:** GitHub Actions
- **IAM:** Dedicated IAM user with scoped permissions (S3 + CloudFront invalidation)
- **CLI Tooling:** AWS CLI (via `aws-actions/configure-aws-credentials`)

---

## CI/CD Pipeline

The workflow is defined in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml).

**Trigger:** Runs on every push to `main`, except changes limited to `.git/`, `.github/`, or `README.md` — avoiding unnecessary deployments for non-website changes.

**Steps:**

1. **Checkout** — Pulls the latest repository code onto the GitHub-hosted runner.
2. **Configure AWS Credentials** — Authenticates the runner using IAM access keys stored securely in GitHub Secrets.
3. **Sync to S3** — Uploads changed files to the S3 bucket using `aws s3 sync`, excluding repository-only files (`.git/`, `.github/`, `README.md`) so the bucket only ever contains live website assets.
4. **Invalidate CloudFront Cache** — Clears the CDN cache so visitors immediately see the latest deployed version.

---

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── deploy.yml      # CI/CD pipeline definition
├── Asserts/
│   ├── background.jpg
│   ├── photo.ico
│   └── photo.jpg
├── Error.html
├── Index.html
└── README.md
```

---

## Key Design Decisions

- **Reused existing IAM user (least-friction approach):** Rather than creating a new IAM user per project, an existing IAM user (already scoped with S3 and CloudFront permissions for personal AWS projects) was reused — keeping credential management simple for a free-tier account while remaining mindful of the least-privilege principle.
- **Secrets vs. Variables:** Sensitive values (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) are stored as encrypted GitHub Secrets. Non-sensitive configuration (`AWS_REGION`, `CLOUDFRONT_DISTRIBUTION_ID`) is stored as GitHub Variables, reflecting how these values would typically be managed in a production environment.
- **Exclude patterns for a clean bucket:** The `aws s3 sync` command explicitly excludes `.git/`, `.github/`, and `README.md` to ensure only production-ready website assets are ever served from S3 — avoiding exposure of repository metadata.
- **Path-based trigger filtering:** The workflow trigger uses negation patterns to skip runs that only touch documentation or pipeline configuration, reducing wasted CI minutes and keeping the deployment history meaningful.
- **Cache invalidation on every deploy:** A CloudFront invalidation (`/*`) is run after every successful sync, ensuring users always receive the latest version without waiting for cache expiry.

---

## Author

**Ragunathan K**
AWS Certified Solutions Architect – Associate | AWS Certified Cloud Practitioner
[GitHub](https://github.com/Ragunathan2305)