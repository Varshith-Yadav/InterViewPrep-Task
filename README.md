# CI/CD Deployment of Flask App using GitHub Actions to GCP VM

## 📘 Documentation

---

## ✅ Steps Followed

1. 📦 Created a simple Flask application with:
   - app.py
   - requirements.txt
   - Dockerfile

2. 🔐 Generated SSH key pair:
   - Used ssh-keygen to generate private and public key.
   - Added public key to ~/.ssh/authorized_keys on the GCP VM.
   - Added private key to GitHub Secrets as SSH_PRIVATE_KEY.

3. ⚙️ Set up GitHub Actions workflow:
   - Created .github/workflows/deploy.yml
   - The workflow includes three steps:
     - Checkout source code
     - SSH into the VM and deploy Docker container
     - Start container with port mapping

4. 🐳 Installed Docker manually on the VM:
   - Used the official Docker install script:
     ```bash
     curl -fsSL https://get.docker.com -o get-docker.sh
     sudo sh get-docker.sh
     ```

5. 🚀 Triggered deployment:
   - Pushed code to the main branch to initiate GitHub Actions workflow.

---

## ❌ Problems Faced and ✅ How I Solved Them

1. ❌ Problem: Jenkins SSH deployment failed due to private key permissions.
   - 🧠 Reason: Jenkins on Windows had loose file permissions on the SSH private key file.
   - ✅ Solution: Switched from Jenkins to GitHub Actions for easier SSH and better compatibility.

2. ❌ Problem: GitHub Actions showed error: "UNPROTECTED PRIVATE KEY FILE"
   - 🧠 Reason: Key file permissions were not properly set inside the runner.
   - ✅ Solution: Explicitly set chmod 600 for ~/.ssh/id_rsa inside GitHub Actions using:
     ```yaml
     - name: Set up SSH
       run: |
         mkdir -p ~/.ssh
         echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
         chmod 600 ~/.ssh/id_rsa
     ```

3. ❌ Problem: Docker was not found on GCP VM
   - 🧠 Reason: Docker was not pre-installed
   - ✅ Solution: Installed Docker manually on the GCP VM using the get-docker.sh script

4. ❌ Problem: Flask app wasn’t accessible from browser
   - 🧠 Reason: The container exposed port 5000, but host needed port 80
   - ✅ Solution: Mapped port 5000 to 80 when running Docker container:
     ```bash
     docker run -d -p 80:5000 --name flask-container flask-app
     ```

5. ❌ Problem: GitHub Actions copy command (scp) failed with permission error
   - 🧠 Reason: Host key not added to known_hosts
   - ✅ Solution: Added host using ssh-keyscan:
     ```yaml
     ssh-keyscan -H <GCP_VM_IP> >> ~/.ssh/known_hosts
     ```

---

✅ Final Result:  
Upon pushing to the main branch, GitHub Actions builds and deploys the Flask app in a Docker container to the GCP VM, making it live on port 80.

