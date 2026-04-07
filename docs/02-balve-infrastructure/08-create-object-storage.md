# Object Storage

In the Hetzner console, go to _Object Storage_ and create the following buckets:

| Bucket name           | Location | Visibility | Object lock |
| --------------------- | -------- | ---------- | ----------- |
| `balve-strapi`        | Helsinki | `Private`  | `Disabled`  |
| `balve-nextcloud`     | Helsinki | `Private`  | `Disabled`  |
| `balve-calendar`      | Helsinki | `Public`   | `Disabled`  |
| `balve-internal-data` | Helsinki | `Private`  | `Disabled`  |
| `balve-shared`        | Helsinki | `Private`  | `Disabled`  |
| `balve-config`        | Helsinki | `Private`  | `Disabled`  |

## CORS

The `balve-strapi` bucket needs a CORS policy for image previews in the Strapi admin panel.

Install the AWS CLI and configure it with Hetzner S3 credentials:

```bash
brew install awscli
aws configure
# AWS Access Key ID: <from Hetzner Console → Security → S3 credentials>
# AWS Secret Access Key: <from Hetzner Console → Security → S3 credentials>
# Default region name: hel1
# Default output format: json
```

Apply the CORS policy:

```bash
aws s3api put-bucket-cors \
  --bucket balve-strapi \
  --endpoint-url https://hel1.your-objectstorage.com \
  --cors-configuration '{
    "CORSRules": [{
      "AllowedOrigins": ["https://strapi.balve.garmeres.com"],
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "HEAD"]
    }]
  }'
```

Verify that the CORS rules were applied:

```bash
aws s3api get-bucket-cors \
  --bucket balve-strapi \
  --endpoint-url https://hel1.your-objectstorage.com
```

Should return the same rules you just applied:

```json
{
    "CORSRules": [
        {
            "AllowedHeaders": ["*"],
            "AllowedMethods": ["GET", "HEAD"],
            "AllowedOrigins": ["https://strapi.balve.garmeres.com"]
        }
    ]
}
```
