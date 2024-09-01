# Dockerfile Instructions

<br/>

## 01 - Dockerfile Instructions for React Application

```dockerfile
# Stage 1: Build the React app
FROM node:18-alpine AS build

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json (if available)
COPY package*.json ./

# Install dependencies
RUN npm install --frozen-lockfile

# Copy the rest of the application code
COPY . .

# Build the React app
RUN npm run build

# Stage 2: Serve the React app with a non-root user
FROM node:18-alpine

# Install a simple HTTP server
RUN npm install -g serve

# Create a system user with limited privileges
RUN adduser -D -H -s /bin/false nonrootuser

# Set the working directory
WORKDIR /app

# Copy the build output from the previous stage
COPY --from=build /app/dist ./dist

# Change ownership to the non-root user
RUN chown -R nonrootuser:nonrootuser /app

# Switch to the non-root user
USER nonrootuser

# Expose the port the app will run on
EXPOSE 3000

# Command to serve the app
CMD ["serve", "-s", "dist", "-l", "3000"]

```

The `--frozen-lockfile` flag in the `npm install` command ensures that the exact versions specified in the package-lock.json file are installed, without modifying the lock file. If the package-lock.json is out of sync with package.json, the installation will fail, ensuring consistent and repeatable builds.

**Key Points:**

- **Multi-stage Build:** The Dockerfile uses multi-stage builds to separate the build process from the final image, reducing the image size.
- **Non-Root User:** A system user named nonrootuser is created with limited privileges to enhance security, limiting access to the file system.
- **Alpine Base Image:** The node:18-alpine image is lightweight, making the final Docker image smaller and more efficient.
- The instruction `RUN chown -R nonrootuser:nonrootuser /app` in the Dockerfile is used to change the ownership of the /app directory and all its contents to the nonrootuser.

**The purpose of defining non root uesr**

- **Security:** By changing the ownership of the application files to a non-root user, you prevent the container from running as the root user, which enhances security. This limits the potential damage that could occur if the application is compromised.
- **Permissions:** It ensures that the nonrootuser has the necessary permissions to read, write, and execute files in the /app directory, which is where the application code and dependencies are stored.

> In essence, this step helps enforce the principle of least privilege by ensuring that the application runs with the minimum permissions necessary.

<br/>

Here's an optimized Dockerfile that packages your React development code using Node on top of Alpine as the base image and creates a system user with limited privileges:

```Dockerfile
# Base image with Node on Alpine
FROM node:18-alpine

# Create a system user with limited privileges
RUN adduser -D -H -s /bin/false nonrootuser

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json (if available)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the development code into the container
COPY . .

# Change ownership to the non-root user
RUN chown -R nonrootuser:nonrootuser /app

# Switch to the non-root user
USER nonrootuser

# Expose the port the app will run on
EXPOSE 3000

# Start the React development server
CMD ["npm", "start"]

```

<br/>

## 02 - Why creating an user define top of the Dockerfile?

When creating an user, file system will be updated. We want to cache that process in one layer, otherwise user will create & written configurations in file system again & again.

<br/>

## 03 - Why does COPY command in Dockerfile result in files owned by root user?

Even though you've set a non-root user with USER app, the COPY command still runs with the root user's permissions. Docker applies the USER instruction to commands that come after it, but COPY and ADD are executed with the default root user unless explicitly specified otherwise.

To ensure the files are owned by the app user, you should use the --chown option with COPY, like this:

```
COPY --chown=app:app package*.json .
```

> The `--chown=app:app` option in the COPY command sets the ownership of the copied files and directories to the user app and group app, ensuring that the files are owned by app instead of the default root user.

The correct order is `--chown=user:group`. For example, --chown=app:app sets the user to app and the group to app.

<br/>

## 04 - Explain `COPY --chown=app:app package*.json .`

The statement COPY --chown=app:app package\*.json . in a Dockerfile performs the following actions:

- **`COPY`:** Copies files from the host machine into the Docker image.
- **`--chown=app`**: Sets the ownership of the copied files to the user app and group app.
- **`package.json\*`:** This is a pattern that matches and copies any file in the build context with a name starting with "package" and ending with ".json" (e.g., package.json and package-lock.json).
- **`.` :** Specifies the destination directory within the container's filesystem (typically the current WORKDIR).

> This command copies all package\*.json files from the host into the container and ensures that these files are owned by the app user and group, which can help maintain proper file permissions and security in the container.

<br/>

## 05 - How to use `chown` option

There are two ways:

1. As an inline instruction (Best Ways)
2. As a seperate lines of instructions

### As an inline instruction

```dockerfile
# Use COPY --chown to set ownership during the copy process
COPY --chown=appuser:appgroup package*.json ./
```

### As a seperate lines of instructions

```dockerfile
# Ensure you're running as root
USER root

# Change ownership of specific files
RUN chown appuser:appgroup /app/package.json /app/package-lock.json

# Optionally, switch back to the non-root user if needed
USER appuser
```

<br/>

## 06 - Adjust file permissions instead of ownership

```dockerfile
# Adjust file permissions instead of ownership
RUN chmod 644 /app/package.json /app/package-lock.json

```
