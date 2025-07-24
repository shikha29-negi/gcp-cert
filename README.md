# gcp-cert

Prerequisites

1.  Create a Service Account
gcloud iam service-accounts create github-actions-sa \
  --display-name="GitHub Actions Service Account"

2. Grant it necessary roles:
   gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="serviceAccount:github-actions-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/secretmanager.admin"

   gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="serviceAccount:github-actions-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountTokenCreator"

3. Create Workload Identity Pool & Provider- This creates a pool that can receive identities from external identity providers like GitHub. Also this sets up the trust between GCP and GitHub OIDC tokens
   gcloud iam workload-identity-pools create "github-pool" \
  --project="YOUR_PROJECT_ID" \
  --location="global" \
  --display-name="GitHub Actions Pool"

  gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --project="YOUR_PROJECT_ID" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --display-name="GitHub OIDC Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --attribute-condition="attribute.repository=='GITHUB_USERNAME/YOUR_REPO_NAME'" \
  --issuer-uri="https://token.actions.githubusercontent.com"

4.Bind GitHub Repo Identity to the Service Account
  gcloud iam service-accounts add-iam-policy-binding github-actions-sa@YOUR_PROJECT_ID.iam.gserviceaccount.com \
  --project=YOUR_PROJECT_ID \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/YOUR_PROJECT_NUMBER/locations/global/workloadIdentityPools/github-    pool/attribute.repository/GITHUB_USERNAME/YOUR_REPO_NAME"



