```shell
# list current config details
gcloud info

# list accounts
gcloud auth list

# list availalbe configs
gcloud config configurations list



# get currently active account
export GCP_ACCOUNT=$(gcloud auth list --format=json | jq '.[].account' -r);

# list billing accounts
gcloud alpha billing accounts list 

# list orgs
gcloud organizations list

# list projects
gcloud projects list

# list project service accounts
gcloud projects get-iam-policy <project_name>

# list service accounts
gcloud iam service-accounts list

# list account permissions
gcloud iam service-accounts

# list service account keys
gcloud iam service-accounts keys list --iam-account=${GCP_ACCOUNT}

```