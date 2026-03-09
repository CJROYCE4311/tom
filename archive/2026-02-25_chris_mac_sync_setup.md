# Chris Mac Sync Setup

This file contains instructions for setting up the `tom` directory sync on Chris's Mac. The server (`caddie-server`) is the go-between — Miki's Mac and Chris's Mac both sync with it every 5 minutes so both always have the latest files.

## Server Details
- **Tailscale IP:** 100.125.74.116
- **SSH login:** chris@100.125.74.116

## Step 1 — Create the tom directory

```
mkdir -p ~/Tom
```

## Step 2 — Pull the files from the server

```
rsync -av chris@100.125.74.116:/home/chris/tom/ ~/Tom/
```

This is a one-time manual pull to get all the existing files onto your Mac.

## Step 3 — Generate an SSH key (so cron can sync without a password)

```
ssh-keygen -t ed25519 -C "chris-macbook" -f ~/.ssh/id_ed25519 -N ""
```

## Step 4 — Copy the key to the server

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub chris@100.125.74.116
```

Enter your password when prompted. This is the last time you'll need it.

## Step 5 — Test passwordless SSH

```
ssh -i ~/.ssh/id_ed25519 chris@100.125.74.116 "echo connected"
```

Should print `connected` with no password prompt.

## Step 6 — Create the sync script

```
mkdir -p ~/scripts
```

Then open a new file in vim:
```
vim ~/scripts/sync_tom.sh
```

In vim:
1. Press `i` to enter insert mode
2. Instead of using vim, run these two commands to write the script safely (replace `YOUR_USERNAME` with your Mac username — run `whoami` if unsure):
```
S=chris@100.125.74.116:/home/chris/tom/
D=/Users/YOUR_USERNAME/Tom/
echo "rsync -av --update $S $D" > ~/scripts/sync_tom.sh
echo "rsync -av --update $D $S" >> ~/scripts/sync_tom.sh
```

## Step 7 — Make the script executable and test it

```
chmod +x ~/scripts/sync_tom.sh && ~/scripts/sync_tom.sh
```

You should see rsync output listing files with no errors.

## Step 8 — Add the cron job

```
crontab -e
```

This opens vim. Press `i`, type this line:
```
*/5 * * * * /Users/YOUR_USERNAME/scripts/sync_tom.sh
```

Replace `YOUR_USERNAME` with your Mac username.

Press **Escape**, type `:wq`, press **Enter**.

## Step 9 — Verify the cron job

```
crontab -l
```

Should show the line you just added.

---

You're done. Your Mac will now sync with the server every 5 minutes. Any changes Miki makes will appear on your Mac within 5 minutes, and vice versa.
