# CI/CD for Terraform

CI/CD workflow for terraform templates

- Ensure your terraform is linted and formatted before it is pushed to your repos
- Collaborate and discuss terraform plan outputs within pull requests
- Automatically apply the plan once PR is approved and merged


## Workflow overview

1. Cut a new branch, commit your changes
2. pre-commit hook triggers locally and tests your code before committing
3. On code push, pre-commit hooks will run again via github actions
4. On PR, a `tf plan` will be run and the plan output will be put into the top comment of the PR
    - A hash of the plan output will also be written as an additional comment in the PR
    - Approval of the PR is approval of the `tf plan` output in addition to code review
5. On merge to main:
    - `terraform plan` will be run again and the hash of that output will be compared to the previous hash left in the comments and `terraform apply` will be run if the hashes match (aka no drift)

If there is drift, `terraform apply` will not be run.  Since the PR will be closed, here is how to remediate:
- revert the state back to what is in terraform (changes were made out of band), or
- update the comment containing the hash with the hash in the output of the failed GH action and rerun the action

## Installing the precommit hooks

Install all dependencies for all available hooks.  Omit hooks that aren't being used if necessary (see the currently enabled hooks here: `.pre-commit-config.yaml`).

### MacOS:
`brew install pre-commit terraform-docs tflint tfsec trivy checkov terrascan infracost tfupdate minamijoyo/hcledit/hcledit jq`

### Enable hooks
Run `pre-commit install` within the repository to turn on the hooks locally.

#### Alternative installation instructions and methods
##### Ubuntu 20.04+
```
sudo apt update
sudo apt install -y unzip software-properties-common python3 python3-pip python-is-python3
python3 -m pip install --upgrade pip
pip3 install --no-cache-dir pre-commit
pip3 install --no-cache-dir checkov
curl -L "$(curl -s https://api.github.com/repos/terraform-docs/terraform-docs/releases/latest | grep -o -E -m 1 "https://.+?-linux-amd64.tar.gz")" > terraform-docs.tgz && tar -xzf terraform-docs.tgz terraform-docs && rm terraform-docs.tgz && chmod +x terraform-docs && sudo mv terraform-docs /usr/bin/
curl -L "$(curl -s https://api.github.com/repos/tenable/terrascan/releases/latest | grep -o -E -m 1 "https://.+?_Linux_x86_64.tar.gz")" > terrascan.tar.gz && tar -xzf terrascan.tar.gz terrascan && rm terrascan.tar.gz && sudo mv terrascan /usr/bin/ && terrascan init
curl -L "$(curl -s https://api.github.com/repos/terraform-linters/tflint/releases/latest | grep -o -E -m 1 "https://.+?_linux_amd64.zip")" > tflint.zip && unzip tflint.zip && rm tflint.zip && sudo mv tflint /usr/bin/
curl -L "$(curl -s https://api.github.com/repos/aquasecurity/tfsec/releases/latest | grep -o -E -m 1 "https://.+?tfsec-linux-amd64")" > tfsec && chmod +x tfsec && sudo mv tfsec /usr/bin/
curl -L "$(curl -s https://api.github.com/repos/aquasecurity/trivy/releases/latest | grep -o -E -i -m 1 "https://.+?/trivy_.+?_Linux-64bit.tar.gz")" > trivy.tar.gz && tar -xzf trivy.tar.gz trivy && rm trivy.tar.gz && sudo mv trivy /usr/bin
sudo apt install -y jq && \
curl -L "$(curl -s https://api.github.com/repos/infracost/infracost/releases/latest | grep -o -E -m 1 "https://.+?-linux-amd64.tar.gz")" > infracost.tgz && tar -xzf infracost.tgz && rm infracost.tgz && sudo mv infracost-linux-amd64 /usr/bin/infracost && infracost auth login
curl -L "$(curl -s https://api.github.com/repos/minamijoyo/tfupdate/releases/latest | grep -o -E -m 1 "https://.+?_linux_amd64.tar.gz")" > tfupdate.tar.gz && tar -xzf tfupdate.tar.gz tfupdate && rm tfupdate.tar.gz && sudo mv tfupdate /usr/bin/
curl -L "$(curl -s https://api.github.com/repos/minamijoyo/hcledit/releases/latest | grep -o -E -m 1 "https://.+?_linux_amd64.tar.gz")" > hcledit.tar.gz && tar -xzf hcledit.tar.gz hcledit && rm hcledit.tar.gz && sudo mv hcledit /usr/bin/
```

##### Docker
If you want a more portable solution, execute the hooks via a docker container that has all of the dependencies:
`docker run -e USERID=$(id -u):$(id -g) -v $(pwd):/lint -w /lint ghcr.io/antonbabenko/pre-commit-terraform:latest run -a`

Only draw back is that pre-commit will not execute this automatically, you will need to finesse some sort of solution to get pre-commit hooks to run automatically in a container.


## Future enhancements
- Cache pre-commit docker image in github actions to increase test completion speed
- Put terraform plan output in a separate comment to preserve the PR's top level comment
- Add timestamp into the comments being put into the PR
- Make terraform plan workflow callable to manually refresh the PR before merging
