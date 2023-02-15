# Sftp

## Docker
For local testing and writing software that connects to a real SFTP server, you can spin up a local docker container.

Create a public key by running `ssh-keygen -t rsa`. The public part must be added to the container.  
The key pair is typically saved to your `~/.ssh/` folder.

```
docker run \
-v ~/.ssh/docker/id_rsa.pub:/home/user/.ssh/keys/id_rsa.pub:ro \
-v ~/docker-volumes:/home/user/upload \
--name sftp- \
-p 2223:22 \
-d atmoz/sftp user::1001
```

## Java
When using JSch, a popular, but old SFTP library, you'll need to have converted the public (.pub file) key to PEM format.

```
cp id_rsa.pub id_rsa.pem
ssh-keygen -f id_rsa.pem -m pem
```