# Windows Setup (client guide)

> [!note] A plain checklist for installing on a Windows PC. Go through it in order, usually together over a screen share. No technical knowledge needed: every step is one download or one paste of a single line.

The system runs natively on Windows. You install a few things once, and then everything works. Go through the steps one by one, and do not move to the next step before the previous one finished. If something gets stuck, stop and ask, no pressure.

Before you start: some installers ask for administrator (admin) rights. That is normal. If a Windows window pops up asking "Do you want to allow this app to make changes?", click **Yes**.

## Step 1: Install Claude Code

This is the brain of the system. You install it through a window called PowerShell.

1. Click the Start button, type `PowerShell`, and open **Windows PowerShell**.
2. Paste the following line (right-click pastes in PowerShell) and press Enter:

```powershell
irm https://claude.ai/install.ps1 | iex
```

3. Wait for the install to finish. When it is done, close the window.

> Note: the install may ask for admin rights. If a confirmation window pops up, click Yes.

## Step 2: Install Git for Windows (this is what runs the system's commands)

This is the most important part of the install. Without it, some of the system's commands simply will not work. With it, everything runs smoothly.

1. Go to: `https://git-scm.com/download/win`
2. The download starts on its own (if not, click the link for 64-bit).
3. Open the file that downloaded and click **Next** on every screen. **Leave all the defaults as they are**, you do not need to change anything. At the end click **Install** and then **Finish**.

> Note: the install asks for admin rights. Click Yes. The reason you install this: it brings a tool called Git Bash, and that is what lets the system run its commands correctly.

## Step 3: Install Node.js (needed for the Google connection)

This is what lets the system connect to your Gmail, Drive, and Calendar, through a tool called gws.

1. Go to: `https://nodejs.org`
2. Click the big button marked **LTS** (this is the stable version).
3. Open the file that downloaded, click **Next** on every screen, and keep the defaults. At the end click **Install** and then **Finish**.

> Note: the install asks for admin rights. Click Yes.

## Step 4: Claude Desktop app (optional, recommended for scheduling)

This is not required, but it is what lets the system run on its own at fixed times (for example a morning report every day at 07:00) through "Routines" (local tasks).

1. Go to: `https://claude.com/download`
2. Download the app for Windows, open the file, and install it.
3. Sign in with the same account.

> Heads-up: this kind of scheduling runs on your computer, so the computer needs to be on at the time the task is supposed to run. This is not a system that runs in the cloud 24/7, it is a push of a result during your working hours.

## Step 5: Install the system

> [!important] The first two commands run in PowerShell, not inside Claude
> You run the marketplace-add and the install in the **PowerShell** window (the same window from the previous steps), not inside the Claude Code screen. Only `/onboard` and `/connect` later run inside Claude.

1. First make sure you have a current version of Claude Code (if not, this updates it on its own). In PowerShell:

```powershell
claude update
claude --version
```

2. Add the marketplace and install the system. In PowerShell, one at a time:

```powershell
claude plugin marketplace add Lowsplix/aios
claude plugin install aios-core@aios
```

3. Open a new empty folder (this will be "the vault", the home of the system). Go into it and open Claude Code inside it (type `claude`), and inside Claude run:

```
/onboard
```

This bootstraps the skeleton and asks you a few short questions to fill the system with you and your business.

4. After onboarding, connect the system to the world (Google, and more):

```
/connect
```

That is it. From here the system is ready. At any time you can run `/doctor` to see that everything is connected and working.

## Optional add-on: WhatsApp connection

The WhatsApp interface (sending and reading messages through the system) works on Windows too. It is based on a tool called wacli that has a Windows build, and the `/connect` command installs it on its own per operating system. This is an optional add-on: even without it the system works great and writes the reports to the vault. If you choose not to connect WhatsApp, just skip this step. The health report to Adir is sent by email anyway, separately.
