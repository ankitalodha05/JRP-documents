Q\&A:

---

**Q: How does my system understand AWS commands?**
To interact with AWS services from your system, you need to install the **AWS CLI (Command Line Interface)**. The AWS CLI acts as a bridge between your local machine and the AWS cloud. Once installed, your system will be able to understand and execute AWS commands by sending requests to AWS services.

---

**Q: How does the request reach my AWS account?**
Authentication is the key. After installing the AWS CLI, you run `aws configure` and provide your **Access Key ID** and **Secret Access Key**. These credentials determine which AWS account the request should be sent to. The CLI uses these keys to authenticate and authorize your requests securely.

---

**Q: I want to interact with AWS using Terraform. How is that different?**
When using Terraform, there’s a separate setup process. Terraform needs specific **provider plugins** to communicate with different cloud platforms like AWS.
To use AWS with Terraform:

* You define the cloud provider in a `provider` block within your Terraform code.
* When you run the `terraform init` command, Terraform contacts the **Terraform Registry** to download the necessary AWS provider plugin and stores it in a hidden `.terraform` folder in your working directory.
* Once initialized, Terraform can interact with your AWS account using the same credentials configured earlier.

---
