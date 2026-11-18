# DevOps Practicals 2 and 3: Jenkins with Maven

**LYCS DevOps Sem 7**

Repository: https://github.com/show-star/aa2

📘 **Viva questions and easy explanations for all 6 practicals:** [VIVA.md](VIVA.md)

Both practicals use the same Java Maven project in this repo.

| Practical | What you build | Jump to |
| --- | --- | --- |
| **Practical 2 (Q2)** | Jenkins Freestyle job `java-build-job` with Poll SCM and `mvn clean package` | [Practical 2](#practical-2-jenkins-freestyle-job-with-maven) |
| **Practical 3 (Q3)** | Jenkins Pipeline `cicd-pipeline` with Checkout, Build, Test, Package and Deploy stages, and an automatic trigger | [Practical 3](#practical-3-cicd-pipeline-with-jenkins-and-maven) |

---

# Practical 2: Jenkins Freestyle Job with Maven

**Q2: Setting up Jenkins and a CI job (30 marks)**

## Aim

Install Jenkins, then create a Jenkins **Freestyle job** named `java-build-job`. The job pulls this Java Maven project from GitHub, checks GitHub for new commits every 2 minutes (**Poll SCM**), and builds the project with `mvn clean package`.

## What this practical is about

- **Jenkins** is an automation server used for **CI (Continuous Integration)**. Every time code changes, Jenkins builds and tests it automatically, so mistakes are caught early.
- A **Freestyle job** is the simplest kind of Jenkins job. You set it up by filling in forms: where the code is, when to build, and which build command to run.
- **Maven** is a Java build tool. It reads `pom.xml`, downloads the libraries the project needs (here, JUnit), compiles the code, runs the tests and packages everything into a `.jar` file.
- **Poll SCM** means Jenkins checks the Git repository on a schedule and starts a build when it finds a new commit.

## What's in this repo

```
aa2/
├── Jenkinsfile                              Pipeline script for Practical 3
├── pom.xml                                  Maven project file (Java 17, JUnit 4)
├── src/main/java/com/example/App.java       The program: adds two numbers
└── src/test/java/com/example/AppTest.java   JUnit test that checks add(2, 3) = 5
```

| Maven command | What it does |
| --- | --- |
| `mvn clean` | Deletes the old `target` folder |
| `mvn compile` | Compiles `App.java` |
| `mvn test` | Runs `AppTest` |
| `mvn package` | Compiles, tests and creates `target/java-maven-demo-1.0.jar` |

---

## Part 0: Install and start Jenkins (Q2.1, 6 marks)

Skip this part if Jenkins already opens at `http://localhost:8080`.

### Step 1: Install Java 21

1. Press the **Windows key**, type **PowerShell**, then click **Run as administrator** and click **Yes**.
2. Run:
   ```powershell
   winget install EclipseAdoptium.Temurin.21.JDK
   ```
3. Close PowerShell, open a new one and check:
   ```powershell
   java -version
   ```
   It should print a version starting with **21**.

If `winget` isn't recognized, download **Temurin 21 (.msi)** from https://adoptium.net and install it. During the install, set **Set JAVA_HOME variable** to *Will be installed*.

### Step 2: Install Git

1. Download **Git for Windows** from https://git-scm.com and install it with the default options.
2. Check it in a new Command Prompt: `git --version`

### Step 3: Install Jenkins

1. Go to https://www.jenkins.io/download and, under **LTS**, download the **Windows** installer (`.msi`).
2. Run the installer and click **Next**.
3. **Logon type:** choose **Run service as LocalSystem**, then **Next**.
4. **Port:** keep **8080**. Click **Test Port**, then **Next**.
5. **Java home:** choose the JDK 21 folder, for example `C:\Program Files\Eclipse Adoptium\jdk-21...`, then **Next**.
6. Click **Install**, then **Finish**.

### Step 4: First-time setup

1. Open Chrome and go to **http://localhost:8080**. The **Unlock Jenkins** page appears.
2. Open this file in Notepad and copy the password inside it:
   ```
   C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword
   ```
3. Paste the password into the page and click **Continue**.
4. Click **Install suggested plugins** and wait until every plugin shows a green tick.
5. Fill in the **Create First Admin User** form (username, password, full name and email), then click **Save and Continue**. Write down the password.
6. Keep the Jenkins URL as `http://localhost:8080/`. Click **Save and Finish**, then **Start using Jenkins**.
7. Restart Jenkins so it can find Git. Open PowerShell or Command Prompt **as Administrator** and run:
   ```
   net stop jenkins
   net start jenkins
   ```

📸 **Screenshots:** `java -version`, the Unlock Jenkins page, plugins installing, Create First Admin User, and the Jenkins dashboard.

---

## Part A: Add Maven to Jenkins (one time)

1. In Jenkins, click **Manage Jenkins**, then **Tools**.
2. Scroll down to **Maven installations** and click **Add Maven**.
3. **Name:** `Maven3` (type exactly this).
4. Tick **Install automatically** and keep the latest **3.9.x** version.
5. Click **Save**.

Jenkins downloads Maven by itself the first time a job uses it. You don't need to install Maven on Windows.

---

## Part B: Create the Freestyle job (Q2.2 to Q2.4)

1. On the Jenkins dashboard, click **New Item**.
2. Name: **`java-build-job`**. Select **Freestyle project** and click **OK**.

### Source Code Management (Q2.3, 6 marks)

3. Select **Git**.
4. **Repository URL:**
   ```
   https://github.com/show-star/aa2.git
   ```
5. **Branch Specifier:** change `*/master` to **`*/main`**.

### Triggers (Q2.4)

6. Tick **Poll SCM**.
7. **Schedule:**
   ```
   H/2 * * * *
   ```
   This tells Jenkins to check GitHub for new commits every 2 minutes. Jenkins shows a note under the box explaining the schedule.

### Build Steps (Q2.4)

8. Click **Add build step**, then **Invoke top-level Maven targets**.
9. **Maven Version:** choose **Maven3**.
10. **Goals:**
    ```
    clean package
    ```
11. Click **Save**.

📸 **Screenshot:** the job configuration (Git URL and branch, Poll SCM, Maven goals).

---

## Part C: Run the build and check Console Output (Q2.5, 7 marks)

1. On the job page, click **Build Now**.
2. Under **Build History**, click **#1**, then **Console Output**.
3. The first build takes a few minutes, because Jenkins downloads Maven and JUnit. The output ends like this:

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
...
[INFO] Building jar: C:\ProgramData\Jenkins\.jenkins\workspace\java-build-job\target\java-maven-demo-1.0.jar
[INFO] BUILD SUCCESS
...
Finished: SUCCESS
```

📸 **Screenshot:** Console Output showing the test result, **BUILD SUCCESS** and **Finished: SUCCESS**.

### Show Poll SCM working

1. On GitHub, open `src/main/java/com/example/App.java` and click the **pencil (Edit)** icon.
2. Change the message, for example to `"Build triggered by Poll SCM!"`, then click **Commit changes**.
3. Go back to Jenkins and wait up to **2 minutes** without clicking anything. Build **#2** starts by itself.
4. Open #2. The top of the page says **Started by an SCM change**.

📸 **Screenshot:** build #2 showing *Started by an SCM change*. You can also click **Polling Log** on the job page to see Jenkins finding the change.

### Show "fix the build if it fails" (optional, good for the viva)

1. On GitHub, edit `src/test/java/com/example/AppTest.java` and change `assertEquals(5, ...)` to `assertEquals(6, ...)`. Commit.
2. The next build turns **red**. Console Output shows `Tests run: 1, Failures: 1` and **BUILD FAILURE**.
3. Change it back to `5` and commit. The next build is **green** again.

📸 **Screenshots:** the failed build and the fixed build.

---

## Errors and fixes

| Error in Console Output | Why it happens | Fix |
| --- | --- | --- |
| `couldn't find remote ref refs/heads/master` | The branch is `main`, not `master` | Configure the job and set **Branch Specifier** to `*/main` |
| `Cannot run program "git"` | Jenkins started before Git was installed | Run `net stop jenkins`, then `net start jenkins` (as Administrator) |
| `Maven3` doesn't appear in the Maven Version list | Maven wasn't added to Jenkins | Do **Part A** again and check the name is exactly `Maven3` |
| `'mvn' is not recognized as an internal or external command` | **Maven Version** in the build step is left on **(Default)**, so Jenkins looks for Maven installed on Windows | Do **Part A** (add `Maven3`), then **Configure** the job and set **Maven Version** to **Maven3**. Also use **Invoke top-level Maven targets**, not *Execute Windows batch command* |
| `release version 17 not supported` | Jenkins is running on an old Java | Install Java 21 and point Jenkins at it (reinstall Jenkins, or edit `jenkins.xml` in `C:\Program Files\Jenkins`) |
| `there is no POM in this directory` | `pom.xml` isn't at the top of the repo | Check that the repo shows `pom.xml` on its main page |
| `Could not transfer artifact ... repo.maven.apache.org` | No internet, or college Wi-Fi blocks Maven | Connect to another network (for example, a phone hotspot) and build again |
| `Tests run: 1, Failures: 1` / `BUILD FAILURE` | The test failed | Fix the code or test on GitHub and commit. Poll SCM rebuilds it |
| Build #2 never starts by itself | Poll SCM isn't ticked, or nothing was committed | Check **Poll SCM** and the schedule `H/2 * * * *`, and check the job's **Polling Log** |

---

## Viva points (Practical 2)

| Question | Answer |
| --- | --- |
| What is Jenkins? | An open-source automation server for CI/CD. It builds, tests and deploys code automatically |
| What is CI? | Continuous Integration: every code change is built and tested automatically, so problems are found early |
| What is a Freestyle job? | A Jenkins job set up through forms (source code, triggers, build steps) rather than a script |
| What is Maven? | A Java build tool that manages libraries through `pom.xml` and compiles, tests and packages the code |
| What is `pom.xml`? | The Project Object Model: the project's name, version, Java version and dependencies |
| What does `clean package` do? | Deletes old build output, then compiles, runs the tests and creates the `.jar` |
| What is Poll SCM? | Jenkins checks the Git repository on a schedule and builds when it finds a new commit |
| What does `H/2 * * * *` mean? | Every 2 minutes. `H` spreads the start time so many jobs don't all run at once |
| Poll SCM vs webhook? | Polling: Jenkins asks GitHub. Webhook: GitHub tells Jenkins. A webhook needs Jenkins to be reachable from the internet, which `localhost` isn't |

## Conclusion (Practical 2)

We installed Jenkins, created a Freestyle job `java-build-job` connected to a Java Maven project on GitHub, set Poll SCM to check for changes every 2 minutes, and built the project with `mvn clean package`. The Console Output showed the test passing and BUILD SUCCESS, and a commit on GitHub started a new build automatically. This shows how Jenkins provides Continuous Integration.

---

# Practical 3: CI/CD Pipeline with Jenkins and Maven

**Q3: Creating a CI/CD pipeline with Jenkins and Maven (30 marks)**

## Aim

Create a Jenkins **Pipeline** job named `cicd-pipeline` that reads a **Jenkinsfile** from this GitHub repo. The pipeline runs five stages (**Checkout, Build, Test, Package, Deploy**), shows them in **Stage View**, and starts automatically when the code on GitHub changes.

## What this practical is about

- **CI/CD:** Continuous Integration (build and test every change) plus Continuous Delivery/Deployment (automatically put the built app where it will run).
- **Pipeline job:** unlike the Freestyle job in Practical 2, the steps are written as code in a file called **Jenkinsfile**. This is called *Pipeline as Code*: the build process lives in Git with the project, so it has version history too.
- **Stages:** each stage is one phase of the pipeline. Jenkins shows each one as a box in **Stage View**: green if it passed, red if it failed.
- **Deploy:** here, deploying means copying the finished `.jar` into a `C:\deploy` folder. In a real company it would be a server or Tomcat.

## Before you start

- Jenkins is installed and running at `http://localhost:8080` (**Practical 2, Part 0**).
- `Maven3` is added under **Manage Jenkins → Tools** (**Practical 2, Part A**). The Jenkinsfile uses this exact name.
- This repo already contains the `Jenkinsfile`, so you don't need to create it.

**Where to do this:** everything happens in **Chrome**: GitHub at `github.com/show-star/aa2` and Jenkins at `http://localhost:8080`. You only need **File Explorer** to check the `C:\deploy` folder at the end.

---

## Part A: The Jenkinsfile (Q3.2, 16 marks)

This file is already in the repo as `Jenkinsfile` (capital J, no extension).

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code checked out from GitHub'
            }
        }
        stage('Build') {
            steps {
                bat 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }
        stage('Package') {
            steps {
                bat 'mvn package -DskipTests'
            }
        }
        stage('Deploy') {
            steps {
                bat 'if not exist C:\\deploy mkdir C:\\deploy'
                bat 'copy /Y target\\*.jar C:\\deploy\\'
                echo 'Deployed jar to C:\\deploy'
            }
        }
    }
}
```

### The Jenkinsfile explained

| Part | Meaning |
| --- | --- |
| `pipeline { }` | The whole declarative pipeline |
| `agent any` | Run on any available machine (your PC) |
| `tools { maven 'Maven3' }` | Use the Maven called `Maven3` from Manage Jenkins → Tools, so `mvn` works |
| `triggers { pollSCM('H/2 * * * *') }` | Check GitHub every 2 minutes and run when there is a new commit |
| `stage('Checkout')` | `checkout scm` downloads the code from the repo the job is linked to |
| `stage('Build')` | `mvn clean compile` deletes old output and compiles `App.java` |
| `stage('Test')` | `mvn test` runs the JUnit test `AppTest` |
| `stage('Package')` | `mvn package` creates `target\java-maven-demo-1.0.jar` (tests already ran, so they are skipped here) |
| `stage('Deploy')` | Creates `C:\deploy` if it doesn't exist and copies the `.jar` into it |
| `bat '...'` | Runs a real Windows command. On Linux this would be `sh '...'` |
| `\\` | Inside a Jenkinsfile, each Windows `\` must be written as `\\` |

---

## Part B: Create the pipeline job (Q3.1, 4 marks)

1. Open **http://localhost:8080** and click **New Item**.
2. Name: **`cicd-pipeline`**. Select **Pipeline** and click **OK**.
3. Scroll down to the **Pipeline** section:
   - **Definition:** `Pipeline script from SCM`
   - **SCM:** `Git`
   - **Repository URL:**
     ```
     https://github.com/show-star/aa2.git
     ```
   - **Branch Specifier:** change `*/master` to **`*/main`**
   - **Script Path:** `Jenkinsfile`
4. Click **Save**.

You don't need to tick Poll SCM on this page. The `triggers` block in the Jenkinsfile turns it on after the first build.

📸 **Screenshot:** the Pipeline section showing Pipeline script from SCM, the repo URL, `*/main` and `Jenkinsfile`.

---

## Part C: Run the pipeline and show Stage View (Q3.3, 5 marks)

1. On the `cicd-pipeline` page, click **Build Now**.
2. Wait for the build to finish. The first build takes a few minutes, because Jenkins downloads Maven and JUnit.
3. The job page shows **Stage View**: five green boxes, **Checkout → Build → Test → Package → Deploy**, each with its time.
   - If you don't see Stage View, go to **Manage Jenkins → Plugins → Available plugins**, search for **Pipeline: Stage View**, install it and restart Jenkins. On newer Jenkins you can also click the build, then **Pipeline Overview**.
4. Click **#1**, then **Console Output**. Near the end you should see:

```
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
...
[INFO] Building jar: C:\ProgramData\Jenkins\.jenkins\workspace\cicd-pipeline\target\java-maven-demo-1.0.jar
[INFO] BUILD SUCCESS
...
        1 file(s) copied.
Deployed jar to C:\deploy
...
Finished: SUCCESS
```

5. Open **File Explorer**, type `C:\deploy` in the address bar and press Enter. The file **`java-maven-demo-1.0.jar`** is there. This proves the Deploy stage worked.

📸 **Screenshots:** Stage View with five green stages, Console Output ending in **Finished: SUCCESS**, and the `C:\deploy` folder with the `.jar`.

---

## Part D: Show the automatic trigger on a code change (Q3.4, 5 marks)

Build #1 must have finished first: that run is what switches on the `pollSCM` trigger from the Jenkinsfile.

1. On GitHub, open `src/main/java/com/example/App.java` and click the **pencil (Edit)** icon.
2. Change the message, for example:
   ```java
   System.out.println("Pipeline auto-triggered! 2 + 3 = " + add(2, 3));
   ```
3. Click **Commit changes**.
4. Go back to the `cicd-pipeline` page in Jenkins and **don't click anything**. Within **2 minutes**, build **#2** starts by itself.
5. Click **#2**. The top of the page says **Started by an SCM change**.
6. Optional: on the job page, click **Polling Log** to see Jenkins finding the new commit.

A GitHub webhook can't reach `localhost` from the internet, so polling is the right automatic trigger for a local Jenkins.

📸 **Screenshots:** the commit on GitHub, build #2 starting on its own in Stage View, and *Started by an SCM change*.

---

## Errors and fixes (Practical 3)

| Error in Console Output | Why it happens | Fix |
| --- | --- | --- |
| `Tool type "maven" does not have an install of "Maven3" configured` | No Maven called `Maven3` in Jenkins | **Manage Jenkins → Tools → Maven installations → Add Maven**, name it exactly `Maven3`, tick Install automatically, Save |
| `'mvn' is not recognized` | The `tools { maven 'Maven3' }` block is missing or misspelled | Check the Jenkinsfile on GitHub matches Part A exactly |
| `couldn't find remote ref refs/heads/master` | The branch is `main` | Set **Branch Specifier** to `*/main` |
| `Unable to find Jenkinsfile` / `does not exist` | Wrong file name or Script Path | The file must be called `Jenkinsfile` exactly (capital J, no `.txt`), and **Script Path** must be `Jenkinsfile` |
| `No such DSL method 'Pipeline'` or `Unknown stage section "step"` | Typo in the Jenkinsfile | It must be lowercase `pipeline` and `steps` with an **s** |
| Deploy stage: `Access is denied` | Jenkins can't write to `C:\deploy` | Create `C:\deploy` yourself in File Explorer, then build again |
| Deploy stage: `The system cannot find the file specified` | The Package stage didn't create the `.jar` | Open the Package stage's log and fix the error shown there |
| Only a list of builds, no stage boxes | The Stage View plugin isn't installed | Install **Pipeline: Stage View** (Part C, step 3) |
| Build #2 never starts on its own | Build #1 never ran, or nothing was committed | Run **Build Now** once, then commit a change on GitHub and wait 2 minutes |
| `Could not transfer artifact ... repo.maven.apache.org` | The network blocks Maven downloads | Switch to another network (for example a phone hotspot) and build again |

---

## Viva points (Practical 3)

| Question | Answer |
| --- | --- |
| What is a Jenkins Pipeline? | A job whose steps are written as code (a Jenkinsfile) and split into stages |
| What is a Jenkinsfile? | A text file in the Git repo that defines the pipeline. This is *Pipeline as Code* |
| Declarative vs scripted pipeline? | Declarative starts with `pipeline { }` and has a fixed, simple structure. Scripted starts with `node { }` and is plain Groovy code |
| What is a stage? | One phase of the pipeline, such as Build or Test, shown as a box in Stage View |
| Freestyle job vs Pipeline job? | Freestyle is set up through forms in Jenkins. Pipeline is code stored in Git, supports stages, and is easier to version and reuse |
| What does `checkout scm` do? | Downloads the code from the repo the job is configured with |
| What is CI/CD? | CI: build and test every change automatically. CD: automatically deliver or deploy the built app |
| Why `bat` and not `sh`? | Jenkins runs on Windows here, so commands run through `cmd.exe` with `bat` |
| How is the pipeline triggered automatically? | `pollSCM('H/2 * * * *')` checks GitHub every 2 minutes and builds when there is a new commit |

## Conclusion (Practical 3)

We created a Jenkins Pipeline job `cicd-pipeline` that reads its Jenkinsfile from GitHub. The pipeline checked out the code, compiled it, ran the JUnit test, packaged it into a `.jar` and deployed it to `C:\deploy`, with every stage shown in Stage View. A commit on GitHub started a new build automatically through Poll SCM. This shows a complete CI/CD pipeline using Jenkins and Maven.
