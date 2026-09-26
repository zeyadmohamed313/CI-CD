# Commands: Put This Project on GitHub and Release It

Run these commands one block at a time. Replace values written like `YOUR-USERNAME` with your real GitHub username.

## Step 0: Open this project folder

```bash
cd "/home/zeyad/work/Study/SideProjects/CI CD"

# Use the JDK installed on this Ubuntu machine.
export JAVA_HOME=/usr/lib/jvm/java-1.25.0-openjdk-amd64
export PATH="$JAVA_HOME/bin:$PATH"

# One-time repair for build files created by Docker with another owner.
if [ -d target ]; then sudo chown -R "$USER":"$USER" target; fi
```

Check that you are in the correct folder:

```bash
ls
```

You should see `pom.xml`, `README.md`, and `Dockerfile`.

## Step 1: Check your tools

```bash
java -version
mvn -version
git --version
```

Java must be version 21 or newer. For this Ubuntu machine, the commands in Step 0 select the installed Java 25 for the current terminal. Docker is needed later when you want to run container images.

## Step 2: Check your GitHub username (optional)

If you have GitHub CLI installed:

```bash
gh auth status
gh api user --jq .login
```

If the command says you are not logged in, run:

```bash
gh auth login
```

If `gh` is not installed, sign in at [github.com](https://github.com). Your username appears at the top-right of the page.

## Step 3: Test the application before uploading it

```bash
mvn clean verify
```

Continue only when you see `BUILD SUCCESS`.

## Step 4: Create an empty GitHub repository

1. Go to [github.com/new](https://github.com/new).
2. Set the repository name to `cicd-learning-app`.
3. Choose **Public** or **Private**.
4. Do **not** add a README, `.gitignore`, or license. This folder already has them.
5. Click **Create repository**.
6. Copy the HTTPS repository URL. It looks like this:

   ```text
   https://github.com/YOUR-USERNAME/cicd-learning-app.git
   ```

## Step 5: Connect this folder to GitHub

First, set your Git name and email if you have never made a Git commit on this computer:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Then create the local Git repository and make the first commit:

```bash
git init -b main
git add .
git status
git commit -m "Initial Spring Boot CI/CD learning app"
```

Connect it to your repository. Replace the URL with the one you copied in Step 4:

```bash
git remote add origin https://github.com/YOUR-USERNAME/cicd-learning-app.git
git remote -v
git push -u origin main
```

Open your GitHub repository in the browser. Click the **Actions** tab and wait for **Continuous Integration** to become green.

## Step 6: Make a safe practice change

Create a branch:

```bash
git checkout -b change-greeting
```

Edit `src/main/java/com/zeyad/cicd/GreetingController.java`. For example, change `Hello` to `Hi`.

Test and upload the branch:

```bash
mvn clean verify
git add src/main/java/com/zeyad/cicd/GreetingController.java
git commit -m "Change greeting text"
git push -u origin change-greeting
```

On GitHub, click **Compare & pull request**, create the pull request, and wait for CI to pass. Then merge it.

## Step 7: Create your first release

After merging the pull request, run:

```bash
git checkout main
git pull --ff-only origin main
git tag -a v0.1.0 -m "First release"
git push origin v0.1.0
```

Go back to **Actions**. The **Continuous Delivery** workflow will:

1. Run the tests again.
2. Build a Docker image.
3. Publish it to GitHub Container Registry.

## Step 8: Run the published image

Replace `YOUR-USERNAME` below:

```bash
docker pull ghcr.io/YOUR-USERNAME/cicd-learning-app:v0.1.0
docker run --rm -p 8080:8080 ghcr.io/YOUR-USERNAME/cicd-learning-app:v0.1.0
```

Open another terminal and test it:

```bash
curl 'http://localhost:8080/api/greeting?name=Zeyad'
curl http://localhost:8080/actuator/health
```

Press `Ctrl+C` in the terminal running Docker when you are finished.

## Useful commands when you are unsure

```bash
git status
git branch
git log --oneline --max-count=5
git remote -v
```

These commands only show information; they do not change your code.
