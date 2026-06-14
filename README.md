# 🌐 Portfolio Website — AWS CI/CD Pipeline

A personal portfolio website hosted on AWS, deployed and updated automatically through a CI/CD pipeline built with GitHub Actions. Every push to `main` syncs the latest files directly to S3 and refreshes the CloudFront cache — no manual deployment steps required.

🔗 **Live Site:** [https://d2az3sc5f38bmc.cloudfront.net/](https://d2az3sc5f38bmc.cloudfront.net/)

---

# 🏗️ Architecture

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
        ├── 📥 Checkout repository code
        ├── 🔑 Configure AWS credentials (IAM user)
        ├── 📤 Sync files to S3 bucket
        └── ♻️ Invalidate CloudFront cache
        ▼
Amazon S3 (Static Website Hosting)
        ▼
Amazon CloudFront (HTTPS + CDN)
        ▼
      👤 End User
```

---

# ⚙️ CI/CD Pipeline

This project uses GitHub Actions for fully automated deployment, defined in [`.github/workflows/deploy.yml`](.github/workflows/deploy.yml).

| Trigger | Action |
|---|---|
| Push to `main` (website files changed) | ✅ Sync to S3 → ♻️ Invalidate CloudFront cache |
| Push touching only `.git/`, `.github/`, or `README.md` | 🚫 Workflow skipped — no wasted deployment |

- ✅ Repository metadata (`.git/`, `.github/`, `README.md`) excluded from the S3 bucket
- ✅ CloudFront cache invalidated after every successful deployment
- ✅ AWS credentials stored securely in GitHub Secrets — never exposed in code
- ✅ Non-sensitive config (region, distribution ID) managed via GitHub Variables

---

# ☁️ AWS Services Used

| Service | Purpose |
|---|---|
| S3 | Static website hosting |
| CloudFront | HTTPS delivery + CDN caching |
| IAM | Scoped credentials for GitHub Actions access |

---

# 📁 Project Structure

```
Portfolio-Website/
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

# 🧩 Key Design Decisions & Challenges

- 🔁 **Reused existing IAM user:** Instead of creating a new IAM user per project, an existing IAM user (already scoped with S3 and CloudFront permissions from a prior project) was reused — simplifying credential management for a personal/free-tier AWS account.
- 🔐 **Secrets vs. Variables:** Sensitive values (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) are stored as encrypted GitHub Secrets. Non-sensitive config (`AWS_REGION`, `CLOUDFRONT_DISTRIBUTION_ID`) is stored as GitHub Variables — reflecting real production practice.
- 🧹 **Clean bucket via excludes:** Initial syncs accidentally pushed `.git/`, `.github/`, and `README.md` into the S3 bucket. Fixed by adding `--exclude` flags to `aws s3 sync`, ensuring only production-ready assets are ever served.
- 🎯 **Smart trigger filtering:** Used negation patterns (`!.git/**`, `!.github/**`, `!README.md`) in the workflow's `paths` filter so documentation-only or pipeline-only changes don't trigger unnecessary deployments.
- ♻️ **Cache invalidation on every deploy:** A CloudFront invalidation (`/*`) runs after every successful sync, ensuring visitors always see the latest version immediately.

---

# 👨‍💻 Author

**Ragunathan K**
AWS Solutions Architect Associate (SAA-C03 | 89%)
AWS Cloud Practitioner (CLF-C02 | 100%)
[GitHub](https://github.com/Ragunathan2305)
