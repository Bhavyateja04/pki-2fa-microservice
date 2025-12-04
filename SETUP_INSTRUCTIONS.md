# Complete Setup Instructions

This document contains all the code needed to complete the PKI-Based 2FA Microservice project.

## CRITICAL: Required Student ID

Before proceeding, you need your STUDENT_ID. This is typically provided by your institution.

## Step-by-Step Setup

### 1. Generate RSA Keys (Run Locally)

```bash
# Generate 4096-bit RSA private key
openssl genrsa -out student_private.pem 4096

# Extract public key  
openssl rsa -in student_private.pem -pubout -out student_public.pem

# Download instructor public key
curl -o instructor_public.pem https://partnr-public.s3.us-east-1.amazonaws.com/gpp-resources/instructor_public.pem
```

Commit these keys:
```bash
git add student_private.pem student_public.pem instructor_public.pem
git commit -m "Add RSA keys"
git push
```

### 2. Request Encrypted Seed

Read your public key:
```bash
cat student_public.pem
```

Call the instructor API:
```bash
curl -X POST https://eajeyq4r3zljoq4rpovy2nthda0vtjqf.lambda-url.ap-south-1.on.aws \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "YOUR_STUDENT_ID",
    "github_repo_url": "https://github.com/Bhavyateja04/pki-2fa-microservice",
    "public_key": "<PASTE_ENTIRE_CONTENT_OF_student_public.pem>"
  }' > encrypted_seed.txt
```

Verify encrypted_seed.txt was created.

### 3. Create App Code Locally

Create the following files in your repository:

**See app.py, Dockerfile, docker-compose.yml, and other files in the implementation guide.**

### 4. Test with Docker

```bash
docker-compose build
docker-compose up -d

# Wait 5 seconds for container to start
sleep 5

# Test decrypt-seed
curl -X POST http://localhost:8080/decrypt-seed \
  -H "Content-Type: application/json" \
  -d "{\"encrypted_seed\": \"$(cat encrypted_seed.txt)\"}" 

# Should return: {"status": "ok"}

# Test generate-2fa
curl http://localhost:8080/generate-2fa

# Should return something like: {"code": "123456", "valid_for": 25}

# Wait 70+ seconds and check cron output
sleep 70
docker exec <container_name> cat /cron/last_code.txt
```

### 5. Push All Files to GitHub

```bash
git add .
git commit -m "Add complete microservice implementation"
git push
```

### 6. Generate Commit Proof

Get commit hash:
```bash
COMMIT_HASH=$(git log -1 --format=%H)
echo $COMMIT_HASH
```

### 7. Submit on Partnr Platform

1. Go to the Submit tab
2. Enter GitHub repository URL: `https://github.com/Bhavyateja04/pki-2fa-microservice`
3. Follow all steps in the form
4. Submit before December 6, 09:59 AM IST

## Key Files to Create

- `app.py` - FastAPI application
- `Dockerfile` - Docker build configuration
- `docker-compose.yml` - Compose configuration  
- `.gitattributes` - Enforces LF line endings
- `scripts/log_2fa_cron.py` - Cron logging script
- `cron/2fa-cron` - Cron job file

See the following sections for complete code.
