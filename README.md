# PKI-Based 2FA Microservice

A secure, containerized microservice implementing enterprise-grade security practices through Public Key Infrastructure (PKI) and Time-based One-Time Password (TOTP) two-factor authentication.

## Quick Setup Guide

### Prerequisites
- Python 3.11+
- Docker & Docker Compose
- OpenSSL
- Git

### Step 1: Generate RSA Keys

```bash
# Generate 4096-bit RSA private key
openssl genrsa -out student_private.pem 4096

# Extract public key
openssl rsa -in student_private.pem -pubout -out student_public.pem
```

### Step 2: Download Instructor Public Key

```bash
curl -o instructor_public.pem https://partnr-public.s3.us-east-1.amazonaws.com/gpp-resources/instructor_public.pem
```

### Step 3: Request Encrypted Seed

Call the instructor API:
```bash
curl -X POST https://eajeyq4r3zljoq4rpovy2nthda0vtjqf.lambda-url.ap-south-1.on.aws \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "YOUR_STUDENT_ID",
    "github_repo_url": "https://github.com/Bhavyateja04/pki-2fa-microservice",
    "public_key": "YOUR_PUBLIC_KEY_CONTENT"
  }' > encrypted_seed.txt
```

### Step 4: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 5: Run with Docker

```bash
docker-compose build
docker-compose up -d
```

### Step 6: Test Endpoints

```bash
# Decrypt seed
curl -X POST http://localhost:8080/decrypt-seed \
  -H "Content-Type: application/json" \
  -d '{"encrypted_seed": "YOUR_ENCRYPTED_SEED"}'

# Generate 2FA code
curl http://localhost:8080/generate-2fa

# Verify 2FA code
curl -X POST http://localhost:8080/verify-2fa \
  -H "Content-Type: application/json" \
  -d '{"code": "123456"}'
```

## File Structure

- `app.py` - FastAPI application with RSA/TOTP implementation
- `Dockerfile` - Multi-stage Docker build
- `docker-compose.yml` - Compose configuration
- `requirements.txt` - Python dependencies
- `scripts/log_2fa_cron.py` - Cron script for TOTP logging
- `cron/2fa-cron` - Cron configuration (LF line endings)
- `.gitattributes` - Enforces LF line endings for cron file

## Implementation Details

### Cryptographic Operations
- RSA 4096-bit encryption/decryption (RSA/OAEP + SHA-256)
- RSA-PSS signing with SHA-256
- TOTP generation (SHA-1, 30-sec periods, 6 digits)

### API Endpoints
- `POST /decrypt-seed` - Decrypt and store encrypted seed
- `GET /generate-2fa` - Generate current TOTP code
- `POST /verify-2fa` - Verify TOTP code with ±30s tolerance

### Docker Features
- Multi-stage build for optimized image size
- Persistent storage via Docker volumes
- Cron job automation (every minute)
- UTC timezone configuration
- Port 8080 exposure

## Security Notes

- Private keys are committed to repo (educational purposes only)
- Encrypted seed (encrypted_seed.txt) is NOT committed
- All cryptographic parameters follow specifications exactly
- Secure file permissions for volume access

## Deadline

**December 6, 2025, 09:59 AM IST**
