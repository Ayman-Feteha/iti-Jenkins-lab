# Docker Permission Setup for Jenkins

## Problem
Jenkins pipeline fails with "permission denied while trying to connect to the Docker daemon socket" error.

## Solutions

### Solution 1: Add Jenkins user to Docker group (Recommended)

1. **Add jenkins user to docker group:**
   ```bash
   sudo usermod -aG docker jenkins
   ```

2. **Restart Jenkins service:**
   ```bash
   sudo systemctl restart jenkins
   ```

3. **Verify the setup:**
   ```bash
   sudo -u jenkins docker ps
   ```

### Solution 2: Configure Docker socket permissions

1. **Set proper permissions on Docker socket:**
   ```bash
   sudo chmod 666 /var/run/docker.sock
   ```

2. **Or create a systemd override (permanent solution):**
   ```bash
   sudo mkdir -p /etc/systemd/system/docker.service.d/
   sudo tee /etc/systemd/system/docker.service.d/override.conf << EOF
   [Service]
   ExecStart=
   ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H unix:///var/run/docker.sock
   EOF
   ```

3. **Reload and restart Docker:**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

### Solution 3: Use Docker-in-Docker (Alternative)

If the above solutions don't work, you can use a Docker agent in your Jenkinsfile:

```groovy
pipeline {
    agent {
        docker {
            image 'docker:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock --group-add $(stat -c %g /var/run/docker.sock)'
        }
    }
    // ... rest of your pipeline
}
```

### Solution 4: Use Podman (Docker alternative)

If Docker permissions continue to be problematic, consider using Podman:

1. **Install Podman:**
   ```bash
   sudo apt update
   sudo apt install podman
   ```

2. **Create Docker alias:**
   ```bash
   echo 'alias docker=podman' >> ~/.bashrc
   ```

## Verification

After applying any solution, test with:
```bash
# As jenkins user
sudo -u jenkins docker --version
sudo -u jenkins docker ps
```

## Security Note

Adding the jenkins user to the docker group gives it root-equivalent privileges. In production environments, consider using:
- Docker daemon with TLS authentication
- Rootless Docker
- Container runtime security tools
- Dedicated Jenkins agents with Docker

## Troubleshooting

If issues persist:

1. **Check Docker service status:**
   ```bash
   sudo systemctl status docker
   ```

2. **Check Jenkins user:**
   ```bash
   id jenkins
   groups jenkins
   ```

3. **Check Docker socket permissions:**
   ```bash
   ls -la /var/run/docker.sock
   ```

4. **Check Jenkins logs:**
   ```bash
   sudo journalctl -u jenkins -f
   ```