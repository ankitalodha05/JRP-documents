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

**Q: What happens if I manually stop an EC2 instance created by Terraform and then run `terraform apply`?**

**A: Nothing will happen to the instance — it will remain in the stopped state.**

Terraform is **declarative** and tracks the infrastructure state. When you run `terraform apply`, it:

* Compares the **actual state** in AWS with your **Terraform configuration**.
* Since stopping an instance doesn't change its configuration, Terraform sees **no difference** and doesn't take any action.

---

**Q: Does Terraform manage the running or stopped state of an EC2 instance?**

**A: No, by default Terraform does not manage the running or stopped state** of EC2 instances.
It only ensures that the instance exists and matches the configuration (like instance type, AMI, tags, etc.).

---

**Q: What if I terminate (delete) the EC2 instance manually and then run `terraform apply`?**

**A: Terraform will detect the missing instance and recreate it.**

Terraform compares your AWS environment with the state file and desired configuration. If a resource is missing (like a terminated EC2), it will **provision it again**.

---

**Q: Is there any way to make Terraform automatically restart stopped EC2 instances?**

**A: Not by default.**
Terraform doesn’t manage instance runtime state unless you:

* Use external tools/scripts.
* Use lifecycle hooks or provisioning logic to start/stop EC2.
* Or manage instance state separately (not recommended with Terraform alone).

---


