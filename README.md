# Secret Volume Mount Documentation

### Step 1: Obtain Files
- Get the `.json` and `.env` files from the developers.

### Step 2: Encode Secrets
- Encode the `.json` file and any secrets from the `.env` file (such as `GROQ_API_KEY`, `GOOGLE_APPLICATION_CREDENTIALS`, or `TAVILY_API_KEY`) using Base64 encoding.

#### Encoding Command:
```bash
echo -n 'your-plain-text-value' | base64
```

- You can verify the encoded values using this [Base64 decode tool](https://www.base64decode.org/).

### Step 3: Create Kubernetes Secret YAML
- Replace the values in the following template with your Base64-encoded data, then apply the secret.

```yaml
apiVersion: v1
data:
  <key-1>: <base64-encoded-value>  # Base64 encoded secret data (.json file)
  <key-2>: <base64-encoded-value>  # Additional key-value pairs as needed (.env)
kind: Secret
metadata:
  name: <secret-name>              # Name of the secret
  namespace: <namespace>           # Namespace where the secret will be used
type: Opaque                       # Secret type, Opaque is used for arbitrary user-defined data
```

### Example:

Before encoding:
```json
{
    "type": "service_account",
    "project_id": "<project-id>",
    "private_key_id": "<private-key-id>",
    "private_key": "-----BEGIN PRIVATE KEY-----\n<private-key>\n-----END PRIVATE KEY-----\n",
    "client_email": "<client-email>@<project-id>.iam.gserviceaccount.com",
    "client_id": "<client-id>",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/<client-email>%40<project-id>.iam.gserviceaccount.com",
    "universe_domain": "googleapis.com"
}
```

After encoding:
```
<base64-encoded-json>
```

Additional encoded secrets:
```
GROQ_API_KEY: <base64-encoded-groq-api-key>
GOOGLE_APPLICATION_CREDENTIALS: <base64-encoded-google-credentials>
TAVILY_API_KEY: <base64-encoded-tavily-api-key>
```

### Complete `secret.yaml` example:

```yaml
apiVersion: v1
data:
  <project-id>.json: <base64-encoded-json>
  GROQ_API_KEY: <base64-encoded-groq-api-key>
  GOOGLE_APPLICATION_CREDENTIALS: <base64-encoded-google-credentials>
  TAVILY_API_KEY: <base64-encoded-tavily-api-key>
kind: Secret
metadata:
  name: <secret-name>
  namespace: <namespace>
type: Opaque
```

### Step 4: Mount Secret in Deployment

To mount this secret as a volume in your Kubernetes deployment, use the following configuration in your `deployment.yaml`:

#### Example `deployment.yaml` (Volume Mount):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <deployment-name>
  namespace: <namespace>
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <app-label>
  template:
    metadata:
      labels:
        app: <app-label>
    spec:
      containers:
      - name: <container-name>
        image: <container-image>
        volumeMounts:
        - name: secret-volume
          mountPath: "/path/in/container"  # Path to mount the secret
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: <secret-name>  # Name of the secret created earlier
```

### Step 5: Access Mounted Secret

Once mounted, you can access the secret in the container as a file at the specified mount path (e.g., `/path/in/container`).

### Example Script to Access Mounted Secrets

If the secrets are mounted as files, you can access them with a script in your application:

```bash
#!/bin/bash

# Path to mounted secrets
SECRET_PATH="/path/in/container"

# Load secrets into environment variables (example for a JSON file)
export GOOGLE_APPLICATION_CREDENTIALS=$(cat $SECRET_PATH/<project-id>.json)
export GROQ_API_KEY=$(cat $SECRET_PATH/GROQ_API_KEY)
export TAVILY_API_KEY=$(cat $SECRET_PATH/TAVILY_API_KEY)

# Use the secrets in your application
echo "Google Application Credentials: $GOOGLE_APPLICATION_CREDENTIALS"
echo "GROQ API Key: $GROQ_API_KEY"
echo "Tavily API Key: $TAVILY_API_KEY"
```

You can adjust the script based on how your secrets are used. Ensure the path matches the `mountPath` in your deployment YAML.

### Secret Copy Script Documentation 
Kudos to [Osama Yusuf](https://github.com/Osama-Yusuf)

### Script Overview
This script automates the process of copying Kubernetes secrets from one namespace to another. It's useful when you need to replicate secrets between different environments, such as from production to a development namespace.

### Script link: [Copy Secrets Script](https://github.com/Osama-Yusuf/Shell-Sorcery/blob/main/DevOps-Sorcery/copy-secret-ns.yml)

### Usage

1. **Prerequisites:**
   - Ensure that you have `kubectl` installed and configured to access the Kubernetes cluster.
   - You must have permission to read from the source namespace and write to the target namespace.

2. **Customization:**
   - Update the `SOURCE_NAMESPACE` and `TARGET_NAMESPACE` variables with your source and target namespace names.
   - Modify the `SECRETS` array to include the list of secret names you wish to copy.

3. **Running the Script:**
   - Save the script to a file (e.g., `copy_secrets.sh`).
   - Make the script executable:
     ```bash
     chmod +x copy_secrets.sh
     ```
   - Run the script:
     ```bash
     ./copy_secrets.sh
     ```

### Additional Resources
For more examples and Kubernetes-related scripts, visit [Shell-Sorcery](https://github.com/Osama-Yusuf/Shell-Sorcery/tree/main).

